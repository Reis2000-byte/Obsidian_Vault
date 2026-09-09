### 1. Smoke Detector and Fire Alarm Signal Polarity Contradiction FIXEDDDDD!!!!!

This is the most critical logical error in the spec and has persisted across all versions.

**Blower, Emergency Stop (LCPCB-66)** triggers when Switch, Smoke Detector is **OPENED** and Switch, Fire Alarm is **OPENED** — correctly treating these as normally-closed safety switches that open on fault.

**Unit, Operating Mode Lockout (LCPCB-595)** triggers when Switch, Smoke Detector is **CLOSED** and Switch, Fire Alarm is **CLOSED** — the exact opposite polarity.

Both requirements reference the same physical switches. They cannot both be correct. One of them will always be in the wrong state. A firmware engineer implementing both requirements as written would produce a system that either always has a unit lockout active, or never does. This needs to be resolved before any other work on these requirements.

---

### 2. Circuit, Lockout Includes Blower, Lockout — But Should It? FIXED!!!!!!

**Circuit, Lockout (LCPCB-81)** lists Blower, Lockout (LCPCB-72) as a contributing condition. Blower, Lockout is only TRUE when Blower, Emergency Stop is TRUE — which is only triggered by Smoke Detector or Fire Alarm opening.

The consequence is that a smoke or fire event locks out the circuit entirely, stopping the compressor. This may be intentional — in a fire alarm scenario you want everything to stop. But if so, the intent should be documented explicitly. The current spec is silent on why Blower, Lockout propagates to Circuit, Lockout, and a reader cannot tell if this is deliberate architecture or an accidental dependency. It also means Circuit, Emergency Stop and Circuit, Lockout both respond to the same smoke/fire event through different paths, which creates a redundant and potentially conflicting response.

---

### 3. Compressor, Lockout Includes Emergency Stop — Lockout and Emergency Stop Are Different Concepts FIXED!!!!

**Compressor, Lockout (LCPCB-93)** lists Compressor, Emergency Stop as one of its contributing conditions. But per LCPCB-571 (Mechanical Alignment Terminology), Lockout and Emergency Stop are distinct state machine concepts — Emergency Stop causes an immediate halt while the machine is running, while Lockout prevents it from starting. Including Emergency Stop inside Lockout collapses this distinction.

The more serious consequence is in the State table — Emergency Stop has its own priority row that transitions the compressor directly into Emergency Stopping. If Compressor, Emergency Stop also makes Compressor, Lockout TRUE simultaneously, the state machine now has two competing inputs firing at the same time. Which one wins depends on table row priority, but the intent is ambiguous. Emergency Stop should feed directly into the State table and not flow through Lockout.

---

### 4. Compressor, Lockout, Defrost Is in Compressor, Lockout — But We Agreed It Should Not Be FIXED!!!!

**Compressor, Lockout (LCPCB-93)** includes Compressor, Lockout, Defrost (LCPCB-620) as a contributing condition. As established in our earlier discussion, during defrost the compressor must keep running. If Compressor, Lockout, Defrost contributes to Compressor, Lockout, and Compressor, Lockout zeroes the Compressor, Capacity Request — then the defrost row in the Capacity Request table (forcing 100%) is fighting against a Lockout that is simultaneously trying to zero it. The resolution of this conflict depends entirely on table row priority, which is not explicitly stated. The agreed fix was to exclude Compressor, Lockout, Defrost from Compressor, Lockout so the defrost row in Capacity Request can work cleanly. This has not been implemented.

---

### 5. Compressor, Anti Short Cycle Lockout Does Not Activate on Emergency Stop  FIXED!!!!

**Compressor, Lockout, Anti Short Cycle (LCPCB-94)** activates when Compressor, Active turns FALSE AND Circuit, Operating Mode is NOT Defrost. This is correct for normal stop conditions but has a gap — if the compressor is stopped by an Emergency Stop, Active will also transition to FALSE, and the Anti Short Cycle timer will start. This may or may not be the intended behavior. On a normal stop the anti-short-cycle delay protects the compressor from rapid cycling. On an Emergency Stop caused by a safety switch, you may want the anti-short-cycle to be waived so the compressor can restart quickly once the safety condition clears — or you may want it to apply. The spec is currently silent on this distinction.

---

### 6. Dedicated Heat, State Emergency Stopping Exits to Not Ready FIXED!!!!!

