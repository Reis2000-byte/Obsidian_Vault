# MUST FIX

## 1. A2L Mitigation is Incomplete

This is the largest gap.

Your spec defines:

- A2L Leak Alarm
- A2L Mitigation Hold
- A2L Mitigation Delay

But it never actually defines:

> What is A2L Mitigation Active?

Yet multiple critical functions depend on it:

- Compressor Emergency Stop
- Dedicated Heat Emergency Stop
- Blower Request

all reference:

> A2L Mitigation Active = TRUE [[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf), [[S4900191 D...Rev 0.pdf | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/S4900191%20Daikin%20A2L%20Nighthawk%20Mini%20Product%20Specification,%20Rev%200.pdf.pdf)

### Missing Requirement

You need an actual requirement similar to:

Plain Text

A2L Mitigation Active shall be TRUE when:

A2L Leak Alarm is TRUE

OR

A2L Mitigation Hold is ACTIVE

Without this your software engineer can implement three completely different behaviors and still claim compliance.

---

# 2. A2L Airflow Requirement Is Not Captured

The Nighthawk requires:

- indoor fan ON
- minimum airflow
- compressor OFF
- heat OFF
- continue for 5 minutes after alarm clears

[[S4900191 D...Rev 0.pdf | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/S4900191%20Daikin%20A2L%20Nighthawk%20Mini%20Product%20Specification,%20Rev%200.pdf.pdf)

Your spec currently provides:

- blower request during mitigation
- compressor shutdown
- dedicated heat shutdown

But does NOT define:

### Missing

What blower speed should be forced?

Example:

Plain Text

When A2L Mitigation Active is TRUE:

Blower Capacity Request = 100%

or

Plain Text

Blower Capacity Request = Configured A2L Airflow %

Right now blower runs, but airflow requirement is not guaranteed. [[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf), [[S4900191 D...Rev 0.pdf | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/S4900191%20Daikin%20A2L%20Nighthawk%20Mini%20Product%20Specification,%20Rev%200.pdf.pdf)

---

# 3. Defrost Exit Criteria Not Fully Defined

Current inputs:

Plain Text

Defrost Switch Opens

Timer Expires

Transition to Cooling

reset the timer. [[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

But you never explicitly say:

Plain Text

Circuit Operating Mode shall exit Defrost

You infer it through the state machine.

### Requirement Needed

Plain Text

Circuit Operating Mode shall transition from Defrost

when any Defrost termination condition occurs.

Makes traceability much cleaner.

---

# 4. Defrost Timer Can Get Stuck

Current logic:

Defrost Timer:

Start:

- Compressor Active
- Defrost Mode

Stop:

- Compressor inactive
- Not Defrost
- Timer max

[[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

Potential problem:

You say timer resumes where it left off.

You don't specify what happens if:

Plain Text

Defrost starts

Compressor trips

Compressor never returns

Timer stops forever.

State remains Defrost.

### Fix

Explicit Defrost timeout independent of compressor operation.

---

# 5. Reversing Valve Hold Previous Value

This one worries me.

Current requirement:

Plain Text

Idle = Hold Previous Value

[[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

Scenario:

Plain Text

Cooling

Idle

Service call

Power brownout

Unexpected restart

Valve state becomes dependent on history.

You now have latent state.

### Recommendation

Use:

Plain Text

Idle = Heating Position

unless there is a strong efficiency reason.

I generally avoid "hold previous state" whenever refrigerant flow direction is involved.

---

# 6. Outdoor Fan Lockin Logic Conflict

Current:

Plain Text

Outdoor Fan Lockin TRUE

when Circuit Active TRUE

and not Defrost

[[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

But:

Outdoor Fan Capacity Request:

Plain Text

Defrost = 0%

``

[[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

You are mixing:

- request logic
- state lockin logic

I think it works.

But I had to analyze it for several minutes.

If a reviewer has to think that long, the spec needs improvement.

---

# 7. Defrost Suspend/Resume Missing

Legacy defrost board includes suspend behavior. [[S4900191 D...Rev 0.pdf | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/S4900191%20Daikin%20A2L%20Nighthawk%20Mini%20Product%20Specification,%20Rev%200.pdf.pdf)

Example:

Plain Text

Y removed during Defrost

Legacy board:

- stop compressor
- stop RV
- stop auxiliary heat
- preserve timer
- resume later

Your spec doesn't define behavior.

Today software could:

Option 1

Plain Text

Exit defrost

Option 2

Plain Text

Suspend defrost

Both satisfy current wording.

Need explicit requirement.

---

# 8. Power Cycle Behavior Missing

Legacy board explicitly defines:

- timer retention
- timer reset
- anti-short-cycle behavior

during power loss. [[S4900191 D...Rev 0.pdf | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/S4900191%20Daikin%20A2L%20Nighthawk%20Mini%20Product%20Specification,%20Rev%200.pdf.pdf)

Your spec only has Startup Delay Lockout. [[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

Missing:

Plain Text

What happens to:

- Defrost Timer

- Defrost Runtime

- Lead/Lag position

- Runtime counters

- Start counts

after power loss.

---

# 9. A2L Fault Behavior Not Fully Mapped

Nighthawk treats:

- Sensor Comm Failure
- Sensor Fault
- Relay Failure

ALL as mitigation events. [[S4900191 D...Rev 0.pdf | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/S4900191%20Daikin%20A2L%20Nighthawk%20Mini%20Product%20Specification,%20Rev%200.pdf.pdf)

Your spec appears to assume:

Plain Text

External A2L board tells me mitigation active.

But doesn't define behavior if:

Plain Text

A2L input wire opens

or

Plain Text

A2L communications lost

If your PCB directly owns A2L functionality, this is a major hole.

---

# 10. Low Pressure Trip Has No Debounce

Current logic:

Plain Text

Switch Open

AND

Inhibit inactive

=

Trip

[[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

Instantaneous trip.

Real switches bounce.

Need:

Plain Text

Low Pressure Trip Delay

Even 3-5 seconds.

Otherwise nuisance trips are likely.

---

# SHOULD IMPROVE

## 1. Define State Evaluation Rate

Every state machine should have:

Plain Text

evaluated every 100ms

or

Plain Text

once per main loop

or equivalent.

Currently not defined.

---

## 2. Add State Transition Diagrams

The requirements are solid but long.

Add diagrams for:

- blower
- circuit
- compressor
- defrost

Review time drops dramatically.

---

## 3. Add Termination Reason Variables

For diagnostics:

Plain Text

Defrost End Reason

Values:

Plain Text

Switch Open

Timeout

Cooling Transition

Power Reset

Very useful in the field.

---

## 4. Define Event Priority During A2L

Example:

Plain Text

Defrost running

A2L Leak occurs

Who wins?

I assume:

Plain Text

A2L wins

but it's not explicitly stated.

Need priority table.

---

## 5. Clarify Two-Circuit Defrost

You correctly prohibit simultaneous circuit operation during defrost. [[R4J-LCPCB_...012-140602 | PDF]](https://goodmanglobal-my.sharepoint.com/personal/fredrrx_daikincomfort_com1/Documents/Microsoft%20Copilot%20Chat%20Files/R4J-LCPCB_20261012-140602.pdf)

However:

Plain Text

Circuit A enters defrost

Circuit B waiting

Need to define:

Plain Text

Can B accumulate runtime?

Can B request defrost?

Can B queue defrost?