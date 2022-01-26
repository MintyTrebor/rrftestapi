RepRapFirmware 3.4.0beta4
Known issues:

[Duet + SBC] If a print is paused then in many cases the print will restart from the start of the file instead of from the paused position
New features and changed behaviour:

When upgrading the main board firmware, if the IAP file is not found in /firmware then RRF looks for it in /sys
M80 now has an optional C parameter allowing the PS_ON port and polarity to be changed
M98 with R parameter but no P parameter is supported, to control whether pausing is permitted while macros are executed
M955 when the accelerometer is connected to an expansion board now behaves the same as for accelerometers connected to the main board, i.e. it no longer reports the configuration when a configuration command completes successfully
In conditional GCode, the unary + operator can now be used to convert a value of type DateTime to a number of seconds from the datum.
In conditional GCode, function datetime(X) is added, to convert X (an integer, or a string with format yyyy-mm-ddThh:mm:ss) to type DateTime
M404 no longer supports the D (nozzle diameter) parameter
M309 (set/report heater feedforward) is supported
M569.2 is now implemented on CAN-connected expansion boards
All main board firmware files now include a version string which can be found via the vector table. Previoualy this was implemented in expansion board and bootloader firmware only.
When switching out of laser mode using M451 or M453 the port assigned to control the laser is released automatically
M290 (babystepping) is no longer permitted on an axis that has not been homed
M122 B# and M115 B# where # is the CAN address of a tool board now include the tool board hardware version to the extent that it is known
Object model changes:

state.atxPower is no longer flagged 'live'. It is present if at least one M80 or M81 command has been executed and the PS_ON port is valid.
state.atxPowerPort is added, present under the same conditions as atxPower
state.macroRestarted is added. Its value is normally false, but when resuming a print that was paused while the file input stream was executing a macro, it is true at the start of the macro, until a regular G, M or T command (but not meta command) is executed. Therefore it can be tested at the start of a macro, to determine whether the macro was restarted after a pause or a power failure.
tools[].feedForward is added, giving the feedforward coefficient for each heater
[ATE mode only] Magnetic filament monitors now provide additional fields: agc mag position
[ATE mode only] Laser filament monitors now provide additional fields: brightness position shutter
Bug fixes:

The step calculation for delta printers occasionally resulted in one microstep being taken in the wrong direction, resulting in leaning prints
Motor idle timeout did not work
M42 Jn C"nil" didn't release the port if it was on an expansion board (the fix is in the expansion board firmware)
M106 without parameters reported the requested speed (last S parameter) when the fan was thermostatic, even though the S parameter is ignored for thermostatic fans
M500 now automatically saves the G31 parameters if they were previously loaded from config-override.g or saved using M500 P31
M906, M913 and M913 updated the object model but did not increment the corresponding sequence number. This meant that the corresponding values in DWC and DSF were sometimes out of date.
Bed levelling (leadscrew adjustment) moves were delayed until a normal move command was executed
String parameters to macros were sometimes rendered invalid
When a line of received GCode included a line number and checksum, RRF ignored a bad checksum
When a GCode input channel was set up to require checksums (e.g. as is normal for a PanelDue port), if the checksum was missing then the line was executed in any case
Heater fault detection was too sensitive during the heating up phase
When running tuning on a prototype EXP1HCL closed loop board, the processor was stuck while waiting for a response, and if tuning took more than 20 seconds then the main board would reset with a "Stuck in spin loop" error
Internal changes:

When bed probing using G29, method IsReachable now sets the Z coordinate of the coordinates parameter to the probe starting height. Any other axes are set to the current coordinates. This is to aid kinematics classes for which IsReachable needs to know the Z coordinate, e.g. Hangprinter. Previously, the values of coordinates not included in the axes bitmap parameter was undefined.
[Duet + SBC] Filament handling is now done by RRF, not by the SBC