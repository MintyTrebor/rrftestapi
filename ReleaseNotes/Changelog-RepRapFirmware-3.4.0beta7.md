Upgrading from a 3.4 beta before 3.4b6? Please see the full changelog for previous beta versions of RRF 3.4 here: https://github.com/Duet3D/RepRapFirmware/wiki/Changelog-RRF-3.x-Beta-&-RC
Upgrading from a previous stable version? Also see  the full changelog for previous stable versions of RRF here: https://github.com/Duet3D/RepRapFirmware/wiki/Changelog-RRF-3.x


RepRapFirmware 3.4.0beta7
=========================

Upgrade notes:
- The handling of filament errors have changed. When a filament error occurs, an event is created. To handle the event, RRF runs macro file filament_error.g without appending the extruder number to the file name and without pausing the print first. The extruder number is passed as param.D along with some other parameters. If filament_error.g is not found then the print is paused (running pause.g) and the error is reported.
- The handling of driver stalls has changed. In the M915 command there is no longer a distinction between R2 and R3; both cause an event to be created when the driver stalls. To handle the event, RRF calls driver_stall.g passing the stalled local driver number in param.D and the CAN address of the board concerned in param.B. File rehome.g is no longer used. If file driver_stall.g is not found then the print is paused without running pause.g and the error is reported.
- The handling of heater faults has changed. When a heater fault occurs, an event is created. To handle the event, RRF runs macro file heater_fault.g without pausing the print first. The heater number is passed as param.D along with some other parameters. If file heater_fault.g is not found then the print is paused and the user is notified. **Currently, RRF does not attempt to turn off power to the whole machine if the user does not respond to the heater fault.** We plan to reinstate this or a similar function in release 3.4.0RC1.
- [Duet 3 + tool/expansion boards] Both the main board and expansion board firmware must updated to version 3.4.0beta7, otherwise some functions may not work e.g. accelerometer data collection from tool boards.
- [Duet 3 + prototype EXP1HCL closed loop boards] The Duet3Firmware-EXP1HCL.bin file is for the version 1.0 production boards, which are not widely available yet. File Duet3Firmware-EXP1HCLv0_3.bin is for the version 0.3 prototypes. If you have a version 0.3 prototype board then before updating the firmware on that board you must delete or rename Duet3Firmware-EXP1HCL.bin, and rename Duet3Firmware-EXP1HCLv0_3.bin to Duet3Firmware-EXP1HCL.bin.

New features and behaviour changes:
- The heater model now includes non-Newtonian cooling to better predict the variation of cooling rate with temperature and the maximum temperature that would be reached at continuous full power
- The algorithm used to detect a heater fault while heating up now averages the heating rate over a longer period of time, so that noise in the reading is less likely to trigger a spurious heater fault
- Increased maximum extruders to 8 on Duet 3 Mini
- The generated resurrect.g file no longer includes a T-1 P0 command. This is to aid print recovery on tool changers.
- Collecting more than 65535 accelerometer samples in a single run is now permitted
- When an accelerometer run is started, a check is made that the interrupt signal from the accelerometer is not stuck high
- When using TMC2208/2209 drivers, the value of the chopper control register is now checked at frequent intervals against the value that should have been programmed
- When collecting data from EXP1HCL closed loop driver boards, some fields in the .csv file have been renamed to match changes to the plugin
- Build time comments in GCode files generates by REALvision slicer are now parsed
- Handling of filament errors has changed - see the upgrade notes
- Handling of driver stalls during printing has changed - see the upgrade notes
- Heater faults during printing now cause macro heater_fault.g in the system folder to be run it if exist, with the heater number passed in param.D. If it isn't found then the user is notified and the print is paused.
- Heater faults that occur on expansion boards are now reported to the main board and treated in the same way as local heater faults.
- Driver stalls, errors and warnings that occur on expansion boards are now reported to the main board and treated in the same way as local driver errors and warnings. System macro driver_stall.g, driver_error.g or driver_warning.g (as appropriate) is run, if it exists. The local driver number is passed as param.D, the CAN board address as param.B, the encoded driver status as param.P and the error or warning message as param.S.
- When a M291 S2 or S3 command is executed in a macro initiated from PanelDue, RRF now notifies PanelDue if the message is cleared from another source, for example from DWC. PanelDueFirmware will need to be updated to clear the message box when it receives this notification.

Object model changes:
- Fields gain, timeConstant and timeConstantFanOn of heaters[].model are removed. Fields coolingRate, coolingExp and fanCoolingRate are added. 
- Field boards[].maxMotors is no longer flagged 'verbose'
- A new flag 'important' has been added to some object model fields, notably state.messageBox and the fields it contains. The "i" flag in the object model request flag indicates that these fields should be returned event if they would not normally be returned (for example because the "v" flag is also used) and that these fields should be return even when they have null values. The main purpose of this is to notify PanelDue of these fields when the Aux GCode channel is unable to make object model requests because it is executing a macro.

Bug fixes:
- In some circumstances the machine state was still be reported as "Changing tool" after a tool change had completed. This was a new bug in 3.4.0beta6.
- Unpredictable behaviour might occur when using a large number of mesh probe points (more than 416 points for most Duets)
- Extrusion would stop when processing a move so tiny that it would take less than 0.7 microseconds to execute
- When evaluating an expression of the form "A & B", if A was false but a subexpression within B contained an array indexing operation in which the index had an undefined value, error message "expected integer expression" was produced and execution terminated. Similarly for expressions of the form "A | B" when A was true.
- When M572 was used to change pressure advance, DWC and DSF were not informed that this part of the object model had changed, so the object model displayed in the DWC object model browser showed the old value.
- If GCode file generated by Superslicer had an estimated build time of a day or more, RRF parsed the build time comment incorrectly
- Object model field move.kinematics.inverseMatrix contained values from the forward matrix instead of the inverse matrix
- The code to limit speed and acceleration for polar kinematics didn't always work correctly
- If M591 S# S1 was used to enable a filament monitor after printing has already started, the filament monitor data was reset. This could lead to a spurious "no data received" filament error.
- [Duet with PanelDue] If a macro file was initiated from PanelDue, and that file used a M291 S3 command followed later by a M291 S2 command followed later by another command that took some time to execute (e.g. another M291 S3 command) then the M291 S2 message was not displayed
- [Delta printers] Under some conditions, incorrect tower movement occurred (new bug in RRF 3.4beta series)
- [Polar printers] Fixes to polar kinematics
- [Duet 3 expansion and tool boards] If a Linear Analog sensor was configured on an expansion or tool board, the board crashed
- [Duet 3, 3 Mini] If M122 was run when the printer was halted, the machine reset and recorded a Hard Fault
- [Duet 2 SBC build] The aux port is now configured for a PanelDue at 57600 baud by default so that automatic test equipment can test the board when there is no SBC connected
- [Duet 2 SBC build] Height maps were not saved to file in 3.4.0beta6
- [Duet 3 Mini, tool board] On a small number of systems it has been observed that a TMC2209 driver would occasionally increase motor current unexpectedly. We don't believe this was a software issue. However, the software now checks the chopper control register frequently; and if it is found to be wrong then the firmware corrects is and records a "cc" error. The count of these errors can be seen in the M122 diagnostics report for the driver.
- [Duet + SBC] SBC file requests were not fully invalidated
- [Duet 3 MB6HC + Hangprinter with ODrives] Fixes to the ODriver CAN interface
- [Duet 3 using MB6HC or Duet 3 Mini as expansion board] Main boards used as expansion boards did not announce themselves to the master main board, so they did not appears in the 'boards' part of the object model