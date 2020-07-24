---
layout: post
title: The Year in Memory Unsafety - Apple's Operating Systems
---

<a href="/2019/07/23/apple-memory-safety/">Last year</a> we did an analysis of memory unsafety bugs in Apple's iOS 12.x and macOS 10.14.x releases. We're now at (nearly) the same point in the release cycle of iOS 13/macOS 10.15 so let's run the numbers again[^1]. If you're interested in a deeper dive on what memory unsafety is and why it is Fish in a Barrel's mission to rid the world of its scourge please read through last year's post for the details. And, of course, follow <a href="https://twitter.com/lazyfishbarrel">@LazyFishBarrel</a> on Twitter where we've been tweeting statistics about the proportion of memory unsafety related vulnerabilities in various software for over two years.

## iOS/iPadOS

iOS 13 was released September 19, 2019 and has subsequently had a total of 15 feature and point releases. Unlike macOS, the iOS 13 series has several small bugfix releases that did not correct security issues. These are marked as N/A for bugs/CVEs but remain present in the list below.

| Release                                                      | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ------------------------------------------------------------ | -------------------- | --------------- | ---------- |
| <a href="https://support.apple.com/en-us/HT211288">13.6</a>  | 15                   | 29              | 51.7%      |
| <a href="https://support.apple.com/en-us/HT211168">13.5</a>  | 28                   | 45              | 62.2%      |
| 13.4.1                                                       | N/A                  | N/A             | N/A        |
| <a href="https://support.apple.com/en-us/HT211102">13.4</a>  | 14                   | 30              | 46.7%      |
| <a href="https://support.apple.com/en-us/HT210918">13.3.1</a>| 15                   | 23              | 65.2%      |
| <a href="https://support.apple.com/en-us/HT210785">13.3</a>  | 11                   | 14              | 78.6%      |
| 13.2.3                                                       | N/A                  | N/A             | N/A        |
| 13.2.2                                                       | N/A                  | N/A             | N/A        |
| 13.2.1                                                       | N/A                  | N/A             | N/A        |
| <a href="https://support.apple.com/en-us/HT210721">13.2</a>  | 21                   | 28              | 75%        |
| 13.1.3                                                       | N/A                  | N/A             | N/A        |
| 13.1.2                                                       | N/A                  | N/A             | N/A        |
| 13.1.1                                                       | N/A                  | N/A             | N/A        |
| <a href="https://support.apple.com/en-us/HT210603">13.1</a>  | 0                    | 1               | 0%         |
| <a href="https://support.apple.com/en-us/HT210606">13.0</a>  | 2                    | 9               | 22.2%      |

It would be interesting to aggregate statistics across major versions, but we only have data for iOS 12[^2] and 13. Let's fix that by going back through Apple's security docs and running the numbers for iOS 11 (we did not run the Fish in a Barrel Twitter account back then).

| Release                                                       | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ------------------------------------------------------------- | -------------------- | --------------- | ---------- |
| <a href="https://support.apple.com/en-us/HT208938">11.4.1</a> | 16                   | 24              | 66.7%      |
| <a href="https://support.apple.com/en-us/HT208848">11.4</a>   | 16                   | 39              | 41.0%      |
| <a href="https://support.apple.com/en-us/HT208743">11.3.1</a> | 3                    | 4               | 75.0%      |
| <a href="https://support.apple.com/en-us/HT208693">11.3</a>   | 31                   | 60              | 51.7%      |
| <a href="https://support.apple.com/en-us/HT208534">11.2.6</a> | 1                    | 1               | 100%       |
| <a href="https://support.apple.com/en-us/HT208463">11.2.5</a> | 13                   | 17              | 76.5%      |
| <a href="https://support.apple.com/en-us/HT208401">11.2.2</a> | 0                    | 2               | 0%         |
| <a href="https://support.apple.com/en-us/HT208357">11.2.1</a> | 0                    | 1               | 0%         |
| <a href="https://support.apple.com/en-us/HT208334">11.2</a>   | 26                   | 38              | 68.4%      |
| 11.1.2                                                        | N/A                  | N/A             | N/A        |
| 11.1.1                                                        | N/A                  | N/A             | N/A        |
| <a href="https://support.apple.com/en-us/HT208222">11.1</a>   | 16                   | 24              | 66.7%      |
| 11.0.3                                                        | N/A                  | N/A             | N/A        |
| 11.0.2                                                        | N/A                  | N/A             | N/A        |
| 11.0.1                                                        | N/A                  | N/A             | N/A        |
| <a href="https://support.apple.com/en-us/HT208112">11.0</a>   | 65                   | 99              | 65.7%      |


Let's look at this aggregated for all of iOS 13 (with iOS 12 for a limited historical perspective).

| Total CVE Count | Memory Unsafety Bugs | Percentage | Release |
| 179             | 106                  | 59.2%      | iOS 13  |
| 261             | 173                  | 66.3%      | iOS 12  |
| 309             | 187                  | 60.5%      | iOS 11  |

To date, Apple has had substantially less bugs with a lower (although unlkely to be statistically significant) percentage of memory unsafety related issues. This, intriguingly, does not align with the general belief that iOS 13 is a buggier release, but security bugs and user-facing bugs (which are what create that sort of reputation) are not necessarily an overlapping set.

## macOS

Apple released macOS 10.15 (Catalina) on October 7, 2019 and subsequently has issued 6 point releases.

