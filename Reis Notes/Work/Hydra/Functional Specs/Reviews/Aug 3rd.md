
## Must fix

#### Deadlocks and unreachable logic

**1. Heat pump heating can never start.** `Compressor, Ready` (8.3.1) requires `Reversing Valve X, Active = TRUE` when the unit is a heat pump. `Reversing Valve, Active` (8.6.4) requires `Reversing Valve, State is Starting`. The RV state machine only reaches Starting via `Reversing Valve, Request` (8.6.5), and Request is **FALSE in Heating** (8.6.2). So in heating the RV sits in Ready, never becomes Active, and the compressor never becomes Ready. The root cause is modeling a two-position valve as a start/stop actuator. Model it as a _commanded position_ + _position confirmed_, and make the compressor precondition "RV position confirmed and matches the commanded position," not "RV is running."

**2. `Active` has no FALSE condition for Compressor, Outdoor Fan, or Dedicated Heat (electric).** 8.3.4, 8.5.4, and 8.4.3 define Active only as "TRUE when Proving Time has elapsed since transition to Starting." Nothing ever makes it FALSE. Every state table exits `Stopping` and `Emergency Stopping` on "Active is FALSE" — so once a compressor runs, it can never leave Stopping or Emergency Stopping. Permanent hang, including on a safety trip. Blower is the only actuator with a symmetric definition (8.1.3); mirror that pattern everywhere.

**3. Outputs are undefined in every state except Starting/Running.** 8.1.5 and 8.1.6 say "when State is Starting OR Running... energize." 8.3.5 and 8.4.5 key the output tables on Capacity Request, and Capacity Request is not state-dependent — so in Emergency Stopping with a 100% request, no row matches and the relay state is undefined. Add an explicit, top-priority rule to each output section: _all associated outputs shall be de-energized whenever State is not Starting or Running._ This also gives item #2 its falling edge.

**4. Defrost cannot sustain.** `Defrost Mode, Switch` (9.5.1) requires `Defrost, Compressor Run Time ≥ Max`. But 9.5.5 resets that run time to 0 on "Circuit Operating Mode transitions from Heating to Defrost." So the instant defrost is entered, the entry condition is destroyed, Defrost Mode Switch goes FALSE, and the circuit exits per 8.2.5. The result is chatter. You clearly intended `Defrost, Mode Timer` (12 min max) to be the defrost _duration_ — but nothing in 9.5.1 or 8.2.5 references it. Define defrost exit explicitly: terminate on switch-open (debounced) OR Mode Timer ≥ max.

**5. The compressor cannot start in defrost.** `Compressor, Ready` requires `Outdoor Fan, State = Running`. During defrost, Outdoor Fan Capacity Request is forced to 0% (8.5.2), so Request is FALSE and the fan is not Running. Defrost requires the compressor. Add a defrost exception to the fan precondition.

**6. Every reversing valve switch costs ~3 minutes of compressor off-time.** To switch the RV, `Switching Inhibit, Pressure Equalization` (8.6.6) requires `Circuit, Active = FALSE`, i.e. the compressor must stop. Stopping the compressor arms `Compressor, Lockout, Anti Short Cycle` = 180 s (8.3.9.4). So defrost entry and defrost exit each stall for 3 minutes. Add an explicit ASC bypass (or a separate, shorter defrost-transition delay) for compressor stops caused by defrost/RV switching.

**7. A W-only heat call can never start (circular dependency).** `Dedicated Heat, Ready` (8.4.1) requires `Blower, State = Running`. `Blower, Request` (8.1.2) is TRUE only for a circuit Starting/Running, a G call, or Dedicated Heat already Starting/Running/Stopping. On an AC unit with a W1 call and no G, the circuit capacity is 0%, so nothing requests the blower, so dedicated heat never becomes Ready. Many commercial thermostats do not assert G on a heat call. Add "Unit Dedicated Heat Capacity Request > 0%" to Blower, Request.

**8. Heat pump cooling is unreachable in §5.** The Unit Operating Mode table is "prioritized in the order listed," and the row `O Terminal Heat Pump | Y1/Y2 → Heating` sits _above_ `O Terminal Heat Pump | Y1/Y2 and O → Cooling`. Y+O always matches the heating row first. Reorder so the more specific criteria evaluate first, or make each row mutually exclusive (`Y AND NOT O`).

**9. `Lead Lag = Circuit B Only` with `Number of Circuits = 1` is self-contradictory.** Lead Circuit (8.2.6) priority 1 forces Circuit A when there is one circuit; Circuit Run Inhibit (8.2.10) inhibits Circuit A when the mode is B Only. Circuit A is simultaneously the lead and inhibited — the unit never runs. Either invalidate the combination at configuration time or add a precedence rule.

#### Contradictory or ambiguous tables

**10. §7 Unit Capacity Request — overlapping rows with no priority.** The two heat-pump tables say "multiple combinations can be active" but give no evaluation order. With Y1+Y2+O asserted, both `Y1 and Y2 → Mechanical Heat 100%` and `Y1, Y2, and O → Cooling 100%` match. With W1+Y1 asserted, heating and mechanical heat both match. Also, `-` in the Mechanical Heat column is undefined: does it mean don't-care, unchanged, or 0%? Add explicit priority and a notation legend.

