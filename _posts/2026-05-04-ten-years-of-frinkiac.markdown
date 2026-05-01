---
layout: post
title: Ten Years of Frinkiac
---

# Ten Years of Frinkiac: A Few Months of Modernization with AI

It's been just over ten years (February 2, 2016) since we launched [Frinkiac](https://frinkiac.com) to the public. In those first few months after release we added GIF support, some new sites (hello [Morbotron](https://morbotron.com)), and then we mostly stopped working on it. Everything worked well enough for us and the code sat unchanged on the same architecture we'd built in 2015.

In February 2026 it occurred to us that maybe we should point those AI coding tools we've been using at the Frinkiac codebase. The result of that work is live now and you can go see it on both [Frinkiac](https://frinkiac.com) and [Morbotron](https://morbotron.com), but we've also documented a subset of the experience in this post.

## Modernizing the Stack

Before we worked on new features we had to drag the project forward a decade. The Go backend was on 1.8 and used a vendored `GOPATH` tree. We moved it to the latest Go release and converted everything to Go modules, which let us actually see/prune the dependency graph for the first time in years.

The frontend also needed attention: Webpack 4, an out of date Babel config, Google Analytics, and some social media SDKs for sharing. We migrated webpack 4 → 5 → Vite 8, upgraded to React 18, and replaced the `superagent` HTTP library with native `fetch`. All third party JS gone. The share experience was rebuilt around copy-to-clipboard, copy-URL, and the Web Share API for mobile.

With that work done we could now look at actual improvements.

# Site Improvements

## What Users See

### New Search System

Frinkiac's original search was a custom prefix-index system that did basic substring matching against subtitle text. It worked well enough for simple queries, but it had no concept of relevance ranking, couldn't handle multi-word queries gracefully, and returned a lot of near-duplicate results from the same scene.

We've now rebuilt search on PostgreSQL's full-text search engine — tsvector/tsquery with proper linguistic stemming, a GIN index for fast lookups, and a relevance ranking system. The new search uses a two-tier matching strategy: we OR all the search terms together for recall (so "steamed hams" finds subtitles containing either word), but rank results using an AND query so subtitles matching *all* terms float to the top. There's also a `window_content` column that concatenates neighboring subtitles, so queries that span a sentence break across two subtitle lines still match.

On top of the core FTS engine, we built a deduplication layer. Without it, searching for a common phrase returns a wall of nearly identical screenshots from the same few seconds of the same scene. The dedup system groups results into proximity windows (frames within 10 seconds of each other in the same episode) and caps how many results can come from each window. Strong matches (where the subtitle itself contains all query terms) get more generous limits than weak matches (where the terms only appear in the neighboring subtitle window). We went through several rounds of tuning the constants — window size, max frames per scene, max frames per subtitle — before the results felt right.

We also added season filtering via a compact popover with a grid of season pills — click one to select it, click a second to select a range, or drag across multiple seasons. There are preset buttons (like "Classic" for seasons 3–9 on Frinkiac), and you can save up to two favorite season ranges as your own presets. Search result thumbnails now show subtitle text overlays so you can see the quote without clicking through, plus episode title badges on each result.

### New Screenshots and Subtitles

The original Frinkiac dataset had some data quality issues that had bugged us for years. Subtitle timing was slightly offset from when characters actually said the lines, which meant search results would sometimes land on a frame a second or two away from the moment you wanted. We re-did the subtitle extraction from scratch to fix the timing alignment.

While we were at it, we re-extracted all the screenshots at higher resolution. The original frames were 640x480; the new ones are sharper and hold up better when used in GIFs and comics, especially on high-DPI displays. This isn't a perfect solution as the original SD presentation avoids some upscaling artifacts (especially over-sharpening), but overall we think it's an improvement.

Oh, and we put up every episode now that they can be easily filtered. This gives us a current count of 4,990,964 frames across 802 episodes and the movie.

### A Real GIF Editor

The new GIF maker is built around an NLE-style timeline with a filmstrip showing actual frame thumbnails, yellow trim handles for setting the clip range, and a draggable playhead for scrubbing. Above the filmstrip sit overlay track bars — one per text overlay — that you can drag and resize to control exactly when each piece of text appears and disappears during the clip. The whole thing renders live on a `<canvas>` element: fonts are loaded via the FontFace API, text is drawn with the same sizing and outline logic as the server-side Go renderer, and every edit updates the preview instantly with no server round-trips.

There's live video preview too. The base clip (without overlays) renders once as an MP4, and a requestAnimationFrame loop composites each frame with the active overlay text drawn on top. You can hit Space to play/pause and watch your GIF with text timing before you commit to rendering. The render itself streams NDJSON progress events back to the client, so you get a real progress bar instead of a spinner.

