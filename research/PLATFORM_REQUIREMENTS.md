# Stage B — Hanoip ARM64 platform requirements

**Research date:** 2026-11-04  
**Target:** Moto G60 XT2135-2 (`hanoip`), Android 12 build `S2RIS32.32-20-9-9-2-3`  
**Primary hardware source:** Motorola `kernel-msm` tag `MMI-S2RIS32.32-20-9-9-2-3`, peeled commit `f6673a1302e3b9ca84c54e956ad4075d676fdf59`.

This is a read-only source analysis. The findings describe the Android/Linux platform and XNU source requirements; they do not demonstrate an XNU port, build, or boot.

## 1. CPU architecture and topology

### Android/Linux source facts

The Motorola kernel's Qualcomm platform device tree is based on `sdmmagpie.dtsi`, included by `sdmmagpie-moto-base.dtsi` and the Hanoip base DTS.

The CPU node declares:
- Eight CPU entries: `CPU0` through `CPU7`.
- ARMv8-compatible CPU nodes.
- MPIDR-style `reg` identifiers `0x0`, `0x100`, `0x200`, `0x300`, `0x400`, `0x500`, `0x600`, and `0x700`.
- `enable-method = "psci"` on the CPU nodes.

The Qualcomm power-management description groups CPU0–CPU5 in one CPU power cluster and CPU6–CPU7 in a second. The device tree's capacity values also distinguish the first six from the final two. This is consistent with a six-plus-two heterogeneous CPU arrangement, but this document avoids assigning commercial core names based solely on the generic device-tree labels.

The Android device tree's `BoardConfig.mk` declares:
- `TARGET_ARCH := arm64`
- `TARGET_ARCH_VARIANT := armv8-2a-dotprod`
- `TARGET_BOARD_PLATFORM := sm6150`
- `TARGET_BOOTLOADER_BOARD_NAME := hanoip`

The kernel defconfig enables `CONFIG_ARCH_QCOM`, `CONFIG_ARCH_SM6150`, `CONFIG_ARCH_SDMMAGPIE`, and `CONFIG_NR_CPUS=8`.

### XNU comparison

The selected XNU candidate, `xnu-8796.121.2`, calls its platform expert initialization, parses CPU topology, identifies the boot CPU, and initializes the CPU timebase in `osfmk/arm/arm_init.c`.

**Gap:** The Linux device tree's PSCI secondary-CPU startup description is not proof that this XNU revision can use the phone's PSCI implementation. XNU's CPU bring-up, topology parsing, register setup, and inter-processor interrupt behavior would need a dedicated platform analysis.

**Status:** ARM64 and eight CPU descriptions confirmed in the Motorola source; XNU compatibility unknown.

## 2. Memory map and reserved regions

### Android/Linux source facts

The generic Qualcomm `skeleton64.dtsi` declares a placeholder memory node with a zero-sized `reg` value. The Qualcomm SoC device tree separately describes multiple reserved-memory regions, including areas for secure monitor/hypervisor use, modem, camera, video, audio DSP, WLAN, GPU, and other firmware or shared-memory purposes.

The Hanoip base DTS modifies reserved-memory declarations and adds a secure-display shared DMA pool. These are important constraints: a future kernel cannot assume all physical RAM is available as ordinary pageable memory.

The community Android `BoardConfig.mk` also specifies Android kernel image packaging values, including a 4096-byte kernel page size and separate DTBO packaging. These are Android build metadata, not an XNU memory map or boot protocol.

### XNU comparison

In the selected XNU candidate, `osfmk/arm/arm_init.c` reads `dram-base` and `dram-size` properties from the `/chosen` device-tree node and treats their absence as a fatal platform initialization error. XNU also receives boot arguments containing physical and virtual kernel image information.

The Android device-tree source inspected here does not establish that the running Motorola boot chain provides the exact `/chosen` properties and Apple-specific boot arguments expected by this XNU revision.

**Gap:** The complete runtime DRAM map, memory reservations as passed to the booted kernel, and a compatible XNU boot-argument handoff remain unverified. The reserved-memory addresses in source are not a substitute for the complete installed-device memory map.

**Status:** Reserved-memory declarations confirmed; complete runtime DRAM map and XNU-compatible handoff unknown.

## 3. Interrupt controller

### Android/Linux source facts

