# MODULE-1-2
RTL design, simulation and logic synthesis using Icarus Verilog, GTKWave and Yosys with the Sky130 standard-cell library.
# RTL Design, Simulation and Synthesis: Modules 1 and 2


## Table of Contents
- [Module 1: RTL Design, Simulation and Yosys Basics](#module-1-rtl-design-simulation-and-yosys-basics)
- [Module 2: Timing Libraries, Hierarchical vs Flat Synthesis and Flip-Flop Coding](#module-2-timing-libraries-hierarchical-vs-flat-synthesis-and-flip-flop-coding)
- [Overall Summary](#overall-summary)

---

# Module 1: RTL Design, Simulation and Yosys Basics

## 1.1 Introduction to RTL Design and Simulation

| Term | Meaning |
|---|---|
| **Design (DUT)** | The hardware logic that implements the required functionality |
| **Simulation** | Testing the behaviour of the design in software before hardware implementation |
| **Test Bench** | A verification environment that generates inputs (stimulus) and observes the outputs of the DUT |

```mermaid
flowchart LR
    A["Specification"] --> B["RTL Design<br/>Verilog"]
    B --> C["Simulator<br/>Icarus Verilog"]
    D["Test Bench<br/>stimulus"] --> C
    C --> E["VCD Waveform File"]
    E --> F["GTKWave<br/>Waveform Viewer"]
    F --> G{"Behaviour<br/>correct?"}
    G -- "Yes" --> H["Design verified"]
    G -- "No" --> I["Debug and fix RTL"]
    I --> B
```

### Testbench and DUT relationship

```mermaid
flowchart LR
    subgraph TB["Test Bench"]
        S["Stimulus Generator"]
        M["Output Observer"]
    end
    subgraph DUT["Design Under Test"]
        L["RTL Logic"]
    end
    S -- "primary inputs" --> L
    L -- "primary outputs" --> M
    M --> V["VCD dump file"]
```

---

## 1.2 Environment Setup

| Tool | Role |
|---|---|
| **Git** | Cloned the `sky130RTLDesignAndSynthesisWorkshop` repository (RTL examples, libraries, exercises) |
| **Icarus Verilog (`iverilog`)** | Open-source Verilog compiler and simulator |
| **GTKWave** | Waveform viewer for VCD files |
| **GVim / Neovim** | Text editor for Verilog source files |
| **Yosys** | Open-source RTL synthesis framework |

```bash
sudo apt install iverilog
sudo apt install gtkwave
sudo apt install neovim-qt
```

```mermaid
flowchart TD
    A["Start"] --> B["Clone workshop repository<br/>from GitHub"]
    B --> C["Install Icarus Verilog"]
    C --> D["Install GTKWave"]
    D --> E["Install GVim or Neovim"]
    E --> F["Explore verilog_files folder"]
    F --> G["Environment ready"]
```

---

## 1.3 Lab 2: Icarus Verilog, GTKWave and GVim

**Files:** `good_mux.v` (2:1 multiplexer) and `tb_good_mux.v` (test bench)

### Design

```verilog
module good_mux (input i0, input i1, input sel, output reg y);
always @(*)
begin
    if(sel)
        y <= i1;
    else
        y <= i0;
end
endmodule
```

> **Note:** The recommended style for combinational logic is blocking assignment (`=`). The `<=` was used in the workshop file as given.

### Test bench highlights

```verilog
good_mux uut (.sel(sel), .i0(i0), .i1(i1), .y(y));

$dumpfile("tb_good_mux.vcd");
$dumpvars(0, tb_good_mux);

always #75 sel = ~sel;
always #10 i0  = ~i0;
always #55 i1  = ~i1;
```

### Simulation commands

```bash
iverilog good_mux.v tb_good_mux.v
./a.out
gtkwave tb_good_mux.vcd
```

```mermaid
flowchart TD
    A["good_mux.v + tb_good_mux.v"] --> B["iverilog good_mux.v tb_good_mux.v"]
    B --> C["a.out generated"]
    C --> D["./a.out"]
    D --> E["tb_good_mux.vcd generated"]
    E --> F["gtkwave tb_good_mux.vcd"]
    F --> G["Drag signals from left panel<br/>into the waveform window"]
    G --> H["Verify mux behaviour"]
```

### 2:1 Multiplexer logic

```mermaid
flowchart LR
    I0["i0"] --> MUX{"2:1 MUX"}
    I1["i1"] --> MUX
    SEL["sel"] --> MUX
    MUX --> Y["y"]
    SEL -. "sel = 0 gives y = i0<br/>sel = 1 gives y = i1" .-> MUX
```

---

## 1.4 Introduction to Yosys and Logic Synthesis

**Yosys** is an open-source RTL synthesis framework. It reads Verilog RTL, optimizes it, and maps it to cells from a technology library to produce a gate-level netlist.

| Command | Purpose |
|---|---|
| `read_verilog` | Reads the RTL design |
| `read_liberty` | Reads the standard-cell library |
| `write_verilog` | Writes the synthesized netlist |

### What is logic synthesis?

```mermaid
flowchart LR
    A["RTL<br/>behavioural description"] --> B["Logic Synthesis<br/>Yosys"]
    L["Standard Cell Library<br/>.lib"] --> B
    B --> C["Gate-Level Netlist<br/>gates + connections"]
```

### Why different cell flavours?

Digital circuits have capacitive loads that must be charged and discharged.

```mermaid
flowchart TD
    A["Standard cell flavours"] --> B["Wider transistors"]
    A --> C["Narrower transistors"]
    B --> B1["Higher current"]
    B1 --> B2["Lower delay<br/>FAST cell"]
    B2 --> B3["More area + more power"]
    C --> C1["Lower current"]
    C1 --> C2["Higher delay<br/>SLOW cell"]
    C2 --> C3["Less area + less power"]
```

### Timing constraint

```
Tclk > Tcq + Tcombi + Tsetup
```

```mermaid
flowchart LR
    FF1["Launch Flip-Flop"] -- "Tcq" --> CL["Combinational Logic<br/>Tcombi"]
    CL -- "Tcombi" --> FF2["Capture Flip-Flop<br/>Tsetup"]
    CL -.-> R["Clock period must exceed<br/>Tcq + Tcombi + Tsetup"]
```

```mermaid
flowchart TD
    A{"Is the path fast enough<br/>for the clock period?"} -- "No" --> B["Choose faster cell flavour"]
    A -- "Yes, with spare margin" --> C["Choose smaller or slower cell<br/>to save area and power"]
    B --> D["Synthesis tool balances<br/>speed, area and power"]
    C --> D
```

### Verifying the synthesized netlist

The same test bench is reused because the primary inputs and outputs do not change.

```mermaid
flowchart TD
    RTL["RTL Design"] --> SYN["Synthesis"]
    SYN --> NET["Gate-Level Netlist"]
    TB["Same Test Bench"] --> SIM1["RTL Simulation"]
    TB --> SIM2["Netlist Simulation"]
    RTL --> SIM1
    NET --> SIM2
    SIM1 --> W1["Waveform A"]
    SIM2 --> W2["Waveform B"]
    W1 --> CMP{"Waveforms<br/>match?"}
    W2 --> CMP
    CMP -- "Yes" --> OK["Synthesis is functionally correct"]
    CMP -- "No" --> BAD["Investigate mismatch"]
```

---

## 1.5 Lab 3: Invoke Yosys and Synthesize `good_mux`

### Commands (in order)

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog good_mux_netlist.v
```

| Step | Command | What it does |
|---|---|---|
| 1 | `read_liberty -lib` | Loads the Sky130 cell library |
| 2 | `read_verilog` | Reads the RTL of the multiplexer |
| 3 | `synth -top good_mux` | Runs RTL synthesis with `good_mux` as top module |
| 4 | `abc -liberty` | Optimizes logic and maps it to Sky130 standard cells |
| 5 | `show` | Displays the synthesized circuit graphically |
| 6 | `write_verilog` | Writes the gate-level netlist file |

```mermaid
flowchart TD
    A["Start Yosys<br/>yosys"] --> B["read_liberty -lib<br/>Load Sky130 .lib"]
    B --> C["read_verilog good_mux.v<br/>Load RTL"]
    C --> D["synth -top good_mux<br/>RTL synthesis"]
    D --> E["abc -liberty<br/>Technology mapping"]
    E --> F["show<br/>View schematic"]
    F --> G["write_verilog good_mux_netlist.v<br/>Save netlist"]
    G --> H["Netlist uses<br/>sky130_fd_sc_hd__mux2_1"]
```

### RTL to gate-level mapping

```mermaid
flowchart LR
    A["if (sel) y = i1;<br/>else y = i0;"] --> B["Yosys + ABC"]
    B --> C["sky130_fd_sc_hd__mux2_1<br/>standard cell"]
```

---

## Module 1 Learning Outcomes
- Understood the difference between Design, Simulation and Test Bench.
- Set up and verified Icarus Verilog, GTKWave and GVim.
- Compiled and simulated `good_mux`, and viewed its VCD waveforms.
- Understood RTL, logic synthesis, `.lib` files and cell flavours.
- Learned the speed, area and power trade-off and the timing constraint.
- Invoked Yosys, synthesized `good_mux` and generated a gate-level netlist.
- Understood how a netlist is verified with the same test bench.

---

# Module 2: Timing Libraries, Hierarchical vs Flat Synthesis and Flip-Flop Coding

## 2.1 Sky130 Standard Cell Library Walkthrough

The `.lib` file was opened in GVim to understand how cells are characterized.

```mermaid
flowchart TD
    LIB["Sky130 .lib file"] --> H["Library header"]
    LIB --> CELLS["cell definitions"]
    H --> H1["PVT conditions"]
    H --> H2["delay_model: table_lookup"]
    H --> H3["technology: cmos"]
    H --> H4["Units"]
    CELLS --> C1["Area"]
    CELLS --> C2["Power"]
    CELLS --> C3["Pins and functionality"]
    CELLS --> C4["Timing tables"]
```

### PVT: Process, Voltage, Temperature

| Parameter | Value in library | Meaning |
|---|---|---|
| **Process** | Typical (tt) | Manufacturing variation |
| **Voltage** | 1.80 V | Supply voltage |
| **Temperature** | 25 °C | Operating temperature |

```mermaid
flowchart LR
    P["Process<br/>tt"] --> PVT["PVT corner<br/>tt_025C_1v80"]
    V["Voltage<br/>1.80 V"] --> PVT
    T["Temperature<br/>25 C"] --> PVT
    PVT --> D["Affects cell delay and power"]
```

### Library settings

| Item | Value |
|---|---|
| Delay model | `table_lookup` (indexed by input transition and output load) |
| Technology | `cmos` |
| Time | ns |
| Voltage | V |
| Current | mA |
| Resistance | kohm |
| Capacitance | pF |
| Leakage power | nW (per the `.lib` header) |

```mermaid
flowchart LR
    IS["Input slew"] --> TBL["Delay lookup table"]
    OL["Output load"] --> TBL
    TBL --> DEL["Cell delay"]
```

### Different flavours of the same gate

| AND gate | Area | Drive strength |
|---|---|---|
| `sky130_fd_sc_hd__and2_0` | 6.25 | Lower |
| `sky130_fd_sc_hd__and2_2` | 7.50 | Medium |
| `sky130_fd_sc_hd__and2_4` | 8.75 | Higher |

```mermaid
flowchart LR
    A["and2_0<br/>area 6.25"] --> B["and2_2<br/>area 7.50"] --> C["and2_4<br/>area 8.75"]
    A -.-> A1["Slowest, least power"]
    C -.-> C1["Fastest, most power"]
```

```mermaid
flowchart TD
    A["Higher drive strength"] --> B["More current"]
    B --> C["Lower delay"]
    A --> D["Larger area"]
    A --> E["More power"]
```

---

## 2.2 Hierarchical vs Flat Synthesis

### Design used: `multiple_modules.v`

```verilog
module sub_module1(input a, input b, output y);
    assign y = a & b;
endmodule

module sub_module2(input a, input b, output y);
    assign y = a | b;
endmodule

module multiple_modules(input a, input b, input c, output y);
    wire net1;
    sub_module1 u1 (.a(a), .b(b), .y(net1));
    sub_module2 u2 (.a(net1), .b(c), .y(y));
endmodule
```

Function implemented: **y = (a & b) | c**

### Design structure

```mermaid
flowchart LR
    a["a"] --> U1["u1: sub_module1<br/>AND"]
    b["b"] --> U1
    U1 -- "net1" --> U2["u2: sub_module2<br/>OR"]
    c["c"] --> U2
    U2 --> y["y"]
```

### Reading the design into Yosys

```
yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
```

### Flow comparison

```mermaid
flowchart TD
    A["read_liberty + read_verilog"] --> B{"Flatten the<br/>hierarchy?"}
    B -- "No" --> C["synth -top multiple_modules"]
    B -- "Yes" --> D["synth -top multiple_modules -flatten"]
    C --> E["Hierarchical netlist<br/>modules preserved"]
    D --> F["Flat netlist<br/>single top module"]
    E --> G["show multiple_modules"]
    F --> G
```

### Hierarchical netlist

```mermaid
flowchart TD
    TOP["multiple_modules"] --> U1["u1"]
    TOP --> U2["u2"]
    U1 --> S1["sub_module1<br/>AND logic"]
    U2 --> S2["sub_module2<br/>OR logic"]
```

### Flat netlist

```mermaid
flowchart TD
    TOP["multiple_modules"] --> AND["AND cell"]
    TOP --> OR["OR cell"]
```

### Side-by-side comparison

| Feature | Hierarchical | Flat |
|---|---|---|
| Command | `synth -top multiple_modules` | `synth -top multiple_modules -flatten` |
| Module boundaries | Preserved | Removed |
| Optimization across modules | Limited | Greater freedom |
| Netlist size | Easier to read | Can become large |
| Debugging | Easier | Harder |
| Best for | Large SoCs, reusable IP, repeated modules, block-level design | Small designs needing maximum optimization |
| Logic function | y = (a & b) \| c | y = (a & b) \| c |

```mermaid
flowchart TD
    Q{"Which approach?"} --> A{"Large design with<br/>reusable IP or blocks?"}
    A -- "Yes" --> H["Hierarchical synthesis"]
    A -- "No" --> B{"Need maximum<br/>cross-module optimization?"}
    B -- "Yes" --> F["Flat synthesis"]
    B -- "No" --> H
    H --> H1["Easier debug and management"]
    F --> F1["More optimization freedom"]
```

> Both approaches implement the **same logic function**. Only the structure of the netlist differs.

---

## 2.3 Why Flip-Flops and Flop Coding Styles

### Why use flip-flops?
- Store the state of a circuit
- Synchronize data with the clock
- Break long combinational paths into smaller timing stages
- Build registers, counters, state machines and pipelines

### What is a glitch?
A short, unwanted output change in a combinational circuit, caused by different propagation delays through different paths.

```mermaid
flowchart TD
    A["Multiple inputs change<br/>at nearly the same time"] --> B["Signals reach gates<br/>at different times"]
    B --> C["Output briefly takes<br/>incorrect value"]
    C --> D["Glitch"]
    D --> E["Flip-flop samples only<br/>at clock edge"]
    E --> F["Glitch not stored as state"]
```

### Flip-flop operation

```mermaid
flowchart LR
    D["D input"] --> FF["Flip-Flop"]
    CLK["Clock edge"] --> FF
    FF --> Q["Q output<br/>holds value until<br/>next active edge"]
```

### Asynchronous reset

```verilog
always @(posedge clk or posedge async_reset)
begin
    if (async_reset)
        q <= 1'b0;
    else
        q <= d;
end
```

### Synchronous reset

```verilog
always @(posedge clk)
begin
    if (sync_reset)
        q <= 1'b0;
    else
        q <= d;
end
```

```mermaid
flowchart TD
    A{"Reset type?"} -- "Asynchronous" --> B["Sensitivity list:<br/>posedge clk or posedge reset"]
    A -- "Synchronous" --> C["Sensitivity list:<br/>posedge clk only"]
    B --> D["Output reset immediately,<br/>independent of clock"]
    C --> E["Reset checked<br/>only at clock edge"]
```

| Feature | Asynchronous Reset | Synchronous Reset |
|---|---|---|
| Clock dependency | Independent of clock | Depends on clock |
| Reset response | Immediate | At clock edge |
| Sensitivity list | Clock + reset | Clock only |
| Main advantage | Immediate initialization | Controlled timing |

### Mixed reset
Reset is **asserted asynchronously** (known state immediately) and **released synchronously** (controlled release with the clock).

```mermaid
flowchart LR
    A["Reset asserted"] --> B["Asynchronous:<br/>immediate reset"]
    C["Reset release"] --> D["Synchronized<br/>to clock"]
    B --> E["Immediate reset + controlled release"]
    D --> E
```

---

## 2.4 Lab 4: Flip-Flop Simulation and Synthesis

**Designs used:**

| File | Description |
|---|---|
| `dff_async.v` | Asynchronous reset flip-flop |
| `dff_async_set.v` | Asynchronous set flip-flop |
| `dff_syncres.v` | Synchronous reset flip-flop |
| `dff_asyncres.v` | Flip-flop with asynchronous reset |

### Simulation flow

```bash
iverilog -o <output>.out <design>.v <testbench>.v
vvp <output>.out
gtkwave <waveform>.vcd
```

```mermaid
flowchart TD
    A["Flip-flop RTL + testbench"] --> B["iverilog -o out design.v tb.v"]
    B --> C["vvp out"]
    C --> D[".vcd waveform generated"]
    D --> E["gtkwave waveform.vcd"]
    E --> F["Check clk, d, reset or set, q"]
    F --> G["RTL behaviour verified"]
```

### Synthesis flow

```
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_async.v
synth -top dff_async
show
```

```mermaid
flowchart TD
    A["Simulate RTL"] --> B["Start Yosys"]
    B --> C["read_liberty -lib"]
    C --> D["read_verilog dff_async.v"]
    D --> E["synth -top dff_async"]
    E --> F["show<br/>Graphviz .dot netlist"]
    F --> G["Inspect Sky130 cells<br/>and connections"]
    G --> H["Repeat for other flip-flop designs"]
```

### What each RTL style synthesizes into

```mermaid
flowchart TD
    R["Flip-flop RTL coding style"] --> A["Async reset"]
    R --> S["Async set"]
    R --> Y["Sync reset"]
    R --> X["Async reset with<br/>additional logic"]
    A --> A1["DFF with dedicated<br/>RESET_B pin"]
    S --> S1["DFF with dedicated<br/>SET_B pin"]
    Y --> Y1["Combinational logic<br/>before D input + DFF"]
    X --> X1["Logic followed by<br/>DFF reset control"]
```

| RTL Design | Synthesized Implementation |
|---|---|
| Asynchronous reset | DFF with dedicated reset pin (`RESET_B`) |
| Asynchronous set | DFF with dedicated set pin (`SET_B`) |
| Synchronous reset | Combinational logic + DFF |
| Asynchronous reset with additional logic | Logic followed by DFF reset control |

### Reset connections

```mermaid
flowchart LR
    subgraph ASYNC["Asynchronous reset"]
        AR["async_reset"] --> FF1["DFF<br/>RESET_B pin"]
        D1["d"] --> FF1
        C1["clk"] --> FF1
        FF1 --> Q1["q"]
    end
    subgraph SYNC["Synchronous reset"]
        SR["sync_reset"] --> G["Combinational<br/>logic"]
        D2["d"] --> G
        G --> FF2["DFF D input"]
        C2["clk"] --> FF2
        FF2 --> Q2["q"]
    end
```

> The `.dot` netlists show that **different RTL coding styles produce different hardware structures** after synthesis.

---

## Module 2 Learning Outcomes
- Explored the Sky130 `.lib` file: PVT, delay model, units, cell definitions.
- Compared `and2_0`, `and2_2` and `and2_4` flavours (area vs drive strength).
- Compared hierarchical and flat synthesis using `multiple_modules.v`.
- Understood why flip-flops are used and what glitches are.
- Differentiated between synchronous, asynchronous and mixed reset.
- Simulated flip-flops with Icarus Verilog and GTKWave.
- Synthesized them in Yosys and inspected the mapped Sky130 cells.
- Related RTL coding style to the synthesized hardware structure.

---

# Overall Summary

```mermaid
flowchart TD
    subgraph M1["Module 1"]
        A1["RTL design and simulation basics"] --> A2["Tool setup"]
        A2 --> A3["Lab 2: good_mux simulation"]
        A3 --> A4["Intro to Yosys and synthesis"]
        A4 --> A5["Lab 3: synthesize good_mux"]
    end
    subgraph M2["Module 2"]
        B1["Sky130 .lib walkthrough"] --> B2["Hierarchical vs flat synthesis"]
        B2 --> B3["Flip-flops and reset styles"]
        B3 --> B4["Lab 4: flip-flop simulation + synthesis"]
    end
    A5 --> B1
    B4 --> Z["Foundation for optimization,<br/>GLS and physical design"]
```

## Complete flow learned so far

```mermaid
flowchart LR
    RTL["Write RTL"] --> SIM["Simulate<br/>iverilog + GTKWave"]
    SIM --> SYN["Synthesize<br/>Yosys + Sky130 .lib"]
    SYN --> NET["Gate-level netlist"]
    NET --> INS["Inspect with show<br/>and write_verilog"]
    INS --> VER["Verify with<br/>same test bench"]
```

## Tools and commands cheat sheet

| Purpose | Command |
|---|---|
| Compile | `iverilog design.v tb.v` |
| Run simulation | `./a.out` or `vvp file.out` |
| View waveform | `gtkwave file.vcd` |
| Load library | `read_liberty -lib <lib>` |
| Read RTL | `read_verilog <file>` |
| Synthesize | `synth -top <module>` |
| Flatten | `synth -top <module> -flatten` |
| Technology map | `abc -liberty <lib>` |
| View schematic | `show` |
| Write netlist | `write_verilog <file>` |
