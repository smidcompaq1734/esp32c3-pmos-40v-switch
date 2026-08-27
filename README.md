# esp32c3-pmos-40v-switch

KiCad 6/7 single-sheet schematic for switching a 40V high-side rail from an ESP32-C3 GPIO using a P-channel MOSFET, an NPN level-shift transistor, and a Zener gate clamp.

## Project files

- `/home/runner/work/esp32c3-pmos-40v-switch/esp32c3-pmos-40v-switch/esp32c3-pmos-40v-switch.kicad_pro`
- `/home/runner/work/esp32c3-pmos-40v-switch/esp32c3-pmos-40v-switch/esp32c3-pmos-40v-switch.kicad_sch`

Open the `.kicad_pro` file in KiCad 6 or 7 to view and edit the schematic.

## Nets

| Net | Purpose |
| --- | --- |
| `+40V` | Main supply rail |
| `GATE_NODE` | PMOS gate-drive node shared by R1, D1, Q1 collector, and R4 input |
| `GND` | Common ground |
| `LOAD_OUT` | High-side switched output to the load |
| `GPIO_PIN` | 3.3V ESP32-C3 GPIO control input |

## Implemented schematic connectivity

| Ref | Type | Value/Part | Connection details |
| --- | --- | --- | --- |
| U1 | P-channel MOSFET | Generic `Q_PMOS_GSD` symbol, value `PMOS >=60V VDS` | Source → `+40V`, Drain → `LOAD_OUT`, Gate → `R4` output |
| R1 | Resistor | 10k | `+40V` → `GATE_NODE` |
| D1 | Zener diode | 1N4744A / 15V | Cathode → `+40V`, Anode → `GATE_NODE` |
| Q1 | NPN BJT | 2N3904 | Collector → `GATE_NODE`, Base ← `R2`, Emitter → `GND` |
| R2 | Resistor | 1k | `GPIO_PIN` → Q1 base |
| R3 | Resistor | 10k | Q1 base → `GND` |
| R4 | Optional gate resistor | 0R/100R | `GATE_NODE` → PMOS gate |
| D2 | Optional flyback diode | 1N4007 | Cathode → `LOAD_OUT`, Anode → `GND` |
| J1 | Load connector | Conn_01x02 | Pin 1 → `LOAD_OUT`, Pin 2 → `GND` |
| J2 | Power connector | Conn_01x02 | Pin 1 → `+40V`, Pin 2 → `GND` |
| J3 | ESP32-C3 control header | Conn_01x02 | Pin 1 → `GPIO_PIN`, Pin 2 → `GND` |

## Behavior

- GPIO HIGH → Q1 turns ON → `GATE_NODE` is pulled toward GND → PMOS `Vgs` becomes negative → PMOS turns ON → the load is powered from `+40V`.
- GPIO LOW → Q1 turns OFF → R1 pulls `GATE_NODE` up to `+40V` → PMOS `Vgs ≈ 0V` → PMOS turns OFF.
- D1 clamps the PMOS gate-to-source voltage so the GPIO-driven level shift does not exceed the MOSFET's safe `Vgs` rating.
- R4 is included as an optional 0Ω/100Ω series gate resistor footprint/value placeholder to reduce ringing and EMI if needed.
- D2 is included as an optional flyback diode location for inductive loads such as relays, motors, or solenoids.

## BOM guidance

| Ref | Suggested part | Why |
| --- | --- | --- |
| U1 | Any P-channel MOSFET with `VDS >= 60V` and `VGS >= ±20V` | A real 40V design needs margin above the rail; many low-voltage logic PMOS parts are not suitable. Select current rating and `RDS(on)` for the actual load. |
| Q1 | 2N3904 or BC547 | Common small-signal NPN transistor suitable for pulling the gate node low. |
| D1 | 1N4744A (15V) or similar 15–18V Zener | Clamps PMOS `Vgs` into a safe range while allowing solid turn-on. |
| D2 | 1N4007 | Appropriate general-purpose flyback diode option for slower inductive loads. |
| R1 | 10k, 1/10W or higher | Holds the PMOS OFF by default with very low bias current. |
| R2 | 1k, 1/10W or higher | Limits ESP32-C3 GPIO base drive current into Q1. |
| R3 | 10k, 1/10W or higher | Keeps Q1 OFF while the ESP32-C3 GPIO is floating during reset/boot. |
| R4 | 0Ω or 100Ω, 1/10W or higher | Optional damping / EMI reduction at the PMOS gate. |
| J1/J2/J3 | Connector choice to suit wiring/current | Pick creepage, insulation, and current rating appropriate for a 40V system and the expected load current. |

## Design notes

- Do **not** drive a 40V PMOS gate directly from a 3.3V ESP32-C3 GPIO.
- The PMOS must be chosen for both voltage and load current, not just logic-level threshold.
- If the load is inductive, populate D2 close to the load/switch path.
- Review connector spacing and isolation for the intended environment before making a PCB.

## Opening in KiCad

1. Open KiCad 6 or 7.
2. Choose **File → Open Project**.
3. Select `/home/runner/work/esp32c3-pmos-40v-switch/esp32c3-pmos-40v-switch/esp32c3-pmos-40v-switch.kicad_pro`.
4. Open the schematic editor to inspect the single-sheet design.
