RepRapFirmware 3.3
Known issues:

If the slicer uses the M486 command to label objects in the GCode file, and there are more than 20 (Duet 2) or 40 (Duet 3) objects, then RRF crashes when printing the file. This bug was also present in release 3.2.x.
If you home any axes when workplace coordinates are in use, then any absolute moves in the homing files have workplace coordinate offsets applied; whereas they should not. This is a new bug in release 3.3.
When M955 reports the configuration of an accelerometer that is connected to the main board using SPI, the orientation is always reported as 20 even if a different orientation has been selected (and is being used) by means of the M955 I parameter.
If a M117 command is executed when there are pending moves in the movement queue, a "string too long" error message is generated and the command is not executed. Workaround: use M400 immediately before M117.
When a line of received GCode includes a line number and checksum, RRF ignores a bad checksum
[Duet 3 + CAN-connected expansion boards] If a sequence of short movements is commanded involving only motors connected to expansion boards, then the moves might be sent too quickly to the expansion boards, eventually resulting in lost moves.
Upgrade notes:

All extruders must be declared explicitly using M584. In previous firmware versions, one default extruder was assign to driver 3.
In previous versions, if a parameter letter should have been followed by a number but no digits were found, zero was assumed. Now RRF reports an error and abandons the command instead.
Firmware files are now stored in folder /firmware of the SD card instead of /sys
Unless you are running with an attached SBC, you must also upload the new Duetx_SDiap32_xxx.bin file for your board. Until you do, you will not be able to upgrade/downgrade from 3.3 firmware to other versions. The new IAP files have the same names as the previous IAP files for RRF3.2 and 3.2.2 and will also work with those versions of RRF.
Previously the command M106 R (with no P parameter and no value after R) would restore the last saved default print cooling fan speed, and if you did supply a value after R then that value was ignored. Now it is mandatory to specify a restore point number after R.
Fan speeds are no longer restored automatically when resuming a print. If you change the print cooling fan speed in pause.g, restore it in resume.g using M106 R1.
At the start of a tool change, the speeds of individual fans are no longer saved; however the speed of the default print cooling fan is saved in restore point 2. In the unlikely event that you need to save the speeds of individual fans before a tool change, you can retrieve the speeds from the object model and save them in variables.
The use of M106 with both P and R parameters is now deprecated and we plan to remove this facility in RRF 3.4. If you need to save/restore individual fan speeds, retrieve the speeds from the object model and save them in variables.
In preparation for for general input shaping in RRF 3.4, the M593 command now takes a P parameter to specify the input shaping type. Valid values in this release are "none" and "daa" or uppercase versions of those.
In the unlikely event that you have multiple Z probes and they all need the same non-empty deploy/retract sequence, you will need to create separate deployprobeN.g and retractprobeN.g for each probe, where N is the probe number. This is because only probe 0 now falls back to looking for deployprobe.g and retractprobe.g.
M675 can now only be used with a probe, not with an endstop (which didn't make much sense anyway). The K or P parameter must be provided.
M675 now uses parameter K for the probe number, in keeping with G29 and G30. Parameter P is still accepted as an alternative, for backwards compatibility.
M675 now calls the deploy and retract files for the selected probe
Previously, when M675 was executed the final position was saved in restore point 2, however this behaviour was not documented. The final position is no longer saved in a restore point. Use can use the G60 command after M675 if you want to save the position.
M585 now requires the F parameter to be provided
M585 now uses parameter K for the probe number, in keeping with G29 and G30. Parameter P is still accepted as an alternative, for backwards compatibility.
Previously, when M585 was executed the initial position was saved in restore point 2, however this behaviour was not documented. The initial position is no longer saved in a restore point.
Previously, when daemon.g was not found or after it had finished running, RRF paused for 1 second before trying to open it again. That time has been increased to 10 seconds to reduce the load on the SD card. If you need a daemon to execute more often than once every 10 seconds, use a loop inside the daemon.g file.
Spindle management and control has been fully rewritten and now resembles better what NIST GCode standard proposes. See M950, M563, M3, M4 and M5 description in GCode wiki for details.
G31 Cnnn (temperature coefficient) parameter has been moved to G31 Tnnn to make C available as an axis (see new features below)
The object model for the height map (move.compensation.probeGrid) has changed
The object model for resonance compensation (move.daa) has moved
The height map file format has been extended. Height maps generated by earlier versions of RRF can still be loaded by this version, but height maps generated by this version of RRF cannot be loaded by earlier versions.
When changing tool, tool change files are now run regardless of whether axes have been homed or not. You can use conditional GCode to choose which commands are only executed if axes have been homed.
RepRapFirmware no longer tries to work out what layer number is printing, and no longer provides an estimate of print completion time based on layer progress. The mechanism to work out the layer number failed in many cases, for example when the slicer used variable layer heights or printed multiple objects one at a time. The removal of these 200 lines of hard-to-maintain code has made more space available for other features on the older Duets that are tight on memory space. A consequence of this change is that DWC will no longer produce a layer chart if the GCode file being printed does not include layer number comments. For slicers that do not normally generate these comments (e.g. PrusaSlicer) it is usually possible to add a layer change script to generate them.
[Duet 3 Mini] Immediately after upgrading to this release you may get spurious overheat warning messages for all the stepper drivers. If this happens, turn off VIN power to the Duet, wait a few seconds, and power on again.
[Duet 3 + expansion/tool boards] You must update the expansion and/or tool board firmware to 3.3 also, otherwise movement and some other functions will not work. If you accidentally end up with firmware 3.3 on a tool or expansion board and 3.2.x on the main board then the tool/expansion board will not achieve CAN sync with the main board; however it will still respond to some commands including M115, M122 and M997.
[EXP1XD board] Previously, the step pulse width requested in the M569 command was rounded up to the next multiple of 1.33us. Now it is rounded up to the next multiple of 83ns. Additionally, the step pulse interval ands the hold time from the trailing edge of the step pulse to direction change are timed from the start of the step pulse, after adjusting for the width of the step pulse. The effect of these changes is that the step pulse width, step pulse interval, and step-to-direction hold time may not be rounded up as much as before; so M569 T values that used to work only because of the rounding up that occurred may no longer work.
New features:

When a "Bad command" error message is generated, any non-printing characters in the received command are shown in hex to make sure that they are visible
Fan RPMs down to 160 are now reported correctly (the previous lower limit was 320)
In the G38.2 command, parameter K can now be used to select the probe number
The CAN diagnostics in M122 reports have been greatly improved
When rehome.g is called because of motor stalls, for each axis for which a stall was detected, a parameter with value 1 is passed. This means that rehome.g can use tests such as "if exists(param.X)" to determine which axes need to be re-homed.
At the start of a print from SD card, the XY plane is selected for G2/G3 moves
G60 now saves the print cooling fan speed in the restore point
If M106 is used with a R parameter but no P parameter, the R parameter value is now the restore point number to get the speed from. Previously there was only one saved value for the default print cooling fan.
For filament monitor types 4 (magnetic + switch) and 6 (laser + switch), M591 with just the extruder number parameter now reports the filament present/not present status
M595 supports a new optional Q parameter to allow the configuration of the secondary movement queue to be changed
M669 when using Hangprinter kinematics now allows the XY coordinates of the D anchor to be specified
Hangprinter kinematics now supports line build-up compensation
The delay before trying to open the daemon.g file has been increased from 1 to 10 seconds, except for the first time. This reduces contention for the SD card.
The M122 report now includes the firmware date and time
The M122 report now shows the CPU utilisation per task. The reported figures will be inaccurate if more than approx. 90 minutes has elapsed since M122 was last run (or power on if M122 has not been run before).
New 'exists' function can be used to check whether a variable or parameter exists before using it
Accelerometers are supported, currently in standalone mode only (not in SBC mode). They can be connected directly to a main board via SPI, or connected to a SAMMYC21 via I2C.
RGBW NeoPixels LED strips are supported
Bit-banged NeoPixel LED strips are supported on Duet WiFi/Ethernet. On these boards RRF will wait stop movement while updating the LED strip, so M150 commands should not be issued while the machine is printing, other than at places where a short pause is acceptable such as at layer change and in the start and end scripts.
The maximum length of a NeoPixel strip is now 240 on Duet 3, 80 on Duet 3 Mini, and 60 on Duet 2
Tool offsets are now reported to 3 decimal places instead of 2 in G10 and in the object model
Object labelling in GCode files produced by Ideamaker are now recognised
Print time remaining comments in GCode files produced by Ideamaker are now recognised and processed like M73 commands
M25, M226 and M601 all accept a P0 parameter, which causes pause.g not to be run. M24 accepts a P0 parameter, which causes resume.g not to be run. Caution: in RRF 3.4 this is likely to be replaced by a P"filename" parameter.
Added M569.2 which allows any TMC22xx or TMC51xx stepper driver register to be read or written
A separate task is now used to finalise moves for execution
Data returned by TMC2209 drivers is now CRC checked
GCode meta commands now provide for creating local variables ('var' command) and global variables ('global' command), changing the values of variables('set' command), and parameters to macro files
M558 now accepts either one or two F values, e.g. F1000:300. If two values are provided, then when executing a G30 command with no P parameter an additional probe using the first speed will first be done to establish the approximate Z=0 position, followed by one or more probes (controlled by the A parameter) to establish Z=0 more precisely.
When using an analog Z probe, the probing speed is no longer reduced when the probe is close to triggering. Use the new M558 facility to do fast-then-slow probing instead, if necessary.
M568 has been repurposed as Set Tool Settings. It can be used to set tool offsets, tool temperatures and tool spindle RPM. G10 can still be used to set temperatures. M568 can also be used to switch tool heaters between off/standby/active without needing to select or deselect the tool.
M557 has been extended to allow defining a grid between an arbitrary axes pair, e.g. X-A and is no longer restricted to X-Y.
G31 has been extended to allow setting offsets for all axes (except Z)
M997 accepts a new parameter P"filename" to specify the filename to use for a firmware update. This can only be used when exactly one module is to be updated, i.e. it will not work with M997 S1:4
M669 S and T parameters are now supported on all kinematics. This allows segmentation to be used on Cartesian, CoreXY etc. and linear delta kinematics, for the purpose of allowing faster response to pause commands, M220 etc.
Auto calibration of rotary delta machines is supported experimentally
M486 now returns an error message if you try to cancel or resume an unknown object
Estimated print time provided by the slicer (using M73 commands if present) is now available. Estimated print completion time based on layer progress has been removed.
M408 and rr_status no longer report first layer duration or first layer height
Maximum step rates on all boards have been improved
Maximum length of expressions passed from DSF to RRF has been increased from 100 to 256 characters
When extracting file information, the support for Kiro Moto, Matter Control, Pathio, Fusion 360, IdeaMaker and SuperSlicer slicers has been improved
When executing G2 and G3 arc movement commands, the segments are finer than before
The maximum number of tracked objects has been increased, and space for the object names is now allocated dynamically
M17 is implemented
G17, G18 and G19 are now supported
Added M595 R parameter (thanks markmaker)
Increased the number of build plate objects tracked from 32 to 40 on Duet 3 and 3 Mini
[Duet 3 Mini] As an experiment, the MCU temperature sensor has been enabled. The manufacturer's errata document states that the sensor is not supported and should not be used; however it does appear to give useful readings, on some samples at least.
[Duet WiFi/Ethernet] When using extended step timings (M569 T parameter), maximum step pulse rates are improved a little
[Duet 3 Mini] M954 is partially implemented, allowing a Duet 3 MB6HC or Duet 3 Mini to be used as an expansion board. In this mode it can support axis motors, extruder motors (but extruders with nonzero pressure advance has not been tested), thermistor, PT100 and thermocouple temperature sensors, GpIn and GpOut pins (including servos). Heaters, fans, filament monitors, endstop switches, Z probes and other types of temperature sensor are not yet supported.
[Duet 3 MB6HC] The maximum number of axes supported on Duet 3 MB6HC is increased to 15. Axis letters abcdefghijkl may be used in addition to XYZUVWABCD. Because GCode is normally case insensitive, these must be prefixed with a single quote character in GCode commands. For example, M584 'A1.2 would assign axis 'a' to driver 1.2, and G1 'A10 would move the 'a' axis to the 10mm or 10 degree position (or by 10mm or 10 degrees if in relative mode).
[Duet 3 expansion/tool boards] Heater tuning is now implemented (M303) along with heater feedforward
[Duet 3 and 3 Mini with CAN-connected expansion boards] Endstops and Z probes connected to the main board can now stop motors connected to expansion boards, removing the previous limitation. However, a small amount of undetected overshoot may occur when a homing or probing move is stopped. Until this is resolved in a future release, we advise against repeated probing (e.g for mesh bed compensation) when the Z motor(s) are connected to expansion boards, or measuring axis length using G1 H3 moves when the axis motor(s) concerned are connected to expansion boards.
[Duet 3 + expansion/tool boards] Improved the accuracy of CAN clock synchronisation between main and expansion boards
[Duet 3 EXP1XD] All step pulses generated by the EXP1XD now have exactly the same step high time as set by M569. The first M569 T value (step high time) will be rounded up to the next multiple of 0.0833us subject to a minimum of 0.25us (previously it was rounded up to the next multiple of 1.33us). The remaining T values will be rounded up to the next multiple of 1.33us as before. The fourth T value (direction hold time from trailing edge of step pulse) can now be negative, down to minus the step-high time.
[SAMMYC21] The sample firmware for SAMMYC21 now sends a diagnostic report to the USB port if character D is received from the USB port
[Duet + SBC] SBC interface diagnostics have been improved
Object model changes:

Restore points now have an additional field 'fanPwm'
The object model for Hangprinter kinematics has changed. Fields anchorA, anchorB, anchorC and anchorDz have been replaced by a single 4-element array called anchor.
If you are using spindles[].active or spindles[].current they will no longer be negative for counter-clockwise RPM but additionally carry a new field spindle[].state that will have one of the three values stopped, forward or reverse and need to be interpreted together.
spindles[].tool has been replaced by tool[].spindle and reverses the assignment.
The height map object move.compensation.probeGrid has been changed to allow for axes other than X and Y
The resonance compensation object move.daa has been replaced by move.shaping to support more general input shaping. The members of move.shaping have not been finalised yet.
Added move.queue to object model, describing the main and (if present) aux movement queues
Added job.pauseDuration which is the total pause time since the job started
Added job.timesLeft.slicer
Added sensors[].probes[].speeds which is a 2-element array
Added global variables to the object model
job.warmUpDuration is now flagged live
job.layer is now only available if the GCode file being printed includes recognised layer number comments
The following object model fields are now flagged as obsolete and will generate a warning when used:
sensors[].probes[].temperatureCoefficient (use temperatureCoefficients[0] instead)
sensors[].probes[].speed (use sensors[].probes[].speeds[1] instead)
move.compensation.probeGrid.(axis0, axis1, xMin, yMin, xMax, yMax, xSpacing, ySpacing) (use axes[], mins[], maxs[], spacings[] instead)
move.workspaceNumber (use move.workplaceNumber - 1 instead)
job.firstLayerDuration (it will always return null)
job.timesLeft.layer (it will always return null)
Bug fixes:

If a tool heater was tuned using M303 with T parameter, and turning on the print cooling fan made no significant difference to the cooling rate, tuning would sometimes fail with a "bad curve fit" message
If a macro was aborted because a movement command failed, the error message was lost
It was not possible to use math operators with some unsigned object model fields e.g. job.filePosition and job.file.size
When a long-running command was initiated from PanelDue (e.g. mesh bed probing or delta auto calibration), "Bad command" and "Error parsing response" errors were sometimes reported on PanelDue
M595 with an out-of-range Q parameter would reset the machine due to an uncaught exception
M671 allowed up to 8 XY coordinates to be provided, but this should have been 4 (the maximum supported by the bed levelling algorithm)
M3 and M4 commands with both P and S parameters didn't work correctly
If pause.g, filament-change.g or filament-error.g used M291 to display a message and wait for a response, movement resumed until the response was provided
If a reset occurred because of an assertion failure, out-of-memory error or uncaught exception, then the task name in the software reset data in a subsequent M122 report was often corrupted depending on which task it was
Removed support for implicit G2/G3 commands because they is not part of the NIST GCode standard
Fixed M308 "string too long" message when configuring a thermocouple
Fixed memory protection fault or stack overflow crash when trying to pass macro parameters from DCS to RRF on Duet 3 MB6HC
When using polar, SCARA or Hangprinter kinematics the default segment length was not set up correctly, leading to the wrong size segments unless M669 was used with a T parameter
Added stack checks to recursive expression evaluation functions so that deeply nested expressions in user input abort gracefully instead of causing a reset
Using boards[n].shortName or boards[n].firmwareVersion in an echo command cleared any existing result string
Hangprinter kinematics reported itself as SCARA
Hangprinter kinematics needed segmentation for Z moves as well as XY moves, to maintain tension in all the cables
Rotary delta kinematics now uses segmented Z moves
M409 command now permits empty K and F parameters
If the pause.g file turned off the print cooling fan and deselected the tool (in either order) then the fan speed could not be restored in resume.g
Thermocouple sensors could not be configured because M308 didn't accept more than 20 characters in the sensor type name
When using a mixing extruder, if multiple E values were provided in a G0..G3 command but fewer than the number of extruders used by the current tool, the additional extrusion amounts were undefined
The Hangprinter kinematics code sometimes attempted to take the square root of a negative number
M675 command did not work reliably (old bug)
M675 ignored any deploy and retract macro files for the probe (old bug)
M675 R parameter did not take account of the inches/mm setting (old bug)
M675 did not report errors if the probe was already triggered at the start of the probing move, or if the probe was not triggered by the end of the probing move (old bug)
M585 using a probe ignored any deploy and retract macro files for the probe (old bug)
M585 using a probe did not report errors if the probe was already triggered at the start of the probing move, or if the probe was not triggered by the end of the probing move (old bug)
Object model value 'seqs.inputs' was not incremented when various fields of object model array 'inputs' were changed, so DCS and DWC didn't keep the values up to date
Tool names were incorrectly converted to lowercase and had spaces removed
Retrieving 'tools[n].offsets[n]' from the object model failed with an error message
The 2-driver expansion board on a Duet Maestro or Duet 3 Mini used a separate channel to make it available as a pseudo temperature sensor, however that channel was not accessible. Those drivers are now included in the main channel.
The object directory state was not saved in resurrect.g because it was incorrectly written to config-override.g instead
G92 with an axis letter parameter also changed the accumulated extruder position reported by M114
DAA was often not applied where it should have been
When more than about 15 tools were configured, the object model fragment describing the tools was too long to send, causing DWC to disconnect
Job progress information was returned too infrequently when simulating
If a file was printed or simulated a second time, filament- and slicer-based print time estimates were not produced
Slicer-based time estimates were produced even when simulating a file
The original selected tool was not restored after simulating a file
Bed heater settings were not written to resurrect.g was if there was also an active chamber heater
When file resurrect.g was created due to a pause or power failure, settings for thermostatic fans were saved instead of settings for non-thermostatic fans
M500 did not persist the M307 Inn value and thus making inverted heaters non-inverted after M501
Pulsed filament monitors were active even when disabled by using S0 in the M591 command
Rarely, a height map generated by G29 S0 could not be loaded by G29 S1 due to rounding error
Most commands that took a pin name parameter did not accept an expression as the pin name
G4 with zero delay did not wait for movement to stop
G2/G3 commands with R parameter sometimes cause the job to abort
If a simulation was aborted due to an error, motion sometimes occurred
When waiting for a heater to reach temperature with M109 there were no more updates sent to PanelDue
It was not possible to delete a temperature sensor using M308 S# P"nil"
When a sensor was configured on a CAN expansion board and M308 was subsequently used to create a sensor with the same number on a different board, the old sensor was not deleted
[Duet 3 Mini] The stall detection sensitivity register was set incorrectly
[Duet 3 Mini] DHT sensors did not work. DHT sensors on the Duet 3 Mini now require both an output pin and an input pin. Note, support for DHT11 is likely to be removed soon.
[Duet + SBC] It was not possible to update PaneDue firmware using M997 S4
[Duet + 12864 display] The firmware would crash or behave unpredictably if in a 12864 display menu file an attempt was made to display an image at a position such that any row of the image was beyond the last row of the display
[Duet WiFi/Ethernet] Driver 11 connected to CONN_LCD did not work. It should now work provided that you have not configured a 12864 LCD.
[Duet 3 + expansion boards] Laser, rotating magnet and pulsed filament sensors connected to expansion boards were active even when disabled by using S0 in the M591 command
[Duet 3 + expansion boards] Under conditions of high movement density, a small number of motion commands might not be received by the expansion board
[Duet 3 expansion/tool boards] Under conditions of heavy load (e.g. a series of short moves at high step pulse rates), the board could stop responding to CAN commands and lose CAN sync
[Duet 3 + expansion boards] The expansion board M122 report incorrectly recorded nonzero CAN 'bm' and 'wbm' error counts after executing G1 H1 or G1 H3 moves involving motors attached to expansion boards
[Duet 3 + expansion boards] Fixed a race condition in the CAN driver that might result in delayed messages, or in rare cases loss of CAN communication
[Duet 3 Mini] When a low PWM frequency (e.g. 10Hz) was used on a heater, then movement was sometimes disrupted causing sudden jerks. Heaters connected to OUT0 and OUT2 suffered from this but OUT1 did not. Even lower PWM frequencies (e.g. 5Hz) disrupted SD card access too.
[Duet 3 Mini] M122 often reported communication timeouts with the TMC2209 drivers
[Duet 3 MB6HC] The DotStar LED driver did not work because the brightness was always set to zero
[Duet 3 Mini] 'state.atxPower' was always reported as 'false' in the object model after an M80 or M81 command
[Duet 2 + DueX] If in the M652 command the laser was configured to use a fan or GPIO port on the DueX, the firmware crashed when the laser was used. It is no longer permitted to configure a laser on these ports.
[Duet 2 WiFi and Duet 3 Mini 5+ WiFi] Using the C (channel number) parameter in M589 did not set the channel and caused loss of communication between the Duet and the WiFi module
[Duet 3 expansion/tool boards] When a tool/expansion board announced itself to the main board, it corrupted the sensors report message
[Duet 3 + expansion/tool boards] A M116 command at the start of a tpost#.g file might not wait for a tool heater connected to an expansion or tool board to reach temperature if the tool heater was off before the tool was selected
[Duet 3 + expansion/tool boards] Spurious CAN transmit timeouts were reported by main boards in the M122 report
[Duet 3 expansion/tool boards] The free system stack was always reported as zero
[Duet 3 expansion/tool boards] Occasionally the red status LED would briefly flash rapidly indicating loss of CAN sync, when in fact sync had not been lost
[Duet 3 + expansion/tool boards] When a motor on an expansion board was commanded to move when not already enabled, and other commands were being sent to expansion boards concurrently, "Discarded message" warnings were sometimes generated
[Duet 3 + expansion/tool boards] An expansion or tool board would very occasionally reset without warning because of a race condition. The software reset data in the M122 report for that board reported an assertion failure.
[Duet 3 MB6HC + expansion/tool boards] There was excessive clock jitter on expansion/tool boards because the CAN timestamp counter in the MB6HC MCU does not always operate in accordance with the manufacturer's documentation
[Duet + SBC] RRF updated job.lastFileName as soon as the print started. RRF no longer reports job.lastFileName while a print is in progress.
[Duet + SBC] The SBC task stack might overflow when a 'set' command was received from the SBC and executed
