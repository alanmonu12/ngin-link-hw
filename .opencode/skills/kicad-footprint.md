# Skill: KiCad Footprint Creation

Create KiCad footprint files (.kicad_mod) for the ngin-link project.

## Rules

- Always read existing .kicad_mod files before modifying
- Store project footprints in `lib/footprints/ngin-link.pretty/`
- Use S-expression format exactly as KiCad generates it
- Follow naming convention: `Package_PackageName` (e.g., `LQFP-48_7x7mm_P0.5mm`)
- Include courtyard, fabrication, and silkscreen layers
- Courtyard must have 0.25mm clearance from pads/edges minimum
- Include 3D model reference if available (.step/.wrl)
- Pad numbering must match symbol pin numbers exactly

## PCB Design Rules

- Minimum trace width: 0.2mm (signal), 0.4mm (power)
- Via: 0.3mm drill / 0.6mm pad (signal), 0.5mm drill / 1.0mm pad (power)
- USB differential pair: 90 ohm impedance, matched length
- CAN differential pair: matched length, minimize ground loops
- Decoupling caps placement: as close to IC power pins as possible

## Key Component Footprints

| Component | Package | Notes |
|-----------|---------|-------|
| STM32F446CET6 | LQFP-48 7x7mm | 0.5mm pitch |
| SN65HVD230 | SOIC-8 3.9x4.9mm | 1.27mm pitch, CAN transceiver |
| AP2112K-3.3 | SOT-223 | LDO regulator |
| USB-C receptacle | USB-C 2.0 SMD | Edge-mount |
| 120R termination | 0805 or 1206 | CAN bus termination |
| LEDs | 0805 | Status and error |
| Common-mode choke | 0805 or custom | CAN bus filtering |