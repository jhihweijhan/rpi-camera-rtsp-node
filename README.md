# rpi-camera-rtsp-node

**Turn a Raspberry Pi Zero + CSI camera into a low-latency RTSP / WebRTC streaming node — hardware H.264, go2rtc & YOLO-ready, one-line install.**

`rpi-camera-rtsp-node` is a tiny, self-contained **streaming node** for the **Raspberry Pi Zero** (reference: **Zero 2 W**). Point it at a **CSI camera**, run one command, and it serves a hardware-encoded **H.264** video stream over **RTSP** (plus WebRTC and HLS) that any third party can pull and process:

- 🔌 **go2rtc** — add the node's RTSP URL as a source ([AlexxIT/go2rtc](https://github.com/AlexxIT/go2rtc))
- 🧠 **YOLO v10+** object detection / any CV pipeline — do inference on a beefier machine, not the Pi
- 📹 NVRs, Frigate, OBS, VLC — anything that speaks RTSP/WebRTC

The Pi only **captures, hardware-encodes, and serves**. All inference and post-processing happen **off-node**, so the Zero stays fast and the stream stays fluent.

> **Status:** binary releases are published from the private development
> pipeline. Install from the latest release asset.

## Why this exists

Streaming a Pi camera "fluently" on hardware this small is mostly about **not wasting the Pi's one superpower: the VideoCore hardware H.264 encoder.** This node drives that encoder directly (via MediaMTX's native `rpiCamera` source), encodes **once**, and fans the stream out to many consumers — keeping CPU low and latency sub-second on a LAN.

## Features

- **Hardware H.264** encoding on the Pi's VideoCore (CPU is never the bottleneck)
- **RTSP** (primary), **WebRTC** (~0.5s latency), and **HLS** from a single encode
- **One-line install** + `systemd` service (auto-start on boot)
- **mDNS discovery** — consumers find the node on the LAN automatically
- **Access control** built in — read requires credentials + optional IP allowlist (no anonymous viewers)
- **Default 720p30** tuned for fluent distribution; resolution/fps/bitrate configurable
- Reference: **Pi Zero 2 W** (64-bit Raspberry Pi OS Bookworm); original **Pi Zero / Zero W** supported as a constrained profile

## Install

```bash
curl -fsSL https://github.com/jhihweijhan/rpi-camera-rtsp-node/releases/latest/download/install.sh | \
  bash -s -- \
    --read-username viewer \
    --read-password '<your-password>'
```

This installs the node binary + bundled MediaMTX, writes a config, and enables the `systemd` service. Then your stream is available at:

```
rtsp://<user>:<pass>@<pi-host>.local:8554/cam
```

## Connect from go2rtc

```yaml
# go2rtc.yaml (runs on another machine — the consumer)
streams:
  pi_cam: rtsp://USER:PASS@rpi-node.local:8554/cam
```

go2rtc then re-publishes the stream as WebRTC/HLS/MJPEG to your viewers, dashboards, or a YOLO pipeline.

## Hardware

- **Raspberry Pi Zero 2 W** (recommended) or original Pi Zero / Zero W
- A **CSI** camera module (libcamera-supported sensor: OV5647 / IMX219 / IMX477 / IMX708, etc.)
- Raspberry Pi OS Bookworm

## Question and Answer

See [Question and Answer](./docs/question-and-answer.md) for installation notes,
stream credential handling, testing from another computer, and the Pi 3B +
OV5647 troubleshooting case study.

## License

Proprietary, **non-commercial** freeware — free for personal and non-commercial use; all rights reserved; no reverse engineering; provided as-is. See [LICENSE](./LICENSE). Bundled MediaMTX is distributed under its own MIT license.

---

<sub>Keywords: Raspberry Pi Zero RTSP camera, Pi Zero 2 W CSI camera stream, hardware H.264 RTSP server, go2rtc Raspberry Pi source, WebRTC Pi camera, low-latency YOLO RTSP node, libcamera RTSP, MediaMTX Raspberry Pi.</sub>
