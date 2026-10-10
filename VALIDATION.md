# 0.2.5 validation record

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