Text overlays are directly draggable on the canvas with hit-test divs. Each overlay has its own properties panel with font selection, size slider, color picker, and alignment controls. Subtitles from the caption page are automatically prepopulated as overlay tracks with sensible timing. We added smart loop detection that analyzes frame similarity to find natural loop points for seamless GIFs, and undo/redo with intelligent coalescing — dragging a slider doesn't create 47 undo entries, but releasing it does create one. The toolbar itself went through several redesign iterations to get pan arrows on the filmstrip, a centered play button, and controls that wrap properly on mobile instead of overlapping.

The GIF maker now redirects to a dedicated shareable result page with proper OG meta tags — og:image, og:video, and Twitter card tags — so links shared in iMessage, Slack, and other apps show an inline preview again. Getting iMessage previews to work correctly required some specific workarounds: iMessage caches og:image at the domain level, so we had to add per-page cache busters, and it won't linkify URLs with very long path segments, so we moved overlay data from the URL path into query parameters.

### Comic Maker

The old meme maker let you write text on a single image. We replaced it with a multi-panel comic strip builder.

You start by picking frames from any episode and arranging them as panels. Clicking "Make Comic" from a caption with multiple subtitle lines now creates a multi-panel comic by default — one panel per subtitle, each using that line's representative frame so you get a ready-made comic strip in one click. The comic maker supports multiple layouts — horizontal strips, vertical strips, grid arrangements like 1-over-2 for three panels, and a "single" layout that collapses everything back into one panel with stacked text. Switching to single is undoable, so you can freely experiment. Panels are re-orderable with a drag and each panel has its own filmstrip scrubber so you can dial in exactly the right frame.

Each subtitle gets its own overlay track, and overlays are directly draggable on the canvas with contrast-aware drag handles that switch between light and dark depending on what's behind them. You can add multiple text overlays per panel with independent font, size, color, and alignment settings. The canvas preview renders at native resolution with text sizing that matches the server-side Go renderer pixel-for-pixel — we went through several rounds of getting the bounding boxes, outline widths, and line heights to match exactly.

There's also multi-select support for bulk operations on overlays (Shift-click on desktop, long-press on mobile), undo/redo, and the entire editor state is serialized into the URL so you can share a work-in-progress or refresh without losing anything.

Both the GIF maker and comic maker share extracted utility code for undo/redo, text dragging, and common UI components — as the two editors evolved in parallel, we periodically pulled shared logic into common modules to prevent drift.

### Browsing and Navigation

The old episode view was a clunky paginated table. We replaced it with a full-episode storyboard grid that loads every frame at once. Each card shows the thumbnail with a timestamp and subtitle overlay, images lazy-load as you scroll, and clicking a card selects it (highlighting it in the grid and updating the URL) while a second click takes you to the caption view. There's also a floating search box that filters the storyboard by subtitle text, so you can quickly find a specific moment within an episode.

We also added a navigator — a new top-level page for browsing by season and episode. Each episode tile shows a rotating selection of frames with fade transitions, and hovering previews additional frames. Much like random, it's intended to help you find something interesting when your mind is blank.

On the caption page itself, the old grid of nearby thumbnails was replaced with a horizontal filmstrip scrubber. Click to jump or drag to scrub. During a drag, the filmstrip holds its frame list steady while the main image, URL, and subtitles update in real-time. Dragging near either edge of the filmstrip auto-pans to load adjacent frames, so you can scrub continuously through an episode without lifting your finger. And episode titles now link to their Wikipedia pages for quick reference.

Finally, the static subtitle display on caption pages was replaced with a scrollable transcript showing ±30 seconds of surrounding dialogue. The current line is highlighted and the view auto-scrolls to keep it centered. Clicking any line jumps to that frame. When the current timestamp falls in a gap between subtitle lines, a blue divider marks the position. The transcript follows the filmstrip during drag, so scrubbing through the filmstrip also scrubs through the dialogue — you can see exactly where you are in the conversation at all times. On mobile, the transcript compacts to about three visible lines to save vertical space while keeping the current line centered.

### Explore Similar

We added a new "Explore Similar" link on the caption page. When clicked, it checks the current frame's CLIP embedding against the HNSW index and returns visually similar frames from across the entire series, deduplicated within 10-second proximity windows so you don't get a wall of sequential frames from the same shot.

### Search History

We added client-side search history backed by localStorage. When you press Enter or click a search result, the term is saved. Focus the search box and your recent searches appear as a dropdown — click one to re-run it instantly. Only intentional actions (pressing Enter or clicking through to a result) save to history, not debounce-triggered searches, so the list stays clean and useful.

### Small Stuff

We rounded things out with a set of small features that address the biggest friction points for daily users — all client-side, no accounts required.

First, we extracted the one-off copy-confirmation toast from the share buttons into a reusable notification system. A single `<Toast />` component lives at the root of the app, and any feature can call `showToast("message")` to trigger it. This gave us a consistent feedback pattern for every subsequent feature.

