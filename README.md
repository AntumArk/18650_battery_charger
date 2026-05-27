## What this is about
This repository contains the design files for a single-cell **18650 battery charger** with load-sharing circuitry, a buck/boost converter (3.3V/5.0V, 500mA output, 20µA quiescent current), **hardware reverse-polarity protection** for the 18650 battery holder, an optional **DW01A protection-controller block**, and an optional **software-latch interface header**.

> **Note:** The Li-Po/JST connector option has been removed. This design is exclusively for a user-insertable 18650 cell in the on-board holder. The charger IC is the **LN4056H** (pin-compatible replacement for the original TP4056).

![18650 battery charger PCB with load sharing and buck/boost converter](https://www.e-tinkers.com/wp-content/uploads/2025/08/18650_battery_charger_PCB.jpg)

## The motivation
All the 18650 battery chargers that can be found from AliExpress have serious design faults, and the 18650 battery is safer to be used with the protection circuit permanently attached to it. With the modification added to 18650, the commercial available battery holders for 18650 no longer fit. So I decided to design my own charger with custom 18650 battery holder to fit the modified battery, and add a low quiescent current buck/boost converter with 3.3V/5V output that I can used for my electronic projects.

![add protection circuit permanently to 18650 battery](https://www.e-tinkers.com/wp-content/uploads/2025/08/add_protection_circuit_permanently_to_18650_battery.png)

## The design goals
1. On-board 18650 battery holder able to fit standard 18650 and modified one;
2. USB Type C power input port;
3. **Reverse-polarity protection** for the 18650 battery path (P-channel MOSFET ideal-diode, low loss);
4. Battery protection support for standard (unprotected) 18650 battery;
5. Load-sharing circuitry so it can power the load while charging;
6. Boost/Buck converter capable to deliver 500mA power with either 3.3V or 5V output;
7. Low quiescent current for IoT application;
8. Optional DW01A-based protection-controller interface for unprotected cells (with external dual-MOSFET stage);
9. Optional software-latch control header.

## Reverse-polarity protection
A high-side P-channel MOSFET (Q2, AO3407, SOT-23) is placed in series with the 18650+ terminal in an "ideal diode" configuration:

```
J2 (18650+)  ─── Drain ──[Q2 P-MOSFET]── Source ─── BAT+ rail
                                │
                               Gate (pin 1)
                                │
                              R11 (100 kΩ)
                                │
                               GND
```

Pin mapping for AO3407 SOT-23: **pin 1 = Gate**, **pin 2 = Source** (→ internal BAT+ rail feeding charger/load circuitry), **pin 3 = Drain** (→ J2 18650+ holder).

- **Correct insertion:** the PMOS body diode (anode = Drain, cathode = Source) is forward-biased, allowing current from the battery to the circuit. As the Source voltage rises, V_GS drops below V_th (≈ −2.5 V) and the low-R_DS(on) channel turns on.
- **Reverse insertion:** the body diode is reverse-biased and V_GS ≈ 0 V > V_th, so the MOSFET stays off and no current flows — the entire circuit is protected.

This approach has much lower loss than a series Schottky diode and is suitable for single-cell Li-ion voltages.

## Charger IC
The charger IC is the **LN4056H** — a pin-compatible, drop-in replacement for the TP4056, with the same SOP-8+EP package and pinout. No PCB layout changes are required to swap between the two parts.

## Optional DW01A protection block
An optional **DW01A** block (`U3`) is included in the schematic for users who do not have protected 18650 cells.  
The block exposes:

- raw battery input pads (`J6`, `J7`);
- DW01A control/sense breakout pads (`J8`..`J11`) for pairing with an external dual-MOSFET protection stage (for example FS8205A, a common dual-NMOS protection device used with DW01A).

This keeps the base charger path unchanged while providing a dedicated integration point for cell protection logic.

## Optional software-latching interface
An optional 2-pin header (`J12`, value `SOFT_LATCH_IF`) has been added:

- Pin 1: HOLD/CTRL node tied to the SW1 power-control node;
- Pin 2: GND.

This header allows adding an external software-controlled latching circuit without forcing it into the default always-manual power path.

Read more in details about the design considerations from my [blog post](https://www.e-tinkers.com/2025/08/design-my-own-tp4056-li-ion-charger-and-iot-power-supply-subsystem/).

## Manual KiCad steps after cloning
The schematic (`.kicad_sch`) has been updated to reflect all changes. If you need to regenerate the PCB netlist or update component values in the `.kicad_pcb` layout:

1. Open `Li-Po_charger.kicad_sch` in KiCad.
2. Run **Tools → Update PCB from Schematic** to push all schematic additions into the layout.
3. Place and route the reverse-polarity parts (`Q2`, `R11`) if not already synced.
4. Place and route the optional DW01A block (`U3`, `J6`..`J11`) only if you want on-board protection-controller support.
5. Place and route the optional software-latch header (`J12`) only if you plan to use software-controlled latching.
6. Remove the footprints for SW2 (BAT_SEL) and J4 (JST connector) from the PCB layout if they remain.
