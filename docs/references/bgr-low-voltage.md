---
description: Low Voltage Bandgap Reference design procedure — parallel CTAT and PTAT current summation topology, 0.61V output at VDD/2, 28.265 ppm/°C temperature coefficient, on core 65nm MOSFET with 1.2V supply, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Parallel Realized Band Gap Reference (BGR)

## Prerequisites

This design shares substantial methodology with the [Series Bandgap Reference](../references/bgr-standard.md). The core distinction is the **quantity** being temperature-stabilized — the Series BGR stabilizes **voltage** (series CTAT+PTAT voltage summation), while this design stabilizes **current** (parallel CTAT+PTAT current summation).

The following sections from the Series BGR apply directly and are not repeated:

- [Fixing PTAT Current Generator resistor, R~1~](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1) — identical procedure, same operating point (5 µA), same parasitic PNP diodes. Resistor value of 10.9 kΩ carries over directly.
- [Added error amplifier](../references/bgr-standard.md#added-error-amplifier) — same operating point (5 µA) and  same V~BE~ of 0.777 V (See [Table-01](#table-01)). However, 0.777 V sits close to V~DD~ here (1.2 V core supply) unlike the Standard BGR (3.3 V I/O supply), giving rise to slight but important differences as discussed in the main body of this page.
- [Startup Circuit Design](../references/bmr.md#startup-circuit-design) — design considerations and procedure are identical to the Series BGR and BMR.

Read the [Series BGR](../references/bgr-standard.md) documentation before proceeding.

# Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of Parallel BGR.

![Parallel BGR Schematic](./bgr-low-voltage-assets/01_SchematicOfParallelBGR_dark.svg#only-dark)
![Parallel BGR Schematic](./bgr-low-voltage-assets/01_SchematicOfParallelBGR_light.svg#only-light)
/// caption
**Figure-01:** Parallel BGR schematic diagram \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 23.30]
///

Just like the series BGR, same parasitic PNP Diode characterized in [PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20) is used here as well, and so the parameters listed there are summarised in *Table-01:*

<a id="table-01"></a>

| Parameter | Value | Comments |
|-----------|-------|----------|
| η | 1003.6 m | Forward current emission coefficient (Emitter-Base junction) |
| V~BE~ | 0.777 V | 1 Diode (At a current of 5 µA) |
| \(\delta V_{BE}/\delta T\) | -1.75 mV/°C | Change in the diode’s voltage with temperature (at current of 5 µA) for 8 Diodes in parallel |
/// caption
**Table-01:** PNP Diode summary
///

Refer to [Introduction](../references/bgr-standard.md#introduction) section for explanation on why 8 diodes is chosen. I chose not to repeat it here to avoid redundancy.