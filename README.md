# ngin-link-hw

[![License: CERN-OHL-S v2](https://img.shields.io/badge/license-CERN--OHL--S%20v2-blue)](LICENSE)
[![KiCad](https://img.shields.io/badge/KiCad-10.x-blue)](https://www.kicad.org)
[![MCU](https://img.shields.io/badge/MCU-STM32F446RET6-informational)](https://www.st.com/en/microcontrollers/stm32f446.html)
[![Firmware](https://img.shields.io/badge/firmware-Rust%20%F0%9F%A6%80-orange)](https://www.rust-lang.org)
[![CAN](https://img.shields.io/badge/bus-CAN%202.0B%20up%20to%201Mbps-green)](#)
[![USB](https://img.shields.io/badge/connector-USB--C%202.0%20FS-green)](#)
[![PCB](https://img.shields.io/badge/PCB-4%20layers%20%7C%2050%C3%9720%20mm-yellow)](#)

USB-CAN dongle basado en STM32F446 con firmware en Rust.

## Descripcion

ngin-link es un adaptador USB-CAN compacto que permite la comunicacion entre un host USB y un bus CAN. El cerebro del sistema es un STM32F446RET6 (Cortex-M4F @ 180 MHz) ejecutando firmware escrito en Rust (embbeded).

## Arquitectura de Hardware

```
                    +-----------------------+
   USB-C ---------> | VBUS -> LDO (3.3V)    |----> VCC
                    |                       |
                    |  STM32F446RET6        |
   USB D+/D- <----> |  PA12 (DP)           |
                    |  PA11 (DM)            |
                    |                       |
                    |  PB8  (CAN1_RX)       |<----- CAN Transceiver <-- CANH
                    |  PB9  (CAN1_TX)       |-----> CAN Transceiver --> CANL
                    |                       |
                    |  PB0  (STATUS_LED)    |----> LED Verde
                    |  PB1  (ERROR_LED)     |----> LED Rojo
                    |                       |
                    |  PA0  (BOOT0)         |<----- Jumper/Pad
                    |  NRST                |<----- Reset button / Tag-Connect
                    +-----------------------+
```

### Componentes principales

| Componente | Referencia | Notas |
|---|---|---|
| MCU | STM32F446RET6 | LQFP-64, Cortex-M4F 180 MHz, 512 KB Flash, 128 KB SRAM |
| CAN Transceiver | SN65HVD230 | CAN 2.0B, 3.3V logic, hasta 1 Mbps |
| Regulador | AP2112K-3.3 | LDO 3.3V 600 mA, SOT-223 |
| Conector USB | USB-C 2.0 | Receptaculo SMD, solo USB 2.0 FS |
| Conector CAN | JST-PH 4-pin o terminal screw | CANH, CANL, VEXT, GND |

### Caracteristicas

- **USB 2.0 Full-Speed**: Conexion al host via USB-C, VID/PID configurable por firmware
- **CAN 2.0B**: Comunicacion CAN hasta 1 Mbps mediante bxCAN del F446
- **Bus-powered**: Alimentacion desde USB VBUS (5V), regulado a 3.3V por LDO
- **Terminacion CAN selectable**: Jumper o resistor 0-ohm para habilitar terminacion de 120 Ohm
- **LEDs de estado**: LED verde (actividad CAN), LED rojo (error/bus-off)
- **Boot mode**: Pad/jumper para BOOT0, permite programacion por USART o USB DFU
- **Debug**: Tag-Connect footprint para SWD (SWDIO, SWCLK, NRST, GND)
- **Proteccion ESD**: TVS en lineas USB y CAN
- **Filtro CAN**: Common-mode choke en CANH/CANL
- **PCB**: 4 capas, 1.6 mm, tamaño objetivo ~50x20 mm (formato dongle), control de impedancia en USB y CAN

### Mapa de pines (STM32F446RET6 LQFP-64)

| Pin | Funcion | Uso |
|---|---|---|
| PA11 | USB_DM | USB D- |
| PA12 | USB_DP | USB D+ |
| PB8 | CAN1_RX | CAN receive |
| PB9 | CAN1_TX | CAN transmit |
| PA0 | GPIO | BOOT0 (con pull-down) |
| PB0 | GPIO | STATUS_LED (active-low) |
| PB1 | GPIO | ERROR_LED (active-low) |
| PA13 | SWDIO | Debug SWD |
| PA14 | SWCLK | Debug SWCK |
| NRST | NRST | Reset |
| BOOT0 | BOOT0 | Modo de arranque |

### Alimentacion

- Entrada: 5V desde USB VBUS
- Regulacion: AP2112K-3.3 (3.3V, 600 mA)
- Consumo estimado: ~50 mA (MCU + transceiver + LEDs)
- Proteccion: Polyswitch PTC + TVS en VBUS

## Estructura del proyecto

```
ngin-link-hw/
├── ngin-link-hw.kicad_pro     # Proyecto KiCad
├── ngin-link-hw.kicad_sch     # Esquematico principal
├── ngin-link-hw.kicad_pcb     # PCB layout
├── docs/                       # Documentacion adicional
├── AGENTS.md                   # Contexto para agentes de IA
├── .opencode/                  # Configuracion y skills de opencode
├── .gitignore                  # Reglas de git
├── LICENSE                     # CERN-OHL-S v2
└── README.md                   # Este archivo
```

Las librerias de simbolos y footprints estan en un repo global separado (`libs-kicad`), referenciadas via `${KICAD_USER_LIBS}`.

## Firmware

El firmware se desarrolla en Rust y se encuentra en un repositorio separado. Utiliza:

- `cortex-m-rt` runtime
- `stm32f4xx-hal` para perifericos
- `usb-device` para USB
- `can-oxide` o similar para CAN
- Programacion via SWD o USB DFU

## Licencia

Hardware liberado bajo [CERN Open Hardware Licence v2 - Strongly Reciprocal](LICENSE).