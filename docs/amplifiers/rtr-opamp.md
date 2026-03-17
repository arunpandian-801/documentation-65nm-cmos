---
description: Rail-to-rail input Class AB output op-amp design procedure — folded cascode topology with parallel NMOS and PMOS input stages on 65nm custom CMOS, characterised across all three CMR regions, simulated on Cadence Virtuoso and Spectre.
---

--8<-- "includes/disclaimer.md"

# Rail to rail input, Class AB output op-amp

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of this op-amp.

<a id="fig-01"></a>

![Rail to rail ip opamp schematic](./rtr-opamp-assets/08_Rail_to_rail_op_amp_Schematic_dark.svg#only-dark)
![Rail to rail ip opamp schematic](./rtr-opamp-assets/08_Rail_to_rail_op_amp_Schematic_light.svg#only-light)
/// caption
**Figure-01:** Rail-to-rail input, class AB output opamp schematic [Ref. CMOS Circuit Design, Layout and Simulation, Fig 24.48]
///

### Design Considerations

The design of this circuit is pretty straightforward as the only challenge in this is frequency compensation. Biasing is done by [General Bias Circuit](/references/general-bias/#general-bias-circuit) and the sizes are from [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table and **half the sizes** noted in [Low Threshold Voltage (LVT)](/mosfet/parameters/#low-threshold-voltage-lvt) table respectively.

For the sake of convenience, those sizes are listed here:

| Flavour | W/L (Drawn) | W/L (Actual) | COMMENTS |
|---------|-------------|--------------|----------|
| NMOS-RVT | 43.1 / 2 | 2.8 µm / 130 nm | Same as standard size |
| PMOS-RVT | 121.5 / 2 | 7.9 µm / 130 nm | Same as standard size |
| NMOS-LVT | 20 / 2 | 1.3 µm / 130 nm | Roughly **half** the standard size |
| PMOS-LVT | 47.7 / 2 | 3.1 µm / 130 nm | Roughly **half** the standard size |
/// caption
**Table-01:** Sizes and bias current summary.
///

!!! note ""
    It is strongly advised to review the documentation of the [General Bias Circuit](/references/general-bias/#general-bias-circuit) before moving forward with this one. Particularly pay attention to the discussion on:

    - [Sizing Class AB Bias Voltage Generation MOSFET](/references/general-bias/#sizing-class-ab-bias-voltage-generation-mosfet)
    - [Biasing a Class AB output stage](/references/general-bias/#biasing-a-class-ab-output-stage)

### Load Considerations

Thanks to class AB output stage, this op-amp is capable of driving low impedance (or heavy) loads. So, the load taken for this documentation is a resistor of 10 kΩ in parallel with a capacitor of 10 pF (10 kΩ || 10 pF). And as such, the output stage MOSFETs are 10 times (just a general number. A good starting point for heavy loads. If this proves insufficient, we will increase it.) the sizes listed in [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table.

!!! info
    Since this schematic diagram is quite large, I don't want to repeat it for every single simulation. Therefore, the schematic shown in [Figure-06](#fig-06) should be our starting point, and it will remain as the one we use until compensation is completed. All testbenches will include a symbol representing an op-amp that abstracts this schematic for simplicity until told otherwise.

Before we compensate this op-amp, let's see the ICMR.

## DC Simulations

### Input Common Mode Range

We will tie both inputs together and sweep it from negative rail to positive rail as shown in *Figure-02*.

![ICMR TB](./rtr-opamp-assets/09_ICMR_TB_dark.svg#only-dark)
![ICMR TB](./rtr-opamp-assets/09_ICMR_TB_light.svg#only-light)
/// caption
**Figure-02:** Testbench to find ICMR. The Output nodes are shown in Figure-03
///

The nodes *OUT_P_BIAS* and *OUT_N_BIAS* are the outputs of the 1rst stage, i.e. the outputs of the Cascode stack. These nodes can be seen in the partial schematic of *Figure-03*.

![Output Nodes in actual schematic](./rtr-opamp-assets/10_ICMR_Outputs_dark.svg#only-dark)
![Output Nodes in actual schematic](./rtr-opamp-assets/10_ICMR_Outputs_light.svg#only-light)
/// caption
**Figure-03:** The Nodes OUT_P_BIAS and OUT_N_BIAS are shown in actual schematic for cross-reference with Figure-02.
///

![Output curve over ICMR](./rtr-opamp-assets/11_CommonModeSweep_dark.svg#only-dark)
![Output curve over ICMR](./rtr-opamp-assets/11_CommonModeSweep_light.svg#only-light)
/// caption
**Figure-04:** Outputs of folded cascode stack vs V~CM~. Used to find ICMR. Notice that **ICMR extends past supply rails**.
///

Clearly from *Figure-04*, we can see that the ICMR for this op-amp is -0.2 V to 0.4 V (PMOS ON), then 0.85 V to 1.4 V (NMOS ON). In between these two ranges, both diff-amps simultaneously turn OFF and ON as the ICMR approaches their operating range. This range, 0.4 V to 0.85 V is called the **cross-over region** for a rail-to-rail input stage, and the term is self-explanatory as the input stage crosses over from PMOS to NMOS.

Even though there are common-mode variations in the **cross-over region** we will still include this in our overall ICMR by keeping in mind that such non-linearities are seldom avoidable.

!!! danger ""
    The **cross-over region** presents a ***HUGE HURDLE*** in frequency compensation.

## AC Simulations

### Cross-over region dictates frequency compensation

In the cross-over region, both input diff-amps are ON. And so both will contribute a change in current of \((g_{mn} + g_{mp}) * v_{in}\) into the cascode stack. And when we're outside this region, only one of the input diff-amps will contribute a change based on our input common mode voltage V~CM~.

To realize a fully compensated op-amp, we have to account for this worst case (*cross-over region*, both N and P Diff-amps ON) which makes the **op-amp slower** when only one input stage (Either N or P Diff-amp) is ON.

??? note
    There are ways to mitigate this problem by implementing constant g~m~ for all three cases (Only N, only P and both N and P Diff-amps ON) \[Ref. *Operational Amplifiers, Johan Huijsing, section 4.4*], but such modifications are beyond the scope of this documentation.

### Frequency compensation strategy

Because we have two paths for changing current in cascode stack, it makes sense that we add two compensation capacitors as shown in *Figure-05*

<a id="fig-05"></a>

![Compensation capacitors connection](./rtr-opamp-assets/13_Compensation_Strategy_dark.svg#only-dark)
![Compensation capacitors connection](./rtr-opamp-assets/13_Compensation_Strategy_light.svg#only-light)
/// caption
**Figure-05:** Connection of compensation capacitors.
///

Comparing this with [Figure-01](#fig-01), we can see that we're ***overcompensating*** the op-amp for stability across all regions of ICMR.

### Compensating the op-amp

!!! tip
    In amplifiers using one of the latter two methods (error feed forward or compensation)‚ the error at the output is not measured and corrected‚ but during design time the expected error of the active components is estimated and a correction circuit to remove this error is added.

    \- *Structured Electronic Design, Negative-feedback Amplifiers: C.J.M Verhoeven, et al., Pg 29*

The above highlighted tip is the accurate representation of design style based on compensation. And as such, let's first see the AC response of uncompensated op-amp to see what we can possibly get and to choose f~ugb~.

To that end, the testbench for measuring Open loop response is shown in *Figure-06*. The simplified schematic of Testbench is shown in *Figure-07* and the results in *Figure-08*.

<a id="fig-06"></a>

![Open Loop Gain TB](./rtr-opamp-assets/01_Uncompensated_AC_TB_dark.png#only-dark)
![Open Loop Gain TB](./rtr-opamp-assets/01_Uncompensated_AC_TB_light.png#only-light)
/// caption
**Figure-06:** Opamp schematic with open loop gain measurement configuration.
///

<a id="fig-07"></a>

![Open Loop Gain TB Simplified](./rtr-opamp-assets/14_Uncompensated_AC_TB_Simplified_dark.svg#only-dark)
![Open Loop Gain TB Simplified](./rtr-opamp-assets/14_Uncompensated_AC_TB_Simplified_light.svg#only-light)
/// caption
**Figure-07:** Simplified Schematic of Figure-06
///

![Open Loop Gain of Uncompensated op-amp](./rtr-opamp-assets/02_Uncompensated_AC_Results_dark.svg#only-dark)
![Open Loop Gain of Uncompensated op-amp](./rtr-opamp-assets/02_Uncompensated_AC_Results_light.svg#only-light)
/// caption
**Figure-08:** Open Loop Gain of uncompensated op-amp in Figure-06
///

We can see that:

\[DC~Gain,~A_{DC} = 74.9~dB\]

\[Unity~Gain~frequency,~f_{ug} = 136.918~MHz\]

Improving DC gain is possible by implementing drain regulation in the cascode stack using auxiliary amplifiers. And the unity gain is 136.918 MHz, so we can set it to anything below that by adding compensation circuitry (just capacitors as shown in [Figure-05](#fig-05)).

I am not touching DC Gain here as our objective is just frequency compensation. In that case, let's make ***100 MHz*** as our unity gain bandwidth using compensation (as I want my op-amp as fast as possible after all).

Remember that each of these capacitors affect their respective current change paths (Actually they do affect other ones too, but for the sake of design, this compromise is good enough for a first order approximation). And we also know the transconductance of input MOSFETs from [Regular Threshold Voltage (RVT)](/mosfet/parameters/#regular-threshold-voltage-rvt) table which is tabulated below for convenience:

| Flavour | g~m~ | COMMENTS |
|---------|------|----------|
| NMOS-RVT | 115.83 µA/V | At VDS = 0.1 V |
| PMOS-RVT | 130.51 µA/V | At VDS = 0.1 V |
/// caption
**Table-02:** Transconductance of input MOSFETs summary
///

And the equation for unity gain frequency is:

\[\tag{1} f_{ugb} = \frac{g_{m}}{2* \pi * C_{c}}\]

We will just use NMOS transconductance, g~mn~, and 100 MHz as our f~ugb~ to find the value for capacitance.

!!! warning
    As for why we're not using PMOS transconductance as well is because *Table-02* has approximate values for g~m~ (Depends on operating point) and trying to do precision design with that is stupidity.

    Finding a capacitance from *Equation-01* merely gives us a starting point (kind of like an initial guess) to our design, and then we will iterate from there in simulations.

    Again, do not attempt for precision design by hand.

Plugging values into *Equation-01* gives,

\[C_{c} = \frac{g_{m}}{2 * \pi * f_{ugb}} = \frac{115.83 µ}{2 * \pi * 100 M} = 184 fF \approx 190 fF\]

### Iteration 1

Let's see the open loop response for three different values: 190 fF, 250 fF, and 300 fF. As for how I got those other two values, do note that 190 fF is close to 200 fF. And from 200 fF I took two other values with a step of 50 fF each. If this proves not enough, we can use a step of 100 fF or even 200 fF between consecutive values in each of our simulations.

Also, I won't replace 190 fF with 200 fF, since it would be useful to see what response our hand-calculated value yields.

The testbench remains same as [Figure-07](#fig-07), with the modifications on [Figure-05](#fig-05) applied to schematic of [Figure-06](#fig-06) with capacitor values of 190 fF, 250 fF and 300 fF in each step.

The results are