# Must Fix

These are items that I think will either result in undefined behavior, contradictory behavior, implementation ambiguity, or field issues.

---

## 1. Input prioritization conflicts with thermostat-based control logic

Section 3 says:

- Most recent valid input wins.
- Maintain last complete valid instruction set.
- HMI overrides thermostat.

However nearly all actuator logic later is driven directly from:

- W1/W2
- Y1/Y2
- O
- G

There is no clear mapping between HMI setpoint control and the later state machines.

### Problem

If Operating Mode Source = Setpoint Calculation:

- How is Unit Capacity Request generated?
- How are Y1/Y2 equivalents generated?
- How does blower capacity get selected?
- How does compressor staging occur?

The document mixes thermostat-based demand control and communicating thermostat/setpoint control but only fully defines the thermostat path.

### Fix

Create an explicit internal abstraction layer:

- Unit Operating Mode
- Heating Capacity Request
- Cooling Capacity Request
- Fan Request
- Dehumidification Request

All actuator logic should consume those internal requests rather than raw thermostat inputs.

---

## 2. "Most Recent Input Wins" is dangerous and underspecified

Section 3.7:

> System shall act based on most recent valid input received.

Example:

BACnet writes Cooling.

Five minutes later App writes Fan Only.

Then Modbus writes setpoint.

What is the active instruction set?

This will create race conditions.

### Fix

Specify ownership model:

Example:

Priority:

1. Modbus HMI
2. BACnet
3. App
4. Thermostat

or

last-write-wins per object

Currently impossible to implement consistently.

---

## 3. Unit Operating Mode does not define conflicting thermostat calls

Examples:

- W1 and Y1 active simultaneously
- W2 and Y2 active simultaneously
- O active without Y
- W1 + O + Y1

There are real thermostats that may briefly overlap outputs.

### Fix

Add explicit conflict resolution table.

---

## 4. State machines frequently transition Stopping → Locked Out

Example:

- Blower
- Circuit
- Compressor
- Dedicated Heat
- Outdoor Fan
- Reversing Valve

Current logic:

Stopping → Active FALSE → Locked Out

That is wrong semantically.

Stopping is a normal operation.

Locked Out implies fault condition.

### Current Result

Normal shutdown causes state:

Running → Stopping → Locked Out → Not Ready → Ready

every cycle.

### Fix

Change:

Stopping → Ready

or

Stopping → Not Ready

Do not enter Locked Out unless an actual lockout condition exists.

This is probably the biggest state-machine problem in the document.

---

## 5. Compressor Ready requires Reversing Valve Active

Section 8.3.1

For HP:

Compressor Ready requires Reversing Valve Active TRUE.

But Reversing Valve Active only becomes TRUE after valve proving.

Potential startup deadlock depending on valve state machine execution order.

### Fix

Define start sequence explicitly:

1. Determine mode
2. Command reversing valve
3. Wait proving delay
4. Declare RV active
5. Permit compressor start

---

## 6. Reversing Valve state machine can stick

State machine requests:

Ready → Starting only when Request TRUE.

What if:

- valve running cooling
- request removed
- must return heating

There is not a clean, obvious path for the opposite transition.

The valve state machine was modeled like a motor/compressor.

A reversing valve is actually a positioning device.

### Fix

Model:

- Requested Position
- Actual Position
- Position Change In Progress

rather than Running/Stopping.

---

## 7. Defrost logic may create circular dependency

Defrost Mode Switch requires:

Circuit Operating Mode = Heating

But Circuit Operating Mode can become Defrost when Defrost Mode Switch TRUE.

Need to verify no oscillation.

### Recommendation

Explicitly state:

> Defrost entry shall only be evaluated while current mode is Heating.

---

## 8. Low-pressure trip appears permanent

Section 9.3 trips compressor.

I do not see:

- auto reset behavior
- retry count
- manual reset behavior
- diagnostic latching

### Fix

Define recovery strategy.

This is essential for field operation.

---

## 9. Outdoor fan lockin can create permanent fan operation

Outdoor Fan Lockin:

TRUE whenever:

- Circuit Active TRUE
- Not Defrost
- No lockout

