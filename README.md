# Sub-BGR-with-slope-and-curvature-correction
In this project we will show the design of a Sub Band Gap Reference (BGR) circuit with slope and curvature correction in 180nm technology.

**Specifications**
* $V_{REF} =  1.0[V]$
* $V_{DD} = 1.8[V]$
* $TC < 30[PPM]$
* Temperature range $-40 : 140 [C]$
* $PSRR > 20[dB]$ up to 1MHz

## Design topology & calculations
Our design is based on the following circuits (Taken from Razavi - The Design of a Low-Voltage Bandgap Reference, The Analog mind - https://ieeexplore.ieee.org/document/9523469) 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/3c2e8ca7-f658-4b4b-bf44-5a2e627280ad)

with a few additions (curvature correction and start-up circuit which will be discussed later on).

**Circuit explained**

As we know $V_{BG} \approx 1.2[V]$ depends on technology. But we can design sub-BG circuits below 1.2[V] by summing currents instead of voltages in the BG core, copying the current to a new branch and adding a resistor for the output voltage.
* The current on $R_2=R_3$ is CTAT and defined as $I_{CTAT} = V_{BE}/R_3$
* The current on $R_1$ is PTAT and defined as $I_{PTAT} = dV_{BE}/R_1$
* By using opamp with negative feedback we assure node X is equal node Y and because $R_2=R_3$ we have CTAT current on both sides
* Then we can write $I_{ZTAT} = I_{PTAT} + I_{CTAT}$
* Using M3 we can copy $I_{ZTAT}$ to a new branch and adjust the output voltage using R4 (RL in Razavi's article)
* Output voltage is defined as $V_{out} = R_4/R_3 * (V_{BE} + R3/R1 * dV_{BE})$
* We note that $V_{DD,min} = V_{out} + V_{ds,satM3}$

In order to have $V_{out}$ with low PPM across temperature range we will define 2 trimming procedures -

**Slope trimming explained**

This is a 1st order issue which caused by the different slopes of PTAT and CTAT voltages, they should cancel each other in order for ZTAT to be constant across temperature.
We should simulate CTAT & PTAT currents, extract the slope from their derivative so that $R3/R1 = dCTAT/dPTAT$

**Curvature trimming explained**

After slope trimming, we still left with 2nd order issue - a curvature curve of the voltage across temperature. This is caused by a non-linear current $I_{NL} = V_t*ln(T/T_r)/R_5$ of the BJT diode-connected structure.
To cancel that current, we would copy $I_{ZTAT}$ to an additional BJT diode-connected branch and add R5 resistor in parallel to trim that current out and correct the curvature curve.
In other words, we trim - $R3/R5 = n-1$ , where 'n' is a technology parameter.

**In Summary** 

$V_{out}$ depends on the following (after adding curvature circuit correction) - $V_{out} = R_4*(I_{PTAT} + I_{CTAT} + I_{NL}) = R_4/R_3 * (V_{BE} + R3/R1 * dV_{BE} + R3/R5*ln(T/T_r))$
Where we can adjust the resistor ratio to - 
* R4/R3 - output voltage scaling
* R3/R1 - slope cancellation
* R3/R5 - curvature cancellation

## Design & Simulations


