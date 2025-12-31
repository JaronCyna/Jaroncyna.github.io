---
title: Creating a Maximum Power Point Tracking (MPPT) System For Max Charge Efficiency on Satellite
date: 2025-12-30 
categories: [pcb, embedded design]
tags: [embedded]
author: Jaron
image:
  path: /assetsweb/mppt/Watchdog.png
  alt: Logic diagram behind the Max Power Point Tracker
---

<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js">
</script>
<script>
  MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']]
    }
  };
</script>

## Introduction
As the Electrical Power Systems lead on the satellite branch of the Queen's Space Engineering Team, I am always looking for ways to enhance all aspects of the power distribution for the satellite. One of the most vital metrics on a cubesat is efficiency; energy is not an abundant source for them, so efficiently receiving as much power as possible is vital to have a functioning satellite. Due to this, I was looking into ways to improve efficiency, which is how I found out about Maximum Power Point Tracking (MPPT). This technology is used on variable power sources, such as solar panels, to vary apparent impedance at the source and always output the max power regardless of light intensity.

## Process
Starting off, it is important to get a high-level overview of how the MPPT will function. At a high level, an MPPT is a DC-DC converter with a duty cycle that varies with the variable source to ensure max power at the current state. So my first draft was essentially this in its most bare-bones state: the solar panels attached to a DC-DC converter of some sort, connecting to a battery, then a buck converter goes from the solar array to a STM32, which then powers the duty cycle of the MPPT.

![](/assetsweb/mppt/diagram1.png)

Although this is a good starting point, there are several problems that need to be addressed before moving forward with the design phase of this project. First of all, it would be less reliable to get energy to the STM from the solar panels as there can be times where minimal energy is received. Due to this, it would be smart to power the STM on a stable source like a battery. However, that presents a new problem: what happens when the batteries are dead? How does the STM start up to get the MPPT to start charging the batteries to power the STM? Additionally, there would need to be a way to power the MOSFETs on the MPPT as the STM runs on 3.3V—not nearly enough to power them. 

So for my next design I made the following changes: I had a buck converter which received power from batteries and solar panels, and an Ideal Diode which would compare and essentially check if the batteries were dead to see what to power the STM32 with. Then there would be a watchdog ensuring space radiation does not cause problems with the STM32, and if it does, it can be reset. Then a gate driver would run from the STM to the MPPT to ensure it can power the MOSFETs.

Below is the diagram used in KiCad to guide the design.

![](/assetsweb/mppt/diagram2.png)


## Design

Now that the general overview of the project has been outlined, the design process should begin. To start, getting the voltage of the solar array and battery is important. Although not finalized, I can make a safe assumption that 24 AZUR Space Triple Junction solar panels will be on the satellite along with 8 Lithium-Ion Batteries. After some consideration, I found that I should organize the solar panels in a 3s8p (series parallel) orientation allowing for a VOC of 8.1V and ISC of about 4A, and the batteries in a 2s4p orientation allowing for a max charge voltage of 8.4V. I chose these to be close to each other as a DC-DC converter is most efficient for smaller boosts. Additionally, the 8.4V is a good middle ground between 3.3V and 12V, which would allow for fairly efficient conversions to other logic levels. 

Now that this is decided, and it is confirmed that the solar array will never get above the 8.4V max charge of the battery, the MPPT can be chosen to be a boost converter, which can only raise voltage. Although a buck-boost would add versatility in the MPPT, the extra space it takes up is likely not worth it for the gains.

The following is a completed table of the specs of the MPPT:

| Parameter | Value |
| :---: | :---: |
| V_in max | 8.1 V |
| I_in max | 4.16 A |
| V_out min | 5.0 V |
| V_out max | 8.4 V |
| I_out max | 5.4 A |
| P_out (max) | 26 W |

### Simulation
Now that all of these specs have been selected, it is advantageous to run a simulation for the proof of concept. To do this, I used the standard boost converter equations for inductance and capacitance. *Note: a freq of 200 kHz was assumed as that is a typical frequency for MPPTs controlled by STM32s.*

The selection of the Inductor ($L$) and Capacitor ($C$) is not arbitrary; it is driven by the need to maintain **Continuous Conduction Mode (CCM)** and to meet specific **signal integrity** targets.

### 1. Determining Minimum Inductance ($L_{min}$)
The primary goal for the inductor is to prevent the current from dropping to zero during the switching cycle. This boundary is known as the **Critical Inductance**.

**The Design Constraint:**
To stay in CCM, the inductor must be larger than:

$$L_{min} = \frac{V_{in} \cdot D}{2 \cdot I_{out} \cdot f_s} \cdot (1-D)^2$$

**Calculation for this Project:**
Using our parameters ($V_{in}=8.1V$, $D=0.0357$, $I_{out}=5.4A$, $f_s=200kHz$):

$$L_{min} = \frac{8.1 \cdot 0.0357}{2 \cdot 5.4 \cdot 200,000} \cdot (1-0.0357)^2 \approx \mathbf{0.12 \mu H}$$

**Decision Logic:** While the mathematical minimum to function is only $0.12 \mu H$, a value of **$33 \mu H$** was chosen. This provides a massive safety buffer, ensuring the converter stays in CCM even if the load current drops significantly (to roughly $20mA$), which prevents the unpredictable voltage spikes associated with Discontinuous Mode (DCM).

---

