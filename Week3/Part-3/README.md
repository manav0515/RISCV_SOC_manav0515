# ⚙️ Part 3 — OpenSTA Timing Analysis Across PVT Corners

---

## 🧭 Table of Contents
1. [Introduction](#introduction)
2. [Theory and Motivation](#theory-and-motivation)
3. [Understanding PVT Corners](#understanding-pvt-corners)
4. [STA Workflow Summary](#sta-workflow-summary)
5. [Visualization & Reports](#visualization--reports)
6. [Deliverables](#deliverables)
7. [Reference](#reference)

---

## Introduction

This section focuses on **Static Timing Analysis (STA)** of the synthesized **RISC-V SoC design** using **OpenSTA**.  
The goal is to verify that the circuit meets **timing constraints** across multiple **PVT (Process, Voltage, Temperature)** corners, ensuring reliability and correctness before moving toward physical design or tape-out.

---

## Theory and Motivation

Imagine your circuit is a racetrack.  
Every signal is a racer, and the **clock** is the referee.  

The racers (signals) must reach their destination **just in time** — not too early (hold violation) and not too late (setup violation).  
Now, what happens when you change the weather (temperature), the fuel (voltage), or even the race conditions (process)?  
That’s exactly what **PVT variation** does.

To ensure your digital system never “crashes the race,” STA is performed at several corners:
- **Fast corner (FF)**: Best-case — faster transistors, high voltage, low temperature.  
- **Typical corner (TT)**: Nominal condition — moderate process and operating points.  
- **Slow corner (SS)**: Worst-case — slower transistors, low voltage, high temperature.  

These checks give key insights into:
- ⏱️ **WNS (Worst Negative Slack)** — the single most critical delay violation.  
- ⚠️ **TNS (Total Negative Slack)** — the sum of all failing path slacks.  
- 🧩 **Worst Setup & Hold Slack** — to locate timing-critical endpoints.  

In short: **If your design meets timing at all PVT corners — it’s ready for silicon.**

---

## Understanding PVT Corners

| Corner Type | Example Library File | Voltage (V) | Temp (°C) | Nature |
|--------------|---------------------|--------------|------------|--------|
| TT           | `sky130_fd_sc_hd__tt_025C_1v80.lib` | 1.80 | 25 | Nominal |
| FF           | `sky130_fd_sc_hd__ff_n40C_1v95.lib` | 1.95 | 100 | Fast (Hold-critical) |
| SS           | `sky130_fd_sc_hd__ss_n40C_1v35.lib` | 1.35 | -40 | Slow (Setup-critical) |

Each corner represents a unique behavior of your standard cell library.  
Running STA on all these ensures **robust timing closure** for silicon fabrication.

---

## STA Workflow Summary

1. **Load Liberty (.lib)** — Process-specific timing models.  
2. **Load Netlist (.v)** — Synthesized gate-level design.  
3. **Read SDC (.sdc)** — Clock and I/O constraints.  
4. **Run STA Checks** — Setup/Hold, WNS, TNS, and slack reports.  
5. **Export Reports** — `sta_timing_report.txt`, `.csv`, and graph visualizations.

The Tcl automation (`sta_across_pvt.tcl`) loops through multiple libraries to generate reports for each corner automatically.

---

## Visualization & Reports

After running STA across all corners, a Python script parses the reports and creates a **combined graph** showing:
- Worst Setup Slack  
- Worst Hold Slack  
- WNS  
- TNS  

This helps visualize how each PVT corner performs in one unified plot.  
The graph also includes **timestamp** and **user ID** for tracking execution identity.

---

## Deliverables

> 📌 **Each section below contains placeholders for input for STA, screenshots, and output files.**

---

## 1. TT Corner (1.80V)

### **Included Libraries: -**
```
"sky130_fd_sc_hd__tt_025C_1v80.lib"
```

- **Input File: -**
```
set list_of_lib_files(1) "sky130_fd_sc_hd__tt_025C_1v80.lib"

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
read_liberty ./timing_libs/$list_of_lib_files($i)
read_verilog ./riscv_pipelined_Final_netlist.v
link_design riscv_core
current_design
read_sdc riscv_core_synthesis.sdc
check_setup -verbose
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} > ./sta_output/min_max_$list_of_lib_files($i).txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_max_slack.txt
report_worst_slack -max -digits {4} >> ./sta_output/sta_worst_max_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_min_slack.txt
report_worst_slack -min -digits {4} >> ./sta_output/sta_worst_min_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_tns.txt
report_tns -digits {4} >> ./sta_output/sta_tns.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_wns.txt
report_wns -digits {4} >> ./sta_output/sta_wns.txt
}
```

**Link to the file: -** **[Input File](./Input%20Files/sta_across_pvt_sky130_fd_sc_hd__tt_025C_1v80.tcl.txt)**

- **Screenshot – STA Timing Report:**  
<img width="1196" height="1350" alt="sky130_fd_sc_hd__tt_025C_1v80___" src="https://github.com/user-attachments/assets/7573b389-eaf5-49cb-86a6-f6f4d6e9cb9a" />


- **Link to Report File:**  
***[sta_timing_report_tt.txt](./outputs/sta_timing_report_sky130_fd_sc_hd__tt_025C_1v80.txt)***

- **Screenshot – Graph Output:**  
<img width="1280" height="800" alt="sky130_fd_sc_hd__tt_025C_1v80_" src="https://github.com/user-attachments/assets/f6d2fee6-1916-4f32-b59b-b985fd21f752" />
<img width="947" height="605" alt="sky130_fd_sc_hd__tt_025C_1v80__" src="https://github.com/user-attachments/assets/4218a7ed-3066-4feb-b309-f16d9dd161e6" />


- **Link to Graph File:**  
***[tt_corner_graph.png](./outputs/sky130_fd_sc_hd__tt_025C_1v80__.png)***

- **Link to CSV File:**  
***[tt_corner_metrics.csv](outputs/metrics_sky130_fd_sc_hd__tt_025C_1v80.csv)***

---

## 2. FF Corner (1.95V)

### **Included Libraries: -**
```
"sky130_fd_sc_hd__ff_n40C_1v95.lib"
```

- ### **Input File: -**
```
set list_of_lib_files(1) "sky130_fd_sc_hd__ff_n40C_1v95"

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
read_liberty ./timing_libs/$list_of_lib_files($i)
read_verilog ./riscv_pipelined_Final_netlist.v
link_design riscv_core
current_design
read_sdc riscv_core_synthesis.sdc
check_setup -verbose
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} > ./sta_output/min_max_$list_of_lib_files($i).txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_max_slack.txt
report_worst_slack -max -digits {4} >> ./sta_output/sta_worst_max_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_min_slack.txt
report_worst_slack -min -digits {4} >> ./sta_output/sta_worst_min_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_tns.txt
report_tns -digits {4} >> ./sta_output/sta_tns.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_wns.txt
report_wns -digits {4} >> ./sta_output/sta_wns.txt
}  
```  
**Link to the file: -** ***[Input File](./Input%20Files/sta_across_pvt_sky130_fd_sc_hd__ff_n40C_1v95_.tcl.txt)***

- ### **Screenshot – STA Timing Report:**  
<img width="1197" height="1350" alt="sky130_fd_sc_hd__ff_n40C_1v95__" src="https://github.com/user-attachments/assets/feb3b515-f9c3-4eda-be20-23d5877d426f" />


- ### **Link to Report File:**  
***[sta_timing_report_ff.txt](./outputs/sta_timing_report_sky130_fd_sc_hd__ff_n40C_1v95.txt)***

- ### **Screenshot – Graph Output:**  
<img width="1280" height="800" alt="sky130_fd_sc_hd__ff_n40C_1v95_" src="https://github.com/user-attachments/assets/d3567958-73f5-47d0-aadb-d22f6925a549" />
<img width="1002" height="538" alt="sky130_fd_sc_hd__ff_n40C_1v95" src="https://github.com/user-attachments/assets/9cc2babe-7eda-4aee-b25b-31bdcc110a3f" />


- ### **Link to Graph File:**  
***[ff_corner_graph.png](./outputs/sky130_fd_sc_hd__ff_n40C_1v95.png)***

- ### **Link to CSV File:**  
***[ff_corner_metrics.csv](./outputs/metrics_sky130_fd_sc_hd__ff_n40C_1v95.csv)***

---

## 🔹 3. All Corners (Combined)

### **Included Libraries: -**
```
"sky130_fd_sc_hd__tt_025C_1v80.lib"
"sky130_fd_sc_hd__ff_100C_1v65.lib"
"sky130_fd_sc_hd__ff_100C_1v95.lib"
"sky130_fd_sc_hd__ff_n40C_1v56.lib"
"sky130_fd_sc_hd__ff_n40C_1v65.lib"
"sky130_fd_sc_hd__ff_n40C_1v76.lib"
"sky130_fd_sc_hd__ss_100C_1v40.lib"
"sky130_fd_sc_hd__ss_100C_1v60.lib"
"sky130_fd_sc_hd__ss_n40C_1v28.lib"
"sky130_fd_sc_hd__ss_n40C_1v35.lib"
"sky130_fd_sc_hd__ss_n40C_1v40.lib"
"sky130_fd_sc_hd__ss_n40C_1v44.lib"
"sky130_fd_sc_hd__ss_n40C_1v76.lib"
```

- ### **Input File: -**
```
set list_of_lib_files(1) "sky130_fd_sc_hd__tt_025C_1v80.lib"
set list_of_lib_files(2) "sky130_fd_sc_hd__ff_100C_1v65.lib"
set list_of_lib_files(3) "sky130_fd_sc_hd__ff_100C_1v95.lib"
set list_of_lib_files(4) "sky130_fd_sc_hd__ff_n40C_1v56.lib"
set list_of_lib_files(5) "sky130_fd_sc_hd__ff_n40C_1v65.lib"
set list_of_lib_files(6) "sky130_fd_sc_hd__ff_n40C_1v76.lib"
set list_of_lib_files(7) "sky130_fd_sc_hd__ss_100C_1v40.lib"
set list_of_lib_files(8) "sky130_fd_sc_hd__ss_100C_1v60.lib"
set list_of_lib_files(9) "sky130_fd_sc_hd__ss_n40C_1v28.lib"
set list_of_lib_files(10) "sky130_fd_sc_hd__ss_n40C_1v35.lib"
set list_of_lib_files(11) "sky130_fd_sc_hd__ss_n40C_1v40.lib"
set list_of_lib_files(12) "sky130_fd_sc_hd__ss_n40C_1v44.lib"
set list_of_lib_files(13) "sky130_fd_sc_hd__ss_n40C_1v76.lib"

for {set i 1} {$i <= [array size list_of_lib_files]} {incr i} {
read_liberty ./timing_libs/$list_of_lib_files($i)
read_verilog ./riscv_pipelined_Final_netlist.v
link_design riscv_core
current_design
read_sdc riscv_core_synthesis.sdc
check_setup -verbose
report_checks -path_delay min_max -fields {nets cap slew input_pins fanout} -digits {4} > ./sta_output/min_max_$list_of_lib_files($i).txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_max_slack.txt
report_worst_slack -max -digits {4} >> ./sta_output/sta_worst_max_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_worst_min_slack.txt
report_worst_slack -min -digits {4} >> ./sta_output/sta_worst_min_slack.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_tns.txt
report_tns -digits {4} >> ./sta_output/sta_tns.txt

exec echo "$list_of_lib_files($i)" >> ./sta_output/sta_wns.txt
report_wns -digits {4} >> ./sta_output/sta_wns.txt
}
``` 

**Link to the file: -** ***[Input File](./Input%20Files/sta_across_pvt.tcl)***

---
- ### **Output Timings Table: -** 
<img width="1337" height="484" alt="Screenshot (525)" src="https://github.com/user-attachments/assets/3b441265-cfd9-4b58-b5fa-4ba808b798bb" />

---

- ### **Combined Graphs: -**  

### **1) WorstSetupSlack**

<img width="1000" height="600" alt="WorstSetupSlack_ns_graph" src="https://github.com/user-attachments/assets/f3cbb0ec-a6bd-4e55-b92d-26af303cbf20" />

---
### **2) WorstHoldSlack**

<img width="1000" height="600" alt="WorstHoldSlack_ns_graph" src="https://github.com/user-attachments/assets/324e11c6-bf77-4cd0-b8dc-29765fd9cf82" />

---
### **3) WNS**

<img width="1000" height="600" alt="WNS_ns_graph" src="https://github.com/user-attachments/assets/f7a6c2ee-7f1e-42e2-b4be-07c54e7c7b1c" />


---
### **4) RNS**

<img width="1000" height="600" alt="TNS_ns_graph" src="https://github.com/user-attachments/assets/7b36032b-9556-4196-b96d-e30cc32dc65d" />

---



- ### **Link to CSV File:**  
***[all_library_metrics.csv](./sta_output/all_library_metrics.pdf)***

---

## Reference

### **Day 19 – PVT Corner Analysis (Post-Synthesis Timing) of the RISC-V CPU Design**

The STA checks are performed across all corners to confirm the design meets the target timing requirements.

> The worst max path (Setup-critical) corners in sub-40nm nodes are usually:  
> `ss_LowTemp_LowVolt`, `ss_HighTemp_LowVolt`  
>  
> The worst min path (Hold-critical) corners are typically:  
> `ff_LowTemp_HighVolt`, `ff_HighTemp_HighVolt`


