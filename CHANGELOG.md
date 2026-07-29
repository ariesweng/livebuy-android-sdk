# Changelog

All notable changes to the Livebuy Android SDK (distributed via this mirror repository) will be
documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Distribution.** The SDK ships as three prebuilt AAR modules (`tv.livebuy:livebuy` /
> `tv.livebuy:livebuy-ui` / `tv.livebuy:livebuy-reference-ui`) served as a static Maven repository via
> GitHub Pages (`https://ariesweng.github.io/livebuy-android-sdk/`). The published Maven `version` is
> read from `LIVEBUY_MAVEN_VERSION` at release time; the channel itself is version-agnostic.

## [4.3.0] - 2026-07-28

> **Minor — no source-breaking change, source-compatible.** Version aligned to the iOS SDK `v4.3.0`
> (cross-platform lockstep) and feature-equivalent on the shared changes. Ships the win-claim email
> flow (`AWARD_CLAIM_RESULT` 4 → 10 keys + a four-stage turnkey claim sheet), product-award
> auto-add-to-cart (`CART_ADD_REQUEST` gains an optional `award_winner_id`), the three-gate join-event
> CTA, and one drop-in bug fix — plus **one behavior change** to `AUTH_STATE_CHANGED.display_name`
> (see the `Changed` section below, read it first). The internal `versionName` (`X-SDK-Version`,
> `1.3.0`) is unchanged. 詳見
> [`docs/release/v4.3.0-tag-runbook.md`](../docs/release/v4.3.0-tag-runbook.md)、
> [真機 e2e 檢查表](../docs/release/v4.3.0-e2e-checklist.md) 與
> [release notes](../docs/release-notes/v4.3.0.md)。

### Changed（⚠️ 行為變更 — 唯一「不改碼但行為會變」的一條，請先讀）

- **⚠️ `AUTH_STATE_CHANGED` 的 `display_name` 語意收斂為「使用者自己選定的名字；未選定時為空字串 `""`」**
  （`9a5bb811`，iOS + Android dual）。**Android 側的實際變化**：`clearUser()` 原本**根本沒帶**
  `display_name` 這個 key（消費端解為 `""`，且與 `tools/event-codegen/events.json` 宣告的必填
  `"display_name": "string"` **契約不完整**）；本版**補上 key**，值為 `""`（或訪客自選暱稱）。
  `setUser` / `state == "logged_in"` 的既有行為**完全不變**。
  - **與 iOS 的差異**：iOS 是從回填系統預設名 `"Guest_4F2A"` 改為 `""`（行為實質改變）；Android 是從
    「沒有 key」改為「有 key 且為空字串」（**解出來的值不變、契約補齊**）。因此 **Android host 受影響的
    可能性遠低於 iOS**——若你原本就對缺 key 做預設空字串處理，本版對你是無感的。
  - **為什麼**：turnkey 的暱稱閘判定是「未登入且 `display_name` 為空 → 要求先設暱稱」。iOS 因為回填了
    非空的預設名，讓「登入過又登出」的訪客被判成「已設過名」，**連留言都不會被要求設暱稱**。
    Android 原本行為才是對的，本版把兩端語意統一並補齊 Android 的 key。
  - **非源碼破壞**：事件名稱、參數 key、參數型別、公開方法簽名**皆不變**，重新編譯不會壞——故仍走 minor。
  - **host 因應**：若你在登出 / 訪客狀態下拿 `display_name` 當「畫面上要顯示的名字」直接用，
    **MUST 自行 fallback**（值為空時改用你自己的訪客預設稱呼，或引導使用者取名）。
  - **明確不受影響：`resolvedDisplayName`** —— 聊天 wire 送出用的名字**仍保留 `Guest_XXXX` fallback，
    一字未改**。留言送出後顯示的名字不會變空白；變的只有 `AUTH_STATE_CHANGED` 帶給你的那個值。

### Added（新公開面，皆 additive、源碼相容、無 breaking）

- **`AWARD_CLAIM_RESULT` params 由 4 key 擴為 10 key**（`c23888f7`，parity iOS `a1be0fd2`；codegen 描述
  `a8adb6ad`）——新增 `winner_id` / `event_title` / `award_name` / `award_expiration` /
  `award_image_url` / `award_stock`；既有 `status` / `award_type` / `event_id` / `award_code` 的語意與
  觸發時機**完全不變**（**純新增 key、向後相容**）。欄位分兩類——**記憶體來源**（`status` /
  `award_type` / `winner_id` / `event_title`，成功失敗都可靠）與 **API 回應來源**（其餘六個，僅成功才
  有）；nil / 空字串的 key **整個省略**；失敗只帶記憶體來源欄位；`award_stock` 含 `0`（＝無庫存）；
  `award_code` / `award_expiration` 僅折扣型獎品。**SDK 領獎成功後不導頁、不渲染**，資訊交 host 處理。
