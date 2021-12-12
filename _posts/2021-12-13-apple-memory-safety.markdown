---
layout: post
title: 2021 in Memory Unsafety - Apple's Operating Systems
---

In <a href="/2019/07/23/apple-memory-safety/">2019</a> and <a href="/2020/07/24/apple-memory-safety/">2020</a> we did an analysis of memory unsafety bugs in Apple's iOS and macOS releases. Another year has passed, so let's run the numbers again[^1][^6]. If you're interested in a deeper dive on what memory unsafety is and why it is Fish in a Barrel's mission to rid the world of its scourge please read through the previous posts for more details. And, of course, follow <a href="https://twitter.com/lazyfishbarrel">@LazyFishBarrel</a> on Twitter where we've been tweeting statistics about the proportion of memory unsafety related vulnerabilities in various software for over three years.

## iOS/iPadOS

iOS 14 was released September 16, 2020 and has subsequently had a total of 15 feature and point releases. One significant change from last year is Apple's new policy of providing limited security patching to the N-1 version of their OS after N is released. This has resulted in version 14.8.1 after the release of 15.x. It is, however, important to note that Apple has not committed to patching all security bugs and that in general security is not just bug fixes, it's also fundamental software architecture choices that cannot be backported. The best security policy is always to run the latest version.

| Release                                                      | Memory Unsafety Bugs | Total CVE Count |
| ------------------------------------------------------------ | -------------------- | --------------- |
| <a href="https://support.apple.com/en-us/HT212868">14.8.1</a>| 10                   | 12              |
| <a href="https://support.apple.com/en-us/HT212807">14.8</a>  | 12[^2]               | 19              |
| <a href="https://support.apple.com/en-us/HT212623">14.7.1</a>| 1                    | 1               |
| <a href="https://support.apple.com/en-us/HT212601">14.7</a>  | 26[^2]               | 38              |
| <a href="https://support.apple.com/en-us/HT212528">14.6</a>  | 28[^2]               | 48              |
| <a href="https://support.apple.com/en-us/HT212336">14.5.1</a>| 2                    | 2               |
| <a href="https://support.apple.com/en-us/HT212317">14.5</a>  | 32[^2]               | 63              |
| <a href="https://support.apple.com/en-us/HT212256">14.4.2</a>| 0                    | 1               |
| <a href="https://support.apple.com/en-us/HT212221">14.4.1</a>| 1                    | 1               |
| <a href="https://support.apple.com/en-us/HT212146">14.4</a>  | 28[^2]               | 56              |
| <a href="https://support.apple.com/en-us/HT212003">14.3</a>  | 15                   | 18              |
| 14.2.1                                                       | N/A                  | N/A             |
| <a href="https://support.apple.com/en-us/HT211929">14.2</a>  | 23                   | 32              |
| 14.1                                                         | N/A                  | N/A             |
| 14.0.1                                                       | N/A                  | N/A             |
| <a href="https://support.apple.com/en-us/HT211850">14.0</a>  | 31                   | 55              |

This totals 209 out of 346 vulnerabilities, for a **60.4%** rate.

### In the wild

A new feature this year was Apple's willingness to mark vulnerabilities that were being actively exploited in the wild.

| Release | Memory Unsafety | Total |
| ------- | --------------- | ----- |
| 14.8.1  | 1               | 1     |
| 14.8    | 2               | 2     |
| 14.7.1  | 1               | 1     |
| 14.5.1  | 2               | 2     |
| 14.5    | 1               | 1     |
| 14.4.2  | 0               | 1     |
| 14.4    | 0               | 3     |

This totals 8 out of 11 vulnerabilities, for a **72.7%** rate.

### Aggregated Stats

| Release | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ------- | -------------------- | --------------- | ---------- |
| iOS 14  | 209                  | 346             | 60.4%      |
| iOS 13  | 107                  | 180[^3]         | 59.4%      |
| iOS 12  | 173                  | 261             | 66.3%      |
| iOS 11  | 187                  | 309             | 60.5%      |

There has been no meaningful change in the prevalence of memory unsafety bugs in the window analyzed. Apple has begun to use memory safe constructions in some components of their OS (see: <a href="https://googleprojectzero.blogspot.com/2021/01/a-look-at-imessage-in-ios-14.html">BlastDoor</a>), but huge surface area remains accessible (looking at you ImageIO).

## macOS

Apple released macOS 11 (Big Sur) on November 12, 2020. Version 11.0 was preinstalled on some M1 machines, but 11.0.1 was actually released to customers first and contains all the relevant CVE information.

| Release                                                                             | Memory Unsafety Bugs | Total CVE Count |
| ----------------------------------------------------------------------------------- | -------------------- | --------------- |
| <a href="https://support.apple.com/en-us/HT212872">11.6.1</a>                       | 18                   | 24              |
| <a href="https://support.apple.com/en-us/HT212804">11.6</a>                         | 12                   | 21              |
| 11.5.2                                                                              | N/A                  | N/A             |
| <a href="https://support.apple.com/en-us/HT212622">11.5.1</a>                       | 1                    | 1               |
| <a href="https://support.apple.com/en-us/HT212602">11.5</a>                         | 19[^2]               | 40              |
| <a href="https://support.apple.com/en-us/HT212529">11.4</a>                         | 34[^2]               | 82              |
| <a href="https://support.apple.com/en-us/HT212335">11.3.1</a>                       | 2                    | 2               |
| <a href="https://support.apple.com/en-us/HT212325">11.3</a>                         | 30 [^2]              | 65              |
| <a href="https://support.apple.com/en-us/HT212220">11.2.3</a>                       | 1                    | 1               |
| 11.2.2                                                                              | N/A                  | N/A             |
| <a href="https://support.apple.com/en-us/HT212177">11.2.1</a>                       | 3                    | 3               |
| <a href="https://support.apple.com/en-us/HT212147">11.2</a>                         | 29[^2][^5]           | 57              |
| <a href="https://support.apple.com/en-us/HT212011">11.1</a>                         | 21[^5]               | 27              |
| <a href="https://support.apple.com/en-us/HT211931">11.0.1</a>                       | 55                   | 102             |

