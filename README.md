# Livebuy Android SDK

Embed live shopping experiences — live streams, replays, and shoppable VODs — directly into your
Android app.

> **Architecture.** The SDK ships as **three prebuilt AAR modules** under a single Maven coordinate
> family. `tv.livebuy:livebuy` is a **headless** core (it renders no UI). The turnkey, ready-to-use UI
> ships as drop-in Jetpack Compose containers (`LivebuyPlayer` / `LivebuyWidget` / `FloatingWidgetView`)
> in **`tv.livebuy:livebuy-reference-ui`**. Most integrators declare only `livebuy-reference-ui` and
> Gradle resolves the whole chain. Teams that want to draw every pixel themselves depend on the
> headless core directly and compose their own views.

> **Prebuilt AARs only.** All three modules — including the two UI layers — are distributed as
> compiled `.aar` artifacts; **no source is published** in this repository. (This is an intentional
> difference from the iOS SDK, whose UI layers ship as source.)

---

## Requirements

| | Minimum | Verified against (consumer) |
|---|---|---|
| `minSdk` | **24** | — |
| Consumer `compileSdk` | **34+** | 35 |
| Consumer AGP | **8.2.2+** | 8.7.3 |
| Consumer Kotlin | **1.9.22+** | 2.0.21 |
| Consumer Gradle | **8.6+** | 8.9 |
| Java (build) | **17** | — |

### The SDK does NOT upgrade its own toolchain for newer consumers

A **prebuilt AAR is backward compatible**: a Kotlin 2.0.21 app consumes a Kotlin 1.9.22-built AAR, and
an AGP 8.7.3 app consumes an AGP 8.2.2-built AAR, without issue. Therefore the SDK **does not** raise
its own toolchain (**AGP 8.2.2 / Kotlin 1.9.22 / Gradle 8.6 / `compileSdk 34`** stay fixed) to serve
consumers on newer toolchains. Consuming a prebuilt AAR — **not** a composite `includeBuild` — is what
sidesteps AGP's "two AGP versions in one build" hard error (consumer 8.7.3 vs SDK 8.2.2).

---

## Installation

The SDK is served as a **static Maven repository via GitHub Pages**. Add the Pages URL as a Maven
repository (keep `mavenCentral()` — the AWS IVS Player native libs are resolved transitively from
there), then declare the coordinate:

### Gradle (Kotlin DSL)

```kotlin
// settings.gradle.kts — dependencyResolutionManagement, or your module build.gradle.kts
repositories {
    maven { url = uri("https://ariesweng.github.io/livebuy-android-sdk/") }
    google()
    mavenCentral()   // resolves the transitive com.amazonaws:ivs-player:1.52.0
}

dependencies {
    // Drop-in default UI (most integrators want this) — pulls livebuy-ui + livebuy transitively.
    implementation("tv.livebuy:livebuy-reference-ui:3.2.2")
    // Headless core only (composing your own UI):
    // implementation("tv.livebuy:livebuy:3.2.2")
}
```

### Gradle (Groovy DSL)

```groovy
repositories {
    maven { url 'https://ariesweng.github.io/livebuy-android-sdk/' }
    google()
    mavenCentral()
}

dependencies {
    implementation 'tv.livebuy:livebuy-reference-ui:3.2.2'
}
```

> **Only declare `reference-ui`.** The three modules form a one-way chain
> (`reference-ui → livebuy-ui → livebuy`), so declaring the reference-ui coordinate resolves the whole
> chain transitively — including the AWS IVS Player native libs (`libplayercore.so`, arm64-v8a /
> armeabi-v7a) that arrive via `com.amazonaws:ivs-player:1.52.0` (`runtime` scope in the core POM) and
> are packed into your APK.

> **⚠️ IVS binary & APK size.** The live low-latency engine (AWS IVS Player) is pulled in transitively;
> your APK gains the IVS native `.so` libs. This is required for live HLS playback.

### Coordinates

