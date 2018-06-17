---
layout: post
title: Subtitles and Internationalizing Frinkiac
---

Recently I've been spending some time getting various pieces of [Frinkiac](https://frinkiac.com) prepped for supporting more languages. That work is essentially done now (maybe I'll blog about accent stripping, unicode letter normalization, et cetera at another date), but we've run into a major problem. To explain what the issue is it will be helpful to talk about the idiosyncrasies of subtitles.


## What Exactly Is A Subtitle?

First, what a subtitle is not guaranteed to be: A direct and exact copy of what you see and hear on screen.

For English television shows English subtitles are provided for use by the hard of hearing. Because of this, they typically do not contain copies of the text that appears on screen.

<div style="margin:0 auto;width:330px">
  <img src="https://frinkiac.com/img/S10E01/38654/medium.jpg">
  <div style="font-size:smaller">You'll have to read this yourself. It isn't in the English subtitles</div>
</div>
---
This drawback is why Frinkiac can't give you the above screenshot without searching for what Homer says out loud[^1]. Similarly, searches for [milpool](https://frinkiac.com/caption/S06E01/445795)[^2] or [old man yells at cloud](https://frinkiac.com/caption/S13E13/354062)[^3] are fruitless if you can't remember what was being said around the time it appeared on screen. Fortunately, you can always find [dignity](https://frinkiac.com/caption/S08E06/357706) thanks to the amazing art skills of a Gudger College graduate.

This behavior is logical, but can be surprising to Frinkiac users who don't understand the way the quote corpus was constructed. However, there are more surprising subtitle behaviors.

Subtitles are frequently constructed by using the original script as a source. This can cause errors in the resulting captions where the actors ad-libbed or the director/writer/whoever made changes that were not propagated back into an updated version of the script.

Typos and missing spaces are also not uncommon. [The latest issue of successmagazine](https://frinkiac.com/caption/S10E01/114563) probably would be more successful if it had a space between those two words. On occasion the subtitles will be wrong with a similar phonetic sound as in ["I don't know your songs losers"](https://frinkiac.com/caption/S14E21/437270). In the episode this is clearly "I own all your songs, losers" in a high-pitched voice and is a reference to Michael Jackson owning a significant chunk of the Beatles' back catalogue, not an admission of ignorance in falsetto. And, of course, sometimes shows just make up words so spelling becomes pure guesswork ([I was saying "buu-urns"](https://frinkiac.com/caption/S06E18/1054102)).

Finally, subtitles are tied to time codes that reference when they should start and stop showing on the screen. This keeps them synchronized with the video you're watching...but of course there's a catch. Recordings from TV may be syndicated (in which case various small scenes are cut to provide additional time for advertisements), but also each person who records the episode will cut out commercials slightly differently, resulting in disparate times between each act. This results in sync problems if your subtitles aren't precisely the same videos that the creator originally used. Even if a DVD source is used differences may be found across regions, as different video standards result in slightly different framerates (and playback speed!).

## Internationalization

So what does this have to do with the internationalization of Frinkiac?

During the localization process each Simpsons episode is (ideally) translated to use alternate language idioms and tweak jokes for the audience. It is not a direct transliteration. This means that each translated subtitle is **not** guaranteed to appear at the same time as the English one did. In fact, thinking of each subtitle as a separate sentence to translate is incorrect. This is great for making The Simpsons more relatable to each audience, but as we are not native speakers we have a great deal of trouble determining the timing. We have no reference without a native audio track that is itself synced to the timings we expect for the video. Within a few seconds is the best we can do and even that takes a great deal of time and conjecture. To add insult to injury the subtitles we have located tend to not even vaguely match the time codes we need (presumably they were created to be used with various television captures).

Accuracy is likewise problematic. Without language fluency and the audio tracks we can't judge the quality of a given subtitle. Experience has taught us that many transcriptions found online (even in English) are woefully inaccurate and significant errors are catastrophic for a quote search engine. Imagine if you searched for ["do you want the job done right"](https://frinkiac.com/caption/S12E03/128837) and nothing came up because the subtitles thought the quote was "do you need to do it well"[^4].

One other small difference: the screenshot shown above may also have a caption where it did not in English (in the case of Spanish it reads `ROJO = FUSIÓN`). This is ostensibly great for searching, but causes a new problem with memes since we populate meme text by default with the captions...and `ROJO = FUSIÓN` was never spoken aloud.

To deliver on the idea of a multilingual Frinkiac we need to do better.

## The Way Forward

Some of what I've described above can be fixed with the addition of crowdsourcing tools to let users provide fixed captions and correct time codes that may be skewed. This may or may not come in the future[^5]. But ultimately that only helps for English, where we've already got a reasonably accurate database.

For foreign languages we're going to need Simpsons fanatics who know The Simpsons in their native language (along with enough English proficiency to take direction from us, sorry!). These volunteers will be helping us by finding/building **accurate** SRT files for their chosen language with time codes matched to the US DVD releases (which we consider the canonical time codes).

This is not an easy task, but we believe it's worthwhile. The Simpsons is a global cultural touchstone and it feels wrong not to make Frinkiac accessible to fans of all stripes. Come help us! If you believe you can help out you can DM [@reaperhulk](https://twitter.com/reaperhulk) or [@sirsean](https://twitter.com/sirsean) and we'll let you know what we need!

[^1]: ["Can't you just write on your arm like I do?"](https://frinkiac.com/?q=can't%20you%20just%20write%20on%20your%20arm%20like%20i%20do) for the record.

[^2]: Search for ["you're wearing your glasses"](https://frinkiac.com/?q=you're%20wearing%20your%20glasses) and then browse forward a few frames.

[^3]: ["Can't you just use this recent photo"](https://frinkiac.com/?q=can't%20you%20just%20use%20this%20recent%20photo)

[^4]: This is not an exaggeration. Many English subtitles are at best paraphrases of the actual show and at worst cruel mockeries.

[^5]: Crowdsourcing comes with attendant abuse issues and handling that is a non-trivial problem.
