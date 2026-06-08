### State-machine logic

FIXED
**1. The "Locked Out (hold)" condition appears inverted in every state table.** In all six state machines (Blower 1.1.4, Circuit 1.2.7, Compressor 1.3.8, Dedicated Heat 1.4.5, Outdoor Fan 1.5.6, Reversing Valve 1.6.5), Priority 3 reads: _Current = Locked Out, Condition = "Emergency Stop is FALSE" → Locked Out (hold)_. That's backwards. With E-Stop **cleared (FALSE)**, this higher-priority row fires and traps you in Locked Out forever, so the lower-priority release row ("Locked Out, Lockout FALSE → Not Ready") can never be reached. Conversely, when E-Stop is **TRUE**, Priority 3 doesn't fire and you can fall through to the release row. The intent is clearly "hold in Locked Out _while_ E-Stop is TRUE," so the condition should be E-Stop **is TRUE**. As written, the device can't recover after any latched lockout.

FIXED
**2. Outdoor Fan readiness is gated on a transient compressor state (1.5.1).** _Outdoor Fan, Ready = TRUE when any associated Compressor State is Ready._ "Ready" is a momentary pre-start state — once the compressor moves Ready → Starting → Running, its state is no longer "Ready," so Outdoor Fan Ready drops to FALSE while the compressor is actually running. The fan only stays up if it happens to catch the one-scan "Ready" window before the compressor starts, which is a race condition. Compare this to _Compressor, Ready_ (1.3.1), which correctly requires _Blower State = Running_ (a stable state). Outdoor Fan Ready should key off the compressor being Starting/Running/Active, not "Ready."

FIXED
**3. "Locked Out (hold)" is used as a New State but is never defined** as one of the actual states (Not Ready, Ready, Starting, Running, Stopping, Emergency Stopping, Locked Out). Minor, but it should be specified as "remain in Locked Out."

### Interlocks and sequencing.

NO INPUT
**4. No condenser-fan interlock on the compressor.** _Outdoor Fan, Lockout shall always be FALSE_ (1.5.7.1), and Compressor Ready (1.3.1) only requires the **blower** to be Running — it never requires the outdoor fan to prove. So a compressor can start and run with a failed/unproven condenser fan. Combined with #2, there is effectively no fan-fail → compressor-lockout path. The only backstop is the high-pressure switch (Circuit Emergency Stop), which is a nuisance/last-resort protection, not a proper interlock. For an RTU this is a real head-pressure exposure.

