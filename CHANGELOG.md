# Changelog

All notable changes to the Livebuy Android SDK (distributed via this mirror repository) will be
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

## [4.1.0] - 2026-07-17

> **Minor — no BREAKING, source-compatible.** Version aligned to the iOS SDK `v4.1.0` (cross-platform
> lockstep) and feature-equivalent on the shared changes. Adds the `environment` data-API switch and the
> prerecorded-live live-edge fix (both parity with iOS), plus two Android-only reference-ui line-height
> tightenings. The internal `versionName` (`X-SDK-Version`, `1.3.0`) is unchanged.

### Changed

- **`configure(environment:)` now switches the data-API base URL too (previously `/stat` only)** —
  `LBEnvironment.DEVELOP` now switches both the `/stat` endpoint **and** the data-API base URL to
  `https://develop-admin.livebuy.tv/v1`; `PRODUCTION` (or omitted) keeps `https://api.livebuy.tv/v1`
  (**default behavior unchanged**). The switch takes effect through the single `APIClient.baseURL`
  chokepoint, covering every `/sdk/*` request (config / video / widget / poll / comments / event upload /
  config refresh). The `/sdk/config` local cache key is environment-scoped (`DEVELOP` gets a `_develop`
  suffix) so a prod / dev shared `shopId` cannot cross-pollute; `PRODUCTION` keeps the
  `lb_sdk_config_{shopId}` key so existing production caches upgrade seamlessly. **Only the URL changes,
  not the credentials** (a host switching to `DEVELOP` must supply its own dev credentials — the SDK ships
  none); the HMAC signing mechanism is unchanged (signs `apiKey` + `timestamp` only). Removed the unused
  `LOCAL_BASE_URL` dead code. **No new / renamed host-facing public symbols** (`LBEnvironment` cases
  unchanged — pure behavior extension of an existing parameter).

### Fixed

- **Prerecorded-live live-edge wall-clock anchor fix (fixes 3 bugs in one, drop-in `LivebuyPlayer`)** —
  prerecorded live (`liveStatus == 1`, IVS engine) previously lacked a wall-clock anchor and mistook the
  full-length `duration` for the live edge, causing three symptoms: (1) **after app background / PiP, the
  playhead froze on return to foreground and fell behind "now" without catching up to live** (the only
  user-reachable path); (2) **`isBehindLiveEdge` was misjudged as replay for the whole session** (LIVE
  badge lost, chat locked to replay mode); (3) **back-to-live jumped to the tail** (`performBackToLive`
  sought to `duration`). This version establishes a wall-clock anchor model: on the first begin-align it
  records `(anchorWallClock, anchorPosition = begin)`, and a pure function computes
  `expectedLivePosition = anchorPosition + max(0, now − anchorWallClock)` (wall clock keeps advancing in
  background / sleep); on foreground it re-aligns to live (seeks only when behind by >5s), `isBehindLiveEdge`
  compares against `expectedLivePosition`, and back-to-live seeks to `expectedLivePosition` (clamped to
  `duration`). The anchor persists for the whole session; re-align fires **only on background→foreground**
  (a foreground manual pause is left alone to avoid clashing with a deliberate scrub-back). Fixes a
  long-standing model defect (predates v4.0.0, not a rename regression). Mirrors iOS `35cd642e`
  (both platforms ship together).

- **Line-height tightened on two previously-missed reference-ui paths (Android-only)** — wrapped the
  floating live-entry card `LivebuyLiveEntry` and the manual-assembly entry `WidgetOverlayView` (covers
  carousel / grid / floating / minimized modes in one) in `ProvideTightText`, bringing them to parity with
  `LivebuyWidget` / `CollapsibleLivebuyPlayer` so the LIVE red pill / text line-spacing no longer reads
  loose on device. This is an Android Jetpack Compose-specific concern (`includeFontPadding` default);
  iOS renders correctly already and has no corresponding change.

### Notes

- No new / changed public symbols; existing host code needs no changes. All three fixes / the environment
  extension take effect automatically for drop-in `LivebuyPlayer` / `LivebuyWidget` users.
- **Accumulating distribution channel** — `3.1.3` / `3.2.0` / `3.2.1` / `3.2.2` / `4.0.0` / `4.1.0` coexist;
  hosts pinned to an older version are unaffected.
- Outward Maven `version` (`4.1.0`) stays decoupled from `:livebuy`'s internal `versionName`
  (`X-SDK-Version`, `1.3.0`); the two differing is normal.
