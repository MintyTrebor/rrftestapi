RepRapFirmware 3.4.0beta5
Known issues:

[Duet 3 MB6HC] The stall detection sensitivity has changed and it is harder to find settings that work. If you use stall homing with the MB6HC then you may wish to wait for the next beta.
Upgrade notes:

If you upgrade tool or expansion board firmware without also upgrading the main board firmware, then the tool or expansion board will not appear in the object model in the "boards" array
[Duet 3/3 Mini in standalone mode] The default MAC address will change in this release. This means that your router is likely to assign it a different IP address from the previous one. If you have a DHCP address reservation for the Duet configured in your router, you will need to update it for the new MAC address.
[EXP1HCL board] You will need to re-tune your PID parameters. High P and D values are more likely to cause oscillation than before.
New features and changed behaviour:

Hangprinter support has been improved, in particular Hangprinter 4 is supported by the Duet 3 MB6HC board + EXP1XD boards, using the secondary CAN bus to configure the ODrives (thanks Torbjørn Ludvigsen)
M280 commands to reset babystepping to absolute values (e.g. M290 R0 Z0.0) are once again permitted even when the axes concerned are flagged as not having been homed; however if the axes have not been homed then no movement will be performed
When updating PanelDue firmware, the firmware file is checked to make sure that it is not too large
The unique IDs of CAN-connected expansion boards are now included in the object model, in their boards[] entries
[Duet 3/3 Mini + CAN-connected expansion boards] M122 for expansion boards now returns min, current and max values for VIN and (if supported) V12. As for the main board, each use of M122 resets the min and max values.
[EXP1HCL expansion board] The M569.5 command may now use higher sample rates. If using a high sample rate results in the sample buffer overflowing then sampling will be stopped at that point, data already collected will be written to the file, followed by "Buffer overflowed" on a new line.
[EXP1HCL expansion board] When the M569.5 commands includes a V parameter, the motor will be enabled automatically, as for the M569.6 command
[EXP1HCL expansion board] When executing a tuning move there is now a 100ms delay before starting the move, to allow the motor brake to be released and the driver to be enabled
[EXP1HCL expansion board] The number of tuning moves has been reduced to simplify tuning. V1 does basic tuning (polarity detection, zeroing for relative encoders, and CPR checking) and is needed after every power on regardless of the encoder type. It moves up to 4 full steps in either direction and returns to the original position. V2 performs calibration of magnetic motor angle sensors. It performs up to two complete motor resolutions. If successful then the calibration data is stored in non-volatile memory so that it need not be run again. No other tuning moves are supported at this time.
[EXP1HCL expansion board] In closed loop mode the minimum holding current in percent is now set by the M569.1 H parameter. In open loop mode, M917 is used as before.
Bug fixes:

[Duet + SBC] Expressions such as "abs(move.calibration.initial.deviation - move.calibration.final.deviation) < 0.000" resulted in a "Expression nesting too deep" error
[Duet 2 WiFi/Ethernet] If a TMC2660 driver was removed from the Duet or Duex then none of the TMC2660 drivers would start up
[Duet 3/3 Mini + CAN-connected expansion boards] In the object model the minimum values of VIN and V12 for expansion boards were always zero even if the board had been reset after power up
[Duet 3/3 Mini in standalone mode] When generating default MAC address, the last byte of the processor unique ID was not used. This could lead to MAC address clashes when two boards from the same manufacturing batch were used on the same network.
[EXP1HCL expansion board] When collecting data from the closed loop motion controller, the last variable that was requested to be collected was not collected properly
[EXP1HCL expansion board] Tuning moves were executed too fast for some types of stepper motor
[EXP3HC expansion board] The buffer used to store the sensor type name in a M308 command was too short to accommodate "thermocouple-max31855"
Object model changes:

All boards[] objects now include the unique ID of the corresponding board if known. Previously, only the entry for the main board included the unique ID.
Expansion boards now have separate firmwareVersion and firmwareDate properties in the boards[] object. Previously, firmwareVersion held both the version and the build date. Both the main boards and the expansion boards need to run firmware 3.4.0beta5 for this to work correctly.
Internal changes:

FatFs has been upgraded from version 0.13c to 0.14b