### 2. Determining Required Capacitance ($C$)
The output capacitor is decided by the maximum allowable "dip" in voltage while the switch is closed and the inductor is disconnecting from the load.

**The Design Constraint:**
The required capacitance is a function of the desired voltage ripple ($\Delta V_{out}$):

$$C = \frac{I_{out} \cdot D}{f_s \cdot \Delta V_{out}}$$

**Calculation for this Project:**
If we set a strict target for high-performance signal integrity—such as a **10mV maximum ripple**—we can solve for the required $C$:

$$C = \frac{5.4 \cdot 0.0357}{200,000 \cdot 0.010} \approx \mathbf{96.3 \mu F}$$

**Decision Logic:** A standard **$100 \mu F$** value was selected. This choice directly satisfies the $10mV$ ripple target while allowing for "Capacitance Derating" (the fact that ceramic capacitors lose some effective capacitance when operating at their rated DC voltage).

---

### Summary of Selection Criteria

| Component | Governing Constraint | Threshold | Chosen Value | Engineering Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **Inductor** | CCM Boundary | > 0.12 µH | **33 µH** | Stability at light loads. |
| **Capacitor** | 10mV Ripple Target | > 96 µF | **100 µF** | Signal integrity & DC derating. |

### LTSpice Sim
By putting all of these values into LTSpice, and deciding on a synchronous boost converter for the extra efficiency gains compared to using a diode, the following plots were made, thus validating choices made. This was done at a min and max voltage with different duty cycles to ensure the voltage could stay the same.

![](/assetsweb/mppt/LT1.png)
![](/assetsweb/mppt/LT2.png)

After this success, starting the design for the PCB schematic was the next step.

## Schematic
### Watchdog
Although separate from what I was previously testing, I decided to start the Schematic with the watchdog. For this, I decided to use a finite state machine which was controlled by D Flip-Flops and a 555 timer clock. The idea behind this was ensuring that this is purely deterministic and hardware-based means there is a lower likelihood of it failing in space. Additionally, it is made out of radiation-hardened components. Below is the design process I used, along with the schematic I made for it in KiCad.

![](/assetsweb/mppt/D_Flip.jpg)
![](/assetsweb/mppt/Watchdog.png)

### MPPT
Now for the MPPT, most of the work for this had been done previously, so I mostly just needed to find a gate driver which could drive the MOSFETs and then it was smooth sailing from there. However, I did add voltage sensors using voltage dividers and Current sensors using Shunt resistors.

![](/assetsweb/mppt/MPPT.png)

### 3.3V Logic
For powering the STM32, I used a buck converter IC which is capable of keeping a 3.3V output with a large variation of input voltages, and I used an ideal diode to separate the battery and solar panel but still allow for one to go through regardless of the situation the battery ends up in.

![](/assetsweb/mppt/33V.png)

### STM32

For the STM32, I ended up choosing between the L4 series for its low power and G4 series for its HRTIM which would increase the resolution of the PWM.

| Feature | STM32L4 | STM32G4 |
| :--- | :---: | :---: |
| **Current Consumption** | 100 µA/MHz | 160 µA/MHz |
| **Active Current** | 8 mA (80 MHz) | 27 mA (170 MHz) |
| **Wattage (3.3V)** | 26 mW | 89 mW |
| **Precision (@ 200kHz)** | 200 Steps | 13,586 Steps |

Due to the incredible extra Precision of the G4, it would get a lot closer to the true max power point, which means that a near 30W output should offset the extra power consumed. To prove this, the following shows the calculations that justify this:

### Duty Cycle Resolution Calculation
The resolution is the smallest possible increment in the duty cycle ($D$), calculated as:

$$\text{Resolution} = \frac{1}{\text{Total Steps}} \times 100\%$$

**STM32L4 (Standard Timer):**

$$\text{Res}_{L4} = \frac{1}{200} = 0.005 = \mathbf{0.5\%}$$

**STM32G4 (High-Resolution Timer):**

$$\text{Res}_{G4} = \frac{1}{13,586} \approx 0.0000736 = \mathbf{0.00736\%}$$

---

### Comparative Analysis

| Metric | STM32L4 | STM32G4 | Difference/Factor |
| :--- | :---: | :---: | :---: |
| **Control Steps (@ 200kHz)** | 200 | 13,586 | **67.93x More Steps** |
| **Duty Cycle Step Size** | 0.5% | 0.0074% | **67.93x Finer Control** |

---

The **67.93x resolution difference** fundamentally changes the performance of the Boost Converter:

* **Reduced "Hunting" Loss:** With the **STM32L4**, a $0.5\%$ step is often too "coarse." If the ideal peak power is at a $3.75\%$ duty cycle, the L4 must oscillate between $3.5\%$ and $4.0\%$, creating a "steady-state ripple" that wastes energy.
* **True Peak Tracking:** The **STM32G4** can move in increments of $0.0074\%$. This allows the control loop to "lock" onto the exact peak power point with virtually zero oscillation.
* **Conclusion:** The trade-off of using an extra $63\text{ mW}$ of MCU power (STM32G4) is justified because the precision gains allow the converter to harvest significantly more power from the solar panel—often increasing total system efficiency by several percentage points compared to low-resolution control.

### STM32 Schematic

> Past This point is a work in progress

## TODO
- Wire the STM32G4 in KiCAD
- Explain and fix issue with temperature varying voltages to unsafe levels
- Add more space safety features to ensure it would work after launch
- Integrate with battery board
- Make PCB Layout