**11. 50% circuit capacity is unreachable with two circuits.** In 8.2.2, every `50%` row maps to Lead = 100%, Lag = 0%. So with `Number of Circuits = 2`, `Circuit, Capacity Request` is only ever 0% or 100%, which makes the `50%` row of the Dual Stage compressor table (8.3.2) dead code. A dual-stage compressor loses its low stage entirely in 2-circuit configs but keeps it in 1-circuit configs. Decide which is intended and make it consistent.

**12. No output exists for compressor stage 2.** §12.1 defines only `Relay, Compressor A1` and `Relay, Compressor B1`. 8.3.5 then uses those two relays to represent _unit_ staging across circuits (100% → both A1 and B1), while `Compressor, Selection, Type = Dual Stage` implies a second stage on a single compressor. There is no A2/B2. As written, a dual-stage compressor on a single-circuit unit cannot command its high stage. Add the outputs, or restate 8.3.5 as a per-compressor table and handle circuit staging in 8.2.2 only.

**13. Overlapping rows in Dedicated Heat dual stage (8.4.4).** `<=50` and `0%` both match a 0% request, and `<=50` is missing a unit. Use `>0% AND ≤50%`.

**14. Typo with functional impact:** 8.1.7 priority 8 maps `Y1` to **High Cool %**. It should be Low Cool %, per the speed-tap table in 8.1.5.

**15. `Blower, Request` omits `Circuit, State = Stopping`.** It includes Dedicated Heat Stopping but not circuit Stopping. When a circuit enters Stopping while the compressor is held by `Compressor, Lockin, Minimum Runtime`, the blower request drops, the blower stops, and `Compressor, Emergency Stop` fires (8.3.9.6, "Blower Active FALSE AND compressor Starting or Running"). A normal shutdown produces a spurious emergency stop.

#### Safety and fault handling

**16. The A2L leak alarm does nothing.** 9.1 defines it and nothing else in the document references it. For an A2L system this is the single most important interlock: leak detection must force the indoor blower to a mitigation airflow, de-energize the compressor, inhibit RV switching, and lock the unit out until reset. Also, `A2L Mitigation Feedback` is referenced but never defined as an input, and 11.5.4 `Sensor, A2L` is an empty heading.

**17. No fault latching, retry limits, or reset behavior anywhere.** High pressure, low pressure, phase, smoke, and float all clear the moment the switch closes, so the unit free-cycles on a hard fault. Industry practice — and most listing agencies — expect a strike count (e.g., 3 trips in an hour → hard lockout) and a defined reset path (power cycle, thermostat cycle, or service reset). Define trip counters, lockout persistence, and reset for each protection.

**18. The crankcase heater fails unsafe.** 9.6.1 requires `Temp, Outdoor Air, Valid Range = VALID`. On sensor failure the heater is never energized — the opposite of what you want, since the heater's job is preventing refrigerant migration. Default to energized on invalid sensor. Also, `Compressor X, Active — NOT Starting AND NOT Running` is a type error: Active is a boolean, Starting/Running are states. It should reference `Compressor, State`.

**19. No behavior defined for loss of communications.** §3 and §4 build the entire control path on a communicating HMI but never say what happens when it drops mid-cycle — revert to 24 V inputs, hold last command, or shut down? Add a heartbeat/timeout requirement and a defined fallback. Related: §3 item 2 prohibits simultaneous 24 V thermostat + Modbus HMI, then item 3 specifies behavior for exactly that case. And item 7 ("act on the most recent valid input from any source") directly conflicts with item 3's priority scheme and leaves BACnet vs. Phone App vs. HMI arbitration as a race condition.

**20. Speed taps need mutual exclusion and break-before-make.** 8.1.5 says "energize per the table" but never says the other taps are de-energized, and there is no dead time between tap changes. Energizing two taps on a multi-speed motor can damage the windings. State it explicitly.

#### Unspecified scope

**21. The `Setpoint Calculation` mode has no logic.** 10.1.6 offers it and §4 describes HMI setpoints, AUTO changeover, RIAT, humidity, and dehumidification — but §5 and §7 are both explicitly scoped to "when Operating Mode Source is configured to 24V Thermostat." There is no requirement anywhere that turns a setpoint and a space temperature into a capacity request. No deadband parameters, no staging thresholds, no stage-up/stage-down timers, no changeover logic, no dehumidification control. This is roughly half the product's behavior and it's currently undefined.

**22. Orphaned configuration parameters.** Nothing in the document references: Dedicated Heat Stage 1/2/3 Thresholds (10.4.1–10.4.3 — and Stage 3 has no corresponding output or selection type), `Defrost, Selection, Method = Temperature` (no temperature-based defrost logic exists), `Switch, Dirty Filter`, or `Indoor Humidity Setpoint` / `Dehumidification Yes/No`.

