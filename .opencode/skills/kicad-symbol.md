# Skill: KiCad Symbol Creation

Create KiCad symbol library files (.kicad_sym) for the ngin-link project.

## Global library location

Symbols are stored in the global library repo (`libs-kicad`), NOT in this project. The repo is at `git@github.com:alanmonu12/libs-kicad.git` and the local path is `${KICAD_USER_LIBS}`.

- Symbol files go in: `${KICAD_USER_LIBS}/symbols/<Category>_ARR.kicad_sym`
- Naming convention: all libraries use suffix `_ARR` (e.g. `Transceiver_ARR.kicad_sym`, `MCU_ARR.kicad_sym`)
- After creating a new library, update:
  1. `libs-kicad/configuration/sym-lib-table` (reference copy)
  2. `~/.config/kicad/10.0/sym-lib-table` (local KiCad installation — this is the one KiCad reads)
- Commit and push changes to the `libs-kicad` repo

## S-expression format (KiCad 10.x)

All symbols must use this header:

```
(kicad_symbol_lib
    (version 20251024)
    (generator "kicad_symbol_editor")
    (generator_version "10.0")
```

### Symbol structure

```
(symbol "COMPONENT_NAME"
    (pin_names (offset 1.016))
    (exclude_from_sim no)
    (in_bom yes)
    (on_board yes)
    (in_pos_files yes)
    (duplicate_pin_numbers_are_jumpers no)
    (property "Reference" "U" ...)
    (property "Value" "COMPONENT_NAME" ...)
    (property "Footprint" "..." ...)
    (property "Datasheet" "..." ...)
    (property "Description" "..." ...)
    (property "Manufacturer" "..." ...)
    (property "Part number" "..." ...)
    (property "Supplier 1" "..." ...)
    (property "Supplier 2" "..." ...)
    ... component-type-specific properties ...
    (symbol "COMPONENT_NAME_1_0"     ← Graphics body
        (rectangle ...)
    )
    (symbol "COMPONENT_NAME_1_1"     ← Alternative: pins + body in same unit
        (rectangle ...)
        (pin ...)
    )
    (embedded_fonts no)
)
```

### Property format (KiCad 10)

Properties must include KiCad 10 fields:

```
(property "Reference" "U"
    (at <x> <y> <rot>)
    (show_name no)
    (do_not_autoplace no)
    (effects
        (font (size 1.27 1.27))
        (justify left bottom)
    )
)
```

Hidden properties add `(hide yes)` inside `(effects ...)`.

### Required properties (all symbols)

| Property | Visible | Notes |
|----------|---------|-------|
| Reference | Yes | U for ICs, J for connectors, R for resistors, etc. |
| Value | Yes | Component name (e.g. SN65HVD230) |
| Footprint | Hidden | Format: `Library_ARR:FootprintName` |
| Datasheet | Hidden | URL to datasheet PDF |
| Description | Hidden | Brief description with key specs |
| Manufacturer | Hidden | Full manufacturer name (e.g. "Texas Instruments") |
| Part number | Hidden | Manufacturer orderable part number (e.g. "SN65HVD230DR") |
| Package | Hidden | Package designation (e.g. "SOIC-8 3.9x4.9mm") |
| Supplier 1 | Hidden | URL to supplier page (empty initially, filled later) |
| Supplier 2 | Hidden | URL to alternative supplier (empty initially, filled later) |

### Component-type-specific properties

Analyze the component type and add relevant technical properties as hidden fields. Always use descriptive names with units where applicable.

**ICs / Transceivers / Regulators:**
- `Supply voltage` — e.g. "3.3V" or "1.8V to 5.5V"
- `Temperature` — operating range, e.g. "-40C to 85C"

**Capacitors / Resistors:**
- `Value` — already in Value field, but add `Tolerance` and `Voltage rating` as hidden properties
- e.g. `Tolerance`: "1%", `Voltage rating`: "25V"

**Diodes / LEDs:**
- `Vf` — forward voltage, e.g. "0.7V" or "2.1V"
- `If` — forward current, e.g. "20mA"
- `Vr` — reverse voltage for Zener/TVS, e.g. "5.1V"

**Crystals / Oscillators:**
- `Frequency` — e.g. "8MHz" or "16MHz"
- `Load capacitance` — e.g. "20pF"
- `Tolerance` — e.g. "50ppm"

**Connectors:**
- `Contacts` — number of positions, e.g. "4"
- `Pitch` — e.g. "2.0mm" or "1.25mm"
- `Current rating` — e.g. "3A"

**Ferrite beads / Inductors:**
- `Impedance` — e.g. "600R @ 100MHz"
- `Current rating` — e.g. "200mA"

**TVS / Protection:**
- `Vrwm` — working voltage, e.g. "5V" or "3.3V"
- `Vclamp` — clamping voltage, e.g. "12V"

**MCUs / Processors:**
- `Core` — e.g. "Cortex-M4F"
- `Flash` — e.g. "512KB"
- `RAM` — e.g. "128KB"
- `Frequency` — e.g. "180MHz"
- `Supply voltage` — e.g. "1.7V to 3.6V"

### Pin conventions

**All pins use `passive` type.** Do NOT use `input`, `output`, `bidirectional`, `power_in`, or `power_out`. This avoids ERC errors from pin type conflicts and follows the project convention.