- **`CART_ADD_REQUEST` 新增選填 `award_winner_id`**（`cedea84d`，parity iOS `809741a7`；codegen 描述
  `dd57ae54`）——本筆加購由獎品領獎觸發時才帶，值＝中獎票券 id（同 `AWARD_CLAIM_RESULT` 的
  `winner_id`）；非獎品觸發時**整個省略 key**。typed accessor `LBCartAddRequest.awardWinnerId`
  **刻意為 optional**（缺 key → `null`，不退空字串）。
- **view-model 層新增帶 email 的領獎提交入口**（`c3e09d08`，parity iOS `f1bfb841`）—— 含 email 驗證純
  函式、`submitInFlight` 送出中狀態、`dismissClaim()`。**舊 EMAIL-LESS 入口 deprecated 但保留、源碼相容。**

### Fixed / drop-in behavior（reference-ui + turnkey，drop-in `LivebuyPlayer` 使用者自動生效）

- **中獎領獎補 email 欄位 → turnkey 內建領獎 sheet 改為四階段流程**（`cb60d95c`，parity iOS
  `62133e9c`）—— 由「單頁通知型 sheet」改為 `claim`（填 email）→ `confirmSubmit` / `confirmClose`
  （二次確認）→ `submitting`（送出中）→ `done` / `fail`。**修好一整類「中獎領取失敗」**：core 領獎路徑
  `email` **必填**，而舊 sheet 不收 email，host 未攔截又沒有 email 時 SDK fail-fast、**連領獎請求都沒送
  出**，訪客沒有任何地方能填。關閉為**純 dismiss**（中獎票保留、徽章不變、可再次領取）；fail 卡顯示
  通用錯誤文案（後端不區分失敗原因）。另修好 view-model 讀錯 key 導致 `claimedWinnerId` 取不到值的
  bug（`d73da5ea`，改讀正確的 `winner_id`）。email 為**純聯絡用、非識別鍵**（後端已確認）：填錯不構成
  領獎失敗、同一 email 可領多個獎、登入態可預填會員 email 但應保持可編輯。**訪客確實能參加、中獎、
  領獎**，訪客中獎後才登入**不會掉票**。
- **商品獎品領獎成功後自動加入購物車**（`cedea84d`）—— `award_type == "product"` 的獎品領獎成功後
  SDK 自動加購，該筆領獎共派**兩個事件**：`AWARD_CLAIM_RESULT`（claim 成功即派）→ `CART_ADD_REQUEST`
  （addcart 成功後派）。**discount 型完全不受影響**（只有一個事件，行為一字未改）。加購失敗時獎品
  **仍算領到**（`status` 維持 `claimed`）且依既有契約**不派任何事件**——host 判斷方式＝收到
  `AWARD_CLAIM_RESULT(claimed, award_type=product)` 卻沒有配對的 `CART_ADD_REQUEST`；此情境下 host 只有
  獎品名稱與圖片、**沒有 host 側商品 id**，無法自行補進自家車（刻意的最小對外面積取捨）。獎品加購
  **豁免 30 秒防重複建單窗口**、且**不送**加購轉換埋點（0 元獎品不是加購轉換）。
- **加入活動 CTA 套用與留言一致的三層閘**（`640a40eb`，parity iOS `efcd06a1`）—— 修好「**沒設暱稱卻能
  參加抽獎**」：參加活動本質上就是送一則帶 `event_id` 的口令留言，卻沒有任何閘。現在 drop-in 的加入活動
  CTA 走 ①登入閘（依 `sdkConfig` 訪客留言開關）→ ②暱稱閘（未登入且沒自選過暱稱）→ ③通過才送出，
  並在閘攔截後**續作**（pending-join，完成登入 / 設暱稱後自動接續原動作）。閘攔截時**一併抑制 host
  觀察 callback**（`d33ce9d0`），讓「host 收到 join 通知」與「參加真的送出」的觸發條件收成 **iff**——
  host 不會再收到「假參加」訊號。
- **自動接播後重試會載到已播完那支**（`0b641c28`，Android-only 修復）—— core 自動接播下一支後，drop-in
  容器記的 shown video id 沒跟著同步，使用者一按重試就重新載入**已經播完的那支**。容器現在會在 core
  自動接播後同步該 id。

### Notes

- **未新增 / 移除 / 改名任何既有 host-facing public 符號**、無參數型別變更、無 wire 破壞。本版唯一需要
  host 留意的是上方 `Changed` 小節的 `display_name` 語意（Android 側影響面小於 iOS）。
- **發佈通道**：Pages `maven-metadata.xml` 為**累加**語意——本版上線後**八版並存**
  （3.1.3 / 3.2.0 / 3.2.1 / 3.2.2 / 4.0.0 / 4.1.0 / 4.2.0 / 4.3.0），舊版仍可解析。
