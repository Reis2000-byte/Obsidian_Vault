### ASC vs AAC vs BC

| Profile | Meaning                         | Typical Use                           |
| ------- | ------------------------------- | ------------------------------------- |
| B-SS    | Smart Sensor                    | Temperature sensor                    |
| B-SA    | Smart Actuator                  | Valve or damper actuator              |
| B-ASC   | Application Specific Controller | VAV, FCU, RTU controller              |
| B-AAC   | Advanced Application Controller | Programmable equipment controller     |
| B-BC    | Building Controller             | Supervisory/building-level controller |
### Feature Mapping

| Feature                             | B-ASC   | B-AAC | B-BC |
| ----------------------------------- | ------- | ----- | ---- |
| Read Property Multiple              | ✅       | ✅     | ✅    |
| Write Property Multiple             | ✅       | ✅     | ✅    |
| Who-Is / I-Am                       | ✅       | ✅     | ✅    |
| Who-Has / I-Have                    | ✅       | ✅     | ✅    |
| Device Communication Control        | ✅       | ✅     | ✅    |
| Reinitialize Device                 | ✅       | ✅     | ✅    |
| Acknowledge Alarm                   | ❌       | ✅     | ✅    |
| Alarm/Event Generation              | ❌       | ✅     | ✅    |
| Alarm Codes & Object References     | ❌       | ✅     | ✅    |
| Peer-to-Peer Value Sharing          | Limited | ✅     | ✅    |
| Writing Values to Other Controllers | Limited | ✅     | ✅    |
| Scheduling                          | ❌       | ✅     | ✅    |
| Trending                            | ❌       | ❌     | ✅    |
| Backup / Restore                    | ❌       | ❌     | ✅    |
B-ASC is primarily a data-sharing controller. It is designed to expose points, accept writes, answer discovery requests, and respond to communication management services, but it does **not require alarming, scheduling, trending, or advanced controller functionality**.

### B-ASC:

- RPM
- WPM
- Who-Is/I-Am
- Who-Has/I-Have
- Reinitialize Device
- Device Communication Control
- Exposed Objects
### B-AAC:

- Alarm generation
- Alarm acknowledgment
- Alarm state tracking
- Peer-to-peer control between controllers
### B-BC:

- Trend Logs
- Schedules
- Backup & Restore
- More supervisory functions

### B-ASC Behavior

![[Pasted image 20260902141843.png]]

- If multiple controllers are using the 