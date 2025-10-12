# 🧾 GLS Verification Note — BabySoC

The Gate-Level Simulation (GLS) for the **BabySoC** design was successfully completed and compared against the **pre-synthesis (functional)** simulation.

Both simulations were analyzed in GTKWave using the following key signals:
- Control signals: `reset`, `EN`, and PLL outputs  
- Core pipeline signals: `D[9:0]`, `Dext[10:0]`  
- DAC output: `OUT` with reference voltages `VREFH` and `VREFL`

### ✅ Observation Summary
- All control and clock signals in the gate-level simulation show the **same logical transitions** as in the functional simulation, with **minor propagation delays** due to gate-level timing.
- Core and DAC outputs are **functionally identical**, demonstrating correct data flow and conversion.
- The DAC output waveform (`OUT`) maintains the **same analog shape and amplitude**, with only a slight time shift — consistent with expected synthesis delay.

### 🧠 Conclusion
The post-synthesis netlist **preserves full logical correctness** of the RTL design.  
Both simulations (functional and gate-level) produce **identical functional behavior** and consistent waveform patterns.

### ✅ Final Verdict
**GLS = Functional Outputs**  
The BabySoC design has been **successfully verified at gate level**, confirming synthesis correctness.
