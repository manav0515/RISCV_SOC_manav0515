[![RISC-V](https://img.shields.io/badge/RISC--V-Digital%20Design-blue?style=for-the-badge&logo=riscv)](https://riscv.org/)
[![Workshop](https://img.shields.io/badge/STA-Workshop-orange?style=for-the-badge)](https://vsdiat.vlsisystemdesign.com/)
[![Week 3](https://img.shields.io/badge/Week%203-Part%202-green?style=for-the-badge)]()

# Week 3 Task – Static Timing Analysis (STA) Fundamentals

## 🎯 Objective
To explore and internalize the **core ideas behind Static Timing Analysis (STA)** — learning how setup and hold checks, timing paths, clocks, and constraints together ensure that a post-synthesis circuit runs safely at its target frequency.

STA may sound theoretical, but it’s the silent gatekeeper that decides whether silicon works on day 1 or fails under real conditions.

---

## 📚 Table of Contents
- [Introduction](#introduction)
- [Timing Paths](#timing-paths)
- [Types of Setup & Hold Analyses](#types-of-setup--hold-analyses)
- [Slew & Transition Analysis](#slew--transition-analysis)
- [Load Analysis](#load-analysis)
- [Clock Analysis](#clock-analysis)
- [Reg-to-Reg Path Timing Parameters](#reg-to-reg-path-timing-parameters)
- [On-Chip Variation (OCV)](#on-chip-variation)
- [OCV Timing and Pessimism Removal](#ocv-timing-and-pessimism-removal)
- [Tools Used](#tools-used)
- [Outcome](#outcome)

---

## 📖 Introduction
Imagine verifying a million-gate design without applying a single test vector.  
That’s **Static Timing Analysis**.

STA divides the entire design into **timing paths** and computes delays mathematically.  
It then checks whether every signal meets its setup and hold requirements — across every possible condition of **process, voltage, and temperature** — all **without simulation**.

This exhaustive coverage ensures reliability in the field and lets designers fix issues before tape-out.  
Think of it as a “time-based audit” of your digital design.

---

## ⏱️ Timing Paths
A **timing path** represents the journey of a signal through logic — from one flip-flop to another, or from an input pin to an output.  
Understanding them is essential; every delay, skew, and load along the way contributes to the circuit’s speed limit.

![Timing Path](https://user-images.githubusercontent.com/110079770/190552665-bf99cee6-aba9-40b2-a0a8-101de7b8d329.png)

### Example
![Example Timing Paths](https://user-images.githubusercontent.com/110079770/190560524-e101803d-cb67-4fc0-9532-a50b12e61c36.png)

A single chip contains thousands of such paths. STA ensures that even the *slowest* one meets setup time and the *fastest* one doesn’t break hold time.

---

## 🔍 Types of Setup & Hold Analyses
Every timing check falls into one of several path categories — each revealing a different interaction between data and clock.

![Types of Checks](https://user-images.githubusercontent.com/110079770/190888247-335fa2a4-0370-498c-a171-d0b9fdbfb558.png)

### 1. **Reg-to-Reg (Register to Register)**
A path that begins and ends at flip-flops.  
It defines the clock-to-clock timing that dominates synchronous performance.

![Reg-to-Reg](https://user-images.githubusercontent.com/62461290/190436869-395360d3-37b5-4490-9248-dcd04471b026.png)

**Think about it:** this is where setup and hold failures show up first.

Key Points  
- Data must remain stable around the clock edge.  
- **Setup violation:** data arrives too late.  
- **Hold violation:** data changes too soon.  
- Most common path in synchronous logic.

---

### 2. **In-to-Reg (Input to Register)**
From an input port to a register — critical for capturing external data.

![In-to-Reg](https://user-images.githubusercontent.com/62461290/190437220-79303d9c-c41b-4827-bef4-67d121b2abcf.png)

Key Points  
- Input delay is defined relative to clock.  
- Determines whether off-chip data can be sampled correctly.  
- Needs accurate board/interface timing info.

---

### 3. **Reg-to-Out (Register to Output)**
From a register to an output pin.  
Used to verify that internal data reaches the outside world on time.

![Reg-to-Out](https://user-images.githubusercontent.com/62461290/190437614-f1ee053b-1de1-49eb-9b53-bb2f3cc50eb9.png)

Key Points  
- Controlled by **output delay** constraints.  
- Must meet setup of the external device.  
- Includes clock-to-Q and logic delays.

---

### 4. **In-to-Out (Input to Output)**
Pure combinational path without registers.  
Common in glue logic and mux structures.

![In-to-Out](https://user-images.githubusercontent.com/62461290/190438268-c83677a9-b361-4253-ad3e-5931937d5b50.png)

Key Points  
- No clock involved.  
- Checked against maximum propagation delay.  
- Ensures combinational logic meets timing requirements.

---

### 5. **Clock Gating Analysis**
From clock pin through gating logic to gated clock output.  
Ensures enable signals don’t introduce glitches or metastability.

![Clock Gating](https://user-images.githubusercontent.com/62461290/190438984-51a8ecc5-a45a-4987-8ea3-be1bcf47cd6e.png)

Key Points  
- Enable must be stable before clock edge.  
- Uses dedicated ICG cells (integrated clock gating).  
- Prevents spurious clock pulses.

---

### 6. **Recovery & Removal Checks**
Asynchronous reset/set pins need special timing care.  
These checks guarantee that resets deassert safely relative to clock.

![Recovery Removal](https://user-images.githubusercontent.com/62461290/190439696-dd8d1c9a-2378-4633-8740-46456a9e12fa.png)

- **Recovery check:** reset is released soon enough before next clock.  
- **Removal check:** reset stays inactive long enough after clock.  

Violating these can cause random start-up states and glitches.

---

### 7. **Data-to-Data Timing**
Sometimes two signals interact without a shared clock.  
Data-to-data checks ensure their relative arrival times are safe.

![Data-to-Data](https://user-images.githubusercontent.com/62461290/190460805-258982bf-a65f-4618-8e0d-864ea31e4745.png)

Key Points  
- Prevents race conditions in multiplexers or handshake logic.  
- Validates causal relationships between signals.

---

### 8. **Latch Timing (Time Borrow / Time Give)**
Unlike flip-flops, **latches are level-sensitive**, which lets them borrow time from the next stage.  
Used strategically to improve timing closure in pipelined designs.

![Latch Timing](https://user-images.githubusercontent.com/62461290/190462109-b07a16c3-417e-467d-897b-3314a3adbefb.png)

Think of it as a friendly loan between neighboring stages — but borrow too much, and you’ll owe timing in the next cycle!

Key Points  
- Provides extra slack flexibility.  
- Needs careful balance to avoid cumulative debt (negative slack).  

---

## ⚡ Slew & Transition Analysis
Every signal edge takes time to transition — too slow, and power rises; too fast, and noise explodes.  
Slew analysis monitors this edge behavior.

### Data Slew (Max/Min)
![Data Slew](https://user-images.githubusercontent.com/62461290/190467735-0c3eec71-6df3-46c5-9595-34d08bc3eda4.png)

- **Max slew:** limits slow edges that increase short-circuit power and timing uncertainty.  
- **Min slew:** guards against too-sharp edges causing bounce and SI issues.

Think of it as controlling how “steep” each digital edge is.

---

### Clock Slew (Max/Min)
The clock needs the cleanest edges in the design.  
Any deviation here directly shrinks timing margins.

![Clock Slew](https://user-images.githubusercontent.com/62461290/190467955-2c211a54-1f08-4a78-a116-da218d337e46.png)

Importance  
- Impacts setup/hold windows and jitter.  
- Managed during Clock Tree Synthesis (CTS).  
- Typically tighter limits than data signals.

---

## 📊 Load Analysis
How much load a gate drives affects delay and power.  
STA tracks this through **fanout** and **capacitance** checks.

![Load Analysis](https://user-images.githubusercontent.com/110079770/190888536-3eb65ad6-2489-4fc1-8686-4ec74db1e40d.png)

### Fanout (Max/Min)
- **Max fanout:** too many loads → slow edges → timing issues.  
- **Min fanout:** too few loads → signal integrity noise.

Buffers or repeaters are inserted to re-balance loads.

---

### Capacitance (Max/Min)
- **Max:** total gate + wire + parasitic capacitance limit.  
- **Min:** ensures there’s some load to stabilize signal.

Rule of thumb: large nets → higher C → slower transition → need stronger drivers.

---

## 🕐 Clock Analysis
Clock quality defines design timing quality. STA looks at skew, pulse width, and timing windows.

### Clock Skew
![Clock Skew](https://user-images.githubusercontent.com/62461290/190468861-b79cae1c-5f51-4728-aa3b-ca8b91b99b0d.png)

**Definition:** difference in clock arrival times between registers.

- **Positive skew:** capture flop gets clock later → helps setup, hurts hold.  
- **Negative skew:** capture flop gets clock earlier → hurts setup, helps hold.

**Interactive note:** Try to guess — if setup fails and you advance the capture clock, what happens? Exactly: you make it worse! Hence skew engineering is delicate.

---

### Pulse Width Check
Real clock trees distort duty cycles.  
Pulse width checks ensure high/low times meet library requirements.

![Pulse Width](https://user-images.githubusercontent.com/62461290/190469402-ff5fc3a0-d3fa-4979-976e-40db0b754897.png)

Violations can lead to flip-flops missing edges or entering metastability.  
Maintaining balanced rise/fall durations is key to robust timing.

---

---

## 🎯 Reg-to-Reg Path Timing Parameters
This section dives into the heartbeat of STA — the **register-to-register path**.  
It’s here that setup and hold constraints decide your clock frequency limits.

**Assume:** Clock Frequency = 1 GHz ⇒ Clock Period = 1 ns.

---

### 🧩 Understanding Slack
Slack = Required Arrival Time (RAT) − Actual Arrival Time (AAT)  
It measures **timing comfort zone** — how early or late data reaches.

| Slack Value | Meaning | Action |
|--------------|----------|--------|
| +ve | Meets timing | Safe |
| 0 | Critical path | Monitor closely |
| −ve | Timing violation | Fix required |

🧠 **Think-through:** Negative slack means the design is asking the impossible — it needs more time than the clock allows.

---

### 📘 Example 1 – Slack Computation
![Setup Example](https://user-images.githubusercontent.com/62461290/190431165-83bc4713-2b6a-4beb-a17d-6944258c8ff2.png)  
Max Slack = 3 ns − 3.5 ns = −0.5 ns → **Setup Violation**

![Hold Example](https://user-images.githubusercontent.com/62461290/190429835-58719349-dfe6-4a48-a97c-cd1403a7c864.png)  
Min Slack = 3.5 ns − 0.5 ns = 3 ns → **Hold Met**

---

### 📘 Example 2 – Another Scenario
![Setup2](https://user-images.githubusercontent.com/62461290/190434143-0b421273-c59b-4926-bf26-92e58ae02069.png)  
Max Slack = 1 ns − 0.2 ns = 0.8 ns → Setup OK  

![Hold2](https://user-images.githubusercontent.com/62461290/190433799-0e17d633-518c-44a7-b92f-8fd36a6d680e.png)  
Min Slack = 0.2 ns − 0.3 ns = −0.1 ns → Hold Fail  

⏱️ Setup and Hold are mirror images — fixing one may hurt the other.

---

### 1️⃣ Arrival Time (AT)
When does data actually arrive?  
AT = Clock Network Delay + Clock-to-Q Delay + Combinational Delay.  

Two views:  
- **Early AT:** best-case (min) delays.  
- **Late AT:** worst-case (max) delays.

---

### 2️⃣ Required Time (RT)
Deadline for arrival.

- **Setup check:** RT = Clock Period + Clk Delay − Setup Time  
- **Hold check:** RT = Clk Delay + Hold Time  

📘 Rule: AAT ≤ RAT for setup and AAT ≥ RAT for hold.

---

### 3️⃣ Slack Equation
Slack = RT − AT  
Positive = Happy chip.  
Negative = Sad chip 🙂  

---

### 4️⃣ Clock-to-Q Delay (Tcq)
Time from active clock edge to Q output change.  
Affects both setup and hold checks.

Typical Range: 50–300 ps  
Higher load = larger Tcq = slower data.

---

### 5️⃣ Setup Time (Tsetup)
Minimum stability before clock edge.  

Equation:  
Tcycle ≥ Tcq + Tcomb + Tsetup + Tskew + Tuncertainty  

🧠 **Reflection:** Setup defines the *latest* moment data may arrive.

Violating it forces either frequency reduction or path optimization.

---

### 6️⃣ Hold Time (Thold)
Minimum stability after clock edge.  
Independent of period — you can’t fix it by slowing clock!  

Equation: Tcq + Tcomb ≥ Thold + Tskew − Tuncertainty  

Usually fixed by adding delays (buffers) on fast paths.

---

### 7️⃣ Jitter and Uncertainty
Jitter = random clock edge movement.  
Uncertainty combines jitter + variations + skew margin.  

Clock Period effective = T ± Δ  
Thus Setup RT = T − Tsetup − Jitter.  
Hold RT = Thold + Jitter.

Visualized using eye diagrams for realistic clock windows.

---

### 8️⃣ Textual Timing Reports
STA tools report timing in tabular text.  
You’ll see net delays, cell delays, and slack values for each path.

Think of it as a map from launch flop → capture flop annotated with numbers.

---

### 9️⃣ Graph-Based vs Path-Based Analysis

| Feature | GBA | PBA |
|----------|-----|-----|
| Speed | Fast (global) | Slow (per path) |
| Accuracy | Pessimistic | Realistic |
| Usage | Early design | Final closure |
| Pessimism | High | Low |

GBA is quick for coverage, PBA is truth for sign-off.

---

### 🔹 Complete Reg-to-Reg Equations
**Setup:** Tclk ≥ LaunchClk(late)+Tcq(max)+Data(max)+Tsetup−CaptureClk(late)+Uncertainty−CRPR  
**Hold:** LaunchClk(early)+Tcq(min)+Data(min) ≥ CaptureClk(early)+Thold+Uncertainty−CRPR

---

## 🧬 On-Chip Variation (OCV)
No two transistors are identical.  
Differences in process, voltage, and temperature create delay mismatch → **OCV**.

### ⚙️ Sources
1. **Process:** etching, doping, oxide thickness.  
2. **Voltage:** IR drop and dynamic fluctuations.  
3. **Temperature:** hot spots and thermal gradients.

---

### 📉 OCV Derating
Delays are scaled to simulate variation:  

Max Delay = Nominal × (1 + OCV)  
Min Delay = Nominal × (1 − OCV)  

Typical cell derate ≈ ±10–15 %.  
Clock path derate ≈ ±3–5 %.  
In this course ≈ ±20 %.

---

### 🧩 Etching and Oxide Impact
Uneven etching alters W/L ratios → changes drain current.  
Oxide thickness variation changes capacitance → affects switch speed.  

Each inverter ends up with slightly different delay.  
This spreads out timing across the chip like ripples in water.

---

## 🛠️ OCV Timing and Pessimism Removal

### 🔧 Setup OCV Scenarios
![OCV Setup](https://user-images.githubusercontent.com/62461290/190919502-519015cf-38f2-4138-98ab-ddbf266e4425.png)

Four delay combinations exist (↑ or ↓ for DRT/DAT).  
- **Clock pull-in:** reducing capture delay.  
- **Clock push-out:** increasing launch delay.  

Combining gives a realistic worst-case picture.

---

### 🧮 Common Path Pessimism Removal (CRPR)
Both launch and capture flops share some clock path.  
Without CRPR, the tool double-counts variation → extra pessimism.

**Rule:** A cell cannot have two delay values at once.  
Subtract the common difference to restore accuracy.

Result → healthier slack and realistic timing sign-off.

---

### 🕐 Hold OCV
Hold analysis uses opposite derating.  
To stress worst case, launch delay is shrunk (pull-in) and capture delay expanded (push-out).  
If hold fails under these conditions, the design is non-recoverable.

---

## 📈 Outcome
After completing this week’s study, you should be able to:

✅ Identify and analyze setup/hold paths of all types.  
✅ Compute slack and understand its impact.  
✅ Interpret textual STA reports.  
✅ Relate flip-flop internals to timing behavior.  
✅ Explain OCV sources and their modeling.  
✅ Apply CRPR to reduce false violations.  

---

## 🎓 Key Takeaways
1. STA verifies timing exhaustively — no test vectors needed.  
2. Setup + Hold define upper/lower timing bounds.  
3. Clock integrity (skew, jitter, pulse width) is paramount.  
4. OCV captures manufacturing and environmental variability.  
5. PBA refines accuracy for sign-off.  
6. Hold violations are fatal; setup violations limit frequency.  
7. CRPR eliminates double-counted pessimism.  
8. Jitter and uncertainty shrink timing margins — always account for them.  
9. A positive slack design means a happy tape-out.

---

## 📝 Conclusion
We have now connected the dots — from basic path timing to deep OCV and CRPR concepts.  
STA is not just a report of numbers; it’s the language your silicon speaks in time.

**Remember:**  
- Every path tells a story of data and clock racing each other.  
- A balanced design keeps them in sync.  
- STA lets you see that balance before hardware exists.  

---

