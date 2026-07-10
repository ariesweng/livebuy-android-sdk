# Changelog

All notable changes to the LiveBuy Android SDK (distributed via this mirror repository) will be
documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Distribution.** The SDK ships as three prebuilt AAR modules (`tv.livebuy:livebuy` /
> `tv.livebuy:livebuy-ui` / `tv.livebuy:livebuy-reference-ui`) served as a static Maven repository via
> GitHub Pages (`https://ariesweng.github.io/livebuy-android-sdk/`). The published Maven `version` is
> read from `LIVEBUY_MAVEN_VERSION` at release time; the channel itself is version-agnostic.

## [Unreleased]

_Nothing yet._

---

## [3.1.3] - 2026-07-10

> **First release published through the remote mirror channel.** The version is aligned to the iOS SDK
> `v3.1.3` (cross-platform lockstep — Android and iOS cut `3.1.3` together) and is feature-equivalent
> to the same-version iOS SDK. No BREAKING changes.

### Added

- **Remote distribution channel** — the SDK is now consumable from a public, organization-owned mirror
  repository (`ariesweng/livebuy-android-sdk`) as a static Maven repository served over GitHub Pages.
  Consumers add `maven { url = uri("https://ariesweng.github.io/livebuy-android-sdk/") }` and declare
  `tv.livebuy:livebuy-reference-ui:3.1.3` — no `includeBuild` / composite build required.
- **`LBChannel.begin: Int?`** — prerecorded-live late-join alignment (seconds); non-null only for the
  prerecorded-live case, `null` otherwise. The public constructor gains `begin: Int? = null` (source-compatible).
- **Template-layer `setMuted` / `toggleMute`** are now public, plus default-unmuted playback parity —
  the drop-in containers wire these automatically.
- **Tier 2 unified add-to-cart** — `CART_ADD_REQUEST` forwards `goods_no` / `specification_no`.
- **Pre-configure identity methods are safe no-ops** — calling `setUser` / `clearUser` /
  `setGuestNickname` before `configure()` no longer throws (parity with iOS).

### Fixed

- **LIVE pinned product card close button** no longer bubbles to open the product detail; the close (X)
  dismisses the card, tapping the card body still opens detail.
- **EndScreen "shuffle" (換一批)** now cycles the local recommendation window instead of accidentally
  starting playback of a video.
- **Merged chat history cap 50 → 500**, and chat / activity feeds now retain independently (chat 500 /
  activity 200) so a burst of activity rows no longer evicts chat messages; re-entering restores history.
- **EndScreen recommended-video cards** gain a live-gated cover image + preview animation.
- **In-progress live experience** — product share entry is hidden (design R12), hold-to-pause gesture is
  suppressed (replay / VOD still pausable), and the chat feed auto-scrolls to the newest message on mount.
- **Product images** reload correctly on URL change and clear stale bitmaps during video switch (no one-frame flash).
- **Variant chips** now flex-wrap and show long option text in full (previously clipped).
- **Collapsible player thumbnail** stays in sync when core auto-advances to the next VOD.
- **Chat composer** send button and keyboard Send both submit; chat message line-height tightened.

### Changed (core)

- **System PiP locks the native seek bar for in-progress live** — scrub / fast-forward / rewind are
  disabled while a live stream is in Picture-in-Picture (headless invariant restored).
- **`live_end` tolerates Int / numeric-string wire values.**

### Notes

- **minSdk 24.** Prebuilt AARs are backward compatible — consumers on newer toolchains (AGP 8.7.3 /
  Kotlin 2.0.21 / Gradle 8.9) consume the SDK without the SDK raising its own toolchain
  (AGP 8.2.2 / Kotlin 1.9.22 / Gradle 8.6 / compileSdk 34 stay fixed).
- AWS IVS Player native libs (`libplayercore.so`, arm64-v8a / armeabi-v7a / x86 / x86_64) arrive
  transitively via `com.amazonaws:ivs-player:1.52.0` (resolved from the consumer's `mavenCentral()`);
  the APK grows accordingly.