- **environment smoke verified** — the develop `POST /sdk/config` returns HTTP 200 / inner code 200 with a
  valid `SDKConfig` body, confirming the base URL is correct, credentials valid, signature correct.

---

## [4.0.0] - 2026-07-16

> **⚠ MAJOR — BREAKING（品牌大小寫識別字改名）。** 全庫程式識別字由 `LiveBuy*` → `Livebuy*`（`liveBuy*` → `livebuy*`），與品牌顯示形（`Livebuy`）一致。**乾淨改名、無 `@Deprecated` alias**——自 `3.2.2` 升級者須一律改匯入的類別名。**不變**：Maven 座標 `tv.livebuy:livebuy` / `tv.livebuy:livebuy-ui` / `tv.livebuy:livebuy-reference-ui`、namespace `tv.livebuy.*`、內部 `versionName`（`X-SDK-Version` = `1.3.0`）、wire 行為。→ **host 的 Gradle 依賴行不變，只需改 `import` 進來的類別名。**

### Changed

- **公開型別改名 `LiveBuy*` → `Livebuy*`**（core `LivebuySDK` / `LivebuyEventListener` / `LivebuyPlayerView` / `LivebuyWidgetCore`、view-model `LivebuyUI`、reference-ui drop-in `LivebuyWidget` / `LivebuyPlayer` / `LivebuyLiveEntry` + 各 `*Config` / PiP helper / `LivebuyWidgetVisibility` 等）。
- **`AndroidManifest` 綁定的 Activity 改名**（`LivebuyPlayerActivity` / `LivebuyPlayerHostActivity`）——以 FQN 啟動它們的 host 須同步改（無 alias 可遮）。

---

## [3.2.2] - 2026-07-15

> **Patch — reference-ui-only, no BREAKING, core / view-model unchanged.** Version converges with the iOS SDK
> `v3.2.2` (both platforms cut 3.2.2 together, continuing the 3.2.0 / 3.2.1 pattern) — same number, different
> diff: both share the presenter-driven widget-cover fix and the `LivebuyWidgetVisibility` KDoc alignment,
> while iOS 3.2.2 additionally carries a PiP-pause foreground-resume fix that is **N/A on Android** (Android has
> no in-PiP pause control / seamless same-player continuation; the AVKit-restore defect is iOS-specific). Same
> number = same parity level (as in 3.2.1); each platform ships through its own channel (Android Maven / iOS
> SPM dist). The internal `versionName` (`X-SDK-Version`, `1.3.0`) is unchanged.

### Fixed

- **Home widget carousel preview yields the decoder when covered / minimized** — the drop-in
  `CollapsibleLivebuyPlayer` presenter now drives `setWidgetsCovered` by phase (fullscreen cover / minimized
  floating), so the home widget carousel preview stops contending with the player for the decoder when it is
  covered by a fullscreen player or shrunk to a floating window, and resumes when the widget becomes visible
  again. Matches iOS presenter routing.
- **`LivebuyWidgetVisibility` KDoc aligned to presenter-owned two paths** — doc comment updated to describe
  the presenter-owned two-path model, removing the stale `presentedVideo != null` example and the accepted
  over-pause framing so the documented behavior matches the presenter routing above (doc-comment only, no
  behavior change).

### Notes

- No new / changed public symbols; existing host code needs no changes. Both fixes take effect automatically
  for drop-in `LivebuyPlayer` / `LivebuyWidget` users.
- All three modules re-publish at `3.2.2` (they share a single outward `version`); `:livebuy` / `:livebuy-ui`
  AAR content is identical to `3.2.1` (== `3.2.0`) — only the coordinate version bumps.
- The channel is accumulating: `3.1.3` / `3.2.0` / `3.2.1` / `3.2.2` coexist; hosts pinned to an older
  version are unaffected.

---

## [3.2.1] - 2026-07-15

> **Patch — reference-ui-only, no BREAKING, core / view-model unchanged.** Version converges with the
> iOS SDK `v3.2.1` and is **behaviorally equivalent** to it (not the same diff: iOS v3.2.1 added
> background→foreground resume, which Android already had; this Android v3.2.1 adds PiP chrome-hiding and
> chat auto-stick, which iOS already had — after both ship, the two platforms match). Both fixes bring
> Android up to existing iOS behavior. The internal `versionName` (`X-SDK-Version`, `1.3.0`) is unchanged.

### Fixed

- **Picture-in-Picture renders video only** — the drop-in `LivebuyPlayer` now hides the entire overlay
  chrome (product card / chat / moments / subtitles) while in system PiP, matching iOS layer-based PiP's
  native video-only presentation; full chrome is restored on return to foreground. (Verified on API 29 device.)
