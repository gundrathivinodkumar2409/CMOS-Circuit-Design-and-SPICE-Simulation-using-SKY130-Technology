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

| Category             | Tool/Technology        | Description                                                       |
| -------------------- | ---------------------- | ----------------------------------------------------------------- |
| **Process Design Kit** | SkyWater SKY130        | An open-source 130nm CMOS process for design and fabrication.     |
| **SPICE Simulator**    | NGSPICE                | Open-source mixed-signal circuit simulator for performance analysis. |
| **Simulation Environment**| Command Line (Linux) | Utilized for running SPICE simulations and processing results.    |
| **Process Corners**    | TT (Typical-Typical)   | Primary process corner used for baseline simulations.             |

---

## 4. Experiments and Analysis

This section details the key laboratory exercises, including the simulation setup, results, and analysis for each experiment.

### 4.1. Experiment 1: NMOS I-V Characterization

#### **Objective**
To simulate and analyze the current-voltage (I-V) characteristics of a SKY130 NMOS transistor, observing its behavior in different regions of operation and the impact of short-channel effects like velocity saturation.

#### **Simulation Setup**
An NMOS transistor (`sky130_fd_pr__nfet_01v8`) was simulated in NGSPICE. A DC sweep of the drain-to-source voltage (`Vds`) from 0V to 1.8V was performed for various gate-to-source voltages (`Vgs`).

**SPICE Netlist Snippet:**
```spice
* NMOS I-V Curve Simulation
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

XM1 Vdd n1 0 0 sky130_fd_pr__nfet_01v8 w=0.39 l=0.15
Vdd vdd 0 1.8V
Vin in 0 1.8V

.dc Vdd 0 1.8 0.1 Vin 0 1.8 0.2
```
#### **Results and Analysis**
The resulting Id vs. Vds plot clearly shows the linear (resistive) and saturation regions of the NMOS transistor. For this short-channel device (L=0.15µm), the saturation current does not exhibit a quadratic dependence on Vgs, a deviation from the classical long-channel model. This is primarily due to velocity saturation, where carrier velocity ceases to increase linearly with the electric field, limiting the drain current.

The plot of Id vs. Vgs was also generated to graphically estimate the threshold voltage (Vt), which was determined by extrapolating the linear portion of the curve to the x-axis.

