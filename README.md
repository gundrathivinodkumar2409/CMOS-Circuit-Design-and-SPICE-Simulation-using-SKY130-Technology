# Project Report: CMOS Circuit Design & SPICE Simulation with the SKY130 PDK

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/53319ae6-ecb7-438e-bd98-cf98ce40a0c0" />


## 1. Executive Summary

This report documents the practical application of CMOS circuit design principles and SPICE simulation techniques learned during the "CMOS Circuit Design and SPICE Simulation using SKY130 Technology" workshop by VLSI System Design. The project involved the end-to-end characterization of fundamental semiconductor devices and digital logic circuits using the open-source **SkyWater 130nm (SKY130) Process Design Kit (PDK)**.

Key activities included characterizing NMOS I-V curves, analyzing velocity saturation effects in short-channel devices, and performing comprehensive static and dynamic analyses of a CMOS inverter. The project culminated in a robustness evaluation of the inverter, examining its performance under variations in transistor sizing, power supply, and process corners. All simulations were conducted using **NGSPICE**, providing hands-on experience with industry-standard circuit analysis methodologies.

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
### 4.2. Experiment 2: CMOS Inverter Static and Dynamic Analysis

#### **Objective**
To design a CMOS inverter using SKY130 transistors and characterize its static Voltage Transfer Curve (VTC) and dynamic (transient) performance, including propagation delay.

#### **Simulation Setup**
A CMOS inverter was constructed using a PMOS and an NMOS transistor. The analysis involved two parts:
1. DC Analysis: Vin was swept from 0V to 1.8V to obtain the VTC.
2. Transient Analysis: A pulse input was applied to Vin to measure rise time, fall time, and propagation delays.
   
**SPICE Netlist Snippet:**
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

### 4.3. Experiment 3: Inverter Robustness Evaluation

#### **Objective**
To evaluate the robustness of the CMOS inverter design against variations in noise, power supply voltage, and manufacturing (device) variations.

#### **A. Noise Margin Analysis**
The Noise Margin High (NMh = Voh - Vih) and Noise Margin Low (NMl = Vil - Vol) were calculated from the VTC plot by identifying the points where the gain (dVout/dVin) equals -1. A larger noise margin indicates greater immunity to signal noise. The analysis showed that increasing the Wp/Wn ratio generally improved NMh at the slight expense of NMl, demonstrating a fundamental design trade-off.
![alt text](https://user-images.githubusercontent.com/89193562/132941130-48c2c93d-3ad2-4a9c-acd1-c7caec2bf87c.JPG)

Figure 5: The points where gain = -1 (Vih, Vil) are used to determine noise margins.

#### **B. Power Supply Variation Analysis**
The inverter's VTC was simulated at different supply voltages (Vdd), from 1.8V down to 0.8V. The results (Figure 6) show that while lower supply voltages drastically reduce power consumption, they also degrade performance by reducing the gate overdrive, leading to slower switching speeds. Furthermore, the normalized noise margins tend to improve at lower voltages, but the absolute noise immunity decreases, making the circuit more susceptible to noise.
![alt text](https://user-images.githubusercontent.com/89193562/132985866-6915c943-8cf2-4c48-a524-c4b4c1169147.JPG)

Figure 6: Superimposed VTCs for supply voltages ranging from 1.8V (rightmost) to 0.8V (leftmost).

#### **C. Device Variation Analysis**
Manufacturing processes are imperfect, leading to variations in transistor characteristics. This was simulated by comparing a "Strong PMOS / Weak NMOS" configuration against a "Weak PMOS / Strong NMOS" configuration (achieved by varying W/L ratios). As shown in Figure 7, these process variations cause a significant shift in the switching threshold (Vm). However, the fundamental inverting behavior remains intact, demonstrating the inherent robustness of the CMOS topology.
![alt text](https://user-images.githubusercontent.com/89193562/132985983-6313769f-5ef4-432d-87c9-ef39f7e572f0.JPG)

Figure 7: A strong PMOS relative to the NMOS shifts the switching threshold to the right.


## 5. Key Learnings and Skill Development
This workshop provided a comprehensive, hands-on understanding of the custom CMOS circuit design flow. Key skills developed include:

1. Semiconductor Device Physics: Gained a practical understanding of MOSFET I-V characteristics, regions of operation, and the impact of short-channel effects like velocity saturation.
2. Digital Circuit Design: Proficient in designing and sizing fundamental CMOS logic gates (inverters) to meet specific performance targets for delay and power.
3. SPICE Simulation and Analysis: Mastered the use of NGSPICE for writing netlists and performing DC, Transient, and Parametric Sweep simulations for thorough circuit characterization.
4. Performance and Robustness Analysis: Learned to evaluate critical design metrics such as switching threshold, propagation delay, noise margins, and the impact of process, voltage, and temperature (PVT) variations.
5. Design Trade-offs: Developed an appreciation for the fundamental trade-offs in circuit design, such as speed vs. power, performance vs. robustness, and area vs. performance.

## 6. Conclusion
Through a series of detailed SPICE simulations, this project successfully validated the theoretical principles of CMOS circuit design using the open-source SKY130 PDK. The analysis of an NMOS transistor and a CMOS inverter provided deep insights into their static and dynamic behaviors. The robustness evaluations further highlighted the resilience of CMOS technology and the critical design considerations required to build reliable digital systems.
The practical skills acquired in this workshop form a strong foundation for tackling more complex digital and analog IC design challenges.


## 7. Acknowledgements
I would like to express my gratitude to VLSI System Design (VSD) for organizing this insightful and practical workshop. The structured content and hands-on labs were instrumental in solidifying my understanding of CMOS circuit design.
