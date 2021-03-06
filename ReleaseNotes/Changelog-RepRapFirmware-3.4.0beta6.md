Upgrading from a 3.4 beta before 3.4b5? Please see the full changelog for previous beta versions of RRF 3.4 here: https://github.com/Duet3D/RepRapFirmware/wiki/Changelog-RRF-3.x-Beta-&-RC
Upgrading from a previous stable version? Also see  the full changelog for previous stable versions of RRF here: https://github.com/Duet3D/RepRapFirmware/wiki/Changelog-RRF-3.x

RepRapFirmware 3.4.0beta6
=========================

Upgrade notes:
- **Important**! After changing tool, RRF no longer moves the new tool head to the coordinates at which the old tool head was at when the Tn command was actioned. In most situations this should not matter, because GCode generators usually generate commands to move to the correct XYZ position after generating a Tn command. Usually, they restore XY before Z. However, Cura does it the other way round, which risks dragging the tool head across the print. Therefore when using Cura, or in any other situations in which you want to restore the old behaviour, we suggest you add command G1 R2 X0 Y0 Z2 Fxxx followed by G1 R2 Z0 Fxxx to the end of your tpost#.g files, if you don't already use those or similar commands.
- There is no longer a power supply control pin assigned by default (in previous firmware versions, PS_ON was assigned by default). Therefore, M80 and M81 will not work until you have assigned a power control pin. If you want to control the power supply, you should use assign a pin using either M80 or M81 with the C parameter in config.g. Use M80 if you want to start with power on, or M81 if you provide separate 5V power and you want to start with VIN power off.

Known issues:
- After a tool change, the status in DWC and the object model field state.status may continue to be reported as "Changing tool"

New features and changed behaviour:
- After a tool change, the new tool is no longer moved to the position that the old tool was at when the tool change started (see the Upgrade Notes section).
- Input shaping types "mzv" and "zvddd" are now supported, in addition to the other types that were supported already
- G68 and G69 are implemented on an experimental basis when the selected plane is the XY plane. Use this with caution until more testing has been done.
- If an attached magnetic filament monitor running version 4 firmware is unable to start because of a magnet problem, RRF now shows the error in the M591 response and M122 report
- When stall detection endstops are used, the acceleration of any move for which a stall detection endstop is enabled is reduced
- The reduced accelerations used by Z probing and stall homing moves can now be configured using command M201.1
- M591 now reports whether all extruder movements or just printing moves are checked when using a Duet3D magnetic or laser filament monitor
- The maximum depth for nesting macro calls and M120/M121 has been increased from 7 to 10
- M81 now accepts a C parameter, allowing a PS_ON pin to be assigned in config.g while leaving VIN power turned off
- The PS_ON pin is no longer allocated as a power control pin by default. See the upgrade notes.
- [Duet 3 MB6HC and 3 Mini] Heaters, fans and filament monitors are now supported on these boards when they are switched into expansion mode using M954. Caution: this has not been tested thoroughly!
- [Duet 3 + expansion boards] Expansion boards now track the temperatures of all sensors in the system. This means that a thermostatic fan connected to an expansion board can now be controlled by sensor(s) on different expansion boards(s).

Object model changes:
- Field fans[].frequency is added (the PWM frequency). This was already shown by DWC, however as RRF did not report the value DWC always displayed the default value of 250Hz.
- Fields move.shaping.amplitudes[] and move.shaping.durations[] are added
- Fields move.rotation.angle and move.rotation.centre are added
- Field move.kinematics.segmentation is added. It is null if the current kinematics is not using segmentation, else it has values segmentsPerSec and minSegmentlength.
- For magnetic and laser filament monitors, field filamentMonitors[].configured.allMoves is added

Bug fixes:
- Fixed typo in message "the height map was loaded when the current Z=0 datum was not determined **by** probing"
- Fixed issue with configuring a temperature sensor of type "thermocouple-max31856" on an expansion board (again).
- The number of decimal places to which an expression was displayed in an echo command was sometimes lower than desirable if a division operation was involved in computing the value
- M0 when not paused always turned the heaters off even if there was a stop.g file
- G1 H2 moves involving only rotational axes did not always work
- Short extruder movements caused magnetic, laser and pulsed filament monitors to report under-extrusion
- Possible race conditions in the magnetic, laser and pulsed filament monitor code have been fixed. This may reduce the occurrent of spurious paused due to reported under or over extrusion.
- DWC did not report the calibration figures magnetic, laser and pulsed filament monitors
- The extruder position reported in the object model and by DWC was always zero
- In laser mode, the laser could be left turned on for up to 5ms after the end of a move
- Z probe types 1,2,3 and 5 are only supported for Z probe 0 but no error message was generated if you tried to use one of these types for a Z probe other than probe 0
- M669 for kinematics that are not usually segmented did not report the segmentation values if segmentation was enabled
- M669 did not report the current A and F parameters when using Polar kinematics
- Requests for ocsp-over-http were not recognised if the requested filename started with "ocsp" instead of "/ocsp"
- The default M203 maximum speeds, M201 accelerations, M566 jerk rates and M558 speeds were not set correctly (this was a new bug in 3.4beta)
- When using Polar kinematics the A and F parameters didn't work (this was a new bug in 3.4beta)
- When using Polar kinematics, XY movement was not permitted until Z had been homed
- [Duet + SBC] Object model variables that represented a date and time, a processor unique ID or a port name could not be passed to DSF. One consequence of this is that they could not be used in echo commands, except as part of a larger expression.
- [Duet 3 MB6HC] Fixed stall detection sensitivity change
- [Duet 3 MB6HC with DotStar LED strip] If the brightness M150 P parameter (brightness) was not a multiple of 8 then the blue LED level was incorrect
- [Duet Maestro] The Zprobe MOD pin went low after approx. 45ms after startup. This confused an attached BLTouch which was starting up, causing it to go into error mode. This delay has been reduced to about 20ms.