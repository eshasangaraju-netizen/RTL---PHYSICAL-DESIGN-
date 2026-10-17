# Module4

 GLS, blocking vs non-blocking and Synthesis-Simulation mismatch

 
1. Blocking and Non-Blocking Assignments

Verilog has two types of assignments:

     * Blocking (=): Executes statements one after another, similar to normal programming. It is mainly used for combinational logic.
     * Non-blocking (<=): Updates signals together at the end of the time step. It is mainly used for sequential circuits such as            flip-flops and registers.

Using the wrong type can make the simulation behave differently from the actual hardware.

2. Sensitivity List

A sensitivity list tells the simulator when an always block should run.

        For example:

        always @(sel)

        This block runs only when sel changes. But if the output also depends on i0 and i1, changes in those signals may not be                         detected.

         Using:  always @(*)

         -->makes the block respond to all the inputs used inside it. This is commonly used for combinational circuits.

3. Simulation and Synthesis

        -> Simulation is used to check how the Verilog code behaves before building the hardware.

        -> Synthesis converts the Verilog code into actual hardware elements such as gates, multiplexers, and flip-flops.

        -> Ideally, the behavior seen in simulation should match the behavior of the synthesized hardware.

4. Simulation-Synthesis Mismatch

Sometimes the simulator gives one result, but the synthesized hardware behaves differently. This is called a simulation-synthesis mismatch.

It can happen because of:

     * Incorrect blocking or non-blocking assignments
     * Incomplete sensitivity lists
     * Improper if or case statements
     * Incorrect or non-standard coding styles



5. Gate-Level Simulation (GLS)

        After synthesis, the design is converted into a gate-level netlist. GLS is then used to simulate this netlist.

        It helps check whether the synthesized hardware still works correctly and can identify possible logic and timing problems before the            final hardware is implemented.

6. Tools

       Icarus Verilog can be used to compile and simulate the Verilog design.

       GTKWave can be used to view the waveforms and understand how signals change with time.



LABWORK--> 

1) ternary_operator_mux.v:


PROGRAM:

<img width="1680" height="893" alt="mod4prg" src="https://github.com/user-attachments/assets/c068ef9a-2194-44c7-9dda-1d5ccab03ebb" />

     Explanation:

      This is a 2:1 multiplexer implemented using the ternary operator.

      If sel = 1, output y = i1.

      If sel = 0, output y = i0.

    Output behavior:

      When sel=0, y follows input i0.

      When sel=1, y follows input i1.

GTKWAVE:


<img width="1680" height="893" alt="tergtk" src="https://github.com/user-attachments/assets/ccf9b719-18d3-4aa7-abcd-def828a5442a" />

SYNTHESIS:


     Yosys Synthesis Commands:

        # 1. Load the standard cell library
       read_liberty -lib ../lib/sky130_fd_sc_hd_tt_025C_1v80.lib

        # 2. Read the Verilog design file
       read_verilog dff_const1.v

       # 3. Run synthesis on the top module
       synth -top dff_const1

       # 4. Map flip‑flops to library cells
       dfflibmap -liberty ../lib/sky130_fd_sc_hd_tt_025C_1v80.lib

        # 5. Technology mapping using ABC
       abc -liberty ../lib/sky130_fd_sc_hd_tt_025C_1v80.lib

        # 6. Visualize the synthesized design
        show

<img width="1680" height="893" alt="ternarymucsyn" src="https://github.com/user-attachments/assets/2b861257-9b5c-48ee-9ce4-40e924014b07" />

2) blocking_caveat.v:

   PROGRAM:
   

    <img width="1680" height="893" alt="BLCPRG" src="https://github.com/user-attachments/assets/f6794e3a-b4eb-4317-8731-2c4fd9ef0f76" />


       Explanation:

          This module demonstrates a blocking assignment caveat in Verilog.

          Inside the always @(*) block:

          First, d = x & c; is executed.

          Then, x = a | b; is executed.

          Because blocking assignments (=) execute in order, d is computed using the            old value of x (before it is updated).

          This can cause unexpected results compared to what you might logically               expect (d should depend on the new x).

        Key Point:

         Blocking assignments (=) happen sequentially.

         Non‑blocking assignments (<=) update simultaneously at the end of the time            step.

2)GTKWAVE:

<img width="1680" height="893" alt="BLCGTK" src="https://github.com/user-attachments/assets/bf9f3895-098e-4e30-8c8f-add7de94f3e6" />


3)SYNTHESIS:


<img width="1612" height="614" alt="Screenshot 2026-08-22 223751" src="https://github.com/user-attachments/assets/dcf0e27c-f2ea-4ce8-b641-a4e7aa902027" />

KEY TAKEAWAYS:

     The main idea is that Verilog should be written carefully so that the simulation result matches the actual synthesized hardware. Using
     
     blocking and non-blocking assignments correctly, maintaining proper sensitivity lists, and performing Gate-Level Simulation helps ensure  
     
     that the final hardware works as expected.




    

















  


