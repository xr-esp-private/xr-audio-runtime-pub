# Basic 发布与验收记录

## 2026-10-10：Modern 0.2.7 / Legacy 0.1.0-rc.4

两种包的 ARM64、AMD64 均已发布到 APT `stable/main`：

| 包 | Release | 正式工作流 |
| --- | --- | --- |
| Modern 0.2.7 | [v0.2.7](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases/tag/v0.2.7)，非 prerelease | [38043970145](https://github.com/xr-esp-private/xr-audio-runtime-pub/actions/runs/38043970145) |
| Legacy rc.4 | [v0.1.0-rc.4](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases/tag/v0.1.0-rc.4)，prerelease | [38043965295](https://github.com/xr-esp-private/xr-audio-runtime-pub/actions/runs/38043965295) |

- Modern source：`d4d21b64feec5a3d701c99ece8f585c80883e5d7`；Legacy source：
  `7ce73842e3697772ae88a94666606610b6ad3942`。
- Caller：`28c0da9`；framework：`08819029fdc1da9392be55c00d8bae73665bacf3`。
- CI 完成双架构构建/测试、Basic-only payload 审计、APT签名/索引/工件校验和GitHub公开。
  发布后维护端再次从GitHub和APT下载，以下四个SHA一致；Nano安装的rc.4与发布工件相同字节。

```text
1a3fbbf36c8da9ddd129d501b4212c4e4195b712b75bd347d691f9e1dc44a199  xraudio-audio-defaults_0.2.7_amd64.deb
b7c9b7c26af2f5d019a28c3d039f05502f71de0d464d154eb94652b0138925d3  xraudio-audio-defaults_0.2.7_arm64.deb
967dc6c595ea668c352e3327d7701e3125b5ee12826304e6956f0f6bd8b604ec  xraudio-audio-defaults-legacy_0.1.0-rc.4_amd64.deb
fc4dc3c402f3c7d395eab5a949b78e077505390cbf0a9f196e36102c872a50b3  xraudio-audio-defaults-legacy_0.1.0-rc.4_arm64.deb
```

### Nano 实板范围

Ubuntu18.04.6 ARM64／内核4.9.140-tegra／PulseAudio11.1，单台 XR-AUD-01
`5852:7103`、完整SN `XR48CA43A3FF98`。APP完整版本未核验，不借用其它板版本。

1. rc.3 整机重启复现：默认Speaker/Clean Voice及duplex profile正确，Pulse100%，
   但硬件master/LR均为raw0（声明−50dB）。此前用户确认手动恢复0dB后多次播放正常。
2. rc.4 安装前置硬件低值、软件62%，DEB升级只升级本包，无系统库升级。
   服务自动恢复硬件raw50／0dB，保留62%；服务重启也保留62%。
3. 恢复用户批准的100%后整机重启：服务自动启动，默认端点正确，硬件master/LR
   自动0dB、Pulse100%，后续回读保持；paplay六秒测试文件返回0，硬件值未降低。
   用户随后执行同一测试指令并明确确认“能正常听到”，完成本台Nano新包重启后的
   基础播放试听验收；不外推为完整AEC或声学性能验收。
4. 配置签名APT源后，Nano成功读取rc.4候选；另外实际从APT下载43.7kB工件并执行
   固定版本reinstall，只有本包重装、无系统库升级。公钥使用固定GitHub commit与SHA验证，
   APT正常验证InRelease。

未重新验证完整AEC、录音、真实物理拔插或其它Nano镜像。rc.4保留prerelease标记；
APT的`stable`是仓库suite，不等于所有平台均通过量产验收。

### Modern 范围与安装兼容性

Modern新装首次100%，旧初始化状态及用户调音保留；增加严格USB绑定的只读Playback
诊断，不写硬件控件、不与PipeWire/WirePlumber争抢。增加系统`alsa-utils`依赖。
Mac全套16测试、Linux双架构CI通过，未对0.2.7重跑RPI/Orin实板；0.2.5的旧实板结论
不冒充新包验收。0.2.6仅有失败CI：测试fixture的单元素optional/vector初始化不兼容
GCC12，未发布DEB；修复采用0.2.7，不覆盖其源码tag。

Nano GnuPG2.2.4不支持原README的`--show-keys`。现使用固定公钥文件SHA-256
`c12ebdc1b91eecb839d29d7f3d8aab8e05e1ad1edbd0426f3c7c36a7c12220a5`校验，
指纹仍为`1FEC138EA11C215E7B47CDFD028022BD3259955F`；APT签名验证不关闭。

## 历史：0.2.5 validation record

GitHub validation-only completed successfully on 2026-10-08:

- workflow run: <https://github.com/xr-esp-private/xr-audio-runtime-pub/actions/runs/37745343336>
- private source tag: `v0.2.5`
- private source commit: `39223ecc86499458649a3120d802ffd7751a82f5`
- release framework: `v2.0.1`
- release framework commit: `4beac4eeee9080359d97d9defd02899181fc09af`
- caller commit: `5f857698c09968fbc238ae7895a10a6f5011dc02`

Validated package SHA-256 values:

```text
6bc92c980f285026579b6dc3c50d1821b293a6b455d8f4c0c428648d0a27ec31  xraudio-audio-defaults_0.2.5_amd64.deb
6eb23967743e78da8b734e634f5647fb3d1a9baffc063c3fc0234722c3633481  xraudio-audio-defaults_0.2.5_arm64.deb
```

The workflow bound the exact private tag and commit, built on native amd64 and
arm64 GitHub-hosted runners, audited the Basic-only package boundary, and
validated the merged checksum set. It did not create a GitHub Release or upload
to APT.

The arm64 artifact also passed checksum validation and installation on Ubuntu
24.04 ARM64 Raspberry Pi 5 with `5852:7103 XR-AUD-FF94`. The package selected
the strict 2-channel Speaker and mono Clean Voice pair, initialized Speaker to
85 percent once and did not change Clean Voice gain. After the user changed
Speaker to 62 percent, restarting the service preserved 62 percent; the final
robot operating state was restored to 85 percent with the service active.

Acoustic playback/recording, full duplex, whole-system reboot and physical USB
hotplug were not rerun for 0.2.5. At the end of the October 8 validation this was
a validation-only artifact. It has since been published as recorded below;
publication does not broaden the physical acceptance boundary.

## 0.2.5 正式发布：2026-10-10

- [GitHub Release v0.2.5](https://github.com/xr-esp-private/xr-audio-runtime-pub/releases/tag/v0.2.5)：
  `draft=false`、`prerelease=false`。
- [正式发布工作流](https://github.com/xr-esp-private/xr-audio-runtime-pub/actions/runs/38023094065)：
  源码绑定、两架构构建／测试、草稿资产、APT 验证和公开 Release 全部通过。
- Caller commit：`f3664b228b6be7ffe84ab14692bdf095228c26e6`。
- Framework commit：`08819029fdc1da9392be55c00d8bae73665bacf3`。
- 源码仍是上述 `v0.2.5` / `39223ecc86499458649a3120d802ffd7751a82f5`，
  两个 DEB 的 SHA-256 与 October 8 的验证工件完全一致。
- APT `http://47.106.100.173:8080` 的 `stable/main` 已收录 `arm64`、`amd64`
  的 `xraudio-audio-defaults 0.2.5`；CI 已验证签名、索引和 DEB SHA-256，
  维护端另从 APT 和 GitHub 下载两包核对同一哈希。

首次运行 `38022604828` 已上传并验证 APT，但在公开 Release 时遇到无 Git tag 的
草稿查询问题；[框架 PR #6](https://github.com/xr-esp-private/xr-release-workflows/pull/6)
修复精确 tag 的认证列表查询后重试成功，复用了原草稿和相同字节，未覆盖已发布包。
只发布 Basic，不包含高级 Runtime、模型或固件，不改变已经验收的音频实现。

## 发布维护

`Validate XR Audio Linux Basic` 只构建／测试，不上传 APT，不创建 Release。
`Release XR Audio Linux Basic` 执行：双架构构建 → GitHub 草稿资产 →
`PUT <APT base URL>/upload/<deb文件名>` → 验证 `dists/stable/main` 的签名和包 SHA-256
→ 正式公开 GitHub Release。上传或验证失败须阻断发布，不能忽略错误。

Release 显式传递组织 Secrets `XR_APT_REPO_URL`、`XR_APT_REPO_USER`、
`XR_APT_REPO_PASS`；组织必须允许发布仓使用它们。URL 只填公共基础地址
`http://47.106.100.173:8080`，不含 `/upload`、`/stable`、凭据或查询参数。
URL Secret 优先，空时回退旧仓库变量 `XR_APT_REPOSITORY_URL`。
目标固定 `stable/main`；公钥 URL 和 fingerprint 由仓库变量提供，客户端不需要
上传凭据。HTTP 安装仍须验证固定 GitHub 公钥和 APT 签名，不使用 `trusted=yes`。
私有源码仅使用本仓限定的只读 Deploy Key，不继承全部组织 Secrets。

0.2.5 的源码 tag 和 commit 保持不变。修复后的 Release caller 锁定框架提交
`08819029fdc1da9392be55c00d8bae73665bacf3`；必须先推送框架提交，再推送／运行
caller。Validate 保留已验证的 v2.0.1 实现，不需要 APT 凭据。
实际发布证据见上节；不能仅凭本地测试宣称上线。

完全断网部署还需目标系统依赖：`libc6`、`libgcc-s1`、`libstdc++6`、
`init-system-helpers`、`pipewire-bin`、`wireplumber`。本包不携带完整系统音频栈。
