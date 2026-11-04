# Moto G60 hardware research

This document is a research inventory, not a confirmed driver-support statement. Verify every value against the exact handset variant and trustworthy sources before relying on it.

## Device identity

| Field | Value | Verification |
|---|---|---|
| Marketing name | Motorola Moto G60 | Confirmed target |
| Exact model number | `XT2135-2` | Reported from device's About phone screen |
| Device codename | `hanoip` | Corroborated by firmware naming and public device references |
| Region / variant | European / RETEU firmware family | Strongly indicated by the matching XT2135-2 firmware package name; exact sales channel still to be confirmed |
| Android version | Android 12 | Reported from device |
| Build number | `S2RIS32.32-20-9-9-2-3` | Reported from device; matches a published XT2135-2 European firmware package |
| RAM / storage | Not recorded | Ask owner only if needed; variants may differ |
| Bootloader state | Not recorded | Leave unchanged during research |

## SoC and major components

| Component | Preliminary identification | Work still required |
|---|---|---|
| SoC | Qualcomm Snapdragon 732G | Verify exact platform and board revision from device sources |
| GPU | Adreno 618 | Determine driver architecture, firmware, and API requirements |
| Display | LCD panel; exact panel unknown | Identify panel, display controller, modes, and initialization |
| Touch | Exact controller unknown | Identify controller, bus, firmware, and input protocol |
| Wi-Fi / Bluetooth | Exact combo module unknown | Identify chipset, firmware, transport, and regulatory constraints |
| Storage | Exact configuration unknown | Identify storage type, controller, and supported interfaces |
| Audio | Exact codec/amplifier unknown | Identify codec, routing, DSP, and firmware dependencies |
| Cameras | Multiple camera modules; exact sensors unknown | Identify sensors, ISP path, calibration, and firmware |
| Battery / charging | Exact fuel gauge and charging IC unknown | Identify telemetry and power-management interfaces |
| Sensors | Exact sensor set unknown | Inventory accelerometer, proximity, compass, etc. |

## Firmware and kernel-source research

- Motorola's public `kernel-msm` repository contains a request issue naming the exact XT2135-2 Android 12 firmware build family `S2RIS32.32-20-9-9-2-3`. The issue is a lead for source research, not by itself proof that a matching kernel source release has been located.
- A public firmware mirror lists a package named `XT2135-2_HANOIP_RETEU_12_S2RIS32.32-20-9-9-2-3...`, consistent with the model and build reported by the device. Third-party mirrors are corroborating references only; prefer Motorola-published sources for source-code and firmware provenance.

## Evidence collection

For each item, record:
- source URL and retrieval date;
- exact device model and firmware version;
- component identifier or kernel driver name;
- upstream license;
- confidence level (confirmed, likely, unknown);
- unresolved dependencies.

Do not publish serial numbers, IMEIs, account identifiers, private logs, or other personal device data.

## Safety

Early research should be read-only. Do not unlock, flash, repartition, or alter boot firmware as part of hardware identification. Before any future experimental boot, require a tested stock restore path and backups.
