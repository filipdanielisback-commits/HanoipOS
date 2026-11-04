# Stage B.6 — XNU platform dependencies: device tree, CPU, timer, and interrupts

**Research date:** 2026-11-04  
**Target:** Motorola Moto G60 XT2135-2 (hanoip)  
**XNU candidate:** xnu-8796.121.2  
**Motorola kernel source:** MMI-S2RIS32.32-20-9-9-2-3  
**Scope:** Public-source comparison only. No phone-side inspection, modification, boot attempt, or hardware experimentation.

## 1. Executive summary

Stage B.6 examined the platform services that the selected XNU source expects after the initial ARM64 entry handoff.

The strongest findings are:

1. **Device tree:** XNU has generic device-tree parsing routines, but its platform code also relies on Apple-specific node paths and property conventions. Hanoip's Linux DTS uses different node names and descriptions for several critical devices.
2. **CPU power management:** Hanoip's Linux DTS declares PSCI with the SMC conduit. XNU's generic ARM64 secondary CPU path calls the platform hook `PE_cpu_start()`; the inspected public XNU source does not provide a Hanoip-specific PSCI implementation.
3. **Timer:** Hanoip describes the architected timer with four PPIs and a 19.2 MHz `clock-frequency`. XNU's machine-identification code looks for `timebase-frequency` on the CPU node marked `state = "running"`, and defaults to 24 MHz if its expected property is not supplied. The inspected static Hanoip CPU nodes do not contain those properties.
4. **Interrupt controller:** Hanoip describes an ARM GICv3 at `interrupt-controller@17a00000` and a Qualcomm PDC. The inspected XNU GICv3 FIQ code looks for `/arm-io/gic` and expects an Apple-style SoC-base description. The inspected Hanoip DTS does not use that path.

These are source-level integration gaps. They do not prove that a port is impossible; they show that the inspected XNU platform assumptions are not a drop-in match for the public Hanoip Linux device tree.

## 2. Sources and revisions

### XNU

- Repository: https://github.com/apple-oss-distributions/xnu
- Tag: `xnu-8796.121.2`
- Files examined:
  - `pexpert/arm/pe_fiq.c`
  - `pexpert/arm/pe_identify_machine.c`
  - `pexpert/arm/pe_init.c`
  - `pexpert/pexpert/device_tree.h`
  - `pexpert/gen/device_tree.c`
  - `osfmk/arm64/cpu.c`
  - `osfmk/arm64/machine_routines.c`
  - `osfmk/arm/rtclock.c`

### Motorola

- Repository: https://github.com/MotorolaMobilityLLC/kernel-msm
- Tag: `MMI-S2RIS32.32-20-9-9-2-3`
- Peeled commit: `f6673a1302e3b9ca84c54e956ad4075d676fdf59`
- Files examined:
  - `arch/arm64/boot/dts/qcom/sdmmagpie.dtsi`
  - `arch/arm64/boot/dts/qcom/sdmmagpie-hanoi-base.dts`
  - Hanoip EVT/DVT overlay source files

The Motorola sources describe the Linux platform. They do not establish the complete runtime device tree or prove which board overlay is selected on a specific XT2135-2 handset.

## 3. Device-tree discovery and schema

### XNU source behavior

XNU provides generic device-tree access through functions such as:
- `SecureDTLookupEntry`
- `SecureDTFindEntry`
- `SecureDTGetProperty`
- `SecureDTInitEntryIterator`

The parser can look up nodes and properties by path, name, or property value. However, platform code must still know which paths and property names to query.

Examples from the selected XNU source:
- `PE_init_platform()` initializes the secure device-tree view from the device-tree pointer and length in `boot_args`.
- `pe_identify_machine()` looks for `/cpus` and a CPU node with `state = "running"`.
- `pe_arm_get_soc_base_phys()` looks for a node named `arm-io` and reads its `ranges` property.
- `pe_init_fiq()` looks up `/arm-io/gic`.

This means generic parser support does not imply that a different device tree automatically matches XNU's platform-specific expectations.

### Hanoip comparison

The inspected Hanoip DTS contains:
- a GIC node named `interrupt-controller@17a00000`;
- a Qualcomm PDC node at `interrupt-controller@b220000`;
- an architected timer node with `compatible = "arm,armv8-timer"`;
- CPU nodes under `/cpus`.

The inspected static DTS does not present the GIC as `/arm-io/gic` and does not show an Apple-style `arm-io` node with the expected `ranges` layout.

**Assessment:** The device tree is structurally useful source material, but XNU's selected platform-specific discovery paths and Hanoip's Linux DTS schema differ. A platform-specific device-tree model would be required; the runtime tree remains unknown.

## 4. CPU power management and secondary CPU startup

### Hanoip evidence

The Motorola DTS describes eight CPU nodes:
- CPU0 through CPU7;
- each CPU has `enable-method = "psci"`;
- the PSCI node uses `compatible = "arm,psci-1.0"` and `method = "smc"`.

This documents the interface expected by the Linux kernel and device tree.

### XNU evidence

In `osfmk/arm64/cpu.c`, XNU's generic `cpu_start()` prepares per-CPU data and then calls:

`PE_cpu_start(cpu_data_ptr->cpu_id, ...)`

The public platform-expert header declares `PE_cpu_start()` as a platform hook. The generic CPU-start path does not itself demonstrate a Hanoip-specific PSCI SMC implementation.

### Assessment

| Question | Finding |
|---|---|
| Does Hanoip's Linux DTS declare PSCI? | Yes; PSCI 1.0, SMC conduit |
| Does generic XNU CPU startup call a platform hook? | Yes; `PE_cpu_start()` |
| Is a Hanoip implementation of that hook present in inspected XNU source? | Not established |
| Is firmware PSCI behavior proven available to XNU in its intended execution state? | Unknown |

The Linux PSCI declaration is useful evidence about the platform firmware interface, but it is not evidence that the selected XNU build can call it correctly.

## 5. Timer and timebase

### Hanoip timer description

The Motorola DTS contains:
- `compatible = "arm,armv8-timer"`;
- four PPI entries: `GIC_PPI 1`, `GIC_PPI 2`, `GIC_PPI 3`, and `GIC_PPI 0`;
- `clock-frequency = <19200000>`.

It also contains a memory-mapped `arm,armv7-timer-mem` block at `0x17c20000`, with additional frame nodes. This is a separate timer-related block in the DTS and should not be conflated with the architected per-CPU timer.

### XNU timebase discovery

In `pexpert/arm/pe_identify_machine.c`, XNU:
1. looks up `/cpus`;
2. iterates CPU nodes;
3. selects a CPU node whose `state` property is `running`;
4. reads `timebase-frequency` from that CPU node if present.

The same function initializes a default timebase frequency of 24,000,000 Hz before reading properties. The Hanoip static CPU nodes inspected in this stage do not show `state = "running"` or `timebase-frequency`. The 19.2 MHz property is on the timer node as `clock-frequency`, not on the selected CPU node under the property name XNU reads.

This is a property-location and naming mismatch in the inspected source trees. Runtime firmware may augment the device tree; that possibility remains unverified.

### XNU timer implementation expectations

The selected XNU source's ARM64 timebase code reads the architectural virtual counter through `CNTVCT_EL0` (or a supported non-speculative variant, depending on compile-time architecture support). The timebase frequency is used to convert ticks to time.

The platform initialization code also contains legacy timer/interrupt-controller discovery that looks for nodes by expected device-tree properties such as `device_type = "timer"` and an interrupt-controller node with a `master` identifier. The Hanoip nodes inspected here use `compatible` strings and a boolean `interrupt-controller` property instead; they do not show those same legacy identifiers.

**Assessment:** The hardware exposes an ARM architected timer in the Linux DTS, but the selected XNU source's frequency discovery and platform timer initialization conventions do not match the static Hanoip description. Exact timer interrupt mapping, timer access permissions, and runtime counter frequency remain to be validated from authoritative platform data.

## 6. Interrupt controller

### Hanoip description

The Hanoip DTS declares:
- GICv3 distributor base `0x17a00000`;
- GICv3 redistributor base `0x17a60000`;
- redistributor region size `0x100000`;
- Qualcomm PDC base `0x0b220000`, connected to the GIC.

The GIC node is named `interrupt-controller@17a00000` and has `compatible = "arm,gic-v3"`.

### XNU GICv3 FIQ path

In `pexpert/arm/pe_fiq.c`, the implementation is compiled only when `HAS_GIC_V3` is enabled. It:
- looks up `/arm-io/gic`;
- reads a `reg` property containing GIC distributor and redistributor regions;
- obtains the SoC physical base from the `arm-io` node;
- maps the GIC distributor and redistributor;
- identifies the per-CPU redistributor using MPIDR affinity;
- configures GIC system-register access and interrupt group state;
- enables PPI 27 for the XNU vtimer/FIQ path.

This source is an implementation of a particular XNU platform convention. It is not by itself a generic GICv3 driver that discovers any GICv3 node by compatible string.

### Comparison and limitations

- Hanoip's GIC node has a different path/name from `/arm-io/gic`.
- The inspected Hanoip source does not show the XNU-expected `arm-io` node and `ranges` property used to derive the SoC base.
- Hanoip also includes a Qualcomm PDC, for which a corresponding XNU platform integration was not identified.
- XNU's FIQ code enables PPI 27; the Hanoip timer DTS lists PPI indices 1, 2, 3, and 0. No mapping between those entries and XNU's timer interrupt setup has been established.
- The presence of GICv3 hardware does not prove the selected XNU target is built with `HAS_GIC_V3` enabled or that the expected platform path is available.

**Assessment:** Architectural controller family overlap exists (GICv3), but XNU's inspected platform discovery and interrupt setup are not a drop-in match for Hanoip's public device tree. PDC support and the timer interrupt routing remain open questions.

## 7. Feasibility matrix

| Subsystem | Hanoip source evidence | XNU expectation in inspected source | Status |
|---|---|---|---|
| Device-tree parser | Standard Linux DTS with Qualcomm nodes | Generic SecureDT parser plus Apple-specific platform paths | Parser exists; platform schema mismatch |
| CPU topology | Eight CPU nodes with MPIDR-style reg values | CPU enumeration and platform-specific CPU setup | Partial architectural overlap |
| CPU power-on | PSCI 1.0 via SMC declared in DTS | `PE_cpu_start()` platform hook | Hanoip implementation unknown |
| Timebase frequency | Timer node has 19.2 MHz `clock-frequency` | Running CPU node's `timebase-frequency`; default 24 MHz | Property mismatch; runtime unknown |
| Architected timer | ARMv8 timer with four PPIs | XNU counter/timebase and platform timer setup | Hardware overlap; integration unproven |
| GIC | GICv3 at documented MMIO regions | GICv3 FIQ code expects `/arm-io/gic` and SoC base | Controller family overlap; DT path mismatch |
| Qualcomm PDC | PDC at `0x0b220000` | No matching Hanoip PDC integration identified | Unknown |
| Interrupt routing | GIC and PDC relationships described in DTS | XNU platform interrupt initialization | Not established |

## 8. Overall conclusion

Stage B.6 identifies several platform-integration gaps between the selected XNU source and the Hanoip Linux device-tree description:

- device-tree node paths and property conventions differ;
- PSCI is declared for Linux, but XNU's platform CPU-start hook is not implemented for Hanoip in the inspected source;
- timer frequency metadata is in a different node/property from the one XNU reads;
- XNU's inspected GICv3 path expects an Apple-style `/arm-io/gic` layout, while Hanoip describes the GIC and PDC using Qualcomm/Linux DTS conventions.

This does not establish that XNU is impossible to adapt. It establishes that the selected source cannot be treated as a generic ARM64 kernel that automatically supports the Moto G60. A Hanoip-specific platform layer and verified hardware/firmware descriptions would be necessary before any compatibility claim could be made.

## 9. Next research items

A safe next stage can consolidate the findings into a platform-service inventory:
1. enumerate the XNU platform hooks and low-level machine routines that need implementations or review;
2. distinguish kernel-core code from Apple-specific platform code and proprietary IOKit drivers;
3. map each Hanoip hardware block to available public Linux documentation and identify where no public XNU driver equivalent exists;
4. prepare a risk/unknowns register before discussing any implementation plan.

## 10. Safety and scope

This is source-level documentation only. No phone-side commands, bootloader changes, image operations, or boot attempts were performed or proposed.

## 11. References

- XNU ARM FIQ/GICv3 code: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_fiq.c
- XNU machine identification and timer discovery: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_identify_machine.c
- XNU platform initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_init.c
- XNU ARM64 CPU startup: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/cpu.c
- XNU ARM clock/timebase: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm/rtclock.c
- XNU device-tree API: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/pexpert/device_tree.h
- Motorola Hanoip SoC DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi
- Motorola Hanoip base DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoi-base.dts
- Apple Platform Expert API overview: https://developer.apple.com/documentation/kernel/driver_support/platform_expert
