# HanoipOS XNU / Darwin Porting Architecture

**Status:** Research proposal only — no port, kernel build, or boot has been demonstrated.

**Target device:** Motorola Moto G60, model `XT2135-2`, codename `hanoip`, Android 12 build `S2RIS32.32-20-9-9-2-3`.

## 1. Project objective

Investigate whether an open-source XNU kernel and a legally usable Darwin user-space environment could be adapted to the Moto G60. The intended direction is a native Darwin/XNU-based operating-system research project, not an Android launcher or visual imitation of iOS.

This document describes research boundaries and architectural questions. It is not a build guide or a claim that the target is currently supported.

## 2. Terminology and scope

- **XNU** is Apple's hybrid kernel, combining Mach, BSD, and I/O Kit components.
- **Darwin** is the broader open-source operating-system foundation built around XNU and associated user-space components.
- **iOS** is Apple's complete mobile operating system and includes proprietary components, services, frameworks, signing systems, and device-specific software not supplied as a complete open-source distribution.

HanoipOS may study and adapt legally available source code. It must not claim to be Apple's iOS, imply Apple services will work, or assume that proprietary iOS components can be redistributed.

## 3. Proposed conceptual architecture

The following is a target architecture to investigate, not an implemented stack:

```text
+-----------------------------------------------+
| HanoipOS applications / command-line tools    |
+-----------------------------------------------+
| Darwin-compatible user space                  |
| libraries, runtime, launch/services, utilities|
+-----------------------------------------------+
| XNU kernel                                    |
| Mach | BSD | I/O Kit | VM | IPC | scheduling  |
+-----------------------------------------------+
| ARM64 platform adaptation layer (to research) |
| boot handoff | interrupt controller | timers  |
| memory map | clocks | power | device discovery|
+-----------------------------------------------+
| Moto G60 hardware                             |
| Snapdragon 732G / Adreno 618 / peripherals    |
+-----------------------------------------------+
```

Every layer above the hardware is currently a design goal. No existing Android driver should be presumed to work unchanged with XNU.

## 4. Main engineering workstreams

### 4.1 Source and licensing audit

- Identify the exact XNU release and its supported architectures.
- Read the applicable license files for every source component before copying, modifying, or redistributing it.
- Inventory required Darwin user-space components and determine which have public source and compatible redistribution terms.
- Keep proprietary firmware, Apple signing material, private keys, and copyrighted system images out of this repository.
- Record provenance and license obligations for every imported component.

**Exit evidence:** a source manifest with version, origin, license, intended use, and unresolved legal questions.

### 4.2 ARM64 platform feasibility

Research the assumptions XNU makes about its ARM64 execution environment and compare them with publicly documented Snapdragon 732G / Moto G60 platform information.

Questions to resolve include:

- What boot-time machine state and handoff information does the chosen XNU revision require?
- Which memory regions and hardware resources must be described to the kernel?
- What interrupt-controller, timer, and SMP facilities are expected?
- Which cache, address-translation, and exception-level assumptions apply?
- Is there a supported and documented way to provide the required platform description?

Do not assume that ARM64 support in XNU automatically means support for arbitrary ARM64 Android phones.

### 4.3 Boot architecture research

Document the device's existing boot architecture using public documentation and source code. Compare it conceptually with the boot requirements of the selected XNU source revision.

The project must not attempt experimental boot changes during this research phase. Any future boot feasibility assessment must first establish a safe, independently tested stock recovery route and a review process. No unverified boot chain or firmware modification belongs in this phase.

### 4.4 Kernel and hardware interface inventory

For each subsystem, identify the hardware block, existing Android/Linux driver, firmware dependency, interface, and what a native XNU implementation would require.

| Subsystem | Current research status | XNU-side question |
|---|---|---|
| CPU / ARM64 | Snapdragon 732G reported; platform details need source verification | CPU topology, exception state, SMP and power states |
| Memory | Not yet mapped | Physical map, reserved regions, memory controller assumptions |
| Interrupts / timers | Not yet mapped | Interrupt controller and timer integration |
| Display | Exact panel and display path unknown | Display-controller and panel support |
| Touch | Controller unknown | Input device and bus support |
| GPU | Adreno 618 reported | GPU command interface, firmware, memory management and graphics stack |
| Storage | Exact controller/configuration unknown | Block-device and filesystem integration |
| USB | Exact controller details unknown | Host/device controller and USB stack |
| Wi-Fi / Bluetooth | Combo chipset unknown | Bus, firmware, networking and Bluetooth stack |
| Audio | Codec and DSP path unknown | Audio controller, codec and DSP integration |
| Cameras | Sensor and ISP details unknown | Sensor, ISP, media and camera frameworks |
| Power / charging | PMIC and fuel-gauge details unknown | Power management, thermal and battery reporting |
| Sensors | Exact inventory unknown | Sensor buses and user-space interfaces |

A Linux driver is useful evidence about hardware behavior, registers, and firmware dependencies, but it is not automatically reusable as an XNU driver. Any reuse must be technically and legally reviewed.

### 4.5 Darwin user-space feasibility

Create a separate inventory of the user-space pieces needed for a minimal Darwin environment:

- C library and runtime dependencies;
- process and system-call expectations;
- filesystem layout and mount support;
- init / service supervision approach;
- shell and basic command-line utilities;
- logging, diagnostics, and device management;
- networking and security boundaries.

