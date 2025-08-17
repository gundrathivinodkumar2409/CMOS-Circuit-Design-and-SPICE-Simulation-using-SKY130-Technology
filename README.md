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

![Figure 1: NMOS Construction](https://user-images.githubusercontent.com/89193562/132697108-70c1704e-7389-4d04-84c3-189d945e04d5.jpg)
*Figure 1: Cross-section of an NMOS transistor.*

For modern, short-channel devices (<250nm), the classical quadratic current-voltage relationship in saturation is modified by **velocity saturation**, where carrier velocity no longer increases linearly with the electric field. This causes the drain current to become linearly dependent on `Vgs` and saturate earlier, impacting device performance.

![Figure 2: Velocity Saturation Effect](https://user-images.githubusercontent.com/89193562/132679374-baa32830-fcca-49c3-be54-10b5caf2c5d3.png)
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
