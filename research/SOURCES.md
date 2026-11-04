# Research sources and license register

This file records candidate references. A link is not an endorsement, proof of compatibility, or permission to copy code. Review each repository's current license, notices, provenance, and redistribution terms before reuse.

## Candidate references

| Project / source | Why it may help | Required checks |
|---|---|---|
| [HakiOS](https://github.com/oosexgang/hakios-xnu-kernel) | Public XNU-on-Android research project and architectural comparison | Verify claims against source, documentation, exact revisions, and reproducible evidence; do not assume its drivers fit hanoip |
| [HakiOS project guide](https://hackios-web.vercel.app/en/guide) | Explains its XNU baseline, loader concepts, and device-specific project layout | Treat as project documentation, not independent proof of Moto G60 compatibility |
| [HakiOS license inventory](https://github.com/oosexgang/hakios-xnu-kernel/blob/main/THIRD_PARTY_LICENSES.md) | Records HakiOS's declared third-party source and license inventory | Independently inspect original upstream licenses and notices before reuse |
| [Motorola exact Android 12 kernel source tag](https://github.com/MotorolaMobilityLLC/kernel-msm/releases/tag/MMI-S2RIS32.32-20-9-9-2-3) | Official Motorola kernel source for the owner's exact firmware build | Keep tag and peeled commit IDs; verify all notices; source availability does not imply XNU compatibility |
| [Motorola source request #574](https://github.com/MotorolaMobilityLLC/kernel-msm/issues/574) | Motorola response explicitly links the exact source tag | Evidence of source publication, not proof that installed binary is identical |
| [Hanoip Android device tree](https://github.com/motorola-sm6150-devs/device_motorola_hanoip) | Community device configuration for Moto G40 Fusion / Moto G60 | Verify branch, commit, exact variant and firmware applicability |
| [XNU source](https://github.com/apple-oss-distributions/xnu) | Public source for Apple's XNU kernel | Review source revision, Apple notices/licenses, build dependencies, and platform-specific assumptions |
| [XNU README](https://github.com/apple-oss-distributions/xnu/blob/main/README.md) | Upstream architecture overview and source-tree guide | ARM64 architecture support does not imply a ready-made Qualcomm board port |
| [Darwin source releases](https://opensource.apple.com/) | Public Apple open-source components | Confirm component license and release compatibility |

## Research log

### XNU baseline candidate

- **Date:** 2026-11-04
- **Source and revision:** Apple `apple-oss-distributions/xnu`; candidate tag `xnu-8796.121.2`, identified in HakiOS public documentation.
- **Claim or code examined:** Apple XNU README architecture overview; HakiOS README, guide, changelog, and third-party license register.
- **What it establishes:** Apple documents ARM64 support in XNU. HakiOS identifies `xnu-8796.121.2` as its base and documents a separate loader, drivers, and device configuration.
- **What it does not establish:** The tag has not yet been pinned to an immutable Apple commit in this project; no Moto G60 XNU port, build, or boot has been demonstrated. HakiOS's POCO X5 Pro target is a different device.
- **License / redistribution notes:** HakiOS lists its XNU modifications as APSL 2.0. HanoipOS must independently inspect Apple's original source license files and retain all required notices before reuse.
- **Follow-up:** Verify the exact upstream tag/commit and license, then locate the exact Moto G60 kernel and device-tree source revisions matching model XT2135-2 and build S2RIS32.32-20-9-9-2-3.

### Moto G60 Android kernel baseline

- **Date:** 2026-11-04
- **Source and revision:** Motorola `MotorolaMobilityLLC/kernel-msm`, tag `MMI-S2RIS32.32-20-9-9-2-3`; tag object `f3db6b21a89490d61f530d40584af8c6c9221cae`; peeled commit `f6673a1302e3b9ca84c54e956ad4075d676fdf59`.
- **Claim or code examined:** Motorola source request #574 and its maintainer response; tagged kernel Makefile, COPYING, ARM64 vendor defconfig and Hanoip-specific config extension.
- **What it establishes:** Motorola explicitly linked the exact source tag for the firmware build reported by the device owner. The source Makefile identifies Linux 4.14.190; the Hanoip-specific config extension enables `CONFIG_HANOIP_DTB=y`.
- **What it does not establish:** It does not establish that XNU can boot on the device, that Linux drivers can be reused by XNU, or that the installed kernel binary is byte-for-byte identical to the source build.
- **License / redistribution notes:** The top-level COPYING contains GNU GPL version 2 terms. Preserve notices and review file-level licensing before reuse. The tag signature is reported as unverified/unsigned.
- **Follow-up:** Trace the Hanoip DTB configuration into the source tree and map board-compatible strings, memory, interrupts, timers, and CPU topology.

### Hanoip Android device-tree reference

- **Date:** 2026-11-04
- **Source and revision:** `motorola-sm6150-devs/device_motorola_hanoip`, branch `lineage-20`, commit `514e0261a117a0c7e7fa3cc3cd44f80819b0b779`.
- **Claim or code examined:** Device tree README and BoardConfig.mk.
- **What it establishes:** The community device tree identifies the Moto G40 Fusion / Moto G60 `hanoip` target and references the `sdmsteppe-perf_defconfig`, `moto-sdmsteppe.config`, and `moto-sdmmagpie-hanoip.config` kernel configuration files.
- **What it does not establish:** This is not an official Motorola device tree and does not prove exact match with the owner's currently installed build.
- **License / redistribution notes:** BoardConfig.mk declares Apache-2.0. Verify licensing of each file before reuse.
- **Follow-up:** Compare this device tree against Motorola's exact source tag and identify any differences.

## Important

Do not assume a device with the same SoC is interchangeable with the Moto G60. Board design, boot firmware, display, touch, power management, peripheral wiring, and vendor firmware can differ substantially. Do not commit proprietary Apple system images, signing keys, device secrets, or firmware without redistribution rights.


### Stage B — Hanoip ARM64 platform requirements

- **Date:** 2026-11-04
- **Motorola source:** `MotorolaMobilityLLC/kernel-msm`, tag `MMI-S2RIS32.32-20-9-9-2-3`, peeled commit `f6673a1302e3b9ca84c54e956ad4075d676fdf59`.
- **Files examined:** `sdmmagpie.dtsi`, `sdmmagpie-moto-base.dtsi`, `sdmmagpie-hanoi-base.dts`, Hanoip EVT1/DVT1/DVT2 overlays, `sdmsteppe-perf_defconfig`, `moto-sdmmagpie-hanoip.config`, and community Hanoip `BoardConfig.mk`.
- **What it establishes:** Eight ARM64 CPU entries, PSCI CPU enable-method declarations, two CPU power-management groups (CPU0–5 and CPU6–7), GICv3 and Qualcomm PDC nodes, an ARMv8 architectural timer node, reserved-memory regions, and multiple Hanoip board-revision overlays.
- **XNU comparison:** Candidate `xnu-8796.121.2` ARM initialization calls platform initialization, parses CPU topology, initializes the CPU timebase, and requires `dram-base` and `dram-size` properties under `/chosen`. This makes boot-argument and memory handoff a significant unresolved compatibility gap.
- **What it does not establish:** It does not prove XNU supports this GIC/PDC, PSCI, timer, board revision, or boot protocol. It does not establish the complete runtime DRAM map or the exact board overlay used by the owner's phone.
- **Follow-up:** Identify the exact installed Hanoip board overlay from reliable model/build-specific evidence, then inspect the XNU platform/interrupt/timer code paths and document which interfaces would require new platform code.


### Stage B.2 — XNU hardware compatibility analysis

- **Date:** 2026-11-04
- **XNU revision:** `xnu-8796.121.2`.
- **Files examined:** `pexpert/arm/pe_fiq.c`, `pexpert/arm/pe_identify_machine.c`, `osfmk/arm64/cpu.c`, `osfmk/arm/arm_init.c`, and `pexpert/pexpert/arm64/VMAPPLE.h`.
- **GIC finding:** XNU's GICv3 FIQ path looks for `/arm-io/gic`, maps distributor/redistributor regions, and enables PPI 27. Hanoip's Linux DTS exposes a GICv3 node named `interrupt-controller@17a00000` and a separate Qualcomm PDC. This is architectural overlap, not drop-in compatibility.
- **Timer finding:** XNU's inspected timebase parser reads `timebase-frequency` from a CPU node marked `state = "running"` and defaults to 24 MHz in that routine. Hanoip's inspected CPU nodes lack those properties, while the timer node has `clock-frequency = 19200000`. Runtime firmware may modify the tree; that has not been verified.
- **CPU startup finding:** XNU's generic ARM64 `cpu_start()` calls platform hook `PE_cpu_start()`; the inspected generic path does not itself demonstrate a PSCI CPU_ON implementation. Hanoip's Linux DTS specifies `enable-method = "psci"`.
- **PDC finding:** Hanoip's DTS has `qcom,pdc-sdmmagpie`; no corresponding Hanoip-specific PDC path was identified in the inspected XNU sources.
- **Conclusion:** GICv3, timer, PDC, and PSCI integration require platform-specific validation/adaptation. No hardware compatibility or boot success is claimed.
- **Report:** `research/XNU_HARDWARE_COMPATIBILITY.md`.


### Stage B.3 — boot environment and memory handoff

- **Date:** 2026-11-04
- **Motorola evidence:** exact Android 12 kernel tag `MMI-S2RIS32.32-20-9-9-2-3`; inspected `sdmmagpie.dtsi`, Hanoip base DTS, and Hanoip DVT2 overlay.
- **Hanoip DTS:** `/chosen/bootargs` is present; common DTS contains multiple fixed `no-map` reserved-memory regions. Hanoip base adds a reusable secure-display shared DMA pool of `0x08c00000` bytes (140 MiB).
- **Board metadata:** DVT2 overlay labels model `hanoip`, MSM ID `365 0x0`, board ID `0x4C 0xB200`; this does not prove that overlay matches this particular handset.
- **XNU boot contract:** ARM64 `boot_args` includes `virtBase`, `physBase`, `memSize`, `topOfKernelData`, device-tree pointer/length, and command line. `osfmk/arm/arm_init.c` explicitly requires `/chosen/dram-base` and `/chosen/dram-size`, panicking if absent.
- **Assessment:** those XNU-specific DRAM properties are absent from the inspected Hanoip source DTS. Firmware may add runtime properties; the actual runtime tree and boot-argument handoff are unknown.
- **Report:** `research/BOOT_ENVIRONMENT_AND_MEMORY_HANDOFF.md`.

### Stage B.4 — Motorola boot chain and Android boot-image format

- **Date:** 2026-11-04
- **Motorola source:** exact tag MMI-S2RIS32.32-20-9-9-2-3, peeled commit f6673a1302e3b9ca84c54e956ad4075d676fdf59.
- **Files examined:** Motorola ARM64 architecture Makefile, ARM64 boot Makefile, Qualcomm DTS Makefile, and Hanoip-specific kernel config extension.
- **Kernel build findings:** source supports ARM64 Image/compressed outputs and Image-dtb / Image.gz-dtb targets; CONFIG_HANOIP_DTB selects Hanoip EVT1, DVT1, and DVT2 overlays.
- **Community build evidence:** motorola-sm6150-devs/device_motorola_hanoip BoardConfig.mk on lineage-20 sets boot header version 3, enables DTB inclusion in image build configuration, uses a separated DTBO setting, enables AVB in that community build, and declares boot/vendor_boot/DTBO partition sizes. This is not proof of the stock Motorola firmware layout or signing configuration.
- **AOSP format finding:** Android 12 permits boot-image header v3 or v4; Android version alone does not establish the stock image header version. DTB placement depends on the image format and build target.
- **Unknown:** XT2135-2 stock boot header and vendor-boot versions, exact partition layout/slots, proprietary boot-stage sequence, AVB chain/policy, runtime DTB selection, and kernel entry contract.
- **XNU comparison:** Android boot-image metadata does not provide Apple's ARM64 boot_args structure or prove the XNU-required runtime memory properties. Android-to-XNU handoff remains unestablished.
- **Report:** MOTOROLA_BOOT_CHAIN_AND_IMAGE_FORMAT.md.
- **References:** https://github.com/MotorolaMobilityLLC/kernel-msm/issues/574 ; https://github.com/motorola-sm6150-devs/device_motorola_hanoip/blob/lineage-20/BoardConfig.mk ; https://source.android.com/docs/core/architecture/bootloader/boot-image-header ; https://source.android.com/docs/core/architecture/bootloader/dtb-images

### Stage B.5 — ARM64 entry requirements and XNU startup assumptions

- **Date:** 2026-11-04
- **XNU revision:** xnu-8796.121.2.
- **XNU files examined:** pexpert/pexpert/arm64/boot.h, osfmk/arm64/start.s, osfmk/arm/arm_init.c, osfmk/arm64/arm_vm_init.c, osfmk/arm64/cpu.c, and pexpert/arm/pe_init.c.
- **Linux protocol reference:** Linux ARM64 boot documentation, including the primary CPU x0 DTB pointer convention and documented CPU execution-state requirements.
- **XNU entry finding:** XNU start_first_cpu documents x0 as the Apple boot_args pointer. boot_args separately carries virtBase, physBase, memSize, topOfKernelData, deviceTreeP, and deviceTreeLength.
- **Memory finding:** XNU VM initialization consumes the boot_args memory-layout values. XNU arm_init also requires /chosen/dram-base and /chosen/dram-size.
- **Hanoip CPU finding:** Motorola's exact DTS declares enable-method=psci for CPU nodes and a PSCI node with method=smc.
- **CPU-start finding:** inspected XNU cpu_start delegates to PE_cpu_start; no Hanoip-specific PSCI implementation was established.
- **Assessment:** Linux ARM64 x0=DTB convention and XNU x0=boot_args contract are different interfaces. Actual Motorola runtime entry state and any compatible handoff are unknown.
- **Report:** ARM64_ENTRY_AND_XNU_STARTUP.md.
- **References:** https://www.kernel.org/doc/html/v6.16/arch/arm64/booting.html ; https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/start.s ; https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/pexpert/arm64/boot.h ; https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi

### Stage B.6 — XNU platform dependencies

- **Date:** 2026-11-04
- **XNU revision:** xnu-8796.121.2.
- **Motorola source:** MMI-S2RIS32.32-20-9-9-2-3, peeled commit f6673a1302e3b9ca84c54e956ad4075d676fdf59.
- **XNU files examined:** pexpert/arm/pe_fiq.c, pexpert/arm/pe_identify_machine.c, pexpert/arm/pe_init.c, pexpert/pexpert/device_tree.h, pexpert/gen/device_tree.c, osfmk/arm64/cpu.c, osfmk/arm64/machine_routines.c, osfmk/arm/rtclock.c.
- **Motorola files examined:** sdmmagpie.dtsi, sdmmagpie-hanoi-base.dts, and Hanoip overlays.
- **Device-tree finding:** XNU uses generic SecureDT parsing but platform code looks for Apple-specific paths such as /arm-io/gic and an arm-io node/ranges description. Hanoip DTS describes GICv3 as interrupt-controller@17a00000 and Qualcomm PDC separately.
- **CPU finding:** Hanoip DTS declares PSCI 1.0 via SMC; XNU cpu_start delegates to PE_cpu_start. No Hanoip-specific implementation was established.
- **Timer finding:** Hanoip's architected timer node uses four PPIs and clock-frequency=19.2 MHz. XNU pe_identify_machine reads timebase-frequency from a CPU node marked state=running, with a 24 MHz default. Those properties are not present on the inspected static Hanoip CPU nodes.
- **Interrupt finding:** XNU's GICv3 FIQ code is conditional on HAS_GIC_V3 and looks up /arm-io/gic. It configures PPI 27; no mapping to Hanoip's timer PPI declarations or PDC support was established.
- **Report:** XNU_PLATFORM_DEPENDENCIES.md.
- **References:** https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_fiq.c ; https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_identify_machine.c ; https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/cpu.c ; https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi
