---
layout: post
title: Morbotron -- Like Frinkiac, But For Futurama
---

[Good news!](https://morbotron.com/?q=good%20news)

After launching [Frinkiac](https://frinkiac.com) one of the most common requests we've received is to build something "like Frinkiac, but for &lt;show&gt;". Our response has been that we do love a few other TV shows and we **did** design it to be mostly abstracted, but we don't have anything to share yet. Today I am pleased to have something to share: [Morbotron](https://morbotron.com).

Morbotron is identical to Frinkiac, but for Futurama. This means all the same search, meme, and GIFing capability is available but with the entire library of Futurama episodes at your fingertips. You can check out the [Frinkiac announcement](/2016/02/02/frinkiac-the-simpsons-screenshot-search-engine/) post for some technical details on how it was all created, but here are some Futurama specific statistics:

* 861,414 unique frames (109 GB)
* 124 episodes
* 4 movies
* 63,527 subtitles

Marvel at the [reverse scuba suit](https://morbotron.com/gif/S01E08/223005/231730/) or complain when it's too hot and the [butter in my pocket](https://morbotron.com/?q=butter%20in%20my%20pocket) is melting. Or maybe you [got no pants](  https://morbotron.com/?q=got%20no%20pants) on and want your friends to know?

Astute Frinkiac observers (aka my Twitter followers) may have noticed that we added the Simpsons Movie to Frinkiac a few weeks ago. This was done (in addition to being a funny movie) to verify that all the code we had added to handle alternate aspect ratios was behaving as intended. And since later seasons of Futurama are available in 1080p we also wanted to be sure we could provide that quality to our users.

Other than the coat of paint and the changes to handle widescreen/high definition images there isn't much in the way of user visible work (YET!). Behind the scenes, however, we've finished refactoring the frinkiac server into a generic platform we call `cghmc`[^1]. New designs and features will be appearing as we find the free time to work on them, and we might have a show or two more waiting in the wings...

If you're in the entertainment industry and have some ideas about the project or what shows need a site like this [email us](mailto:frinkiac.feedback@gmail.com) or [tweet at me](https://twitter.com/reaperhulk) and let's talk!

Now [where's that blue box with our universe in it?](https://morbotron.com/?q=Where's%20that%20blue%20box%20with%20our%20universe%20in%20it)

[^1]: What that stands for is left as an exercise for the discerning reader.
