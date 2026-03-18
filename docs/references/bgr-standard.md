---
description: Conventional Bandgap Reference design procedure — series CTAT and PTAT voltage summation topology, 1.22V silicon bandgap output, 14.204 ppm/°C temperature coefficient, on I/O 500nm MOSFET, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Series Realized Band Gap Reference (BGR)

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of BGR.

<a id="fig-01"></a>

![Series BGR Schematic Diagram](./bgr-standard-assets/01_Schemtic_Series_BGR_dark.svg#only-dark)
![Series BGR Schematic Diagram](./bgr-standard-assets/01_Schemtic_Series_BGR_light.svg#only-light)
/// caption
**Figure-01:** Series BGR Schematic Diagram
///

The PNP BJTs are vertical parasitic components that can be used to generate a V~BE~ voltage (CTAT) in a CMOS process. One such diode is characterized in [PNP 20 X 20](/mosfet/BJT-parameters/#pnp-20-x-20). The parameters listed there are what we will use to design the BGR, and as such, some necessary parameters for our design are summarised in *Table-01:*

<a id="table-01"></a>

| Parameter | Value | Comments |
|-----------|-------|----------|
| η | 1003.6 m | Forward current emission coefficient (Emitter-Base junction) |
| I~S~ | 0.285 aA | Saturation Current |
| \(dV_{D}/dT\) | -1.75 mV/°C | Change in the diode’s voltage with temperature (at current of 5 µA) |
/// caption
**Table-01:** PNP Diode summary
///

!!! question "Why 8 PNP Diodes in parallel? Why the number 8?"
    This is entirely due to the ΔVBE caused by the emitter area mismatch between a single PNP diode and `K` PNP diodes connected in parallel, where `K` is any integer.

    \[\Delta V_{BE} = \eta V_{T} \ln K\]

    From [PNP 20 X 20](/mosfet/BJT-parameters/#pnp-20-x-20), we see that η = 1003.6m and V~T~ = 26 mV (at room temperature). Plugging it into the above equation yields:

    \[\Delta V_{BE} = 1003.6m * 26m * \ln 8 = 54 mV\]

    Just 54 mV! For 8 PNP Diodes in parallel. Some people will even go for a K of 100, to get a ΔVBE of 120 mV. I know it is painful, but see that to double what we got before with just 8 diodes, now needs 100 diodes. Logarithmic functions are scary!

    That is why, I choose 8. Keeping 10 diodes is also a good choice (for a ΔVBE of 60 mV), but I am going to stick with 8.

For designing this, we need to find:

- Sizes for PMOS.
- Value of PTAT Current generator resistor, R~1~.
- Error amplifier topology plus sizes for MOSFETs used to build it.
- Value of PTAT voltage source resistor, R~2~.

The MOSFET sizes are already choosen and their parameters are thoroughly listed in [I/O Device — 500 nm](/mosfet/parameters/#io-device-500-nm) table. Unless specified otherwise, these sizes will be the ones we use to construct the entire circuit.

And as such, these are tabulated here for convenience:

<a id="table-02"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ |
|---------|-------------|--------------|------|
| NMOS | 5.8 / 1 | 2.9 µm / 500 nm | 10 µA |
| PMOS | 13.8 / 1 | 6.9 µm / 500 nm  | 10 µA |
/// caption
**Table-02:** Sizes and Bias Current summary
///

!!! question
    The observant reader would have questioned these sizing choices considering [Table-01](#table-01) characterizes PNP Diodes for 5 µA while here it is characterized for 10 µA. But still, follow along. These sizes are chosen for a general analog design. Just to tease you on how these two gets tied up, the error-amp steps in to adjust the V~SG~ of top PMOS in [Figure-01](#fig-01) to reduce it's current to 5 µA.

This leaves us with values for resistors R~1~ and R~2~ and the error amplifer.

