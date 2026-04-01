---
description: Current Starved Ring VCO design procedure — 7-stage current starved ring oscillator, 500 MHz centre frequency, 172.8 Mrad/s/V tuning gain, 495.5–512 MHz tuning range over 0.4–1.0V control input, on core 65nm custom CMOS with 1.2V supply, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Current Starved Ring VCO (CS-VCO)

## Prerequisites

The procedure for fixing the range resistor value uses the **substitution theorem**, which is covered extensively in the [Beta Multiplier Reference (BMR)](../references/bmr.md). Specifically, refer to the [Fixing Resistor Value](../references/bmr.md#fixing-resistor-value) section there before proceeding — the rationale and testbench methodology described there apply directly here.

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
    
    An NMOS with a width of `116.9` is simply too ridiculous considering that the negative charges in the channel are more mobile than the one in PMOS, and this size would have pulled ridiculous amounts of current if the V~GS~ were to increase above threshold. See *Figure-06* which shows the I~D~ vs V~DS~ curve with a V~GS~ of 430 mV.

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
    ***What if we add a dummy constant current along with the input controlled current? Like a current source in parallel with MOSFET?***

Like the one shown below:

![Adding a Dummy Current](./csvco-assets/12_FixingLowerLimitToFosc_dark.svg#only-dark)
![Adding a Dummy Current](./csvco-assets/12_FixingLowerLimitToFosc_light.svg#only-light)
/// caption
**Figure-08:** A dummy current load which always pulls a current regardless of input MOSFET ON/OFF state
///

When this is done, *even if I~VCCS~ where to vanish, thanks to current source pulling a non-zero current at all times, the VCO's output frequency will now have a lower limit!*

### Implementing this current source - Simple resistor as a pull down load

If the VCCS is off, that is V~in,VCO~ is so low, then, you have only the current source as a pull down. So, in that case, consider this modification:

![Simplest Current Sink - Resistor](./csvco-assets/13_ImplementingConstantPullDown_LowerlimitingCurrent_dark.svg#only-dark "Resistor Diode Connected PMOS Divider to set minimum Current")
![Simplest Current Sink - Resistor](./csvco-assets/13_ImplementingConstantPullDown_LowerlimitingCurrent_light.svg#only-light "Resistor Diode Connected PMOS Divider to set minimum Current")
/// caption
**Figure-09:** MOSFET resistor bias leg to derive a non-zero current
///

The benefit of this modification is **it sets a lower frequency limit (i.e. a lower current limit) regardless of input voltage** and so, the TF seems to get an offset.

![TF Offset](./csvco-assets/14_TFwithNewLowerLimit_dark.svg#only-dark)
![TF Offset](./csvco-assets/14_TFwithNewLowerLimit_light.svg#only-light)
/// caption
**Figure-10:** A lower limit in TF curve manifesting as an offset due to dummy load current
///

!!! success
    And this is **desirable, as the VCO cannot wander to lower frequencies thanks to the lower limit**. Thus, the addition of resistor limits the lower frequency by sinking a non-zero current even if VCCS MOSFET turns off.

### Any possible issue with supply dependance?

Of course, Diode Connected PMOS and Resistor together forms a voltage divider, which heavily depends on supply. But, **That is not a concern** as the PLL is an extremely sensitive circuit and generally, such circuit blocks gets accompanied by Supply regulators (LDO).

#### *Not Convinced with the above reason?*

Imagine, even if we make the current source somehow independent of supply voltage, because we're using the ring oscillator topology, the frequency of oscillation is still dictated by supply voltage! (see [Equation-01](#eqn-01))

I mean think about it, Ring oscillator is just a bunch of inverters (in odd number of quantity, very important) connected in a loop. If supply drops down, then the inverter output voltage range also drops down, thereby making it easier to cover this reduced range in little time (i.e. frequency has changed).

There's no point in making a supply independent current source, when the VCO itself has so much supply dependance. Let's just push it onto the regulator from a system perspective and relax!

### A MOSFET only solution

If we have opted to use a simple MOSFET resistor divider, we can also opt for MOSFET only dividers as shown in *Figure-11*.

![MOSFET Divider](./csvco-assets/15_MOSFETOnlyDivider_dark.svg#only-dark)
![MOSFET Divider](./csvco-assets/15_MOSFETOnlyDivider_light.svg#only-light)
/// caption
**Figure-11:** Replacing resistor with an NMOS as pull-down
///

The Pull-up PMOS has the standard digital size as characterized in [Table-01](#table-01) and the pull-down NMOS can be sized easily noting that our target current is 20 µA (see [Estimating minimum current in Current starved inverter](#estimating-minimum-current-in-current-starved-inverter) section).

With this in mind, I have sized the pull-down NMOS to be `66.2/10 (4.3 µm/ 650 nm)` in order to roughly pull a current of 20 µA as seen in *Figure-12*.

![Lower Limit NMOS Schematic](./csvco-assets/02_01_LowerLimit_NMOS_PullDown_DCAnnotatedSchematic_dark.png#only-dark)
![Lower Limit NMOS Schematic](./csvco-assets/02_01_LowerLimit_NMOS_PullDown_DCAnnotatedSchematic_light.png#only-light)
/// caption
**Figure-12** Sizing the pull-down NMOS \[`66.2/10 (4.3 µm/ 650 nm)`] for a current of 20 µA
///

## Fixing the range resistor, R~range~

We've already chosen a range current of 2 µA (see, [Target current range for input VCCS](#target-current-range-for-input-vccs) section), and the sizes for input NMOS (`66.2/1`) and lower limit NMOS (`66.2/10`).

It is time to choose the range resistor value. The trick in finding this resistor lies in our TF curve. Since our supply voltage is 1.2 V, **an acceptable range of input voltages, V~in,VCO~ can be from 0.4 V to 1.0 V**. There is no way a VCO can accommodate full supply rail as it's input voltage range, as near the rails, our TF can show severe non-linearity (more on this later).

### Concocting a tesbench to find this resistor

Just as stated in the [Prerequisites](#prerequisites) section, we will invoke the [Substitution theorem](../references/bmr.md#substitution-theorem), but with a little modification. This is not exactly the substitution theorem, rather something inspired by it.

Remember, that we need our output current to have a range of 2 µA, that is, **0 A to 2 µA** when V~in,VCO~ varies from **0.4 V to 1.0 V**.

!!! info
    Input voltage range may extend past **0.4 V to 1.0 V** range, and this is taken arbitrarily.

Therefore, 

- when V~in,VCO~ takes it's maximum value of 1.0 V, the output current should be 2 µA.
- And this current must linearly vary with input voltage with the slope dictated by the maximum limit, in it's input voltage range.

!!! success ""
    This is easily achieved by substituting an **Ideal VCCS** with a transfer ratio of:

    \[Transconductance ~of ~VCCS, ~A_{VCCS} = \frac{2 ~\mu A}{1 ~V} = 2 ~\mu S\]

    in place of our range resistor as seen in *Figure-13*.

![Ideal VCCS Substitution](./csvco-assets/03_01_CurrentGenerator_schematic_dark.png#only-dark)
![Ideal VCCS Substitution](./csvco-assets/03_01_CurrentGenerator_schematic_light.png#only-light)
/// caption
**Figure-13:** Substituting the range resistor, R~range~ with an ideal VCCS of transfer, A~VCCS~ of 2 µS
///

Such a substitution allows us to see how the total current varies even before we fix our resistor value. Let's see this variation by sweeping the input voltage, V~in,VCO~.

### Current Variation with input Voltage

The results of sweeping the input voltage are shown in *Figure-14*.

<a id="fig-18"></a>

![Input sweep results - 1](./csvco-assets/03_05_Irange_Vcurrent_dark.svg#only-dark)
![Input sweep results - 1](./csvco-assets/03_05_Irange_Vcurrent_light.svg#only-light)

![Input sweep results - 2](./csvco-assets/03_04_ITotal_Vs_VINVCO_dark.svg#only-dark)
![Input sweep results - 2](./csvco-assets/03_04_ITotal_Vs_VINVCO_light.svg#only-light)
/// caption
**Figure-18:** Input Voltage sweep Vs (a) Range Current, (b) Voltage developed across VCCS and \(c) Total Current
///

Some observations:

- The linear range current and it's value at V~in,VCO~ of 1.0 V as 2 µA is unsurprising, considering an ideal VCCS is used.
- The voltage developed across VCCS goes negative roughly below an input voltage, V~in,VCO~ of 0.3 V as seen from *Figure-18(b)*. It also saturates above 1.0 V, indicating our potential range of input is **0.3 V to 1.0 V**
- **Total current variation is just 1 µA instead of 2 µA** as seen from *Figure-18\(c)*.  
This is easily explained from the fact that the diode connected lower limit NMOS reduces the pull-down current if the voltage across it reduces, which it does, when the input VCCS pulls additional current through pull-up PMOS, essentially increasing the voltage drop across it.

!!! success ""
    In fact, this is good for us, as **lesser the variation in input current, the lesser the gain of CS-VCO**.

#### Computing the value of this resistor

Following [Fixing Resistor Value](../references/bmr.md#fixing-resistor-value) section of [Beta Multiplier Reference (BMR)](../references/bmr.md), we have found the voltage across ideal VCCS at maximum current of 2 µA to be **537.89 mV** from [Figure-18(b)](#fig-18).

Therefore,

\[Range ~resistor, R_{range} = \frac{537.89m}{2 \mu} = 268.9 ~k\Omega \approx 270 k\Omega\]

This is a huge resistance, but it is necessary to ensure low gain. We will see that this value is justified shortly in Transfer function measurement sections.

## Current Generator summary

Much of the design is complete, as the current generator is the elephant in the room. Let's summarize the results here for convenience:

<a id="table-02"></a>

| Instance | Value | Comment |
|----------|-------|---------|
| Input VCCS NMOS | 66.2/1 | Actual Dimensions: (4.3 µm / 65 nm) |
| Lower Limit NMOS | 66.2/10 | Actual Dimensions: (4.3 µm / 650 nm) |
| R~range~ | 270 kΩ | For low Gain |
/// caption
**Table-02:** Current Generator instance values summary
///

And the complete schematic of current generator is shown in *Figure-19*.

![Complete Current generator schematic](./csvco-assets/04_01_CurrentGenerator_270k_schematic_dark.png#only-dark)
![Complete Current generator schematic](./csvco-assets/04_01_CurrentGenerator_270k_schematic_light.png#only-light)
/// caption
**Figure-19:** Final schematic of Voltage controlled Current generator portion
///

## Fixing the number of stages for Ring Oscillator

Now that our current generator design is complete, it's about time we found the number of stages needed for a center frequency of 500 MHz. Even though our target current is 20 µA for center frequency, it would be better if we found the actual current in the current starved inverter to reduce the errors in our hand calculations.

The current starved inverter schematic is shown in *Figure-20*.

![CS Inverter schematic](./csvco-assets/05_01_CSInverter_schematic_dark.png#only-dark)
![CS Inverter schematic](./csvco-assets/05_01_CSInverter_schematic_light.png#only-light)
/// caption
**Figure-20:** Current Starved Inverter cell schematic (Sizes from [Table-01](#table-01))
///

Clearly, from *Figure-20* the current in the inverter cell is controlled by *VBP* and *VBN* voltages which are generated using the principle of current mirror. ***And this is exactly why, instead of using 20 µA, we will find the actual current in the inverter using simulations to account for errors in current mirror***.

### Finding actual current in Current Starved Inverter

To that, see the testbench for finding current in inverter shown below. 

![Current in Inverter](./csvco-assets/04_05_CurrentInInverter_DCAnnotatedSchematic_dark.png#only-dark)
![Current in Inverter](./csvco-assets/04_05_CurrentInInverter_DCAnnotatedSchematic_light.png#only-light)
/// caption
**Figure-21:** DC Annotated testbench to find current in inverter (Sizes from [Table-01](#table-01) and [Table-02](#table-02)) (V~in,VCO~ = 0.6 V to find center current)
///

This is nothing but a partial schematic of [Figure-01](#fig-01).

Some commentary:

- Input voltage is 0.6 V, as [Equation-01](#eqn-01) is used with center frequency and current used for center frequency. And for XOR DPLL, VCO TF is chosen to have the center frequency at V~DD~\/2 (0.6 V) as XOR Phase detector average output in locked state is V~DD~\/2.
- Generated current is 20.54 µA as seen in PM1.