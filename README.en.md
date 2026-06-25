# rpi-camera-rtsp-node

<!-- README-I18N:START -->

[繁體中文](./README.md) | **English**

<!-- README-I18N:END -->

## Supply-chain-conscious RTSP node for commercial video systems

`rpi-camera-rtsp-node` turns a Raspberry Pi Zero / Zero 2 W with a CSI camera
into a self-hosted RTSP / WebRTC / HLS video node.

It is not a full commercial IP camera replacement. It is a clean, hardware-H.264
RTSP source for go2rtc, NVRs, Frigate, YOLO v10+, and other AI/CV hosts.

The Pi node only captures, hardware-encodes, and serves. Inference, recording,
alerts, fan-out, and central management belong on consumer-side machines.

## Who it is for

- SI, low-voltage, and security integrators who need a controllable RTSP source.
- Commercial or public-sector deployments that need to reduce China-made camera
  device risk on internal networks.
- AI/NVR projects that need a Raspberry Pi CSI camera source for go2rtc, NVRs,
  YOLO hosts, or CV pipelines.
- Personal, household, non-profit educational, and non-profit deployments that
  need a small, rebuildable Raspberry Pi camera node.

## One-line install

Run this on the Raspberry Pi:

```bash
curl -fsSL https://github.com/jhihweijhan/rpi-camera-rtsp-node/releases/latest/download/install.sh | \
  bash -s -- \
    --read-username viewer \
    --read-password '<your-password>'
```

The installer deploys the Node binary, bundled MediaMTX, config, and `systemd`
service. You choose the stream password; the installer does not generate or
print it later.

## Stream endpoints

Default stream path: `cam`.

```text
RTSP:   rtsp://<user>:<pass>@<pi-host>.local:8554/cam
WebRTC: http://<pi-host>.local:8889/cam/
HLS:    http://<pi-host>.local:8888/cam/index.m3u8
```

Example go2rtc source:

```yaml
streams:
  rpi_camera:
    - rtsp://viewer:<read-password>@<node-host>:8554/cam
```

## Architecture

MediaMTX owns the media plane through its native `rpiCamera` source. The Node
Agent only handles control and provisioning: render config, manage service,
advertise mDNS, and report status.

```text
CSI camera -> Raspberry Pi Node -> RTSP/WebRTC/HLS -> go2rtc / NVR / YOLO host
                  |                    |
                  |                    +-- read credentials + optional IP allowlist
                  +-- hardware H.264 encode, no Python video frame path
```

## Supply-chain and licensing boundaries

This project provides a supply-chain-conscious deployment direction. Integrators
may choose standard distributors such as RS, DigiKey, and Mouser, then verify
part numbers, country-of-origin labels, receiving records, and BOM compliance
for their own deployment.

This project does not guarantee that a user's full BOM is non-China-made and
does not provide public-procurement, security-compliance, or country-of-origin
certification.

Read the Traditional Chinese checklist:

- [Supply-chain self-verification checklist](./docs/supply-chain-verification.zh-TW.md)

## Commercial licensing

Personal, household, non-profit educational, and other non-commercial use may be
free under the public license terms. Commercial deployment, government procurement projects,
SI/low-voltage/security integration, OEM/redistribution, and enterprise internal
operations require a separate commercial license.

Commercial licensing and integration inquiries:

- Non-sensitive general questions:
  [Commercial inquiry](https://github.com/jhihweijhan/rpi-camera-rtsp-node/issues/new?template=commercial-license.yml)
- Pricing, procurement, NDA, site topology, customer information, or deployment
  details: <jhihweijhan@gmail.com>

Do not post credentials, RTSP URLs, private network topology, camera locations,
customer names, procurement-sensitive information, or NDA-covered information in
public GitHub issues.

## Hardware profiles

| Profile | Hardware | Stream profile | Purpose |
| --- | --- | --- | --- |
| Reference | Raspberry Pi Zero 2 W, 64-bit OS | H.264 1280x720 around 30 fps | Release acceptance target |
| Constrained | Original Pi Zero / Zero W, ARMv6 | H.264 640x480 around 15 fps | Low-resource supported profile |
| Integration proxy | Pi 3B | Same verifier flow, non-reference | Day-to-day integration testing |

The OS must see the CSI camera through `rpicam-hello --list-cameras` or
`libcamera-hello --list-cameras`.

## Roadmap

Today each Node is self-contained and does not depend on a central coordinator.

Future direction: a consumer-side Fleet Layer that manages many Nodes, pulls
RTSP streams into machines with real compute, runs YOLO-style inference, and
shows a central view. This is roadmap language, not a current Node capability.

## Q&A and troubleshooting

- [Question and Answer](./docs/question-and-answer.md)
- [繁體中文 Q&A](./docs/question-and-answer.zh-TW.md)

## License

Proprietary, **non-commercial** freeware. Free for personal, household,
non-profit educational, and other non-commercial use. Commercial use requires a
separate written commercial license. See [LICENSE](./LICENSE). Bundled MediaMTX
remains governed by its own MIT license.