![alt text](https://user-images.githubusercontent.com/89193562/132675399-e8f69dc7-f222-4e91-81fc-4cb2639213d4.JPG)
Figure 1: Id vs. Vds characteristics, demonstrating the onset of velocity saturation.

![alt text](https://user-images.githubusercontent.com/89193562/132675655-f779b9be-bcee-4d31-8a62-6204bc0bca40.JPG)
Figure 2: Id vs. Vgs curve used for the graphical extraction of the threshold voltage (Vt).

### 4.2. Experiment 2: CMOS Inverter Static and Dynamic Analysis

#### **Objective**
To design a CMOS inverter using SKY130 transistors and characterize its static Voltage Transfer Curve (VTC) and dynamic (transient) performance, including propagation delay.

#### **Simulation Setup**
A CMOS inverter was constructed using a PMOS and an NMOS transistor. The analysis involved two parts:
1. DC Analysis: Vin was swept from 0V to 1.8V to obtain the VTC.
2. Transient Analysis: A pulse input was applied to Vin to measure rise time, fall time, and propagation delays.

#### **SPICE Netlist Snippet (Transient Analysis):**
```spice
* CMOS Inverter Transient Analysis
.lib "sky130_fd_pr/models/sky130.lib.spice" tt

XM1 out in vdd vdd sky130_fd_pr__pfet_01v8 w=0.84 l=0.15
XM2 out in 0 0 sky130_fd_pr__nfet_01v8 w=0.36 l=0.15
Cload out 0 50fF
Vdd vdd 0 1.8V
Vin in 0 PULSE(0V 1.8V 0 0.1ns 0.1ns 2ns 4ns)

.tran 1n 10n
```
#### **Results and Analysis**
The VTC plot (Figure 3) shows a sharp transition region, indicative of high gain, a desirable characteristic for digital logic. The switching threshold (Vm), where Vin = Vout, was measured from this plot.

The transient analysis (Figure 4) provided key timing metrics. By adjusting the PMOS to NMOS width ratio (Wp/Wn), the rise and fall delays could be balanced. A ratio of approximately Wp ≈ 2 * Wn was found to provide roughly symmetrical delays, compensating for the lower mobility of holes in the PMOS device.

![alt text](https://user-images.githubusercontent.com/89193562/132863366-e80c7b36-42af-45bb-a140-c75502008046.JPG)
Figure 3: The inverter's VTC, used to determine the switching threshold (Vm).	

![alt text](https://user-images.githubusercontent.com/89193562/132863939-9f777f44-e10e-4bc3-a9a2-41e833dd465b.JPG)
Figure 4: Input pulse (blue) and output response (red), used to measure propagation delays.

The effect of varying the Wp/Wn ratio on performance was systematically studied:
| (Wp/Lp) / (Wn/Ln) Ratio | Rise Delay | Fall Delay | Switching Threshold (Vm) |
| :----------------------: | :--------: | :--------: | :----------------------: |
| 1x | 148 ps | 71 ps | 0.99 V |
| 2x | 80 ps | 76 ps | 1.20 V |
| 3x | 57 ps | 80 ps | 1.25 V |
| 5x | 37 ps | 88 ps | 1.40 V |

### 4.3. Experiment 3: Inverter Robustness Evaluation

### **Objective**
To evaluate the robustness of the CMOS inverter design against variations in noise, power supply voltage, and manufacturing (device) variations.

#### **A. Noise Margin Analysis**
The Noise Margin High (NMh = Voh - Vih) and Noise Margin Low (NMl = Vil - Vol) were calculated from the VTC plot by identifying the points where the gain (dVout/dVin) equals -1. A larger noise margin indicates greater immunity to signal noise. The analysis showed that increasing the Wp/Wn ratio generally improved NMh at the slight expense of NMl, demonstrating a fundamental design trade-off.

![alt text](https://user-images.githubusercontent.com/89193562/132941130-48c2c93d-3ad2-4a9c-acd1-c7caec2bf87c.JPG)
Figure 5: The points where gain = -1 (Vih, Vil) are used to determine noise margins.

#### **B. Power Supply Variation Analysis**
The inverter's VTC was simulated at different supply voltages (Vdd), from 1.8V down to 0.8V. The results (Figure 6) show that while lower supply voltages drastically reduce power consumption, they also degrade performance by reducing the gate overdrive, leading to slower switching speeds. Furthermore, the normalized noise margins tend to improve at lower voltages, but the absolute noise immunity decreases, making the circuit more susceptible to noise.

![alt text](https://user-images.githubusercontent.com/89193562/132985866-6915c943-8cf2-4c48-a524-c4b4c1169147.JPG)
Figure 6: Superimposed VTCs for supply voltages ranging from 1.8V (rightmost) to 0.8V (leftmost).

##### **C. Device Variation Analysis**
Manufacturing processes are imperfect, leading to variations in transistor characteristics. This was simulated by comparing a "Strong PMOS / Weak NMOS" configuration against a "Weak PMOS / Strong NMOS" configuration (achieved by varying W/L ratios). As shown in Figure 7, these process variations cause a significant shift in the switching threshold (Vm). However, the fundamental inverting behavior remains intact, demonstrating the inherent robustness of the CMOS topology.

![alt text](https://user-images.githubusercontent.com/89193562/132985983-6313769f-5ef4-432d-87c9-ef39f7e572f0.JPG)
Figure 7: A strong PMOS relative to the NMOS shifts the switching threshold to the right.

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
