# DAY-04 : GLS, Synthesis-Simulation mismatch, Blocking vs Non-Blocking 
> DAY-04 Covers Gate Level Simulation, Synthesis-Simulation Mismatch, Blocking vs Non-Blocking Statements in verilog, which play an essential role in providing a clean and Error free design.
---
# 1. Gate Level Simulation (GLS)
## What is Gate Level Simulation 
- GLS is considered as running the simulation with Test bench and the netlist as the DUT(Design under Test).
- Netlist is logicslly same as the RTL code.
  - Same Test bench will align with the design.

## Why we need GLS 
- GLS is required to verify the logical correctness of the design after Synthesis.
- Also we ensure the timing of the design is met.
   - For this GLS needs to be run with delay annotation.

 ![Screenshot (257)](https://github.com/user-attachments/assets/9bdc219f-db0e-4bb0-ad84-8dd5fba6f365)

 # 2. Synthesis - Simulation Mismatch 
- Synthesis-Simulation mismatch occurs when the behavior of your RTL differs between simulation (functional verification) and synthesis (hardware implementation). These mismatches can lead to functional bugs, incorrect timing, or non-functional hardware.

## 2.1 Missing Sensitivity List in `always` Blocks
### Issue:
In Verilog-2001 and earlier, manual sensitivity lists are used for combinational logic. Missing signals can cause incorrect simulation results.

```verilog
// Incorrect: 'b' is missing from sensitivity list
always @(a)
  y = a & b;
```
### Simulation Behavior:
- Simulator only updates y when a changes.
- If b changes, y remains stale — leads to incorrect simulation output.

### Synthesis Behavior:
- Synthesizer assumes always @(*) (complete sensitivity).
- Final circuit behaves correctly, creating a mismatch with simulation.

### Fix:
- Use always @(*) to ensure complete sensitivity.

```verilog
Copy
Edit
always @(*) // or always_comb
  y = a & b;
```
---

## 2.2 Blocking vs Non-blocking Assignments
### Key Difference:
- = (blocking): Executes in order, can cause race conditions.
- <= (non-blocking): Executes concurrently, matches flip-flop behavior.

### Issue:
- Using blocking (=) assignments in sequential (always @(posedge clk)) blocks can cause simulation to behave differently from synthesized hardware.

```verilog
Copy
Edit
// Bad: Mixing blocking in sequential logic
always @(posedge clk) begin
  q1 = d;
  q2 = q1;  // q2 gets updated with new q1 value in sim, but not in synthesis
end
```
### Simulation:
- Executes line-by-line.
- q2 gets new q1 in same cycle — leads to incorrect simulation behavior.

### Synthesis:
- Flip-flops update in parallel — q2 gets old value of q1.

### Fix:
Use non-blocking assignments (<=) for all sequential logic.

```verilog
Copy
Edit
always @(posedge clk) begin
  q1 <= d;
  q2 <= q1;
end
```
## Comparison Table: Blocking (`=`) vs Non-blocking (`<=`) Assignments

| Feature / Aspect                | Blocking Assignment (`=`)                | Non-blocking Assignment (`<=`)              |
|----------------------------------|------------------------------------------|----------------------------------------------|
| Execution Order               | Sequential (line-by-line)                | Concurrent (evaluated, then updated)         |
| Simulation Behavior           | Executes immediately                     | Schedules update after current time step     |
| Typical Usage                 | Combinational logic                      | Sequential logic (flip-flops)                |
| Risk of Race Conditions       | High if used in sequential blocks        | Low (preferred for sequential design)        |
| Overwriting Intermediate Values | Possible                                 | Avoided due to concurrent update             |
| Synthesis Compatibility       | Yes (with care)                          | Yes (safest for sequential logic)            |
| Common Mistake                | Using `=` in `always @(posedge clk)`     | Rare if used correctly                       |
| Best Practice                 | Use for `always @(*)` (combinational)    | Use for `always @(posedge clk)` (sequential) |

--- 

## 2.3 Non-standard or Unintended Verilog Constructs
### Examples of Risky or Ambiguous Code:
- Uninitialized registers

- Inferred latches due to partial assignments

- Use of delays (#10) in synthesizable code

- Assignments inside initial blocks for synthesized logic


