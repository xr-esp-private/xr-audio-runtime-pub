# 0.1.6 validation record

GitHub validation-only completed successfully on 2026-09-03:

- workflow run: <https://github.com/xr-esp-private/xr-audio-runtime-pub/actions/runs/33739568852>
- private source tag: `v0.1.6`
- private source commit: `1d9a0ec2954e04cf72868ce7451b274a24216ea0`
- release framework: `v2.0.1`
- release framework commit: `4beac4eeee9080359d97d9defd02899181fc09af`
- caller commit: `ce07b4560bb281b3c7018be62865307e17bfacbe`

Validated package SHA-256 values:

```text
a3275338d4cf0f1d1c2dfb4b7a3fd2a58b8d64760893131f36d0a51a8b87231d  xraudio-audio-defaults_0.1.6_amd64.deb
ad2e27909aec93591ac5838b606c093f672e55cb25118a5436be8b245e40f625  xraudio-audio-defaults_0.1.6_arm64.deb
```

The workflow bound the exact private tag and commit, built on native amd64 and
arm64 GitHub-hosted runners, audited the Basic-only package boundary, and
validated the merged checksum set. It did not create a GitHub Release or upload
to APT.

The arm64 artifact also passed a non-privileged Raspberry Pi OS Bookworm
preflight: checksum, package identity/dependencies, extraction, and native
`xraudio-audio-session 0.1.6` execution. Privileged installation, reboot,
default-endpoint, full-duplex and hotplug acceptance remain pending. Jetson and
the additional Ubuntu host were offline during this check.
