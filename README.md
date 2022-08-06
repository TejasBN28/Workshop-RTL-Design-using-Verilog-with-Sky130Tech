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
    - [2.4 Interesting Optimizations](#24-Interesting-Optimizations)
  - [Day-3- Combinational and Sequential Optimization](#3-Combinational-and-Sequential-Optimization)
    - [3.1 Introduction to Optimizations](#31-Introduction-to-Optimizations)
    - [3.2 Combinational Logic Optimizations](#32-Combinational-Logic-Optimizations)
    - [3.3 Sequential Logic Optimizations](#33-Sequential-Logic-Optimizations)
    - [3.4 Sequential Optimization for unused outputs](#34-Sequential-Optimization-for-unused-outputs)
  - [Day-4- GLS, Blocking vs Nonblocking and Synthesis-Simulation Mismatch](#4-GLS-Blocking-vs-Nonblocking-and-Synthesis-Simulation-Mismatch)
  
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
In a combinational circuit, when the inputs are altered, the output changes after the circuit's propagation delay. If several pathways with differing propagation delays are used during data propagation, there may be a probability of an output glitch. Because more glitches arise when there are more combinational circuits in the design, the output becomes unstable. We are opting for flops to store the data from the communication lines in order to address this issue. When a flop is utilised, the output of the combinational circuit is stored in it and transmitted only at the positive or negative edge of the clock so that the subsequent combinational circuit receives a glitch-free input, stabilising the output. The set and reset control pins of a flop are used as initialization signals because without them, the following combinational circuit would get a trash value. Both synchronous and asynchronous control pins are possible.

### D Flip Flop with Asynchronous Reset
```
module dff_asyncres ( input clk ,  input async_reset , input d , output reg q );
always @ (posedge clk , posedge async_reset)
begin
	if(async_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
<p align="center">
  <img src="/Images/Pic15.png">
</p><br>

<p align="center">
  <img src="/Images/Pic16.png">
</p><br>

### D Flip Flop with Asynchronous Set
```
module dff_async_set ( input clk ,  input async_set , input d , output reg q );
always @ (posedge clk , posedge async_set)
begin
	if(async_set)
		q <= 1'b1;
	else	
		q <= d;
end
endmodule
```
<p align="center">
  <img src="/Images/Pic17.png">
</p><br>

<p align="center">
  <img src="/Images/Pic18.png">
</p><br>

### D Flip Flop with Synchronous Reset
```
module dff_syncres ( input clk , input async_reset , input sync_reset , input d , output reg q );
always @ (posedge clk )
begin
	if (sync_reset)
		q <= 1'b0;
	else	
		q <= d;
end
endmodule
```
<p align="center">
  <img src="/Images/Pic19.png">
</p><br>

<p align="center">
  <img src="/Images/Pic20.png">
</p><br>


## 2.4 Interesting Optimizations
In this lab, we consider the implementation of multipliers.
```
module mul2 (input [2:0] a, output [3:0] y);
	assign y = a * 2;
endmodule
```
Here, yosys implements this design just by left shift operator.
<p align="center">
  <img src="/Images/Pic21.png">
</p><br>

Similarly, 
```
module mult8 (input [2:0] a , output [5:0] y);
	assign y = a * 9;
endmodule
```
Here, this multiplier is implemented as `a*8 + a*1`. Since a is 3-bit, it `a*9 = aa`
<p align="center">
  <img src="/Images/Pic22.png">
</p><br>

# 3. Combinational and Sequential Optimization
## 3.1 Introduction to Optimizations
### Combinational Logic Optimizations
Combinational logic optimizations helps in squeezing the logic to get the most optimized design. Also, optimized logic will be power and area efficient.
Techniques used for combinational logic optimization are 
 - Constant Propogation - direct optimization technique
 - Boolean Logic Optimization - KMap, Quinn Muclusky, etc

### Sequential Logic Optimizations
Sequential Logic Optimization techniques include
 - Basic - Sequential Constant Propogation
 - Advanced
    - State Optimization
    - Retiming
    - Sequential Logic Cloning (Floorplan aware synthesis)
   
## 3.2 Combinational Logic Optimizations
The command used to perform logic optimization of combinational logic is `opt_clean -purge`. Here we consider the example opt_check.v,opt_check2.v, opt_check3.v, opt_check4.v and multiple_module_opt.v.

### Example: opt_check.v
```
module opt_check (input a , input b , output y);
	assign y = a?b:0;
endmodule
```
The commands to run optimizations are
```
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge
abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr opt_check_synth.v
```
<p align="center">
  <img src="/Images/Pic23.png">
</p><br>

### Example: opt_check2.v
```
module opt_check2 (input a , input b , output y);
	assign y = a?1:b;
endmodule
```
<p align="center">
  <img src="/Images/Pic24.png">
</p><br>

### Example: opt_check3.v
```
module opt_check3 (input a , input b, input c , output y);
	assign y = a?(c?b:0):0;
endmodule
```
<p align="center">
  <img src="/Images/Pic25.png">
</p><br>


### Example: opt_check4.v
```
module opt_check4 (input a , input b , input c , output y);
 assign y = a?(b?(a & c ):c):(!c);
 endmodule
```
<p align="center">
  <img src="/Images/Pic26.png">
</p><br>

## 3.3 Sequential Logic Optimizations
### Example-1
```
module dff_const1(input clk, input reset, output reg q);
	always @(posedge clk, posedge reset)
	begin
		if(reset)
			q <= 1'b0;
		else
			q <= 1'b1;
	end
endmodule
```
<p align="center">
  <img src="/Images/Pic27.png">
</p><br>
<p align="center">
  <img src="/Images/Pic28.png">
</p><br>

### Example-2
```
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
	if(reset)
		q <= 1'b1;
	else
		q <= 1'b1;
end

endmodule
```
Here, since the output is always 1, just a buffer is sufficient. No need of flipflop.
<p align="center">
  <img src="/Images/Pic29.png">
</p><br>


### Example-3
```
module dff_const3(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```
<p align="center">
  <img src="/Images/Pic30.png">
</p><br>
<p align="center">
  <img src="/Images/Pic31.png">
</p><br>

### Example-4
```
module dff_const4(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b1;
		q1 <= 1'b1;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```
Here, since the output is always 1, just a buffer is sufficient. No need of flipflop.
<p align="center">
  <img src="/Images/Pic32.png">
</p><br>
<p align="center">
  <img src="/Images/Pic33.png">
</p><br>

### Example-5
```
module dff_const5(input clk, input reset, output reg q);
reg q1;

always @(posedge clk, posedge reset)
begin
	if(reset)
	begin
		q <= 1'b0;
		q1 <= 1'b0;
	end
	else
	begin
		q1 <= 1'b1;
		q <= q1;
	end
end

endmodule
```
Here, no optimizations are possible
<p align="center">
  <img src="/Images/Pic34.png">
</p><br>
<p align="center">
  <img src="/Images/Pic35.png">
</p><br>

## 3.4 Sequential Optimization for unused outputs
### Example 1
```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = count[0];

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```
Here the output of the module is just sensing the output q as count[0] Basically q toggles on every clock cycle. count[1] and count[2] have no functionality. So one d_ff and one inverter will realize the circuit.

<p align="center">
  <img src="/Images/Pic36.png">
</p><br>

### Example 2
```
module counter_opt (input clk , input reset , output q);
reg [2:0] count;
assign q = (count[2:0] == 3'b100);

always @(posedge clk ,posedge reset)
begin
	if(reset)
		count <= 3'b000;
	else
		count <= count + 1;
end

endmodule
```
Here, all the three flip flops are inferred.

<p align="center">
  <img src="/Images/Pic37.png">
</p><br>

We can conclude that all the logicthat does not have imact in determining the primary output of the design, all of the those logics will be eliminated.

# 4. GLS, Blocking vs Nonblocking and Synthesis-Simulation Mismatch
## 4.1 GLS Concepts And Flow 
### What is GLS?
GLS stands for gate level simulation. When we write the RTL code, we test it by giving it some stimulus through the testbench and check it for the desired specifications. Similarly, we run the netlist as the design under test (dut) with the same testbench. 
## Why GLS?
Gate level simulation is done to verify the logical correctness of the design after synthesis. Also, it ensures the timing of the design. 

<p align="center">
  <img src="/Images/Pic38.png">
</p><br>

## 4.2 Synthesis Simulation Mismatch
The possible reasons for Synthesis Simulation Mismatches are 
 - Missing Sensitivity List
 - Blocking and Nonblocking Assignments
 - Non-standard Verilog Coding Practices

### Missing Sensitivity List
The simulator works based on the activity, i.e., output will change only when input changes. 
