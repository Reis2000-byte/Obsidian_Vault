## Ideal Project Flow
- Projects normally always will start off with a schedule
- Large objective of making a schedule is figuring out which of the processes can be done in parallel
- Challenge can be that embedded software is at the end of the schedule so a lot of work will have to be with prototypes or proof of concept boards
## Hardware Design
- As hardware team works on schematics the software team can work with a development board of the same chip
- Look to generate an I/O map that describes the connection made at every pin
- Ask electrical engineer which circuits are the riskiest for the system and get to being able to test those as soon as possible
## Board Bring-Up
- Finding bugs early in the design can be considered a gift
- Make tools so that other people can test this without you being present
- Take time for the hardware verification code because there is a good chance that it will stay around for future releases
- Knowing what tests need to be written relies on the knowledge of the processor and the peripherals