Do not assume that the complete macOS or iOS user space is open source or portable to this device.

### 4.6 Graphics and user experience

Treat graphics as a later, independent workstream. A working kernel does not imply accelerated graphics, compositing, touch interaction, or a mobile interface. First establish whether the display and input hardware can be supported through documented interfaces.

## 5. Staged research plan and exit criteria

### Stage A — Establish source baseline

- Identify Apple's maintained public XNU source repository.
- Confirm the upstream architecture claims from its README.
- Document a candidate source revision and explain why it is useful as a comparison baseline.
- Verify the exact upstream tag and immutable commit directly before pinning it for any source checkout.
- Inspect the candidate tag's actual license and notices.
- Create a source and license manifest.
- Locate exact Moto G60 kernel and device-tree sources for the reported build, if publicly available.

**Current result:** A candidate baseline, `xnu-8796.121.2`, is documented in [XNU_SOURCE_BASELINE.md](../research/XNU_SOURCE_BASELINE.md). Apple's README documents ARM64 support, and HakiOS identifies this tag as its own base. The tag has not yet been pinned to an immutable Apple commit in this project, and exact Moto G60 source revisions remain to be verified.

**Exit:** immutable, versioned references and a written compatibility gap analysis.

### Stage B — Platform requirements matrix

- Map ARM64 requirements against the available Moto G60 documentation.
- Identify missing information for boot handoff, memory, interrupts, timers, and CPU topology.
- Classify each item as confirmed, likely, unknown, or incompatible.

**Exit:** every critical platform requirement has evidence or an explicit blocker.

### Stage C — Hardware feasibility matrix

- Trace each hardware subsystem to a source-level driver or reliable hardware reference.
- Identify firmware and proprietary dependencies.
- Estimate which subsystems have documented interfaces and which lack a feasible path.

**Exit:** a reviewed subsystem-by-subsystem feasibility report. No claim of device support based solely on Android operation.

### Stage D — User-space and integration design

- Inventory publicly available Darwin components and their licenses.
- Define a minimal user-space target independent of proprietary iOS services.
- Document expected kernel/user-space interfaces.

**Exit:** a legally reviewed, technically coherent minimal-system design.

### Stage E — Separate implementation proposal

Only after the earlier stages pass review should the project propose isolated, non-destructive development experiments. Any device-level experiment would require a documented safety review, verified stock recovery procedure, and explicit maintainer approval. This document does not authorize or describe flashing, bootloader changes, or firmware modification.

## 6. Evidence and status rules

Use these labels consistently:

- **Confirmed:** directly supported by an authoritative source for the exact model or source revision.
- **Likely:** supported by credible but incomplete evidence.
- **Unknown:** not yet established.
- **Incompatible:** a documented requirement conflicts with the target platform.

A source repository existing is not proof that its code builds for this phone. A kernel compiling is not proof that it boots. A kernel booting is not proof that peripherals work. A functioning kernel is not proof that a complete Darwin user space or iOS application environment is available.

## 7. Current project status

| Milestone | Status |
|---|---|
| Exact device model recorded | Confirmed by device owner: XT2135-2 |
| Android version / build recorded | Confirmed by device owner: Android 12, S2RIS32.32-20-9-9-2-3 |
| Apple XNU repository identified | Confirmed: apple-oss-distributions/xnu |
| XNU architecture baseline | Upstream README documents ARM64 and x86_64 |
| Candidate source revision | `xnu-8796.121.2`, documented; immutable commit verification pending |
| Candidate source license audit | Pending direct inspection of source notices |
| Moto G60 kernel/device-tree source revisions | Not yet verified for the exact build |
| XNU ARM64 platform compatibility with hanoip | Unknown |
| Moto G60 boot handoff compatibility | Unknown |
| Moto G60 XNU hardware drivers | Not established |
| Darwin user-space composition | Not established |
| HanoipOS kernel build | Not started |
| HanoipOS boot | Not achieved |

## 8. Initial references

- Apple open-source XNU: https://github.com/apple-oss-distributions/xnu
- Apple XNU README: https://github.com/apple-oss-distributions/xnu/blob/main/README.md
- Candidate baseline research: [XNU_SOURCE_BASELINE.md](../research/XNU_SOURCE_BASELINE.md)
- HanoipOS hardware inventory: [HARDWARE.md](HARDWARE.md)
- HanoipOS compatibility tracker: [COMPATIBILITY.md](COMPATIBILITY.md)
- HanoipOS roadmap: [ROADMAP.md](ROADMAP.md)
- HanoipOS source notes: [SOURCES.md](../research/SOURCES.md)

References are starting points. Verify the revision, platform applicability, and license of each component before relying on it.

## 9. Non-goals and safeguards

- Do not describe HanoipOS as Apple's iOS or imply Apple endorsement.
- Do not promise Apple ID, App Store, iMessage, FaceTime, or proprietary iOS framework compatibility.
- Do not add proprietary Apple system images, firmware, signing keys, or credentials to the repository.
- Do not make changes to the owner's phone as part of this research plan.
- Do not mark a feature as supported without reproducible evidence and exact-device testing.
- Preserve the stock device software and personal data throughout the research phase.