- **Merged chat feed auto-stick to bottom** — new incoming messages push the feed up and keep it pinned to
  the bottom when the user is at the bottom, driven by a persistent `autoStick` intent rather than the
  instantaneous scroll position; scrolling away from the bottom no longer gets yanked back by new messages.
  Fixes "new messages don't push up" and aligns with iOS.

### Notes

- No new / changed public symbols; existing host code needs no changes. Both fixes take effect automatically
  for drop-in `LivebuyPlayer` / `LivebuyWidget` users.
- All three modules re-publish at `3.2.1` (they share a single outward `version`); `:livebuy` / `:livebuy-ui`
  AAR content is identical to `3.2.0` — only the coordinate version bumps.

---

## [3.2.0] - 2026-07-14

> **Minor — default flip (`/stat` opt-in → default-on), no BREAKING, source-compatible.** Version aligned
> to the iOS SDK `v3.2.0` (cross-platform lockstep) and feature-equivalent to the same-version iOS SDK.

### ⚠ Behavior change (read before upgrading): `/stat` analytics now default-on (was default-off)

`configure(...)`'s `enableStatReporting` default flips from `false` to `true`. After upgrading, a host that
does **not** pass `enableStatReporting` will **start sending** `/stat` (10 event types — view counts / shares /
add-to-cart / product impressions; endpoint `https://livebuy.tv/stat`, unsigned, form-urlencoded,
fire-and-forget). To keep it off, pass `configure(..., enableStatReporting = false)` (fully preserves the
headless no-op — nothing generated / persisted / sent). Safe to default-on because the `/stat` wire body
carries only `type` + video / goods `id` (+ a few flags) — **no PII, no device id, no ip** (backend derives
IP from `X-Forwarded-For`). This flips `/stat` only; `enableConversionAttribution` (Meta attribution ids
`fbp` / `fbc` = PII) stays opt-in / default-off. ATT / GDPR consent remains the host's responsibility — a
strict-compliance host should explicitly pass `enableStatReporting = false`.

### Added

- **Native `/stat` analytics subsystem (10 types)** — the SDK emits view / share / add-to-cart / product
  impression stats natively, including `person_time` (watch duration) and `person_duration` (foreground dwell)
  timers. **Default-on** (see behavior change); endpoint selectable to develop.
- **`configure(..., enableStatReporting: Boolean = true, environment: LBEnvironment = LBEnvironment.PRODUCTION, enablePowerProfileAdaptation: Boolean = true)`**
  — three new parameters, all with defaults (existing call sites unaffected).
- **New `LBEnvironment`** (`PRODUCTION` / `DEVELOP`) — global environment selector; currently switches the
  `/stat` endpoint (`DEVELOP` → `https://develop.livebuy.tv/stat`). Endpoint-only; does not change whether
  stats are sent or the wire body / dedupe / timer / no-HMAC contracts.
- **`notifyPictureInPictureModeChanged`** (public) — host forwards from the Activity's
  `onPictureInPictureModeChanged`, closing the View-mode PiP gap.
- **`LBEvent.POWER_PROFILE_CHANGED`** constant (emit logic is a follow-up; not emitted in this version).

### Fixed

- **Live heat optimization** — thermalStatus-aware auto-throttle (quality cap + polling backoff scaled by
  temperature tier), the two live 5s polls coalesced onto a single loop tick (radio-wakeup power saving), and
  a screen-aware quality cap that lowers live-decode heat. The reference-ui winner pulse ring throttles by
  power profile.
- **Widget preview / player background power** — app background (`ON_STOP`) pauses the fullscreen player and
  widget carousel preview; off-screen widgets pause decoding; a widget covered by a fullscreen player pauses
  the underlying preview — resuming on foreground / visibility. Eliminates ~150% background CPU heat.
- **`LivebuyPlayerView.unload()` idempotency guard** — repeated `unload()` no longer re-dispatches the end
  event (aligns with iOS).
- **Wire field null tolerance** — 4 nullable wire fields tolerated to remove mapper NPEs (aligns with iOS).

### Changed

- **onsale product-card dead code removed** — unused product-launch card producer / card code cleaned up
  (no behavior change).

### Notes

- **Accumulating distribution channel** — `3.1.3` and `3.2.0` coexist in the channel; hosts pinned to
  `3.1.3` are unaffected.
- Outward Maven `version` (`3.2.0`) is decoupled from `:livebuy`'s internal `versionName` (`X-SDK-Version`);
  the two differing is normal.

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
