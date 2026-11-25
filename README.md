# 時光足跡 Time Footprint（timelapse-footprint）

手機瀏覽器上的路線日誌原型。單一 HTML、沒有後端、沒有建置、沒有 API 金鑰。GitHub Pages 從 `main` 根目錄提供 `index.html`：https://akila33-jpg.github.io/timelapse-footprint/

產品方向：ChengChi。若 Google Drive `00_AI_Memory_Hub/05_TIME_FOOTPRINT_時光足跡` 另有產品決定，以那邊為準；這個儲存庫的程式以這裡的版本檔為準。

## 模組

| 模組 | 版本 | 狀態 | 負責 |
| --- | --- | --- | --- |
| 軌跡記錄（開始／停止、統計、精度與移動過濾、localStorage） | v0.1 | 已完成 | Claude |
| 地圖（Leaflet + OpenStreetMap） | v0.1 | 已完成 | Claude |
| GPX 軌跡匯出 | v0.1 | 已完成 | Claude |
| 拍照（即時取景；失敗時改系統相機） | v0.2 | 測試中 | 程式工程師 Bot |
| 照片、座標與時間（IndexedDB） | v0.2 | 測試中 | 程式工程師 Bot |
| 地圖照片縮圖與檢視 | v0.2 | 測試中 | 程式工程師 Bot |
| 定位關閉／一般／省電 | v0.2 | 測試中 | 程式工程師 Bot |
| GPX 照片航點（`<wpt>`） | v0.2 | 測試中 | 程式工程師 Bot |
| 相機式介面（繁體中文、台灣用詞） | v0.2 | 測試中 | 程式工程師 Bot |
| 復古濾鏡與時間戳水印 | v0.3 | 規劃中 | — |
| 路線日誌頁 | v0.4 | 規劃中 | — |

v0.2 已在無頭 Chrome（模擬定位與相機、iPhone 視窗 430×932）測過。實機，尤其 iOS Safari，仍是測試中。

## 檔案

| 檔案 | 說明 |
| --- | --- |
| `index.html` | 現行程式（v0.2）。Pages 直接開這個檔。 |
| `v0.2.html` | 與 `index.html` 相同的複本，給之後的版本對照。 |
| `v0.1.html` | 凍結。不要改。 |
| `CHANGELOG.md` | 版本差異。 |

改 `index.html` 之後要立刻 `cp index.html v0.2.html`，兩個檔必須相同。

## 怎麼接手

下一個 AI 或工程師維持單檔架構即可，不要加建置步驟、後端或付費服務。這個版本不使用 API 金鑰；以後若真的需要，從設定讀（例如另一個 localStorage 設定），不要寫進程式、不要提交金鑰。

本地預覽要有 secure context，相機和定位在 `file://` 不會工作：

```bash
python3 -m http.server 8765
```

然後開 http://127.0.0.1:8765/ 。

### 資料，不要弄丟

- **軌跡** `localStorage['tf_track_v01']`：`{ track: [{ lat, lng, t, acc }], accumulatedMs }`。這是 v0.1 的格式。不要改 key，不要拿掉這兩個欄位。可以留著點上多出來的欄位。清除軌跡只刪這個 key，不刪照片。
- **設定** `localStorage['tf_settings_v02']`：`{ gpsMode: "off" | "normal" | "power" }`。預設 `normal`。
- **照片** IndexedDB `tf_photos_v02`，version 1，object store `photos`，`keyPath: "id"`。每筆 `{ id, t, lat, lng, acc, coordNote, blob }`。`blob` 是長邊最多 1600px 的 JPEG。有座標時 `coordNote` 是 `null`；否則是 `off`、`weak`、`denied`、`timeout`、`unavailable`。升版時在 `onupgradeneeded` 加東西，不要刪掉這個 store。

### 定位模式

- **關閉**：不呼叫 `watchPosition` / `getCurrentPosition`。記錄可以繼續計時，但不會新增軌跡點。新照片沒有座標，畫面要寫明「無座標（定位已關閉）」。
- **一般**：`enableHighAccuracy: true`、`maximumAge: 2000`、`timeout: 20000`。誤差大於 50 公尺丟掉，移動少於 5 公尺不記新點。記錄中嘗試螢幕恆亮（Wake Lock）。
- **省電**：`enableHighAccuracy: false`、`maximumAge: 30000`、`timeout: 30000`。移動少於 20 公尺不記新點。不使用 Wake Lock。誤差大於 50 公尺的點仍然丟掉，所以裝置只給很粗的網路定位時，軌跡可能是空的、照片會標成訊號不足。

照片附座標的規則：模式是關閉就沒有座標。否則若 20 秒內（省電為 60 秒）有一筆誤差 ≤ 50 公尺的定位就用它；沒有才 `getCurrentPosition`，而且 `maximumAge: 0`。時間戳是按下快門的時間，不是定位回來的時間。

### 畫面

兩個畫面：相機、地圖。相機是黑底、底部置中的白色圓形快門，地圖與設定是小圖示。文案用繁體中文（台灣）。即時預覽優先用 `getUserMedia`（`playsinline`、後鏡頭）。失敗時，快門在同一個點擊裡打開 `<input type="file" accept="image/*" capture="environment">`，不要先 `await` 再打開，否則 iOS 會吃掉使用者手勢。「開啟相機」是另一次手勢，用來再試即時預覽。

地圖照片只標有座標的點。點縮圖開全螢幕，顯示時間和座標。沒有座標的照片在底片牆用「無座標」標出來。

GPX 仍是軌跡 `<trkpt>`，另外為有座標的照片加 `<wpt>`。

### 不要做的事

- 不要改 `v0.1.html`。
- 不要把 v0.3 的濾鏡、水印或 v0.4 的日誌頁先做進來，除非任務明確要求。
- 不要接需要金鑰或付費的圖磚、辨識或雲端空間。圖磚維持 OpenStreetMap，Leaflet 維持目前的 CDN。
- 不要為了測試把精度門檻改掉。無頭瀏覽器給的定位誤差常常大於 50 公尺，測試要自己把模擬定位的 `accuracy` 設在 50 以內。

### 實機限制（寫給下一個 AI）

- iOS Safari：第一次 `getUserMedia` 若沒有使用者手勢可能失敗；快門的檔案輸入是備援。背景分頁會暫停定位。沒有 Wake Lock。私密瀏覽的 IndexedDB 可能失敗。HEIC 可能無法畫進 canvas，系統相機的 `capture` 通常給 JPEG。
- 必須 https 或 localhost。
- 省電模式不代表一定省電成功：瀏覽器沒有真正的 distance filter，20 公尺是程式自己濾的；`enableHighAccuracy: false` 仍可能回傳很粗的點，然後被 50 公尺規則丟掉。
