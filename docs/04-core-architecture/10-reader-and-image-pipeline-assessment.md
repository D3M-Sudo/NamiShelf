# NamiShelf — Reader and Image Pipeline Assessment

**Status:** Static architecture assessment; no rendering benchmarks or runtime tests  
**Reviewed candidate:** `AltairCardinal/mihon-desktop@701080c251c8fd8b35d0b57a481d93b9a9eb3701`  
**Date:** 2026-11-03

## 1. Executive assessment

The candidate has a desktop-specific reader implementation rather than merely reusing Android UI. The inspected code uses Compose Desktop image types and Skia native decoding, with separate page decode, region/tile decode, presentation ownership, and memory-accounting components.

This is a substantial head start, but it increases the importance of native runtime packaging and memory validation on Linux. Skia/Skiko binaries, image codecs, GPU/CPU fallback, and memory ownership must be validated on each target architecture.

## 2. Observed pipeline

The inspected `SkiaImageDecoder` uses Skia codecs and Compose image bitmaps. It documents JPEG, PNG, WebP, GIF and other formats supported by the packaged Skia build. It includes dimension probing, pixel decode checks, downsampling, orientation handling, and decode paths that reserve memory before simultaneously live rasters are allocated.

Other inspected components include desktop page decoders, page-image pipeline, region decoder, presentation image owner, and tests/contracts for shared image-memory authority and region decoding.

The repository also contains reader architecture documentation describing stable page identities, presentation slots, paired-page layout, visible-page progress, and renderer behavior. These design details should be retained as evidence, not assumed to be correct on Linux without execution.

## 3. Linux risks

1. **Native library availability:** Skiko/Skia native artifacts must resolve for each supported Linux architecture and glibc baseline.
2. **Memory pressure:** full-resolution manga scans can be very large. The candidate has explicit memory ownership/accounting concepts, but target-hardware budgets and eviction behavior require measurement.
3. **Image format variance:** WebP/AVIF/GIF behavior depends on the actual Skia build and codec support; verify real files rather than infer from API comments.
4. **Color/orientation:** EXIF orientation, alpha, color profile, animated frames, and downsampling must be tested with representative fixtures.
5. **Region/tile rendering:** verify that zoomed high-resolution pages do not trigger unbounded native allocations or stale image leases.
6. **Resource lifecycle:** native bitmaps/codecs must be closed on success, cancellation, failure, navigation, and window shutdown.
7. **Wayland/X11:** rendering surface, HiDPI scaling, multi-monitor, fullscreen transitions, and GPU fallback need platform-specific smoke coverage.

## 4. Target hardware considerations

NamiShelf's primary environment includes an 8 GB RAM Linux Mint laptop. Do not assume desktop-class memory or discrete GPU. Establish measurable budgets for:
- maximum decoded image memory;
- tile/region cache;
- concurrent decode jobs;
- prefetch depth;
- memory release after chapter changes;
- startup and first-page latency;
- sustained reading under memory pressure.

The candidate's test contracts and stated cache limits are useful starting points, not target-device performance evidence.

## 5. Required acceptance fixtures

- Large JPEG/PNG/WebP pages, including high-resolution scans.
- EXIF orientations and mirrored images.
- Alpha/transparency and color profiles.
- Corrupt/truncated files and incorrect MIME/content types.
- Animated GIF/WebP where supported.
- Very wide/tall pages and two-page spreads.
- Zoom/region decode at multiple scales.
- Rapid page navigation, chapter switching, cancellation, and repeated open/close.
- Memory pressure and native resource cleanup.
- X11 and Wayland, HiDPI and multi-monitor where available.

## 6. Recommendation

Retain the candidate reader architecture provisionally. Do not rewrite the reader before Linux runtime validation. Treat native image pipeline, memory budgets, codec availability, and GPU fallback as a dedicated Linux acceptance gate.

## 7. Source references

- [SkiaImageDecoder.kt](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/reader/SkiaImageDecoder.kt)
- [Desktop page decoders](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/reader/DesktopPageDecoders.kt)
- [Desktop page image pipeline](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/app-desktop/src/main/kotlin/mihon/desktop/reader/DesktopReaderPageImagePipeline.kt)
- [Reader authority architecture](https://github.com/AltairCardinal/mihon-desktop/blob/701080c251c8fd8b35d0b57a481d93b9a9eb3701/docs/architecture/reader-authority.md)