| Release                                                                             | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ----------------------------------------------------------------------------------- | -------------------- | --------------- | ---------- |
| <a href="https://support.apple.com/en-us/HT211289">10.15.6</a>                      | 11                   | 19              | 57.9%      |
| <a href="https://support.apple.com/en-us/HT211215">10.15.5 Supplemental Update</a>  | 1                    | 1               | 100%       |
| <a href="https://support.apple.com/en-us/HT211170">10.15.5</a>                      | 29                   | 48              | 60.4%      |
| <a href="https://support.apple.com/en-us/HT211100">10.15.4</a>                      | 17                   | 27              | 63.0%      |
| <a href="https://support.apple.com/en-us/HT210919">10.15.3</a>                      | 27                   | 32              | 84.4%      |
| <a href="https://support.apple.com/en-us/HT210788">10.15.2</a>                      | 41                   | 52              | 78.8%      |
| <a href="https://support.apple.com/en-us/HT210722">10.15.1</a>                      | 21                   | 33              | 63.6%      |
| <a href="https://support.apple.com/en-us/HT210634">10.15</a>                        | 7                    | 16              | 43.8%      |

One interesting data point in these numbers is that 10.15.2 had the unusual distinction of having 27/32 memory unsafety vulnerabilities be in tcpdump alone.

Let's run the macOS 10.13 (High Sierra) numbers as well so we have three generations of data available to us.

| Release                                                                            | Memory Unsafety Bugs | Total CVE Count | Percentage |
| ---------------------------------------------------------------------------------- | -------------------- | --------------- | ---------- |
| <a href="https://support.apple.com/en-us/HT208937">10.13.6</a>                     | 20                   | 33              | 60.6%      |
| <a href="https://support.apple.com/en-us/HT208849">10.13.5</a>                     | 20                   | 47              | 42.6%      |
| <a href="https://support.apple.com/en-us/HT208692">10.13.4</a>                     | 11                   | 44              | 25.0%      |
| <a href="https://support.apple.com/en-us/HT208535">10.13.3 Supplemental Update</a> | 1                    | 1               | 100%       |
| <a href="https://support.apple.com/en-us/HT208465">10.13.3</a>                     | 18                   | 27              | 66.7%      |
| <a href="https://support.apple.com/en-us/HT208397">10.13.2 Supplemental Update</a> | 2                    | 0               | 0%         |
| <a href="https://support.apple.com/en-us/HT208331">10.13.2</a>                     | 26                   | 39              | 66.7%      |
| <a href="https://support.apple.com/en-us/HT208221">10.13.1</a>                     | 104                  | 161             | 64.6%      |
| <a href="https://support.apple.com/en-us/HT208165">10.13 Supplemental Update</a>   | 2                    | 0               | 0%         |
| <a href="https://support.apple.com/en-us/HT208144">10.13</a>                       | 56                   | 102             | 54.9%      |

`Since we conducted the macOS 10.13 analysis years after the fact the numbers include more third party libraries. Apple has a tendency to update their security notes weeks or even months later to include the CVEs of projects (like Apache, tcpdump, et cetera) that were not necessarily public when they shipped the updated version. In macOS 10.13.1, for example, tcpdump had 60 (!!) memory unsafety vulnerabilities.

Despite these limitations, let's aggregate the data again.

| Release      | Memory Unsafety Bugs | Total CVE Count | Percentage |
| macOS 10.15  | 154                  | 228             | 67.5%      |
| macOS 10.14  | 213                  | 298             | 71.5%      |
| macOS 10.13  | 256                  | 458             | 55.9%      |


## XNU (macOS only)

The above statistics are for any code that ships with macOS or iOS (including the kernel), but what if we counted just kernel (and kernel extension) issues in macOS[^3]?


| Release     | Memory Unsafety Bugs | Total CVE Count | Percentage |
| macOS 10.15 | 58                   | 65              | 89.2%      |
| macOS 10.14 | 67                   | 76              | 88.2%      |
| macOS 10.13 | 65                   | 88              | 73.9%      |


[^1]: No blog post discussing another company's vulnerabilities would be complete without a discussion of the limitations.  Apple provides reasonably well-written descriptions of bugs and these allow us to generally bucket the vulnerabilities into our two categories of interest. However, occasionally a bug will defy easy categorization. For WebKit issues Apple does not distinguish between JIT compiler induced memory unsafety and C/C++ induced.  Apple also occasionally releases fixes that are still embargoed. When this happens they release their security notes, but update them at a later date with additional fixes. They denote these additional entries with an "Entry added on **date**" subscript, but we have typically long-since tweeted about it and won't catch the additional entries.  Since a significant amount of code is shared across Apple's operating systems (especially at the kernel level) many of these CVEs are duplicates across macOS and iOS. This means you cannot sum iOS and macOS bugs together to get a total across Apple's platforms without significantly miscounting.  For the above reasons you should not treat these numbers as **exact** but instead as reasonably accurate estimates of the relative percentage of memory unsafety issues Apple has seen in its products in this time range.
[^2]: See <a href="/2019/07/23/apple-memory-safety/">Last year's blog post</a> for iOS 12 and macOS 10.14 numbers.
[^3]: iOS and macOS use the same kernel, but since it's a different target the set of vulnerabilities may not entirely overlap. Counting iOS kernel vulnerabilities is left as an exercise for the reader.
