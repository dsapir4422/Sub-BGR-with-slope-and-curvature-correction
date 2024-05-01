# Sub-BGR-with-slope-and-curvature-correction
In this project we will show the design of a Sub Band Gap Reference (BGR) circuit with slope and curvature correction in 180nm technology.

**Specifications**
* $V_{REF} =  1.0[V]$
* $V_{DD} = 1.8[V]$
* $TC = dVout/(V_{avg}*dT) < 30[PPM]$
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

**PTAT, CTAT model**

First we build a model to simulate CTAT and PTAT behavior, and calculate their slope - 

<img width="400" height="400" alt="image" align=left src="https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/9afe0f64-aa2d-47c5-9f3c-9aa816a49213">
<img width="450" height="450" alt="image" align=center src="https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/80261444-6092-4b14-979e-4abf37773735">

We can see that the slope correction is R3/R1 = 1.416/0.211 = 6.75.

We can also simulate the bandgap voltage, as we can see it has a curvature behavior (after slope cancellation) - 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/909f8d06-218d-4cfd-ba1c-19d5ad054a24)

**BGR Final circuit**

We chose the following values based on hand calculations & tool simulations (see attach excel) - 
* R1=12[KOhm] for ~ 10[uA] current on each branch
* N=8 for symmetric layout
* R3=81[KOhm] according to calculations
* R4=70.5[KOhm] according to calculations


Applying all of the above into a final circuit - 
![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/87fe6670-fc52-4f7f-8e78-832cda12ef5a)

Vout (e.g VREF) with slope trimming only - 
![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/ecbaa6b6-66e6-4538-a6ff-d8f7b68446d7)

which corresponds to a TC = 33.5[PPM].

After applying curvature correction we reduced TC = 27[PPM] - 
![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/db8ca726-b1c8-47ac-aaca-9add8e427cb1)

Note - we are still seeing curvature curve but peak2min value dropped by 2mV.

Looking at DC range, we see a change of 17mV over +/-10% supply change - 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/2423293c-db68-495d-b3d8-d0d1dae1c7c1)

**Start-up circuit**

BGR has 2 solutions for Ids, this is why we need a start-up circuit, and the goal is to increase CTAT branch voltage after VDD stabilize.

We chose the following circuit - 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/01a8d37f-4799-4fab-bdd6-1c0980a906f6)

where M12 is generating a current which starts the op-amp and raises CTAT voltage. once M0 $V_s > V_d$ , it will go to cut-off and no current will flow in start-up circuit. 

From transient simulation we can see Vout start-up after 9[msec] - 
![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/7edcea99-7462-4b9c-bfda-21644287b46d)

**Op-Amp**

We chose 5T-OTA with native Vth differential pair (PMOS can also work here) because of high VBE voltage (regular NMOS won't have enough head-room).
OTA design parameters - 
* A = 20dB
* PM = 92[deg]
* PSRR = 28dB

Op-amp design - 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/ad73a7db-2532-4f39-b235-65c3e3ddde39)

Stability simulation - 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/ae28e0e5-58fa-4e66-aaf4-90fcc71ac037)

PSRR - 

![image](https://github.com/dsapir4422/Sub-BGR-with-slope-and-curvature-correction/assets/87266625/2f0835ac-1230-40a7-9d86-93385f5e0bc0)








