# Moto G60 hardware research

This document is a research inventory, not a confirmed driver-support statement. Verify every value against the exact handset variant and trustworthy sources before relying on it.

## Device identity

| Field | Value | Verification |
|---|---|---|
| Marketing name | Motorola Moto G60 | Confirmed target |
| Device codename | `hanoip` | Verify against device/source tree |
| Exact model number | To be recorded | Not yet verified |
| Region / variant | To be recorded | Not yet verified |
| RAM / storage | To be recorded | Variants may differ |
| Android build / firmware | To be recorded | Obtain from device owner |
| Bootloader state | To be recorded | Do not change during research |

## SoC and major components

| Component | Preliminary identification | Work still required |
|---|---|---|
| SoC | Qualcomm Snapdragon 732G | Verify exact platform and board revision |
| GPU | Adreno 618 | Determine driver architecture, firmware, and API requirements |
| Display | LCD panel; exact panel unknown | Identify panel, display controller, modes, and initialization |
| Touch | Exact controller unknown | Identify controller, bus, firmware, and input protocol |
| Wi-Fi / Bluetooth | Exact combo module unknown | Identify chipset, firmware, transport, and regulatory constraints |
| Storage | Exact configuration unknown | Identify storage type, controller, and supported interfaces |
| Audio | Exact codec/amplifier unknown | Identify codec, routing, DSP, and firmware dependencies |
| Cameras | Multiple camera modules; exact sensors unknown | Identify sensors, ISP path, calibration, and firmware |
| Battery / charging | Exact fuel gauge and charging IC unknown | Identify telemetry and power-management interfaces |
| Sensors | Exact sensor set unknown | Inventory accelerometer, proximity, compass, etc. |

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
