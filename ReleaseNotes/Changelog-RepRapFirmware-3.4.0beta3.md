RepRapFirmware 3.4.0beta3
New features and changed behaviour:

Input shaping is supported experimentally on the Duet Maestro
The ACT LED on the Duet3 Mini is supported
The prototype EXP1HCL board is supported
The option to append a re-only file system to the firmware binary an the end of the build process has been added
The H parameter of M0 is no longer supported. This means that if a print finishes with M0 and there is no stop.g file, all heaters will be turned off. Likewise, if a print is cancelled and there is no cancel.g file, all heaters will be turned off. Use a stop.g and/or cancel.g file if you want to leave heaters on.
M567 may now be used without a P parameter, in which case it defaults to the current tool
M569.7 (configure driver brake port) is supported, initially on the main board only
M955 with parameters other than P no longer returns the accelerometer details if the command was successful. M955 with just the P parameter still does.
echo >"filename" ... and echo >>"filename" ... are now supported
When a fan is configured as thermostatic using M106, the S parameter is now ignored. If a single T value is given, then when the temperature is above the T parameter the fan will run at the PWM specified by the X (maximum PWM) parameter (default 1.0).
The EXP1HCL closed loop driver board is now supported
The maximum number of fans supported on Duet 3 systems is now 20
Object model changes:

Added field canReverse to Spindle
Added fields percentCurrent and percentStstCurrent to move.axes[] and move.extruders[]
Heater PWM is now reported to 3 decimal places (was 2)
boards[].vin, boards[].v12 and boards[].mcuTemp for CAN-connected expansion boards are now populated if known and null if unknown
boards[].accelerometer is now added. It is null if there is no accelerometer connected to that board; otherwise it has fields "runs" and "points". "runs" is the number of runs commanded that completed or failed. "points" is the number of data points written to file when the last run completed, or usually 0 if the run failed.
Bug fixes:

M106 proportional thermostatic control (i.e. using two T values) didn't work
M150 would hang if the LED type required bit-banging and the movement queue was not empty when it was executed
M505 did not report an error if the specified folder did not exist
M568 did not allow the P parameter to be omitted
The M584 command did not accept an array expression as a parameter value
M600 did not work in SBC mode
If a M956 command selected only some axes to collect accelerometer data for, and the orientation was not 20, then some collected axis data might be all zeros or from the wrong axis
Pressure advance was reported incorrectly in the object model
Detection of initial heating failure did not happen for heaters with slow heating rates
Under some conditions (most likely with a short movement queue length and many short moves) the movement queue would stall and the machine came to a standstill. When this happened, it could be restarted by pausing and resuming the print.
Local variables created within a job file were not cleared when the job completed or was cancelled
[Duet 3 expansion/tool boards] If a PT1000 sensor was configured but not connected, RRF reported a crazy temperature
[Duet 3 tool boards] A small number of tool boards gave obviously inaccurate temperature readings when running tool board firmware 3.4.0beta2