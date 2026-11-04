# Stage B.3 — Boot environment and memory handoff analysis

**Research date:** 2026-11-04  
**Target:** Moto G60 XT2135-2 (`hanoip`), Android 12 build `S2RIS32.32-20-9-9-2-3`  
**Motorola source:** `MotorolaMobilityLLC/kernel-msm`, tag `MMI-S2RIS32.32-20-9-9-2-3`, commit `f6673a1302e3b9ca84c54e956ad4075d676fdf59`  
**XNU comparison:** `xnu-8796.121.2`.

This is source-level research only. No device configuration, bootloader, partition, or firmware is being changed. The published Linux DTS is not assumed to be identical to the runtime tree supplied by firmware.

## Executive summary

Hanoip's published Linux DTS and XNU's ARM64 boot contract are different interfaces. Hanoip's common DTS has a Linux `/chosen/bootargs` string and extensive reserved-memory declarations. XNU's ARM64 `boot_args` structure additionally carries kernel virtual/physical bases, memory size, a flattened device-tree pointer and length, and command-line data. In addition, the inspected XNU ARM initialization explicitly requires `/chosen/dram-base` and `/chosen/dram-size`; it panics if either is absent.

Neither property appears in the inspected Hanoip source DTS. Firmware may add runtime properties, so this is an unresolved handoff requirement, not proof that they are absent at runtime.

## 1. Hanoip Linux DTS

### Board composition and overlay evidence

The Hanoip base file `sdmmagpie-hanoi-base.dts` includes `sdmmagpie-moto-base.dtsi`. The common SoC description is in `sdmmagpie.dtsi`.

The inspected DVT2 overlay identifies:
- `model = "hanoip"`;
- `qcom,msm-id = <365 0x0>`;
- `qcom,board-id = <0x4C 0xB200>`.

This identifies that overlay's metadata only. It does not prove that DVT2 matches this particular XT2135-2 handset; board revision selection remains unknown.

### /chosen

The common SoC DTS includes:

```dts
chosen {
    bootargs = "rcupdate.rcu_expedited=1 rcu_nocbs=0-7";
};
```

This is a Linux kernel command-line property in the published source. It does not establish the exact runtime command line and is not equivalent to XNU's binary `boot_args` structure. The inspected Hanoip DTS files do not define `dram-base` or `dram-size`.

### Reserved memory

The common DTS declares `reserved-memory` with 64-bit address and size cells. It contains multiple fixed `no-map` regions for firmware and peripheral consumers, including hypervisor/early firmware, shared memory, modem, camera, video, compute/DSP, WLAN and other platform services.

The Hanoip base DTS adds a reusable secure-display shared DMA pool:
- size: `0x08c00000` bytes (140 MiB);
- alignment: 4 MiB;
- allocation range specified in the DTS.

This is a pool description, not a statement of total installed RAM. These reserved regions must not be treated as ordinary free RAM. Their presence in Linux DTS does not automatically reserve equivalent ranges for XNU.

## 2. XNU ARM64 boot contract

### boot_args

In `pexpert/pexpert/arm64/boot.h`, XNU defines a `boot_args` structure with fields including:
- `Revision` and `Version`;
- `virtBase` and `physBase`;
- `memSize` and `topOfKernelData`;
- `deviceTreeP` and `deviceTreeLength`;
- `CommandLine`;
- `bootFlags` and `memSizeActual`.

This is a binary interface passed to the Mach kernel. A Linux DTS `/chosen/bootargs` string does not provide this structure.

### Required DRAM properties

In `osfmk/arm/arm_init.c`, XNU looks up `/chosen`, then reads `dram-base` and `dram-size`. It calls `panic()` if the node or either property is missing, then stores these as actual DRAM base and size.

The source comments describe these values as actual DRAM information reported by Apple's iBoot through the device tree. This is an Apple boot-environment convention, not a generic promise made by Linux device-tree bindings.

### Kernel memory placement

In `osfmk/arm64/arm_vm_init.c`, XNU initializes kernel-managed virtual and physical memory bases from `boot_args.virtBase` and `boot_args.physBase`. The `memSize` and `topOfKernelData` fields are also part of the boot-time memory contract.

Thus, the XNU handoff must reconcile both:
1. kernel image placement and managed memory described by `boot_args`;
2. actual DRAM base and size described under `/chosen`.

Providing only a generic Linux memory node or only a kernel load address does not establish that contract.

## 3. Compatibility matrix

| Area | Hanoip evidence | XNU requirement | Assessment |
|---|---|---|---|
| Boot arguments | Linux DTS command-line string | ARM64 `boot_args` with bases, size, DT pointer/length, command line | Different interfaces; handoff unresolved |
| Device tree | Linux DTS source and overlays | Flattened tree pointer and length in `boot_args`; SecureDT parser | Runtime tree not inspected |
| DRAM base/size | No properties found in inspected Hanoip DTS | `/chosen/dram-base` and `/chosen/dram-size` explicitly required | Critical open requirement |
| Kernel placement | Linux kernel source/config and DTS | `physBase`, `virtBase`, `memSize`, `topOfKernelData` | XNU placement contract not established |
| Reserved ranges | Many fixed `no-map` regions and a secure-display pool | XNU must avoid firmware/peripheral-owned ranges | Full map and equivalent handling unresolved |
| Board overlay | DVT2 metadata includes board ID `0x4C 0xB200` | Device tree must match actual board | Exact overlay unknown |

## 4. Known facts vs unknowns

**Confirmed from checked source**
- Hanoip common DTS has a `/chosen/bootargs` string.
- Hanoip DTS contains many fixed `no-map` reserved-memory regions.
- Hanoip base DTS adds a 140 MiB reusable secure-display shared DMA pool.
- XNU ARM64 `boot_args` includes virtual/physical bases, memory size, kernel-data boundary, and flattened device-tree pointer/length.
- XNU ARM init explicitly requires `/chosen/dram-base` and `/chosen/dram-size`.

**Not established**
- Exact runtime device tree passed by Motorola firmware.
- Whether firmware adds the two DRAM properties or other platform metadata.
- Actual physical DRAM banks, holes, and runtime reservations.
- Which overlay applies to this exact XT2135-2.
- Exact boot-argument structure and entry contract supplied by the Motorola boot chain.
- Whether the XNU memory layout can coexist with secure firmware and peripheral reservations.

## 5. Next research tasks

1. Trace Linux ARM64 device-tree memory discovery and reservation at source level, separating static DTS reservations from runtime firmware additions.
2. Inspect publicly available Motorola boot-image and boot-chain documentation for documented handoff formats and metadata.
3. Inventory every Hanoip reserved-memory entry and its stated consumer; mark dynamic and firmware-owned entries separately.
4. Compare that inventory with XNU's early memory initialization and device-tree parser.
5. Keep board overlay selection open until device-specific evidence exists.

No phone-side probing, bootloader changes, flashing, or partition writes are part of this step.

## 6. Source references

### Motorola
- Exact kernel tag: https://github.com/MotorolaMobilityLLC/kernel-msm/tree/MMI-S2RIS32.32-20-9-9-2-3
- Common SoC DTS and reserved-memory declarations: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi
- Hanoip base DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoi-base.dts
- Hanoip DVT2 overlay: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoip-dvt2-overlay.dts

### Apple XNU
- ARM64 boot arguments: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/pexpert/arm64/boot.h
- Required DRAM properties and ARM init: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm/arm_init.c
- ARM64 virtual/physical memory initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/arm_vm_init.c
