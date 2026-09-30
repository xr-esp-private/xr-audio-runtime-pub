# 0.2.3 validation record

GitHub validation-only completed successfully on 2026-09-30:

- workflow run: <https://github.com/xr-esp-private/xr-audio-runtime-pub/actions/runs/36696181316>
- private source tag: `v0.2.3`
- private source commit: `a720d64679dd9dd37db5ce7091b93f732d45a20c`
- release framework: `v2.0.1`
- release framework commit: `4beac4eeee9080359d97d9defd02899181fc09af`
- caller commit: `4fc647ad82298ee385bbc1297acfb516206a2b7a`

Validated package SHA-256 values:

```text
988a13eff2c4e6785a0e2a6e33ff33f269a935e3cace51a0bbc105938f6c30aa  xraudio-audio-defaults_0.2.3_amd64.deb
2506feaba0b0316a6014aae0ba416abb510837b44c7abd72efcdd1d2e637e824  xraudio-audio-defaults_0.2.3_arm64.deb
```

The workflow bound the exact private tag and commit, built on native amd64 and
arm64 GitHub-hosted runners, audited the Basic-only package boundary, and
validated the merged checksum set. It did not create a GitHub Release or upload
to APT.

The arm64 artifact also passed checksum validation and privileged installation
on an Ubuntu 24.04.4 ARM64 Raspberry Pi running ROS 2 Jazzy, PipeWire 1.0.5 and
WirePlumber 0.4.17. The user service was enabled, remained active, restarted
cleanly, and correctly waited when no XR-AUD device was attached. Reboot,
default-endpoint, full-duplex, hotplug, HDMI competition and volume-retention
acceptance remain pending until `5852:7103` hardware is attached.
