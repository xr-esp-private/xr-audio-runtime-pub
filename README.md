# XR Audio Linux Runtime

Public delivery repository for XR-AUD Linux Basic packages. It contains release
workflows, public signing material, installation documentation, and published
DEB assets. Product source and advanced SDK algorithms are maintained in a
private repository and are not distributed here.

## Basic package boundary

`xraudio-audio-defaults` configures supported Linux desktops to select the
XR-AUD Standard Speaker and XR-AUD Clean Voice endpoints. Basic USB audio does
not require an XR license. Raw-8 capture, DOA, wake-word processing, ROS 2 and
other advanced capabilities are not part of this package.

Current formal candidate: `0.1.6` for Debian 12 compatible amd64 and arm64
systems.

## Install from a GitHub Release

Download the matching `xraudio-audio-defaults_0.1.6_<arch>.deb` and
`SHA256SUMS` from Releases, verify it, then install it with APT:

```bash
sha256sum --check SHA256SUMS --ignore-missing
sudo apt install ./xraudio-audio-defaults_0.1.6_arm64.deb
```

Use the amd64 package on x86-64 systems. APT installs declared system
dependencies automatically.

## Install from the XR APT repository

The repository currently uses HTTP. Package authenticity therefore depends on
the independently pinned signing key below; HTTP does not provide
confidentiality. Do not obtain the key from the HTTP APT origin.

Expected full signing-key fingerprint:

```text
1FEC138EA11C215E7B47CDFD028022BD3259955F
```

Install the key from its immutable GitHub commit, then configure the repository:

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL \
  https://raw.githubusercontent.com/xr-esp-private/xr-audio-runtime-pub/35df7bc02d43c8eaf1aa5c890e5a847c350b8888/keys/xr-apt-signing-key.asc \
  | sudo tee /etc/apt/keyrings/xr-audio-runtime.asc >/dev/null
sudo chmod 0644 /etc/apt/keyrings/xr-audio-runtime.asc
```

The APT source entry is:

```text
deb [signed-by=/etc/apt/keyrings/xr-audio-runtime.asc] http://47.106.100.173:8080 stable main
```

Then run:

```bash
sudo apt update
sudo apt install xraudio-audio-defaults
```

## Remove

```bash
sudo apt remove xraudio-audio-defaults
```

Removal restores the package-managed default-audio integration. User-selected
desktop audio settings remain under the control of the desktop audio stack.

## Maintainer release flow

1. Freeze an exact private source tag and commit.
2. Run `Validate XR Audio Linux Basic`; it builds and audits native amd64 and
   arm64 packages without Release or APT credentials.
3. Smoke-test the validated artifacts on supported Linux systems.
4. Configure the APT upload secrets and run `Release XR Audio Linux Basic`.

Both workflows call the immutable release framework commit
`78cb5530b2f5bed2c4c56206764b1cce71a645d9`. Private source checkout uses a
repository-scoped read-only Deploy Key.
