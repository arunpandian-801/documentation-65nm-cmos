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

<a id="fig-02"></a>

![Expected Transfer](./csvco-assets/08_Expected_Transfer_Curve_dark.svg#only-dark)
![Expected Transfer](./csvco-assets/08_Expected_Transfer_Curve_light.svg#only-light)
/// caption
**Figure-02:** Transfer curve of CS-VCO of Figure-01 \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 19.26]
///

### For use in XOR DPLL

The schematic diagram of CS-VCO shown [Figure-01](#fig-01) is a modified variant to posses **low gain and linear transfer function**. This modified CS-VCO is suitable for use in an XOR DPLL. 

The reasoning behind these modifications and the motivation for choosing a low gain and linear transfer function are not discussed here. As before, for a detailed explanation of how this circuit works, you can refer to standard textbooks or lecture materials.

### Design Considerations

For this documentation, the center frequency is taken to be 500 MHz, that is f~out~ = 500 MHz when V~in,VCO~ = V~DD~\/2 = 0.6 V.

And to design this, we need to chose values for four things:

- MOSFET Sizes (for both NMOS and PMOS) suitable for Current starved inverter
- Input NMOS size (VCCS MOSFET)
- Lower limit resistor value
- Range resistor value

The MOSFET sizes suitable for digital design are already chosen, and their parameters are thoroughly listed in [Switching Parameters (Chosen Size) section of Digital Models](../mosfet/parameters.md#switching-parameters-chosen-size) table. For the sake of convenience, sizes are listed here:

<a id="table-01"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | R~n,p~ | C~oxn,p~ |
|---------|-------------|--------------|--------|----------|
| NMOS | 10/1 | 650 nm / 65 nm | 3.4 kΩ | 561 aF |
| PMOS | 20/1 | 1.3 µm / 65 nm | 3.4 kΩ | 1.06 fF |
/// caption
**Table-01:** Sizes and Parameter summary
///

This leaves us with the remaining three: Input NMOS size, Range resistor value, and Lower limit resistor value, but before we address them, let's estimate the current used in Current-starved inverters for operating frequency of 500 MHz.

### Estimating minimum current in Current starved inverter

CS-VCO ouput frequency is related to current through *Equation-01*.

<a id="eqn-01"></a>

\[\tag{1} F_{osc} = \frac{I_D}{N * C_{tot} * V_{DD}}\]

where, 

- C~tot~ is total capacitance at output of current starved inverter. 
- N is the number of stages.

---

C~tot~ is estimated from *Figure-03*:

![Total Capacitance at CS Inv](./csvco-assets/10_CurrentStarvedInverter_partialSchematic_Ctot_dark.svg#only-dark)
![Total Capacitance at CS Inv](./csvco-assets/10_CurrentStarvedInverter_partialSchematic_Ctot_light.svg#only-light)
/// caption
**Figure-03:** Capacitance loading each current starved inverter \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 19.15]
///

And is given by,

\[C_{tot} = C_{in} + C_{out} = (C_{ox,n} + C_{ox,p}) + \frac{3}{2} (C_{ox,n} + C_{ox,p}) = \frac{5}{2} (C_{ox,n} + C_{ox,p})\]

From [Table-01](#table-01), we get C~oxn,p~ as 561 aF and 1.06 fF for the chosen sizes.

Therefore,

<a id="eqn-02"></a>

\[\tag{2} C_{tot} = \frac{5}{2} (561 ~aF + 1.06 ~fF) = 4.052 ~fF\]

---

**It is generally recommended to keep a minimum of 5 stages, i.e., \(N \ge 5\)**

Using this in [Equation-01](#eqn-01) yields,

\[N = \frac{I_D}{f_{osc} * C_{tot} * V_{DD}} \ge 5\]

We know our target frequency is 500 MHz, and the V~DD~ is 1.2 V.

\[\frac{I_D}{500 ~M * 4.052 ~fF * 1.2 ~V} \ge 5\]

\[I_D \ge 12.16 ~\mu A\]

!!! success ""
    It is always good to leave some margin from minimum values. So, our ***target current is taken to be 20 µA (> 12.16 µA)***.

    \[I_{D,target} = 20 \mu A\]

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

I found this value while playing around with the input current to a CS-VCO. In fact, in later parts of this documentation, we will find that this estimate is good enough.

Follow along, you will find that this value is a good starting point.

??? question "This particular value of 2 µA seems overly arbitrary, and the explanation given feels a bit vague?"
    I understand that this feels like I am randomly throwing a number out of the blue. This feeling is valid. But,

    **An actual design doesn't follow a linear procedure. You calculate a value for something and proceed to the next step only to find an error which makes you to trace back 10 steps, essentially re designing it.**

    **So explaining such a non-linear journey in a linear story format is extremely difficult and one of the major reasons why textbooks are often confusing until you try to build what it says.**

    ***The only way to know what value will be enough is to experiment with a bunch of values.***
    
    I started with a range of 5 µA and found it to posses too much gain, and then reduced it to 2 µA. You iterate. You iterate a lot and see failures. Only then, will you able to say that this much is enough. ***In short, gain experience***.

    In order to back this statement, see the refined procedure below which I became capable of giving, only after fully designing it.

    ??? example "Refined procedure to find a value for this range current"
        This is not the way the documentation follows, but is the refined procedure developed through experience. Much of the design in CS-VCO of [Figure-01](#fig-01) revolves around the input VCCS.

        And you need a range of current to even start designing the said VCCS. So, using [Substitution theorem](../references/bmr.md#substitution-theorem), ***why not just replace the entire current generator portion with an ideal current source to conduct some abstract simulations to find a reasonable estimate?***

        ![Testbench for range](./csvco-assets/09_Refined_way_to_estimate_range_dark.svg#only-dark)
        ![Testbench for range](./csvco-assets/09_Refined_way_to_estimate_range_light.svg#only-light)
        /// caption
        **Figure-04:** Testbench to figure out the input current range
        ///

        With this substitution highlighted in *Figure-04*, **step through various current values** for the ideal current source to generate the transfer curve as shown in Figure ---. Then you will know, how much sensitive your CS-VCO is, and then define a suitable constraint on the input current range.

        Again, the constraint gets defined by the requirements for the VCO by the overall system. In my case, this is being built for use in an XOR DPLL, which itself paints a constraint on centre frequency of 500 MHz, and an output range of no more than 10 MHz around this frequency.

        And then you choose a current range with the above discussions and you try designing it. See the result. Is it satisfactory? Good. If not, just re-iterate.

### Sizing a wide NMOS

With all the previous discussions, the maximum current this NMOS will conduct is just 2 µA. And because it is so wide, it will just develop a V~GS~ of V~TH~.

In order to size this NMOS, all we have to do is: 

- Fix \(V_{GS} = V_{TH} = 430 ~mV\). (V~TH~ comes from [Regular Threshold Voltage (RVT)](../mosfet/parameters.md#regular-threshold-voltage-rvt) table)
- And then choose a size which shows an I~D~ vs V~DS~ curve with current values slightly larger than 2 µA (say 5 µA or even 6 µA) to leave some margin.

Following this, I have chosen a size of `66.2/1` and it's I~D~ vs V~DS~ curve for a V~GS~ of 430 mV is shown in *Figure-05*.

![ID VDS curve Wide NMOS](./csvco-assets/02_02_WIDENMOS_64_6_by_1_430mV_dark.svg#only-dark)
![ID VDS curve Wide NMOS](./csvco-assets/02_02_WIDENMOS_64_6_by_1_430mV_light.svg#only-light)
/// caption
**Figure-05:** I~D~ vs V~DS~ curve of a `66.2/1` NMOS with a V~GS~ of 430 mV \(\approx V_{TH}\)
///

??? question "How come `66.2/1 (4.3 µm/65 nm)` is considered wide?"
    A `66.2/1 (4.3 µm/65 nm)` may not seem ridiculously wide, but don't be fooled by the width being `4.3 µm`. Look at *Figure-05*. ***This NMOS can conduct a current of 5 µA while having a V~GS~ of just the threshold voltage and a V~DS~ of 200 mV!***

    Normally, when you see the I~D~ vs V~DS~ curve for any MOSFET which is barely ON (or with a \(V_{GS} \approx V_{TH}\)), it will have at most a couple of µA. Meanwhile *Figure-05* shows a current of 5 µA with just a V~DS~ of 200 mV while barely turning ON! And it even goes up to several tens of µA!

    This is possible only when your MOSFET is ridiculously wide!

    Still not convinced with this? How about increasing the length to 2? With a drawn length of 2 (or actual length of 130 nm), you get a width of `116.9 (7.6 µm)` to conduct a similar current of 5 µA at about a similar V~DS~ of 200 mV! 
    
    An NMOS with a width of `116.9` is simply too ridiculous considering the negative charges in the channel are more mobile than the one in PMOS, and this size would have pulled ridiculous amounts of current if the V~GS~ were to increase above threshold. See *Figure-06* which shows the I~D~ vs V~DS~ curve with a V~GS~ of 430 mV.

    ![ID VDS curve Wide NMOS L of 2](./csvco-assets/02_03_WIDENMOS_116_9_by_2_430mV_dark.svg#only-dark)
    ![ID VDS curve Wide NMOS L of 2](./csvco-assets/02_03_WIDENMOS_116_9_by_2_430mV_light.svg#only-light)
    /// caption
    **Figure-06:** I~D~ vs V~DS~ curve of a `116.9/2` NMOS with a V~GS~ of 430 mV \(\approx V_{TH}\)
    ///

    Another way to accept that this is a wide NMOS is to see the aspect ratio and not the actual dimensions. A `66.2/1` size is too large for use in digital design considering such wider MOSFETs tends to load your previous stage.

With this chosen, let's address the lower limit resistor

## Why do you need this lower limit resistor, R~low~?

A quick glance at [Figure-01](#fig-01) can puzzle you as to why we attach the resistor R~low~. Addressing this is really important, because once we understand it's role, **we can then replace it with a MOSFET** for ease of integration.

Recall that an XOR DPLL needs a VCO with low gain, and whose TF curve is centered around a frequency f~center~ as seen in [Figure-02](#fig-02).

That is, ***even when input voltage were to go to ground, the output frequency shouldn't go below a lower limit in order for the XOR DPLL to not lose lock or worse, lock onto harmonics of center frequency.***

With that said, think about what happens without this resistor? If this resistor was not present, then all you have is just the source degenerated NMOS. And when the input voltage goes down, so does the current (As an NMOS needs some V~GS~ to conduct a current)! And with that your output frequency will also go down without settling to a lower limit!

So, you get a Transfer curve as seen in *Figure-07* where the output frequency just continues to go to lower frequencies as your input voltage goes down.

![TF curve without lower limit](./csvco-assets/11_TF_Curve_without_lower_limit_dark.svg#only-dark)
![TF curve without lower limit](./csvco-assets/11_TF_Curve_without_lower_limit_light.svg#only-light)
/// caption
**Figure-07:** Transfer Curve of a CS-VCO without lower limit resistor \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 19.18]
///

So, it's important to establish a lower limit on the output frequency in some way, ensuring that even when the input voltage drops and turns our input NMOS off, the VCO will still continue to oscillate at its lowest possible frequency.

## *Fixing a lower limit to output frequency by adding a constant current to Input Controlled Current*

The solution to the lower limit problem is concocted from the observation that VCCS MOSFET shuts off as input voltage goes low and Output current also goes low. So,

!!! tip ""
    ***What if we add a constant current along with the input controlled current? Like a current source in parallel with MOSFET?***

Like the one shown below:

