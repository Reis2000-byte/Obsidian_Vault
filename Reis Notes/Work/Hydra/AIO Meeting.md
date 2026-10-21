## Write only registers

| Availability at launch | Name                            | Values/Notes                                                |
| ---------------------- | ------------------------------- | ----------------------------------------------------------- |
| Yes                    | effective setpoint request      | 0-99                                                        |
| Yes                    | room temp                       | 0-120                                                       |
| Yes                    | mode request                    | cool, heat, none                                            |
| Yes                    | off request                     | false, true                                                 |
| Yes                    | constant fan only speed request | low, high, default for 0-10V \| taps will always be default |
| Yes                    | idle fan request                | false, true                                                 |
| No                     | Occupancy                       | use for only onboard economizer                             |
| No                     | Dehum                           | on, off \| only use with 0-10V control                      |
| No                     | CO2 Space                       | if offered by venstar send it                               |

- AIO handles occupancy
- Hydra will keep PID and building control
- AIO will handle the effective SP logic
- Will only use occupancy information if we are using the on board economizer. Otherwise we want Venstar to do it
- Modbus Client will be what Hydra needs and Modbus Server is what will be on the AIO
- Hydra will define the poll rate and for example if our poll rate is 5 seconds then it will take 5 seconds for the unit to react
- BACnet and AIO will share point list for control
- If you have BACnet and AIO connected then AIO wins and will be the value that we follow. If the BACnet point exists then it becomes view only
- 24V control trumps all