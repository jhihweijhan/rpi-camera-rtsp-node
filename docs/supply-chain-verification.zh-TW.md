# 供應鏈自我驗證清單

<!-- README-I18N:START -->

**繁體中文**

<!-- README-I18N:END -->

這份文件提供導入方自我驗證硬體供應鏈的檢查方向。它不是推薦 BOM、不是產地保證、
不是政府採購或法遵背書。

`rpi-camera-rtsp-node` 只提供 Raspberry Pi CSI camera RTSP node 的軟體與部署方向。
實際硬體料號、採購來源、產地文件、驗收紀錄與合規責任，必須由導入方自行確認。

## 使用時機

在以下情境使用這份清單：

- 商業或公部門場域需要降低中國製攝影設備進入內網的風險。
- SI、弱電、安防整合商需要替客戶整理可說明的 RTSP source 部署方式。
- 導入方打算從 RS、DigiKey、Mouser 或其他標準通路採購 Raspberry Pi、CSI camera、
  電源、外殼、線材與周邊。

## 不要把這份文件當成什麼

- 不是非中國製 BOM。
- 不是產地證明。
- 不是合規意見。
- 不是採購建議書。
- 不是保證所有 Raspberry Pi 或 camera module 都符合你的採購規範。

## 自我驗證流程

### 1. 固定採購料號

每個硬體項目都應記錄：

- 品名。
- 製造商。
- distributor。
- distributor part number。
- manufacturer part number。
- 訂購頁 URL。
- 訂購日期。
- 數量。

不要只記「Raspberry Pi Zero 2 W」或「CSI camera」。同名產品可能有不同包裝、
revision、來源或替代料。

### 2. 保存採購頁與產地資訊

採購前保存：

- distributor 商品頁截圖或 PDF。
- country of origin 欄位，如果頁面有提供。
- datasheet 或 product brief。
- manufacturer 頁面。
- 報價單或購物車明細。

若頁面沒有 country of origin，不要自行推論。請向 distributor 或供應商確認。

### 3. 收貨驗證

收貨時檢查並拍照保存：

- 外盒標籤。
- 產品標籤。
- country-of-origin 標示。
- 批號、序號、revision。
- 與採購料號是否一致。

若收貨品與採購頁不一致，先暫停導入，不要直接部署到客戶或內網。

### 4. 建立專案 BOM 紀錄

每次導入至少保存一份 BOM：

```text
Item | Manufacturer | MPN | Distributor | Distributor PN | Qty | Claimed origin | Evidence path | Checked by | Date
```

`Claimed origin` 只填有文件或標籤支持的資訊。沒有證據就填 `unknown`。

### 5. 分離軟體與硬體責任

本專案能提供：

- Node binary。
- MediaMTX-based RTSP/WebRTC/HLS streaming。
- install script。
- access control options。
- troubleshooting docs。

導入方必須自行負責：

- 硬體採購。
- 產地驗證。
- 客戶採購規範。
- 網路隔離。
- 實體安裝。
- 後續保固與維護。

### 6. 不要公開敏感部署資訊

不要在 GitHub issue 貼出：

- 攝影機位置。
- 內部網段。
- RTSP URL。
- 帳號密碼。
- 客戶名稱。
- 標案未公開資訊。
- NDA 內容。

商用授權與敏感導入問題請寄至：<jhihweijhan@gmail.com>

## 最小驗收問題

導入前至少回答：

1. 每個硬體項目是否有固定料號？
2. 是否保存採購頁、報價單或訂單紀錄？
3. 是否有可查驗的 country-of-origin 證據？
4. 收貨標籤是否與採購紀錄一致？
5. 是否有 BOM 表可交接？
6. 是否避免把敏感部署資訊公開到 GitHub？
7. 客戶是否理解本專案不保證完整 BOM 產地？

如果任何答案是「否」或「不知道」，請先完成驗證，再進行商業或公部門導入。
