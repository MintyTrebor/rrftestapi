RepRapFirmware 3.4.0beta1
Known issues:

If the slicer uses the M486 command to label objects in the GCode file, and there are more than 20 (Duet 2) or 40 (Duet 3) objects, then RRF crashes when printing the file. This bug is also present in release 3.2.x and 3.3.
If you home any axes when workplace coordinates are in use, then any absolute moves in the homing files have workplace coordinate offsets applied; whereas they should not. This bug is also present in release 3.3.
When M955 reports the configuration of an accelerometer that is connected to the main board using SPI, the orientation is always reported as 20 even if a different orientation has been selected (and is being used) by means of the M955 I parameter. This bug is also present in release 3.3.
Sending a M956 command that refers to an accelerometer on a CAN-connected board via the Pi (e.g.rom DWC) does not work and report that the accelerometer was not found. Workaround: send the command to the Duet from a PanelDue or via USB instead. [This will need to be fixed in DSF]
Upgrade notes and behaviour changes:

[Duet + SBC] This version of RepRapFirmware requires Duet Software Framework version 3.4.0beta1
M106 with both P and R parameters is no longer supported. M106 with R parameter but no P parameter works as in release 3.3.
DHT11 temperature/humidity sensors are no longer supported. DHT21 and DHT22 sensors continue to be supported.
Previously, if a G- or M-code did not have any variants with fractional parts, any fractional parts would be ignored. For example, M114.1 would be treated the same as M114. Now, M114.1 will provoke a "not implemented" warning; but M114.0 will still be treated the same as M114.
New features and changed behaviour:

Collection of accelerometer data is now supported in SBC mode
DHT11 temperature/humidity sensors are no longer supported
DHT21 and DHT22 sensors now use less RAM
The first layer height of the file being printed is no longer reported in the object model or by M408
The number of layers of the file being printed is no longer reported in the object model
G- and M-codes with fractional parts in the code number can now be implemented using macro files. For example, if RRF received command M1134.1 then it will look for file M1134.g. This works even if the main code without fractional parts is already implemented by RRF; for example, you could provide file M114.1.g to implement code M114.1.
When creating a heater using M950 with H parameter, multiple output ports can now be used
M955 with just a P parameter now reports the SPI frequency if the accelerometer is connected via SPI
Added M956 F"filename" parameter
Increased maximum number of RGB NeoPixel LEDs on Duet WiFi/Ethernet from 60 to 80
If a G31 command specifies H and/or T parameters but they are not valid, the G31 command is no longer aborted and the remaining parameters are processed
[Duet 3 MB6HC] The second CAN port is now configured and enabled. It uses plain CAN 2.0 protocol (not CAN-FD) and a default bit rate of 250kbps. It is provided primarily to facilitate configuration of external devices (e.g. ODrive) in forks or future versions of RRF.
Bug fixes:

M117 commands processed when the movement queue was not empty gave rise to "string too long" error messages
If the selected kinematics had both rotary and linear axes, and commanding movement of a rotary axes only required movement of a motor controlling a linear axis, then the feed rate calculation failed
[Duet 3 + expansion/tool boards] If some axes were driven using external drivers only, the main board could send moves too fast to the expansion board(s), which could eventually result in lost moves
Object model changes:

Field heaters[].avgPwm is added
Field heaters[].model is no longer flagged 'verbose'
Fields job.file.firstLayerHeight and job.file.numLayers are removed