---
description: Current Starved Ring VCO design procedure — 7-stage current starved ring oscillator, 500 MHz centre frequency, 172.8 Mrad/s/V tuning gain, 495.5–512 MHz tuning range over 0.4–1.0V control input, on core 65nm custom CMOS with 1.2V supply, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Current Starved Ring VCO (CS-VCO)

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of CS-VCO.

<a id="fig-01"></a>

![CS-VCO Schematic](./csvco-assets/07_CSVCO_Schematic_dark.svg#only-dark)
![CS-VCO Schematic](./csvco-assets/07_CSVCO_Schematic_light.svg#only-light)
/// caption
**Figure-01:** CS-VCO schematic diagram \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 19.25]
///

Such a VCO will posses a low-gain and a linear transfer curve as shown in *Figure-02*.

![Expected Transfer](./csvco-assets/08_Expected_Transfer_Curve_dark.svg#only-dark)
![Expected Transfer](./csvco-assets/08_Expected_Transfer_Curve_light.svg#only-light)
/// caption
**Figure-02:** Transfer curve of CS-VCO of Figure-01
///

### For use in XOR DPLL

The schematic diagram of CS-VCO shown [Figure-01](#fig-01) is a modified variant to posses **low gain and linear transfer function**. This modified CS-VCO is suitable for use in an XOR DPLL. 

The reasoning behind these modifications and the motivation for choosing a low gain and linear transfer function are not discussed here. As before, for a detailed explanation of how this circuit works, you can refer to standard textbooks or lecture materials.

### Design Considerations

For this documentation, the center frequency is taken to be 500 MHz, that is f~out~ = 500 MHz when V~in,VCO~ = V~DD~\/2 = 0.6 V.

And to design this, we need to chose values for four things:

- MOSFET Sizes (for both NMOS and PMOS) suitable for Current starved inverter
- Input NMOS size (VCCS MOSFET)
- Range resistor value
- Lower limit resistor value

The MOSFET sizes suitable for digital design are already chosen, and their parameters are thoroughly listed in [Switching Parameters (Chosen Size) section of Digital Models](../mosfet/parameters.md#switching-parameters-chosen-size) table. For the sake of convenience, sizes are listed here:

<a id="table-01"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | R~n,p~ | C~oxn,p~ |
|---------|-------------|--------------|--------|----------|
| NMOS | 10/1 | 650 nm / 65 nm | 3.4 kΩ | 561 aF |
| PMOS | 20/1 | 1.3 µm / 65 nm | 3.4 kΩ | 1.06 fF |
/// caption
**Table-01:** Sizes and Parameter summary
///

This leaves us with the remaining three: Input NMOS size, Range resistor value, and Lower limit resistor value.

## Sizing the input VCCS NMOS

The input NMOS along with degeneration resistor acts as a linear VCCS. The difference between input voltage and V~GS~ developed is dropped across the resistor to generate an output current.

\[I_{out} = \frac{V_{in,VCO} - V_{GS}}{R_{range}}\]

Clearly, for different values of current, different V~GS~ develops and that leads to a non-linear output current. We don't want that.

Ideally, we want the V~GS~ generated to be constant regardless of the output current.

### Fixing non-linear V~GS~ with a wide MOSFET

The solution to this problem is to make this input NMOS really wide. Calling a MOSFET wide depends on the context of the current it can conduct. For example, sizing a MOSFET to conduct 50 µA and then actually make it conduct 5 µA makes it wider for the use case. 

Under such conditions, the V~GS~ developed will be lower than what it generates to conduct it's target current. When this is so, for all the range of output currents which gets developed, a wide MOSFET need not to turn ON harder.

***In fact, if we make it ridiculously wide, it will barely turn ON and develop a \(V_{GS} \approx V_{TH}\) and will remain the same as long as the current that actually flows is too smaller than what it is sized to conduct***.

This makes us to define our target current range, so we can size an NMOS that can be considered wider with respect to these current range.

### Target current range for input VCCS

We will target a current range of ***2 µA***, with the actual currents varying from ***0 A to 2 µA***. 

I found this value while playing around with the input current to a CS-VCO. In fact, in later parts of this documentation, we will find that this estimate is a good enough.

Follow along, you will find that this value is a good starting point.

!!! question "This particular value of 2 µA seems overly arbitrary, and the explanation given feels a bit vague?"
    I understand that this feels like I am randomly throwing a number out of the blue. This feeling is valid. But,

    **An actual design doesn't follow a linear procedure. You calculate a value for something and proceed to the next step only to find an error which makes you to trace back 10 steps, essentially re designing it.**

    **So explaining such a non-linear journey in a linear story format is extremely difficult and one of the major reasons why textbooks are often confusing until you try to build what it says.**

    ***The only way to know what value will be enough is to experiment with a bunch of values.***
    
    I started with a range of 5 µA and found it to posses too much gain, and then reduced it to 2 µA. You iterate. You iterate a lot and see failures. Only then, will you able to say that this much is enough. In short gain experience.

    In order to back this statement, see the refined procedure below which I became capable of giving, only after fully designing it.

    ??? example "Refined procedure to find a value for this range current"
        This is not the way the documentation follows, but is the refined procedure developed through experience. Much of the design in CS-VCO of [Figure-01](#fig-01) revolves around the input VCCS.

        And you need a range of current to even start designing the said VCCS. So, using [Substitution theorem](../references/bmr.md#substitution-theorem), ***why not just replace the entire current generator portion with an ideal current source to conduct some abstract simulations to find a reasonable estimate?***

        ![Testbench for range](./csvco-assets/09_Refined_way_to_estimate_range_dark.svg#only-dark)
        ![Testbench for range](./csvco-assets/09_Refined_way_to_estimate_range_light.svg#only-light)
        /// caption
        **Figure-03:** Testbench to figure out the input current range
        ///

        With this substitution highlighted in *Figure-03*, **step through various current values** for the ideal current source to generate the transfer curve as shown in Figure ---. Then you will know, how much sensitive your CS-VCO is, and then define a suitable constraint on the input current range.

        Again, the constraint gets defined by the requirements for the VCO by the overall system. In my case, this is being built for use in an XOR DPLL, which itself paints a constraint on centre frequency of 500 MHz, and an output range of no more than 10 MHz around this frequency.

        And then you choose a current range with the above discussions and you try designing it. See the result. Is it satisfactory? Good. If not, just re-iterate.