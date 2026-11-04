# Hanoip kernel and Android device-tree baseline

**Research date:** 2026-11-04  
**Target:** Motorola Moto G60, XT2135-2, `hanoip`, Android 12 build `S2RIS32.32-20-9-9-2-3`.

This document records source references for understanding the phone's existing Linux/Android platform. These sources are research evidence only. They do not provide XNU drivers, do not establish XNU compatibility, and do not authorize experiments on the physical phone.

## 1. Exact Motorola kernel source release

Motorola's own `MotorolaMobilityLLC/kernel-msm` repository has the exact source tag:

- Tag: `MMI-S2RIS32.32-20-9-9-2-3`
- Tag object: `f3db6b21a89490d61f530d40584af8c6c9221cae`
- Peeled commit: `f6673a1302e3b9ca84c54e956ad4075d676fdf59`
- Tag message: `Hanoip push for Android 12`
- Tagger date: 2023-12-07
- Kernel Makefile version: Linux `4.14.190`
- Top-level `COPYING`: GNU General Public License version 2 text

Motorola linked this exact tag in its response to source request #574, which names the owner's exact firmware build and XT2135-2 RETEU package. This is strong evidence that the matching source release exists in Motorola's official repository.

References:
- Official tag: https://github.com/MotorolaMobilityLLC/kernel-msm/releases/tag/MMI-S2RIS32.32-20-9-9-2-3
- Tag object: https://github.com/MotorolaMobilityLLC/kernel-msm/tree/MMI-S2RIS32.32-20-9-9-2-3
- Motorola source request #574: https://github.com/MotorolaMobilityLLC/kernel-msm/issues/574

The tag object is annotated but GitHub reports its signature as unverified/unsigned. Preserve the tag and commit IDs as provenance references; do not describe the tag as cryptographically verified.

## 2. Kernel configuration evidence

The exact Motorola tag contains an ARM64 vendor configuration named:

`arch/arm64/configs/vendor/sdmsteppe-perf_defconfig`

The file enables Qualcomm platform options including:
- `CONFIG_ARCH_QCOM`
- `CONFIG_ARCH_SM6150`
- `CONFIG_ARCH_SDMMAGPIE`
- `CONFIG_NR_CPUS=8`
- Qualcomm clock, interrupt, power, storage, display, USB, input, audio, camera, and networking options.

It also contains a Motorola-specific extension referenced by the hanoip Android device tree:

`arch/arm64/configs/vendor/ext_config/moto-sdmmagpie-hanoip.config`

This extension includes `CONFIG_HANOIP_DTB=y`. This is particularly useful because it directly ties the exact kernel source release to a Hanoip device-tree build configuration.

A second extension, `moto-sdmsteppe.config`, supplies additional Motorola kernel configuration. The device-tree's build configuration combines the Qualcomm base defconfig and Motorola extensions.

These are Linux kernel configuration facts. They describe the Android kernel's existing hardware enablement; they are not an XNU configuration and cannot be copied over as XNU drivers.

## 3. Android device-tree reference

The public repository:

https://github.com/motorola-sm6150-devs/device_motorola_hanoip

identifies itself as the Android device tree for Moto G40 Fusion / Moto G60 (`hanoip`). Its `lineage-20` branch currently resolves to:

`514e0261a117a0c7e7fa3cc3cd44f80819b0b779`

Its `BoardConfig.mk` records:
- `TARGET_ARCH := arm64`
- `TARGET_ARCH_VARIANT := armv8-2a-dotprod`
- `TARGET_CPU_VARIANT := cortex-a76`
- `TARGET_BOARD_PLATFORM := sm6150`
- `TARGET_BOOTLOADER_BOARD_NAME := hanoip`
- `TARGET_KERNEL_SOURCE := kernel/motorola/sm6150`
- `TARGET_KERNEL_CONFIG` includes `vendor/sdmsteppe-perf_defconfig`, `vendor/ext_config/moto-sdmsteppe.config`, and `vendor/ext_config/moto-sdmmagpie-hanoip.config`

It also records Android boot command-line and image metadata. Those details are useful for documenting the existing Android boot environment, but must not be treated as an XNU boot protocol.

The branch is a community Android device-tree reference, not an official Motorola source publication. Its lineage release branch and commit date do not prove it exactly matches the owner's installed firmware.

## 4. Hardware naming discrepancy to resolve

Some community device trees describe the Snapdragon 732G as Qualcomm `SM7150`, while the Android device tree uses `sm6150` as its Android board platform and the Motorola kernel configuration enables both `SM6150` and `SDMMAGPIE` platform options.

Do not flatten these identifiers into one claim. They may refer to different layers of Qualcomm platform naming, product configuration, or shared kernel support. HanoipOS should record the exact source-level identifiers and verify the SoC and board mapping from the kernel's device-tree sources before making architectural assumptions.

## 5. What this source baseline establishes

| Finding | Status |
|---|---|
| Exact Motorola kernel source tag for the owner's build exists | Confirmed by Motorola's source request response and tag |
| Kernel source version | Linux 4.14.190 |
| ARM64 kernel configuration exists | Confirmed |
| Hanoip-specific config extension exists in the exact tag | Confirmed: `CONFIG_HANOIP_DTB=y` |
| Community Android device tree identifies `hanoip` | Confirmed |
| Exact kernel source tag is suitable as an XNU driver source | No; Linux code is not an XNU driver |
| XNU can boot on the Moto G60 | Unknown |
| XNU can use the Android device tree directly | Not established |
| Exact running kernel binary on the owner's phone equals this source tag | Not verified; matching firmware source tag is not binary identity proof |

## 6. Next research tasks

1. Trace the `CONFIG_HANOIP_DTB` build option to the Hanoip device-tree source files in the exact Motorola tag.
2. Identify the board-compatible strings, memory description, interrupt controller, timers, and CPU topology from the source tree.
3. Compare those platform facts with the requirements of the selected XNU source revision.
4. Maintain a separate inventory for Linux drivers, firmware dependencies, and hardware blocks. Treat every prospective XNU driver as new engineering work unless a legally and technically reviewed compatible implementation is found.

All work in this stage is source inspection and documentation. Do not alter the phone, boot firmware, partitions, or installed operating system.