```
(pin passive line
    (at <x> <y> <angle>)
    (length 5.08)
    (name "PIN_NAME"
        (effects (font (size 1.016 1.016)))
    )
    (number "1"
        (effects (font (size 1.016 1.016)))
    )
)
```

- Pin shape: always `line`
- Pin name font: `(size 1.016 1.016)`
- Pin number font: `(size 1.016 1.016)`
- Pin length: `5.08` (standard)
- Pin numbers must match the datasheet exactly

### Pin layout for ICs

Place pins logically:
- **Left side** (angle 0): MCU-side signals (TX, RX, control)
- **Right side** (angle 180): Bus-side signals (CANH, CANL, etc.)
- **Top** (angle 270): Power pins (VCC)
- **Bottom** (angle 90): Ground pins (GND)

### Body graphics

Use a single rectangle for the IC body:

```
(rectangle
    (start -7.62 5.08)
    (end 7.62 -5.08)
    (stroke (width 0.254) (type solid))
    (fill (type none))
)
```

## Workflow

1. Check if the library file already exists in `${KICAD_USER_LIBS}/symbols/<Category>_ARR.kicad_sym`
2. If it doesn't exist, create it with the KiCad 10 header
3. If a matching category doesn't exist, create a new `_ARR` library file
4. Define the symbol with all pins as `passive`, proper pin numbers, and logical pin placement
5. Add all required properties (Reference, Value, Footprint, Datasheet, Description, Manufacturer, Part number, Package, Supplier 1, Supplier 2) plus component-type-specific properties
6. Run `kicad-cli sym upgrade <file>` to validate (should say "no se actualizó" if format is correct)
7. Update `libs-kicad/configuration/sym-lib-table` with new category if needed
8. Update `~/.config/kicad/10.0/sym-lib-table` with new entry if needed
9. Commit and push to `libs-kicad` repo

## Multi-unit symbols (MCUs, complex ICs)

For ICs with many pins (MCUs, processors), use a **multi-unit symbol** to separate signal pins from power pins into independent units (Unit A, Unit B). This makes schematic routing much cleaner.

### How KiCad unit numbering works

The sub-symbol name format is `SYMBOLNAME_UNIT_CONVERT`:

- **UNIT** determines which unit the element belongs to:
  - `0` = **common** — shown in ALL units (shared graphics, never pins)
  - `1` = **Unit A** — signal pins (GPIOs, peripherals)
  - `2` = **Unit B** — power pins (VDD, GND, VDDA, VCAP)
  - `3`, `4`, etc. = additional units if needed
- **CONVERT** determines the body style:
  - `0` = normal (default)
  - `1` = De Morgan alternate style (rarely used)

### Required structure for a 2-unit symbol

```
(kicad_symbol_lib ...
  (symbol "STM32F446RET6"
    ... properties ...
    (symbol "STM32F446RET6_0_0"     ← COMMON: shared graphics only
      (rectangle ...)                ← body rectangle (appears in ALL units)
      NO PINS HERE
    )
    (symbol "STM32F446RET6_1_0"     ← UNIT A: signal pins (no graphics)
      (pin passive line ... NRST)
      (pin passive line ... PA0)
      ... all GPIO/signal pins ...
    )
    (symbol "STM32F446RET6_2_0"     ← UNIT B: power pins + own rectangle
      (rectangle ...)                ← body rectangle for power unit
      (pin passive line ... VDD)
      (pin passive line ... VSS)
      ... all power pins ...
    )
    (embedded_fonts no)
  )
)
```

### Critical rules for multi-unit symbols

1. **Unit 0 (`_0_0`) must NEVER contain pins** — only shared graphics (rectangle). Pins in unit 0 appear in every unit, which causes duplicate pin errors.

2. **Each unit > 0 must have its own body rectangle** — Units without a rectangle will display as just floating pins. Either:
   - Put the rectangle in `_0_0` (common, appears in all units), OR
   - Put the rectangle in each unit's sub-symbol (`_2_0` for Unit B)

3. **Every pin number must appear exactly once** across all units. Never duplicate pin numbers between units.

4. **Signal pins go in Unit 1 (`_1_0`)**, **Power pins go in Unit 2 (`_2_0`)**.

5. **For simple ICs (transceivers, regulators, etc.)**: Use a single-unit symbol with `_1_0` containing both rectangle and pins. No need for multi-unit.

### Simple IC (single unit) structure

```
(symbol "SN65HVD230"
  ... properties ...
  (symbol "SN65HVD230_1_0"   ← Unit 1: rectangle + all pins
    (rectangle ...)
    (pin ... D)
    (pin ... GND)
    (pin ... VCC)
    ...
  )
  (embedded_fonts no)
)
```

For simple ICs, `_1_0` is fine — just one unit with graphics and pins together.

### Footprint reference format

Use the global library naming: `Package_SO_ARR:SOIC-8_3.9x4.9mm_P1.27mm`

Common footprint libraries:
- `Package_QFP_ARR` — QFP packages
- `Package_SO_ARR` — SOIC, SOP packages
- `Package_DFN_QFN_ARR` — QFN packages
- `Package_SOD_ARR` — SOD packages
- `Resistor_SMD_ARR` — SMD resistor footprints
- `Capacitor_SMD_ARR` — SMD capacitor footprints
- `Diode_SMD_ARR` — SMD diode footprints
- `Connector_SMD_ARR` — SMD connectors
- `Connector_THT_ARR` — THT connectors