# 問題與解答

<!-- README-I18N:START -->

[English](./question-and-answer.md) | **繁體中文**

<!-- README-I18N:END -->

這份文件記錄 `rpi-camera-rtsp-node` 的實用安裝筆記與疑難排解案例。公開使用者面向的
Q&A 以這份文件為主。

## Q: 串流密碼是什麼？installer 會自動產生嗎？

不會。讀取密碼由安裝者自己設定。

安裝時必須用 `READ_PASSWORD` 或 `--read-password` 提供：

```bash
curl -fsSL https://github.com/jhihweijhan/rpi-camera-rtsp-node/releases/latest/download/install.sh | \
  bash -s -- \
    --read-username viewer \
    --read-password '<your-password>' \
    --read-ip-allowlist 192.168.5.0/24
```

除非用 `--read-username` 覆蓋，預設讀取 username 是 `viewer`。

如果忘記已設定的帳密，可以在 Pi 上讀取已安裝的 node config：

```bash
sudo python3 - <<'PY'
import json
from pathlib import Path

config = json.loads(Path("/etc/rpi-camera-rtsp-node/node-config.json").read_text())
print(f"username={config['read_username']}")
print(f"password={config['read_password']}")
PY
```

## Q: 要怎麼從另一台電腦測試串流？

測試電腦必須和 Pi 在同一個 LAN 或 VPN。若安裝時設定了 IP allowlist，測試電腦也必須在
allowlist 範圍內。

RTSP：

```bash
ffplay "rtsp://viewer:<read-password>@<node-host>:8554/cam"
```

WebRTC：

```text
http://<node-host>:8889/cam/
```

HLS smoke test：

```bash
curl -u viewer:'<read-password>' -I http://<node-host>:8888/cam/index.m3u8
```

## Q: 為什麼 Pi 3B 安裝時曾出現 `Exec format error`？

這表示在 32-bit OS 上安裝了 `arm64` binary。公開 release 現在同時提供：

- `linux-arm64`：給 64-bit `aarch64` Raspberry Pi OS。
- `linux-armhf`：給 ARMv7-class 32-bit Raspberry Pi OS，例如跑 32-bit userland 的
  Pi 3B。

在目標機上檢查：

```bash
uname -m
getconf LONG_BIT
dpkg --print-architecture
```

Pi 3B + 32-bit Raspberry Pi OS 通常會看到：

```text
armv7l
32
armhf
```

## Q: 為什麼 `rpi-camera-mediamtx.service` 失敗並顯示 `no CSI camera detected`？

service 啟動 MediaMTX 前會先跑 camera preflight：

```bash
/opt/rpi-camera-rtsp-node/bin/rpi-camera-node-agent check-camera
```

這個 preflight 會呼叫：

```bash
rpicam-hello --list-cameras
```

如果 Raspberry Pi OS 看不到 CSI camera，service 會在 MediaMTX 啟動前失敗。這是刻意的：
當 libcamera 沒有 camera 時，MediaMTX 也無法驅動 `rpiCamera`。

最小診斷迴路：

```bash
rpicam-hello --list-cameras
vcgencmd get_camera
dmesg | grep -Ei 'unicam|ov5647|imx|camera|i2c'
```

當 `rpicam-hello --list-cameras` 顯示以下內容時，迴路是紅燈：

```text
No cameras available!
```

當它列出 camera 時，迴路是綠燈，例如：

```text
0 : ov5647 [2592x1944 10-bit GBRG] (.../ov5647@36)
```

## Q: 案例紀錄：Pi 3B + OV5647，camera 燈亮，但 OS 找不到 camera

### 初始症狀

installer 已經建立 service，但 service 啟動失敗：

```text
rpi-camera-node-agent: error: no CSI camera detected; connect a CSI camera and verify rpicam-hello --list-cameras
rpi-camera-mediamtx.service: Control process exited, code=exited, status=2/INVALIDARGUMENT
```

目標機資訊：

```text
Raspberry Pi 3 Model B Rev 1.2
Raspbian GNU/Linux 13 (trixie)
uname -m: armv7l
getconf LONG_BIT: 32
dpkg --print-architecture: armhf
```

camera module 預期是 OV5647。

### 第一輪診斷結果

