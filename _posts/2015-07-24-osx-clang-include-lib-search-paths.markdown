---
layout: post
title: OS X clang include lib search path
---

A [recent](https://github.com/Homebrew/homebrew/commit/9ca3c054c070cc825870a6fc0fc5c5a22f7d8559) (and even more recently [reverted](https://github.com/Homebrew/homebrew/commit/2e191b19b8eccfbc031c224892bda140b1b38e7b)) change to [Homebrew](https://brew.sh) highlighted an interesting (read: maddening) quirk of clang on OS X. Here's the background.

When you compile something using clang on OS X there are (roughly speaking) two stages to the compile. In the first, your source code is loaded and the compiler searches its include paths to find your includes. Here's what clang's defaults look like for the Xcode 6.4 command line tools:

```bash
$ clang -x c -v -E /dev/null
Apple LLVM version 6.1.0 (clang-602.0.53) (based on LLVM 3.6.0svn)
Target: x86_64-apple-darwin14.4.0
Thread model: posix
 "/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang" -cc1 -triple x86_64-apple-macosx10.10.0 -E -disable-free -disable-llvm-verifier -main-file-name null -mrelocation-model pic -pic-level 2 -mdisable-fp-elim -masm-verbose -munwind-tables -target-cpu core2 -target-linker-version 242.2 -v -dwarf-column-info -resource-dir /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/clang/6.1.0 -fdebug-compilation-dir /Users/pkehrer -ferror-limit 19 -fmessage-length 120 -stack-protector 1 -mstackrealign -fblocks -fobjc-runtime=macosx-10.10.0 -fencode-extended-block-signature -fmax-type-align=16 -fdiagnostics-show-option -fcolor-diagnostics -o - -x c /dev/null
clang -cc1 version 6.1.0 based upon LLVM 3.6.0svn default target x86_64-apple-darwin14.4.0
#include "..." search starts here:
#include <...> search starts here:
 /usr/local/include
 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/clang/6.1.0/include
 /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include
 /usr/include
 /System/Library/Frameworks (framework directory)
 /Library/Frameworks (framework directory)
End of search list.
...snip...
```

As you can see, the include search starts with `/usr/local/include` and `/usr/include` is the fourth item. So, in the case of a vanilla OS X install a binary attempting to compile using OpenSSL (let's say `#include <openssl/x509.h>`) would find the OpenSSL headers in `/usr/include` after searching the previous 3 locations fruitlessly.

Once it's found (and the C preprocessor has run and all the objects have been compiled) we now need to link it. When compiling something against OpenSSL you'll link against `-lssl`, `-lcrypto`, or both. These command line flags simply tell the linker to look for something named "libssl.dylib" or "libcrypto.dylib" in the library search paths. But what are those paths?

```bash
$ clang -Xlinker -v
@(#)PROGRAM:ld  PROJECT:ld64-242.2
configured to support archs: armv6 armv7 armv7s arm64 i386 x86_64 x86_64h armv6m armv7m armv7em
Library search paths:
	/usr/lib
	/usr/local/lib
Framework search paths:
	/Library/Frameworks/
	/System/Library/Frameworks/
...snip...
```

This time we see two primary library search paths, `/usr/lib` and `/usr/local/lib`...which are reversed in priority from our include search path.

The result of this is that if you have something (like OpenSSL) that is present in both `/usr/local/{include,lib}` and `/usr/{include,lib}` you'll end up with the compiler using the headers from `/usr/local/include` and then linking against the library in `/usr/lib`. This can result in a variety of problems, the severity of which depend on how different the two versions of the library are and what features the binary you're compiling is using.

So why does this matter? Well, in El Capitan (10.11) Apple has chosen to remove the OpenSSL development headers, but not remove the dylibs. They deprecated use of system OpenSSL in Lion (10.7) so this makes sense on the surface, but the weird include/linker ordering means that if homebrew (or anything else living in your include/search paths) duplicates a system library *bad things* may occur. There are four possible paths (all of which are under the control of Apple and not us plebes):

* Change the linker order preference. Probably a good idea long term but likely to cause all sorts of unintended breakage as we find things that are implicitly depending on this crazy ordering.
* Re-add the OpenSSL headers for 0.9.8. Not a great option since 0.9.8 is scheduled for EOL at the end of this year and Apple has marked it deprecated in OS X since Lion (originally released July 20, 2011), but probably the safest and lowest friction option.
* Remove OpenSSL entirely. This would break any OS X app that links against it and would require Apple to ship updates to Ruby, Python, Apache, etc that statically link OpenSSL (or go down the route of a "private" dylib like they've done with OpenSSH in El Capitan)
* Do nothing and let this be a significant source of pain for developers during El Capitan's lifecycle. This is the most likely scenario.

I favor either removing the OpenSSL dylibs entirely for El Capitan or re-adding the OpenSSL headers and then removing everything in the next major release, but I don't envy whoever has to make this choice. Everything has downsides.
