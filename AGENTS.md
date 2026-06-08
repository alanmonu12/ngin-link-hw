# AGENTS.md - Contexto para agentes de IA

## Proyecto

**ngin-link-hw** es un dongle USB-CAN basado en STM32F446CET6 (LQFP-48) con firmware en Rust.

## Herramientas

- **KiCad 10.x**: Esquematicos, PCB, librerias. Los archivos usan formato S-expression (`.kicad_sch`, `.kicad_pcb`, `.kicad_sym`, `.kicad_mod`).
- **Git**: Control de versiones. Seguir convenciones de commits conventional commits.
- **Linter**: `kicad-cli` para verificaciones. No ejecutar simulaciones SPICE en CI.

## Convenciones del proyecto

### Nomenclatura de componentes

- References: `U` (ICs), `J` (conectores), `R` (resistores), `C` (capacitores), `L` (inductores), `D` (diodos/LEDs), `F` (fusibles/PTC), `Y` (cristales), `TP` (test points), `FB` (ferrite beads)
- Valores: Usar sufijos SI (10k, 100n, 4u7). No usar espacios entre valor y unidad.
- Net labels: `ALL_CAPS_SNAKE_CASE` en el esquematico (ej: `CAN_TX`, `USB_DM`, `VCC`, `CANH`)

### Reglas de PCB

- 4 capas, tamaño objetivo ~50x20 mm
- Impedancia USB: 90 ohm diferencial en D+/D-
- CAN:���traza diferencial cuando sea posible, minimizar loops de tierra
- Desacoplo: 100nF + 4.7uF por cada par VDD/GND del MCU, colocados lo mas cerca posible
- El conector USB-C debe estar en un borde de la PCB con un corte (edge cut)
 Via: 0.3mm drill / 0.6mm pad minimo para senales
- Via de potencia: 0.5mm drill / 1.0mm pad

### Librerias de componentes

Las librerias de simbolos y footprints son **globales** y se comparten entre todos los proyectos. Estan en un repositorio separado:

- **Repo**: `git@github.com:alanmonu12/libs-kicad.git`
- **Path local**: `${KICAD_USER_LIBS}` (configurado en KiCad como variable de entorno)
- **Convencion de nombres**: Todas las librerias usan el sufijo `_ARR` (ej: `MCU_ARR`, `Resistor_ARR`)

Estructura del repo de librerias:

```
libs-kicad/
├── symbols/          # .kicad_sym (sufijo _ARR)
│   ├── MCU_ARR.kicad_sym
│   ├── Resistor_ARR.kicad_sym
│   ├── Capacitor_ARR.kicad_sym
│   ├── Diode_ARR.kicad_sym
│   ├── LED_SMD_ARR.kicad_sym
│   ├── Crystal_ARR.kicad_sym
│   ├── Power_IC_SMD_ARR.kicad_sym
│   ├── SMPS_SMD_ARR.kicad_sym
│   ├── Connector_SMD_ARR.kicad_sym
│   ├── Connector_THT_ARR.kicad_sym
│   └── ...
├── footprints/       # .pretty/ (sufijo _ARR)
│   ├── Package_QFP_ARR.pretty/
│   ├── Package_SO_ARR.pretty/
│   ├── Resistor_SMD_ARR.pretty/
│   ├── Capacitor_SMD_ARR.pretty/
│   ├── Diode_SMD_ARR.pretty/
│   └── ...
├── 3dmodels/         # Modelos 3D (.STEP, .stp, .wrl)
├── configuration/    # sym-lib-table y fp-lib-table de referencia
│   ├── sym-lib-table    # Usa ${KICAD_USER_LIBS}
│   └── fp-lib-table     # Usa ${KICAD_USER_LIBS}
└── templates/
```

**Reglas sobre librerias**:
- **No** crear librerias locales en `lib/`. Si un simbolo o footprint no existe en las librerias globales, se debe crear en el repo `libs-kicad` y luego referenciar desde este proyecto.
- Para agregar componentes nuevos, seguir la convencion `_ARR` en el repo global.
- Las tablas de librerias (`sym-lib-table`, `fp-lib-table`) en `~/.config/kicad/10.0/` se configuran una vez y apuntan a `${KICAD_USER_LIBS}`.
- No modificar las librerias estandar de KiCad.

## Arquitectura de hardware (resumen)

- **MCU**: STM32F446CET6 LQFP-48, Cortex-M4F @ 180 MHz, 512 KB Flash, 128 KB SRAM
- **CAN Transceiver**: SN65HVD230 (CAN 2.0B, 3.3V logic)
- **LDO**: AP2112K-3.3 (VBUS 5V -> 3.3V)
- **USB**: USB-C 2.0, Full-Speed (PA11/PA12)
- **CAN**: CAN1 en PB8 (RX) / PB9 (TX)
- **LEDs**: PB0 (status, active-low), PB1 (error, active-low)
- **Debug**: Tag-Connect SWD en PA13/PA14/NRST/GND
- **Proteccion**: TVS en USB, TVS en CAN, PTC en VBUS, common-mode choke en CAN bus
- **Terminacion CAN**: Resistor 120R con jumper/0R selectable

## Reglas para agentes de IA

1. **Siempre** leer los archivos KiCad existentes antes de modificarlos. Son S-expressions y la estructura exacta importa.
2. **No** generar archivos `.kicad_pcb` desde cero sin contexto del esquematico. Siempre partir del esquematico existente.
3. Al crear simbolos/footprints, seguir la estructura del repo global `libs-kicad` y la convencion de nomenclatura `_ARR`.
4. Al revisar esquematicos, verificar: desacoplo completo, pines no conectados, net labels consistentes, power flags.
5. Al revisar PCB, verificar: DRC, Courtyard,Ref. Des., zonas de tierra, ruteo diferencial.
6. **No** modificar el `.kicad_pro` manualmente a menos que sea estrictamente necesario (KiCad lo gestiona automaticamente).
7. Los comentarios en código S-expression de KiCad no son necesarios. Mantener sintaxis limpia.
8. Al generar documentacion, usar Markdown y colocarla en `docs/`.
9. Commits: usar conventional commits (`feat:`, `fix:`, `refactor:`, `docs:`, `lib:`, `sch:`, `pcb:`).

## Formato de archivos KiCad

Los archivos KiCad usan S-expressions. Ejemplo de estructura minima de un simbolo:

```kicad_sym
(kicad_symbol_lib
  (version 20250114)
  (generator "symbol_editor")
  (symbol "STM32F446CET6"
    (pin power_in line (at 0 0 0) (length 2.54)
      (name "VDD") (number "1")
    )
  )
)
```

Los archivos `.kicad_sch`, `.kicad_pcb`, `.kicad_sym`, `.kicad_mod` son el source of truth. No editar versiones cache o backup.