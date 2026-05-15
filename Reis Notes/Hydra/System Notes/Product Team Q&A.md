Q: If there is a fire or smoke fault then you have to power cycle the PCB because it locks out the A2L stuff
A: Need to ask EE team. Also ask Roberto about it again to understand the behavior

Q: Ask about condenser fan rundown can help or is there a mechanical reason?
A: Don't think it helps much but cant see a harm either. Field might think it is a little funky

Q: Ask about how we will capture A2L requirements in the Hydra requirements?
A: We will do it and own it. Copy from Patrick spec and edit if need and take what we need.

Q: Checkout what to do with float phase and float switch faults
A: Float: Maybe just keep blower going. Def turn off comp
A: Phase: Blower might run because a2l is priority. Smoke>Fire>A2L>Everything else. Will have to still turn on/off certain items based on other safeties
A: BMS: Didn't ask product team about this. Will look at Emergency Shutdown on DDC for reference.

Q: Dedicated Heat, Stage Request ask about running compressor and heater together to help stage 2 heat. 
A: Mechanical is good with this but to an extent. If there is a small delay then it was mentioned that compressor heating is much more efficient than electric heat.

Q: Is there a requirement for staging heaters?
A: All heater kits are staged

Q: Is the heater kit equally sized between stages also can we turn on 1 stage and not another for electric heat?
A: They are generally equal sized but there are a range of heater kits that can be chosen. Some are not equally sized. Also not all heater kits are factory installed. They are also frequently installed in the field and as long as it meets the ranges that fit the system they can install it.

Q: Lead lag for electric heat?
A: They are quite robust. Mechanical has no concerns about wearing them down. Concern is that if one of the coils fails you then have to replace both of them. Mechanical sees not much concern. There are cases if the heater kits are different sized. Gets done on field a ton. Can happen in factory but not always

Q: Second Stage for staged compressor anti short cycle or delay?
A: Yes but reduced timer because opening the valves fast could be bad but also to protect the solenoid connected to the compressor.

- **Product Team Concern:** There was a concern voiced from mechanical team that constantly switching between heater kit and compressor could cause confusion to field techs/installers. Also can lead to confusion to lab and production teams due to new more complex behavior.
- Explanation: Here is a situation. We have a medium HP unit. The 


