<div align="center">
  <img src="./docs/assets/rpi-camera-rtsp-node.svg" width="112" alt="rpi-camera-rtsp-node icon">
  <h1>rpi-camera-rtsp-node</h1>
  <p><strong>把 Raspberry Pi CSI camera 變成可控、可採購驗證的 RTSP 節點。</strong></p>
  <p>
    <a href="https://github.com/jhihweijhan/rpi-camera-rtsp-node/releases/latest"><img alt="Latest release" src="https://img.shields.io/github/v/release/jhihweijhan/rpi-camera-rtsp-node?style=flat-square"></a>
    <img alt="Raspberry Pi" src="https://img.shields.io/badge/Raspberry%20Pi-Zero%202%20W-c51a4a?style=flat-square&logo=raspberrypi&logoColor=white">
    <img alt="Streams" src="https://img.shields.io/badge/RTSP%20%2F%20WebRTC%20%2F%20HLS-H.264-2f855a?style=flat-square">
    <a href="./LICENSE"><img alt="License" src="https://img.shields.io/badge/license-non--commercial-blue?style=flat-square"></a>
  </p>
  <p><strong>繁體中文</strong> | <a href="./README.en.md">English</a></p>
</div>

`rpi-camera-rtsp-node` 是給 SI、弱電、安防整合商與公部門/商用評估使用的
Raspberry Pi camera node。它把 Raspberry Pi Zero / Zero 2 W + CSI camera
變成自架的 RTSP / WebRTC / HLS source，讓 go2rtc、NVR、Frigate、YOLO v10+
或其他 AI/CV 主機在乾淨的串流入口上工作。

> [!IMPORTANT]
> 本專案不是完整商用 IP camera，也不保證任何使用者的完整 BOM 皆為非中國製。
> 它提供可重建的軟體節點、明確的供應鏈驗證方向，以及商用授權入口。

[快速開始](#快速開始) • [適合場景](#適合場景) • [架構](#架構)
• [供應鏈](#供應鏈與採購邊界) • [商用授權](#商用授權與聯絡)
• [Q&A](#qa-與疑難排解)

## 你得到什麼

- **硬體 H.264 source**：Pi 只負責 capture、hardware encode、serve。
- **多協議輸出**：單一路徑輸出 RTSP、WebRTC、HLS。
- **低資源節點**：目標硬體是 Raspberry Pi Zero 2 W，也保留 Pi Zero constrained profile。
- **Consumer-side AI/NVR**：YOLO、錄影、告警、轉發與中央管理放在有算力的主機。
- **供應鏈意識**：可搭配 RS、DigiKey、Mouser 等標準通路做導入方自我驗證。

## 快速開始

在 Raspberry Pi 上執行：

```bash
curl -fsSL https://github.com/jhihweijhan/rpi-camera-rtsp-node/releases/latest/download/install.sh | \
  bash -s -- \
    --read-username viewer \
    --read-password '<your-password>'
```

安裝器會部署 Node binary、bundled MediaMTX、設定檔與 `systemd` service。
請自行設定強密碼；installer 不會自動產生或公開列印串流密碼。

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

## 適合場景

| 場景 | 本專案提供的價值 |
| --- | --- |
| SI / 弱電 / 安防整合 | 可說明、可重建、可交給 NVR/AI 主機的 RTSP source |
| 商業與公部門評估 | 降低黑盒攝影設備進入內網的風險，保留採購驗證空間 |
| AI/NVR pipeline | 給 go2rtc、Frigate、YOLO 或自建 CV pipeline 穩定輸入 |
| 家庭與非營利用途 | 低成本、自架、可重新安裝的 Raspberry Pi camera node |

不適合需要完整保固、PoE 外殼、雲端管理、廠商 SLA，或已交付中央管理平台的場景。
Fleet Layer / 中央管理是 roadmap，不是目前已交付功能。

## 架構

MediaMTX 負責 media plane。Node Agent 只負責產生設定、管理 service、
發布 mDNS、檢查 camera 狀態。它不讀取、不轉碼、不處理 video frames。

```text
CSI camera
   |
   v
Raspberry Pi Node
   |  hardware H.264 encode
   v
RTSP / WebRTC / HLS
   |
   +--> go2rtc / NVR / Frigate
   +--> YOLO / AI-CV host
   +--> internal dashboard
```

go2rtc 範例：

```yaml
streams:
  rpi_camera:
    - rtsp://viewer:<read-password>@<node-host>:8554/cam
```

## 供應鏈與採購邊界

本專案可以配合導入方從 RS、DigiKey、Mouser 或其他標準通路採購硬體，
並以固定料號、收貨標示、country-of-origin 證據與 BOM 紀錄做自我驗證。

> [!NOTE]
> 硬體產地、料號、收貨標示、採購文件與 BOM 合規性，必須由導入方依實際採購項目自行確認。
> 本專案不提供政府採購、資安法遵或產地合規背書。

導入前請閱讀：[供應鏈自我驗證清單](./docs/supply-chain-verification.zh-TW.md)。

## 支援硬體

| Profile | Hardware | Stream profile | 用途 |
| --- | --- | --- | --- |
| Reference | Raspberry Pi Zero 2 W, 64-bit OS | H.264 1280x720 約 30 fps | release acceptance 參考目標 |
| Constrained | Original Pi Zero / Zero W, ARMv6 | H.264 640x480 約 15 fps | 低資源支援 profile |
| Integration proxy | Pi 3B | 同一驗證流程，非 reference | 日常整合測試 |

需要 CSI camera，且 OS 必須能透過 `rpicam-hello --list-cameras` 或
`libcamera-hello --list-cameras` 列出 camera。

## 商用授權與聯絡

個人、家庭、非營利教育與其他非商業用途可免費使用。商業導入、政府標案、
SI/弱電/安防整合、OEM/再散布、企業內部營運使用，請先取得商用授權。

- 非敏感的一般問題：
  [Commercial inquiry](https://github.com/jhihweijhan/rpi-camera-rtsp-node/issues/new?template=commercial-license.yml)
- 報價、採購、NDA、案場拓撲、客戶資訊或部署細節：
  <jhihweijhan@gmail.com>

> [!WARNING]
> 請勿在公開 GitHub issue 貼出憑證、RTSP URL、內部網段、攝影機位置、
> 客戶名稱、標案未公開資訊或 NDA 內容。

## Q&A 與疑難排解

- [繁體中文 Q&A](./docs/question-and-answer.zh-TW.md)
- [English Q&A](./docs/question-and-answer.md)

常用診斷命令：

```bash
rpicam-hello --list-cameras
sudo systemctl status rpi-camera-mediamtx.service
```

<sub>Keywords: Raspberry Pi RTSP camera, Raspberry Pi Zero 2 W CSI camera,
hardware H.264, go2rtc source, YOLO RTSP source, NVR RTSP,
supply-chain-conscious camera node, non-commercial freeware.</sub>
