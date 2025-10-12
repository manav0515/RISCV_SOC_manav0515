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

## 🧩 Introduction

In modern VLSI design flows, **Gate-Level Simulation (GLS)** is a crucial verification step that ensures the synthesized netlist behaves identically to the original RTL design.  
While **RTL simulations** are logic-based and idealized, **GLS** takes into account the actual logic gates, standard cells, and interconnect delays introduced during synthesis.

This repository presents the **Post-Synthesis GLS implementation of BabySoC**, a miniature System-on-Chip (SoC) containing a RISC-V processor, PLL, and DAC modules.  
The objective is to confirm that synthesis preserves both **logical correctness** and **timing consistency**.

---

## 🎯 Purpose of GLS

| Aspect | Description |
|--------|--------------|
| **Functional Verification** | Ensures the synthesized netlist maintains the same logical behavior as the RTL. |
| **Timing Verification** | Evaluates gate-level delays, setup/hold timing, and clock skews. |
| **Real-World Simulation** | Reflects the true hardware implementation using standard cell delays. |
| **Confidence Before PnR** | Guarantees that the design is ready for backend implementation and sign-off. |

In simpler terms, **GLS = “Reality Check” for your RTL design.**

---

## ⚙️ About BabySoC Design

**BabySoC** is a compact SoC consisting of:
- **RISC-V Core** (`rvmyth.v`) – A 5-stage pipelined processor.
- **PLL** (`avsdpll.v`) – Generates required clock frequencies.
- **DAC** (`avsddac.v`) – Converts digital signals to analog form.
- **Clock Gating & Reset Logic** – For low-power and stable operation.

Each module interacts through well-defined interfaces, and the top-level integration is handled in `vsdbabysoc.v`.

---

## 🏗️ Post-Synthesis Workflow

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

## 🧪 Steps to Run Post-Synthesis Simulation

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
Screenshot: -

### Inside the Yosys shell, run:
```
yosys
read_verilog /home/manav/VSDBabySoC/src/module/vsdbabysoc.v
read_verilog -I /home/manav/VSDBabySoC/src/include /home/manav/src/module/rvmyth.v
read_verilog -I /home/manav/VSDBabySoC/src/include /home/manav/src/module/clk_gate.v

```
![WhatsApp Image 2024-11-16 at 5 54 12 AM](https://github.com/user-attachments/assets/648dc511-7c3c-496a-97c7-a24aa6cb0bae)

![WhatsApp Image 2024-11-16 at 5 20 29 AM (2)](https://github.com/user-attachments/assets/6db87310-6389-4f7c-9418-40e4f6780c18)

![WhatsApp Image 2024-11-16 at 5 20 29 AM (1)](https://github.com/user-attachments/assets/8eddf6c8-c3fb-44d9-b804-5eb836558c44)

---
### **Step 2: Load the Liberty Files for Synthesis**
Inside the same Yosys shell, run: -
```
yosys
read_liberty -lib /home/manav/VSDBabySoC/src/lib/avsdpll.lib
read_liberty -lib /home/manav/VSDBabySoC/src/lib/avsddac.lib
read_liberty -lib /home/manav/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
![WhatsApp Image 2024-11-16 at 5 20 29 AM](https://github.com/user-attachments/assets/2ec505bd-8004-415f-ba9c-3b76a41562f8)

---

### **Step 3: Run Synthesis Targeting `vsdbabysoc`**
```yosys
synth -top vsdbabysoc
```
![WhatsApp Image 2024-11-16 at 5 20 28 AM](https://github.com/user-attachments/assets/8a49050d-55cb-4ae2-9a93-5fe7c2c72710)
![WhatsApp Image 2024-11-16 at 5 20 26 AM](https://github.com/user-attachments/assets/f00545e7-bb37-4444-80e7-0881938fb634)
![WhatsApp Image 2024-11-16 at 5 20 24 AM (2)](https://github.com/user-attachments/assets/655dfaaf-bece-47dc-8a24-bf257e064a4f)
![WhatsApp Image 2024-11-16 at 5 20 24 AM (1)](https://github.com/user-attachments/assets/5d7a9d12-7722-432c-8ad6-270be51b1df9)
![WhatsApp Image 2024-11-16 at 5 20 24 AM](https://github.com/user-attachments/assets/51f25b92-c968-4cf3-b553-21ecdbefc828)
![WhatsApp Image 2024-11-16 at 5 20 23 AM (8)](https://github.com/user-attachments/assets/241a089c-ce62-4f2c-8c6b-9e76d3929197)

---

### **Step 4: Map D Flip-Flops to Standard Cells**
```yosys
dfflibmap -liberty /home/manav/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```
![WhatsApp Image 2024-11-16 at 5 20 23 AM (7)](https://github.com/user-attachments/assets/566b121d-a5da-47c2-a09b-1660592569c5)

---

### **Step 5: Perform Optimization and Technology Mapping**
```yosys
opt
abc -liberty /home/manav/VSDBabySoC/src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib -script +strash;scorr;ifraig;retime;{D};strash;dch,-f;map,-M,1,{D}
```
![WhatsApp Image 2024-11-16 at 5 20 23 AM (6)](https://github.com/user-attachments/assets/5657a167-e0e2-431a-882e-4a785b059b5d)
![WhatsApp Image 2024-11-16 at 5 20 23 AM (5)](https://github.com/user-attachments/assets/a0ab61ba-24dc-4b9b-83fa-eb5b78f79f40)

---

### **Step 6: Perform Final Clean-Up and Renaming**
```yosys
flatten
setundef -zero
clean -purge
rename -enumerate
```
![WhatsApp Image 2024-11-16 at 5 20 23 AM (4)](https://github.com/user-attachments/assets/e2fd7bc4-5e8a-4236-84dc-002887f3eb82)

---

### **Step 7: Check Statistics**
```yosys
stat
```
![WhatsApp Image 2024-11-16 at 5 20 23 AM (3)](https://github.com/user-attachments/assets/292c9093-9a6d-417e-b094-0b8a6e27e7c3)
![WhatsApp Image 2024-11-16 at 5 20 23 AM (2)](https://github.com/user-attachments/assets/ce8ad45b-92ae-4cc8-a4dd-0f52028e078e)
![WhatsApp Image 2024-11-16 at 5 20 23 AM (1)](https://github.com/user-attachments/assets/e1741767-2b83-4d88-909e-e5d4c73411f4)

---

### **Step 8: Write the Synthesized Netlist**
```yosys
write_verilog -noattr /home/manav/VSDBabySoC/output/post_synth_sim/vsdbabysoc.synth.v
```
![WhatsApp Image 2024-11-16 at 5 20 23 AM](https://github.com/user-attachments/assets/1e0444b4-ad66-4798-b7f7-7bc1e13cf88a)

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

---