The Qualcomm device tree declares:
- An ARM GICv3 interrupt controller at `0x17a00000`.
- A GIC redistributor region beginning at `0x17a60000`.
- A Qualcomm PDC interrupt controller at `0x0b220000`, connected to the GIC.
- GIC interrupt bindings in the SoC device-tree source.

The Hanoip overlays include the ARM GIC binding header. The Android kernel defconfig enables GICv3-related support.

### XNU comparison

XNU requires platform interrupt initialization, interrupt routing, and per-CPU interrupt handling. The existence of a standard GICv3-compatible node is encouraging as an architectural reference, but does not prove that this XNU branch has a usable GICv3 driver for this non-Apple platform or that Qualcomm PDC integration is covered.

**Gap:** Verify the exact GICv3 version/features, redistributor discovery, PDC wake routing, interrupt affinity, and XNU interrupt-controller support in the selected revision.

**Status:** GICv3 and Qualcomm PDC nodes confirmed in the Linux DTS; XNU driver/platform support unknown.

## 4. Timers and timebase

### Android/Linux source facts

The Qualcomm DTS declares an `arm,armv8-timer` node with four per-CPU timer interrupt entries and a `clock-frequency` property of 19.2 MHz. It also describes a memory-mapped timer block and frame interrupts.

The kernel configuration enables the normal ARM64/Linux timing facilities, but those Linux configuration options do not establish XNU timebase compatibility.

### XNU comparison

XNU's ARM initialization calls platform timer-register configuration and `cpu_timebase_init`. In this candidate source, timer setup is part of the platform initialization path.

**Gap:** The source-level timer description is not enough to establish the exact counter frequency, access permissions, interrupt routing, stability across CPU power states, or compatibility with XNU's expected timer registers and timebase semantics.

**Status:** ARMv8 timer node and timer interrupt declarations confirmed; XNU timebase compatibility unknown.

## 5. Device-tree and board-revision structure

The Motorola kernel source contains a shared SDMMAGPIE SoC description and a Hanoip-specific base:
- `sdmmagpie.dtsi`
- `sdmmagpie-moto-base.dtsi`
- `sdmmagpie-hanoi-base.dts`

Hanoip-specific overlay files include:
- `sdmmagpie-hanoip-evt1-overlay.dts`
- `sdmmagpie-hanoip-dvt1-overlay.dts`
- `sdmmagpie-hanoip-dvt2-overlay.dts`

The overlays set `model = "hanoip"` and use different Qualcomm board IDs. The kernel Makefile maps these overlays to the Hanoip base DTB when `CONFIG_HANOIP_DTB` is enabled.

The common Hanoip overlay includes panel, touchscreen, sensor, audio, and other device-specific descriptions. This confirms that the board-level device description is composed from a base plus revision-specific overlays; the SoC DTS alone is not the complete phone description.

The community Android `BoardConfig.mk` confirms that its build target is `hanoip`, uses the `sdmsteppe-perf_defconfig` plus Motorola extensions, and packages the DTB/DTBO separately. This community build configuration is a reference, not proof that its exact board revision matches the owner's installed firmware.

**Gap:** The owner's exact board revision / overlay selection is not yet identified from the publicly inspected source. Do not choose EVT1, DVT1, or DVT2 by guesswork.

**Status:** Hanoip base and three board-revision overlays confirmed in source; exact installed overlay unknown.

## 6. Current platform requirements matrix

| Requirement | Evidence from exact Android source | XNU status |
|---|---|---|
| ARM64 execution environment | ARM64 device-tree and Android build target | Architecture-level support only; board support unknown |
| CPU topology | Eight CPUs, IDs 0x0–0x700, two power-management groups | XNU topology/bring-up adaptation unknown |
| Secondary CPU startup | PSCI declared by Linux DTS | XNU PSCI integration unknown |
| Physical memory | Reserved-memory declarations; generic memory placeholder | Complete runtime map and boot handoff unknown |
| Boot arguments | Android boot-image metadata and command line in BoardConfig | Apple XNU boot_args compatibility unknown |
| Interrupt controller | GICv3 plus Qualcomm PDC nodes | XNU GIC/PDC support unknown |
| Timer/timebase | ARMv8 timer node and timer interrupts | XNU timer/timebase integration unknown |
| Board identity | Hanoip EVT1/DVT1/DVT2 overlays with distinct board IDs | Exact installed overlay unknown |