| Module | Maven coordinate | Role |
|---|---|---|
| Core (headless) | `tv.livebuy:livebuy` | networking, sessions, playback-engine selection |
| View-model layer | `tv.livebuy:livebuy-ui` | headless (zero-pixel) view-models |
| Reference UI | `tv.livebuy:livebuy-reference-ui` | drop-in Compose UI (`LivebuyPlayer` / `LivebuyWidget` / `FloatingWidgetView`) |

---

## Getting Started

### 1. Configure the SDK

Call `configure` once at app launch, before any SDK feature is used. It is a `suspend fun`, so call it
from a coroutine.

```kotlin
import tv.livebuy.sdk.LivebuySDK

lifecycleScope.launch {
    LivebuySDK.configure(
        context = applicationContext,
        apiKey = "12345",        // String (numeric string) — provided by Livebuy
        secret = "your-secret",  // HMAC signing secret — provided by Livebuy
        shopId = "Pw8PJ99J",     // required — provided by Livebuy
        // lang = "en",          // optional language override (zh-TW / zh-CN / en / ms-MY / id-ID)
    )
}
```

### 2. Present the Player

`LivebuyPlayer` is a turnkey Jetpack Compose container. It assembles the headless engine + the default
chrome (header / rail / info panel / product + chat overlays) and starts playback.

```kotlin
import tv.livebuy.reference.LivebuyPlayer

setContent {
    LivebuyPlayer(videoId = "abc123")
}
```

Wire interactions through the player config (every closure is optional with a sensible default — but
`onLogin` is effectively required for any integration whose guests may hit a login gate):

```kotlin
LivebuyPlayer(
    videoId = "abc123",
    onProductTap = { product -> /* open your product detail page — product.id, product.name, product.goodsGpn */ },
    onLogin = { /* open your login flow; on success call LivebuySDK.login(...) */ },
)
```

### 3. Embed a Widget

`LivebuyWidget` lists a shop's videos (horizontal carousel or infinite-scroll grid).

```kotlin
import tv.livebuy.reference.LivebuyWidget

LivebuyWidget(shopId = "Pw8PJ99J")                          // carousel (default)
LivebuyWidget(shopId = "Pw8PJ99J", mode = WidgetMode.Grid) // infinite-scroll grid
```

Tapping a card does nothing until you wire it:

```kotlin
LivebuyWidget(
    shopId = "Pw8PJ99J",
    onTapVideo = { video -> /* open your player page — video.id, video.title */ },
    onSeeMore = { /* push your "see all" page */ },
)
```

> The full integration guide (Kotlin + Compose — login / add-to-cart / view-cart) lives in the monorepo
> handoff doc; request it from Livebuy.

---

## Membership, Cart & Events

The SDK funnels every user interaction and system signal through a single event listener. Install one
listener and dispatch by `eventName` (`CART_ADD_REQUEST`, `AUTH_REQUIRED`, `PRODUCT_CLICK`, …). Attach a
logged-in user with `LivebuySDK.login(memberId, memberName, avatarUrl)` /
`LivebuySDK.bindSession(...)`; clear on logout with `LivebuySDK.clearUser()`.

Tier 2 add-to-cart: the SDK adds to the Livebuy backend cart, then fires `CART_ADD_REQUEST` as a
notification — the host adds to its own cart and (best-effort) calls `reportCartTrack(...)` for
attribution. See the integration guide for the full event catalogue and cart flow.

---

## Localization

Active language resolution priority: `setLanguage(...)` at runtime > `configure(lang=...)` >
API-returned `lang` > fallback `zh-TW`. Supported: `zh-TW` (default) / `zh-CN` / `en` / `ms-MY` / `id-ID`.

---

## Manifest / Runtime notes

- **Background audio** — the SDK uses a media session for background playback.
- **Picture in Picture** — requires API 26+; the host Activity needs
  `android:configChanges="screenSize|smallestScreenSize|screenLayout|orientation"`.

---

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

---

## License

Copyright © Livebuy. All rights reserved.
