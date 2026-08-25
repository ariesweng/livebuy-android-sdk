# Changelog

All notable changes to the Livebuy Android SDK (distributed via this mirror repository) will be
documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

> **Distribution.** The SDK ships as three prebuilt AAR modules (`tv.livebuy:livebuy` /
> `tv.livebuy:livebuy-ui` / `tv.livebuy:livebuy-reference-ui`) served as a static Maven repository via
> GitHub Pages (`https://ariesweng.github.io/livebuy-android-sdk/`). The published Maven `version` is
> read from `LIVEBUY_MAVEN_VERSION` at release time; the channel itself is version-agnostic.

## [4.8.0] - 2026-08-25

> **Minor.** 延續 v4.7.0 剛上線的「更多商品」推薦格，本輪多項精進＋新增商品選項可購性計算＋
> icon 對齊新版設計稿。有 2 項 package-internal BREAKING（非 public API 簽章變更，見下方
> Changed）：推薦格導覽簡化（移除巢狀 breadcrumb 返回）；iOS-only 拖曳手勢局部回退（不影響
> Android）。版號對齊 iOS SDK `v4.8.0`（兩端 lockstep）。內部 `versionName`
> （`X-SDK-Version`，`1.3.0`）不變。iOS 對照見
> [`livebuy-ios-sdk/CHANGELOG.md`](../livebuy-ios-sdk/CHANGELOG.md#480---2026-08-25)。

### Added

- **商品選項不可購性計算＋disabled 攔截**：多層規格選項（如顏色×尺寸）改用精確比對取代先前的
  子字串搜尋，不可購組合顯示 disabled 灰階並攔截點擊。
- **`LBProduct` 新增 `description` 欄位（`String?`）**：承載商品真實介紹文字，Gson 預設容錯
  （缺鍵/null → `null`）。
- **「更多商品」推薦格新增原價劃線顯示**：推薦卡片新增原價欄位透傳與劃線渲染，並修正呼叫端
  `hideSub` 判斷邏輯缺口（劃線資料已透傳但呼叫端先前沒有正確依「原價是否非空」決定是否顯示），
  修正後原價劃線功能才真正生效。
- **icon 對齊新版設計稿**：8 個 composable 改寫/新增對齊 `icons.jsx` 新版設計來源，30 張
  Roborazzi baseline 重生。純視覺，不影響互動行為。
- **商品袋 row 播放提示改為「看講解」白底膠囊**：對齊設計稿 R21，tap handler 邏輯不變。

### Changed

- **⚠️ 「更多商品」推薦格導覽簡化（BREAKING，reference-ui 內部行為，非 public API）**：移除
  v4.7.0 引入的 breadcrumb 逐層返回機制，header 關閉鈕永遠是「✕ 全部關閉」；播放圖示換片後
  額外呼叫既有的 `closeDetail()`，整個商品 sheet stack 隨換片一併關閉（v4.7.0 是「换片不連動
  dismiss」）。如果你的 host 依賴 v4.7.0「推薦格巢狀返回」行為，本版已改變。
- **「更多商品」推薦格上限 4 → 12**。
- **換片時一併關閉外層商品袋/清單抽屜**：先前換片後外層抽屜若已開啟會維持開著，本版起一併關閉。
- **商品介紹文字區改顯示真實資料**：v4.7.0 上線時固定顯示佔位文案，本版接上真實
  `LBProduct.description`；**沒有真實介紹文字時整個區塊（含標題）都不顯示**，不再顯示佔位文案
  （比照既有 `brief` 欄位「空字串不畫」規則對齊）。
- **更多商品卡片版面調整**：原價移到售價下方、加購鈕貼齊卡片底部；grid 卡片移除邊框。
- **商品明細 sheet 底部灰色間隔修正**：捲到底時原本會露出外層灰色背景的間隔，本版移入白卡內側，
  捲到底不再露灰。
- **iOS-only 拖曳手勢局部回退**（不影響 Android）：iOS 端把 v4.7.0 整併的拖曳調高/收合單一
  連續手勢局部撤回為各自獨立判斷；Android 維持 v4.7.0 的整併手勢不變。

### Fixed

- **推薦資料源改從容器持久快取退回提供**：修正特定邊界情況（容器快取命中但欄位為空）下漏抓
  推薦資料的問題；純技術韌性修正，行為對使用者不可見。

## [4.7.0] - 2026-08-25

> **Minor.** 新增 3 項 host-facing 能力——商品明細/加購 sheet 的 Sale 促銷徽章、`LBProduct` 補上
> `videoId` 欄位、商品明細新增「商品介紹」區塊＋「更多商品」2×2 推薦格；另把底部 sheet 的拖曳
> 調高與拖曳收合手勢整併為單一連續手勢並擴大到全部 5 個 sheet。版號對齊 iOS SDK `v4.7.0`（兩端
> lockstep）。內部 `versionName`（`X-SDK-Version`，`1.3.0`）不變。iOS 對照見
> [`livebuy-ios-sdk/CHANGELOG.md`](../livebuy-ios-sdk/CHANGELOG.md#470---2026-08-25)。

### Added

- **商品明細 / 加購 sheet 新增 Sale 促銷徽章**：商品有原價（劃線價）且未售完時，商品圖 / 96×96
  縮圖旁顯示一個「Sale」徽章 chip（accent 底色、白字，`SaleBadge` composable）；純 reference-ui
  視覺渲染，不需任何新的資料欄位或 view-model 改動，售完或無原價時不顯示。
- **`LBProduct` 新增 `videoId` 欄位（`String?`）**：承載 `LBChannel.otherGoods[]` 每筆商品所屬的
  影片 id（一般 `goods[]` 內項目此欄位為 `null`），Gson 預設容錯（缺鍵/null → `null`）。補齊
  `component-contracts` 規格先前已要求、但四端從未真正實作的缺口，是下方「更多商品」推薦格能夠
  換片的必要資料來源。
- **商品明細新增「商品介紹」文字區 ＋「更多商品」2×2 推薦格**：`DETAIL` 呈現底部新增商品介紹
  說明文字（後端 `description` 欄位就緒前，先以固定文案佔位）與最多 4 筆「更多商品」推薦卡片
  （資料源 `LBChannel.otherGoods`，過濾掉目前商品）。點推薦卡的播放圖示會直接換到該商品所屬
  影片（沿用既有容器層換片機制，比照 EndScreen 熱門推薦的既有模式，不新增換片入口）；點卡片
  本體或加購鈕會切換到該商品自己的明細/加購畫面——同一個 sheet 換內容＋本地返回路徑（Compose
  `Box` 疊層比照既有 zoom overlay 寫法，非疊出第二個 sheet 實例），header 關閉鈕在有返回路徑時
  變成「返回」；從推薦卡加購會帶該商品自己的 `videoId`，確保購物車去重鍵 `(goodsId, videoId)`
  正確。商品卡新增 `hideSub`/`onPlayClick` 兩個渲染參數，grid 呈現的播放鈕改為右上角呼吸動畫
  圓鈕＋獨立加購圓鈕，既有 row 呈現的播放提示改為「看講解」文字膠囊（既有 `RowCartButton` 不
  動）。
- **Roborazzi baseline 新增/重錄**：Sale 徽章新增後，既有帶原價未售完的 in-stock snapshot（含
  `ProductDetailSheetSnapshotTest` / `AddToCartSheetSnapshotTest`）已重新錄製；新增反例（無原價 /
  售完不顯示）baseline。

### Changed

- **底部 sheet 拖曳調高與拖曳收合整併為單一連續手勢，並擴大到全部 5 個 bottom sheet**：先前僅
  商品明細 / 加購 / 補貨通知三張 sheet 可選擇性拖曳調高（`resizable = true` opt-in，下限寫死
  25%），本版起商品列表抽屜（`ProductListSheet`）與影片資訊面板（`VideoInfoPanel`）也一併具備
  拖曳調高能力；往上拖調高、往下拖收合合併成同一條手勢——高度下限改為「該次呈現實際渲染出的
  高度」而非寫死值，超出下限才轉為既有的拖曳收合位移（沿用既有 100dp 累積位移門檻與彈回/滑出
  動畫，門檻本身不變）。**商品明細（`DETAIL`）呈現的預設高度上限由 v4.6.2 的 90% 改回 50%
  （內容自適應）**——90% 現在只在使用者主動拖曳到頂時才出現，不再是開啟就逼近全螢幕的預設值；
  如果你的 host 依賴 v4.6.2「明細一開啟就是 90%」的行為，這個預設值本版已改變。`ADD_TO_CART`
  與補貨通知既有固定 40% 高度不受影響。

## [4.6.2] - 2026-08-24

> **Patch.** 觀看人數進場假 0 修復 + 一批 reference-ui 視覺/互動細節收斂，無新增符號、無破壞性
> 變更。版號對齊 iOS SDK `v4.6.2`（兩端 lockstep）。內部 `versionName`（`X-SDK-Version`，
> `1.3.0`）不變。iOS 對照見
> [`livebuy-ios-sdk/CHANGELOG.md`](../livebuy-ios-sdk/CHANGELOG.md#462---2026-08-24)。
> 本版 2 項為 **iOS-only**（下方已標註），Android 無對應改動——四端一致的其餘項目皆兩端同步。

### Fixed

- **觀看人數進場顯示假 0**：`publishMomentState()` 組裝 `viewerCount` 時，`channel` 尚未 resolve
  （含 `unload()` 清空 channel）不再硬編覆寫成 `0`，改為沿用上一個已知值；`VideoInfoPanel` 觀看
  人數徽章新增第四道顯示閘——冷啟動、真實資料尚未到位期間不渲染任何具體數字（含 `0`）。兩者
  共同解決「一進入直播間先顯示 0、過一陣子才變成正常人數」的症狀。
- **公告分頁為空時不再顯示灰階死路徑**：`VideoInfoPanel` 的公告分頁在系統公告與商城公告皆空
  時，改為整個不渲染，不再畫出永遠點不動的 disabled 灰階分頁。
- **商品袋縮圖跳轉後自動收合**：商品清單抽屜內點擊商品縮圖跳轉到影片對應時間點時，同步關閉
  商品清單抽屜。
- **PlayerHeader 商家 pill 背景收斂**：移除整塊商家資訊（logo / 標題 / 商家名稱 / LIVE 標籤 /
  觀看人數）共用的半透明灰底，改由觀看人數獨立套用該背景，對齊最新設計稿。
- **Feed 訊息頭像/icon 先隱藏**：聊天 / 活動 feed 每則訊息前方的 24dp 圓形頭像/icon 槽暫時隱藏，
  文字/氣泡貼齊列最左側起點（可逆的暫時性設計決定，繪製邏輯保留）。
- **商品明細 sheet 拖曳調整高度 + 收藏鈕橫排內置 + 明細呈現拉高到 90%**：商品明細 / 加入購物車 /
  補貨通知三個底部 sheet 的把手新增拖曳即時調整高度（25%–90%）能力；收藏鈕從底部操作列移到
  內文區塊置中橫排；商品明細（`DETAIL`）呈現高度上限由 50% 提高到 90%。
- **直播疊層聊天室左邊距 / 釘選商品卡右邊距對齊底部 icon**：對齊底部功能列購物袋（左）與愛心
  （右）icon 的既有 10dp 邊距。
- **商品搜尋框移除清除鈕，只留取消**：商品清單 sheet 展開態搜尋框移除叉叉清除鈕，只留取消
  （收合整個搜尋列並清空查詢字串）。

### Changed

- **直播進行中停用垂直滑動切影片**：先前任何播放狀態下垂直滑動皆會切換影片，本版起直播正在
  進行中時滑動不再切換影片（拖曳仍會被手勢層吞掉，不會誤觸 tap-to-mute / hold-to-pause）；
  預告倒數（upcoming）與已結束直播的回放（finished-live replay）不受影響，維持可滑動切片。

### 不在本版（iOS-only，Android 無對應改動）

- 底部 sheet 拖曳關閉不再跳動。
- 聊天氣泡垂直間距收緊（iOS `LBChatLineRow` 一般觀眾留言分支）。

## [4.6.1] - 2026-08-13

> **Patch.** 補齊一個既有背景繪製語意的缺口，無新增符號、無破壞性變更。版號對齊 iOS SDK
> `v4.6.1`（兩端 lockstep），本輪共用功能為 feature-equivalent。內部 `versionName`
> （`X-SDK-Version`，`1.3.0`）不變。iOS 對照見
> [`livebuy-ios-sdk/CHANGELOG.md`](../livebuy-ios-sdk/CHANGELOG.md#461---2026-08-13)。

### Fixed

- **`ScrollableCarouselView`（`CarouselView.kt` 同檔內獨立 composable，`MinimalDesign.WidgetCarousel`
  使用的 turnkey 全量水平捲動變體，`LivebuyWidget` drop-in 容器實際渲染的表面）根 `Column` 補畫
  `widget_bgcolor` 衍生後的背景色**，比照既有「窗口式」`CarouselView`（`WidgetOverlayView`
  CAROUSEL 分支使用的表面）與 `VideoShopGridView` 語意：合法 hex 覆寫背景；缺值 / 空字串 /
  `null` 維持既有背景值不變，不引入新預設色。`widget_color`（文字色反轉）與背景色可同時獨立
  生效。這是 v4.6.0 讓「窗口式」`CarouselView` 補畫背景時明確排除、並寫進 spec 當作正式
  Requirement 排除條款的缺口（`ScrollableCarouselView` 維持不繪製背景的現狀）——本版推翻該
  排除，drop-in 容器實際渲染的輪播 widget 才真正會顯示商家設定的背景色。
  > 測試環境備註：Roborazzi 的 composable-only capture（無 host Activity）下，
  > `ScrollableCarouselView` 未設定背景時先前渲染為完全透明像素；補畫後即使未設定
  > `widget_bgcolor`，該區域也會變成不透明的既有背景色（色值不變，只有 alpha 從 0 變 255）——
  > 這只影響測試快照，不影響真實 App 內的實際顯示效果。2 個既有 Roborazzi baseline 因此重新
  > 錄製，非行為 regression；另有一個既有色彩反轉 golden（`widget-embed-inverted-scrollable-carousel.png`）
  > 因新背景繪製需要程式碼修正（顯式提供匹配的 `widgetBgcolor`），非單純重錄。

## [4.6.0] - 2026-08-12

> **Minor.** 兩項 reference-ui 新增設定面，皆 additive，無移除、無破壞性變更。版號對齊 iOS SDK
> `v4.6.0`（兩端 lockstep），本輪共用功能為 feature-equivalent。內部 `versionName`
> （`X-SDK-Version`，`1.3.0`）不變。iOS 對照見
> [`livebuy-ios-sdk/CHANGELOG.md`](../livebuy-ios-sdk/CHANGELOG.md#460---2026-08-12)。

### Added

- **`LivebuyPlayerConfig` 新增 `position: String?` 欄位**（DEFAULT `null` → 右下角，即既有
  落點）。`CollapsibleLivebuyPlayer` 縮小後出現的懸浮預覽卡先前恆固定右下角，現在依此欄位比照
  「現正直播」入口（`LivebuyLiveEntryConfig.position`）換邊——複用同一套
  `LBFloatingEntryPosition.normalized()` 正規化邏輯（`LivebuyLiveEntry.kt`），懸浮卡靜止對齊 /
  padding / 拖曳夾限三處共用同一次解析結果。未注入時渲染與現況逐位元組相同（既有
  Roborazzi baseline 零變動）。
- **`CarouselView`（輪播 widget）根 `Column` 補畫 `widget_bgcolor` 衍生後的背景色**，比照既有
  `VideoShopGridView` 語意：合法 hex 覆寫背景；缺值 / 空字串 / 不可解析維持既有背景值不變，不
  引入新預設色。`widget_color`（文字色反轉）與背景色可同時獨立生效。`ScrollableCarouselView`
  （獨立的 turnkey 捲動變體）維持不繪製背景的現狀，不在本項範圍內。
  > 測試環境備註：Roborazzi 的 composable-only capture（無 host Activity）下，`CarouselView`
  > 未設定背景時先前渲染為完全透明像素；補畫後即使未設定 `widget_bgcolor`，該區域也會變成
  > 不透明的既有背景色（色值不變，只有 alpha 從 0 變 255）——這只影響測試快照，不影響真實 App
  > 內的實際顯示效果（真實 App 的輪播容器本來就疊在有背景色的頁面之上）。4 個既有 Roborazzi
  > baseline 因此重新錄製，非行為 regression。

## [4.5.0] - 2026-08-12

> **Minor — 一條 BREAKING 移除，但實務衝擊視為零（見下方 `Removed` 說明）。** 版號對齊 iOS SDK
> `v4.5.0`（兩端 lockstep），本輪共用功能為 feature-equivalent。內部 `versionName`（`X-SDK-Version`，
> `1.3.0`）不變。iOS 對照見 [`livebuy-ios-sdk/CHANGELOG.md`](../livebuy-ios-sdk/CHANGELOG.md#450---2026-08-12)。

### Removed（⚠️ BREAKING）

- **⚠️ 移除 `LBWidgetResponseDTO` / `LBWidgetResponse` 的 `showGoods: Int?`**（含 data class 建構子
  參數）。該欄位對應的 wire key `show_goods` **後端從未 emit**，因此它永遠是 `null` —— 留著等於在
  public API 上擺一個看起來可用、實際永遠沒值的假設定。其原本標註的語意（「0=名後 / 1=影片中 /
  2=不顯示」）源自後端 repo 一則已被該 repo 自己更正的稽核錯誤。**遷移**：wire 上真正承載「商品卡
  顯示模式」的是新增的 `productCard`（見下方 `Added`）。讀過 `showGoods` 的 host 只可能拿到
  `null`，改讀 `productCard` 即可；若原本就寫了 null 分支，刪掉該欄位的引用即可編譯。
  **為什麼仍是 minor、不是 v5.0.0**：該欄位對應的 wire key 後端從未送過值，repo 內零消費端——沒有
  任何一個真實 host 讀過非 `null` 的值。實務衝擊視為零，這是團隊已確認的判斷，非自動套用 SemVer
  字面規則。

### Added

- **`LBWidgetResponse.productCard: String?`** —— `POST /sdk/widget` 回應 root 的 `product_card`
  raw passthrough。語意為 widget 輪播卡上「商品卡」的顯示模式，後端值域 `below`（卡片下方）/
  `inside`（卡內疊層）/ `hidden`（不顯示），後端預設 `inside`。**SDK 不解讀語意、不據此排版**。
  缺欄位或 JSON `null` → `null`，**SDK 刻意不補後端預設 `"inside"`**，讓 UI 層能區分「後端沒送」
  與「後端明確送 inside」。
- **`LivebuyWidgetCore.productCard: String?`** —— 同一個值的 host 可讀唯讀狀態（`private set`），
  比照既有 `widgetColor` / `widgetBgcolor`，於 carousel / grid 載入成功後更新。floating
  （`/sdk/widget/live`）不帶此欄，維持 `null`。
- **Widget 輪播卡依 `product_card` 渲染三態**（`CarouselCardView`）——`inside`（維持既有縮圖內
  dark-glass 疊層，像素不變）/ `below`（商品卡移到縮圖外，落在**標題之下、卡片最底**；未綁商品時
  渲染等高透明佔位維持同列同格等高）/ `hidden`（完全不畫，不留佔位）。缺值或白名單外字串一律
  fallback 為 `inside`。
- **Widget 表面顏色接上 `widget_color` / `widget_bgcolor`**——`CarouselView` / `ScrollableCarouselView` /
  `VideoShopGridView` 三個 widget 表面依後台設定衍生文字與背景色：`widget_color == 2` 時文字轉
  `#FFFFFF`（`1` 不覆寫）；`widget_bgcolor` 為合法 hex 時覆寫背景（空字串 / `null` / 缺 key 視同
  不覆寫）。未設定時渲染與現況不變。僅套用於這三個 widget 表面，不影響全域主題。
- **商品明細 / 快速購買 sheet 新增庫存文案開關** `LivebuyPlayerConfig.showStock`（DEFAULT
  `true`）——`false` 時「只剩庫存 N 組」整行不畫、不留佔位，既有「售完不顯示」閘不變（AND 關係）。
- **PlayerHeader 標題新增跑馬燈開關** `LivebuyPlayerConfig.titleScroll`（DEFAULT `true`）——是否
  捲動仍 100% 由內容是否覆蓋容器的量測決定，`titleScroll` 是疊加在量測之上的後端能力閘（AND
  關係），`false` 時維持既有單行省略顯示、行高不變（含直播預告 upcoming 分支）。
- **浮動直播入口新增初始落點與延遲出現時機**（`LivebuyLiveEntryConfig`）——`position`（`null` →
  右下，既有落點）/ `timing`（`null` → 立即，既有時機）/ `delaySeconds`（DEFAULT `3`）。`timing
  == "delay"` 時延遲指定秒數才出現並播進場動畫；`immediate` 維持現況、零行為變動。可拖曳與不可
  拖曳兩種模式皆適用。

### Fixed

- **`widget_color` 為不可解析字串或非數值型態時，不再讓整份影片清單消失。** 該欄宣告型別是 Int，
  但後端 widget-group override 路徑是逐字賦值、沒有整數轉型。Gson 的預設 `Integer` adapter 遇
  `"abc"` / `"2.7"` / `true` / `{}` 會在 `fromJson` 當下拋 `JsonSyntaxException`，早於
  `WidgetResponseMapper` 執行 → 例外冒到呼叫端，**整個 widget 影片清單消失**。現改掛 field-scope
  `LenientNullableIntDeserializer`：Int 與數字字串（`"2"`）皆正確解析，其餘一律落預設 `1` 且不
  拋錯。**API 面零變化**（`widgetColor` 仍是 `Int`）。
- **`sdkConfig.extensions` / `layout.player` / `layout.widget` 巢狀值正規化為純 Kotlin 型別**，
  不再洩漏 `org.json` 內部表示（`org.json.JSONObject` → `Map`、`org.json.JSONArray` → `List`、
  `JSONObject.NULL` → Kotlin `null`）。此前巢狀值（如後端新增的 `extensions.floating_app`）會
  原封不動帶著 `org.json` 內部型別流到下游——RN Android bridge 因型別不是 `Map` 而序列化成 JSON
  字串（同一份 JS 程式碼在 iOS/Android 行為不同）、`JSONObject.NULL` 不是 Kotlin `null` 導致既有
  `filterValues { it != null }` 濾不掉、Flutter `StandardMethodCodec` 不支援 `org.json` 型別而
  出錯。純量值（此前一直存在的情況）行為不變。**API 面零變化**（`extensions` 仍是
  `Map<String, Any?>`）。
- **浮動直播入口關閉鈕改對齊現行設計稿。** 舊造型抄自一個已於 2026-06-09 移除的設計元件（框外
  24dp 深色玻璃 + 白描邊），現改為框內右上 20dp、`rgba(0,0,0,0.55)`、無描邊（對齊現行
  `sdk-components.jsx:LBPFloatingWidget`）。卡片本身尺寸與位置不變，只有關閉鈕像素改變。
- **In-app browser（聯絡商家 / 中獎領獎頁尾法務連結）改開在呼叫端 task。** 先前 Custom Tabs 被
  無條件加上 `FLAG_ACTIVITY_NEW_TASK`（測試環境遺留的保底邏輯被固化進正式行為），使用者點擊後
  落在另一個 task——recents 出現兩張卡、返回鍵回不到播放器，體感等同跳出 App。現在能解析出
  host Activity 時不再加此 flag（同 task 開啟）；解析不到才維持加上（避免非 Activity context
  拋 `AndroidRuntimeException`）。

> **平台範圍**：iOS + Android 兩端本輪皆完整落地（parity change，見 iOS CHANGELOG）。
> React Native / Flutter 對應能力已在主線，隨各自待發 `2.0.0` 出貨。

---

## [4.4.0] - 2026-07-31

> **Minor — no source-breaking change, source-compatible (one behavior BREAKING, see `Changed`).**
> Version aligned to the iOS SDK `v4.4.0` (cross-platform lockstep) and feature-equivalent on the
> shared changes. Ships the URL-open policy + host routing (including two real Android security
> bypass fixes), guest-nickname verification hardening, and clickable win-claim modal footer links.
> The internal `versionName` (`X-SDK-Version`, `1.3.0`) is unchanged. 詳見
> [`docs/release/v4.4.0-tag-runbook.md`](../docs/release/v4.4.0-tag-runbook.md) 與
> [release notes](../docs/release-notes/v4.4.0.md)。

### Changed（⚠️ 行為變更 — 唯一「不改簽章但行為會變」的一條，請先讀）

- **⚠️ 商品導購頁連結（`diversion == 1`，host 未攔截時）改依網址分流**（`a15163a5`，parity iOS
  `5457c97e`；消費 core `d408b23a` 新增的 `LBURLOpenPolicy.decide(rawUrl:)`）。**規則**：
  `livebuy.tv`（含任意層子網域）→ 維持 in-app（Chrome Custom Tabs）；其他可開網址
  （`http`/`https`/`mailto`/`tel`/`sms`）→ 系統瀏覽器；不在允許清單內（`javascript:` /
  `intent:` / `data:` / `file:` / 自訂 scheme 等）→ 安全 no-op（先前無 scheme 過濾，可能被原樣
  丟進 `CustomTabsIntent`）。**典型後果**：非 `livebuy.tv` 網域的商品導購頁會從 Custom Tabs 改
  為 eject 到系統瀏覽器——這是刻意的行為變更，不是 regression。host 攔截順序逐字不變，策略只在
  host 未攔截時套用。**API 面零破壞**：`DefaultPlayerTemplate` 建構子本來就是 `internal`，host
  從未能注入 opener，相容性只涉及 `livebuy-ui` 模組內與其測試。Android 本輪只有商品導購頁這一個
  出口（客服連結呈現在 reference-ui 層，本輪未動）。

### Added（新公開面，皆 additive、源碼相容、無 breaking）

- **`tv.livebuy.sdk.core.LBURLOpenPolicy.decide(rawUrl: String)` / `LBURLOpenTarget` /
  `LBLegalLinks.TERMS_OF_USE` / `.PRIVACY_POLICY`**（`d408b23a`，parity iOS `dcea410f`）——純函式
  URL 開啟目標裁決規則與法務連結網址事實來源。對 host 而言是**可選用的新增 API**（不呼叫即無任何
  行為變化）；SDK 內部的消費點隨本版一起出貨，見上方 `Changed`（view-model 層 `a15163a5`）與下方
  `Fixed / drop-in behavior`（reference-ui 層 `fc278b0b`）。🔴 **一併
  修掉兩個真實繞過**：① `intent:` scheme 先前未過濾，`diversionUrl` 這類後端可控字串可被原樣送進
  `CustomTabsIntent`（公認可觸達任意 exported component 的向量），新策略的正面 scheme 允許清單在
  呈現之前就擋下它；② API ≤ 26（`minSdk` 是 24，實際出貨環境）`android.net.Uri` 以 authority 內
  **第一個** `@` 切 userinfo 的解析差異，會把 authority 中間片段誤判為 host（例如
  `https://a@develop.livebuy.tv:8443@attacker.example/x` 誤判 host 為 `develop.livebuy.tv`，
  真實連線目標其實是 `attacker.example`），修法為 in-app 授予加上第二個連言條件（解析器回報的
  host 須與 authority 依 WHATWG 推導的 host 一致）。host 不需要做任何事，兩者皆在 SDK 內部判定
  完成。
- **`LivebuyPlayerView.suspend fun setGuestNicknameVerified(name: String)`**（`6cc0f973`，parity
  iOS `81a76425`）——設定留言暱稱前先對目前 video 呼叫既有 `checkName` 驗證，通過才持久化 + 廣播
  `AUTH_STATE_CHANGED`；被取走或其他錯誤一律不持久化、不廣播，拋出可分辨的 `LBError`（複用既有
  `GuestNameTaken` / `NetworkError` 等分類，不新增 case）。既有 `LivebuySDK.setGuestNickname(name)`
  （同步、無驗證）簽章與行為不變。
- **`setGuestNicknameVerified` 不再有靜默成功路徑**（`4f5b4adf`，parity iOS `12c894a4`）——先前
  有三道 `return` 會在完全沒有提交暱稱的情況下正常返回（名稱 trim 後為空 / SDK 未 configure /
  播放器未載入影片）。現在一律 `throw`：SDK 未 configure → 複用既有 `LBError.NotConfigured`；
  名稱為空或無影片 → **新增** `LBError.NicknameSetPreconditionFailed`（`object`），
  `LivebuySDK.errorCode()` 補上 `"nickname_set_precondition_failed"` 映射（與 iOS 字串逐字一致）。
  public 簽章不變（本來就是 `suspend fun`），成功路徑與 `checkName` 失敗映射完全不變。⚠️
  `LBError` 是 `sealed class` 且既有 `when` 分支已有 `else`，新增此 `object` **不會**讓既有程式碼
  編譯失敗（與 iOS Swift exhaustive switch 不同）；若你的 `when` 對 `LBError` 做 exhaustive 處理
  且沒有 `else`，需留意新增這個 case。

### Fixed / drop-in behavior（reference-ui + turnkey，drop-in `LivebuyPlayer` 使用者自動生效）

- **中獎領獎 sheet 底部使用條款／隱私政策改為可點擊**（`fc278b0b`，parity iOS `55ef0e8d`）——先前
  是單一組合字串、純版面佔位；現在拆成兩段各自可點擊，經 `LBURLOpenPolicy.decide()` 裁決開啟方式
  （Chrome Custom Tabs / 系統瀏覽器 / 安全 no-op），連結來源為 `LBLegalLinks`。兩個開啟入口皆接進
  既有的外送啟動 PiP 抑制閘（見下方 `f608b9fb`）。8 張 Roborazzi baseline 因 footer 拆成獨立可點
  擊區塊產生的次像素捨入差異已重新產生並驗證 `changed:0`。
- **暱稱被佔用時就地顯示錯誤、不關閉 modal**（`bccfd0f8`，parity iOS `fad82c7f`）——暱稱設定 modal
  送出改走 `setGuestNicknameVerified`，只有驗證成功才關閉；被取走或其他錯誤則以 inline 錯誤留在
  modal 內讓使用者改名重試。`onSubmit` 簽章維持 `(String) -> Unit` 不變，既有呼叫端零改動。一併
  修掉一個併發世代缺失（送出→取消→改點加入活動會讓舊請求對新一次呈現送出 join、關掉正在輸入的
  modal，加 `presentationGeneration` gate 堵住）。
- **送出中 CTA 保留品牌填色，補上四端最後一格交集態**（`6836633b`，parity iOS/Flutter/RN 既有
  行為）——送出中若使用者中途清空輸入框，先前 Android 的 CTA 會因 `canSubmit` 轉假而退成灰色
  disabled 面。改為填色 / 標籤色吃 `submitting || canSubmit`，互動仍鎖在 `canSubmit &&
  !submitting`。
- **中獎領獎 sheet 的 scrim 全 stage 吃掉點擊**（`3f7aeb3d`）——`claim` / `submitting` / `fail`
  階段的 scrim 先前只在 `done` 階段才掛觸控攔截，其餘階段的點擊會穿透到底下播放器的靜音手勢層。
  現在任何 stage 皆無條件吸收觸控，是否觸發 dismiss 仍只在 `done` 階段生效。
- **分享 / 服務連結 / 外部直播卡不再誤觸發 PiP**（`f608b9fb`）——先前跳出分享 chooser、Custom
  Tabs、或外部直播卡時，容器會誤判為「使用者離開」而自動進入 PiP。新增外送啟動抑制閘：主動啟動
  外部 Activity 期間暫停三條 PiP 進入路徑，回前景後依「SDK 是否真的武裝過這個 Activity」的追蹤
  結果復原，涵蓋 host Activity 在復原前被系統銷毀的情境。

### Notes

- **未新增 / 移除 / 改名任何既有 host-facing public 符號**、無參數型別變更、無 wire 破壞。本版
  唯一需要 host 留意的是上方 `Changed` 小節的商品導購頁開啟方式（Android 本輪影響面小於 iOS——
  iOS 另有客服連結出口，Android 沒有）。
- **發佈通道**：Pages `maven-metadata.xml` 為**累加**語意——本版上線後**九版並存**
  （3.1.3 / 3.2.0 / 3.2.1 / 3.2.2 / 4.0.0 / 4.1.0 / 4.2.0 / 4.3.0 / 4.4.0），舊版仍可解析。
- **RN / Flutter 本輪不發**（停在待發 `2.0.0`）；本版全部主題於其主線皆已落地，隨各自 2.0.0 出貨。

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
