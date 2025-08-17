# Project Report: CMOS Circuit Design & SPICE Simulation with the SKY130 PDK

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/53319ae6-ecb7-438e-bd98-cf98ce40a0c0" />

## 1. Executive Summary

This report provides a comprehensive analysis of CMOS circuit design, simulation, and characterization, based on the hands-on "NgspiceSky130" workshop by [VLSI System Design]( https://www.vlsisystemdesign.com/). Using the open-source **SkyWater 130nm (SKY130) PDK** and **NGSPICE**, this project documents the end-to-end workflow from fundamental device physics to system-level performance considerations.

The analysis begins with the characterization of NMOS I-V curves, exploring the impact of short-channel effects like **velocity saturation**. It then progresses to the design and optimization of a CMOS inverter, covering its static (VTC, Noise Margin, Switching Threshold) and dynamic (propagation delay) behavior. A key focus was placed on **robustness evaluation**, systematically analyzing the inverter's resilience to power supply and device variations through scripted, parametric SPICE simulations. The project concludes by connecting these circuit-level metrics to their real-world impact on digital systems, such as in clock networks and **Static Timing Analysis (STA)**.

## 2. Table of Contents

1.  [Executive Summary](#1-executive-summary)
2.  [Table of Contents](#2-table-of-contents)
3.  [Tools and Technology Stack](#3-tools-and-technology-stack)
4.  [Experiments and Analysis](#4-experiments-and-analysis)
    *   [4.1. Experiment 1: NMOS I-V Characterization](#41-experiment-1-nmos-i-v-characterization)
    *   [4.2. Experiment 2: CMOS Inverter Static and Dynamic Analysis](#42-experiment-2-cmos-inverter-static-and-dynamic-analysis)
    *   [4.3. Experiment 3: Inverter Robustness Evaluation](#43-experiment-3-inverter-robustness-evaluation)
5.  [Key Learnings and Skill Development](#5-key-learnings-and-skill-development)
6.  [Conclusion](#6-conclusion)
7.  [Acknowledgements](#7-acknowledgements)

## 3. Tools and Technology Stack

| Category                 | Tool/Technology        | Description                                                       |
| ------------------------ | ---------------------- | ----------------------------------------------------------------- |
| **Process Design Kit**   | SkyWater SKY130        | An open-source 130nm CMOS process for design and fabrication.     |
| **SPICE Simulator**      | NGSPICE                | Open-source mixed-signal circuit simulator for performance analysis. |
| **Simulation Environment** | Command Line (Linux)   | Utilized for running SPICE simulations and processing results.    |
| **Process Corners**      | TT (Typical-Typical)   | Primary process corner used for baseline simulations.             |

---

## 4. Experiments and Analysis

This section details the key laboratory exercises, including the theoretical background, simulation setup, results, and analysis for each experiment.

### 4.1. Experiment 1: NMOS I-V Characterization

#### **Objective**
To simulate and analyze the current-voltage (I-V) characteristics of a SKY130 NMOS transistor, observing its behavior in different regions of operation and the impact of short-channel effects like velocity saturation.

#### **Theoretical Background**
The operation of an NMOS transistor is governed by its gate-to-source (`Vgs`) and drain-to-source (`Vds`) voltages. Key concepts include the **Threshold Voltage (Vt)**, the minimum `Vgs` required to form a conductive channel, and the distinct regions of operation: cut-off, linear (resistive), and saturation.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/47664757-b6de-4b23-9c03-65585f06d1c0" />
*Figure 1: NMOS Characteristics: Cut-off, Linear, and Saturation Regions.*

For modern, short-channel devices (<250nm), the classical quadratic current-voltage relationship in saturation is modified by **velocity saturation**, where carrier velocity no longer increases linearly with the electric field. This causes the drain current to become linearly dependent on `Vgs` and saturate earlier, impacting device performance.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/9739fbbe-e3f8-4b54-8674-1c499e9a8c2b" />
*Figure 2: Carrier velocity saturates at high electric fields.*

#### **Lab Activity: I-V Curve Simulation**
An NMOS transistor (`sky130_fd_pr__nfet_01v8`) was simulated in NGSPICE. A DC sweep of `Vds` from 0V to 1.8V was performed for various `Vgs` values to generate the characteristic I-V curves.

**SPICE Netlist:**
```spice
* Model Description
.param temp=27

* Including sky130 library files
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

* Netlist Description
XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
R1 n1 in 55
Vdd vdd 0 1.8V
Vin in 0 1.8V

* Simulation commands
.op
.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2

.control
run
display
setplot dc1
.endc

.end
```
#### **Results and Analysis**
The simulation produced the Id vs. Vds plot (Figure 3), clearly showing the linear and saturation regions. The curves flatten out due to channel length modulation and velocity saturation. A second simulation sweeping Vin (Vgs) at a fixed Vdd was used to generate the Id vs. Vgs curve (Figure 4) for graphical extraction of the threshold voltage.

![alt text](https://user-images.githubusercontent.com/89193562/132675399-e8f69dc7-f222-4e91-81fc-4cb2639213d4.JPG)
Figure 3: Id vs. Vds characteristics, demonstrating the onset of velocity saturation in a short-channel device.	

![alt text](https://user-images.githubusercontent.com/89193562/132675655-f779b9be-bcee-4d31-8a62-6204bc0bca40.JPG)
Figure 4: Id vs. Vgs curve used for the graphical extraction of the threshold voltage (Vt).


### 4.2. Experiment 2: CMOS Inverter Static and Dynamic Analysis

#### **Objective**
To design a CMOS inverter using SKY130 transistors and characterize its static Voltage Transfer Curve (VTC) and dynamic (transient) performance, including propagation delay and the impact of transistor sizing.

#### **Theoretical Background**
A CMOS inverter consists of a PMOS and an NMOS transistor. Its VTC is derived by superimposing the load curves of the two transistors. The intersection points for different input voltages define the output voltage, resulting in a characteristic curve with a high-gain transition region.

Figure 5: The inverter's VTC is determined by the intersection of the PMOS and NMOS load curves.

#### **Lab Activity 1: VTC Simulation**
A DC sweep of the input voltage (Vin) from 0V to 1.8V was performed to generate the VTC and determine the switching threshold (Vm), the point where Vin = Vout.

**SPICE Netlist (DC Analysis):**
```spice
* VTC Simulation
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 1.8V

.dc Vin 0 1.8 0.01
```
#### **Lab Activity 2: Transient Analysis**
A pulse input was applied to Vin to measure the inverter's dynamic response, including rise/fall times and propagation delays (t_pLH and t_pHL).

**SPICE Netlist (Transient Analysis):**
```spice
* Transient Analysis
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

.tran 1n 10n
```
#### **Results and Analysis**
The VTC plot (Figure 6) shows a sharp transition, indicating high gain. The transient analysis (Figure 7) allows for precise measurement of timing characteristics. By adjusting the PMOS to NMOS width ratio (Wp/Wn), the rise and fall delays can be balanced. Due to the lower mobility of holes, a ratio of approximately Wp ≈ 2 * Wn is required for symmetrical delays. The impact of this sizing ratio on performance was systematically studied and is summarized in the table below.

![alt text](https://user-images.githubusercontent.com/89193562/132863366-e80c7b36-42af-45bb-a140-c75502008046.JPG)
Figure 6: The inverter's VTC, used to determine the switching threshold (Vm).	

![alt text](https://user-images.githubusercontent.com/89193562/132863939-9f777f44-e10e-4bc3-a9a2-41e833dd465b.JPG)
Figure 7: Input pulse (blue) and output response (red), used to measure propagation delays.

#### **Performance vs. Transistor Sizing:**
| (Wp/Lp) / (Wn/Ln) Ratio | Rise Delay | Fall Delay | Switching Threshold (Vm) |
| :----------------------: | :--------: | :--------: | :----------------------: |
| 1x | 148 ps | 71 ps | 0.99 V |
| 2x | 80 ps | 76 ps | 1.20 V |
| 3x | 57 ps | 80 ps | 1.25 V |
| 5x | 37 ps | 88 ps | 1.40 V |

### 4.3. Experiment 3: Inverter Robustness Evaluation

### **Objective**
To evaluate the robustness of the CMOS inverter design against variations in noise, power supply voltage, and manufacturing processes.

#### **A. Noise Margin Analysis**
**Concept:** Noise margins (NMh and NMl) quantify a gate's ability to tolerate noise on its input. They are determined from the VTC by finding the points where the gain |dVout/dVin| = 1.

* Noise Margin High (NMh) = Voh - Vih
* Noise Margin Low (NMl) = Vil - Vol

**Results:** The analysis showed that increasing the Wp/Wn ratio generally improved NMh at the slight expense of NMl, demonstrating a fundamental design trade-off between pull-up and pull-down network strengths.

![alt text](https://user-images.githubusercontent.com/89193562/132941130-48c2c93d-3ad2-4a9c-acd1-c7caec2bf87c.JPG)
Figure 8: The points where gain = -1 (Vih, Vil) are used to determine noise margins.

#### **B. Power Supply Variation Analysis**
**Concept:** Modern circuits operate at scaled supply voltages to save power. This analysis explores the impact of Vdd scaling on inverter performance using a scripted "smart simulation."

**SPICE Snippet (with loop):**
```spice
.control
let powersupply = 1.8
dowhile voltagesupplyvariation < 6
    dc Vin 0 1.8 0.01
    let powersupply = powersupply - 0.2
    alter Vdd = powersupply
    let voltagesupplyvariation = voltagesupplyvariation + 1
end
.endc
```
**Results:** The simulation (Figure 9) shows that lowering Vdd significantly improves gain but degrades switching speed due to reduced gate overdrive. While normalized noise margins improve, the absolute noise immunity decreases, making the circuit more susceptible to noise.

![alt text](https://user-images.githubusercontent.com/89193562/132985866-6915c943-8cf2-4c48-a524-c4b4c1169147.JPG)
Figure 9: Superimposed VTCs for supply voltages ranging from 1.8V (rightmost) to 0.8V (leftmost).

#### **C. Device Variation Analysis**
**Concept:** Manufacturing imperfections lead to variations in transistor characteristics. This was simulated by comparing a "Strong PMOS / Weak NMOS" configuration against its opposite.

**Results:** As shown in Figure 10, these process variations cause a significant shift in the switching threshold (Vm). However, the fundamental inverting behavior remains intact, demonstrating the inherent robustness of the CMOS topology.

![alt text](https://user-images.githubusercontent.com/89193562/132985983-6313769f-5ef4-432d-87c9-ef39f7e572f0.JPG)
Figure 10: A strong PMOS relative to the NMOS shifts the switching threshold to the right.

## 5. Key Learnings and Skill Development
This workshop provided a deep, practical understanding of the custom digital circuit design flow. The key skills developed are:

1. **Semiconductor Device Physics:** Gained a practical understanding of MOSFET I-V characteristics, distinguishing between long and short-channel device behavior, and analyzing the impact of **velocity saturation** and the body effect.
2. **Digital Circuit Design & Optimization:** Proficient in designing and sizing CMOS inverters by manipulating W/L ratios to meet specific performance targets, such as achieving a desired **switching threshold (Vm)**, balancing propagation delays, and optimizing noise margins.
3. **Advanced SPICE Simulation:** Mastered the creation of SPICE decks from scratch, writing netlists, and executing various analyses including .dc, .tran, and scripted parametric sweeps ("smart simulations") to efficiently characterize circuits across multiple conditions.
4. **Robustness and Variation Analysis:** Learned to evaluate circuit robustness by simulating the effects of **Process and Voltage (PV) variations**. This includes analyzing performance shifts due to power supply scaling and inherent manufacturing variations (e.g., etching, oxide thickness).
5. **System-Level Application Context:** Developed an understanding of how fundamental cell characteristics (like delay and slew rate) directly impact the timing of larger digital systems, establishing a crucial link between circuit design and **Static Timing Analysis (STA)**.

## 6. Conclusion
This project successfully bridged the gap between theoretical device physics and practical, system-aware circuit design. The journey from characterizing a single NMOS transistor to performing a comprehensive robustness evaluation of a CMOS inverter has solidified a deep understanding of the entire design and analysis flow. The ability to not only simulate but also analytically model circuit behavior and understand its downstream impact on system timing represents a critical milestone.

The skills in analytical modeling, advanced SPICE simulation, and performance optimization acquired through this workshop form a robust foundation for tackling more complex challenges in custom digital and analog IC design.

## 7. Acknowledgements
My sincere gratitude to [VLSI System Design]( https://www.vlsisystemdesign.com/) for conducting this in-depth and meticulously structured workshop. The clear progression from fundamental device physics to critical system-level considerations like STA was exceptionally valuable. The hands-on labs and constant support were instrumental in translating complex concepts into practical, applicable skills.
