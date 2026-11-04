# Stage B.5 — ARM64 kernel entry requirements and XNU startup assumptions

**Research date:** 2026-11-04  
**Target:** Motorola Moto G60 XT2135-2 (hanoip)  
**XNU candidate:** xnu-8796.121.2  
**Motorola kernel source:** MMI-S2RIS32.32-20-9-9-2-3  
**Scope:** Static source/documentation comparison only. No device-side inspection, boot testing, bootloader modification, or image handling.

## 1. Executive summary

This stage compared the generic AArch64 Linux kernel entry convention, the Hanoip Linux device-tree declarations, and the selected XNU source's ARM64 startup path.

The most important distinction is the meaning of register x0 at kernel entry:
- The documented AArch64 Linux boot protocol passes the physical address of the flattened device tree (DTB) in x0.
- The inspected XNU assembly's `start_first_cpu` routine documents x0 as a pointer to Apple's `boot_args` structure. That structure separately carries physical/virtual memory parameters and the device-tree pointer and length.

These are different software interfaces. The Linux convention is not a universal rule for every ARM64 operating system, and it does not establish the actual register state used by Motorola's proprietary firmware on XT2135-2.

Hanoip's published Linux DTS declares `enable-method = "psci"` for its CPU nodes and a PSCI node with `method = "smc"`. In the inspected XNU source, generic secondary CPU startup calls the platform hook `PE_cpu_start()`; the inspected code does not demonstrate a Hanoip-specific PSCI implementation.

The source evidence therefore identifies a substantial boot-interface and platform-integration gap. It does not establish whether XNU could be made compatible with this device, and no claim of bootability is made.

## 2. Sources examined

### AArch64 Linux boot protocol

- Linux ARM64 boot documentation: https://www.kernel.org/doc/html/v6.16/arch/arm64/booting.html
- Android common kernel copy of ARM64 boot documentation: https://android.googlesource.com/kernel/common/+/f06a4da3adf252a6ecd37c4ea1c3540db0b27738/Documentation/arm64/booting.rst

These describe the Linux kernel's expected entry environment. They are used here as a reference for the Linux-side convention, not as proof that Motorola's proprietary firmware exposes every condition exactly as documented.

### XNU

Repository: https://github.com/apple-oss-distributions/xnu  
Tag: `xnu-8796.121.2`

Files examined:
- `pexpert/pexpert/arm64/boot.h`
- `osfmk/arm64/start.s`
- `osfmk/arm/arm_init.c`
- `osfmk/arm64/arm_vm_init.c`
- `osfmk/arm64/cpu.c`
- `pexpert/arm/pe_init.c`

### Motorola

Repository: https://github.com/MotorolaMobilityLLC/kernel-msm  
Tag: `MMI-S2RIS32.32-20-9-9-2-3`

Files examined:
- `arch/arm64/boot/dts/qcom/sdmmagpie.dtsi`
- Hanoip base DTS and Hanoip EVT/DVT overlays in the same source tag
- Hanoip-specific kernel config extension
- Hanoip DTS Makefile

## 3. Generic AArch64 Linux entry convention

The Linux ARM64 boot documentation specifies, among other conditions:

| Area | Linux ARM64 documented convention |
|---|---|
| Primary CPU register x0 | Physical address of the DTB in RAM |
| Primary CPU registers x1-x3 | Zero / reserved |
| Exception level | Non-secure EL2 recommended, or non-secure EL1 |
| Interrupts | Masked in PSTATE.DAIF |
| MMU | Off at entry |
| Device tree | Must describe memory and platform devices; DTB placement/alignment constraints apply |
| Timer state | Architected timer frequency and virtual offset must be appropriately initialized |
| CPU coherency | CPUs brought into the kernel must satisfy the documented coherency conditions |
| Secondary CPUs using PSCI | Remain outside the kernel until the kernel issues PSCI CPU_ON calls |

The Linux document also states that firmware/boot software is responsible for initializing RAM and preparing the device tree before entering the Linux kernel.

**Scope caution:** These are Linux kernel entry requirements. They are not a universal AArch64 ABI for all kernels and cannot be treated as Motorola's verified runtime handoff or as XNU's required entry contract.

## 4. XNU ARM64 startup contract visible in source

### 4.1 boot_args structure

