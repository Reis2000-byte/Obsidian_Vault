- The seeds that are getting written for the code can be used on the actual hardware
- Each seed will have a software part and hardware part. There is a compilation part that will decide if it is software or hardware
- The seeds for firmware get executed in ceedling
- Goal is to have one seed executed in both ways BUT for now that has not been implemented
- Doing test on 7/7/2026 to verify that the tool works with the defrost functionality. The test is going to be for the new defrost PCB on the old Hydra hardware
- Monitors are created fro connecting to the dataabse and checking that the values are correct. Uses the protobuff database. If there is no database they will use the gpio
- Marco created the fuzzing logic. He uses the same seeds that we create with ceedling. From this there is a generated report. It can create a report of where vulnerabilities are
- Fuzzing will run for a number of cycles or number of iterations that can be set by the tester. Unsure how to determine the number of iterations that are needed.
- Fuzzing is fairly simple and a linux server. Marco has the image and we can configure the linux machine with the image and it takes around 4-5 hours to setup then we can also have one on site at DTTP.

Current Modules made for Hydra:
![[Pasted image 20260707143130.png]]

Test Framework:
- Fuzzing
- Software Ceedling Test Cases
- Hardware in the Loop testing

What Mexico Team Needs:
- Reis to give "OK" to go an proceed in creating the monitors
- PCB for testing

Implementation for Team:
- hardware test bench hardware
- PCBs to connect
- PC to connect

Example of hardware test: 
![[Pasted image 20260707134720.png]]
`InputSequence pev_operation_mode = {`
    `.test_name = "pev_operation_mode",`
    `.parent = &normal_operation,`
    `.io_sequence = { // TODO: the structure shall be placed in the LC defrost repository, adapt to the real names.`
        `#ifdef HVAC_OUTPUTS_ENABLE`
        `{`
            `.hw_inputs = {`
                `.pwr = false,`
            `},`
            `.runtime_seconds = 1,`
        `},`
        `#endif`
        `{`
            `#ifdef HVAC_OUTPUTS_ENABLE`
            `.hw_inputs = {`
                `.pwr = true,`
            `},`
            `#endif`
            `.thermostat = {`
                `.y1 = true,`
            `},`
            `.runtime_seconds = 179,`
            `#ifdef HVAC_OUTPUTS_ENABLE`
            `.outputs = {`
                `.cnt = false,`
                `.pev = true`
            `},`
            `#endif`
        `},`
        `{`
            `.thermostat = {`
                `.y1 = true,`
            `},`
            `.runtime_seconds = 1,`
            `#ifdef HVAC_OUTPUTS_ENABLE`
            `.outputs = {`
                `.cnt = false,`
                `.pev = false`
            `},`
            `#endif`
        `},`
        `{`
            `.thermostat = {`
                `.y1 = true,`
            `},`
            `.runtime_seconds = 1,`
            `#ifdef HVAC_OUTPUTS_ENABLE`
            `.outputs = {`
                `.cnt = true,`
                `.pev = false`
            `},`
            `#endif`
        `},`
        `{`
            `.thermostat = {`
                `.y1 = true,`
            `},`
            `.runtime_seconds = 179,`
            `#ifdef HVAC_OUTPUTS_ENABLE`
            `.outputs = {`
                `.cnt = true,`
                `.pev = false`
            `},`
            `#endif`
        `},`
        `{`
            `.thermostat = {`
                `.y1 = false,`
            `},`
            `.runtime_seconds = 1,`
            `#ifdef HVAC_OUTPUTS_ENABLE`
            `.outputs = {`
                `.cnt = false,`
                `.pev = true`
            `},`
            `#endif`
        `},`
        `#ifdef HVAC_OUTPUTS_ENABLE`
        `{`
            `.hw_inputs = {`
                `.pwr = false,`
            `},`
            `.runtime_seconds = 1,`
        `},`
        `#endif`
    `},`
`};`