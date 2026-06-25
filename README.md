# rpi-camera-rtsp-node

<!-- README-I18N:START -->

**繁體中文** | [English](./README.en.md)

<!-- README-I18N:END -->

## 商用與公部門影像系統的可控 RTSP 節點

`rpi-camera-rtsp-node` 將 Raspberry Pi Zero / Zero 2 W 加上 CSI camera
變成一個可自架、可控、供應鏈意識明確的 RTSP / WebRTC / HLS 影像節點。

它的核心用途不是取代完整商用安防攝影機，而是提供一個乾淨、硬體 H.264、
可長期部署的 RTSP source，交給 go2rtc、NVR、Frigate、YOLO v10+ 或其他
AI/CV 主機處理。

Pi 節點只做三件事：**capture、hardware encode、serve**。所有推論、錄影、
告警、轉發和中央管理都應在 Consumer 端完成。

## 適合誰

- SI、弱電、安防整合商，需要可說明、可自架的 RTSP source。
- 商業或公部門影像系統，需要降低中國製攝影設備進入內網的風險。
- AI/NVR 專案，需要把現場 CSI camera 送到 go2rtc、NVR 或 YOLO 主機。
- 家庭、非營利教育與非營利場景，需要低成本、可重建的 Raspberry Pi camera node。

## 不適合誰

- 需要完整保固、雲端管理、PoE 外殼與廠商 SLA 的商用 IP camera。
- 期待本專案保證完整 BOM 產地、法遵認證或採購合規背書。
- 需要中央管理平台已經可用。本專案目前是單節點，Fleet Layer 是 roadmap。

## 一行安裝

在 Raspberry Pi 上執行：

```bash
curl -fsSL https://github.com/jhihweijhan/rpi-camera-rtsp-node/releases/latest/download/install.sh | \
  bash -s -- \
    --read-username viewer \
    --read-password '<your-password>'
```

安裝器會部署 Node binary、bundled MediaMTX、設定檔與 `systemd` service。
請自行設定強密碼；installer 不會自動產生或公開列印串流密碼。

## 串流端點

預設 stream path 是 `cam`：

```text
RTSP:   rtsp://<user>:<pass>@<pi-host>.local:8554/cam
WebRTC: http://<pi-host>.local:8889/cam/
HLS:    http://<pi-host>.local:8888/cam/index.m3u8
```

從另一台電腦測試 RTSP：

```bash
ffplay "rtsp://viewer:<read-password>@<node-host>:8554/cam"
```

## 為什麼不是一般 IP Camera

一般低價 IP camera 常把韌體、雲端、供應鏈、資安責任包在黑盒裡。
本專案把節點拆成可理解、可重建的最小單元：

- Raspberry Pi + CSI camera：現場 capture。
- MediaMTX `rpiCamera` source：硬體 H.264 encode，一次 encode，多協議輸出。
- Node Agent：產生設定、管理 service、發布 mDNS、回報狀態。
- Consumer：go2rtc、NVR、YOLO、Frigate 或其他 AI/CV 主機。

```text
CSI camera -> Raspberry Pi Node -> RTSP/WebRTC/HLS -> go2rtc / NVR / YOLO host
                  |                    |
                  |                    +-- read credentials + optional IP allowlist
                  +-- hardware H.264 encode, no Python video frame path
```

## 供應鏈與採購責任邊界

本專案提供供應鏈意識明確的部署方向，可搭配 RS、DigiKey、Mouser 等標準通路
採購與導入方驗證流程。硬體產地、料號、收貨標示、採購文件與 BOM 合規性，
必須由導入方依實際採購項目自行確認。

本專案不保證任何使用者的完整 BOM 皆為非中國製，也不提供政府採購、資安法遵
或產地合規背書。

導入前請閱讀：

- [供應鏈自我驗證清單](./docs/supply-chain-verification.zh-TW.md)

## 商用授權

個人、家庭、非營利教育與其他非商業用途可免費使用。商業導入、政府標案、
SI/弱電/安防整合、OEM/再散布、企業內部營運使用，請先取得商用授權。

商用授權與導入詢問：

- 非敏感的一般問題：請使用
  [Commercial inquiry](https://github.com/jhihweijhan/rpi-camera-rtsp-node/issues/new?template=commercial-license.yml)。
- 報價、採購、NDA、案場拓撲、客戶資訊或部署細節：請寄至
  <jhihweijhan@gmail.com>。

請勿在公開 GitHub issue 貼出憑證、RTSP URL、內部網段、攝影機位置、
客戶名稱、標案未公開資訊或 NDA 內容。

## 架構：Node 只負責 capture / hardware encode / serve

Node 的 media plane 由 MediaMTX 負責。Node Agent 只負責控制與 provisioning：
產生 MediaMTX config、管理 service、發布 mDNS、檢查 camera 狀態。

Node Agent 不讀取、不轉碼、不處理 video frames。這是設計邊界。

## 接到 go2rtc、NVR、YOLO

go2rtc 範例：

```yaml
# go2rtc.yaml
streams:
  rpi_camera:
    - rtsp://viewer:<read-password>@<node-host>:8554/cam
```

go2rtc 可再轉發 WebRTC/HLS/MJPEG 給 dashboard、NVR 或 AI pipeline。
YOLO、錄影、告警與中央分析應在較有算力的 Consumer 主機上執行，不要放在 Pi Zero。

## 支援硬體與 profile

| Profile | Hardware | Stream profile | 用途 |
| --- | --- | --- | --- |
| Reference | Raspberry Pi Zero 2 W, 64-bit OS | H.264 1280x720 約 30 fps | release acceptance 參考目標 |
| Constrained | Original Pi Zero / Zero W, ARMv6 | H.264 640x480 約 15 fps | 低資源支援 profile |
| Integration proxy | Pi 3B | 同一驗證流程，非 reference | 日常整合測試 |

需要 CSI camera，且 OS 必須能透過 `rpicam-hello --list-cameras` 或
`libcamera-hello --list-cameras` 列出 camera。

## Roadmap：Fleet Layer / 中央管理

目前每個 Node 都是自包含單節點，不依賴中央 coordinator。

未來方向是 Consumer-side Fleet Layer：由有算力的主機管理多個 Node、
接收 RTSP、執行 YOLO 類推論、提供中央 view。這不是目前已交付功能。

## Q&A 與疑難排解

安裝、密碼、跨電腦測試、Pi 3B `Exec format error`、CSI camera 偵測失敗、
OV5647 case study 請看：

- [繁體中文 Q&A](./docs/question-and-answer.zh-TW.md)
- [English Q&A](./docs/question-and-answer.md)

最常用診斷命令：

```bash
rpicam-hello --list-cameras
sudo systemctl status rpi-camera-mediamtx.service
```

## License

Proprietary, **non-commercial** freeware. Free for personal, household,
non-profit educational, and other non-commercial use. Commercial use requires
a separate written commercial license.
See [LICENSE](./LICENSE). Bundled MediaMTX remains governed by its own MIT license.

---

<sub>Keywords: Raspberry Pi RTSP camera, Raspberry Pi Zero 2 W CSI camera, hardware H.264, go2rtc source, YOLO RTSP source, NVR RTSP, supply-chain-conscious camera node, non-commercial freeware.</sub>
