# SPI-Slave-design-using-DC-shell


### WHAT IS VLSI?
 
Vlsi stands for “very large scal integration” . it is the the proces of creating the integrated circuit by combining of 1000’s of transistors on a single chip.
Moore's law is the observation that the number of transistors in an integrated circuit (IC) doubles about every two years.

![flow](https://github.com/Devi-charan-29/SPI-Slave-design-using-DC-shell/assets/95524221/2d61074f-8b67-4416-b8ca-a3ebcd2ef631)

Fig:1 – vlsi flow and physical design flow


### WHAT IS SYNTHESIS:
Synthesis is the process where the technology independent RTL code that is converted into technology dependent gate level netlist , where meeting the time,area and power.


![synthesis](https://github.com/Devi-charan-29/SPI-Slave-design-using-DC-shell/assets/95524221/7e03d979-0227-400a-8033-20c7879aa0d3)

Fig:2 –synthesis flow

TYPES OF SYNTHESIS:
•	Logical aware synthesis
•	Physical aware synthesis



DC SHELL :
The dc_shell is the original format that is based on Synopsys's own language while dc_shell-t uses the standard Tcl language. Command to enter the dc shell.
Command 01	dc_shell –output sspi.log
 
Log file :
The log file saves all commands we enter and the results of the command we entered. We can also open the file further to see.
WHY WE GIVE SEARCH PATH? IF WE WONT GIVE WHAT HAPPENS :
Search path is given to extract the files from the provided path. If we don’t provid search path we need to give again in the link library ,target library and analyz (RTL file) full path for file.
Command 02	set search_path {/proj1/dataIn/libs/ /proj1/}
WHAT IS LINK LIBRARY? WHY WE ARE GIVE “*” BEFORE THE .DB FILE :
Link library is the predefind variable , that we are linking the file with the instance lib file. It also extract the default files by using “*” before the .db file ( like Gtech library , symbol library , design ware library.)
Command 03 set link_library {* tcbn28hpcplusbwp40hvtssg072vm40c.db}
WHAT HAPPENS IF WE WONT GIVE LINK LIBRARY? AND HOW MANY FILES WE CAN GIVE IN LINK LIBRARY:
If we won’t provide the link library it will analyze and eloborate and throws the warning. After the compile stage we see many warnings one for the each cell. We can give files in link library for .db file ,macros in the link library.
WHAT IS TARGET LIBRARY ? WHAT HAPPENS IF WE WONT GIVE:
Target library contains the standard cells information for the particular technology we are working on. We have to give the target library as we are specified for the technology of the design that we are working on.
Command 04 set target_library { tcbn28hpcplusbwp40hvtssg072vm40c.db}
 
ANALYZE:
Analyze command loads the files and check the syntax errors line by line. And it throws the errors if, after checking is done at the end.
Command 05 analyze –format verilog sspi.v

ELABORATE:
It converts the verilog code to GTECH(gates). Simply it builds the desing on specified parameters.
Command 06 elaborate spi_slave
READ VERILOG:
Read verilog do the task that which analyze and elaborate do. It also check line by line but it stops at line where error found and throws error, and will not proceed for the next line.
Command 07  read_verilog sspi.v



FLOW WITH SEARCH PATH AND WITHOUT SEARCH PATH:
	WITH SEARCH PATH :
set search_path {/proj1/dataIn/libs/ /proj1/}
set link_library {* tcbn28hpcplusbwp40hvtssg072vm40c.db} set target_library { tcbn28hpcplusbwp40hvtssg072vm40c.db} analyze –format verilog spi_slave.v
elaborate counter

	WITHOUT SEARCH PATH:
set link_library {* /proj1/dataIn/libs/ tcbn28hpcplusbwp40hvtssg072vm40c.db} set target_library {/proj1/dataIn/libs/ tcbn28hpcplusbwp40hvtssg072vm40c.db} analyze –format verilog /proj1/spi_slave.v
elaborate counter
 
START GUI:
By this command we can see the block leve schematic circuit of netlist after the elaborate.
Command 08 start gui

 
STOP GUI :
It is used to close the window of the block level schematic circuit of netlist.
Command 09 stop gui
COMPILE:
By this command the various obtimisation is done. For meeting the power, performance , area.
Command 10 compile

 
 
 
START GUI:
After the compile by using this command we can see the gate level schematic circuit of netlist.
Command 11 start gui

 
STOP GUI:
It closes the gate level schematic circuit of netlist window.
Command 12 stop gui
 
TCL SCRIPT FOR CLOCK PORT:
This the script to know the clock port name. It is used while creating the clock.
Script:
foreach a [get_object_name [all_registers –clock_pins]]{
set b [get_object_name [all_fanin –to $a –flat –startpoints_only]] if {[llength $b] == 1 }{
foreach c [get_object_name [get_ports [all_inputs]]] { if {$c == $b} {
lappend d $b
}}}}
puts[ lsort –u $d]
CHECK TIMING:
Here the clock is checked wether it is created or not. If it is not there we will see the error , so we should create the clock based on the requirement.
Command 13 check_timing
CREATE CLOCK:
By creating the clock , so the tool will understand the clock period and the frequency. Based on the clock only everything is checked.
Command 14 create_clock –name clk –period 5 [get_ports SCK]
INPUT AND OUTPUT DELAYS:
We don’t know the excat time that we receive data from another block output to our block input , so in the worst case we take 60% of the clock period for input and same 60% of clock period for output in worst case. For data from our block ouput to next block input.
Command 15 set_input_delay 1.2 –clock clk [get_ports [remove_from_collection [all_inputs] SCK]]
Command 16	set_input_delay 1.2 –clock clk [get_ports [all_outputs ]]
 
DRIVING CELL AND LOAD INFORMATION:
•	We will set the driving cell to all the input ports except clock port.Driving cell maintains the signal strength from other block output to our block input.Were the input is driven from the driving cell.
•	Load keeps the signal strength strong from our block output to next block input.We provide the output (Load) through this cell to another Block.
Command 17 set_driving_cell -lib_cell BUFFD4BWP40P140HVT [get_ports [remove_from_collection [all_outputs] SCK]]
Command 18 set_load 0.00146193 [get_ports [all_outputs]]
CLOCK UNCERTAINITIES:
Clock uncertainity we take 20% of clock period for setup in the worst case and 15% of clock period for hold in worst case.
Command 19 set_clock_uncertainty -setup 1 [get_clocks SCK]
Command 20 set_clock_uncertainty -hold 0.75 [get_clocks SCK]
CREATING GROUP PATHS:
Here the group paths will be created , so that we can check the timing report of each group for the meeting of the slack
Command 21   group_path -name i2r -from [all_inputs] -to [all_registers -data_pins]
Command 22   group_path -name r2r -from [all_registers -clock_pins] -to [all_registers -data_pins]
Command 23	group_path -name r2o -from [all_registers -clock_pins] -to [all_outputs]
Command 24   group_path -name i2o -from [all_inputs] -to [all_outputs]

REPORT TIMING:
Based on the group paths generated we will can see the timing report of the groups . for meeting the slack.
Command 25   report_timing

 
 
 

 

 
 
CREATING VIRTUAL CLOCK :
Virtual clock is created to meet the i2o group path. The virtual clock is given for the path i2o because the data from the previous block was with the reference with the clock so the i2o has only combinational block so its has no clock so we are checking with the reference of clock of the i2o slack. So we have to create the virtual clock for the i2o path.

 

 
REPORT TIMING (after the virtual clock):
Here the path i2o will meet. After creating the virtual clock.

	Here we do again give set input and output delays by decreasing the delay time period to meet the slack for the path of i2r and r2o. generally project guid has the information of these delay information particularally.
	Here the group path i2r and r2o have met the slack after decreasing the input and output delays.

 
 
Check Design.
Check design checks the internal representation of the current design for issues and throws the error if any.
Command 26 check_design


REPORT QOR:
This command reports the timing path group ,cell count details , current design statitics such as combinational, noncombinational, and total area and etc.
Command 27 report_qor

 
 
 

 
REPORT POWER:
This command calculates the power and gives the reports of the design.
Command 28 report_power

 

 
 
REPORT AREA:
This command calculates the area and gives the reports of the design.
Command 29 report_area


Command	30	write_file -format verilog -hierarchy -output sppi.v
(it save the netlis in verilog , by extending the file name by .v )

Command 31 write_sdc sppi.sdc
(All SDC commands saved in the file with the extention the file name by .sdc)

DDC FILE:
In the .ddc file the whole design of sdc and netlist overall will be saved . where the ddc file will be readable by the toll.
Command 32 write_file -format ddc -hierarchy –output sppi.ddc

After creating the ddc file we can execute or run the project by call the ddc file , after the seting the search path ,link libray and target library. By entering this command.
Command 32 read_ddc sppi.ddc
 
HISTORY:
History command is used to see the history of commands that you have entered.

 
 
