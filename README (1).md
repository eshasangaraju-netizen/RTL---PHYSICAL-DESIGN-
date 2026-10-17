
# MODULE 3

Module 3- Combinational and sequential optmizations


-> Digital Logic Optimisation:
    
   Combinational Logic Optimisation:

       To save area, power, and make circuits faster.

   Techniques:

  1)       Constant Propagation → If an input is fixed, the circuit shrinks.
           Example: 
                   𝑌= (𝐴𝐵+𝐶)’. With 𝐴=0, it simplifies to 𝑌=𝐶’.
           Result: A 6‑transistor circuit reduced to just 2.

  2)       Boolean Simplification → Use K‑Maps or Quine McCluskey.
           Example:
                   Complex expression: y = a ? (b ? c : (c ? a : 0)) : (!c)
                   --> By simplification we get, 
                                                 𝑌=𝐴⊕𝐶(an XOR gate).

-> Sequential Logic Optimisation:
   
   Sequential circuits (like flip‑flops) can also be optimised to make designs faster and simpler.

     Basic: Sequential Constant Propagation-->  
     Flip‑flops can hold on to fixed values (like always 0 or always 1), which helps cut down extra logic and keeps the design             simpler.
  Advanced Techniques:

     1)State Optimisation → Remove unused states from a design.


     2)Retiming → Rearrange flip‑flops to improve speed.
     Example: A circuit running at 200 MHz can be retimed to reach 500 MHz.


     3)Logic Cloning → Duplicate logic blocks so they’re placed more efficiently on the chip

--> Key Takeaways
        
        ~ Circuits can be simplified by spotting constants.

        ~ Boolean algebra can simplify complex equations.

        ~ Sequential tricks like retiming and cloning make designs faster.

        ~ Optimisation saves area, power, and transistor count — crucial for modern chip design.

 LabWork--> 
 
   Combinational Logic Optimisations 


1)opt_check.v execution:
  
  -->PROGRAM:

 <img width="1680" height="893" alt="optcheck1prg" src="https://github.com/user-attachments/assets/e3eb8979-6159-4ab9-8b56-2b51f51169e5" />

    Explanation:
    
      Uses the ternary operator (a ? b : 0).
      
      If a = 1, then y = b.
     
      If a = 0, then y = 0.
    
      This is equivalent to an AND gate (y = a & b).

    Output behavior:
    
      a=0 → y=0 (regardless of b).
      
      a=1 → y=b.
      
      Waveform shows y high only when both a and b are high.


  -->GTKWAVE:

 <img width="1680" height="893" alt="optcheck1gtk" src="https://github.com/user-attachments/assets/a91fbd61-0eca-4599-8e8c-95d02de791e9" />

 -->SYNTHESIS:

 <img width="1680" height="893" alt="optchecksyn" src="https://github.com/user-attachments/assets/0afe2c1c-08aa-44a8-b983-017b04c736ec" />



2)opt_check2.v execution:
  
  -->PROGRAM:

 <img width="1680" height="893" alt="optck2prg" src="https://github.com/user-attachments/assets/98111e0d-6077-41c2-be4f-c0d58d667f07" />

    Explanation:
    
      Uses the ternary operator (a ? 1 : b). 
      
      If a = 1, then y = 1.
      
      If a = 0, then y = b.
      
      This is equivalent to an OR gate (y = a | b).

    Output behavior:
    
      a=1 → y=1 (regardless of b).
      
      a=0 → y=b.
      
      Waveform shows y high whenever either a or b is high.


  -->GTKWAVE:

<img width="1680" height="893" alt="optck2gtk" src="https://github.com/user-attachments/assets/b6d8f312-06ec-46cb-b027-5fc011b155ba" />


 -->SYNTHESIS:
 
<img width="1680" height="893" alt="optchk2syn" src="https://github.com/user-attachments/assets/3fcff392-1f3c-4c81-81e1-087aa349e027" />



3)opt_check3.v execution:
  
  -->PROGRAM:

  <img width="1680" height="893" alt="optchk3prg" src="https://github.com/user-attachments/assets/e64cb54b-c8e4-4b5d-8441-7795b0873c1a" />

     Explanation:
 
      This module uses nested ternary operators.

      Logic breakdown:

      If a = 1, then check c.

      If c = 1, then y = b.

      If c = 0, then y = 0.

      If a = 0, then y = 0.

    Functionally, this is equivalent to:

         𝑦=𝑎⋅𝑐⋅𝑏



  -->GTKWAVE:


  <img width="1680" height="893" alt="optch3gtk" src="https://github.com/user-attachments/assets/2ab13dc4-32bd-44ce-8403-936101347c7a" />




 -->SYNTHESIS:
 
<img width="1680" height="893" alt="optchk3syn" src="https://github.com/user-attachments/assets/ff08c1db-3f16-4ad0-8a26-6415f7787200" />



Sequential Logic Optimisations


commands used:

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



     

      

1)dff_const1.v execution:

  
  -->PROGRAM:

  <img width="1680" height="893" alt="dffprg" src="https://github.com/user-attachments/assets/9efda28d-5aae-49b7-9069-ee4edaa27677" />



        Explanation:

           Clock (clk) toggles every 10 ns → 20 ns period.

           Reset (reset) toggles every 1547 ns.

           Simulation runs for 3000 ns.

       Behavior:

         At start, reset=1 → q=0.

         When reset goes low, q becomes 1 on the next rising clock edge.

         When reset goes high again, q is forced back to 0.
  



  -->GTKWAVE:





<img width="1680" height="893" alt="dffgtk" src="https://github.com/user-attachments/assets/0343be63-eb6f-40dc-88b4-27e586d128e8" />



 

 -->SYNTHESIS:



 <img width="1680" height="893" alt="dffc1syn" src="https://github.com/user-attachments/assets/71a2974c-f7d7-4f5a-80a6-bf0f97abf489" />


 SUMMARY:

Combinational Logic Optimisation:

         Simplifies Boolean expressions and reduces gate count to make circuits smaller, faster, and more power‑efficient.

Sequential Logic Optimisation:
 
            Improves flip‑flop‑based designs by removing unused states, retiming registers, or cloning logic to boost speed and efficiency.

 








    














     
