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

## Unit naming convention

- `SYMBOL_1_0` for when pins and body are in the same unit
- `SYMBOL_0_0` for body graphics only (when using separate units)
- `SYMBOL_1_0` for pins-only unit (when using separate units)
- For simple ICs (8-pin, etc.), put rectangle and pins all in `_1_1`

## Footprint reference format

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