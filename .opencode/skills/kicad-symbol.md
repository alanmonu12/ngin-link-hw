# Skill: KiCad Symbol Creation

Create KiCad symbol library files (.kicad_sym) for the ngin-link project.

## Rules

- Always read existing .kicad_sym files before modifying
- Store project symbols in `lib/symbols/ngin-link.kicad_sym`
- Use S-expression format exactly as KiCad generates it
- Follow pin naming: ALL_CAPS for functions (CAN_TX, USB_DM)
- Use SI suffixes for values (10k, 100n, 4u7)
- Reference designator prefixes: U=ICs, J=connectors, R=resistors, C=capacitors
- No unnecessary comments in S-expressions
- Run `kicad-cli sym verify` after creation if available

## Workflow

1. Read the target .kicad_sym file (or create if it doesn't exist)
2. Define symbol with proper pins, including pin type (input/output/bidirectional/power_in/power_out)
3. Add visible properties: Reference, Value, Footprint, Datasheet
4. Ensure pin numbers match the component datasheet exactly
5. For MCU symbols, group pins by function (power, GPIO, peripheral)

## MCU Pin Mapping Reference (STM32F446CET6 LQFP-48)

| Pin | Name | Function |
|-----|------|----------|
| 1   | VDD  | Power    |
| 2   | PA0  | BOOT0_GPIO |
| 3   | PA1  | GPIO     |
| 4   | PA2  | GPIO     |
| 5   | PA3  | GPIO     |
| 6   | PA4  | GPIO     |
| 7   | PA5  | GPIO     |
| 8   | PA6  | GPIO     |
| 9   | PA7  | GPIO     |
| 10  | PB0  | STATUS_LED |
| 11  | PB1  | ERROR_LED |
| 12  | PB2  | GPIO     |
| 13  | VSSA | Analog GND |
| 14  | VREF+| Analog Ref |
| 15  | VDDA | Analog Power |
| 16  | PB8  | CAN1_RX  |
| 17  | PB9  | CAN1_TX  |
| 18  | BOOT0| Boot mode |
| 19  | VSS  | GND      |
| 20  | VDD  | Power    |
| 21  | PB10 | GPIO     |
| 22  | PB11 | GPIO     |
| 23  | VSS  | GND      |
| 24  | PB12 | GPIO     |
| 25  | PB13 | GPIO     |
| 26  | PB14 | GPIO     |
| 27  | PB15 | GPIO     |
| 28  | PA8  | GPIO     |
| 29  | PA9  | GPIO     |
| 30  | PA10 | GPIO     |
| 31  | PA11 | USB_DM   |
| 32  | PA12 | USB_DP   |
| 33  | VSS  | GND      |
| 34  | VDD  | Power    |
| 35  | PA13 | SWDIO    |
| 36  | PA14 | SWCLK    |
| 37  | PA15 | GPIO     |
| 38  | PB3  | GPIO     |
| 39  | PB4  | GPIO     |
| 40  | PB5  | GPIO     |
| 41  | PB6  | GPIO     |
| 42  | PB7  | GPIO     |
| 43  | BOOT1| PB2/BOOT1 |
| 44  | VSS  | GND      |
| 45  | VDD  | Power    |
| 46  | NRST | Reset    |
| 47  | VSS  | GND      |
| 48  | VDD  | Power    |