**23. Empty requirement sections.** 11.4.3, 11.5.1–11.5.4, 11.6.3–11.6.10, 11.7.1–11.7.2 are headings with no content. Either populate them or mark them explicitly "reserved — not used in this release" so a reviewer can tell intent from omission.

**24. Terminology collisions that will cause implementation bugs.**

- `HP` (8.3.1, 9.5.1) is not a defined value; 10.1.2 defines only `AC` and `O Terminal Heat Pump`. 9.2 uses the long form. Pick one.
- 8.2.7 priority 2 transitions to `Lockout`, which is not a state — the state is `Locked Out`.
- `Blower Variable Voltage Output` (8.1.6) vs. `10V, Blower Control` (10.2.1) are the same selection with two names.
- 8.3.9.5 compares the relay against `Reversing Valve, Request`, while 8.6.4 compares it against `Reversing Valve, Output`. Those differ during the pressure-equalization hold.
- ACTIVE/INACTIVE and TRUE/FALSE are used interchangeably for booleans, while §2 also uses INACTIVE/ACTIVE to classify _states_. Reserve one vocabulary for each.

## Would make the spec better

**Traceability.** §1 defines formatting conventions but requirements have no unique identifiers — only section numbers, which renumber every time you insert a requirement. Add stable IDs (e.g. `SYS-BLR-0120`) so test cases, ECNs, and review comments survive edits.

**Notation legend.** Define `-`, `Any`, and blank cells once, up front. Today `-` means "don't care" in some tables and "no criteria matched" in others, and you can't tell which without inference.

**Normative vs. explanatory text.** §8.2.11 ("Locks are safety and timing rules…", "The big picture:") is rationale embedded in a shall-section, and it restates 8.2.11.1–8.2.11.3 in a slightly different way. Rationale is valuable — move it to a non-normative appendix or clearly marked Note blocks so there's exactly one authoritative statement per behavior.

**Complete the parameter tables.** Min/Step/Max are bullets or dashes in 10.1.7, 10.1.10, 10.2.3–10.2.5, 10.3.1–10.3.3, 10.4.1–10.4.7, 10.5.1, 10.6.1–10.6.3, 10.7.1. Also add cross-parameter constraints (High Cool % ≥ Low Cool %, etc. — all five blower setpoints default to 50%, which makes staging a no-op out of the box) and a timer resolution/tolerance requirement.

**Input conditioning.** You debounce Operating Mode and the defrost switch, but nothing else. `Switch, Blower Proving` in particular feeds `Compressor, Emergency Stop` directly — one bounce costs you a 180-second compressor lockout. Add a debounce/filter column to §11 for every discrete input and a filtering requirement for analog inputs.

**Power-up initialization.** Only `Restart Delay` and the RV request default are specified. Define the initial state of every state machine, initial timer values, and which values are non-volatile. Related: 8.3.6/8.3.7 say run hours and start counts "shall maintain a tally" with no storage, resolution, rollover, reset, or persistence-across-power-loss requirement.

**Configuration validation.** Define behavior for invalid combinations — `Heat Pump = AC` with `Dedicated Heat Source = None` (unit cannot heat at all), `Number of Circuits = 1` with a B-only lead-lag mode, dual-stage selections without the outputs to support them. Say whether the board rejects, clamps, or defaults.

**Lead/lag on failure.** `Circuit, Available` (8.2.8) is derived from Lockout only. A circuit in Emergency Stopping from a low-pressure trip is still "Available" and can be selected as lead, so the unit keeps picking the broken circuit. Fold circuit health into Available.

**Heat pump auxiliary heat.** There is no balance point, no compressor-plus-aux staging, no aux heat lockout, and no emergency heat mode. The O-terminal capacity table shows `-` for Mechanical Heat on a W call, which leaves "does the compressor keep running with aux heat?" unanswered.

**Blower anti-short-cycle on fan calls.** A 180-second blower ASC applies to a plain G/fan-only cycle, and because `Circuit, Lockout` includes `Blower, Lockout`, a momentary blower interruption also locks out mechanical cooling for 3 minutes. Consider scoping blower ASC to heat/cool cycles only, or a much shorter value.

**Electric heat post-purge.** Blower Lockin picks up Dedicated Heat Lockin (minimum runtime), but there is no requirement for the blower to run for a fixed period _after_ electric heat de-energizes to clear residual element heat. Add a post-purge delay.

**Economizer completeness.** `E_Alr` is a heading with no definition and no consuming logic, and §12 has no output terminals for forwarding Y to the economizer — yet 10.1.3 describes exactly that forwarding behavior.

**Diagram/table alignment.** The §2 diagram omits transitions the tables contain (Ready → Not Ready, Not Ready → Locked Out) and labels the Stopping → Locked Out edge "Not Active," which is where the semantic confusion in #2 and #3 comes from. A normal, uncommanded stop landing in "Locked Out" reads as a fault to anyone implementing or servicing this. Consider a distinct terminal state (Off/Idle) for clean stops and reserve Locked Out for actual lockout conditions.

If it's useful, I can take any one of these — the reversing valve model or the defrost sequence are the two I'd start with — and draft replacement requirement text in your existing table format.