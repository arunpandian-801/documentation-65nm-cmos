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

## Introduction

I have documented only the design procedure, and anything relevant to that. For theory of operation, please refer to standard textbooks or any lectures.

Before we proceed, let's take a look at the schematic of Parallel BGR.

<a id="fig-01"></a>

![Parallel BGR Schematic](./bgr-low-voltage-assets/01_SchematicOfParallelBGR_dark.svg#only-dark)
![Parallel BGR Schematic](./bgr-low-voltage-assets/01_SchematicOfParallelBGR_light.svg#only-light)
/// caption
**Figure-01:** Parallel BGR schematic diagram \[Ref. CMOS Circuit Design, Layout and Simulation, Fig 23.30]
///

Just like the series BGR, same parasitic PNP Diode characterized in [PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20) is used here as well, and so the parameters listed there are summarized in *Table-01:*

<a id="table-01"></a>

| Parameter | Value | Comments |
|-----------|-------|----------|
| η | 1003.6 m | Forward current emission coefficient (Emitter-Base junction) |
| V~BE~ | 0.777 V | 1 Diode (At a current of 5 µA) |
| \(\delta V_{BE}/\delta T\) | -1.75 mV/°C | Change in the diode’s voltage with temperature (at current of 5 µA) for 8 Diodes in parallel |
/// caption
**Table-01:** PNP Diode summary
///

!!! example ""
    Refer to [Introductory section of series BGR](../references/bgr-standard.md#introduction) for explanation on why 8 diodes is chosen. I chose not to repeat it here to avoid redundancy.

### Target output for this topology - 0.6 V (V~DD~\/2)

Since this topology allows to establish an output voltage less than the bandgap voltage of silicon (about 1.22 V), let's aim for an output voltage of 0.6 V as it is halfway through our supply rails (V~DD~\/2).

This value is chosen arbitrarily and could be set to anything else.

!!! danger ""
    Just make sure that it is not too close to supply rails, i.e. neither to close to V~DD~ (1.2 V) nor GND (0 V) as such extremes are difficult to handle for a MOSFET which demands some minimum voltage across it and not too much at the same time!

### Design Considerations

For designing this, we need to find:

- Sizes for all MOSFETs (Current sourcing PMOS as well as for building error-amp).
- PTAT Current Generator resistor, R.
- CTAT Current Generator resistor, L\*R.
- Output voltage resistor, N\*R for a nominal output of 0.6 V (V~DD~\/2).

The MOSFET sizes are already chosen and their parameters are thoroughly listed in [Core Device — 65 nm](../mosfet/parameters.md#core-device-65-nm) table. Unless specified otherwise, these sizes will be the ones we use to construct the entire circuit.

And as such, these are tabulated here for convenience:

<a id="table-02"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ | V~GS~ |
|---------|-------------|--------------|------|-------|
| NMOS | 43.1 / 2 | 2.8 µm / 130 nm | 10 µA | 500 mV |
| PMOS | 121.5 / 2 | 7.9 µm / 130 nm  | 10 µA | 470 mV |
/// caption
**Table-02:** Sizes and Bias Current summary
///

!!! question
    The observant reader would have questioned these sizing choices considering [Table-01](#table-01) characterizes PNP Diodes for 5 µA while here it is characterized for 10 µA. This is not a problem as, this time, we're designing parallel BGR ***which generates a sum of PTAT and CTAT current***. So it will *not just be 5 µA* needed for PTAT current generator, but **also some more** for CTAT current generator as well.

    On the occasion that the sum of PTAT and CTAT currents exceed 10 µA, the error-amp steps in to adjust the V~SG~ of top PMOS in [Figure-01](#fig-01) to source the necessary current as long as it is in the vicinity of 10 µA.

    !!! warning "This puts a constraint on Error amplifier output swing limit"
        To allow PMOS with these sizes to source more current, it becomes necessary that the error-amp's output should be capable of swinging freely. This is discussed in [Added error amplifier](#added-error-amplifier) section.

### Value for PTAT Current Generator resistor, R

Given the same operating point (5 µA) and same PNP Diode ([PNP 20 X 20](../mosfet/BJT-parameters.md#pnp-20-x-20)), the value of *PTAT Current Generator Resistor, R, remains the same as the [one calculated for Series BGR](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1).* 

And it's value is: \(R = 10.9 ~k\Omega\)  
Refer to [Fixing PTAT Current Generator resistor, R~1~, section of Series BGR](../references/bgr-standard.md#fixing-ptat-current-generator-resistor-r1) for information regarding the testbench and method to find this value.

## Fixing CTAT Current Generator resistor, L\*R

The PTAT current is generated by dropping ΔVBE across resistor R.

<a id="eqn-01"></a>

\[\tag{1} I_{PTAT} = \frac{\eta * V_T * \ln K}{R}\]

And the CTAT current is generated by dropping VBE across resistor L\*R.

<a id="eqn-02"></a>

\[\tag{2} I_{CTAT} = \frac{V_{BE}}{L*R}\]

The total current is the sum of I~PTAT~ and I~CTAT~.

\[I_{Total} = I_{PTAT} + I_{CTAT}\]

Making this sum total temperature independent means:

\[\frac{\delta I_{Total}}{\delta T} = \frac{\delta I_{PTAT}}{\delta T} + \frac{\delta I_{CTAT}}{\delta T} = 0\]

Substituting [Equation-01](#eqn-01) and [Equation-02](#eqn-02) into the above equation yields,

\[\frac{\delta I_{Total}}{\delta T} = \frac{\delta}{\delta T} (\frac{\eta * V_T * \ln K}{R}) + \frac{\delta}{\delta T} (\frac{V_{BE}}{L*R}) = 0\]

\[\frac{1}{R} [ \eta * \ln K * \frac{\delta V_T}{\delta T} + \frac{1}{L} * \frac{\delta V_{BE}}{\delta T} ] = 0 \]

\[\tag{3} L = \frac{- \delta V_{BE} / \delta T }{\eta * \ln K * \delta V_T / \delta T} \]

The values for:

- η is known from [Table-01](#table-01) as 1003.6 m.
- K is chosen to be 8 (from [Introductory section of series BGR](../references/bgr-standard.md#introduction))
- V~T~ is thermal voltage and δV~T~/δT is 0.085 mV/°C.
- δV~BE~/δT is also known from [Table-01](#table-01) as -1.75 mV/°C.

Therefore, the value of L is:

\[L = \frac{- (-1.75m)}{1003.6m * \ln 8 * 0.085m} = 9.8653\]

and the value of L\*R resistor is:

\[L * R = 9.8653 * 10.9k\Omega = 107.5 ~k\Omega\]

!!! note
    This value of L\*R is just a starting guess. That is, we will start our design with this value, and later adjust it to get better temperature performance in iterations.

## Added error amplifier

<a id="fig-02"></a>

![Two stage Op amp schematic](./bgr-low-voltage-assets/02_05_ErrorAmp_schematic_dark.png#only-dark)
![Two stage Op amp schematic](./bgr-low-voltage-assets/02_05_ErrorAmp_schematic_light.png#only-light)
/// caption
**Figure-02:** Error amplifier schematic (Fully compensated Two stage op-amp)
///

The error amplifier for the design of parallel BGR is a simple two stage op-amp depicted in *Figure-02*. Much detail regarding choosing topology for an error amplifier is discussed in [Added error amplifier section of series BGR](../references/bgr-standard.md#added-error-amplifier) and so, to avoid redundancy, it is not covered here.

Still, here's a short explanation for why we chose this topology for our error amplifier:

1. *NMOS input stage is used to accomodate common mode voltage of 0.777 V*.

    Common mode voltage which appears at the input of error amplifier is 0.777 V (See V~BE~ in [Table-01](#table-01)) as stated in [Prerequisites](#prerequisites) and also in [Input common mode constraint section of series BGR](../references/bgr-standard.md#inherent-constraint-on-input-common-mode-of-nmos-and-pmos-diff-amp).

    However, 0.777 V is close to VDD (1.2 V) in this design (which uses the core-device 65nm), unlike in the [Series BGR](../references/bgr-standard.md), where it was closer to GND — since that version uses  I/O device 500nm with a higher VDD of 3.3 V.

    Hence, this mandates an NMOS input stage as discussed in [Input common mode constraint section of series BGR](../references/bgr-standard.md#inherent-constraint-on-input-common-mode-of-nmos-and-pmos-diff-amp).

2. *Output swing should be high to adjust V~SG~ of top PMOS in [Figure-01](#fig-01) to source necessary current*.

    Since we're doing current summation (I~PTAT~ + I~CTAT~), our total current of operating point is not just 5 µA (I~PTAT~). So, our error-amp must adjust Gate voltage of sourcing PMOS to source this current while maintaining very little difference in the regulated nodes. This calls for two stages (more gain) with a common source stage at the output (more swing).

### Concerning ICMR

This error-amp has an ICMR of 0.6 V to 1.2 V as shown in *Figure-03*. But sometimes, the lower limit of ICMR may come closer to our voltage of interest (which in this case is 0.777 V). When that happens, it is necessary to try to increase it and **keep some margin for error.**

![ICMR of error amp RVT](./bgr-low-voltage-assets/02_01_ICMR_RVTNMOSInput_dark.svg#only-dark)
![ICMR of error amp RVT](./bgr-low-voltage-assets/02_01_ICMR_RVTNMOSInput_light.svg#only-light)
/// caption
**Figure-03:** ICMR of error-amp shown in [Figure-02](#fig-02)
///

For example, when ICMR~min~ is 0.7 V, there is a chance it may increase to 0.75 V due to process variations and that is not good as it is closer to our desired value of 0.777 V. It's *always better to leave some margin* for error.

**One simple way to increase ICMR is to replace just the input NMOS with LVT** (Low Threshold Voltage) variant. Just to demonstrate it, let's use the sizes listed in [Low Threshold Voltage (LVT)](../mosfet/parameters.md#low-threshold-voltage-lvt) table which is repeated here in *Table-03* for convenience.

<a id="table-03"></a>

| Flavour | W/L (Drawn) | W/L (Actual) | I~D~ |
|---------|-------------|--------------|------|
| NMOS-LVT | 38.5 / 2 | 2.5 µm / 130 nm | 10 µA |
/// caption
**Table-03:** Sizes and bias current summary for LVT NMOS
///

And the resulting ICMR is 0.5 V to 1.2 V, (a 0.1 V extra margin from our RVT variant) which can be seen in *Figure-04*.

![ICMR of error amp LVT](./bgr-low-voltage-assets/02_02_ICMR_LVTNMOSInput_dark.svg#only-dark)
![ICMR of error amp LVT](./bgr-low-voltage-assets/02_02_ICMR_LVTNMOSInput_light.svg#only-light)
/// caption
**Figure-04:** ICMR of error-amp shown in [Figure-02](#fig-02) with LVT NMOS (Sizes from [Table-03](#table-03)) as input
///

I will go with RVT as it already has 0.777 V (V~BE~) in it's ICMR.

### Concerning Stability

Both [Series BGR](../references/bgr-standard.md) and [BMR](../references/bmr.md) utilized single-stage error amplifiers, each contributing one pole to the feedback loop, while the current generator added another pole, resulting in the loop having a total of two poles, which is easy to compensate by making the output of error amp the dominant pole.

The selection of a two-stage error amplifier introduces an additional pole from the second stage into the loop, bringing the total number of poles in the loop to three, which significantly **threatens the stability** of the overall loop.

However, before we can attempt to resolve this issue, we need to construct the current generator section (and consequently, the complete loop) in order to analyze the stability of the loop.

![Feedback loop with poles](./bgr-low-voltage-assets/08_01_FeedbackLoop_Poles_dark.svg#only-dark)
![Feedback loop with poles](./bgr-low-voltage-assets/08_01_FeedbackLoop_Poles_light.svg#only-light)
/// caption
**Figure-05:** Feedback loop with nodes that contribute poles
///

!!! note "Of course it is not three poles, but four"
    I know it is four poles in total. But generally P~3~ and P~3~' seen in *Figure-05* tends to be low impedance nodes (or high frequency poles) and will be close to each other. Even though they are non-dominant, they **still contribute phase lag, eating up our phase margin.**

#### Open Loop Gain of error-amp

We will look at stability again when we completed our DC temperature characteristics, but in the meantime, let's take a look at the Open loop Response of error-amp of [Figure-02](#fig-02) with loading as shown in *Figure-06*.

<a id="fig-06"></a>

![Open loop Gain measurement](./bgr-low-voltage-assets/02_06_ErrorAmp_AC_TB_Loaded_dark.png#only-dark)
![Open loop Gain measurement](./bgr-low-voltage-assets/02_06_ErrorAmp_AC_TB_Loaded_light.png#only-light)
/// caption
**Figure-06:** Open loop gain measurement testbench for error-amp of [Figure-02](#fig-02)
///

Notice that in *Figure-06*:

- The output of the error amplifier includes ***a large NMOS capacitor***, consisting of 20 NMOS transistors each with a `20/20` size amounting to **roughly 400 fF** (Chosen arbitrarily) (Refer to [Core device RVT MOSFET as a Capacitor](../mosfet/parameters.md#core-device-65-nm_2) table), acting as a load. This setup is used to select a compensation capacitor C~c~ that provides an adequate phase margin.

!!! warning
    Adding a capacitor to the output of a two stage amplifier generally degrades it's phase margin and threatens stability. In order to avoid that, I decided to design this error amp with a capability to ***accommodate an intentional capacitor load at the output***.

    In other words, I have designed for a margin, where even IF I add a capacitor, as long as it is less than 400 fF (nominal capacitance of 20 NMOS of `20/20` size each), I know it will be stable.

    On the rare occasion that I do need to drive an even larger capacitor, I have no choice but redesign the error-amp.

- The output also has the ***current generator PMOS*** (2 of them in parallel to model the two PMOS which loads the error amp) with the pull down part which should have included PNP Diodes along with resistors substituted with a voltage source of 0.777 V to model them. The rationale behind this is explained in [Substitution theorem section of BMR](../references/bmr.md#substitution-theorem). This models the parasitics due to the PMOS which loads the error amp.

The open loop gain of [Figure-06](#fig-06) is shown in *Figure-07* with a C~c~ of 200 fF.

!!! note "As for how I got this C~c~ of 200 fF"
    The procedure to compensate an amplifier is extensively covered in [Compensating the op-amp section of Rail-to-rail opamp](../amplifiers/rtr-opamp.md#compensating-the-op-amp). Refer to that, as I don't want to repeat it here to avoid redundancy.

![Open Loop Gain of error-amp](./bgr-low-voltage-assets/02_07_ACResp_TwoStage_200fF_Loaded_dark.svg#only-dark)
![Open Loop Gain of error-amp](./bgr-low-voltage-assets/02_07_ACResp_TwoStage_200fF_Loaded_light.svg#only-light)
/// caption
**Figure-07:** Open Loop gain of error amp of [Figure-02](#fig-02) with a C~c~ of 200 fF and a load of 400 fF
///

And the parameters are summarized in *Table-04*:

| Parameter | Value |
|-----------|-------|
| C~c~ | 200 fF |
| A~DC~ | 47.9 dB |
| f~ugb~ | 165.9 MHz |
| Φ~ugb~ | -98° |
| Φ~margin~ | 82° |
/// caption
**Table-04:** AC open loop response key parameters for VCM of 0.777 V (V~BE~)
///

!!! danger ""
    This doesn't paint the full picture yet. Until we see the loop gain, we can never be confident about the stability.

## Fixing output voltage resistor, N\*R

Let's build the core current generator portion of [Figure-01](#fig-01) and see whether if it works. 

![DC Annotated Current Generator](./bgr-low-voltage-assets/03_01_Itr1_DCAnnotatedCurrGenerator_dark.png#only-dark)
![DC Annotated Current Generator](./bgr-low-voltage-assets/03_01_Itr1_DCAnnotatedCurrGenerator_light.png#only-light)
/// caption
**Figure-08:** DC Annotated schematic of core Current Generator
///

The values for R and L\*R are found to be **10.9 kΩ** and **107.5 kΩ** from [Value for PTAT Current Generator resistor, R](../references/bgr-low-voltage.md#value-for-ptat-current-generator-resistor-r) and [Fixing CTAT Current Generator resistor, L*R](../references/bgr-low-voltage.md#fixing-ctat-current-generator-resistor-lr).

From *Figure-08*, our design so far seems to at least bias up properly. If this was not the case, we have to fix it, before we proceed to output.

!!! danger "Sometimes, even DC Op Point needs a startup circuit"
    If you ever see that your current generator doesn't bias up to it's operating point, try to replace your error-amp with an ideal VCVS of appropriate gain (say 10,000). Generally a VCVS biases up the current generator without the need of a startup.

    But if even that fails, **do add a startup circuit** as it at least tries to induce a voltage to startup the circuit. It could be useful to troubleshoot the error.

Now that this is done, let's focus on fixing a value for output resistor N\*R.

Noting that these MOSFETs have short channel, trying to use equations for computing N\*R resistor leads to errors. Instead, let's use the Substitution theorem to find this value. This way is explained in [Fixing Resistor value section of BMR](../references/bmr.md#fixing-resistor-value) and so refer to that for more information.

![Substitution for output resistor](./bgr-low-voltage-assets/03_02_Itr1_OutputResistorSubstitute_dark.png#only-dark)
![Substitution for output resistor](./bgr-low-voltage-assets/03_02_Itr1_OutputResistorSubstitute_light.png#only-light)
/// caption
**Figure-09:** Output resistor N\*R substituted with a voltage source of 0.6 V
///

*Figure-09* gave the current flowing through the voltage source as 13.629 µA and from this, we can compute the impedance looking into the voltage source (the impedance which needs to be present to generate 0.6 V, in other words the value of N\*R) (Refer to [Resistance computation testbench](../references/bmr.md#resistance-computation-testbench)).

\[Output ~voltage ~resistor, ~N * R = \frac{0.6}{13.629 \mu} = 44 k\Omega\]