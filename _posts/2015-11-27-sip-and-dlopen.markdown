---
layout: post
title: System Integrity Protection and dlopen
---

We recently encountered an interesting bug in [PyCA cryptography](https://github.com/pyca/cryptography) (A Python cryptography library) that we discovered was related to [System Integrity Protection](https://en.wikipedia.org/wiki/System_Integrity_Protection) on OS X 10.11 El Capitan. This post is an attempt to document the process used to understand and resolve the problem.

## The Report

A user filed an issue stating that they're running El Capitan using Python 2.7.10 and got the following traceback when attempting to import cryptography:

```
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/backends/openssl/__init__.py", line 7, in <module>
    from cryptography.hazmat.backends.openssl.backend import backend
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/backends/openssl/backend.py", line 43, in <module>
    from cryptography.hazmat.bindings.openssl import binding
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/bindings/openssl/binding.py", line 182, in <module>
    Binding.init_static_locks()
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/bindings/openssl/binding.py", line 139, in init_static_locks
    cls._ensure_ffi_initialized()
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/bindings/openssl/binding.py", line 134, in _ensure_ffi_initialized
    cls._register_osrandom_engine()
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/bindings/openssl/binding.py", line 99, in _register_osrandom_engine
    _openssl_assert(cls.lib, cls.lib.ERR_peek_error() == 0)
  File "/Library/Python/2.7/site-packages/cryptography/hazmat/bindings/openssl/binding.py", line 43, in _openssl_assert
    errors
cryptography.exceptions.InternalError: Unknown OpenSSL error. Please file an issue at https://github.com/pyca/cryptography/issues with information on how to reproduce this.
```

The same user also reports that downgrading from version 1.1 to version 1.0.2 resolves their problem.

Great, let's investigate...

## Initial Investigation

The OpenSSL backend cryptography uses requires initialization on import to register the locking callbacks and set up the C library for use. As part of this process we assert that the error stack is empty before adding an engine. The exception is occurring when calling ```_openssl_assert(cls.lib, cls.lib.ERR_peek_error() == 0)```, which raises the exception in ```_openssl_assert```. The implementation of ```_openssl_assert``` is below along with ```_consume_errors```.

```
def _consume_errors(lib):
    errors = []
    while True:
        code = lib.ERR_get_error()
        if code == 0:
            break

        err_lib = lib.ERR_GET_LIB(code)
        err_func = lib.ERR_GET_FUNC(code)
        err_reason = lib.ERR_GET_REASON(code)

        errors.append(_OpenSSLError(code, err_lib, err_func, err_reason))
    return errors


def _openssl_assert(lib, ok):
    if not ok:
        errors = _consume_errors(lib)
        raise InternalError(
            "Unknown OpenSSL error. Please file an issue at https://github.com"
            "/pyca/cryptography/issues with information on how to reproduce this."
        )
```

This error isn't showing up in our continuous integration environment, nor does it appear on any developer Mac when they make a new virtualenv with system Python (```virtualenv venv -p /usr/bin/python```). Unfortunately, the released version of the library that is exhibiting this problem does not log the actual OpenSSL error message. This oversight requires us to engage the reporter and get them to make a small change to the library to get the error message. Once we do that we get ```_OpenSSLError(code=621174887L, lib=37, func=102, reason=103)``` as our error data on the top of the stack.

If you feed that error code to ```ERR_error_string``` you'll get ```error:25066067:DSO support routines:DLFCN_LOAD:could not load the shared library```[^0]. Why is this not occurring on any developer's machine? Maybe we need to replicate the circumstances more directly.

## Narrowing In

Since no one wants to ```sudo pip install``` on their own machine[^1] we'll go ahead and build an El Capitan VM. Once that's done, ```sudo pip install cryptography``` and then ```/usr/bin/python -c "from cryptography.hazmat.backends.openssl.backend import backend"``` ... and we got an exception! Hooray, we've replicated it!

Now that we have an environment exhibiting the user's problem let's try to narrow it down. Rather than just importing, let's look at what calls are made to the library during import and check for error codes after each one. Doing this, we find that the error code is added to the stack when calling `SSL_library_init`.[^2]

From the error code we know the error is occurring when we call ```DSO_load```. This must be called during `SSL_library_init`. Looking at the [source code](https://github.com/openssl/openssl/blob/33dd08320648ac71d7d9d732be774ed3818dccc5/ssl/ssl_algs.c#L64) we can see some calls to various `EVP` functions. Those are dead ends. However, at the end there's a call to `SSL_COMP_get_compression_methods`. This, in turn, calls `load_builtin_compressions`, which calls `COMP_zlib`:

```cpp
COMP_METHOD *COMP_zlib(void)
{
    COMP_METHOD *meth = &zlib_method_nozlib;

#ifdef ZLIB_SHARED
    if (!zlib_loaded) {
# if defined(OPENSSL_SYS_WINDOWS) || defined(OPENSSL_SYS_WIN32)
        zlib_dso = DSO_load(NULL, "ZLIB1", NULL, 0);
# else
        zlib_dso = DSO_load(NULL, "z", NULL, 0);
# endif
/* ... */
```

So, provided OpenSSL is built with `zlib-dynamic` (the default), it will attempt (and sometimes fail) to load "z" with the code above.

## Isolation

What is going on here? If we use the same VM but install cryptography into a virtual environment it all works. On Linux we could maybe see what's going on if we used ```ltrace```, so let's try using some ```dtrace``` tooling to see what's happening:

```dtrace: failed to execute /usr/bin/python: dtrace cannot control executables signed with restricted entitlements```

Oh right, SIP disables dtrace on system executables. That's really annoying! Well, we're in a VM so we can boot into the recovery volume and disable SIP from the terminal with ```csrutil disable```.

Right, now that we've done that we can attach dtrace[^3], but first let's confirm that the error still occurs... and it doesn't?! What the hell? How could SIP possibly affect this? Commence rebooting with SIP on and off several times to confirm we're not crazy...

## Explanation

`DSO_load` ultimately calls `dlopen`, which expects to find zlib via the standard library fallback paths. When executing a system binary like ```/usr/bin/python``` certain SIP restrictions are in place. One of those restrictions is the disabling of library fallback paths when calling `dlopen`. Without the fallback paths available OpenSSL is unable to find the library and adds the original error to the stack.

This does not occur when executing inside a virtual environment because virtualenv copies a binary that depends on a symlink to the ```/System/Library/Frameworks``` Python framework path and somehow[^4] that evades SIP. To see this in action if you're on El Capitan:

```
/usr/bin/python -c "import ctypes; ctypes.cdll.LoadLibrary('libz.dylib')"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/ctypes/__init__.py", line 443, in LoadLibrary
    return self._dlltype(name)
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/ctypes/__init__.py", line 365, in __init__
    self._handle = _dlopen(self._name, mode)
OSError: dlopen(libz.dylib, 6): image not found
```

But in a virtualenv you will get no error. Similarly, if you change the above call to: ```/usr/bin/python -c "import ctypes; ctypes.cdll.LoadLibrary('/usr/lib/libz.dylib')"``` it will work without issue.

In our case the solution is relatively simple. TLS compression is a security vulnerability[^5] thanks to attacks like [CRIME](https://en.wikipedia.org/wiki/CRIME), so we can just rebuild our own OpenSSL with ```no-comp```, which disables this entire code path as well as improving the baseline security of the library. This improvement (along with several other small fixes) was released to PyPI as version 1.1.1.

Bugs like this are difficult to track down for countless reasons, but in this case once the problem was isolated the challenge was understanding **why** the issue was occurring. Apple does not appear to document this fallback path restriction, so we were reduced to experimenting until people we know at Apple could inquire on internal mailing lists. This is especially unfortunate as most people do not have access to avenues of inquiry like that. While SIP is fascinating technology, it is moderately frustrating to run into highly opaque errors like this and end up wasting days or even weeks of time trying to understand the black box within.

[^1]: Seriously, don't do that people.

[^2]: This also explains why the problem was apparently resolved by downgrading. In 1.0.2 we called `SSL_library_init` later in the initialization process, so the `ERR_peek_error` call didn't see the error on the stack since it hadn't been added yet. We should be more rigorous in checking the stack even in edge cases like this where the documentation says things like `SSL_library_init() always returns "1", so it is safe to discard the return value.`. Just because the return value is always 1 doesn't mean the error stack can't have things on it!

[^3]: Yeah, you apparently still can't. Hopefully I'm wrong about this.

[^4]: If you understand why this is the case, please let me know!

[^5]: There are edge cases where it is safe to use TLS compression but in general it should be considered best practice to disable it permanently.

[^0]: Somehow I (note the singular and not collective in this instance) managed to actually look up the wrong error code at first, thus leading us down a blind alley for almost a week. Whoops.