As written, it may never release until circuit becomes inactive.

Need explicit stop criteria.

---

## 10. Dedicated heat + heat pump interaction unclear

Examples:

HP heating call active.

W1 becomes active.

Questions:

- Is supplemental heat allowed?
- Does compressor continue?
- Is dual-fuel supported?
- Is compressor locked out?

Not fully defined.

### Fix

Need full heat-source arbitration table.

---

## 11. Defrost timers have ambiguous pause/resume behavior

One section says:

> resume from where it left off

Others imply reset conditions.

Not always clear whether timers:

- reset
- pause
- hold
- resume

### Fix

For every timer define:

- start condition
- stop condition
- reset condition
- retention behavior

---

## 12. Sensor failure strategy is largely missing

Numerous sensors exist:

- Outdoor Air
- Coil
- Suction
- Humidity
- A2L

But very little defines behavior when invalid.

### Example

Outdoor air sensor invalid:

- Crankcase heater disabled?
- Enabled?
- Alarm?

### Fix

Define degraded mode operation.

---

## 13. Circuit Operating Mode and Unit Operating Mode can disagree

Circuit mode derives from unit mode plus requests.

Defrost can override.

There are cases where:

Unit = Heating

Circuit A = Defrost

Circuit B = Idle

Need explicit statement regarding system status reporting.

Otherwise BAS integrations become inconsistent.

---

## 14. BACnet is listed as an input source but not defined

Section 3 lists BACnet.

No definition of:

- writable points
- ownership
- priority
- failure behavior

Need at least a high-level BACnet command model.

---

## 15. "Always FALSE" lockouts eliminate future extensibility

Examples:

- Blower Lockout
- Outdoor Fan Lockout
- Reversing Valve Lockout

Later logic references these heavily.

Either:

- remove them entirely

or

- define possible future fault conditions.

Right now they're dead logic.

---

# Would Make the Spec Better

These are quality improvements rather than true logical errors.

---

## 1. Add a system architecture layer

I'd strongly recommend adding:

Plain Text

1

Inputs

2

↓

3

Request Generation

4

↓

5

Capacity Request Generation

6

↓

7

Equipment Arbitration

8

↓

9

State Machines

10

↓

11

Outputs

Show more lines

Right now the logic is scattered.

---

## 2. Add sequence diagrams

Especially for:

- cooling start
- cooling stop
- heating start
- heat pump mode change
- defrost

This would eliminate interpretation differences.

---

## 3. Add power-up behavior section

Define:

- state initialization
- counters
- lead circuit selection
- stored values
- restoring previous state

---

## 4. Add communications failure behavior

For:

- Modbus loss
- BACnet loss
- HMI loss

Questions:

- Continue running?
- Revert to thermostat?
- Shutdown?

Not specified.

---

## 5. Define fault prioritization

A matrix would help:

|Fault|Alarm|Lockout|Emergency Stop|
|---|---|---|---|
|Smoke|Yes|Yes|Yes|
|High Pressure|Yes|Yes|Yes|
|Low Pressure|Yes|Yes|Yes|
|Sensor Failure|Yes|?|?|

---

## 6. Define alarm persistence

Should alarms:

- auto clear?
- latch?
- require power cycle?
- require service reset?

---

## 7. Define all state transition timing

Many state changes rely on scan execution order.

Documenting timing assumptions would help firmware implementation.

---

# Overall Assessment

For a Rev 0 control specification, this is actually quite good. The overall architecture of:

- Unit
- Circuit
- Compressor
- Blower
- Dedicated Heat
- Outdoor Fan
- Reversing Valve

and the lockin/lockout/emergency-stop philosophy is very solid.

The biggest issues I would address before implementation are:

1. **Stopping → Locked Out state machine behavior**
2. **Communicating thermostat/HMI path not fully integrated into the control model**
3. **Input ownership ("most recent wins") ambiguity**
4. **Heat pump/reversing valve sequencing**
5. **Missing fault recovery definitions**
6. **Undefined sensor-failure behavior**

Those are the items most likely to generate firmware defects, test failures, or field behavior disagreements between controls engineers