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

> First release published through the remote mirror channel. The version string is aligned to the
> iOS SDK `v3.1.3` (cross-platform lockstep — Android and iOS cut `3.1.3` together).

### Added

- **Remote distribution channel** — the SDK is now consumable from a public, organization-owned mirror
  repository (`ariesweng/livebuy-android-sdk`) as a static Maven repository served over GitHub Pages.
  Consumers add `maven { url = uri("https://ariesweng.github.io/livebuy-android-sdk/") }` and declare
  `tv.livebuy:livebuy-reference-ui:3.1.3` — no `includeBuild` / composite build required.
- **Feature parity with iOS `v3.1.3`** — Tier 2 unified add-to-cart (`CART_ADD_REQUEST`),
  the player's default share sheet (when the host does not set `onShare`), and
  `bindSession(memberId, memberName, avatarUrl)` membership.

### Notes

- **minSdk 24.** Prebuilt AARs are backward compatible — consumers on newer toolchains (AGP 8.7.3 /
  Kotlin 2.0.21 / Gradle 8.9) consume the SDK without the SDK raising its own toolchain
  (AGP 8.2.2 / Kotlin 1.9.22 / Gradle 8.6 / compileSdk 34 stay fixed).
- AWS IVS Player native libs (`libplayercore.so`, arm64-v8a / armeabi-v7a) arrive transitively via
  `com.amazonaws:ivs-player:1.52.0` (resolved from the consumer's `mavenCentral()`); the APK grows
  accordingly.
