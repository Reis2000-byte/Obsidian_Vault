
Read Property Multiple
- We have
Write Property Multiple
- We have
Who-Is/I-Am
- command to send to basically ping device and it will return name and id
Who-Has / I-Have
- Ping for a sensor or data point and then everyone will return their data points
Device Communication Control
- Ability to disable BACnet remotely from the controller
- Think how to make it to where we can turn the unit back on
Reinitialize Device
- Reset device communication stack
Acknowledge Alarm
- Less critical alarms that you can ignore for couple days
- BAS will send acknowledge for multiple hours to not send
- Look into the need for real time clock
Alarms - not just the state but also Alarm code with device and object identifier
- Makes sense to Reis
Peer to Peer value sharing - Being able to write sensor values from one controller to another or multiple.
- Makes sense to Reis
- Ideally I could see this being used for sensors that we don't have then allowing us to do more control with less IO