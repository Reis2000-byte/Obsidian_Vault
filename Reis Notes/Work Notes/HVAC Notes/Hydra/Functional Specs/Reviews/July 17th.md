### Must Fix

**1. Circuit, Available (1.2.8) is undefined.**  DONE
It's just a header with a Requirement Key and no body — no logic at all. But Lead Circuit (1.2.6), the entire lead/lag circuit selection algorithm, depends entirely on "Circuit A Available" / "Circuit B Available." This is a hard blocker, not a nice-to-have.

**2. Compressor, Ready (1.3.1) — AC units may never be allowed to run.**  DONE
The table lists this as a required condition: _"Reversing Valve X, Active is TRUE AND Heat Pump, Selection, Type = HP" → Value: TRUE_. As literally written, this is a single boolean expression that must equal TRUE. For an AC-only unit (Heat Pump, Selection, Type = AC), "Type = HP" is always false, so the whole ANDed expression can never be TRUE — meaning Compressor, Ready could never assert on a straight AC unit. This needs to be reframed as conditional ("only applies when Type = HP; otherwise not evaluated"), not as an always-required AND.

**3. Outdoor Fan, Output, Staged Control (1.5.5) contradicts its own note.**  DONE
The Dual Stage table uses Lead Circuit-based staging (100%→both fans, 50%/Lead A→Fan A only, 50%/Lead B→Fan B only) — the same pattern as the Compressor output table. But the note directly underneath it says: _"Fan A follows Circuit A's capacity, Fan B follows Circuit B's capacity... No Lead Circuit involved."_ This also contradicts Outdoor Fan, Capacity Request (1.5.2), which is defined per-circuit with no Lead Circuit concept at all. This looks like a copy-paste from the Compressor Output table that wasn't updated. Pick one model and make both sections agree.

**4. Blower Stage thresholds are non-monotonic.**  DONE
Stage 2 default = 5°F (min 5), Stage 3 default = 4°F. If these represent increasing capacity-staging thresholds (as the Compressor thresholds correctly do: 0 → 4 → 10), Stage 3 should not have a _lower_ default than Stage 2. Compare directly against the Compressor thresholds (3.3.7–3.3.9), which are correctly increasing.

**5. "Variable Speed" compressor type has no control logic.**  DONE
Compressor, Selection, Type (3.3.5) defines three types, including Variable Speed ("should always start first," adjusts speed dynamically). But Compressor, Capacity Request (1.3.2) and Compressor, Output, Staged Control (1.3.5) only define behavior for Single Stage and Dual Stage. There's no requirement anywhere describing how a Variable Speed compressor's capacity request or output is computed.

**6. Several internally-referenced sub-requirements are missing entirely.**  DONE
These aren't external/out-of-scope items — they're specific to this document's own subsystems and are referenced by other requirements in this same doc, but never defined anywhere:

