# 變更紀錄

## v0.2

- 相機與地圖兩個畫面。黑底、底部置中白色快門，繁體中文（台灣）。
- 瀏覽器內拍照（`getUserMedia` 即時預覽）。打不開時，快門改走系統相機（`input capture="environment"`）。
- 每張照片記下按下快門的時間，以及當時的定位（沒有定位就標成無座標）。JPEG 長邊約 1600px，存在 IndexedDB。
- 有座標的照片在地圖上顯示成小縮圖，點開可看時間與座標。
- 定位可切換：關閉、一般、省電。設定存在 `localStorage`（`tf_settings_v02`）。
- GPX 除了軌跡，還會帶有座標照片的 `<wpt>`。
- 沿用 v0.1 的軌跡 key `tf_track_v01`，舊資料直接讀得回來。`v0.1.html` 沒有改。

## v0.1

- 前景 GPS 軌跡、Leaflet 與 OpenStreetMap、`watchPosition`。
- 誤差大於 50 公尺不記，移動少於 5 公尺不記。
- 距離、時間、點數；軌跡存在 `localStorage`；匯出 GPX；記錄時嘗試螢幕恆亮。
