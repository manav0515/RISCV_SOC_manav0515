# 🧠 BabySoC – Gate-Level Simulation (Post-Synthesis Verification)

## 📑 Table of Contents
1. [Introduction](#introduction)
2. [Purpose of GLS](#purpose-of-gls)
3. [About BabySoC Design](#about-babysoc-design)
4. [Post-Synthesis Workflow](#post-synthesis-workflow)
5. [Steps to Run Post-Synthesis Simulation](#steps-to-run-post-synthesis-simulation)
6. [Waveform Analysis](#waveform-analysis)
7. [Deliverables](#deliverables)
8. [Conclusion](#conclusion)

---

## Introduction

In modern VLSI design flows, **Gate-Level Simulation (GLS)** is a crucial verification step that ensures the synthesized netlist behaves identically to the original RTL design.  
While **RTL simulations** are logic-based and idealized, **GLS** takes into account the actual logic gates, standard cells, and interconnect delays introduced during synthesis.

This repository presents the **Post-Synthesis GLS implementation of BabySoC**, a miniature System-on-Chip (SoC) containing a RISC-V processor, PLL, and DAC modules.  
The objective is to confirm that synthesis preserves both **logical correctness** and **timing consistency**.

---

## Purpose of GLS

| Aspect | Description |
|--------|--------------|
| **Functional Verification** | Ensures the synthesized netlist maintains the same logical behavior as the RTL. |
| **Timing Verification** | Evaluates gate-level delays, setup/hold timing, and clock skews. |
| **Real-World Simulation** | Reflects the true hardware implementation using standard cell delays. |
| **Confidence Before PnR** | Guarantees that the design is ready for backend implementation and sign-off. |

In simpler terms, **GLS = “Reality Check” for your RTL design.**

---

## About BabySoC Design

**BabySoC** is a compact SoC consisting of:
- **RISC-V Core** (`rvmyth.v`) – A 5-stage pipelined processor.
- **PLL** (`avsdpll.v`) – Generates required clock frequencies.
- **DAC** (`avsddac.v`) – Converts digital signals to analog form.
- **Clock Gating & Reset Logic** – For low-power and stable operation.

Each module interacts through well-defined interfaces, and the top-level integration is handled in `vsdbabysoc.v`.

---

## Post-Synthesis Workflow

### 1. **Synthesis (Logic Mapping)**
- Tool used: **Yosys**
- RTL converted into gate-level netlist using the Sky130 standard cell library.
- Output: `vsdbabysoc.synth.v`
- Log captured in `output/synthesis_log.txt`.

### 2. **GLS Setup**
- The synthesized design is simulated with gate-level models.
- Include `sky130_fd_sc_hd.v` and `primitives.v` for correct standard cell functionality.

### 3. **Comparison**
- The output waveforms from RTL (functional) simulation and post-synthesis GLS are compared.
- Expected: *Identical logic behavior, with timing shifts due to gate delays.*

---

## Steps to Run Post-Synthesis Simulation

### 🧰 Prerequisites
Ensure you have:
- **Icarus Verilog (iverilog)**
- **GTKWave**
- **Yosys**

## step-by-step execution plan for running the commands manually: -
---
### **Step 1: Load the Top-Level Design and Supporting Modules**
```bash
yosys
```
<img width="1212" height="309" alt="Screenshot from 2025-10-05 16-57-25" src="https://github.com/user-attachments/assets/605e8f7c-f7da-4e10-b4d2-a33a4806b79e" />

### Inside the Yosys shell, run:
```
yosys
read_verilog /home/manav/VSDBabySoC/src/module/vsdbabysoc.v
read_verilog -I /home/manav/VSDBabySoC/src/include /home/manav/src/module/rvmyth.v
read_verilog -I /home/manav/VSDBabySoC/src/include /home/manav/src/module/clk_gate.v

```
<img width="1212" height="309" alt="Screenshot from 2025-10-05 17-01-55" src="https://github.com/user-attachments/assets/28c5b48d-a335-46e2-8b11-21855b2d576c" />


---
### **Step 2: Load the Liberty Files for Synthesis**
Inside the same Yosys shell, run: -
```
yosys
read_liberty -lib /home/manav/VSDBabySoC/src/lib/avsdpll.lib
read_liberty -lib /home/manav/VSDBabySoC/src/lib/avsddac.lib
read_liberty -lib /home/manav/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

```
<img width="1212" height="266" alt="Screenshot from 2025-10-05 17-03-57" src="https://github.com/user-attachments/assets/2c5a5f85-35a7-4f36-880e-ff3c9ed7dfd4" />

---

### **Step 3: Run Synthesis Targeting `vsdbabysoc`**
```yosys
synth -top vsdbabysoc
```
<img width="1212" height="772" alt="Screenshot from 2025-10-05 17-06-31" src="https://github.com/user-attachments/assets/96e7847d-4506-4e15-88e9-d172892e31ad" />
<img width="1208" height="257" alt="Screenshot from 2025-10-05 17-07-40" src="https://github.com/user-attachments/assets/59920857-a3c0-4092-95b9-e1f40609bb6b" />
<img width="1194" height="512" alt="Screenshot from 2025-10-12 16-15-45" src="https://github.com/user-attachments/assets/9050465e-69fc-49cb-967f-df0c60a71aa5" />
<img width="902" height="336" alt="Screenshot from 2025-10-05 17-08-24" src="https://github.com/user-attachments/assets/af1e66dc-e626-4e61-af65-3fdd1bfcc5e8" />
<img width="1209" height="731" alt="Screenshot from 2025-10-05 17-09-13" src="https://github.com/user-attachments/assets/8f74f4dc-97a3-4e37-b94b-2f935db318b3" />
<img width="1006" height="145" alt="Screenshot from 2025-10-05 17-09-48" src="https://github.com/user-attachments/assets/fcfad714-f4a0-4e3e-b683-236b6c697c19" />

---

### **Step 4: Map D Flip-Flops to Standard Cells**
```yosys
dfflibmap -liberty /home/manav/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
<img width="1212" height="774" alt="Screenshot from 2025-10-05 17-11-58" src="https://github.com/user-attachments/assets/53f6b7a2-40ac-4ea3-a06f-3d1aad8bfa73" />
<img width="1212" height="774" alt="Screenshot from 2025-10-05 17-14-43" src="https://github.com/user-attachments/assets/b9c8067c-cd0d-4019-83af-d51dabd0a897" />
<img width="1194" height="512" alt="Screenshot from 2025-10-12 16-18-00" src="https://github.com/user-attachments/assets/e89b2a8d-d996-4eb0-bfd9-86346c65e1a9" />

---

### **Step 5: Perform Optimization and Technology Mapping**
```yosys
opt
abc -liberty /home/manav/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```

---

### **Step 6: Perform Final Clean-Up and Renaming**
```yosys
flatten
setundef -zero
clean -purge
rename -enumerate
```
<img width="1212" height="262" alt="Screenshot from 2025-10-05 17-15-52" src="https://github.com/user-attachments/assets/c6b568d5-8e02-4bee-8d83-b412213781d8" />


---


### **Step 7: Check Statistics**
```yosys
stat
```
<img width="1111" height="1144" alt="image" src="https://github.com/user-attachments/assets/57d30eee-f6e5-44f2-8f7a-50cee29009a3" />


---

### **Step 8: Write the Synthesized Netlist**
```yosys
write_verilog -noattr /home/manav/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v
```
<img width="1212" height="184" alt="Screenshot from 2025-10-05 17-17-25" src="https://github.com/user-attachments/assets/78534a5f-159c-4734-aa72-64c5759d5656" />

---
### **Outputs Of Yosys(Synthesis) Are In The Following Log File**
- [synthesis_log.txt](./output/synthesis_log.txt)
---

## POST_SYNTHESIS SIMULATION AND WAVEFORMS
---

### **Step 1: Compile the Testbench**
Run the following `iverilog` command to compile the testbench:
```bash
iverilog -o /home/manav/VSDBabySoC/output/post_synth_sim/post_synth_sim.out \
-DPOST_SYNTH_SIM -DFUNCTIONAL -DUNIT_DELAY=#1 \
-I /home/manav/VSDBabySoC/src/include \
-I /home/manav/VSDBabySoC/src/module \
-I /home/manav/VSDBabySoC/src/gls_model \
/home/manav/VSDBabySoC/src/module/testbench.v
```
---
### **Step 2: Navigate to the Post-Synthesis Simulation Output Directory**
```bash
cd output/post_synth_sim/
```
---
### **Step 3: Run the Simulation**

```bash
./post_synth_sim.out
```
---
### **Step 4: View the Waveforms in GTKWave**

```bash
gtkwave post_synth_sim.vcd
```

<img width="1210" height="769" alt="Output_1" src="https://github.com/user-attachments/assets/6beec547-15ca-4e2e-8ea3-2bcd01c17e1a" />

<img width="1210" height="769" alt="Output_2" src="https://github.com/user-attachments/assets/e3cf2f36-a026-42bb-ac55-5f6ce8b033a6" />

---

# 🧠 BabySoC – Gate-Level Simulation (GLS) Verification Report

---

## Waveform Analysis

| **Observation** | **Functional Simulation (Pre-Synthesis)** | **Gate-Level Simulation (Post-Synthesis)** |
|------------------|-------------------------------------------|---------------------------------------------|
| **Reset & Control Signals** | All control signals (`reset`, `EN`, etc.) transition cleanly with ideal timing. | Same transitions observed with **minor gate-level propagation delay**. |
| **Clock Behavior (PLL Output)** | Stable and periodic clock waveform from PLL. Ideal clock edges visible. | Clock frequency and duty cycle match RTL, but **edges slightly shifted** due to synthesis delays. |
| **Core / Pipeline Activity** | DAC input data (`D[9:0]`, `Dext[10:0]`) transitions correctly, showing proper pipeline functionality. | Same logical behavior observed; **minor skew** introduced by gate-level net delays. |
| **DAC Output (`OUT`)** | Smooth analog-like waveform formed by the DAC, representing correct digital-to-analog conversion. | Identical DAC waveform shape, **slightly delayed and smoothed** due to realistic timing effects. |
| **VREFH / VREFL Behavior** | Reference voltages steady and clean throughout simulation. | Same reference stability, confirming proper power and bias modeling. |

---

## Deliverables

| **Deliverable** | **File / Folder** | **Description** |
|-----------------|------------------|------------------|
| **Synthesis Log** | [synthesis_log.txt](./output/synthesis_log.txt) | Log of synthesis run from Yosys. |
| **GLS Waveform Screenshots** | [screenshots](screenshots) | GTKWave captures of gate-level simulation. |
| **GLS Verification Note** | [NOTES.md](NOTES.md) | Confirmation that GLS results match functional outputs. |

---

## Conclusion

The **Gate-Level Simulation (GLS)** for **BabySoC** confirms that:

- Post-synthesis behavior **matches the RTL functional simulation**.  
- Control, pipeline, and DAC output waveforms remain **functionally identical**.  
- Only expected **timing and propagation delays** are observed.  
- The synthesized design **retains full logical correctness**.

---

✅ **Final Verdict:**  
**GLS = Functional Outputs**  
> BabySoC design successfully verified at gate-level.

