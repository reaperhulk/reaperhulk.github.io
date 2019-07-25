---
layout: post
title: Memory Unsafety in Apple's Operating Systems
---

Over at <a href="https://twitter.com/lazyfishbarrel">@LazyFishBarrel</a> we've been tweeting statistics about the proportion of memory unsafety related vulnerabilities in various software for a little over a year[^1]. However, while Twitter is a great medium for dispensing this information on a per release basis, it's not great for deeper analysis. Rather than just talking about a single release, what if we aggregated the total memory unsafety-related vulnerability statistics in Apple's two flagship operating systems: iOS and macOS?[^2]

## What is memory unsafety and why should I care?

Memory unsafety[^3] is a property of a programming language that allows the creation of bugs and security vulnerabilities related to memory access. Languages like C and C++[^4] are memory unsafe because they do not enforce bounds checks on memory reads and writes. Instead, the programmer must perfectly allocate, write, read, and deallocate memory or else serious vulnerabilities can easily occur. These bugs are especially frustrating because they represent an entire class of issues that can be **entirely fixed** by moving to languages that do not suffer from these limitations[^5].

## iOS 12

iOS 12 was released September 17, 2018 and has subsequently had a total of 11 feature and point releases. Unlike macOS, the iOS 12 series has several small bugfix releases that did not correct security issues. These are marked as N/A for bugs/CVEs but remain present in the list below.

| Total CVE Count | Memory Unsafety Bugs | Percentage | Release |
| --------------- | -------------------- | ---------- | ------- |
| 37              | 28                   | 75.7%      |<a href="https://support.apple.com/en-us/HT210346">12.4</a> |
| N/A             | N/A                  | N/A        | 12.3.2 |
| N/A             | N/A                  | N/A        | 12.3.1 |
| 42              | 34                   | 81%        | <a href="https://support.apple.com/en-us/HT210118">12.3</a> |
| 51              | 29                   | 56.9%      | <a href="https://support.apple.com/en-us/HT209599">12.2</a> |
| 4               | 2                    | 50%        | <a href="https://support.apple.com/en-us/HT209520">12.1.4</a> |
| 31              | 27                   | 87.1%      | <a href="https://support.apple.com/en-us/HT209443">12.1.3</a> |
| N/A             | N/A                  | N/A        | 12.1.2 |
| 24              | 13                   | 54.2%      | <a href="https://support.apple.com/en-us/HT209340">12.1.1</a> |
| 32              | 20                   | 62.5%      | <a href="https://support.apple.com/en-us/HT209192">12.1</a> |
| 2               | 0                    | 0%         | <a href="https://support.apple.com/en-us/HT209162">12.0.1</a> |
| 38              | 20                   | 52.6%      | <a href="https://support.apple.com/en-us/HT209106">12.0</a> |

Across the entirety of iOS 12 Apple has fixed **261** CVEs, **173** of which were memory unsafety. That's **66.3%** of all vulnerabilities.

## Mojave (aka macOS 10.14)

Apple released macOS 10.14 Mojave on September 24, 2018 and subsequently has issued 6 point releases.

| Total CVE Count | Memory Unsafety Bugs | Percentage | Release  |
| --------------- | -------------------- | ---------- | -------- |
| 44              | 36                   | 81.8%      | <a href="https://support.apple.com/en-us/HT210348">10.14.6</a> |
| 45              | 40                   | 88.9%        | <a href="https://support.apple.com/en-us/HT210119">10.14.5</a> |
| 38              | 20                   | 52.6%        | <a href="https://support.apple.com/en-us/HT209600">10.14.4</a> |
| 23              | 22                   | 95.7%        | <a href="https://support.apple.com/en-us/HT209446">10.14.3</a> |
| 13              | 11                   | 84.6%        | <a href="https://support.apple.com/en-us/HT209341">10.14.2</a> |
| 71              | 40                   | 56.3%        | <a href="https://support.apple.com/en-us/HT209193">10.14.1</a> |
| 64              | 44                   | 68.8%        | <a href="https://support.apple.com/en-us/HT209139">10.14</a> |

