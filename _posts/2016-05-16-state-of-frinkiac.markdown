---
layout: post
title: State of Frinkiac
---
<img src="https://frinkiac.com/gif/S07E07/649464/653218.gif?b64lines=QUxMIFRISVMgQ09NUFVURVIgSEFDS0lORyBJUwpNQUtJTkcgTUUgVEhJUlNUWS4=" />

In the 3 months since we launched [Frinkiac](https://frinkiac.com) we've been humbled and thrilled to see what people have done with it. However, we have always intended to continue improving it and this post is an attempt to clarify where we are and where we're going.

## New Seasons

As of the afternoon of May 15, 2016 we've enabled season 16 and 17 in Frinkiac. This brings our total to 378 episodes across the first 17 seasons. Seasons 16 and 17 add an additional 201,877 screenshots (and 21,444 subtitles) to the data set.

## GIFs

Generating high quality GIFs with somewhat reasonably limited file size costs a lot of CPU and a lot of RAM. The quantization (reducing the color palette to 256 without destroying image quality) and the calculations to determine what pixels should be set to transparent (or cropped out entirely) frame to frame require far more cycles than simply encoding it in h264. In our infrastructure generating a GIF costs 10-20x the CPU time for a file that's ~10x larger as well as lower quality. So why generate a GIF at all? Well, GIFs are the one animation format you can guarantee will work anywhere. They autoplay everywhere and can be uploaded to almost any service. However, many services now automatically convert GIFs into mp4/webm and show that to users for better efficiency. We've got some plans for doing that as well, but haven't finished that work yet (see MP4 below).

## GIF length limit

We started out limiting to 4 seconds to keep GIF size a bit lower and slightly limit CPU consumption, but today we'll be raising it to 7 seconds (new limit derived from my desire to see [Grandpa walk in circles at the burlesque house entrance](https://frinkiac.com/gif/S08E05/567616/574489/)).

## Loops

As we went down the path it also became clear that GIFs that appeared to loop perfectly were both desirable and difficult to create, so we wrote some code that compares the first frame and an empirically determined number of frames near the end of the selection to see if one of them will generate something that loops well. If it hits the detection threshold then the GIF is stopped there (even though you actually requested a few more frames) and hopefully a good loop is generated. We are still experimenting with variants of the loop detection threshold and number of examined frames, but in general this feature is working well.


## MP4 Support

We'll be adding mp4 support (once we work out some bugs). This will provide a significant optimization on both bandwidth and general user experience, although since the GIF fallback is potentially needed we'll still pay the CPU penalty for generating it. The biggest advantage will be seen with linking to a GIF page (not the bare image, but rather the page that contains it) since we can detect browser and deliver the optimized version. Additionally it will help with OpenGraph style embeds (e.g. Slack), which can pick up the optimized version with no user intervention.

## The Future

We've got many grandiose ideas, but here are a few we're actually working on:

* Font and font size control for memes and GIFs.
* UI redesign/improvements
* Showcase/Gallery
* Lots of behind the scenes work to optimize and improve the codebase.
* Additional languages (this is challenging due to subtitle synchronization issues)