**5. Blower Request omits dedicated heat → electric heat can deadlock (1.1.2).** Blower Request is TRUE only when a Circuit is Starting/Running or G is active. But _Dedicated Heat, Ready_ (1.4.1) requires _Blower State = Running_. On a W-only electric-heat call (no Y, and many thermostats don't energize G on a heat call), the blower never gets a request, so dedicated heat can never reach Ready, so it never runs — and the blower never starts because nothing requests it. The "Referenced To" list for Blower Request even includes Dedicated Heat Request/State, so the condition was intended but is missing from the prose. (Gas may survive because the furnace board runs its own blower, but electric is broken.)

**6. Loss of airflow proving during Run is not handled.** The Blower State table has no transition out of _Running_ for _Blower, Active = FALSE_ (only Emergency Stop, or NO Request/Lockout). If the proving switch opens mid-cycle (belt break, blocked filter), the blower stays in Running with its relay energized and nothing trips. This matters most with electric heat running — your control logic won't react to airflow loss; you'd be relying entirely on a hardware limit. I'd add a "Running, Blower Active FALSE → fault/lockout" path.

**7. Staging delay is ineffective on simultaneous cold start (1.3.1).** The staging delay only applies "since _another_ compressor on a different circuit transitioned to Active and is still TRUE." If two compressors come up to demand at the same time and neither is Active yet, neither sees an "other active compressor," so both start together — defeating the inrush/electrical-staging purpose exactly when you most want it (first-on after a call or power restore).

### Reversing valve / defrost

FIXED
**8. The pressure-equalization lockout is self-defeating (1.6.6.4 + 1.6.3).** The lockout is meant to prevent the valve from switching for ~30 s after Circuit Active goes FALSE so pressures can equalize. But when that lockout is ACTIVE, the RV state goes to Locked Out, and the Output table (1.6.3) energizes the relay **only** in Starting/Running. So entering the lockout immediately forces the relay INACTIVE — i.e., the valve switches position right when you intended to prevent it. The lockout and the output table contradict each other; you need the output to _hold_ its prior commanded state during the equalization window, not drop out.

FIXED
**9. Defrost can't sustain as written (2.4.1 / 1.2.5 / 2.4.5).** Defrost initiation (Frost Protection Switch, 2.4.1) requires _Defrost Compressor Run Timer = Max (30 min)_. But the Circuit Operating Mode table (1.2.5) only enters/exits Defrost on _Outdoor Coil Frost Protection, **Temperature**_, and the run-timer **resets to 0 on the Heating→Defrost transition** (2.4.5). So the moment you enter Defrost, the condition that triggered it (run timer = max) is cleared, the demand signal goes FALSE, and the mode table exits Defrost. There's also no path that uses the _Defrost Mode Timer_ (12-min max) as the termination — that timer is defined but never wired into the mode transition. Net: no stable defrost duration and no time/temperature termination. (I'm assuming the "Switch" and "Temperature" frost signals are the same underlying flag; if they're genuinely separate, then the Switch-based method also never reaches the mode table at all, which is its own gap.)

### Capacity / threshold configuration

**10. Variable-speed compressor is declared but unsupported.** _Compressor, Selection, Type_ allows "Variable Speed" (3.3.5, "should always start first"), but the Capacity Request table (1.3.2) only has Single Stage and Dual Stage rows, the Output is purely relay-staged (1.3.5, Comp A1/B1), and there's no modulating/0-10V or speed output in Section 5. So a variable-speed selection has undefined capacity and no actuator. Either remove the option or add the modulation path and a "start first" rule in the staging logic.

**11. Non-monotonic / duplicate stage thresholds.**

- Blower thresholds: Stage 1 = 0, Stage 2 = 5, **Stage 3 = 4** (3.2.4–3.2.6). Stage 3 < Stage 2 is non-monotonic.
- Dedicated Heat thresholds: Stage 1 = 0, **Stage 2 = 4, Stage 3 = 4** (3.4.1–3.4.3). Identical, so stage 2 and 3 can't be distinguished.

Compressor thresholds (0/4/10) are fine; the other two need ascending, distinct defaults.

**12. A1/B1 relay table conflates two different machines (1.3.5).** The Output Staged Control table uses _Lead Circuit A/B_ to pick between Relay A1 / Relay B1, but Section 5 describes A1/B1 as separate compressor outputs (Comp A1, Comp B1). For two single-stage compressors (one per circuit), each relay should follow its own circuit's request, not the Lead Circuit. For a dual-stage single compressor, A1/B1 = stages — in which case the 50%/Lead-B row energizing **B1 alone (high stage without low stage)** is invalid for most two-stage compressors. The table needs to commit to one topology, and stage sequencing (low before high) needs enforcement. The same conflation exists for the Outdoor Fan output (1.5.5): single-stage treats A/B as per-circuit fans, dual-stage treats them as two speeds tied to Lead Circuit.

### Safety / protection gaps

**13. Low-pressure suction switch drives an immediate Emergency Stop with no startup or defrost bypass.** Suction pressure normally dips on every compressor start and during defrost/low-ambient operation. Tying the LP switch directly to a Priority-1 Emergency Stop (via Circuit/Compressor E-Stop) with no ignore-on-start timer or debounce will cause nuisance trips. LP is usually a soft lockout with a short bypass window and a retry/lockout count, not an instantaneous hard trip.

**14. No latching safety lockout / manual-reset or trip counter.** HP, LP, phase, and similar safety trips flow through Emergency Stop and then are released after only the anti-short-cycle time (1.3.9.4 note). There's no "lock out after N trips, require manual reset" behavior. Repeated HP/LP faults will just cycle indefinitely.

**15. A2L leak alarm isn't wired to any mitigation/shutdown (2.1).** _A2L Leak, Alarm_ only sets a flag from the mitigation feedback; nothing in the actuator logic responds (compressor shutoff, forced blower/dispersion, etc.). Given the unit explicitly uses A2L refrigerant, the leak condition should command a defined safety response in this logic, not just raise an alarm. The note "blower motor might be driven by the A2L microcontroller" suggests mitigation lives elsewhere, but that hand-off should be specified.

**16. Crankcase heater fails to the unsafe side (2.5.1).** Crankcase Heater Active requires _Temp, Outdoor Air, Valid Range = VALID_. If the outdoor sensor faults on a cold day, the heater turns off, leaving the compressor unprotected against refrigerant migration/slugging. Crankcase heat usually fails ON (or to a safe default) on sensor loss. Note this is the opposite fail-safe philosophy from most of the rest of the spec.

### Two-circuit modeling and miscellaneous

**17. Safety switches aren't instanced per circuit.** Reversing Valve and Crankcase Heater carry an "X = A or B" designator, but Switch High Pressure, Switch Low Pressure, and Switch Defrost are single signals (4.14, 2.4). A genuine two-circuit unit needs HP/LP/defrost per circuit; otherwise one circuit's fault trips both, and per-circuit defrost (which Circuit Operating Mode already models independently) can't actually be driven.

**18. Gas dedicated heat has no output or staging.** Dedicated Heat allows Source = Gas (3.4.4) and Active uses Furnace PCB feedback (1.4.3), but the Staged Output table (1.4.4) and Section 5 only define Electric Heat A/B relays — there's no gas valve/heat-call output. Gas staging is undefined.

**19. Emergency Stop doesn't zero Capacity Request.** Because E-Stop is intentionally excluded from Compressor Lockout (1.3.9.3 note), the Capacity Request table (1.3.2) still computes a nonzero value during an E-Stop (it only zeros on _Lockout_). The state machine keeps the relay off, but any downstream consumer that reads capacity-request rather than state could behave inconsistently. Worth deciding whether E-Stop should also force capacity to 0.

**20. Lead/Lag config not validated against circuit count (3.1.1 vs 1.2.6.1).** "Circuit B Lead" / "Circuit B Only" are selectable even when Number of Circuits = 1, where only Circuit A exists. Needs a config-validity rule.

If it's useful, the two I'd treat as blocking before anything else are #1 (the inverted lockout-hold makes recovery impossible across the whole system) and #2/#4 (no real condenser-fan interlock). Want me to draft corrected wording for the state-table Priority-3 row and the Outdoor Fan Ready/interlock conditions?