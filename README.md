# HanoipOS

**Experimental Darwin/XNU port research for the Motorola Moto G60 (`hanoip`).**

HanoipOS is an independent research project investigating the feasibility of a Darwin/XNU-based environment on the Motorola Moto G60. The project studies public work such as HakiOS for architectural reference; it is not affiliated with HakiOS, Apple, Motorola, or Qualcomm.

> **Status: early research.** This repository does not contain a bootable Moto G60 port. iOS 27.2 booting, hardware support, and application compatibility are research goals—not verified capabilities.

## Project goals

- Identify and document the exact Moto G60 hardware and software variants.
- Study relevant boot architecture, Darwin/XNU, and upstream device support.
- Determine what device-specific kernel and driver work would be needed.
- Track each subsystem with evidence-based compatibility status.
- Establish safe, non-destructive research and testing practices.

## Target device

| Item | Research target |
|---|---|
| Device | Motorola Moto G60 |
| Codename | `hanoip` |
| SoC | Qualcomm Snapdragon 732G (verify per variant) |
| GPU | Qualcomm Adreno 618 (driver support unverified) |
| Port status | Research only; no working port yet |

## Important distinctions

A Darwin/XNU kernel, a Darwin user space, Apple's complete iOS distribution, and an iOS-like interface are different things. A successful kernel boot would not prove that Apple's frameworks, services, applications, Apple ID, or App Store work.

HanoipOS will not claim to be Apple's iOS or promise iOS 27.2 compatibility without reproducible evidence. Do not commit proprietary Apple system images, firmware, signing keys, or other materials without redistribution rights.

## Planned repository layout

- `docs/HARDWARE.md` — device and hardware research
- `docs/ROADMAP.md` — staged milestones
- `docs/COMPATIBILITY.md` — evidence-based feature status
- `research/SOURCES.md` — upstream references and license notes
- `CONTRIBUTING.md` — contribution guidelines

## Development principles

1. Research and verify before implementing.
2. Record exact device variants, source URLs, versions, and test conditions.
3. Preserve the stock operating system and user data during early research.
4. Do not flash or modify partitions without a verified recovery plan.
5. Respect upstream licenses and keep proprietary files out of the repository.
6. Mark a feature as working only after reproducible testing.

## Disclaimer

This is an independent experimental project. All third-party names are used for identification only. There is no guarantee that Darwin/XNU can be ported to this device.
