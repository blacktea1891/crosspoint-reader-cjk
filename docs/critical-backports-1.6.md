# CrossPoint 1.6 critical backports

This branch intentionally does **not** merge the full CrossPoint 1.6 release-candidate line. It carries a small set of reader fixes that are high-value for the CJK/Sticky build while preserving fork-specific CJK layout, DarkRedrive, input/orientation, font, networking, and recovery behavior.

## Backported fixes

### Long EPUB footnote hrefs

Upstream: `crosspoint-reader/crosspoint-reader#2722`.

`FOOTNOTE_HREF_LEN` is increased from 96 to 256 bytes. Because `FootnoteEntry` is serialized inside section caches, the CJK fork's local `SECTION_FILE_VERSION` is bumped from 35 to 36 so existing caches are rejected and rebuilt instead of being decoded with the new record size.

The version number is deliberately adapted to this fork's cache history rather than copied mechanically from a later upstream value.

### Explicit navigation beats deferred reflow/resume state

Upstream concept: `crosspoint-reader/crosspoint-reader#2962`.

The upstream fix clears stale deferred reader-position state when the user explicitly navigates while background section work is still in flight. This fork has a different deferred-position representation, so the backport clears `cachedChapterTotalPageCount` through `clearDeferredReposition()` at the corresponding explicit-navigation boundaries.

The existing `resetSection()` continues to clear the fork's separate `pendingResumePage` state.

### KOSync generic 2xx handling

Upstream: `crosspoint-reader/crosspoint-reader#2945`.

KOSync-compatible servers may return idiomatic successful HTTP status codes instead of only the small set originally accepted by the client. Authentication, account creation, progress fetch, and progress update therefore accept the appropriate 2xx range. HTTP 204 from progress fetch is treated as the normal `NOT_FOUND` / no-remote-progress case instead of attempting to parse an empty JSON body.

## Verification expectations

Before merging, require the repository's normal clang-format, cppcheck, audit/security contract, release-contract, default target, and Sticky target checks to pass. A Sticky feature-branch artifact should also remain within the configured OTA application-slot limit.
