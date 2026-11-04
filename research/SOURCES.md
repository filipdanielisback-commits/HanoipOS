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