In `pexpert/pexpert/arm64/boot.h`, the candidate XNU defines an Apple-specific `boot_args` structure. Relevant fields include:

- `Revision` and `Version`
- `virtBase` and `physBase`
- `memSize`
- `topOfKernelData`
- `deviceTreeP` and `deviceTreeLength`
- `CommandLine`
- `bootFlags`
- `memSizeActual`

It also includes boot-time video information and a machine type.

This is not the same data structure as an Android boot-image header or a Linux DTB pointer. A DTB can be one component of the data XNU consumes, but the XNU structure also carries memory and kernel-layout values.

### 4.2 Register x0 in XNU startup

In `osfmk/arm64/start.s`, the `start_first_cpu` routine is documented as taking x0 as the boot-arguments pointer. It stores this pointer, reads kernel virtual/physical base and memory size fields, establishes early stack and translation-table state, and eventually calls `arm_init()` with the boot-arguments pointer.

This is a direct source-level distinction from the generic Linux convention, where x0 is the physical DTB address.

Do not infer from this comparison that the stock Motorola bootloader passes either interface to an alternative kernel. Its exact runtime handoff has not been inspected or established.

### 4.3 Memory layout requirements

The candidate XNU's `arm_vm_init.c` obtains kernel-managed memory information from `boot_args`:
- `virtBase`
- `physBase`
- `memSize)
- `topOfKernelData`

It uses these values in setting up the kernel's physical/virtual memory mappings and determining the memory range it manages.

The candidate XNU's `osfmk/arm/arm_init.c` also looks up `/chosen/dram-base` and `/chosen/dram-size` in the device tree and panics if either property is missing. The source comments describe these as actual DRAM information reported by iBoot through the device tree.

This matches the Stage B.3 finding: the inspected static Hanoip DTS source did not contain those two XNU-specific `/chosen` properties. Firmware could add properties at runtime, but the runtime device tree remains unknown.

### 4.4 Platform initialization

The XNU `arm_init()` path:
- copies and retains the supplied boot arguments;
- initializes the platform expert using those arguments;
- initializes Apple-specific timer and miscellaneous register configuration hooks;
- parses CPU topology;
- initializes CPU/timebase state;
- later initializes VM and the remaining machine startup.

The platform expert initialization in `pexpert/arm/pe_init.c` uses `deviceTreeP` and `deviceTreeLength` from `boot_args` to initialize its device-tree view.

The inspected source is an Apple-platform-oriented implementation. It does not establish a generic Qualcomm Hanoip platform implementation.

## 5. Hanoip CPU startup declarations

The exact Motorola source tag's `sdmmagpie.dtsi` declares eight CPU nodes. Each CPU node uses:

`enable-method = "psci"`

The same source includes a PSCI node:
- `compatible = "arm,psci-1.0"`
- `method = "smc"`

This is meaningful evidence about the Linux device-tree description and the intended firmware interface for Linux CPU power management.

However:
- A Linux DTS declaration does not prove that the XNU candidate has a matching PSCI implementation.
- It does not prove that the proprietary firmware's PSCI service is accessible under the exact execution state XNU would use.
- It does not establish that XNU's platform hook is implemented for Hanoip.

### XNU secondary CPU path

In the inspected `osfmk/arm64/cpu.c`, `cpu_start()` prepares per-CPU state and invokes:

`PE_cpu_start(cpu_data_ptr->cpu_id, ...)`

The generic path delegates the actual platform-specific CPU start operation to this hook. The inspected code does not show a generic PSCI SMC CPU_ON implementation for Hanoip.

**Assessment:** Hanoip's Linux source declares PSCI/SMC; XNU's Hanoip PSCI integration is unknown and not demonstrated.

## 6. Side-by-side interface comparison

| Requirement / datum | Linux ARM64 convention or Hanoip evidence | XNU candidate expectation | Assessment |
|---|---|---|---|
| x0 at primary entry | Linux convention: physical DTB address | XNU `start_first_cpu`: pointer to `boot_args` | Different interfaces |
| Device tree | DTB passed directly in x0 | DTB pointer and length are fields inside `boot_args` | Translation/packaging gap |
| Memory layout | Described in DTB and reserved regions | `virtBase`, `physBase`, `memSize`, `topOfKernelData`, plus DRAM DT properties | XNU-specific handoff not established |
| DRAM properties | Not present in inspected static Hanoip `/chosen` source | XNU requires `/chosen/dram-base` and `/chosen/dram-size` | Runtime availability unknown |
| CPU enable method | Hanoip DTS says PSCI | XNU generic CPU start calls `PE_cpu_start()` | Platform PSCI support unknown |
| PSCI calling convention | Hanoip DTS says SMC | XNU implementation for Hanoip not found | Unknown |
| Exception level / register state | Linux protocol documents EL2 or EL1, non-secure | XNU assembly uses EL1 system-register setup; complete Motorola-to-XNU entry state unknown | Cannot assert compatibility |
| MMU and caches | Linux protocol specifies its own entry state | XNU startup establishes its own mappings and control state | Exact incoming state requirements need platform-specific validation |
| Timer state | Linux protocol documents architected timer setup | XNU invokes Apple timer configuration hooks and initializes timebase | Hanoip XNU timer setup unknown |

## 7. What can and cannot be concluded

### Confirmed from inspected source

- The generic Linux ARM64 boot convention passes the DTB physical address in x0.
- XNU's `start_first_cpu` routine documents x0 as a pointer to `boot_args`.
- XNU `boot_args` carries kernel memory layout values and a separate DTB pointer/length.
- XNU ARM initialization requires `/chosen/dram-base` and `/chosen/dram-size`.
- Hanoip's static Linux DTS declares PSCI for CPU nodes and a PSCI SMC method.
- The inspected XNU generic secondary CPU startup calls `PE_cpu_start()`.

### Not established

- The exact register contents and exception level used by Motorola's stock firmware at a kernel entry point.
- Whether Motorola's firmware supplies an XNU-compatible `boot_args` structure.
- Whether runtime firmware adds the XNU-required DRAM properties to the device tree.
- Whether the selected XNU source can be adapted to use Hanoip's PSCI service.
- Whether Hanoip's interrupt controller, timer, memory map, or CPU startup can be initialized by XNU.
- Whether a complete Darwin system could run on this device.

## 8. Stage B.5 conclusion

The generic Linux ARM64 entry convention and XNU's startup contract are not interchangeable. In particular, the primary-entry x0 argument differs, and XNU depends on an Apple-specific boot-arguments structure containing memory-layout values beyond a DTB pointer.

Hanoip's PSCI/SMC declarations are promising evidence that the platform firmware exposes a standard CPU power-management interface to its Linux environment, but XNU's platform-specific CPU-start hook has not been shown to use it.

**Stage B.5 is documented as a source-level comparison.** Boot compatibility remains unproven. No kernel has been built or booted, and no changes have been made to the phone.

## 9. Next research questions

A safe continuation is to document the exact XNU platform dependencies behind:
1. `PE_init_platform` and the platform expert's device-tree assumptions;
2. `PE_cpu_start` and the source-level CPU power-management abstraction;
3. timer/timebase initialization and interrupt-controller expectations;
4. the complete physical memory and reserved-region model.

The objective is a feasibility matrix of missing platform services, not a device boot procedure.

## 10. References

- Linux ARM64 boot protocol: https://www.kernel.org/doc/html/v6.16/arch/arm64/booting.html
- Android common kernel ARM64 boot documentation: https://android.googlesource.com/kernel/common/+/f06a4da3adf252a6ecd37c4ea1c3540db0b27738/Documentation/arm64/booting.rst
- XNU ARM64 boot_args: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/pexpert/arm64/boot.h
- XNU ARM64 startup assembly: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/start.s
- XNU ARM initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm/arm_init.c
- XNU ARM64 VM initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/arm_vm_init.c
- XNU ARM64 CPU startup: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/osfmk/arm64/cpu.c
- XNU platform expert initialization: https://github.com/apple-oss-distributions/xnu/blob/xnu-8796.121.2/pexpert/arm/pe_init.c
- Motorola exact kernel source tag: https://github.com/MotorolaMobilityLLC/kernel-msm/tree/MMI-S2RIS32.32-20-9-9-2-3
- Motorola Hanoip SoC DTS: https://github.com/MotorolaMobilityLLC/kernel-msm/blob/MMI-S2RIS32.32-20-9-9-2-3/arch/arm64/boot/dts/qcom/sdmmagpie.dtsi
