# Compatibility tracker

**Status date:** 2026-11-04  
**Target:** Motorola Moto G60 (`hanoip`)  
**Overall status:** Not booted; research only.

“Unknown” means no verified implementation or test evidence is recorded here. It does not mean the hardware is impossible to support.

| Subsystem | Status | Evidence / next investigation |
|---|---|---|
| Boot environment | Unknown | Document device boot chain and compatible interfaces |
| XNU kernel | Unknown | Assess platform assumptions and required porting work |
| Darwin user space | Unknown | Define minimum user-space requirements |
| Display | Unknown | Identify exact panel and display pipeline |
| Touchscreen | Unknown | Identify controller and input path |
| GPU (Adreno 618) | Unknown | Research driver, firmware, and graphics stack compatibility |
| Wi-Fi | Unknown | Identify chipset, firmware, and networking integration |
| Bluetooth | Unknown | Identify controller and transport |
| Battery / charging | Unknown | Identify fuel gauge and charging IC |
| Audio | Unknown | Identify codec, DSP, and routing |
| Cameras | Unknown | Identify sensors and ISP dependencies |
| Cellular | Unknown | Research modem interfaces and required services |
| Sensors | Unknown | Inventory sensor hardware and interfaces |
| Apple ID / App Store | Unsupported / not investigated | Proprietary service compatibility is not assumed |
| iOS application compatibility | Unknown | Requires separate framework/runtime investigation |

## Evidence standard

A status may be changed to **Working** only when a reproducible test is documented with:
- exact handset model and firmware;
- source revision and build configuration;
- test procedure and observed result;
- logs or other evidence with personal identifiers removed;
- known limitations.

Use **Partial** for incomplete functionality, **Blocked** for a demonstrated blocker, and **Unknown** when evidence is insufficient. Do not infer support from another phone using the same SoC.
