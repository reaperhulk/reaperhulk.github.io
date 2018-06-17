---
layout: post
title: -Wunused-command-line-argument-hard-error-in-future is a harsh mistress
---

Apple updated the Xcode command line tools today, which upgraded clang to `Apple LLVM version 5.1 (clang-503.0.38) (based on LLVM 3.4svn)` and enabled a feature in clang that causes unknown compiler flags to hard error rather than warn. The error is accompanied by an amusing/frustrating note:

```
clang: error: unknown argument: '-mno-fused-madd' [-Wunused-command-line-argument-hard-error-in-future]
clang: note: this will be a hard error (cannot be downgraded to a warning) in the future
```

Apparently the future is here.

This is a serious problem because several common gcc flags are not supported under clang (most notably `-mno-fused-madd`).

For now you can work around the issue using

```bash
export ARCHFLAGS="-Wno-error=unused-command-line-argument-hard-error-in-future"
```

but be aware that this will stop working in a future clang.
