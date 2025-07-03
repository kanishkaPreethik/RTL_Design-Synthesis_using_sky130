# Day-02 : Timing Libs, Hierarchical vs Flat Synthesis and efficient Flop Coding Styles 

  This day covers three crucial topics:
- Understanding the `.lib` timing library (sky130_fd_sc_hd__tt_025C_1v80.lib) used in open-source PDKs.
- Comparing hierarchical vs. flat synthesis methods.
- Exploring efficient coding styles for flip-flops in RTL design.

# Contents

- [Timing Libraries](#timing-libraries)
  - [SKY130 PDK Overview](#sky130-pdk-overview)
  - [Decoding tt_025C_1v80 in the SKY130 PDK](#decoding-tt_025c_1v80-in-the-sky130-pdk)
  - [Opening and Exploring the .lib File](#opening-and-exploring-the-lib-file)

- [Hierarchical vs. Flattened Synthesis](#hierarchical-vs-flattened-synthesis)
  - [Hierarchical Synthesis](#hierarchical-synthesis)
  - [Flattened Synthesis](#flattened-synthesis)
  - [Key Differences](#key-differences)

- [Flip-Flop Coding Styles](#flip-flop-coding-styles)
  - [Asynchronous Reset D Flip-Flop](#asynchronous-reset-d-flip-flop)
  - [Asynchronous Set D Flip-Flop](#asynchronous-set-d-flip-flop)
  - [Synchronous Reset D Flip-Flop](#synchronous-reset-d-flip-flop)

- [Simulation and Synthesis Workflow](#simulation-and-synthesis-workflow)
  - [Icarus Verilog Simulation](#icarus-verilog-simulation)
  - [Synthesis with Yosys](#synthesis-with-yosys)
## 1. Introduction to timing .libs 
## About Sky130 PDK and `tt_25c_1v80` Timing Library

The **Sky130 PDK** (Process Design Kit) is an open-source 130nm CMOS process developed by **SkyWater Technology** in collaboration with **Google**. It provides all the necessary design rules, models, and libraries to enable both analog and digital IC design using industry-grade tools.

### What is `tt_25c_1v80.lib`?

This file is a **Liberty (`.lib`) format timing library**, representing the behavior of standard cells under **Typical-Typical (TT)** process conditions at:

- **Temperature:** 25°C  
- **Supply Voltage:** 1.80V

This specific PVT (Process-Voltage-Temperature) corner — `tt_25c_1v80` — is typically used during:

-  Functional simulation  
-  Logic synthesis  
-  Static timing analysis  

### How It's Used in the Flow

The `tt_25c_1v80.lib` file is used by key open-source EDA tools such as:

- [`Yosys`](https://github.com/YosysHQ/yosys) – for logic synthesis  
- [`OpenSTA`](https://github.com/The-OpenROAD-Project/OpenSTA) – for static timing analysis  
- [`OpenROAD`](https://github.com/The-OpenROAD-Project/OpenROAD) – for full RTL2GDS flow

### What does a timing lib contains 
It contains crucial information for each standard cell, including:

- Propagation delays  
- Setup and hold times  
- Transition (slew) times  
- Dynamic and leakage power data
- Different flavours of different cells as well as different flavours of same cells.
- Different features of each cell like power info and timing info as mentioned above.

 ---

## 2. Hierarchical vs Flat Synthesis 

In RTL-to-Gates design flow, **synthesis** converts RTL code (like Verilog) into gate-level netlists using standard cell libraries. There are two main approaches to how synthesis tools handle module hierarchy:

### Flat Synthesis

In **flat synthesis**, the synthesis tool *flattens* the entire RTL hierarchy before optimization. This means:

- All submodules are inlined and treated as one large design.
- Tool has full visibility across module boundaries.
- Enables **global optimizations**, such as:
  - Logic sharing across modules
  - Better area/timing/power trade-offs

#### Advantages:
- Potentially better **timing**, **area**, and **power** optimization.
- Tool can perform **global logic restructuring**.

#### Disadvantages:
- Can be **very memory- and time-intensive** for large designs.
- Debugging becomes harder due to loss of module boundaries.

### Hierarchical Synthesis

In **hierarchical synthesis**, each module is synthesized **separately**, preserving the RTL hierarchy.

- Top-level module keeps references to synthesized submodules.
- Useful for large SoCs with reusable IP blocks or black-box modules.

#### Advantages:
- **Faster** and **memory-efficient** synthesis.
- Promotes **IP reuse** and modularity.
- Easier **debug** and design management.

#### Disadvantages:
- Limited optimization **across module boundaries**.
- May lead to suboptimal timing or area due to less global visibility.



### When to Use What?

| Scenario                        | Preferred Approach      |
|--------------------------------|-------------------------|
| Small to medium design         | Flat Synthesis          |
| Very large SoC / multi-IP      | Hierarchical Synthesis  |
| Emphasis on timing optimization| Flat Synthesis          |
| Emphasis on design reuse/modularity | Hierarchical Synthesis  |

> *In real-world flows, a hybrid approach is often used: critical modules are synthesized flat, while non-critical blocks use hierarchical synthesis.*

---
## Module-level vs Submodule-level Synthesis

In hierarchical RTL design, synthesis can be performed at different levels of abstraction depending on design reuse, optimization needs, and scalability. Two commonly used approaches are **submodule-level synthesis** and **module-level synthesis**.


### Submodule-level Synthesis

- In submodule-level synthesis, individual submodules are synthesized independently and the resulting netlist is reused across all instances. This approach is ideal when the same submodule is instantiated multiple times in the design — such as ALUs, buffers, FIFOs, or custom IPs.

- This method improves synthesis runtime, promotes IP reuse, and reduces overall netlist size. However, since each instance uses the same pre-synthesized logic, the tool cannot optimize the submodule based on its context within different parts of the design.


### Module-level Synthesis

- Module-level synthesis refers to synthesizing a higher-level module (such as a complete block or top module) that may include many submodules within it. In this approach, the synthesis tool sees the full context of the design and can optimize each instance of a submodule differently, depending on timing and logic conditions.

- This method enables more effective area/timing/power optimization at the cost of increased synthesis time and memory usage. It is especially useful for performance-critical designs or modules with unique logic paths.


### When to Use What?

| Use Case                                  | Recommended Approach      |
|-------------------------------------------|---------------------------|
| Repeated modules (e.g. FIFOs, IPs)        | Submodule-level synthesis |
| Unique, timing-critical instances         | Module-level synthesis    |
| Faster synthesis with reusable blocks     | Submodule-level synthesis |
| Better optimization and performance tuning| Module-level synthesis    |

>  *In practice, a hybrid strategy is often used — reusable IPs are synthesized at the submodule level, while performance-sensitive blocks are synthesized at the module level with full context.*
---

## 3. Flop Coding Styles and Optimization

- Flip-flops are fundamental sequential elements in digital design, used to store binary data. Below are efficient coding styles for different reset/set behaviors.

### Asynchronous Reset D Flip-Flop

```verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
- **Asynchronous reset**: Overrides clock, setting q to 0 immediately.
- **Edge-triggered**: Captures d on rising clock edge if reset is low.

### Asynchronous Set D Flip-Flop

```verilog
module dff_async_set (input clk, input async_set, input d, output reg q);
  always @ (posedge clk, posedge async_set)
    if (async_set)
      q <= 1'b1;
    else
      q <= d;
endmodule
```
- **Asynchronous set**: Overrides clock, setting q to 1 immediately.

### Synchronous Reset D Flip-Flop

```verilog
module dff_syncres (input clk, input async_reset, input sync_reset, input d, output reg q);
  always @ (posedge clk)
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
- **Synchronous reset**: Takes effect only on the clock edge.

---

## Simulation and Synthesis Workflow

### Icarus Verilog Simulation

1. **Compile:**
   ```shell
   iverilog dff_asyncres.v tb_dff_asyncres.v
   ```
2. **Run:**
   ```shell
   ./a.out
   ```
3. **View Waveform:**
   ```shell
   gtkwave tb_dff_asyncres.vcd
   ```
   ---
