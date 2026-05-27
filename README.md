## What this is about
This repository contains the design files for a single-cell **18650 battery charger** with load-sharing circuitry, a buck/boost converter (3.3V/5.0V, 500mA output, 20µA quiescent current), and **hardware reverse-polarity protection** for the 18650 battery holder.

> **Note:** The Li-Po/JST connector option has been removed. This design is exclusively for a user-insertable 18650 cell in the on-board holder. The charger IC is the **LN4056H** (pin-compatible replacement for the original TP4056).

![18650 battery charger PCB with load sharing and buck/boost converter](https://www.e-tinkers.com/wp-content/uploads/2025/08/18650_battery_charger_PCB.jpg)

## The motivation
All the 18650 battery chargers that can be found from AliExpress have serious design faults, and the 18650 battery is safer to be used with the protection circuit permanently attached to it. With the modification added to 18650, the commercial available battery holders for 18650 no longer fit. So I decided to design my own charger with custom 18650 battery holder to fit the modified battery, and add a low quiescent current buck/boost converter with 3.3V/5V output that I can used for my electronic projects.

![add protection circuit permanently to 18650 battery](https://www.e-tinkers.com/wp-content/uploads/2025/08/add_protection_circuit_permanently_to_18650_battery.png)

## The design goals
1. On-board 18650 battery holder able to fit standard 18650 and modified one;
2. USB Type C power input port;
3. **Reverse-polarity protection** for the 18650 battery path (P-channel MOSFET ideal-diode, low loss);
4. Battery protection for standard 18650 battery;
5. Load-sharing circuitry so it can power the load while charging;
6. Boost/Buck converter capable to deliver 500mA power with either 3.3V or 5V output;
7. Low quiescent current for IoT application.

## Reverse-polarity protection
A high-side P-channel MOSFET (Q2, AO3407) is placed in series with the 18650+ terminal in an "ideal diode" configuration:

```
18650+  ──── Drain(Q2) Source(Q2) ──── BAT+ rail
                         │
                        Gate
                         │
                        R11 (100 kΩ)
                         │
                        GND
```

- **Correct insertion:** the body diode conducts momentarily, charging the Source node. Once V_GS < V_th (≈ −2.5 V), the channel turns on with low R_DS(on), giving minimal voltage drop.
- **Reverse insertion:** the body diode is reverse-biased and V_GS ≈ 0 V, so the MOSFET stays off and no current flows — the entire circuit is protected.

This approach has much lower loss than a series Schottky diode and is suitable for single-cell Li-ion voltages.

## Charger IC
The charger IC is the **LN4056H** — a pin-compatible, drop-in replacement for the TP4056, with the same SOP-8+EP package and pinout. No PCB layout changes are required to swap between the two parts.

Read more in details about the design considerations from my [blog post](https://www.e-tinkers.com/2025/08/design-my-own-tp4056-li-ion-charger-and-iot-power-supply-subsystem/).

## Manual KiCad steps after cloning
The schematic (`.kicad_sch`) has been updated to reflect all changes. If you need to regenerate the PCB netlist or update component values in the `.kicad_pcb` layout:

1. Open `Li-Po_charger.kicad_sch` in KiCad.
2. Run **Tools → Update PCB from Schematic** to push the Q2/R11 additions and U1 value change into the layout.
3. Manually place Q2 (SOT-23) near the 18650+ holder footprint and route the three new connections (Source→BAT rail, Drain→J2+, Gate→R11→GND).
4. Remove the footprints for SW2 (BAT_SEL) and J4 (JST connector) from the PCB layout if they remain.

