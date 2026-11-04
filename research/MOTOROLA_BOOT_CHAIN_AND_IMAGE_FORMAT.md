# Stage B.4 — Motorola boot chain and Android boot-image format research

**Research date:** 2026-11-04  
**Target:** Motorola Moto G60, model XT2135-2, codename hanoip  
**Firmware context:** Android 12, build S2RIS32.32-20-9-9-2-3  
**Scope:** Public-source documentation only. No device-side inspection, bootloader changes, image modification, flashing, or boot attempts.

## 1. Executive summary

The exact Motorola kernel source tag is available for the firmware build named in Motorola kernel-source issue #574. It shows how the Linux kernel and Hanoip device-tree overlays are built, but it does not document the proprietary boot firmware stages or prove the exact layout of the stock firmware's boot-related partitions.

A separate community Hanoip Android device tree (lineage-20) contains a concrete Android build configuration. That configuration sets Android boot-image header version 3, enables DTB inclusion in the image-building configuration, uses a separated DTBO image, enables AVB in that community build, and declares a vendor-boot partition size. These are facts about that community build configuration—not proof of the stock Motorola Android 12 firmware's image header, partition map, AVB keys, or boot chain.

Android's official documentation says Android 12 devices may use boot-image header version 3 or 4. Therefore Android 12 by itself does not determine the header version. The community configuration is useful evidence of a plausible Hanoip build arrangement, but the stock firmware format remains unverified.

For HanoipOS, the central finding remains that an Android boot image is not an XNU boot protocol. XNU expects an Apple-specific ARM64 boot_args structure and device-tree/memory information. No public evidence inspected in this stage shows that Motorola's stock boot firmware supplies that contract.

## 2. Sources and revisions examined

### Motorola kernel source

- Repository: https://github.com/MotorolaMobilityLLC/kernel-msm
- Tag: MMI-S2RIS32.32-20-9-9-2-3
- Peeled commit: f6673a1302e3b9ca84c54e956ad4075d676fdf59
- Files inspected:
  - arch/arm64/Makefile
  - arch/arm64/boot/Makefile
  - arch/arm64/boot/dts/qcom/Makefile
  - arch/arm64/configs/vendor/sdmsteppe-perf_defconfig
  - arch/arm64/configs/vendor/ext_config/moto-sdmmagpie-hanoip.config

Motorola issue #574 identifies the source firmware package as XT2135-2_HANOIP_RETEU_12_S2RIS32.32-20-9-9-2-3_subsidy-DEFAULT_regulatory-XT2135-2-EUROPE_CFC.xml.zip and links the exact source tag:
https://github.com/MotorolaMobilityLLC/kernel-msm/issues/574

This strongly associates the kernel source tag with the reported XT2135-2 firmware build. It does not prove that a specific retail handset's currently installed kernel binary is byte-for-byte identical to a build from this source.

### Community Hanoip Android device tree

- Repository: https://github.com/motorola-sm6150-devs/device_motorola_hanoip
- Branch: lineage-20
- Previously recorded commit: 514e0261a117a0c7e7fa3cc3cd44f80819b0b779
- File inspected: BoardConfig.mk
- File: https://github.com/motorola-sm6150-devs/device_motorola_hanoip/blob/lineage-20/BoardConfig.mk

This is a community Android build tree, not an official Motorola firmware build manifest. Its settings are evidence about that community configuration only.

### Android boot-image format documentation

- AOSP boot-image header documentation: https://source.android.com/docs/core/architecture/bootloader/boot-image-header
- AOSP DTB image documentation: https://source.android.com/docs/core/architecture/bootloader/dtb-images
- AOSP mkbootimg header definitions: https://android.googlesource.com/platform/system/tools/mkbootimg/+/refs/heads/main/include/bootimg/bootimg.h
- AOSP build logic reference: https://android.googlesource.com/platform/build/+/refs/heads/master/core/Makefile

These describe Android's generic build and image formats. They do not establish the format used by a particular stock Motorola firmware release.

## 3. What the exact Motorola kernel source establishes

### ARM64 kernel output

The exact source tag's arch/arm64/Makefile selects Image as the default ARM64 kernel image unless the compression configuration selects Image.gz. It also supports an appended-DTB build target when CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE is enabled.

The architecture boot Makefile defines:
- Image from vmlinux;
- compressed forms such as Image.gz and Image.lz4;
- Image-dtb and Image.gz-dtb concatenation targets;
- DTB selection from the configured device-tree build outputs.

This describes kernel build outputs. It does not tell us which exact output was included in the stock firmware package or how Motorola's proprietary boot firmware consumed it.

