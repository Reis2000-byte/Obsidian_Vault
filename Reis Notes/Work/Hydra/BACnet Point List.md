Types of BACnet Variables:

AI: Sensor value (temperature, pressure, humidity)
AO: Commanded analog output (damper %, fan speed %, valve %)
AV: Internal variable, setpoint, calculated value

BI: Status feedback, switch, proof-of-operation
BO: Relay command, compressor enable
BV: Internal software flag

MSI: Read-only state feedback
MSO: Commanded operating mode
MSV: Internal operating state

- **Measured value** → **AI**
- **Commanded analog value** → **AO**
- **Internal numeric value/setpoint** → **AV**
- **Status input** → **BI**
- **Command output** → **BO**
- **Internal Boolean flag** → **BV**
- **State machine or operating mode** → **MSV**

| Type | Variable Name              | Units | Explanation                                  | R/W | On Board? | Future? |
| ---- | -------------------------- | ----- | -------------------------------------------- | --- | --------- | ------- |
| AI   | Space Temp                 |       | The one that is being used by the controller | R   |           |         |
| AI   | Supply Air Temp            |       | The one that is being used by the controller | R   |           |         |
| AI   | Space Humidity             |       | The one that is being used by the controller | R   |           |         |
| AI   | Outdoor Air Temp           |       | The one that is being used by the controller | R   |           |         |
| AI   | Outdoor Air Humidity       |       | The one that is being used by the controller | R   |           |         |
| AI   | Economizer Feedback        |       |                                              | R   |           |         |
| AI   | CO2                        |       |                                              | R   |           |         |
| AI   | Head Pressure circuit 1    |       |                                              | R   | Yes       |         |
| AI   | Head Pressure circuit 2    |       |                                              | R   | Yes       |         |
| AI   | Suction Pressure circuit 1 |       |                                              | R   | Yes       |         |
| AI   | Suction Pressure circuit 2 |       |                                              | R   | Yes       |         |
| AV   | Blower Speed               |       |                                              | R/W |           |         |
| AV   | Active Cooling SP          |       |                                              | R/W |           |         |
| AV   | Active Heating SP          |       |                                              | R/W |           |         |
| AV   | Active SAT SP              |       |                                              | R/W |           |         |
| AV   | Active Humidity SP         |       |                                              | R/W |           |         |
| AV   | Heating Demand             |       |                                              | R/W |           |         |
| AV   | Cooling Demand             |       |                                              | R/W |           |         |
| AV   | Space Temp BAS             |       |                                              | R/W |           |         |
| AV   | Space Humidity BAS         |       |                                              | R/W |           |         |
| AV   | Outdoor Air Humidity BAS   |       |                                              | R/W |           |         |
| AV   | Outdoor Air Temp BAS       |       |                                              | R/W |           |         |
| AV   | CO2 BAS                    |       |                                              | R/W |           |         |
| AV   | OccCool SP                 |       |                                              | R/W |           |         |
| AV   | OccHeat SP                 |       |                                              | R/W |           |         |
| AV   | UnoccCool Offset           |       |                                              | R/W |           |         |
| AV   | UnoccHeat Offset           |       |                                              | R/W |           |         |
| AV   | SpaceTempAlarmOffset       |       |                                              | R/W |           |         |
| AV   | SpaceHumAlarmOffset        |       |                                              | R/W |           |         |
| AV   | SpaceHumSP                 |       |                                              | R/W |           |         |
| AV   | SpaceHumUnoccOffset        |       |                                              | R/W |           |         |
| AV   | CoolingBand                |       |                                              | R/W |           |         |
| AV   | MechCoolAlarmOffset        |       |                                              | R/W |           |         |
| AV   | LowSuplyAirAlarmSP         |       | Low Supply Air Temp Alarm setpoint           | R/W |           |         |
| AV   | HighSupplyAirAlarmSP       |       | High Supply Air temp Alarm setpoint          | R/W |           |         |
| BI   | BlowerProvingSw            |       | Status                                       | R   | Yes       |         |
| BI   | Compressor 1HPS            |       | Status                                       | R   | Yes       |         |
| BI   | Compressor 2HPS            |       | Status                                       | R   | Yes       |         |
| BI   | Defrost Switch 1           |       | Status                                       | R   | Yes       |         |
| BI   | Defrost Switch 2           |       | Status                                       | R   | Yes       |         |
| BI   | Dirty Filter Switch        |       | Status                                       | R   | Yes       |         |
| BI   | Furnace Board Feedback     |       | Status                                       | R   | Yes       |         |
| BI   | TSTAT G                    |       | Status                                       | R   | Yes       |         |
| BI   | TSTAT Y1                   |       | Status                                       | R   | Yes       |         |
| BI   | TSTAT Y2                   |       | Status                                       | R   | Yes       |         |
| BI   | TSTAT W1                   |       | Status                                       | R   | Yes       |         |
| BI   | TSTAT W2                   |       | Status                                       | R   | Yes       |         |
| BI   | TSTAT O                    |       | Status                                       | R   | Yes       |         |
| BI   | Cooling Lockout            |       | Status                                       | R   | Yes       |         |
| BI   | Heatlockout                |       | Status                                       | R   | Yes       |         |
| BI   | HeatpumpHeat Lockout       |       | Status                                       | R   | Yes       |         |
| BI   | Economizer Enable          |       | Status                                       | R   | Yes       |         |
| BI   | Defrost Enabled            |       | Status                                       | R   | Yes       |         |
| BI   | Dehum Lockout              |       | Status                                       | R   | Yes       |         |
| BI   | RevValve 1                 |       | Status                                       | R   | Yes       |         |
| BI   | RevValve 2                 |       | Status                                       | R   | Yes       |         |
| BI   | Comp 1                     |       | Status                                       | R   | Yes       |         |
| BI   | Comp 2                     |       | Status                                       | R   | Yes       |         |
| BI   | Exhaust Fan                |       | Status                                       | R   | Yes       |         |
| BI   | AuxHeat 1                  |       | Status                                       | R   | Yes       |         |
| BI   | AuxHeat 2                  |       | Status                                       | R   | Yes       |         |
| BI   | Cond Fan 1                 |       | Status                                       | R   | Yes       |         |
| BI   | Cond Fan 2                 |       | Status                                       | R   | Yes       |         |
| BI   | LeadLagEnable              |       | Status                                       | R   | Yes       |         |
| BI   | ExhaustFanInstalled        |       | Status                                       | R   | Yes       |         |


Does Hydra need server or client for BACnet:
- BACnet server:
	- Typical relationship for BMS devices
	- We would be the server sharing our information to the BAS
	- This is what DDC is
- BACnet Client:
	- Do we need to retrieve information from any device? If no then we do not
Does Hydra need server or client for Modbus:
- Modbus Client is when the device polls for information:
	- this is a use case for when we have a daughter board and we will need to poll it for information
- Modbus Server is when the device shares information:
	- this would be the case for the BAS stuff if we offer that over Modbus
	- It would also be what our daughter PCBs have to have
- Note:
	- The controller can be both the server and the client BUT it needs different network lines
