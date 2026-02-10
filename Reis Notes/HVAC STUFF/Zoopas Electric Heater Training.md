
- electric heater is essentially just a resistor
- current goes through the resistor and emits heat
- 1W = 3.41 BTU/h
- open coil is what we use and a ton of HVAC companies
- Common components:
	- resistance wire:
		- Higher Nickel content for more expensive product
	- supporting frame
	- sheet metal structure
		- use to mount and support the heaters in the product
	- insulators
		- point suspension or string through are the two styles of insulators
		- LC uses string through
		- These are used to suspend heaters in space because the resistance wire is electrically charged so to prevent them shorting to the sheet metal
	- thermal safety
		- required by UL standard
		- Two safeties. Backup and primary
		- Primary senses an abnormal case and will shut down then reset. Can be 100k cycles
		- Back up needs to be replaced or manually reset. Example is a thermal fuse
	- overcurrent protection
		- to mitigate electric faults
	- electrical controls
		- use contacts and relays
		- newer products use SSRs

- PTAC uses a less complex design with just coils and insulators
- More complex products use circuit breakers, relays, and stepper controls

Development:
- Typically systems should be tested for 0 clearance because that is the worst case
- Typically with Daikin we assume 0 degree clearance due to work history

Simultaneous Heating (HP w/ Electric Heat):
- they test a steady state test with minimum airflow and maximum static pressure with both heating elements energized
- when performing the tests they look for no safety trips
- Abnormal Test:
	- Restricted inlet test: to simulate a dirty filter
	- Fan Failure:
		- protects against OEM fan failures. Shuts the heater off and gives it a very small duty cycle
	- Blocked Outlet:
		- Shut off heater and give short duty cycles

- R&D Process:
	- Try to limit number of SKUs to allow for easier compliance due to less variability
	- Look at heater position. Radiant impact is difficult to simulate and changes based on heater position

- STEPPER MOTOR CONTROL IS COMING
- We are now looking to use stepper motor and SSR control to allow full heat control and past discrete heating
- Drain Pan heater