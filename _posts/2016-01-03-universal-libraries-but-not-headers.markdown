---
layout: post
title: OS X Universal Libraries...But Not Headers
---

Time for another fun [PyCA/cryptography](https://github.com/pyca/cryptogpraphy) bug story!

[hynek](https://hynek.me) filed an [issue](https://github.com/pyca/pyopenssl/issues/391) against [pyOpenSSL](https://github.com/pyca/pyopenssl) stating that the test suite was crashing on OS X with a `SIGBUS`. The crash dump looked like this:

```
* thread #1: tid = 0x15138, 0x0000000104c73c40 _openssl.so`EC_GFp_nistp224_method, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=2, address=0x104c73c40)
    frame #0: 0x0000000104c73c40 _openssl.so`EC_GFp_nistp224_method
_openssl.so`EC_GFp_nistp224_method:
->  0x104c73c40 <+0>: addb   %al, (%rax)
    0x104c73c42 <+2>: addb   %al, (%rax)
    0x104c73c44 <+4>: addb   %al, (%rax)
    0x104c73c46 <+6>: addb   %al, (%rax)


* thread #1: tid = 0x15138, 0x0000000104c73c40 _openssl.so`EC_GFp_nistp224_method, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=2, address=0x104c73c40)
  * frame #0: 0x0000000104c73c40 _openssl.so`EC_GFp_nistp224_method
    frame #1: 0x0000000104a6b749 _openssl.so`EC_GROUP_new_by_curve_name + 249
    frame #2: 0x0000000104a6e2d1 _openssl.so`EC_KEY_new_by_curve_name + 37
    frame #3: 0x0000000104b37fbd _openssl.so`_cffi_f_EC_KEY_new_by_curve_name + 61
    ... (snipped about 150 lines of irrelevant Python frames) ...
```

Looking at the tests I was able to find the specific test that caused the crash, so the first step was to pull that test out as a standalone script:

```python
from cryptography.hazmat.backends.openssl.backend import backend
from cryptography.hazmat.primitives.asymmetric import ec


vector = {'y': 17502091976454499611505625879374774917623246385936044091564633236054L, 'x': 1075547452530705099852971289727135188477809665711500674593974877078L, 'curve': 'secp224r1', 'd': 24409883047820789832454338490434427874387169505509737608417835723568L}

numbers = ec.EllipticCurvePrivateNumbers(
    vector['d'],
    ec.EllipticCurvePublicNumbers(
        vector['x'],
        vector['y'],
        ec.SECP224R1()
    )
)
key = numbers.private_key(backend)
```

From there, looking at the stack trace we can see that `EC_GROUP_new_by_curve_name` is the function called directly before `EC_GFp_nistp224_method`, so we can look in the cryptography source code to see where it's used. Does this crash?

```python
from cryptography.hazmat.backends.openssl.backend import backend
from cryptography.hazmat.primitives.asymmetric import ec

backend.elliptic_curve_supported(ec.SECP224R1())
```

`Bus Error: 10`

Okay, what if we remove everything but the shared object we build from this? We convert the `ec.SECP224R1()` object to the integer `713` before calling `EC_GROUP_new_by_curve_name` so let's just call it directly...

`python -c "from cryptography.hazmat.bindings._openssl import lib; lib.EC_GROUP_new_by_curve_name(713)`

`Bus Error: 10`

Let's try recompiling OpenSSL. We use Homebrew, but custom compile by editing the brew formula to use `no-comp` instead of `zlib-dynamic`. This problem cropped up after we switched to compiling our own. If we just use the bottle it doesn't crash. Maybe looking at the symbol table would sort this out?

```
nm /usr/local/opt/openssl/lib/libcrypto.a
... snip ...
0000000000000000 T _EC_GFp_nistp224_method
```

Okay, so it's present. What about in the broken install?

```
nm /usr/local/opt/openssl/lib/libcrypto.a
... snip ...
0000000000000000 T _EC_GFp_nistp224_method
```

No help there. The symbol is present in both. What if we don't even use Python here?

```cpp
#include <openssl/ec.h>

int main(void) {
    EC_GROUP_new_by_curve_name(713);
}
```

We can compile that with `clang crash.c -I/usr/local/opt/openssl/include /usr/local/opt/openssl/lib/libcrypto.a`[^1]. When we execute the resulting `a.out` we get...no crash.

This at least tells us that it's not likely to be a linker bug. Let's look at the compile phase in Python. When compiling we have [this conditional](https://github.com/pyca/cryptography/blob/master/src/_cffi_src/openssl/ec.py#L362-L371):


```cpp
#if defined(OPENSSL_NO_EC) || OPENSSL_VERSION_NUMBER < 0x1000100f || \
    defined(OPENSSL_NO_EC_NISTP_64_GCC_128)
static const long Cryptography_HAS_EC_NISTP_64_GCC_128 = 0;

const EC_METHOD *(*EC_GFp_nistp224_method)(void) = NULL;
const EC_METHOD *(*EC_GFp_nistp256_method)(void) = NULL;
const EC_METHOD *(*EC_GFp_nistp521_method)(void) = NULL;
#else
static const long Cryptography_HAS_EC_NISTP_64_GCC_128 = 1;
#endif
```

The reason this exists is because cryptography binds functions that may or may not be available. Unfortunately, cffi provides us no method by which we can say "include this symbol if present, otherwise leave it out". Instead we need to call `cdef` to declare our types and functions, but we don't actually know if those functions exist until we invoke the compiler/linker. To handle this we developed a pattern whereby we use the C preprocessor to introspect our OpenSSL (via the headers) and, if symbols are missing, we declare them as null function pointers (or other values if they're not functions) and set a flag. During import of the Python module we then iterate over the set of conditional symbols and remove all attributes that aren't present from the `lib` object.

