---
layout: post
title: Apple Censoring the News+
---

Yesterday Apple announced News+, "<a href="https://www.apple.com/newsroom/2019/03/apple-launches-apple-news-plus-an-immersive-magazine-and-news-reading-experience/">an immersive magazine and news reading experience</a>". This immersive experience is, for now, a US/Canada only product, with Apple noting: "Available in the US and Canada, Apple News+ presents the best and most relevant articles to meet any range of interests from renowned publications such as Vogue, National Geographic Magazine, People, ELLE, The Wall Street Journal and Los Angeles Times."

Starting with the US is understandable for a variety of reasons, but in the modern world people travel between countries. So, now that Apple accepts money to act as the distributor of news content, what users is it available to?

### A Brief Digression About Feature Gating

Apple sells its products in many jurisdictions and those products have myriad features. Sometimes those features need local regulatory approval (like heart rate variability or ECG/EKG on Apple Watch) and on occasion countries decide specific features are unacceptable (like FaceTime in the UAE, FaceTime Audio (but not video!) in China, etc). To enforce these restrictions Apple uses hardware identifiers in phones sold within these regions. If the phone comes from that region then the prohibited features will be disabled *even if the operating system is set to a region where the feature is available*. In my experience this is a surprising fact to many people. So, an Apple Watch Series 4 sold in France will not allow ECG/EKG readings even if you set the region to US, an iPhone sold in China will not support FaceTime Audio with any region setting, and an iPhone sold in the UAE will not support FaceTime Video with any region setting.

Conversely, a Series 4 watch sold in the US will continue to support ECG features even when physically taken to another country and a US-based iPhone is perfectly capable of using FaceTime Audio (without a VPN) inside mainland China.


### China and the News

Apple only shows the News app on phones that are within the appropriate region. However, this region enforcement is not accomplished via the hardware feature gating described above. Rather, it is a pure software region block that can be bypassed by setting your operating system’s region setting to a supported one. Since the launch of News in iOS 9 the set of supported regions has slowly expanded to the following: US, Canada, UK, and Australia. So to see the News app you can change your region and once you have it downloaded traveling to other countries will not disable News and you'll be able to use all of its features...with one exception.

News content is a sensitive topic in China. The government exercises a significant degree of control over information sources so it is unsurprising that Apple would choose to not support News there. However, instead of simply being locked behind a hardware feature gate, Apple chose to disable it much more forcefully. If you enter China with a US iPhone (e.g. one purchased in the US from a US carrier or at a US Apple Store), using a US carrier, with your phone set to the US region, and with location services disabled for the News app, you will still receive this message upon opening News:


<div style="margin:0 auto"><img src="/assets/media/2019/feed-unavailable.png" style="width:281px"></div>

To accomplish this censorship Apple is using a form of location fingerprinting that is not available to normal applications on iOS. It works like this: despite the fact that your phone uses a SIM from a US carrier it must connect to a Chinese cellular network. Apple is using private APIs to identify that you are in mainland China based on the name of the underlying cellular network and blocking access to the News app. This information is not available via public APIs in iOS[1. There is an API for querying the cellular network, but it will only return your carrier, not the network you may be roaming on] specifically to improve privacy for users.

This censorship occurs despite the fact that when in China a cell phone using a foreign SIM is not subject to the firewall restrictions (all traffic is tunneled back to your provider first), so Google, Twitter, Facebook, et al all work fine on a non-mainland China SIM even though you're connected via China Mobile or China Unicom's network.


All of the above has been true since 2015 when News launched and it was even covered at the time by a <a href="https://www.nytimes.com/2015/10/12/technology/apple-is-said-to-deactivate-its-news-app-in-china.html">variety</a> <a href="https://qz.com/521871/apple-is-blocking-its-news-app-from-everyone-in-china-even-if-their-phone-is-registered-in-the-us/">of</a> <a href="http://money.cnn.com/2015/10/12/technology/apple-news-blocked-china/">outlets</a>. What makes this even uglier now is that Apple has chosen to start **selling news subscriptions while simultaneously censoring access for their customers on a low-level regional basis**. A US citizen in China can (without resorting to an "illegal" VPN) load webpages like the Washington Post[2. Incidentally, WaPo isn't blocked by the firewall at all], New York Times, or WSJ on their (non-Chinese SIM card) phone but they can't access Apple News. I find it profoundly worrying that Apple is seemingly willing to use technology only available to the platform holder to censor far above and beyond typical feature gating when it comes to news.
