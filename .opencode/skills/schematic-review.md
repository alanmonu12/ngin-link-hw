# Skill: KiCad Schematic Review

Systematic review of KiCad schematic files for correctness and completeness.

## Review Checklist

### Power
- [ ] Every IC has decoupling caps (100nF + 4.7uF per VDD/GND pair)
- [ ] Power flags present (PWR_FLAG on all net segments driven by power symbols)
- [ ] LDO input/output caps present (typically 1uF + 4.7uF for AP2112)
- [ ] SN65HVD230: Vref decoupling cap, split termination on CAN bus
- [ ] VBUS has PTC/fuse protection
- [ ] TVS diodes on USB lines (D+, D-)
- [ ] TVS diodes on CAN lines (CANH, CANL)
- [ ] No floating power pins

### Connections
- [ ] MCU pin assignments match the project pin map in AGENTS.md
- [ ] CAN transceiver connected: MCU CAN1_TX -> TXD, RXD -> MCU CAN1_RX
- [ ] USB connections: PA12 -> D+, PA11 -> D- (with 1.5k pull-up on D+ for FS)
- [ ] LEDs connected: PB0 -> LED -> GND (active-low), PB1 -> LED -> GND (active-low)
- [ ] BOOT0 has pull-down resistor (100k typical)
- [ ] NRST has pull-up resistor (100k typical) + decoupling cap (100nF)

### CAN Bus
- [ ] Common-mode choke on CANH/CANL
- [ ] TVS/protection on CANH/CANL
- [ ] 120 ohm termination resistor with jumper/0R selectable
- [ ] CAN connector includes GND and optional VEXT

### USB-C
- [ ] CC1/CC2 pulldown resistors (5.1k to GND for UFP device)
- [ ] USB-C edge-cut connector if using edge-mount
- [ ] ESD protection on D+/D-

### General
- [ ] No unconnected pins (use NC flag or net labels)
- [ ] Net labels consistent (ALL_CAPS_SNAKE_CASE)
- [ ] All components have valid footprints assigned
- [ ] All components have values filled in
- [ ] ERC (Electrical Rules Check) passes with no errors

## How to Run ERC

```bash
kicad-cli sch erc ngin-link-hw.kicad_sch
```