- **Circuit, Available** (see #1)
- **Two Circuit System, One Circuit Defrost** / **One Circuit Lockout**
- **Manual Lead Selection**
- **Auto Switching, Two Circuit System, Fixed Speed Compressor**
- **Compressor, Output, Two Compressors with One Stage** — this matters because it's unclear whether "Relay, Compressor A1 / B1" represents two stages of _one_ compressor on a single circuit, or two _separate_ compressors on two circuits. The one defined table (1.3.5) only models the single-circuit, two-stage case.
- **Blower rundown/off-delay** — "Blower, Lockin, Rundown," "Blower, Rundown Timer Start," "Blower, Rundown Timer Reset" are all referenced from Blower, Ready/Active/State, but there is no section anywhere defining blower rundown behavior, and it isn't even included as an input to Blower, Lockin (1.1.8.1), which only lists Minimum Runtime and Dedicated Heat Lockin.

**7. Blower/Dedicated Heat race condition can force an emergency stop instead of a normal shutdown.**  DONE
Blower, Request drops the instant Dedicated Heat, Request drops (1.1.2). Once Blower, Request is false, Blower can transition Running→Stopping immediately (1.1.4). But Dedicated Heat, Emergency Stop (1.4.7.5) fires whenever "Selection = Electric AND Blower State is NOT Running" — and Blower leaves the Running state as soon as it starts stopping. So a normal end-of-call sequence can trip an _emergency_ stop on the electric heat instead of a controlled Stopping transition. The Blower needs some kind of lock-in tied to Dedicated Heat's own shutdown sequence (beyond just Dedicated Heat, Lockin) to prevent this.

**8. Gas dedicated heat has no blower-failure protection.**  DONE
The "Blower not Running" trip in Dedicated Heat, Emergency Stop (1.4.7.5) is explicitly scoped to `Selection, Source = Electric`. If the blower fails while a gas furnace is firing, nothing in this control logic forces a stop — you're relying entirely on the furnace's own board. Worth confirming that's the intended split of responsibility.

**9. No timeout on the Gas Dedicated Heat proving path.**  
Electric heat has an explicit Proving Time before Active asserts. Gas heat's Active condition (1.4.3) is just "Furnace PCB Feedback is TRUE" — no timeout. If the furnace never asserts feedback (failed ignition, wiring fault, etc.), Dedicated Heat, State sits in Starting indefinitely with no path to Lockout.

**10. Circuit, Capacity Request (1.2.2) leaves Lead Circuit's own value undefined in single-circuit-only modes.**  DONE
In the "Circuit A Only" / "Circuit B Only" rows, the table specifies the Lag circuit is 0%, but leaves the Lead circuit's Capacity Request as "-" — never actually defining what the running circuit's request should be.

**11. Circuit, Run Inhibit (1.2.10) appears to be an orphaned requirement.**  DONE
It's calculated, but nothing in the document consumes it — it doesn't feed Circuit, Ready, Circuit, State, or Circuit, Lockout. Either it's dead logic that should be removed, or there's a missing link where it should be gating something.

**12. Start-up Delay, Lockout (2.4) isn't actually wired into Circuit, Ready.**  DONE
Section 2.4 defines a fairly detailed behavior (random vs. user delay, blackout/brownout/overvoltage triggers, override by operating mode). But Circuit, Ready (1.2.1) — the actual gate in the Circuit state machine — only checks "Restart Delay, Duration has elapsed since system power up." It ignores the Random mode path, the override condition, and the brownout/overvoltage trigger events entirely.

**13. Defrost timers use exact equality instead of a threshold comparison.**  DONE
Defrost Mode, Switch (2.5.1), Mode Timer Reset/Stop (2.5.2, 2.5.4) all trigger on the timer being _"Equal to"_ the max value. If any tick or scan-cycle timing causes the counter to skip past that exact value, the trigger never fires — the unit could run in Heating with a frosted coil indefinitely (or hang in Defrost). Recommend `>=` instead of exact equality throughout.

**14. Defrost, Compressor Run Time reset condition is vulnerable to switch chatter.**  DONE
It resets to 0 any time "Heating AND Switch, Defrost is Open" (2.5.5). A defrost switch chattering near its threshold (common in the field) would repeatedly zero out accumulated compressor run time, potentially preventing the unit from ever reaching the run-time threshold needed to force a defrost cycle.

**15. Reversing Valve "Hold Previous Value" (Idle mode) has no defined initial/power-on state.**  DONE
1.6.2 says Idle mode holds the previous Request value — but there's no previous value at first power-up before any Heating/Cooling call has occurred. Needs an explicit default.

**16. Dedicated Heat, Capacity Request ambiguous for 2-circuit systems.**  
It references a single "Circuit, Operating Mode" (1.4.4), but with Number of Circuits = 2, circuits can be in different modes simultaneously (e.g., one heating, one idle/defrosting). Which circuit's mode governs backup heat activation isn't specified.

### Improvements

- **Blower, Output, Speed Tap (1.1.5) vs. Blower, Capacity Request (1.1.7):** the Y2&O / Y1&O rows require "AND HP Unit" in the capacity-request table but not in the speed-tap table. Reconcile so behavior doesn't differ by blower motor type for the same thermostat inputs. **DONE**
- **Blower, Output, Variable Voltage (1.1.6):** the torque intermediate (Tmin/Tmax) algebraically cancels out of the final voltage formula — it reduces to a plain linear map from Capacity% to Voltage. If ECM torque-voltage response is meant to be non-linear, this formula doesn't capture that; worth confirming intent.
- **Crankcase Heater, Active (2.6.1)** keys off "Compressor, Active = FALSE" rather than compressor State — during the brief Starting/proving window, Active is false but the compressor is already energized, which could theoretically allow both simultaneously. **DONE**
- **Naming inconsistencies:** "Defrost, Compressor Run Timer" (2.5.1) vs. "Defrost, Compressor Run Time" (2.5.5–2.5.7); "Compressor, Defrost,Run Time Maximum Value" missing a space. **DONE**
- **Dedicated Heat, Lockin, Minimum Runtime (1.4.7.3)** uses an edge-triggered "on transition to Running, latch for duration" style, while the equivalent Blower (1.1.8.4) and Compressor (1.3.9.2) versions use a continuous "Active has been true for less than duration" style. Functionally similar, but inconsistent — worth standardizing. **DONE**
- **Blower, Capacity, Setpoint Table (3.2.10)** and similar "(configurable)" placeholders lack Default/Min/Max values that every other config parameter in the doc has. **DONE**
- **Compressor, Scale Factor, Run Hour (3.3.10):** purpose and consumer aren't clear from what's in this document — worth clarifying how it actually factors into lead/lag decisions. **DONE**
- **Blower Motor table (3.2.9):** the 2.3 HP motor has a lower max torque (132 oz-ft) than the 1.7 HP motor (138 oz-ft) — not necessarily wrong, but worth double-checking against datasheets. **DONE**
- **Circuit, Operating Mode / Defrost Mode, Switch** have a mutual dependency (Defrost checks "currently Heating"; Heating exits when Defrost becomes true). Probably fine as a once-per-scan state machine, but an explicit note on evaluation order would help implementers avoid treating it as a live algebraic loop. DONE

If it'd help, I can also go through the AUTOGEN test case references and the embedded "flagged issue" annotations in the document itself (there are several, like the ones on Blower Ready and Compressor Output) and cross-check whether my independent findings above line up with those — some clearly do (e.g. #3 and #7).