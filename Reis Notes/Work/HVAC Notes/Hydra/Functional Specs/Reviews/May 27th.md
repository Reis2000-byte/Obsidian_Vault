### Critical Logic Errors

FIXED
**1. Crankcase Heater, Active (LCPCB-643) — Missing compressor condition.** The table has three conditions: Enable = Installed, Temp, Outdoor Air < Threshold, and Temp, Outdoor Air, Valid Range = VALID. The fundamental reason a crankcase heater exists — that the compressor is NOT running — is missing entirely. Without Compressor X, Active = FALSE as a condition, the heater would energize while the compressor is running, which defeats its purpose and could cause compressor damage. This needs to be added as a fourth AND condition.

FIXED
**2. Circuit, Operating Mode (LCPCB-85) — Defrost entry only covers Temperature method, not Switch method.** Priority 2 activation criteria only references "Outdoor Coil Frost Protection, Temperature is TRUE." Outdoor Coil Frost Protection, Switch (LCPCB-481) is entirely absent from this row. This means when Defrost, Selection, Method is set to Switch, the circuit will never enter Defrost mode — the Switch method defrost trigger has no path into the operating mode table. The activation criteria should read "Outdoor Coil Frost Protection, Temperature is TRUE OR Outdoor Coil Frost Protection, Switch is TRUE." The exit criteria has the same gap — it only covers temperature protection clearing, not the switch-based exit conditions.

FIXED
**3. Outdoor Fan, Active (LCPCB-129) — Circular and proving-time-free definition.** The body says "Outdoor Fan, Active shall be TRUE when the Outdoor Fan, State is Starting." This is circular — Active is defined by State, and State transitions from Starting to Running when Active is TRUE. The result is that Active becomes TRUE the instant State enters Starting, causing an immediate transition to Running with no proving delay. Every other actuator in the spec uses either a proving switch (Blower) or a proving time delay (Compressor, Reversing Valve) to confirm the actuator is physically operating before declaring Active. Outdoor Fan, Active needs the same — either a dedicated proving switch or a proving time delay similar to Compressor, Active (LCPCB-97).

---

### Significant Logical Issues

**4. Dedicated Heat, Emergency Stop (LCPCB-627) — References Blower, Lockout instead of Blower, Emergency Stop.** The body says "Blower, Lockout is ACTIVE." Blower, Lockout (LCPCB-72) now only contains Blower, Lockout, Anti Short Cycle. This means Dedicated Heat Emergency Stop fires every time the blower completes a run and its anti-short-cycle timer is active — a completely unintended behavior that would prevent dedicated heat from ever running within 3 minutes of a blower stop. The correct reference is Blower, Emergency Stop (LCPCB-66), which captures the actual safety conditions (smoke, fire) that should prevent dedicated heat from running.

**5. Circuit, Lockout (LCPCB-81) — Three issues in one requirement.** First, the body still includes "Blower, Lockout is TRUE" — per our discussion this was to be removed since Blower, Emergency Stop now flows to Unit, Operating Mode Lockout at the unit level. Second, "Circuit, Lockout, Defrost" appears in Referenced To but is completely absent from the body's bullet list — it is neither included nor excluded, which is ambiguous. Third, the body has Blower, Lockout listed but the note that was supposed to explain why it was retained (or removed) is also absent.

**6. Circuit, Active (LCPCB-84) — Referenced To points to wrong requirement.** The body says "any associated Compressor(s) State is Starting OR Running" — this checks the State enum (LCPCB-104). Referenced To links to Compressor, Active (LCPCB-97), which is the boolean. These are different requirements. Referenced To should point to Compressor, State (LCPCB-104). This has been flagged in every previous review.

**7. Circuit, Lockin (LCPCB-454) — Referenced To points to Compressor, Lockout instead of Compressor, Lockin.** The body says "Compressor, Lockin is TRUE" but Referenced To links to Compressor, Lockout (LCPCB-93). The correct link is Compressor, Lockin (LCPCB-91). This has been present in every version of the spec.

---

### Stale and Inconsistent References

