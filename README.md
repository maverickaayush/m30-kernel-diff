# Galaxy M30 RFCOMM kernel diff

Source changes and configuration for the RFCOMM-enabled kernel flashed to an
owned Samsung SM-M305F (`m30lte`) on 2026-10-31. This repository contains
reviewable source diffs and build inputs. The full kernel source, toolchain,
BOOT image, Magisk ramdisk, PIT, stock firmware and phone backups are not here.

The base is the public [M30-family kernel at commit
`095206579f69121278ed1dd5fd0d7975c0705c6a`](https://github.com/akhil1999/android_kernel_samsung_m30lte/tree/095206579f69121278ed1dd5fd0d7975c0705c6a).
This is a family source revision; an exact Samsung source release for the
phone's `M305FDDS8CVB2` build was not identified. The source patch set is
recorded in application order:

1. `research/kernel/scsc_bluez-family-compat.patch` adapts SCSC BlueZ bridge
   code to this 4.4 tree.
2. `research/kernel/scsc_bluez-send-lifetime.patch` retains the transmit skb
   through statistics accounting.
3. `research/kernel/abox-io-copy.patch` fixes ABOX copy paths found during the
   earlier boot trial.
4. `research/kernel/scsc_bluez-auto-register.patch` registers Linux HCI after
   the SCSC service probes.
5. `research/kernel/scsc_rfccomm-build.patch` links the RFCOMM protocol object
   that the family Makefile left commented out.

[`patches/combined-source.patch`](patches/combined-source.patch) is the
resulting cumulative diff against that base commit, covering four source
files. The five patches were applied in order to fresh copies of those four
upstream files with `git apply --check` and `git apply`; no conflict occurred.
`research/kernel/scsc_bt-keep-service.patch`, present in the broader lab, was
not selected for this build.

[`config/flashed.config`](config/flashed.config) is the effective kernel
configuration from the recorded RFCOMM build. It matched `/proc/config.gz`
from the previously installed research kernel byte for byte. The build used
`bluez-minimal` mode with `ABOX_IO_PATCH=1`,
`SCSC_BLUEZ_AUTO_REGISTER=1` and `SCSC_BT_RFCOMM=1`. The build script in
[`scripts/build_m30_kernel.sh`](scripts/build_m30_kernel.sh) requires a clean
checkout of the base source, an AArch64 cross compiler, and an isolated output
directory. For example:

```bash
SRC=/path/to/android_kernel_samsung_m30lte \
OUT=/path/to/new-build-dir \
CROSS_COMPILE=/path/to/aarch64-linux-gnu- \
ABOX_IO_PATCH=1 SCSC_BLUEZ_AUTO_REGISTER=1 SCSC_BT_RFCOMM=1 \
  ./scripts/build_m30_kernel.sh bluez-minimal
```

The recorded build's raw `Image` SHA-256 is
`7e49a023a071b7ef4f02a33de5d8d24ead63dc01ae0119c4adf2f843a56e53b3`.
Its `.config` SHA-256 is
`0566aa7ae2ab32065378ff27c88554a7c09a3e08558b59d5e02130e48a336e11`.
These hashes identify the actual build inputs and output; a fresh full rebuild
has not been run for this diff repository.

For this phone's Magisk legacy-SAR ramdisk, the raw `Image` was copied and
patched from `skip_initramfs` to `want_initramfs` at one marker, changing four
bytes. [`scripts/prepare_magisk_kernel.sh`](scripts/prepare_magisk_kernel.sh)
records and checks that transformation. Its patched `Image` SHA-256 is
`5e5d0b6af4fceeff4a8f89e84ea3cfb0742d0395142e130c95436aee06846c2c`.
[`scripts/package_m30_minimal.sh`](scripts/package_m30_minimal.sh) reproduced
the original working BOOT byte for byte before packaging the candidate with
the preserved boot header, device tree and ramdisk. The flashed 32 MiB BOOT
SHA-256 is
`498999fea57b10f6f95bcaef3ec0c57fe52a4088864233734764d59bcfce6549`.
The phone's post-flash BOOT readback matched that hash exactly.

Android booted; SELinux remained Enforcing; root, Wi-Fi and Android Bluetooth
worked; Linux `hci0` existed; and `/proc/net/protocols` listed RFCOMM and
L2CAP. A phone-side RFCOMM socket exchange with the cooperating laptop has not
been tested on this image. See the [lab's installation report](https://github.com/maverickaayush/m30-security-lab/blob/main/reports/rfcomm-kernel-flash-2026-10-31.md)
for the rollback and post-boot checks.
