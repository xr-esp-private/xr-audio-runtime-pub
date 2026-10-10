# XR-AUD-01 Linux 安装

安装本包后，系统自动选择 XR-AUD 的 **Speaker（扬声器）和 Clean Voice（麦克风）**，
开机和重新插入设备后自动恢复。首次 Speaker 音量为 **85%**，之后保留你的调整。
也支持 XR-AUD-02 的基础音频；无需授权文件或 ROS 2，不包含唤醒、声源定位等高级功能。
不安装本包也能使用 USB 音频，但需要手动选择输入／输出设备。

适用于使用 **PipeWire + WirePlumber** 的 64 位 Debian／Ubuntu 系统。
树莓派／Jetson 通常选择 `arm64`，x86-64 电脑选择 `amd64`；可用
`dpkg --print-architecture` 确认。不支持 `armhf`、纯 ALSA 或仅 PulseAudio 环境。

当前版本 **0.2.5**：已提供在线 APT 安装和
[离线下载](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases/tag/v0.2.5)。

## 1. 在线安装

首次安装复制下面整个代码块执行；已配置软件源时可跳过配置步骤。

```bash
(
  set -eu
  sudo apt update
  sudo apt install -y ca-certificates curl gnupg
  xr_apt_key=$(mktemp)
  trap 'rm -f "$xr_apt_key"' EXIT
  curl -fsSL --max-time 60 \
    https://raw.githubusercontent.com/xr-esp-private/xr-audio-runtime-pub/35df7bc02d43c8eaf1aa5c890e5a847c350b8888/keys/xr-apt-signing-key.asc \
    -o "$xr_apt_key"
  test "$(gpg --batch --show-keys --with-colons "$xr_apt_key" | awk -F: '$1 == "fpr" {print $10; exit}')" \
    = 1FEC138EA11C215E7B47CDFD028022BD3259955F
  sudo install -d -m 0755 /etc/apt/keyrings
  sudo install -m 0644 "$xr_apt_key" /etc/apt/keyrings/xr-audio-runtime.asc
  printf '%s\n' 'deb [signed-by=/etc/apt/keyrings/xr-audio-runtime.asc] http://47.106.100.173:8080 stable main' \
    | sudo tee /etc/apt/sources.list.d/xr-audio-runtime.list >/dev/null
  sudo apt update
  sudo apt install xraudio-audio-defaults
)
```

以后安装／升级只需：

```bash
sudo apt update
sudo apt install xraudio-audio-defaults
```

## 2. 离线 DEB／ZIP 安装

从 [Releases](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases) 下载
对应架构的 DEB 和 `SHA256SUMS`，或联系获取 ZIP。ZIP 先解压，在文件所在目录执行：

```bash
sha256sum --check SHA256SUMS --ignore-missing && \
  sudo apt install ./xraudio-audio-defaults_0.2.5_arm64.deb
```

`amd64` 系统换成对应文件名。旧版直接覆盖安装，无需先卸载。
缺失的系统依赖由 `apt` 补齐；完全断网时请先准备目标系统的依赖包。

## 3. 启用与检查

连接 XR-AUD，在正常使用音频的用户下执行（**不要加 sudo**）：

```bash
systemctl --user daemon-reload
systemctl --user enable --now xraudio-audio-session.service
wpctl status
```

带 `*` 的默认输出应为 XR-AUD Speaker，默认输入应为 XR-AUD 单声道 Clean Voice。
用音乐播放器和录音应用测试即可。首次测试建议只连接一块 XR-AUD。

不正常时，提供以下日志给维护人员：

```bash
journalctl --user -u xraudio-audio-session.service -n 50 --no-pager
```

## 4. 卸载

```bash
sudo apt remove xraudio-audio-defaults
```

若提示旧默认设备无法恢复，连接原声卡后重试，不要强删状态文件。

[维护与验收记录](VALIDATION.md)（普通使用者无需阅读）。
