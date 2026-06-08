# Skill: KiCad PCB Review

Systematic review of KiCad PCB layout for correctness and manufacturing readiness.

## Review Checklist

### Layout
- [ ] PCB outline defined on Edge.Cuts layer
- [ ] Board dimensions ~50x20mm (dongle format)
- [ ] USB-C connector placed at board edge with proper edge cut
- [ ] CAN connector accessible from opposite end
- [ ] Component placement follows signal flow (USB -> MCU -> CAN)
- [ ] Decoupling caps placed as close as possible to IC power pins
- [ ] Crystal/oscillator close to MCU (if used)

### Routing
- [ ] USB D+/D- routed as differential pair (90 ohm impedance)
- [ ] CAN CANH/CANL routed as differential pair where possible
- [ ] Differential pairs matched in length
- [ ] No 90-degree traces on signal lines (use 45-degree or arcs)
- [ ] Power traces wider than signal traces (min 0.4mm)
- [ ] No traces under crystals/oscillators

### Grounding
- [ ] Ground pour on bottom layer (or both layers)
- [ ] Sufficient vias stitching ground pour to ground nets
- [ ] No floating copper islands
- [ ] Ground return paths short and direct

### Manufacturing
- [ ] DRC (Design Rules Check) passes with zero errors
- [ ] All courtyard overlaps resolved
- [ ] Silkscreen not overlapping pads
- [ ] Reference designators readable and not overlapping
- [ ] Test points accessible (SWD, UART, CAN)
- [ ] All footprints have correct 3D models assigned

## How to Run DRC

```bash
kicad-cli pcb drc ngin-link-hw.kicad_pcb
```

## PCB Stackup (4-layer)

| Layer | Material | Thickness | Copper | Purpose |
|-------|----------|-----------|--------|---------|
| Top | Copper | 0.035mm | 1 oz | Signals, components |
| Prepreg | FR-4 | 0.2mm | - | Impedance control |
| Inner 1 | Copper | 0.035mm | 1 oz | GND plane |
| Core | FR-4 | 0.9mm | - | - |
| Inner 2 | Copper | 0.035mm | 1 oz | Power plane (3.3V) |
| Prepreg | FR-4 | 0.2mm | - | Impedance control |
| Bottom | Copper | 0.035mm | 1 oz | Signals, components |

Total: ~1.6mm