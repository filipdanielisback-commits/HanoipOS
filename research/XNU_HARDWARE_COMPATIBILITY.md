# Stage B.2 — XNU hardware compatibility analysis

**Research date:** 2026-11-04  
**Target:** Motorola Moto G60 XT2135-2 (`hanoip`), Android 12 build `S2RIS32.32-20-9-9-2-3`  
**Motorola source:** `MotorolaMobilityLLC/kernel-msm`, tag `MMI-S2RIS32.32-20-9-9-2-3`, peeled commit `f6673a1302e3b9ca84c54e956ad4075d676fdf59`  
**XNU comparison:** Apple XNU tag `xnu-8796.121.2`.

This is a source-level compatibility assessment only. It does not prove that XNU boots on Hanoip, and it does not provide instructions for changing the phone's boot chain or firmware.

## Executive summary

XNU contains ARM64 code for GICv3-related interrupt setup and architectural timer handling, but the inspected implementation is not a ready-made Qualcomm phone platform port. Its GIC setup expects an Apple-style device-tree layout and platform hooks. Hanoip's Linux device tree uses a different layout and includes a Qualcomm PDC controller that has not been matched to an XNU implementation.

The timer description also differs: Hanoip describes an ARMv8 timer and a 19.2 MHz clock-frequency property, while this XNU revision obtains its timebase frequency from a running CPU node's `timebase-frequency` property (otherwise using a 24 MHz default in the inspected function). The Hanoip CPU nodes examined do not declare either `state = "running"` or `timebase-frequency`. The timer node's `clock-frequency` is not the property this XNU routine reads.

## 1. GICv3 interrupt controller

### Hanoip source facts

The Motorola `sdmmagpie.dtsi` describes:
- GICv3 distributor at `0x17a00000`, size `0x10000`;
- redistributor region beginning at `0x17a60000`, size `0x100000`;
- redistributor stride `0x20000`;
- a separate Qualcomm PDC node at `0x0b220000`, compatible with `qcom,pdc-sdmmagpie`.

The GIC node is labelled `intc` and named `interrupt-controller@17a00000`. It is not presented in the Linux DTS as `/arm-io/gic`.

### XNU source facts

In `pexpert/arm/pe_fiq.c`, XNU's `pe_init_fiq()` looks up the device-tree node at the fixed path `/arm-io/gic`. It reads the node's `reg` property, maps the distributor and redistributor, finds the current CPU's redistributor by MPIDR affinity, wakes it, configures Group 0, and enables PPI 27.

The GICv3 code is conditional on `HAS_GIC_V3`. The inspected `VMAPPLE.h` defines that macro for the VMAPPLE platform configuration; that alone does not mean a generic Qualcomm platform configuration enables or correctly integrates this code.

### Compatibility assessment

****Status: Platform adaptation required**** GICv3 is present on both sides at the architectural level, but the XNU path's expected device-tree path and platform assumptions do not match the Hanoip Linux DTS layout. The Qualcomm PDC is an additional controller whose wake/interrupt routing must be understood; no Hanoip-specific PDC implementation was identified in the XNU files searched.

Required research before any implementation:
- identify the correct XNU platform configuration and whether its GICv3 code can be selected without Apple-only assumptions;
- map Hanoip GIC distributor and redistributor data into the structure expected by XNU;
- determine which interrupts are routed through the PDC and how wake events are delivered;
- compare interrupt group, priority, affinity, and per-CPU redistributor behavior.

## 2. ARM architectural timer and timebase

### Hanoip source facts

The Qualcomm DTS declares an `arm,armv8-timer` node with four PPI entries and `clock-frequency = <19200000>`. It also declares a memory-mapped timer block at `0x17c20000` with timer frames and SPI interrupts.

The CPU nodes in the inspected `sdmmagpie.dtsi` declare `device_type`, `compatible`, `reg`, and `enable-method = "psci"`; they do not contain `state = "running"` or `timebase-frequency` in that source file.

### XNU source facts

In `pexpert/arm/pe_identify_machine.c`, XNU iterates the children of `/cpus`, skips entries without `state = "running"`, and reads `timebase-frequency` from the selected CPU node. In the inspected function, the initial fallback is 24 MHz.

In `pexpert/arm/pe_fiq.c`, the GIC setup enables PPI 27 for the timer/FIQ path. The existence of an ARMv8 timer node in the Linux DTS does not prove that its PPI mapping, interrupt group, or frequency interpretation matches this XNU implementation.

### Compatibility assessment

****Status: Timebase handoff mismatch to resolve**** The 19.2 MHz `clock-frequency` property on Hanoip's timer node is not the same property that the inspected XNU timebase parser reads. If the runtime device tree does not add the CPU-node properties XNU expects, the inspected XNU code would retain its 24 MHz fallback. The actual runtime device tree supplied by firmware has not been inspected, so this remains a source-level warning rather than a confirmed runtime failure.

Required research:
- establish the actual counter frequency and whether it is stable across CPU power states;
- verify the architectural timer interrupt IDs and their GIC group/routing against XNU's PPI 27 FIQ path;
- determine whether runtime firmware modifies the CPU nodes before handing the tree to the kernel;
- establish whether XNU's timer setup can be adapted to the Hanoip timer description.

## 3. PSCI and secondary CPU startup

### Hanoip source facts

The eight CPU nodes in the Motorola DTS declare `enable-method = "psci"`. This tells the Linux kernel that PSCI is the firmware-mediated CPU enable method for this platform.

### XNU source facts

In `osfmk/arm64/cpu.c`, XNU's generic `cpu_start()` prepares the secondary CPU's state and calls the platform expert entry point `PE_cpu_start()`. The generic code inspected does not itself show a PSCI `CPU_ON` invocation. The platform-specific implementation behind `PE_cpu_start()` is therefore a required part of the port.

### Compatibility assessment

**<badge color="warning">Platform startup implementation required</badge>** Hanoip's Linux PSCI declaration is not evidence that XNU already knows how to start Hanoip secondary CPUs. The XNU platform layer would need to provide the correct firmware call convention, target affinity values, entry address, context argument, and startup synchronization, after those details are verified from authoritative platform documentation/source.

No CPU-start code or firmware call should be inferred from the Linux `enable-method` property alone.

## 4. Qualcomm PDC

Hanoip's Linux DTS includes a Qualcomm PDC interrupt-controller node. The inspected XNU GIC code handles GIC distributor/redistributor programming and does not, by itself, establish Qualcomm PDC support.

****Status: Unverified; likely new platform work**** Determine the PDC's role in wake interrupts, which sources are routed through it, and whether XNU needs PDC-specific handling or can rely on a simpler GIC-only configuration for an initial non-sleeping research target. Do not assume suspend/resume support.

## 5. Compatibility matrix

| Subsystem | XNU source evidence | Hanoip evidence | Assessment |
|---|---|---|---|
| GICv3 | GICv3-specific FIQ setup exists under `HAS_GIC_V3`; fixed `/arm-io/gic` lookup | GICv3 at 0x17a00000, Linux DTS node named `interrupt-controller@17a00000` | Platform adaptation required |
| Qualcomm PDC | No matching Hanoip PDC support identified in inspected XNU sources | `qcom,pdc-sdmmagpie` node at 0x0b220000 | Unverified; likely platform work |
| ARMv8 timer | XNU timer/FIQ path enables PPI 27 | ARMv8 timer node with four PPI entries; timer clock-frequency 19.2 MHz | Interrupt/frequency mapping unresolved |
| Timebase frequency | Reads `/cpus/*/timebase-frequency` from a CPU marked `state = "running"`; inspected fallback 24 MHz | Base CPU nodes inspected lack those properties; timer node has 19.2 MHz `clock-frequency` | Device-tree/runtime handoff mismatch to resolve |
| Secondary CPU startup | Generic `cpu_start()` calls platform `PE_cpu_start()` | Linux DTS specifies `enable-method = "psci"` | Platform PSCI implementation required |
| CPU topology | XNU parses CPU topology during ARM initialization | Eight CPU nodes with MPIDR-style IDs 0x0–0x700 | Mapping and cluster semantics still need validation |

## 6. Next evidence-gathering tasks

1. Inspect the runtime device-tree format expected from the Motorola boot firmware, without modifying the device.
2. Read the complete XNU platform-expert interface and identify what board-specific hooks must exist for CPU startup, GIC mapping, and timer initialization.
3. Compare Hanoip's PDC and GIC interrupt routing with the ARM GIC architecture requirements.
4. Document the minimum interrupt and timer feature set needed for an initial research-only kernel configuration, keeping power management and suspend/resume out of scope.

## 7. Source references

### Motorola
- Exact kernel tag: https://github.com/MotorolaMobilityLLC/kernel-msm/tree/MMI-S2RIS32.32-20-9-9-2-3
- Qualcomm SoC DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi
- Hanoip base DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie-hanoi-base.dts

### Apple XNU
- GICv3/FIQ setup: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_fiq.c
- Timebase frequency parsing: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_identify_machine.c
- ARM64 CPU startup: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/cpu.c
- ARM64 platform configuration header: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/pexpert/arm64/VMAPPLE.h
- ARM initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm/arm_init.c