FIXED
**8. Compressor, Ready (LCPCB-96) — Three stale Referenced To entries.** Referenced To includes Blower, Delay, Proving Time, Switch, Blower Proving, and Relay, Reversing Valve A — none of which appear anywhere in the body. The body only references Compressor, Delay, Staging Delay Time, Outdoor Coil Frost Protection Switch, Defrost Compressor Operation Delay, Compressor, Lockout, and Compressor, Active. The three stale entries need to be removed.

**9. Compressor, Lockout (LCPCB-93) — Compressor, Lockout, Defrost still in Referenced To.** The body correctly lists only Anti Short Cycle and Restart Delay, Selection, Mode, and the note correctly states Emergency Stop is excluded. But Referenced To still links to Compressor, Lockout, Defrost (LCPCB-620), which was supposed to be deprecated and absorbed into LCPCB-94. If it is still referenced here it has not actually been deprecated.

**10. Outdoor Fan, Output, Staged Control (LCPCB-576) — Stale Referenced To.** Referenced To still lists Compressor, Request (LCPCB-100), Dedicated Heat, Stage Request, and Dedicated Heat, State. None of these appear in the output table, which is now correctly driven by Outdoor Fan, Capacity Request, Lead Circuit, and Outdoor Fan, State. These three stale references need to be replaced with the actual inputs to the table. This has been flagged in multiple previous reviews.

**11. Outdoor Coil Frost Protection, Switch (LCPCB-481) — Three issues.** First, Referenced To shows "Unit, Operating Mode" but the body now correctly uses Circuit, Operating Mode — the Referenced To link needs to be updated to Circuit, Operating Mode. Second, Referenced By shows "Defrost, State" — there is no Defrost, State requirement in this spec. Defrost is a mode within Circuit, Operating Mode, not a separate state machine. This should reference Circuit, Operating Mode (LCPCB-85). Third, the FALSE conditions from our final rewrite are missing — the table only shows activation conditions with no explicit deactivation conditions, and the state retention clause during active defrost is absent.

**12. Dedicated Heat, Lockout (LCPCB-107) — Stale Referenced To.** Referenced To shows Dedicated Heat, Selection, Source and Blower, Lockout — neither appears in the body, which only lists Anti Short Cycle and Emergency Stop. These are stale references from earlier versions of this requirement.

**13. Dedicated Heat, Staged Output (LCPCB-108) — References non-existent requirement.** Referenced To shows "Dedicated Heat, Stage Request" but this requirement does not appear anywhere in the spec. The body uses "Dedicated Heat Capacity Request" as the input. The reference should be updated to Dedicated Heat, Capacity Request.

**14. Blower, Request (LCPCB-75) — Stale Referenced To.** Referenced To includes "Dedicated Heat, Request" and "Dedicated Heat, State" but neither appears in the body. The body only references Circuit, State and G Call.

---

### Incomplete Requirements

FIXED
**15. Blower, Output, Variable Voltage (LCPCB-603) — Still empty.** The body literally reads "NEED TO FILL OUT THIS SECTION STILL." This has been flagged in every single review across all versions. It is the only requirement in the entire document with no content whatsoever.

**16. Reversing Valve, Active (LCPCB-486) — Missing FALSE condition.** The body correctly describes the proving time behavior and uses the X designator. However there is no FALSE condition — consistent with every other Active requirement in the spec which includes an explicit FALSE statement when State is not Starting or Running.

---

### Configuration Issues

FIXED
**17. Compressor, Defrost, Run Time Maximum Value (LCPCB-505) — Step value of 30 minutes is very coarse.** The range is 30–90 minutes in 30-minute steps, giving only three possible values. For a light commercial RTU this seems inflexible — typical field conditions may require finer adjustment. This may be intentional based on product team decisions but is worth confirming.

FIXED
**18. Dedicated Heat, Delay, Minimum Runtime (LCPCB-175) — All values are dashes.** Default, Min, Step, and Max are all "-" with no units. This configuration parameter has no defined values. Since Dedicated Heat, Lockin, Minimum Runtime depends on this parameter, it effectively means the minimum runtime lockin is not implementable without a value. The product team needs to define this.