Remember how we're getting a `SIGBUS`? A [bus error](https://en.wikipedia.org/wiki/Bus_error) tells the OS that the memory is invalid for the address bus. A null function pointer would definitely cause it to attempt to access invalid memory, but `OPENSSL_NO_EC_NISTP_64_GCC_128` shouldn't be present in the headers right? After all, in the homebrew build flags we see:

```ruby
def arch_args
  {
    :x86_64 => %w[darwin64-x86_64-cc enable-ec_nistp_64_gcc_128],
    :i386   => %w[darwin-i386-cc]
  }
end
```

But a quick look at the headers shows:

```cpp
#ifndef OPENSSL_NO_EC_NISTP_64_GCC_128
# define OPENSSL_NO_EC_NISTP_64_GCC_128
#endif
```

That...should not be.

## Explanation

Many users of `cryptography` consume it via system Python. This Python is a [universal binary](https://en.wikipedia.org/wiki/Universal_binary). To build a wheel for a universal Python you need a universal library on OS X. Homebrew provides this by default in the bottle for OpenSSL (and other packages), but if you recompile yourself you can specify `--universal` on the command line as well. This is accomplished by invoking the build for OpenSSL twice and stitching the resulting libraries and binaries together using a tool Apple provides called `lipo`.

When compiling cryptography (or any project) against a universal lib you specify the headers and then the library location (if they're not already in your include and lib search paths) and then the compiler does the rest. Unfortunately, universal libraries assume that the headers will be the same between architectures. This is not true for any project that leaks build flags into the public headers. In OpenSSL's case this comes through with `OPENSSL_NO_EC_NISTP_64_GCC_128`. The conditional code shown a bit above declares `EC_GFp_nistp224_method` (and friends) as NULL function pointers if `OPENSSL_NO_EC_NISTP_64_GCC_128` is set. Unfortunately, while this is set on the i386 build, it is **not** set on x86_64. Homebrew, when building a universal library, chooses to keep the headers from whatever is the last architecture to build. In this case it happened to be i386. So, when cryptography compiles the C preprocessor dutifully says that `OPENSSL_NO_EC_NISTP_64_GCC_128` is set and therefore `EC_GFp_nistp224_method`, etc are not present and should be defined as NULL function pointers.

Of course, this should still be caught right? After all, the `ec.h` header contains a different declaration for this function and we explicitly include the headers so that any mismatch will cause a compiler error! Let's look at the header file!

```cpp
# ifndef OPENSSL_NO_EC_NISTP_64_GCC_128
/** Returns 64-bit optimized methods for nistp224
 *  \return  EC_METHOD object
 */
const EC_METHOD *EC_GFp_nistp224_method(void);

/** Returns 64-bit optimized methods for nistp256
 *  \return  EC_METHOD object
 */
const EC_METHOD *EC_GFp_nistp256_method(void);

/** Returns 64-bit optimized methods for nistp521
 *  \return  EC_METHOD object
 */
const EC_METHOD *EC_GFp_nistp521_method(void);
# endif
```

:sob:

But wait, even if this all gets compiled the symbol is still present in the `libcrypto.a` right? Why isn't that called? I'm not entirely clear on this but I believe the linker prioritizes the locally available symbol over what's in the `.a` file. Similarly, this doesn't occur if you dynamically link rather than statically link.

Finally, if we're removing the function conditionally why is it being called and failing? Well, internally `EC_GROUP_new_by_curve_name` (and a few other functions) call it. Since the x86\_64 build **was** compiled with `EC_NISTP_64_GCC_128` support they believe the function is available, but when they call it they get the null function pointer and it crashes. This is from the C layer, so the removal of the Python attribute is not relevant.

## Summary

So, when running in 64-bit mode linked against a universal OpenSSL built using homebrew cryptography 1.1.1 will `SIGBUS` due to a header mismatch during compilation causing a symbol to be improperly defined as a NULL function pointer. This affects many scenarios involving elliptic curve keys on OS X in cryptography 1.1.1 (but not 1.1.2+).

The functions in question have no need to be bound at all (they're really internal implementation details and are of no benefit to us), so dropping them from the bindings resolved this issue. However, the underlying issue with architecture mismatch in the homebrew formula remains and we'll need to put in a patch to Homebrew to resolve it at some point[^3].

In 1.1.2 we have removed the bindings, but the null function pointers are still present. I ultimately would like to transition to something that looks more like:

```cpp
 const EC_METHOD *EC_GFp_nistp224_method(void) {
     printf("fake EC_GFp_nistp224_method was called. Aborting!\n");
     abort();
 }
```

rather than

```cpp
const EC_METHOD *(*EC_GFp_nistp224_method)(void) = NULL;
```

to reduce the confusion if this happens again in the future[^2].

[^1]: Note the use of the static library because that's how we compile wheels for OS X. The paths also assume the default install location for homebrew here.

[^2]: This could also be improved if cffi's cdef allowed C preprocessor checks inside `cdef`, but that is not yet available.

[^3]: Tim Smith has submitted a [PR](https://github.com/Homebrew/homebrew/pull/47652) to resolve this.