修改 boot config 前：

```bash
rpicam-hello --list-cameras
vcgencmd get_camera
dtoverlay -l
```

顯示：

```text
No cameras available!
supported=0 detected=0, libcamera interfaces=0
No overlays loaded
```

live device tree 也顯示 CSI 與 camera I2C mux 是 disabled：

```text
/proc/device-tree/soc/csi@7e800000/status=disabled
/proc/device-tree/soc/csi@7e801000/status=disabled
/proc/device-tree/soc/i2c0mux/status=disabled
```

使用 libcamera debug logging：

```bash
LIBCAMERA_LOG_LEVELS='*:DEBUG' rpicam-hello -n -t 1000 -v 2
```

關鍵訊息是：

```text
Unable to acquire a Unicam instance
```

判讀：Raspberry Pi OS 尚未啟用 CSI/Unicam camera stack，所以 libcamera 沒有 sensor 可列舉。

### 手動指定 OV5647 overlay

對 OV5647/V1 camera，Raspberry Pi OS 支援手動 overlay：

```text
camera_auto_detect=0
dtoverlay=ov5647
```

將它套用到 `/boot/firmware/config.txt`，然後 reboot：

```bash
sudo cp -a /boot/firmware/config.txt /boot/firmware/config.txt.bak-$(date +%Y%m%d-%H%M%S)
sudo sed -i 's/^camera_auto_detect=.*/camera_auto_detect=0/' /boot/firmware/config.txt
grep -q '^dtoverlay=ov5647\b' /boot/firmware/config.txt || \
  echo 'dtoverlay=ov5647' | sudo tee -a /boot/firmware/config.txt
sudo reboot
```

### 重開後結果

重開後，overlay 已生效：

```text
/proc/device-tree/soc/csi@7e801000/status=okay
/proc/device-tree/soc/i2c0mux/status=okay
```

kernel 開始嘗試 probe OV5647：

```text
i2c i2c-11: Added multiplexed i2c bus 10
ov5647 10-0036: ov5647_read: i2c read error, reg: 300a = -5
ov5647 10-0036: ov5647_read: i2c read error, reg: 100 = -5
ov5647 10-0036: probe with driver ov5647 failed with error -5
```

但 camera 仍未被列舉：

```text
rpicam-hello --list-cameras
No cameras available!
```

I2C bus 10 也沒有顯示預期的 OV5647 address：

```bash
sudo i2cdetect -y 10
```

當 driver 擁有 sensor 時，預期會在 `0x36` 看到 `UU`。本案例中 bus 是空的。

### 結論

手動 overlay 已修好 OS 設定層：CSI/Unicam 與 camera I2C mux 變成 active，而且 OV5647
driver 開始 probe。

剩下的失敗在更低層：sensor 沒有在 CSI I2C bus 上回應。camera 燈亮只能證明有某種供電，
不能證明 sensor 已正確供電、脫離 reset、接線正常，或能透過 I2C 回應。

最可能原因：

1. Pi 端 CSI 排線方向或接觸問題。
2. camera board 端 CSI 排線方向或接觸問題。
3. sensor 與 camera PCB 之間的小接頭鬆動。
4. CSI 排線故障或不相容。
5. camera module 故障。
6. 模組實際上不是 OV5647，即使賣場標示為 V1-compatible camera。

下一步硬體檢查：

1. 將 Pi 完全斷電。
2. 重新插拔 CSI cable 兩端。
3. 確認 Pi 端與 camera board 端的排線金手指方向正確。
4. 檢查 sensor 與 camera PCB 之間是否有小接頭鬆動。
5. 換一條已知可用的 CSI ribbon cable。
6. 換一顆已知可用的 OV5647 camera module 測試。

硬體層修好後：

```bash
rpicam-hello --list-cameras
sudo systemctl start rpi-camera-mediamtx.service
```

## References

- Raspberry Pi camera software documentation:
  <https://www.raspberrypi.com/documentation/computers/camera_software.html>
- Raspberry Pi camera hardware documentation:
  <https://www.raspberrypi.com/documentation/accessories/camera.html>
- Raspberry Pi forum note on camera regulator/power and I2C response:
  <https://forums.raspberrypi.com/viewtopic.php?t=381389>