Across the entirety of Mojave Apple has fixed **298** CVEs, **213** of which were memory unsafety. That's **71.5%** of all vulnerabilities.

## XNU (macOS only)

The above statistics are for any code that ships with macOS or iOS (including the kernel), but what if we counted just kernel (and kernel extension) issues in macOS[^6]?

By slicing the numbers this way we find that **67** out of **76** vulnerabilities, a whopping **88.2%**, were attributable directly to memory unsafety.

## Bugs vs Security Architecture

At BlackHat USA 2018 <a href="https://twitter.com/i41nbeer">Ian Beer</a> gave a talk called <a href="https://docs.google.com/presentation/d/16LZ6T-tcjgp3T8_N3m0pa5kNA1DwIsuMcQYDhpMU7uU/edit?usp=sharing">the path to EL1 in iOS 11</a>. In it, he discusses Apple's approach to security design and contends that

> security systems design is making promises which poor software development practises cannot keep

He goes on to state that

> given sufficient bug density, security design is irrelevant

As you can see from the above statistics, writing software in memory unsafe languages like C and C++ is providing at least two thirds of that bug density. Additionally, although we have not conducted a specific analysis to this effect, it is relatively rare for logical vulnerabilities to cause code execution (with the exception of JIT bugs), while this is a common outcome for memory corruption. Thus, not only is memory unsafety causing the majority of all security bugs, but those bugs are also more likely to be severe.

Switching languages can't be done overnight, but incremental conversion to a memory safe world needs to be a top priority. Improving the memory safe toolchains, training programmers in new idioms, and phasing out legacy memory unsafe codebases is not a luxury, it's a necessity.

[^1]: And we've been <a href="https://alexgaynor.net/2017/nov/20/a-vulnerability-by-any-other-name/">harping on it</a> for longer than that.
[^2]: No blog post discussing another company's vulnerabilities would be complete without a discussion of the limitations.  Apple provides reasonably well-written descriptions of bugs and these allow us to generally bucket the vulnerabilities into our two categories of interest. However, occasionally a bug will defy easy categorization. For WebKit issues Apple does not distinguish between JIT compiler induced memory unsafety and C/C++ induced.  Apple also occasionally releases fixes that are still embargoed. When this happens they release their security notes, but update them at a later date with additional fixes. They denote these additional entries with an "Entry added on **date**" subscript, but we have typically long-since tweeted about it and won't catch the additional entries.  Since a significant amount of code is shared across Apple's operating systems (especially at the kernel level) many of these CVEs are duplicates across macOS and iOS. This means you cannot sum iOS and macOS bugs together to get a total across Apple's platforms without significantly miscounting.  For the above reasons you should not treat these numbers as **exact** but instead as reasonably accurate estimates of the relative percentage of memory unsafety issues Apple has seen in its products in this time range.
[^3]: We say memory unsafety instead off "not memory safe" because we want to emphasize the fact that these languages are <a href="https://en.wikipedia.org/wiki/Unsafe_at_Any_Speed">unsafe at any speed</a>. You can learn more about how modern languages achieve memory safety with little to no performance penalty in Mozilla's <a href="https://hacks.mozilla.org/2019/01/fearless-security-memory-safety/">Fearless Security: Memory Safety</a> blog post.
[^4]: See <a href="https://alexgaynor.net/2019/apr/21/modern-c++-wont-save-us/">Modern C++ Won't Save Us</a>.
[^5]: Yes, of course migration is difficult! Any company doing it will need to retrain people, make toolchain changes, and undoubtedly invest in improving both the compiler and language itself. If only there was some enormous company with vast resources and a commitment to privacy and security who could embark on a journey like this...
[^6]: iOS and macOS use the same kernel, but since it's a different target the set of vulnerabilities may not entirely overlap. Counting iOS kernel vulnerabilities is left as an exercise for the reader.
