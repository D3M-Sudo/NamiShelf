# Reader Runtime and Page Pipeline

**Status:** Bounded lazy loading with LRU cache and controlled prefetch approved under ADR-018. Numeric budgets and concrete libraries remain deferred to M0 and measured profiling.

## 1. Goals

The reader must support long chapters and high-resolution pages without eagerly decoding an entire chapter or allowing decoded image memory to grow without a bound. It must remain responsive during navigation and recover from individual page failures.

## 2. Pipeline boundaries

1. **Page source:** resolves a page's local download/cache artifact or requests it through the source/download application boundary.
2. **Byte access:** streams or reads the page payload without forcing the full chapter into memory.
3. **Decode scheduler:** performs image inspection and decoding off the Compose/UI thread, with bounded concurrency and cancellation.
4. **Decoded-page cache:** stores reusable decoded page representations under an explicit memory budget and LRU eviction policy.
5. **Presentation:** Compose displays already-available page state, loading placeholders or recoverable error placeholders. Recomposition must not itself start uncontrolled I/O or decoding.
6. **Reading progress:** persists the stable chapter/page/position through application use cases, independently of cache contents.

## 3. Scheduling policy

- Prioritize visible pages.
- Request a small configurable neighborhood around the current position.
- Prefetch a bounded number of adjacent pages, biased toward the likely navigation direction.
- Re-evaluate priorities when the user jumps, changes direction, switches chapter or exits the reader.
- Cancel obsolete queued/in-flight work where safe; deduplicate requests for the same page.
- Bound concurrent I/O and decoding separately where the implementation requires it.
- Do not let prefetch starve visible-page work.

## 4. Cache policy

- Use an LRU policy for decoded page objects.
- Enforce a byte-oriented memory budget; page count alone is not a reliable memory bound because dimensions vary.
- Evict least-recently-used entries that are not currently pinned for presentation.
- Treat downloaded source files, disk cache and decoded bitmap cache as separate resource layers with separate lifecycle and limits.
- Respond to memory pressure by trimming decoded entries and reducing speculative prefetch.
- Cache eviction must never delete the user's durable downloaded chapter or alter reading progress.

## 5. Error and lifecycle behavior

- A page decode failure produces a local placeholder and retry action; it must not crash the reader or invalidate other pages.
- Unsupported formats and corrupt data must be distinguishable in diagnostics without leaking sensitive paths or source credentials.
- Chapter switch, reader exit and rapid navigation must invalidate stale callbacks and release unneeded resources.
- Persist reading progress independently of composition lifecycle and cache retention.
- Background downloads continue independently of the reader screen lifecycle.

## 6. Observability

Instrument page request, byte availability, decode start/end, first visible render, cache hit/miss/eviction, cancellation and failure. Diagnostics must avoid logging credentials, cookies, signed URLs or page contents.

## 7. Deferred parameters

M0 and profiling must select the decoder/library, cache byte budget, page-count safety ceiling, prefetch radius, direction policy, concurrency, decode subsampling, disk-cache policy and platform memory-pressure integration. Do not choose arbitrary numeric values before testing representative fixtures on the Linux reference environment.

## 8. Validation scenarios

Release validation must cover:
- very long chapters and high-resolution pages;
- mixed dimensions, aspect ratios and supported image formats;
- rapid forward/backward navigation and page jumps;
- repeated chapter entry/exit and cancellation;
- corrupt and unsupported images;
- memory pressure and cache trimming;
- download completion while reading;
- reader restoration after application restart;
- measured peak/sustained memory, page-ready latency, stalls and cache hit/eviction rates.

No implementation, build, dependency installation or test execution is authorized by this specification.
