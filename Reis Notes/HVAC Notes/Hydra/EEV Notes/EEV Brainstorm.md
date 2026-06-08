
People: Thomas, Sean, Dominic, Song, Tats

Notes:
- Get the sensors hal ready and a shell of some type of control or state machine to allow testing 
- One thing that we would have to do is interrupt the compressor call. Delay the compressor then fully open the EEV.
- Can do lookup table or some type of formula to decide how much to preopen the compressor based on ambient and pressure conditions
- EEV keeps position regardless of power loss
- High Level:
	1. Y1 comes in
	2. EEV opens to a certain designated position
	3. Run compressor
- Cannot keep the EEV shut because it may crack the pin (look into this common failure mode)
- Good on power up of the unit to ensure that the EEV comes back open to avoid the issue of cracking the EEV
- EEVs typically ship with the EEV closed so that the refrigerant wont leak out of the system (split systems). Need to ask product team how this will be managed on packaged units

- Velocity PI control is what we will look into using:
	- EEV doesn't know where it is so using a velocity we just find out how much to move not where exactly to go
	- The delta will be determined from how far we are from the goal via delta