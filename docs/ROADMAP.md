# HanoipOS development roadmap

**Current phase: Phase 0 — research and feasibility.**

No bootable Moto G60 port or working driver set currently exists in this repository. Milestones below are plans, not claims of completed functionality.

## Phase 0 — establish facts

- [ ] Confirm exact Moto G60 model number, region, RAM/storage configuration, and firmware build.
- [ ] Collect reliable hardware specifications and map each component to its driver/firmware dependencies.
- [ ] Identify available bootloader, kernel, device-tree, and recovery sources.
- [ ] Record upstream licenses and redistribution constraints.
- [ ] Compare the target device with documented Darwin/XNU-on-Android research platforms.
- [ ] Publish a feasibility report with known blockers.

**Exit criteria:** hardware inventory, source inventory, license notes, and a documented feasibility assessment.

## Phase 1 — architecture design

- [ ] Document the target boot flow at a conceptual level.
- [ ] Identify required interfaces between firmware, boot environment, kernel, and user space.
- [ ] Determine whether existing upstream code is technically and legally reusable.
- [ ] Define a non-destructive test strategy and recovery requirements.

**Exit criteria:** reviewed architecture and test plan. No device flashing is required for this phase.

## Phase 2 — minimal platform investigation

- [ ] Evaluate whether a compatible boot environment can be built from lawful, documented sources.
- [ ] Investigate XNU platform assumptions and required kernel changes.
- [ ] Identify missing platform primitives and hardware dependencies.
- [ ] Keep experiments in emulation or other recoverable environments where possible.

**Exit criteria:** reproducible build artifacts and a written account of what has and has not been demonstrated.

## Phase 3 — device subsystem feasibility

Investigate independently: display, touch, storage, USB, power/battery, Wi-Fi, Bluetooth, audio, sensors, camera, and GPU.

- [ ] Identify hardware blocks and existing driver sources.
- [ ] Determine protocol/API compatibility.
- [ ] Record firmware dependencies and licensing.
- [ ] Define safe tests and expected evidence for each subsystem.

## Phase 4 — user-space and application research

- [ ] Determine the minimum viable Darwin user-space components.
- [ ] Separate open-source components from proprietary Apple components.
- [ ] Investigate graphics and application compatibility.
- [ ] Treat Apple ID, App Store, and proprietary services as unsupported unless independently and lawfully demonstrated.

## Release criteria

A release must state the exact device variant, tested build, boot method, known limitations, and reproducible test evidence. Never label a feature “working” based only on a source-code stub or an unverified screenshot.
