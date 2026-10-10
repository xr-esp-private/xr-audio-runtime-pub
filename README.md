# XR-AUD Linux 基础音频安装

安装本包后，系统自动选择 XR-AUD 的 **Speaker（扬声器）和 Clean Voice（麦克风）**，
开机和重新插入设备后自动恢复。新设备首次 Speaker 音量为 **100%**，之后保留你的调整；
升级不会把已有设备的用户音量强制改为100%。
也支持 XR-AUD-02 的基础音频；无需授权文件或 ROS 2，不包含唤醒、声源定位等高级功能。
不安装本包也能使用 USB 音频，但需要手动选择输入／输出设备。

## 选择安装包

| 系统／音频后端 | 使用的包 |
| --- | --- |
| RPI Ubuntu 24.04、Pi OS Bookworm 等，使用 PipeWire + WirePlumber | `xraudio-audio-defaults` **0.2.7** |
| Jetson Nano Ubuntu 18.04，使用原生 PulseAudio 11 | `xraudio-audio-defaults-legacy` **0.1.0-rc.4** |
| Jetson Orin 或其它镜像 | 先确认实际后端；不能沿用 Nano 或 RPI 的验收结论 |

树莓派／Jetson 的 64 位系统通常为 `arm64`，x86-64 电脑为 `amd64`；以
`dpkg --print-architecture` 为准。不支持 `armhf` 和纯 ALSA 环境。
两个包互斥，不要同时安装；Legacy 不替换系统音频栈。

Legacy rc.4 针对已核验、Pulse未接管的 USB Playback 控件恢复0 dB，解决本次 Nano
重启后系统显示100%但硬件音量过低的问题。现代包仅增加只读诊断，不与PipeWire争抢
硬件音量控制。具体实板范围见[验收记录](VALIDATION.md)，不代表所有定制镜像均已验证。

## 1. 在线安装

两种系统使用同一个软件源。首次复制下面代码配置；已配置过可跳过。

```bash
(
  set -eu
  sudo apt update
  sudo apt install -y ca-certificates curl
  xr_apt_key=$(mktemp)
  trap 'rm -f "$xr_apt_key"' EXIT
  curl -fsSL --max-time 60 \
    https://raw.githubusercontent.com/xr-esp-private/xr-audio-runtime-pub/35df7bc02d43c8eaf1aa5c890e5a847c350b8888/keys/xr-apt-signing-key.asc \
    -o "$xr_apt_key"
  printf '%s  %s\n' c12ebdc1b91eecb839d29d7f3d8aab8e05e1ad1edbd0426f3c7c36a7c12220a5 \
    "$xr_apt_key" | sha256sum --check -
  sudo install -d -m 0755 /etc/apt/keyrings
  sudo install -m 0644 "$xr_apt_key" /etc/apt/keyrings/xr-audio-runtime.asc
  printf '%s\n' 'deb [signed-by=/etc/apt/keyrings/xr-audio-runtime.asc] http://47.106.100.173:8080 stable main' \
    | sudo tee /etc/apt/sources.list.d/xr-audio-runtime.list >/dev/null
  sudo apt update
)
```

**RPI／PipeWire 系统：**

```bash
sudo apt update
sudo apt install xraudio-audio-defaults
```

**Jetson Nano／Ubuntu 18.04／PulseAudio 系统：**

```bash
sudo apt update
sudo apt install xraudio-audio-defaults-legacy
```

## 2. 离线安装／升级

从 [Releases](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases) 下载
对应平台／架构的 DEB 和 `SHA256SUMS`，或联系获取 ZIP。ZIP 先解压，在文件所在目录执行。

**RPI／PipeWire 系统（0.2.7）：**

```bash
sha256sum --check SHA256SUMS --ignore-missing && \
  sudo apt install ./xraudio-audio-defaults_0.2.7_arm64.deb
```

**Jetson Nano／Ubuntu 18.04／PulseAudio：**

```bash
sha256sum --check SHA256SUMS --ignore-missing && \
  sudo apt install ./xraudio-audio-defaults-legacy_0.1.0-rc.4_arm64.deb
```

`amd64` 系统换成对应文件名。旧版直接覆盖安装，无需先卸载。
缺失的系统依赖由 `apt` 补齐；完全断网时请先准备目标系统的依赖包。
Nano 不要强装新版 libc 或从新 Ubuntu 软件源补库。

## 3. 启用与检查

连接 XR-AUD，在正常使用音频的用户下执行（**不要加 sudo**）。

**PipeWire：**

```bash
systemctl --user daemon-reload
systemctl --user enable --now xraudio-audio-session.service
wpctl status
```

**Nano／PulseAudio：**

```bash
systemctl --user daemon-reload
systemctl --user enable --now xraudio-audio-session-legacy.service
pactl info
```

默认输出应为 XR-AUD Speaker，默认输入应为 XR-AUD 单声道 Clean Voice。
用音乐播放器和录音应用测试即可。首次测试只连接一块 XR-AUD；应用如果指定了
HDMI 等固定声卡，请改为使用系统默认。

不正常时，提供以下日志给维护人员：

```bash
journalctl --user -u xraudio-audio-session.service -n 50 --no-pager
```

Legacy 的日志将服务名改为 `xraudio-audio-session-legacy.service`。

## 4. 卸载

```bash
sudo apt remove xraudio-audio-defaults
```

Legacy 使用 `sudo apt remove xraudio-audio-defaults-legacy`。

若提示旧默认设备无法恢复，连接原声卡后重试，不要强删状态文件。

[维护与验收记录](VALIDATION.md)（普通使用者无需阅读）。
