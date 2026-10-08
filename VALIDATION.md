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
hotplug were not rerun for 0.2.5. This remains a validation artifact until the
APT upload credentials are configured and the transactional release workflow
publishes both the GitHub Release and APT package.