### Hanoip DTB and overlays

The Hanoip-specific kernel configuration sets CONFIG_HANOIP_DTB=y.

The exact kernel DTS Makefile maps this setting to three Hanoip overlays:
- sdmmagpie-hanoip-evt1-overlay.dtbo
- sdmmagpie-hanoip-dvt1-overlay.dtbo
- sdmmagpie-hanoip-dvt2-overlay.dtbo

Each overlay is associated with the shared sdmmagpie-hanoi-base.dts. The Makefile also enables 1 KiB of padding for DTBs under the relevant device-tree configuration, with a comment explaining that this allows bootloader-added environment properties such as /chosen values.

This supports the previous Stage B.3 observation that runtime firmware may augment the static device tree. It does not reveal which overlay is selected on the user's particular handset or what properties are added at runtime.

## 4. What the community Hanoip build configuration says

The community lineage-20 BoardConfig.mk contains the following configuration facts:

| Setting | Community configuration value | Interpretation / limitation |
|---|---|---|
| BOARD_KERNEL_IMAGE_NAME | Image | Names the kernel build output |
| BOARD_KERNEL_BASE | 0x00000000 | Build-time image parameter; not a verified stock load address |
| BOARD_KERNEL_PAGESIZE | 4096 | Build-time page-size setting |
| BOARD_BOOT_HEADER_VERSION | 3 | Community build selects Android boot-image header v3 |
| BOARD_KERNEL_SEPARATED_DTBO | true | Community build config treats DTBO as a separate image |
| BOARD_INCLUDE_DTB_IN_BOOTIMG | true | Enables DTB inclusion in the Android image-building configuration; exact destination depends on the build target and Android build logic |
| BOARD_RAMDISK_USE_LZ4 | true | Community ramdisk compression setting |
| BOARD_USES_RECOVERY_AS_BOOT | true | Community recovery/boot layout setting |
| BOARD_VENDOR_BOOTIMAGE_PARTITION_SIZE | 67108864 | Community build declares a 64 MiB vendor-boot image size |
| BOARD_DTBOIMG_PARTITION_SIZE | 25165824 | Community build declares a 24 MiB DTBO image size |
| BOARD_BOOTIMAGE_PARTITION_SIZE | 67108864 | Community build declares a 64 MiB boot image size |
| BOARD_AVB_ENABLE | true | AVB enabled in this community build; does not establish stock signing configuration |
| BOARD_USES_METADATA_PARTITION | true | Community build declares metadata partition use |

The same file sets TARGET_BOOTLOADER_BOARD_NAME := hanoip and uses the Hanoip Motorola kernel configuration extensions. It includes proprietary vendor configuration files that are not part of the public device-tree repository.

### Important distinction about DTB placement

AOSP's generic documentation describes the v3 boot-image layout as moving the DTB out of the boot image header and into the vendor-boot image. AOSP build logic also routes DTB arguments differently depending on whether a vendor-boot image is being built.

The community Hanoip configuration enables BOARD_INCLUDE_DTB_IN_BOOTIMG and declares a vendor-boot partition. The variable name alone is therefore not enough to assert the final physical placement. The exact Android build-system revision, target image set, and generated image metadata would be needed to resolve that detail. No generated images were inspected here.

## 5. Android header versions: generic facts, not stock-device proof

AOSP documents these relevant rules:

- Android 12 devices may use boot-image header version 3 or version 4.
- Header v3 removes the legacy second-stage field and does not carry a DTB field in the boot-image header; the DTB is handled through vendor-boot.
- Header v4 adds a boot signature field and is paired with vendor-boot v4, which supports multiple vendor ramdisk fragments and bootconfig.
- The Android build/bootloader must parse the header version and associated image structure correctly.

Consequently:
- The handset's Android 12 version does not prove it uses header v4.
- The community Hanoip build's explicit header-v3 setting is evidence about that build only.
- The exact stock XT2135-2 firmware header version is **unknown** from the sources inspected.
- The stock boot/vendor_boot/dtbo partition layout, partition slots, and image payloads are **not confirmed** by the kernel source tag.

## 6. Boot-chain stages: what remains unknown

The public Motorola kernel source is the Linux kernel source, not the complete proprietary boot firmware source. The sources inspected in this stage do not establish the exact XT2135-2 sequence of boot ROM, Qualcomm early boot stages, Motorola bootloader stages, verified-boot decision points, partition selection, or kernel entry contract.

