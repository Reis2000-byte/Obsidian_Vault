Compressor Features:
- Minimum and anti short cycle runtimes
- temp monitored crankcase heater control (thermistor value)
Circuit Features:
- lead lag support
- multiple compressor configuration support
- delays
Blower Features:
- 0-10V
- 24V speed tap
- delays
Outdoor Fan:
- ability to turn on/off
- can add delays
Reversing Valve:
- control to turn on/off
- ability to control to help stabilize pressure (time delays nothing monitoring pressure)
Aux Heat:
- 1-2 stage electric heat control
- ability to work with gas furnace PCB (monitor blower feedback)
Defrost:
- Timed defrost support
- Demand defrost capabilities (Software will need to be developed)
EEV:
- hardware already on board
- goal is to replicate control being used for DDC
- can support up to 2 EEVs
Economizer:
- basic economizer support HW on board (not for initial lauch no software developed) internally developed
- Support for current FIOPs (plug on board)
Sensors:
- Ability to reject poor values
- ambient air Temp
- coil temp A
- coil temp B
- suction temp A
- suction temp B
- discharge temp A
- discharge temp B
- pressure A
- pressure B
- outdoor air temp
- return air temp
- outdoor humidity
- return humidity
- mixed air temp
- co2 sensor
Control type:
- 24V
- communicating (not for initial launch to Reis knowledge)
- AIO Venstar Stat (not for initial launch)
Safety Switches and Diagnostics:
- HPS
- LPS
- Freeze
- Float
- TSTAT Fault
- BMS Fault
- A2L Mitigation
- Phase Monitor
- Economizer Alarm
- Smoke Alarm
- Fire Alarm
- Dirty Filter Switch
- Blower Proving
- Compressor Discharge temp
Communication:
- BacNet (Venstar AIO and other connections)
- Modbus (Venstar AIO and other connections)
- ACS interface
- DG Connect interface
- Bluetooth app connection on board (pending FCC testing)
	- ability to change parameters
	- OTA ability
	- ability to save configurations to then push onto another unit (template system for field)
General:
- on board minimal UI and fault codes (seven segment and 2 buttons)
- ability to support numerous configurations
- field configuration support

