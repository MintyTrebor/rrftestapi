Upgrading from a stable version earlier than 3.2? Please see the full changelog for previous stable versions of RRF here: https://github.com/Duet3D/RepRapFirmware/wiki/Changelog-RRF-3.x

RepRapFirmware 3.2.2
====================

Upgrade notes: none since 3.2

Bug fixes:
- Using an external SD card could result in the firmware crashing if there was concurrent access to the Duet from DWC or another network client. [This is an old bug, however changes in firmware 3.1 and later made it much more likely to happen.]
- If two DueX endstop inputs changed state approximately 2.2ms from each other, the second one might not be detected by the firmware. [This was a regression in 3.2]
- The pin name string in M574 commands was truncated to 50 characters if it was longer, which was too short to allow four DueX endstop pins to be used. This has now been increased to 100 characters.
- The M109 command caused a "Homing failed" message if it was used in a homing file and that homing file also called another macro file
- In GCode expressions it was not possible to compare a value with character type (e.g. move.axes[nn].letter) with a string literal
- Spurious blank lines were sometimes sent to USB and Telnet clients when the daemon.g file was used
- If a G2 or G3 command with R parameter described an arc of exactly 180 degrees, then due to rounding error RRF might report "G2/G3: invalid combination of parameters".
- Implemented a partial fix for G2/G3 moves on IDEX machines
- [Duet 3 Mini] It was not possible to write to an external SD card
- [Duet + SBC] Expressions were not supported in GCode parameters that accept an array of values. Expressions are now supported when only a single element needs to be provided.
- [Duet 3 + expansion/tool boards] If DAA (M593) was used then motors connected to expansion or tool boards might lose steps
- [Duet 3 + expansion/tool boards] The M591 E parameter was ignored for filament monitors connected to expansion or tool boards
- [Duet 3 MB6HC] (regression in 3.2) DHT sensors did not work 
- [Duet 3 MB6HC] (regression in 3.2) The minimum step pulse width time of the TMC5160 was not always met. This could lead to missed steps when high step rates were used.