**Favorites** lets you bookmark frames. A heart icon on every caption page toggles the frame in and out of a localStorage-backed favorites list (up to 50 entries). There's a dedicated `/favorites` page that shows your saved frames in the same grid layout as search results. Favorited frames also get a subtle heart indicator when they appear in search results, so you can spot your bookmarks at a glance.

**Recent Creations** solves the "I just spent five minutes making a GIF and then closed the tab" problem. Every time you generate a GIF or comic, the URL is saved to localStorage (up to 50 entries). A dedicated `/creations` page shows thumbnail cards with type badges (GIF/Comic) and relative timestamps, so you can get back to your work with one click.

**Embed Codes** adds a `< >` button to the share row on every caption page. Click it and a popover offers ready-to-paste snippets in four formats — HTML, Markdown, BBCode, and a direct image URL — with one-click copy for each. The popover dismisses on Escape or click-outside, matching the existing UI patterns.

All four features use the same versioned-envelope localStorage pattern as search history, with migration support so the storage format can evolve without breaking existing users' data.

### Keyboard Shortcuts

We added them:

- `/` focuses the search bar from any page
- Arrow keys navigate search results and scrub through frames on the caption page
- `Enter` triggers an instant search, bypassing the debounce
- `Escape` goes back
- In the GIF maker: `Space` for play/pause, arrow keys for frame-by-frame navigation
- In the comic maker: all the same, plus shortcuts for multi-select operations
- A `?` icon in the header opens a keyboard shortcuts popover so you can actually discover all of this

## Under the Hood

The database layer got a major overhaul: we migrated from gorp + lib/pq to pgx v5 with native pgvector support, dropping 7 transitive dependencies (including drivers for MySQL, SQLite, and Cassandra that gorp pulled in but we never used). We also split the single connection pool into separate read and search pools so expensive full-text and pgvector queries can't starve cheap caption/nearby/transcript lookups of connections.

We also packed all frame images into flat `.pack` files — one per episode — with an embedded interpolation-searchable index, served via mmap for zero-copy reads. This eliminated millions of individual files on disk and made frame serving faster by avoiding filesystem metadata overhead. The pack format itself went through a v2 redesign: a 24-byte header with magic number, version, and size group boundaries replaced per-entry size codes, shrinking index entries from 32 to 16 bytes.

For the S3 object cache (where generated GIFs and MP4s are stored), we added access tracking via a buffered channel that flushes to PostgreSQL in batches, and an hourly scrubber that evicts objects not accessed within 90 days. This keeps storage from growing unbounded without affecting request latency. The fire-and-forget upload goroutines were replaced with a bounded worker pool — 4 workers draining a 64-slot channel — so an S3 outage can't cause unbounded goroutine and memory growth. If the queue fills, uploads are dropped (the content remains in groupcache) and a Prometheus counter tracks the drops for alerting.

The original schema had accumulated redundant indices as well as some missing ones. We audited the full set, dropped single-column indices that were subsumed by composite or unique indices, and added new ones where queries needed them.

So, so much more. Eliminating cgo in favor of subprocesses with pipes, optimizing GIF/MP4 generation, moving to native Go HTTP routing, the list goes on.

## Hosting

We've hosted Frinkiac on Digital Ocean since 2018, but with this upgrade we're transitioning to a more radical setup. We now run the entire stack in-house (literally, in a home) and connect to our CDN via cloudflared. This makes things more fun, but less highly available on a few axes. Will the negatives outweigh the positives? No idea, but we'll have fun finding out.

## The Future

Visual search! We've been working on this for a while, but we're not happy with the results yet so it's feature flagged off. The biggest challenge is that the visual models which describe scenes are not good at identifying characters. We need to improve character recognition and integrate it with scene descriptions to provide acceptable visual search results and that remains a work in progress. For example, it is not enough to know that Homer, Bart, and Moe are in a scene, we need the textual description to correctly identify which characters are performing which actions. We make no promises on dates (we did neglect this site for nearly ten years after all), but keep an eye out.

We're also sure the community will come up with new ideas we want to try, and we're excited to see what happens.

## Working with AI

Almost every one of the commits was co-authored with an AI coding tool. For a codebase that had been sitting largely untouched for years, having an AI collaborator that could quickly understand the existing architecture and propose changes across the full stack was incredibly useful. We had forgotten more than we remembered about the Go backend, React frontend, SQL migrations, Docker configuration, shell scripts, etc. And as we added newer features it was invaluable to rapidly iterate with Python ML pipelines and rapidly generate new data sets with minimal effort.

The cgo-to-subprocess migration is a good example. That's the kind of change we'd been vaguely meaning to do for years but never got around to because it touched so many parts of the system. With AI, it went from "we should really do this someday" to "done in an afternoon."

Despite the assistance, we still poured an immense number of hours into the modernization and enhancements. Rather than taking less time, our ambitions grew.

We think we're just getting started, but maybe we'll be content with this for another ten years. I guess we'll see. Happy 10th birthday Frinkiac. Here's to many more.