- **RN / Flutter 本輪不發**（停在待發 `2.0.0`）；本版四個主題於其主線皆已落地，隨各自 2.0.0 出貨。

## [4.2.0] - 2026-07-22

> **Minor — no BREAKING, source-compatible.** Version aligned to the iOS SDK `v4.2.0` (cross-platform
> lockstep) and feature-equivalent on the shared changes. Adds the in-progress live-event (live giveaway)
> host-facing exposure (all parity with iOS), plus a batch of drop-in reference-ui fixes (turnkey
> giveaway "join" / real shop logos / spec-linked product-detail price & photo) and the `WIN_RECEIVED`
> dedup parity fix. The internal `versionName` (`X-SDK-Version`, `1.3.0`) is unchanged.

### Added（新公開符號，皆 additive、源碼相容、無 breaking）

- **`ACTIVE_EVENT_STARTED` notification event (in-progress live event / live giveaway)** — the SDK
  dispatches this when `POST /sdk/video/goods` returns an `event[]` entry it has not notified before
  (**fire-once per event id**; the dedup set is cleared on video switch). Params (flat):
  `{ id, title, keyword?, duration, surplus, award }` — `keyword` (the "join event" passphrase) is
  omitted when empty, `surplus` is a seconds snapshot at dispatch time (the host counts down locally
  from `duration` + the wall-clock time it received the event), and `award` reuses the winner
  `[{type, name, code}]` shape. **Does not carry `stayTime`** (a turnkey-internal dwell threshold).
  Lets the host draw its own event countdown / prize teaser / join-event entry point.
- **`LBActiveEvent` public model** — `{ id, title, keyword, award, duration, surplus, stayTime }`,
  produced via the `core/dto` → `core/mapper` route (not a Gson-decoded type, consistent with the other
  mapped public models). `LBVideoGoodsResponse.event` is promoted from internal to **public** alongside it.
- **`activeEvents()` public accessor** — returns a snapshot of the in-progress events in the current
  goods cache, covering the late-subscriber blind spot where a host that attaches mid-stream would miss
  the fire-once `ACTIVE_EVENT_STARTED` event.

### Fixed / drop-in behavior（reference-ui + turnkey，drop-in `LivebuyPlayer` 使用者自動生效）

- **Turnkey live-giveaway "join" (drop-in `LivebuyPlayer`)** — when the host does not intercept
  `EVENT_JOIN_INTENT`, the drop-in container now auto-sends the join passphrase comment (carrying
  `event_id` + a pure wall-clock `stay_time` that keeps counting in the background and resets per video),
  so the host needs no wiring to take part. The poll fires a fire-once `eventstay` per in-progress event
  each round. Mirrors iOS (both platforms ship together).
- **Shop logo now drawn as the real image (drop-in player header row + product-info panel shop row)** —
  both shop rows now render the real merchant logo (the gradient monogram chip drops to an always-drawn
  underlay placeholder rather than being the only presentation). Matches iOS / the info panel.
- **Product-detail sheet price / main photo follow the selected spec (drop-in `ProductDetailSheet`)** —
  after picking a spec, the **sale / original price** and the **main photo / zoom lightbox** now switch to
  that spec (previously the price stayed at the product level — a **misleading bug** — and the photo did
  not follow the spec); source validity and the drawn item share a single predicate. Mirrors iOS (both
  platforms ship together).
- **`WIN_RECEIVED` dedup set now cleared on video switch / unload (parity with iOS)** — the winner
  dedup set (`notifiedWinnerIds`) was not reset when switching video or unloading, so a re-entry /
  video switch could wrongly suppress a legitimate `WIN_RECEIVED`. It is now cleared to match the iOS
  behavior (fixes a long-standing four-platform parity gap, not a rename regression).

### Notes

- No new / changed **existing** public symbols; the newly added symbols are additive, so existing host
  code needs no changes. The ACTIVE_EVENT API is for headless-host consumption; the drop-in fixes take
  effect automatically for `LivebuyPlayer` / `LivebuyWidget` users.
- **`WIN_RECEIVED` params KDoc correction (no wire / behavior change)** — the event-registry winner
  params were corrected from a never-populated phantom `name` field to the actual wire `event_id` /
  `title` (the emit logic already sent `event_id` / `title`; only the generated KDoc was wrong from the
  stale source). The **dispatched params are unchanged** — hosts see no difference.
- **Accumulating distribution channel** — `3.1.3` / `3.2.0` / `3.2.1` / `3.2.2` / `4.0.0` / `4.1.0` /
  `4.2.0` coexist; hosts pinned to an older version are unaffected.
- Outward Maven `version` (`4.2.0`) stays decoupled from `:livebuy`'s internal `versionName`
  (`X-SDK-Version`, `1.3.0`); the two differing is normal.

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
