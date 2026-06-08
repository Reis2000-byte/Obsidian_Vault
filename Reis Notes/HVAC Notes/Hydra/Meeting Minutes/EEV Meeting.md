Date: 4/13/2026
Participants: Hiro, Sean, Song, Devin, Kyle
- Currently for steady state they use a PI control
- For transient states they used fixed positions for every one
- EEV control logic for the INV system resides in the main PCB

- One EEV per circuit
- If accumulator is empty then it is possible to put the sensors close to the compressor. If the accumulator has liquid in it then this is bad. Read into it and understand more!
- INV has a thermistor and pressure before the accumulator and one thermistor after the compressor
- Proposed for the regular unit is one thermistor and one pressure before the accumulator
- Filter dryer pulls out moisture
- Would want mechanical filter just to protect the EEV

Rough Cost:
- EEV 10
- TXV 25
- Thermistor 4
- Pressure 7

- For sensors they braze a clip onto the unit. They then slide it in and wrap with insulation and zip ties. There is a spring inside of the clip to hold the thermistor in to prevent it from falling out

- Transient State:
	- Defrost
	- Start Up
	- Shutdown
	- 0 ->1 Stage
	- 1 -> 2 Stage
- INV uses a gain formula for transient states BUT for us because we have a fixed speed system we could probably get away with just simplified EEV positions and maybe a gain formula for transition?
- FIT is the product that uses the two thermistor design for EEV calculation

- Why don't we use FIT sensors because they are one buck?

- For two thermistors we would put one right the heat exchanger and one inside the heat exchange. Therefore for HP units we need 4 per circuit but AC 2 per circuit. To calculate SH we would just take the temperature differential.
