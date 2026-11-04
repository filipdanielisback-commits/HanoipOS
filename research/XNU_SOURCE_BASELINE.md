# XNU source baseline — Stage A

**Research status:** Candidate selected; source compatibility and exact-device feasibility remain unproven.

**Target:** Motorola Moto G60, model `XT2135-2`, codename `hanoip`, Android 12 build `S2RIS32.32-20-9-9-2-3`.

## 1. Candidate source revision

| Field | Current finding |
|---|---|
| Upstream | Apple OSS Distributions — `apple-oss-distributions/xnu` |
| Candidate tag | `xnu-8796.121.2` |
| Related Darwin version | Darwin 22.5.0 |
| Related Apple OS release | macOS 13.4 / iOS 16.5 source family |
| Architecture evidence | Apple's XNU README states that XNU runs on x86_64 and ARM64 |
| Porting status | Candidate research baseline only; not confirmed compatible with Moto G60 |

### Why this revision?

HakiOS's public project documentation identifies `xnu-8796.121.2` as its XNU base. Its public changelog also describes that revision as the kernel source used with its Darwin 16.5 user-space work. This makes it a useful comparison point for HanoipOS research and a way to understand an existing Android-phone XNU porting effort.

This is **not** proof that HakiOS's patches or drivers can be reused for the Moto G60. HakiOS documents a POCO X5 Pro 5G (`redwood`) as its primary phone. That is a different handset and platform from the Moto G60. HanoipOS must independently map the Moto G60 board, boot environment, peripherals, and firmware dependencies.

The candidate is deliberately not described as the newest XNU. A newer source snapshot is not automatically a better starting point for a first port. We should first understand a documented baseline, then compare newer revisions only when a concrete requirement justifies it.

## 2. What the upstream source establishes

Apple's public XNU README describes XNU as a hybrid kernel combining Mach, FreeBSD-derived components, and I/O Kit. It explicitly lists ARM64 and x86_64 as supported CPU architectures.

The README also identifies the `pexpert` tree as containing platform-specific code, including interrupt handling and atomics. This is important: generic ARM64 support does not provide a ready-made platform implementation for the Snapdragon 732G or the Moto G60 board.

The README's build examples refer to an SDK and include Apple's internal SDK name `macosx.internal`. Therefore, the presence of a public source tree does not establish that HanoipOS has a complete, publicly reproducible cross-build environment for its target.

## 3. Architecture and platform questions to answer

| Area | Evidence so far | HanoipOS status |
|---|---|---|
| CPU instruction-set family | XNU upstream documents ARM64 support | Partial architectural overlap; exact CPU/platform requirements unknown |
| Platform-specific initialization | XNU has platform-specific `pexpert` code | No Moto G60 implementation identified |
| Interrupt controller | Not established for this XNU baseline on hanoip | Unknown |
| Timer and clock sources | Not established | Unknown |
| Memory map and reserved regions | Not established | Unknown |
| SMP / CPU topology | Not established for this target | Unknown |
| Boot handoff | No supported Moto G60 handoff demonstrated | Unknown |
| Device drivers | No Moto G60 XNU driver set established | Unknown |
| Darwin user space | HakiOS describes a separate user-space layer | Not part of the XNU source baseline |

## 4. HakiOS comparison — useful, but not a shortcut

HakiOS is a relevant reference project because it documents:
- the XNU base tag `xnu-8796.121.2`;
- a separate loader and device-specific configuration;
- its own driver work;
- a QEMU-based open-kernel research path;
- a POCO X5 Pro 5G (`redwood`) device target.

HanoipOS may study the public architecture and source organization, while respecting each repository's license and notices. Do not copy drivers or patches without checking their provenance, license, dependencies, and applicability to the Moto G60.

## 5. Licensing and provenance

- Apple publishes XNU source through the Apple OSS Distributions GitHub organization.
- HakiOS's third-party license register identifies its XNU modifications as APSL 2.0 and records the base tag above. This is HakiOS's own license inventory, not a substitute for checking the license and notices in Apple's original source tree.
- Before any source is copied or modified, record the exact upstream URL, tag or commit, file-level license, copyright notices, modifications, and redistribution obligations.
- Do not add proprietary Apple operating-system images, signing material, private keys, or vendor firmware to HanoipOS.

## 6. Source references

1. Apple XNU repository: https://github.com/apple-oss-distributions/xnu
2. Apple XNU README (architecture, source tree, build overview): https://github.com/apple-oss-distributions/xnu/blob/main/README.md
3. HakiOS repository: https://github.com/oosexgang/hakios-xnu-kernel
4. HakiOS source/license inventory: https://github.com/oosexgang/hakios-xnu-kernel/blob/main/THIRD_PARTY_LICENSES.md
5. HakiOS changelog and source baseline: https://github.com/oosexgang/hakios-xnu-kernel/blob/main/CHANGELOG.md
6. HakiOS guide: https://hackios-web.vercel.app/en/guide

## 7. Stage A checklist

- [x] Identify Apple's maintained public XNU repository.
- [x] Confirm upstream README documents ARM64 support.
- [x] Identify a candidate source tag used by a documented Android-phone XNU research project.
- [x] Record why the candidate is useful and why it does not prove Moto G60 compatibility.
- [x] Record the key source and licensing references.
- [ ] Verify the exact tag and immutable commit directly from Apple's repository before pinning it for any source checkout.
- [ ] Inspect the candidate tag's actual license and notices.
- [ ] Locate and verify the exact Moto G60 Android kernel source and device-tree revisions for the owner's firmware build.
- [ ] Produce a source manifest with immutable commit IDs and per-component license information.

**Stage A result:** Baseline research has started and a candidate is documented. Stage A remains open until the remaining verification items are complete. No XNU build has been attempted, and no device boot or hardware support is claimed.
