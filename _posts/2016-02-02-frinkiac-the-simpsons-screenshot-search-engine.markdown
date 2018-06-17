---
layout: post
title: Frinkiac - The Simpsons Screenshot Search Engine
---
**Update: Much has changed in the months since I originally wrote this post. Go ahead and read this but you can get newer information by reading about the [State of Frinkiac](/2016/05/16/state-of-frinkiac/).**

*Frinkiac. It's a website that's [simple yet ingenious and it fits right in the palm of your hand](https://frinkiac.com/?p=caption&q=simple+yet+ingenious+and+it+fits+right+in+the+palm+of+your+hand&e=S06E17&t=211493).*

<img src="https://frinkiac.com/meme/S04E09/609258.jpg?lines=+WELL%2C+JOHN+Q.+DRIVEWAY%0A+HAS+OUR+NUMBER%0A%0A%0A%0A%0A%0A%0ANOW+WE+PLAY+%0ATHE+WAITING+GAME" />

Aw, the [waiting game sucks](https://frinkiac.com/?p=caption&q=waiting+game+sucks&e=S04E09&t=614863). Let's play hungry hungry hippos!

Today I'm pleased to finally talk publicly about a side project [Sean Schulte](https://twitter.com/sirsean), [Allie Young](https://twitter.com/seriousallie), and [I](https://twitter.com/reaperhulk) have been working on: [Frinkiac](https://frinkiac.com).

Frinkiac is a search engine for Simpsons quotes. It contains nearly 3 million screenshots (every episode from season 1 through 15[^1]) indexed by the quote they are associated with and has a variety of features to help you find the exact screenshot you're looking for. Once you've found it you can share it with your friends or make a quick meme. Never again find yourself wishing you could [pinpoint the second his heart rips in half](https://frinkiac.com/?p=search&q=pinpoint+the+second+his+heart+rips+in+half). You'll feel like [god must feel when he's holding a gun](https://frinkiac.com/?p=search&q=like+god+must+feel+when+he's+holding+a+gun).

The Simpsons is one of greatest television comedies of all time and we hope that having ready access to the perfect screenshot will make people laugh and remind them to rewatch their favorite episodes.

Now, [let's all get drunk and play ping-pong](https://frinkiac.com/?p=caption&q=let's+all+get+drunk&e=S06E12&t=573038)!

### Some Technical Details

*[Ever heard of a little thing called the internet?](https://frinkiac.com/?p=caption&q=internet+eh&e=S09E14&t=303285)*

`frinkiac` parses episodes and generates screen captures. It attempts to determine what are relevant screens to capture in a fairly naïve way:

- Cut the scene into 100 equal-size buckets.
- Take the average color of each bucket.
- Compare that color to the corresponding bucket from the previously-saved image.
- If the total difference is large enough, save the image.
- Continue until all the frames of the episode have been inspected.

The code is written in Go and the video parsing is done with cgo+ffmpeg.

We also parse subtitle files and correlate each subtitle line's timecode with the timecode of the screenshot. Finally, the frinkiac binary can upload the data set to `frinkiac-server`.

`frinkiac-server` is also written in Go. It uses [gorilla/mux](https://github.com/gorilla/mux) as the router, React for the frontend, and indexes the quotes and screenshot metadata in PostgreSQL.

The index is constructed by taking the words from each subtitle and breaking them into prefixes (with a minimum length of two characters). So when you start searching you can get instantaneous results: typing "ill" is a prefix for "[illegal](https://frinkiac.com/?p=caption&q=ill&e=S05E02&t=392107)", "[illegitimate](https://frinkiac.com/?p=caption&q=ill&e=S04E04&t=978310)", and "[ill-gotten](https://frinkiac.com/?p=caption&q=ill&e=S03E14&t=969750)".

The more you type, the more accurate your search becomes. For example, "[kill anyone who looks](https://frinkiac.com/?p=search&q=kill+anyone+who+looks&e=S04E06&t=895227)" isn't accurate enough to find Rex Banner, so keep typing, ["kill anyone who looks at me cockeyed](https://frinkiac.com/?p=search&q=kill+anyone+who+looks+at+me+cockeyed&e=S04E06&t=895227)" to find the scene. Of course ... it turns out that was the only scene in which the Simpsons ever said "[cockeyed](https://frinkiac.com/?p=search&q=cockeyed&e=S04E06&t=895227)".


[^1]: With one small caveat. Season 11 has a significant time skew problem. We're aware of it! There are other seasons with small time skew as well, but nothing as major.
