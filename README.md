# Workshop-RTL-Design-using-Verilog-with-Sky130Tech
A report on 5 day workshop on RTL design and synthesis using opensource tools - iverilog, gtkwave, yosys and sky130-130nm technology PDK and standard cell library.
# Table of contents
 - [Day-1- Introduction to Verilog RTL Design and Synthesis](#1-Introduction-to-Verilog-RTL-Design-and-Synthesis)
    - [1.1 Introduction](#11-Introduction)
    - [1.2 Introduction to iverilog and gtkwave based simulation](#12-Introduction-to-iveriilog-and-gtkwave-based-simulation)
    - [1.3 Introduction to Yosys and Logic Synthesizer](#13-Introduction-to-Yosys-and-Logic-Synthesizer)
  - [Day-2- Timing Libraries, Hierarchical and Flat Synthesis](#2-Timing-Libraries-Hierarchical-and-Flat-Synthesis)
    - [2.1 Introduction to Timing Libraries](#21-Introduction-to-Timing-Libraries)
    - [2.2 Hierarchical Synthesis and Flat Synthesis](#22-Hierarchical-Synthesis-and-Flat-Synthesis)
    - [2.3 Various Flip Flop coding Styles and Optimizations](#23-Various-Flip-Flop-coding-Styles-and-Optimizations)

# 1. Introduction to Verilog RTL Design and Synthesis
## 1.1 Introduction
### What is a design?
Design is the actual verilog code or set of verilog codes which has the intended functionality to meet the required specifications.
### What is a testbench?
Testbench is the setup to apply stimulus to the design to check for its functionality
### Simulator and its working
Simulator is a tool used for checking a verilog design. RTL design is basically the implementation of the design specifications. RTL design is checked for adherence to the specifications by simulating the design. Simulator looks for changes on the input signals. Upon any chaange to the input, output is evaluated. If there are no changes in the input, simulator will not evaluate the output. 
<p align="center">
  <img width=""1000 height="400" src="/Images/Pic1.png">
</p><br>
Here, design will be instantiated  in the testbench so that there is a mechanism to apply the stimulus. Design may have one or more primary inputs and one or more primary outputs. Whereas the testbench has no primary inputs and primary outputs

## 1.2 Introduction to iveriilog and gtkwave based simulation
Create a folder VLSI workshop. Move into this folder and clone the git repository https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git.
```
cd VLSI
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop
```
Here, a folder called `my_lib` contains to subfolders - `lib` and `verilog_model`. `lib` contains the sky130 stdcell library which is used for synthesis. `verilog_model` contains standard cell verilog models od the cells present in the `lib` file. `verilog_files` contains all the lab experiments needed for the workshop.<br><br>
Iverilog takes the design and testbench as the input and generates a `vcd` file as the output. This `vcd` file stands for value change dump which can be opened with `gtkwave`. 
<p align="center">
  <img width=""1000 height="400" src="/Images/Pic2.png">
</p><br>
As an example, goodmux.v is simulated with the testbench tb_goodmux.v.

```
cd verilog_files
iverilog good_mux.v tb_good_mux.v
./a.out
gtkwave tb_good_mux.vcd
```

<p align="center">
  <img width=""1000 height="250" src="/Images/Pic3.png">
</p><br>

## 1.3 Introduction to Yosys and Logic Synthesizer
### What is a synthesizer?
Synthesizer is a tool to convert RTL to netlist. Basically, RTL to gatelevel translation is called as synthesis. `read_verilog` is the command used to read the verilog design, `read_liberty` is the command used to read the standard cell library (.lib) and `write_verilog` is used to write the netlist. Basically, netlist is the representation of the design in terms of standard cells present in the .lib file.
<p align="center">
  <img width=""1000 height="425" src="/Images/Pic4.png">
</p><br>
To verify the generated netlist, iverilog simulator is used. <br>
<p align="center">
  <img width=""1000 height="350" src="/Images/Pic5.png">
</p><br>

### Why different flavours of gates?
The .lib file is a collection of logical modules. Any digital design can be implemented with the cells present in the stand cell library file. It includes logic gates like AND, OR, NOT, NAND, D_FFs, ....... etc. Also, it includes different flavours of the same gates such as 2-ip AND gate, 3-ip AND gate, 4-ip AND gate etc with different variations in speed, power and area. Yosys is the synthesizer used in this course. 

Combinational delay in logic patth determines the maximum speed of operation of digital logic circuit. So we need cells that work fast to decrease the Tclk. Basically, the faster cells have wider mosfets to increase their current driving capacities. 
<p align="center">
  <img width=""1000 height="200" src="/Images/Pic6.png">
</p><br>
But faster cells have wider mosfets, hence they 

- require more area (due to wider mosfets) and power (due to larger currents)
- May cause hold time violations

So, slower cells are also required to meet the hold time constraints. So, guidance is offered to the synthesizer to select the cells appropriately in the form of constraints.
As an example, good_mux.v is synthesized. Go to the file verilog_files and type `yosys`. This invokes yosys.
```
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog good_mux.v
synth -top good_mux
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr good_mux_synth.v
```

The generated circuit and the netlist are shown below. 
<p align="center">
  <img src="/Images/Pic7.png">
</p>
<p align="center">
  <img src="/Images/Pic8.png">
</p><br>

# 2. Timing Libraries, Hierarchical and Flat Synthesis
## 2.1 Introduction to Timing Libraries
When we look into any library, three important factrors are 
 - Process: Signifies some variation due to fabrication
 - Voltage: Voltage of operation
 - Temperature: Ideal temperature of Silicon
<p align="center">
  <img src="/Images/Pic9.png">
</p><br>

Other impoertant information is mentioned in the library, such as technology, delay models, units of power, delay, resistance, capacitance, etc. <br>
<p align="center">
  <img src="/Images/Pic10.png">
</p><br>
Also, operating conditions of the design is mentioned. <br>
<p align="center">
  <img src="/Images/Pic11.png">
</p><br>
Keyword cell is used to define the keyword. As discussed earlier, there are different flavours of the same gate with varying area, power and delays. As an example, 2-ip AND gate is shown. Since 2-ip AND gate has 2 inputs, there are 4 possible input combinations. Power consumption of all the possible input combinations is mentioned in the `.lib` file. <br><br>
<p align="center">
  <img src="/Images/Pic12.png">
</p><br>

## 2.2 Hierarchical Synthesis and Flat Synthesis
When we perform hierarchical synthesis using the folloing commands,
```
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr multiple_modules_synth.v
```
We get the following schematic. 
<p align="center">
  <img src="/Images/Pic13.png">
</p><br>

Whereas, flattened netlist can be generated using the keyword `flatten`. The commands are as follows:
```
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
show
write_verilog -noattr multiple_modules_synth.v
```
We get the following schematic. 
<p align="center">
  <img src="/Images/Pic14.png">
</p><br>

## We can perform module level synthesis as well. But why do this?
There are two circumstances when we do this 
 - Module level synthesis is preferred when we have multiple instances of the same module. For example, if a design uses n multipliers where n is a large number, then, we synthesise the multiplier once and use it multiple times.
 - This approach is used for divide and conquer. If the design is too massive for the synthesis tool, we synthesis mudule by module and stitch them to get the entire design. 

## 2.3 Various Flip Flop coding Styles and Optimizations