Do not fill these gaps by copying a generic Qualcomm boot-chain diagram and labeling it as confirmed for this handset. Device-specific boot firmware, fuses, configuration, and firmware release details can affect the chain.

| Question | Status |
|---|---|
| Exact proprietary boot-stage sequence on XT2135-2 | Unknown |
| Exact stock boot-image header version | Unknown |
| Exact stock vendor-boot header version and contents | Unknown |
| Exact stock partition names, sizes, and slot arrangement | Unknown |
| Exact stock DTB/DTBO selection and handoff | Unknown |
| Exact stock AVB chain, keys, rollback metadata, and policy | Unknown |
| Runtime device tree passed to the kernel | Unknown |
| Kernel entry registers/state and exact handoff contract | Unknown |
| Whether stock boot firmware can provide Apple's XNU boot_args contract | Not established |

## 7. Implications for XNU / Darwin

Stage B.3 documented that the inspected XNU candidate's ARM initialization expects an Apple-style ARM64 boot_args structure. It includes values such as virtual and physical kernel bases, memory size, top of kernel data, device-tree pointer/length, and command line. The inspected XNU initialization also looks for /chosen/dram-base and /chosen/dram-size.

Android boot-image headers describe a kernel payload, ramdisk, and (depending on version and companion image) vendor ramdisk and device-tree data. They do not themselves provide the complete Apple boot_args structure or prove the required XNU device-tree properties are present.

Therefore, even if an ARM64 kernel image is present in an Android boot-image container, that is not evidence that XNU can be entered correctly. A compatible platform handoff would need to be established independently. This report does not propose or describe a device-side method to change that handoff.

## 8. Compatibility and evidence matrix

| Topic | Evidence | Status for stock XT2135-2 |
|---|---|---|
| Motorola Linux kernel source for named firmware build | Exact Motorola tag and issue #574 | Strongly associated |
| Kernel output format in source tree | ARM64 Makefiles | Confirmed source behavior |
| Hanoip board DTBO variants | Exact Motorola DTS Makefile | Confirmed source behavior |
| Android boot header v3 in community Hanoip tree | Community BoardConfig.mk | Confirmed for that community build only |
| Android boot header version in stock firmware | No stock image metadata inspected | Unknown |
| Stock vendor_boot and DTBO layout | Community configuration only | Unknown |
| Stock proprietary boot chain | No device-specific boot-firmware source inspected | Unknown |
| Android-to-XNU boot_args compatibility | XNU source requirements compared with Android formats | Not established |

## 9. Next research items

The next safe research work should remain on public source and documentation:
1. Identify authoritative, model/build-specific documentation for the XT2135-2 boot firmware and partition metadata, if any exists.
2. Inspect the exact Android build-system revision used by the community tree to resolve how its DTB inclusion setting is applied with header v3 and vendor-boot.
3. Compare the XNU candidate's complete ARM64 entry assumptions with documented generic ARM64 boot protocol requirements, without assuming Motorola implements them.
4. Keep the stock firmware header version, runtime device tree, boot chain, and exact board overlay marked unknown until direct, reliable evidence exists.

## 10. Safety and scope

This stage is a documentation exercise only. No instructions are provided for unlocking, flashing, repacking, signing, selecting, or booting images. No phone-side changes are part of Stage B.4.

## 11. References

- Motorola exact kernel source tag: https://github.com/MotorolaMobilityLLC/kernel-msm/tree/MMI-S2RIS32.32-20-9-9-2-3
- Motorola source request #574: https://github.com/MotorolaMobilityLLC/kernel-msm/issues/574
- Motorola ARM64 boot Makefile: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/Makefile
- Motorola ARM64 architecture Makefile: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/Makefile
- Motorola Hanoip DTS Makefile: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/Makefile
- Motorola Hanoip kernel config: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/configs/vendor/ext_config/moto-sdmmagpie-hanoip.config
- Community Hanoip BoardConfig: https://github.com/motorola-sm6150-devs/device_motorola_hanoip/blob/lineage-20/BoardConfig.mk
- AOSP boot-image header: https://source.android.com/docs/core/architecture/bootloader/boot-image-header
- AOSP DTB image format: https://source.android.com/docs/core/architecture/bootloader/dtb-images
- AOSP mkbootimg header definitions: https://android.googlesource.com/platform/system/tools/mkbootimg/+/refs/heads/main/include/bootimg/bootimg.h
- AOSP Android build logic: https://android.googlesource.com/platform/build/+/refs/heads/master/core/Makefile
- XNU ARM64 boot arguments: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/pexpert/arm64/boot.h
- XNU ARM initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm/arm_init.c
