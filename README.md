# Workshop-RTL-Design-using-Verilog-with-Sky130Tech
A report on 5 day workshop on RTL design and synthesis using opensource tools - iverilog, gtkwave, yosys and sky130-130nm technology PDK and standard cell library.
# Table of contents
 - [Day-1- Introduction to Verilog RTL Design and Synthesis](#1-Introduction-to-Verilog-RTL-Design-and-Synthesis)


# 1. Introduction to Verilog RTL Design and Synthesis
## 1.1 Introduction to iVeriilog Design and Testbench
### What is a design?
Design is the actual verilog code or set of verilog codes which has the intended functionality to meet the required specifications.
### What is a testbench?
Testbench is the setup to apply stimulus to the design to check for its functionality
### Simulator and its working
Simulator is a tool used for checking a verilog design. RTL design is basically the implementation of the design specifications. RTL design is checked for adherence to the specifications by simulating the design. Simulator looks for changes on the input signals. Upon any chaange to the input, output is evaluated. If there are no changes in the input, simulator will not evaluate the output. 
<p align="center">
  <img width="350" height="200" src="/images/pic1.png">
</p><br>