**Dedicated Heat, State (LCPCB-115)** — Priority 2 exits Emergency Stopping to **Not Ready**. Every other subsystem exits Emergency Stopping to **Locked Out** — Blower, Circuit, Compressor, Outdoor Fan, and Reversing Valve all follow this pattern. This was flagged in previous reviews and remains unchanged with no explanatory note. If intentional, a note is required. If not intentional, it should exit to Locked Out.

---

### 7. Dedicated Heat, Active Has Two Separate Proving Time Behaviors With No Common Framework FIXED!!!!!

**Dedicated Heat, Active (LCPCB-112)** defines two different behaviors depending on Dedicated Heat Selection Source. For Electric heat it uses a 3-second relay-based proving logic. For Gas heat it uses Furnace PCB Feedback. These are reasonable hardware differences but the requirement is On Hold and the 3-second value is hardcoded in the body text rather than referencing a configuration parameter. Electric Heat, Delay, Proving Time (LCPCB-628) exists as a configuration parameter — it should be referenced here instead of the hardcoded value.

---

### 8. Outdoor Coil Frost Protection, Temperature Exit Condition Has a Logic Error FIXED!!!

**LCPCB-224** exit condition reads: the protection shall be FALSE when Temp, Outdoor Coil is INVALID OR > 33°F, AND either of the below conditions is VALID — then lists two conditions that read as "compressors have NOT run for 120 minutes" and "compressors have run in Cooling mode for 2 minutes."

The second condition says "All associated compressors have **NOT** run for 120 Minutes since this was triggered" — the word "NOT" here appears to be an error. The intent is almost certainly that the timer must elapse — meaning they HAVE run (or been off) for 120 minutes — not that they have NOT run. As written the condition is TRUE immediately at trigger time when no compressors have run yet, which would allow the protection to clear instantly.

---

### 9. Outdoor Coil Frost Protection, Switch Timer Reference Points to Itself FIXED!!!!!

**LCPCB-481** (Outdoor Coil Frost Protection, Switch) deactivates when Defrost, Mode Timer is equal to **Defrost, Mode Timer Maximum Value**, and it references **LCPCB-481** as the source of that Maximum Value. LCPCB-481 is the same requirement — it is referencing itself. The Maximum Value should be a separate configuration parameter, not the protection requirement itself.

---

### 10. Defrost, Mode Timer Reset Condition References Circuit Operating Mode = Cooling Only FIXED!!!!!

**LCPCB-616** (Defrost, Mode Timer, Reset Conditions) resets the timer when Circuit Operation Mode equals Cooling. This means if the circuit transitions from Defrost to Heating directly — without going through Cooling — the timer does not reset. On a heat pump that exits defrost and immediately resumes heating this is a meaningful gap. The reset condition should likely include any transition out of Defrost, not only a transition to Cooling specifically.

---

### 11. Manual Lead Selection Is Deprecated But Still Listed in Lead Circuit FIXED!!!!

**Lead Circuit (LCPCB-88)** still lists Manual Lead Selection (LCPCB-464) as one of four determination options. LCPCB-464 is marked Deprecated. This creates a contradiction — the parent requirement references a child that no longer exists. Either LCPCB-464 should be removed from the LCPCB-88 bullet list, or it should be un-deprecated. This has been present since the first review.

---

### 12. Blower, Output, Variable Voltage (LCPCB-603) Is Still Empty FIXED!!!!!

No description, no table, no content of any kind. This is the only requirement across the entire spec that remains completely unwritten after multiple revision cycles. It needs to be authored before the blower control section can be considered complete.

---

### 13. Section 2.4.7 Is Still Named TBD FIXED!!!!

The folder containing A2L Mitigation Feedback, Alarm Economizer, BMS Return, Phase Monitor Feedback, R Monitor, and TSTAT FLT is still named TBD. All six items inside it are On Hold with no descriptions. This entire section needs either a proper name and content, or a decision to defer it formally with a note explaining why.

---

### 14. Crankcase Heater, Active Table Did Not Export FIXED!!!!!

**LCPCB-643** (Crankcase Heater, Active) has the body text and the note about X designators but the actual condition table is absent from the export. The table we developed together — with Crankcase Heater X Enable, Compressor X Active, Temp Outdoor Air VALID, and Temp Outdoor Air < Threshold — is not present. This needs to be verified in Jira and re-exported.

---

### 15. Reversing Valve, Active Still Only References Relay A  FIXED!!!!!!!

**LCPCB-486** body references only Relay, Reversing Valve A (LCPCB-316). Relay B (LCPCB-605) is absent despite the Output requirement having separate tables for both A and B. This has been flagged in every previous review without resolution.