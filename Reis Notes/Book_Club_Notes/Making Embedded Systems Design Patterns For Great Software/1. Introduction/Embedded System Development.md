## Debugging
- Unlike other software where you can run and debug on the same computer debugging isn't the same for embedded systems
- Embedded Systems require cross debuggers
- Interfaces are created so that they allow use to eavesdrop on the processor as it works. JTAG is the most common.
![[Pasted image 20260805110451.png|425]]
- Processors have to offer debugging because they can add cost and resources. Examples:
	- Breakpoints can be as simple as a assembly command to stop OR if the program is sent to flash memory it has to use an internal register (hardware breakpoint)
- Debugging device types:
	- emulator
		- can be processor specific
	- in-circuit emulator (ICE)
	- JTAG adaptor
- Can use printf instead of debugging tools to offer universal serial debugging but this can lead to unfound errors due to timings
## More Challenges
- Resources considered for embedded systems:
	- Memory (RAM)
	- Code Space (ROM)
	- Processor cycles or speed
	- Battery Life (or power savings)
	- Processor Peripherals
- Balancing the above challenges is one of the main challenges with embedded design
- Another main challenge is the need to know the hardware design because of the uncertainty if the error is in the software or the hardware
- Another challenge is creating reasonably cost system with support for manufacturability
- Another challenge is the environment which the product will be used. Allows for millions of avenues for potential bugs
- Change is probably the biggest challenge because it almost never ends
- We need to be able to have embedded designs that can be flexible to the system that they are in. CRYSTAL BALL BABY

## Principles to Confront those Challenges
- Need to make designs that can meet product goals, meet design constraints, and other challenges outside of the design
- Software principles to assist us:
	- Modularity
		- Separate the functionality into subsystems and hide the data each subsystem uses
	- Encapsulation
		- We can create interfaces between the subsystems so they don't know much about each other
- By creating subsystems (objects) we can change one area of software without the worry that it will affect another area of the code
- Good rule of thumb for where to draw the line for subsystems is which parts can change independently
- Documenting the code with comments is a good way to reduce bugs
- Look to document what the code does not how it does it. Not line by line and more summaries for functions
- Before optimization look to get the solution working and functional then go back to optimize

## Interview Question
![[Pasted image 20260805113057.png]]
![[Pasted image 20260805113116.png]]

## Prototypes and Maker Boards

- Just because you have the system working on a Pi or Arduino does not mean that it will work with the product
- Using custom boards push you out of simplified dev environments
- Adding external programmer/debugger gives us debugging beyond `printf()`