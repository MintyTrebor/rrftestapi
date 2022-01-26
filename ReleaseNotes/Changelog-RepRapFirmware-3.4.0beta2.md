RepRapFirmware 3.4.0beta2
Upgrade notes:

Where a GCode parameter accepts multiple values, you can no longer use a mixture of literals and expressions (which was previously supported in standalone mode only). However, the whole parameter can now be an array expression. For example, M92 E{var.e1}:400:{var.e3} will no longer work and must be replaced by M92 E{var.e1,400,var.e3}.
The H parameter of the M0 and M1 commands has been removed. Heaters will always be turned off when these commands are executed; except that if M1 is used to cancel a print that is paused and file cancel.g exists, cancel.g will be run and heaters will remain on unless cancel.g turned them off. Previously, M0 H1 or M1 H1 would leave heaters turned on.
New features and behaviour changes:

Input shaping types ZVD, ZVDD, EI2 and EI3 are supported, except in the Duet Maestro build. See the M593 command in the Duet3D GCodes wiki page. DAA is no longer supported except in the Duet Maestro build.
A floating point expression can now be used where a driver ID is expected, provided that the value of the expression is close to a whole number plus a whole number of tenths
Array expressions are now supported in GCode parameters that expect multiple values, both in standalone and SBC modes using syntax { expression, expression, ... }. For example: M92 E{var.e1,var.e2}
Files with extension .nc are now assumed to be GCode files and parsed for all the usual parameters
Improved the speed of display refresh on 12864 displays with ST7567 controllers
LIS3DSH accelerometers are now supported in addition to LIS3DH (for accelerometers connected to SAMMYC21 boards, needs new SAMMYC21 firmware too)
The M956 command now accepts an optional F (filename) parameter
G10 L2 and G10 L20 commands now accept the P0 parameter, meaning use the current coordinate system (LinuxCNC extension)
Values of type DateTime can now be compared using equality and comparison operators
The F parameter of the M563 command can now be F-1 meaning there are no print cooling fans for this tool
New state "Cancelling" (meaning that the print has been cancelled and cancel.g is executing) is reported in the object model and displayed by DWC
[Duet with attached SBC] Resume after power fail and resume after pause/power down are now supported
[Duet 3 + expansion/tool boards] Improved the reporting of clock jitter on CAN-connected expansion boards
[Duet 3 + expansion/tool boards] Implemented M569.2 for CAN-connected drives (needs new expansion/tool board firmware too)
Bug fixes:

If the slicer uses the M486 command to label objects in the GCode file, and there were more than 20 (Duet 2) or 40 (Duet 3) objects, then RRF crashed when printing the file
If you homed any axes or changed tools when workplace coordinates are in use, then any absolute moves in the homing or tool change files had workplace coordinate offsets applied; whereas they should not
When M955 reported the configuration of an accelerometer that is connected to the main board using SPI, the orientation was always reported as 20 even if a different orientation had been selected by means of the M955 I parameter
The subtraction operator between two values of type DateTime errored out
When using the & operator in conditional GCode, if the first operand was false then an error might be reported and the macro aborted when parsing the second operand. Similarly if the first operand of | was true.
[Duet 3 + expansion/tool boards] If a "stuck in spin loop" error occurred then RRF tried to turn off all heaters. This led to an assertion error if any of the configured heaters was connected to an expansion or tool board, so the details of the original error were not recorded in the software reset data.
[Duet 3 + expansion/tool boards] M569 and M569.1 did not wait for movement to stop when the P parameter was a remote driver
[Input shaping branch only] Several bugs related to input shaping have been fixed
Object model changes:

The value of job.build.currentObject may now be greater than the length of the job.build.objects array. This is because the size of job.build.objects is limited to conserve memory (to 20 on Duet 2 or 40 on Duet 3), whereas when M486 labelling is used up to 64 objects can be numbered and individually cancelled.
Added field tools[N].isRetracted
The Z offset of a Z probe is now reported as the negative of the trigger height instead of always being zero