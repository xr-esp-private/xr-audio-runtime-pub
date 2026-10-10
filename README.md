# XR-AUD-01 Linux 基础音频安装

本仓提供 `xraudio-audio-defaults`：自动把 XR-AUD-01 的双声道 Speaker 和单声道
Clean Voice 设为系统默认输出／输入，避免重启或连接 HDMI 后选错声卡。
同一 Basic 包也支持已登记的 XR-AUD-02 Standard Audio；不包含 Raw-8、声源定位、
唤醒、ASR、ROS 2、OTA 或授权校验。设备固件中的 AEC 不依赖本包。

不安装时仍可使用标准 USB 音频，但需要在系统／应用中手动选择 XR-AUD。
本仓不公开产品源码。

## 系统和当前状态

- 需要 64 位 `arm64` 或 `amd64` Debian／Ubuntu，以及正在运行的
  PipeWire + WirePlumber 用户音频会话；不支持 `armhf`、纯 ALSA 或仅 PulseAudio。
- 树莓派／Jetson 使用 `arm64`，普通 x86-64 Linux 使用 `amd64`，以
  `dpkg --print-architecture` 的输出为准。ROS 2 不是依赖。
- 当前包版本 **0.2.5**。双架构 CI 已通过，ARM64 已在 Raspberry Pi 5 /
  Ubuntu 24.04 / XR-AUD-01 实测安装、默认端点和音量保持；Pi OS Bookworm、Jetson
  的完整录放／重启验收是历史版本证据，不等于 0.2.5 已全部复验。
- **0.2.5 目前仍是验证工件，尚未发布到 GitHub Release／APT。**在线安装命令在
  正式发布完成后使用；此前请联系提供离线包。准确证据见 [VALIDATION.md](VALIDATION.md)。

首次确认正确音频端点后，Speaker 初始化为 **85% 一次**；之后保留用户音量，
不修改 Clean Voice 输入增益。多块 XR-AUD 同时接入时，需要绑定完整 USB serial，
不能靠设备名称后四位猜测。

## 在线安装（正式 APT 发布完成后）

APT 地址为 `http://47.106.100.173:8080`，发行通道 **stable**，组件 **main**。
客户端读取无需上传账号／密码。HTTP 沿用现有部署，必须启用签名验证，不能使用
`trusted=yes`。签名公钥从固定 GitHub 提交获取，不从 HTTP 软件源获取。

先安装工具、下载并核对公钥；整个代码块作为一组执行：

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
  apt-cache policy xraudio-audio-defaults
  sudo apt install xraudio-audio-defaults
)
```

正式发布 0.2.5 后，`apt-cache policy` 应能看到对应候选版本。没有候选时先检查
软件源／发布状态，不要通过关闭签名验证解决。以后更新：

```bash
sudo apt update
sudo apt install --only-upgrade xraudio-audio-defaults
```

## 离线 DEB／ZIP 安装

发布后从 [Releases](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases)
下载对应 DEB 与 `SHA256SUMS`；发布前由维护人员提供同一验证工件。若收到 ZIP，
先解压并进入 DEB 和校验清单所在目录，然后执行（树莓派／Jetson）：

```bash
dpkg --print-architecture
sha256sum --check SHA256SUMS --ignore-missing && \
  sudo apt install ./xraudio-audio-defaults_0.2.5_arm64.deb
```

架构为 `amd64` 时改用对应 `amd64.deb`；校验不通过不要安装。
已安装旧版也用同一命令升级，不需要先卸载。

“离线包”指产品包从本地读取，**不代表系统依赖全被打包进去**。
`apt` 会补齐缺失依赖；完全断网时须预先安装目标系统对应的依赖：`libc6`、
`libgcc-s1`、`libstdc++6`、`init-system-helpers`、`pipewire-bin`、`wireplumber`。
不要把 Ubuntu 的系统库复制给 Pi OS，也不要用 `dpkg -i` 忽略缺失依赖。

## 启用与检查

在正常桌面用户下执行以下命令，**不要加 sudo**：

```bash
systemctl --user daemon-reload
systemctl --user enable --now xraudio-audio-session.service
systemctl --user is-active xraudio-audio-session.service
wpctl status
```

服务应为 `active`；默认 Sink 为 XR-AUD 双声道 Speaker，默认 Source 为单声道
Clean Voice，不是 HDMI 或 Raw Array。通过系统播放器播放、录音应用录音即可测试。
无设备时服务等待连接，不应把“进程运行”当成已选中设备。

异常时保存：

```bash
journalctl --user -u xraudio-audio-session.service -n 50 --no-pager
```

卸载：`sudo apt remove xraudio-audio-defaults`。若旧声卡未连接导致默认值恢复失败，
先连接旧声卡后重试，不要强删状态文件。

## 发布维护

`Validate XR Audio Linux Basic` 只构建／测试，**不会上传 APT 或创建 Release**。
`Release XR Audio Linux Basic` 才执行：双架构构建 → GitHub 草稿资产 →
`PUT <APT base URL>/upload/<deb文件名>` → 校验 `dists/stable/main` 的签名与包 SHA-256
→ 正式公开 GitHub Release。上传失败不能显示为发布成功。

正式工作流显式传递组织 Secrets `XR_APT_REPO_URL`、`XR_APT_REPO_USER`、
`XR_APT_REPO_PASS`；组织需允许本仓使用。URL 只填基础地址，不包含 `/upload`、
`/stable`、账号、密码或查询参数；旧 `XR_APT_REPOSITORY_URL` 仓库变量保留作回退。
发布目标固定 `stable/main`，签名公钥 URL 与 fingerprint 继续由仓库变量提供。
私有源码使用本仓限定的只读 Deploy Key，不继承所有组织 Secrets。

源码 tag `v0.2.5` 和 commit `39223ecc86499458649a3120d802ffd7751a82f5` 保持不变；
工作流实现须单独锁定完整 commit。工作流修复不改已经验证的音频二进制版本。

本次 Release caller 锁定框架提交 `219ea93831159e132d69bddc25ae93227b6c9b85`；
此提交必须先推到框架远端，再推送／运行 caller。Validate 保留已验证的 v2.0.1
实现，不需要 APT 凭据。发布完成后再更新本页状态和 VALIDATION 记录。
