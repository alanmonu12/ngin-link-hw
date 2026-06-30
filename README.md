View this project on [CADLAB.io](https://cadlab.io/project/30445). 

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
   USB-C ---------> +-------------------+        +-------[B0505S]------+
   VBUS (5V)        | VBUS -> LDO 3.3V  |        |  Aislacion galv.     |
                    | (AP2112K-3.3)     |        |  USB side  | CAN side|
   USB D+/D- <----> |                   |        |  VBUS      | VISO_5V |
                    |  STM32F446RET6    |        +------|------------|---+
                    |  PA12 (DP)        |               |            |
                    |  PA11 (DM)        |               V            V
                    |                   |        +------------+ +-----------+
                    |  PB8  (CAN1_RX)   |<--[ADUM1201]--+  | | LDO 3.3V  |
                    |  PB9  (CAN1_TX)   |---[ADUM1201]----+ | | (CAN)    |
                    |                   |                  | +-----------+
                    |  PB0  (STATUS_LED)|----> LED Verde   |       |
                    |  PB1  (ERROR_LED) |----> LED Rojo    |       v
                    |                   |                  |   CAN Transceiver
                    |  PA0  (BOOT0)     |<----- Jumper/Pad |   (SN65HVD230)
                    |  NRST             |<----- Reset / TC |   <-- CANH/CANL
                    +-------------------+                  +--- GND_ISO
```

### Componentes principales

| Componente | Referencia | Notas |
|---|---|---|
| MCU | STM32F446RET6 | LQFP-64, Cortex-M4F 180 MHz, 512 KB Flash, 128 KB SRAM |
| CAN Transceiver | SN65HVD230 | CAN 2.0B, 3.3V logic, hasta 1 Mbps |
| Regulador (USB side) | AP2112K-3.3 | LDO 3.3V 600 mA, SOT-223 |
| Regulador (CAN side) | AP2112K-3.3 | LDO 3.3V 600 mA sobre VISO_5V |
| DC-DC aislado | **B0505S** | 5V/5V 1W, aislacion galv. 1 kV/3 kV, para aislar VBUS del bus CAN |
| Aislador digital | **ADUM1201** (propuesto) | Dual-channel digital isolator, SOIC-8, entre MCU y CAN transceiver |
| Conector USB | USB-C 2.0 | Receptaculo SMD, solo USB 2.0 FS |
| Conector CAN | JST-PH 4-pin o terminal screw | CANH, CANL, VEXT, GND |

### Caracteristicas

- **USB 2.0 Full-Speed**: Conexion al host via USB-C, VID/PID configurable por firmware
- **CAN 2.0B**: Comunicacion CAN hasta 1 Mbps mediante bxCAN del F446
- **Bus-powered**: Alimentacion desde USB VBUS (5V), regulado a 3.3V por LDO
- **Aislacion galv. USB <-> CAN**: DC-DC aislado **B0505S** (5V/5V, 1W) + **ADUM1201** digital isolator en CAN TX/RX; el lado USB (GND_USB) y el lado CAN (GND_ISO) quedan separados electricamente, eliminando loops de tierra y protegiendo el host de fallas en el bus
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
- **Lado USB** (referido a `GND_USB`):
  - Regulacion: AP2112K-3.3 (3.3V, 600 mA) → `+3V3_USB` para el MCU
- **Lado CAN** (referido a `GND_ISO`, galvanicamente aislado):
  - DC-DC aislado **B0505S** (5V in → 5V out, 1W) alimentado desde VBUS → `VISO_5V`
  - Regulacion: AP2112K-3.3 (3.3V, 600 mA) sobre `VISO_5V` → `+3V3_CAN` para el transceiver
- Consumo estimado: ~50 mA (MCU + transceiver + LEDs)
- Proteccion: Polyswitch PTC + TVS en VBUS
- Aislacion: la barrera galv. entre `GND_USB` y `GND_ISO` la establecen el B0505S (alimentacion) y el ADUM1201 (señales CAN TX/RX)

## Estructura del proyecto

```
ngin-link-hw/
├── ngin-link-hw.kicad_pro         # Proyecto KiCad
├── ngin-link-hw.kicad_sch         # Hoja raiz: portada + indice (Nivel 0)
├── ngin-link-hw.kicad_pcb         # PCB layout
├── sch/                            # Esquematico jerarquico (Niveles 1 y 2)
│   ├── 01_block_diagram.kicad_sch  # Nivel 1: diagrama a bloques funcional
│   ├── 10_usb.kicad_sch            # Nivel 2: USB-C + ESD + PTC + CC pull-downs
│   ├── 20_power.kicad_sch          # Nivel 2: AP2112K-3.3 LDO + filtrado
│   ├── 30_mcu.kicad_sch            # Nivel 2: STM32F446RET6 + crystal + reset
│   └── 40_can.kicad_sch            # Nivel 2: SN65HVD230 + TVS + terminacion
├── docs/                           # Documentacion adicional
├── AGENTS.md                       # Contexto para agentes de IA
├── .opencode/                      # Configuracion y skills de opencode
├── .gitignore                      # Reglas de git
├── LICENSE                         # CERN-OHL-S v2
└── README.md                       # Este archivo
```

### Esquematico jerarquico

El esquematico se organiza en **2 niveles de jerarquia** y 6 hojas totales:

| Nivel | Hoja | Descripcion |
|-------|------|-------------|
| 0 | `ngin-link-hw.kicad_sch` (root) | Portada con titulo, indice y sheet block unico hacia el block diagram |
| 1 | `sch/01_block_diagram.kicad_sch` | Diagrama a bloques funcional con sheet blocks hacia las hojas hijas |
| 2 | `sch/10_usb.kicad_sch` | USB-C connector, ESD, VBUS protection, CC pull-downs |
| 2 | `sch/20_power.kicad_sch` | AP2112K-3.3 LDO con filtro de entrada/salida |
| 2 | `sch/30_mcu.kicad_sch` | STM32F446RET6, crystal 8 MHz, reset, decoupling, LEDs y SWD |
| 2 | `sch/40_can.kicad_sch` | SN65HVD230 transceiver, TVS, common-mode choke, 120R term |

**Flujo de navegacion**: root → block diagram → seccion funcional. El root solo
contiene titulo, indice y un `sheet` block que apunta al block diagram; el block
diagram contiene los sheet blocks hacia las 4 hojas funcionales (USB, Power, MCU,
CAN).

Las hojas hijas exponen su interfaz electrica con `rectangle` + `text` blocks
que representan los bloques funcionales y sus anotaciones. Los nombres de los
identificadores conceptuales (VBUS, +3V3, GND, DM, DP, CAN_TX, CAN_RX, etc.)
son los que conectan las hojas entre si en el block diagram.

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