This totals 225 out of 425 vulnerabilities, for a **52.9%** rate. Please see the footnotes in the table for some caveats.

Despite these limitations, let's aggregate the data again.

| Release      | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ------------ | -------------------- | --------------- | ---------- |
| macOS 11     | 225                  | 425             | 52.9%      |
| macOS 10.15  | 154                  | 228             | 67.5%      |
| macOS 10.14  | 213                  | 298             | 71.5%      |
| macOS 10.13  | 256                  | 458             | 55.9%      |

These numbers are significantly more volatile, but the average over the past 4 years is ~60%, in line with iOS.

### In the wild

Much like iOS, for Big Sur Apple now notes vulnerabilities that were being actively exploited in the wild.

| Release | Memory Unsafety | Total |
| ------- | --------------- | ----- |
| 11.6    | 2               | 2     |
| 11.5.1  | 1               | 1     |
| 11.4    | 0               | 1     |
| 11.3.1  | 2               | 2     |
| 11.3.1  | 2               | 2     |
| 11.3    | 1               | 2     |
| 11.2    | 0               | 2     |
| 11.0.1  | 3               | 3     |

This totals 11 out of 15 vulnerabilities, for a **73.3%** rate.


### XNU (macOS only)

The above statistics are for any code that ships with macOS or iOS (including the kernel), but what if we counted just kernel (and kernel extension) issues in macOS[^4]?


| Release     | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ----------- | -------------------- | --------------- | ---------- |
| macOS 11    | 48                   | 69              | 69.6%      |
| macOS 10.15 | 58                   | 65              | 89.2%      |
| macOS 10.14 | 67                   | 76              | 88.2%      |
| macOS 10.13 | 65                   | 88              | 73.9%      |

The vast majority of kernel-related CVEs continue to be memory unsafety related, although there is a drop with macOS 11. Hopefully that's the beginning of a trend...

## Analysis (Bugs vs Security Architecture Yearly Statement)

Memory unsafety continues to dominate the total percentage of security bugs on Apple's platforms. Engineers there should **strenuously lobby management** to prioritize migration of code that parses untrusted input to memory safe languages. Doing so would drastically cut the number of security vulnerabilities. In some cases this work appears to have begun (e.g. BlastDoor), but extensive use of memory unsafe libraries like ImageIO weaken the protections. Migrations must be done incrementally, so Apple should continue to identify their most vulnerable surface area and systematically harden it. Might we suggest ImageIO as a candidate?

Major investment should also be undertaken in determining how to migrate XNU itself. Moving kernel extensions to a C++ userland (via the <a href="https://developer.apple.com/system-extensions/">system extension</a> concept) will mitigate the severity of vulnerabilities in some cases, but is ultimately a poor substitute for a memory safe solution. The <a href="https://github.com/Rust-for-Linux">Rust for Linux</a> project, while nascent itself, is demonstrating the potential viability of memory safe languages in a kernel.

As we note every year, <a href="https://twitter.com/i41nbeer">Ian Beer</a> once said:

> given sufficient bug density, security design is irrelevant

Apple creates robust security architectures, but their systematic under-investment in memory safety (and even some mitigations like pervasive fuzzing) remains their single biggest weakness.

<br><br><br>

[^1]: No blog post discussing another company's vulnerabilities would be complete without a discussion of the limitations. Apple provides reasonably well-written descriptions of bugs and these allow us to generally bucket the vulnerabilities into our two categories of interest. However, occasionally a bug will defy easy categorization or we will make a mistake. For WebKit issues Apple **does not distinguish between JIT compiler induced memory unsafety and C/C++ induced**. Apple also occasionally releases fixes that are still embargoed. When this happens they release their security notes, but update them at a later date with additional fixes. They denote these additional entries with an "Entry added on **date**" subscript, but we have typically long-since tweeted about it and won't catch the additional entries. This blog post performed a recount as part of the analysis rather than relying on past data.
[^6]: Since a significant amount of code is shared across Apple's operating systems (especially at the kernel level) many of these CVEs are duplicates across macOS and iOS. This means you cannot sum iOS and macOS bugs together to get a total across Apple's platforms without significantly miscounting. Finally, Apple typically does not document security bugs they find and fix internally. Thus, you should not treat these numbers as **exact** but instead as reasonably accurate estimates of the relative percentage of memory unsafety issues Apple has seen in its products in this time range.
[^2]: Apple's information about several vulnerabilities is insufficient to categorize memory unsafety. The biggest offender are the ImageIO vulnerabilities. Some of them are clearly labeled, but many fall into a "This issue was addressed with improved checks" bucket which tells us nothing about the underlying issue. When we do not have enough information we generally do **not** count them as memory unsafety, even if it seems likely that they are.
[^3]: These numbers are low as they have not been recomputed with Apple's updated documents.
[^4]: iOS and macOS use the same kernel, but since it's a different target the set of vulnerabilities may not entirely overlap. Counting iOS kernel vulnerabilities is left as an exercise for the reader. Our counting methodology here is to use Apple's own information about whether kernel memory is affected (which generally covers kernel extensions like IOMobileFrameBuffer, even though they are not directly labeled kernel).
[^5]: In macOS Big Sur 11.1 and 11.2 Apple listed both Catalina and Mojave security updates within the same document. These numbers are just for bugs affecting Big Sur.
