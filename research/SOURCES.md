# Research sources and license register

This file records candidate references. A link is not an endorsement, proof of compatibility, or permission to copy code. Review each repository's current license, notices, provenance, and redistribution terms before reuse.

## Candidate references

| Project / source | Why it may help | Required checks |
|---|---|---|
| [HakiOS](https://github.com/oosexgang/hakios-xnu-kernel) | Public XNU-on-Android research project and architectural comparison | Verify claims against source, documentation, exact revisions, and reproducible evidence; do not assume its drivers fit hanoip |
| [HakiOS project guide](https://hackios-web.vercel.app/en/guide) | Explains its XNU baseline, loader concepts, and device-specific project layout | Treat as project documentation, not independent proof of Moto G60 compatibility |
| [HakiOS license inventory](https://github.com/oosexgang/hakios-xnu-kernel/blob/main/THIRD_PARTY_LICENSES.md) | Records HakiOS's declared third-party source and license inventory | Independently inspect original upstream licenses and notices before reuse |
| [Motorola hanoip device sources](https://github.com/search?q=hanoip+motorola&type=repositories) | Device-specific Android configuration references | Confirm exact device variant, provenance, license, and source revision |
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

## Important

Do not assume a device with the same SoC is interchangeable with the Moto G60. Board design, boot firmware, display, touch, power management, peripheral wiring, and vendor firmware can differ substantially. Do not commit proprietary Apple system images, signing keys, device secrets, or firmware without redistribution rights.
