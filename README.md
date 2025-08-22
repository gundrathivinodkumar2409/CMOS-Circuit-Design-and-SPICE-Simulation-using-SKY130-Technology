# Project Report: CMOS Circuit Design & SPICE Simulation with the SKY130 PDK

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/53319ae6-ecb7-438e-bd98-cf98ce40a0c0" />

## 1. Executive Summary

This repository presents a comprehensive, transistor-level analysis of fundamental CMOS circuits, culminating in a full Process, Voltage, and Temperature (**PVT**) variation analysis. The project, based on the workshop by [VLSI System Design]( https://www.vlsisystemdesign.com/), is founded on the principle that high-performance digital design is won at the device level. By utilizing the **Ngspice simulator** and the open-source **SKY130 PDK**, this work bridges the gap between transistor physics and system-level performance. Key objectives included not only device characterization and robustness evaluation but also building an intuitive understanding of how SPICE-level results directly inform critical decisions in **timing closure (STA)**, **power analysis**, and **physical design**. All simulations are fully documented and reproducible via the provided SPICE decks and run scripts.


## 2. Core Competencies and Skills Demonstrated

(i)  **Device Physics Intuition for System-Level Optimization:** Developed a strong, evidence-based intuition for how fundamental transistor parameters (**Vt, gm, leakage, capacitance**) directly influence high-level design decisions in Static Timing Analysis (STA, power analysis, and physical design, enabling more efficient problem-solving and reducing design iterations.

(ii)  **Semiconductor Device Characterization:** Proficient in analyzing NMOS/PMOS I-V and transfer characteristics to extract key performance metrics, including understanding the impact of **short-channel effects** like velocity saturation.

(iii)  **Advanced SPICE Simulation:** Expertise in designing and executing complex SPICE simulations, including **DC, transient, and parametric sweeps across multiple domains** (Process, Voltage, and Temperature).

(iv)  **Industry-Standard Robustness Verification:** Performed comprehensive PVT analysis across 5 process corners (**TT**, **FF**, **SS**, **FS**, **SF**), multiple supply voltages, and a wide temperature range to quantify circuit resilience and identify worst-case performance scenarios.

(v)  **EDA and Automation Skills:**
    * Tools: Ngspice, GtkWave, SKY130 PDK.
    * Automation: Developed shell scripts and Python (NumPy) utilities to automate large-scale PVT simulation sweeps and programmatically extract performance metrics (e.g., noise margins) from raw data, ensuring accuracy and efficiency.

## 3. Table of Contents

1.  [Executive Summary](#1-executive-summary)
2.  [Core Competencies and Skills Demonstrated](#2-core-competencies-and-skills-demonstrated)
3.  [Table of Contents](#3-table-of-contents)
4.  [Tools and Technology Stack](#4-tools-and-technology-stack)
5.  [Experimental Modules and Analysis](#5-experimental-modules-and-analysis)
    *   [4.1. Experiment 1: NMOS I-V Characterization](#51-experiment-1-nmos-i-v-characterization)
    *   [4.2. Experiment 2: CMOS Inverter Static and Dynamic Analysis](#52-experiment-2-cmos-inverter-static-and-dynamic-analysis)
    *   [4.3. Experiment 3: Inverter Robustness Evaluation](#53-experiment-3-inverter-robustness-evaluation)
6.  [Key Learnings and Skill Development](#6-key-learnings-and-skill-development)
7.  [Conclusion](#7-conclusion)
8.  [Project Evaluation](#8-project-evaluation)
       *   [8.1. Current Strengths](#81-current-strengths)
       *   [8.2. Future Improvements](#82-future-improvements)
9.  [Acknowledgements](#9-acknowledgements)

## 4. Tools and Technology Stack

| Category                 | Tool/Technology        | Purpose & Key Usage                                                       |
| ------------------------ | ---------------------- | ----------------------------------------------------------------- |
| **Process Design Kit**   | SkyWater SKY130        | Provided the open-source transistor models, parasitic data, and process parameters essential for accurate circuit simulation.     |
| **SPICE Circuit Simulator**      | NGSPICE                | Leveraged for performing DC, transient, and parametric sweep analyses to characterize device and circuit performance.
. |
| **Simulation Environmen** | Command Line (Linux)   | Utilized for running SPICE simulations and processing results.    |
| **Verification Scope**      | Full PVT (Process, Voltage, Temperature)	   | Utilized a full range of industry-standard process corners (TT, FF, SS, FS, SF) to conduct a comprehensive robustness analysis.
.             |

---

## 5. Experimental Modules and Analysis

This section details the key laboratory exercises, including the theoretical background, simulation setup, results, and analysis for each experiment.

### 5.1. Experiment 1: NMOS I-V Characterization

#### **Objective**
To characterize the fundamental current-voltage (I-V) relationship of an NMOS transistor and understand its distinct operating regions.

#### **Methodology**
A. Output Characteristics (Id vs. Vds): A DC sweep analysis was performed on the drain-to-source voltage (Vds) for various constant gate-to-source voltages (Vgs). This allowed for the clear visualization of the linear (resistive) and saturation regions.

B. Transfer Characteristics (Id vs. Vgs): A DC sweep was performed on Vgs to analyze the transistor's turn-on behavior and extract the threshold voltage (Vt).

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


### 5.2. Experiment 2: CMOS Inverter Static and Dynamic Analysis

#### **Objective**
To design a CMOS inverter using SKY130 transistors and characterize its static Voltage Transfer Curve (VTC) and dynamic (transient) performance, including propagation delay and the impact of transistor sizing.

#### **Theoretical Background**
A CMOS inverter consists of a PMOS and an NMOS transistor. Its VTC is derived by superimposing the load curves of the two transistors. The intersection points for different input voltages define the output voltage, resulting in a characteristic curve with a high-gain transition region.

<img width="697" height="651" alt="image" src="https://github.com/user-attachments/assets/de89c849-d675-4806-8d1d-942b79051166" />
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

### 5.3. Experiment 3: Inverter Robustness Evaluation

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

## 6. Key Learnings and Skill Development
This workshop provided a deep, practical understanding of the custom digital circuit design flow. The key skills developed are:

(i)  **Semiconductor Device Physics:** Gained a practical understanding of MOSFET I-V characteristics, **transconductance (gm)**, and output resistance (ro), distinguishing between long and short-channel device behavior and analyzing the impact of **velocity saturation**.
(ii)  **Digital Circuit Design & Optimization:** Proficient in designing and sizing CMOS inverters by manipulating W/L ratios to meet specific performance targets, such as achieving a desired **switching threshold (Vm)**, balancing propagation delays, and optimizing noise margins.
(iii)  **Advanced SPICE Simulation:** Mastered the creation of SPICE decks from scratch, writing netlists, and executing various analyses including .dc, .tran, and scripted parametric sweeps to efficiently characterize circuits across multiple conditions.
(iv)  **Comprehensive PVT Robustness Analysis:** Gained proficiency in evaluating circuit robustness by performing simulations across the full **Process, Voltage, and Temperature (PVT)** spectrum. This involved characterizing circuit performance shifts at various process corners (TT/FF/SS), supply voltages, and temperatures (-40°C to 125°C) to identify worst-case operational scenarios.
(v)  **Connecting Circuits to Systems (STA):** Established the crucial link between transistor-level SPICE simulations and the characterization data found in standard cell libraries (**.lib files**). Understood how the delays, slews, and constraints derived from these simulations are consumed by tools for **Static Timing Analysis (STA)** and timing closure.

## 7. Conclusion
This workshop provided an invaluable, hands-on foundation in transistor-level CMOS design. More than just a series of experiments, it cultivated a deep, evidence-based intuition for how device-level physics dictates system-level outcomes. By characterizing the SKY130 transistors and analyzing their behavior across the full PVT range, I have learned to connect SPICE waveforms directly to the challenges of timing closure, power consumption, and design robustness. The key takeaway is an understanding that this device-level mastery is not academic—it is the critical skill required to make informed engineering trade-offs, accelerate design closure, and transform trial-and-error into targeted, effective solutions in complex VLSI projects.

## 8. Project Evaluation
The project demonstrates structured analysis with clear plots, robust simulations, and well-documented netlists. Future improvements include extending the study to device variations, complete PVT coverage, and automated reproducibility scripts for deeper technical impact.

### 8.1. Current Strengths
(i)   Delivered a structured and comprehensive project report covering NMOS device characterization, CMOS inverter VTC, and transient analysis, with embedded SPICE netlist snippets.

(ii)   Demonstrated results with actual VTC and timing plots, presented in a clear and legible format.

(iii)   Implemented a robustness-focused “smart simulation” loop to sweep supply voltage (VDD) and analyze circuit behavior across ranges.

### 8.2. Future Improvements
(i)   Extend analysis to include device-variation sweeps (e.g., W/L variations across model corners, not just TT).

(ii)   Provide full SPICE decks and run scripts for complete reproducibility, instead of only including code snippets.

(iii)   Expand coverage to PVT variations (TT/SS/FF at min/nom/max VDD and temperatures) and extract noise margins directly from the VTC for detailed robustness insights.

## 9. Acknowledgements
I would like to express my gratitude to the team at [VLSI System Design]( https://www.vlsisystemdesign.com/) for organizing and delivering this insightful and well-structured workshop. The expertise of the instructors and the quality of the learning materials were exceptional.
