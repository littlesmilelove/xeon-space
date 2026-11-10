---
title: "Zero-Upload GIF Tools: OffscreenCanvas and $0 CPV"
description: "How BulkMerge moved GIF resizing, compression, WebP conversion, and image merging into the browser with OffscreenCanvas and LZW, cutting CPV to $0."
publishDate: "2026-11-10"
tags: ["browser", "canvas", "gif", "web-performance", "architecture"]
---

The first architecture diagram for BulkMerge looked like every other image tool: upload, queue, worker, object storage, download. It is a familiar shape because it is easy to draw. It is also a business model hiding inside a technical decision. Every uploaded file needs a bucket, a lifecycle rule, a cleanup job, an egress bill, and a rate limiter to stop someone from turning the service into a free transcoding API.

We removed that entire branch. The tools now decode, resize, quantize, and re-encode inside the tab. The server serves the page; the image path never touches it. The number we watch is CPV, cost per visit. For the image-processing path, it is $0.

That does not mean hosting is free. The domain, the edge deployment, the fonts, and the engineering time are real. The narrower claim is the useful one: there is no per-visit image-processing bill. The visitor's CPU does the work, and the file never leaves the device.

## The pipeline in one sentence

A file becomes an `ImageBitmap` or a set of RGBA frames, the frames are drawn through Canvas 2D or `OffscreenCanvas`, and the result is encoded back to a `Blob` that the browser downloads.

The interesting parts are between those verbs.

For a GIF, the browser cannot simply call `createImageBitmap` and get an animation. A GIF is a logical screen, a palette, and a sequence of image descriptors. Each descriptor can contain only the rectangle that changed, a local palette, a delay, and a disposal method. `gifuct-js` gives us those pieces. We composite each patch onto a full RGBA frame, keep the pixels underneath transparent patches, and clear the canvas after a frame whose disposal type is "restore to background."

Two bugs in that loop are worth recording because both were invisible in code review and obvious in a browser:

1. **The ten-times-slow bug.** `gifuct-js` already converts GIF delays from hundredths of a second to milliseconds. We multiplied by ten again. Every animation played at one-tenth speed. A test that compared total running time, not just frame count, caught it.
2. **The 128×96 emoji bug.** The Discord preset asked for 128×128, but the animation path scaled by the smaller ratio and never padded the result. A 4:3 GIF came out 128×96. The still-image path already had a `contain` mode; the animated path did not. We added a fit-mode-aware frame resize and the preset finally produced the square it promised.

The second bug is a good argument for screenshots as tests. The code passed type checking and every unit test. The UI was wrong in a way that only a rendered result could show.

## OffscreenCanvas is the surface, not the whole story

`OffscreenCanvas` is often described as "canvas in a worker." That is one way to use it, but it is not the only reason to use it. Our first win is simpler: the hot path no longer allocates a DOM element for every frame.

The pipeline already holds RGBA buffers, draws them into a canvas, reads them back, and encodes. A DOM canvas is not needed for any of that. `OffscreenCanvas` gives us the same 2D context and `convertToBlob`, without attaching anything to the document. Where the API is missing, the code falls back to `document.createElement('canvas')` and `toBlob`. The rest of the pipeline does not care.

That is a deliberately modest claim. We are not pretending the current loop runs in parallel. The frame work is still synchronous and still on the main thread. What we bought is a surface that can move to a worker without rewriting the image code, plus one less DOM allocation per frame. A future version can put the same functions behind a worker boundary and keep the UI responsive on large animations.

The current loop processes one file at a time and yields between files. That keeps the page alive without pretending a single tab is a render farm.

## LZW, palettes, and the 256-colour ceiling

GIF encoding is two jobs: reduce the image to an indexed palette, then compress the index stream with LZW. `gifenc` does both in a few kilobytes. We chose it because the alternative, a full image library compiled to WebAssembly, costs more bytes than the feature is worth.

The encoder choices that matter:

- **One palette per file.** We build it from the middle frame. Per-frame palettes can flicker when the same colour is quantized differently from one frame to the next. Middle-frame colour is not perfect, but it is stable.
- **`rgb444` quantization.** It is cheaper than `rgb565` and good enough for screenshots, flat artwork, and emoji. The UI exposes 256, 128, 64, and 32 colours. 256 is the format ceiling, not a target.
- **Delays are carried, not regenerated.** A frame keeps its own delay. When we drop frames, the survivor inherits the sum of the delays it replaces, so the clip keeps its running time. Dropping every second frame halves the sample count without speeding up the animation.

The compression loop is a small search. It applies the requested frame drop and scale, encodes, and compares the result to the target size. If the file is still too large, it halves the palette down to 32 colours. After that it scales the animation down and tries again. It gives up after a few attempts with a message that says which lever to pull, rather than returning a file that is still over the limit.

The three levers are ordered by what they cost visually. Frames are usually the cheapest to drop if the delays are summed correctly. Colours are next, especially on flat artwork. Pixels are last because a smaller canvas changes the composition, not just the encoding.

## Merging images is a layout problem

The [Merge Images Online](https://bulkmerge.com/) tool has the least exotic code and the most edge cases. `createImageBitmap` decodes each file. The layout code then picks a common dimension:

- Vertical: scale every image to the widest image's width, then stack.
- Horizontal: scale every image to the tallest image's height, then place them side by side.
- Grid: fit every image into a common cell, then arrange rows and columns.

The arithmetic must be literal. A vertical stack's height is the sum of the scaled heights plus gaps. A horizontal row's height is the tallest image, not the sum of all rows. A grid is the only layout that uses equal cells.

We shipped the wrong version first. A side-by-side merge of three 640×420 images produced a 1936×1276 canvas instead of 1936×420, with a band of empty space above and below the row. The code reused the grid height formula for every non-vertical layout. The screenshot showed it immediately: three small tiles floating in a tall white rectangle. The fix was two branches, but the lesson was that layout code needs a rendered assertion, not just a unit test on numbers.

JPEG export adds one more detail. JPEG has no alpha channel, so we fill the merge canvas with white before drawing. Without that, transparent areas become black.

## What CPV = $0 actually buys

The server-side version of these tools has a cost per file that scales with usage: CPU for decode and encode, memory for frames, storage for the upload and the result, egress for the download, and the operational surface for deleting all of it.

The browser-side version has a cost per visit that is effectively the cost of serving the page. The image work is paid for by the device that already has the file. There is no queue to absorb a spike, because there is no queue. There is no bucket to clean, because nothing was stored. There is no upload endpoint to abuse, because there is no endpoint.

This changes the product, not just the bill. We can offer the tools without an account, because there is no per-use cost to recover. We can show the result size before download, because the result already exists in the tab. We can process a folder of GIFs in one pass and build the ZIP locally, because the bytes are already there.

The trade-offs are equally real:

- GIF is limited to 256 colours and one-bit transparency. Soft shadows and gradients will band.
- A browser tab is bounded by device memory. A hundred full-resolution frames will struggle on a low-memory laptop.
- Animated WebP decoding depends on `ImageDecoder`. Chrome, Edge, and Safari have it; Firefox does not yet, and the page says so.
- Re-encoding an already optimized GIF can make it slightly larger, because the palette is rebuilt.

None of those limits matter for the jobs these tools target: a screenshot merge, a 128×128 emoji, a WebP that needs to become a GIF, and an animation that has to fit under a chat app's upload cap. For everything else, a desktop encoder is still the right tool.

## The four tools

The whole point of the architecture is that these are ordinary pages with no image API behind them:

- [Merge Images Online](https://bulkmerge.com/) for stacking, side-by-side, and grid layouts.
- [GIF Resizer](https://bulkmerge.com/tools/gif-resizer) for scaling every frame without losing the timing.
- [WebP to GIF Converter](https://bulkmerge.com/tools/webp-to-gif) for animated and still WebP input.
- [Compress GIF](https://bulkmerge.com/tools/compress-gif) for fitting an animation under a size limit.

The next step is not a bigger server. It is moving the same frame loop into a worker and seeing how much of the main-thread work disappears. `OffscreenCanvas` is already the surface. The rest is plumbing.