## 7. Key conclusion

The source confirms that Hanoip is an ARM64 Qualcomm platform with a detailed Linux device-tree description, eight CPU entries, GICv3/PDC interrupt infrastructure, and an ARMv8 timer description.

That is useful groundwork, but it does not make the Android DTB directly usable by XNU. The most important architectural blockers now visible are:
1. XNU's platform-specific early initialization and CPU bring-up;
2. the mismatch between Android boot metadata and XNU's expected boot arguments / `/chosen` memory properties;
3. interrupt-controller and timer integration;
4. exact Hanoip board-revision identification.

No XNU build or device boot has been attempted.

## 8. Source references

- Motorola exact kernel source tag: https://github.com/MotorolaMobilityLLC/kernel-msm/tree/MMI-S2RIS32.32-20-9-9-2-3
- Qualcomm SoC DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi
- Hanoip base DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoi-base.dts
- Hanoip DVT1 overlay: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoip-dvt1-overlay.dts
- Hanoip DVT2 overlay: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoip-dvt2-overlay.dts
- Hanoip EVT1 overlay: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoip-evt1-overlay.dts
- Hanoip kernel config extension: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/configs/vendor/ext_config/moto-sdmmagpie-hanoip.config
- Community Hanoip BoardConfig: https://github.com/motorola-sm6150-devs/device_motorola_hanoip/blob/lineage-20/BoardConfig.mk
- XNU candidate ARM initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm/arm_init.c


## Stage B.3 — boot environment and memory handoff

See [BOOT_ENVIRONMENT_AND_MEMORY_HANDOFF.md](BOOT_ENVIRONMENT_AND_MEMORY_HANDOFF.md). Hanoip's published DTS has Linux `/chosen/bootargs` and extensive reserved-memory declarations, but no inspected source DTS entry for XNU's required `/chosen/dram-base` and `/chosen/dram-size`. XNU's ARM64 `boot_args` additionally carries kernel virtual/physical bases, managed memory size, and a flattened device-tree pointer/length. The runtime firmware handoff is not yet known. Board overlay selection for this exact handset also remains open.


## Stage B.3 — boot environment and memory handoff

See [BOOT_ENVIRONMENT_AND_MEMORY_HANDOFF.md](BOOT_ENVIRONMENT_AND_MEMORY_HANDOFF.md). Hanoip's published DTS has Linux `/chosen/bootargs` and extensive reserved-memory declarations, but no inspected source DTS entry for XNU's required `/chosen/dram-base` and `/chosen/dram-size`. XNU's ARM64 `boot_args` also carries kernel virtual/physical bases, managed memory size, and a flattened device-tree pointer/length. The runtime firmware handoff and exact board overlay are not yet known.

## 9. Stage B.4 — boot chain and Android image format

See [MOTOROLA_BOOT_CHAIN_AND_IMAGE_FORMAT.md](MOTOROLA_BOOT_CHAIN_AND_IMAGE_FORMAT.md).

The exact Motorola kernel tag provides ARM64 kernel image build rules and Hanoip-specific DTBO target definitions. The Hanoip kernel configuration enables CONFIG_HANOIP_DTB; the DTS Makefile maps it to EVT1, DVT1, and DVT2 overlays. This confirms source-tree build behavior, not which overlay is selected by the stock firmware on the target handset.

The community Hanoip lineage-20 BoardConfig selects boot header v3, enables DTB inclusion in its image-build configuration, declares a separate DTBO setting, and provides boot/vendor_boot/DTBO image-size settings. It also enables AVB for that community build. These values must not be represented as verified stock XT2135-2 firmware properties.

AOSP documentation allows Android 12 devices to use header v3 or v4. The stock image header version and vendor_boot layout therefore remain unknown. The exact proprietary boot-stage sequence, partition/slot layout, stock AVB policy, runtime DTB selection, and kernel entry contract are also unknown.

**XNU implication:** Android boot-image containers and Android command-line metadata do not themselves provide Apple's ARM64 boot_args structure. The required XNU memory/device-tree handoff remains unproven.

**Status:** Stage B.4 research documented; stock Motorola boot-chain and XNU handoff compatibility remain unknown.
