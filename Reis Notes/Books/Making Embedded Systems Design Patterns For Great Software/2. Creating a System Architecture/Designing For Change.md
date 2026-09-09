- Boxes in the diagrams should represent the Platonic ideals of each thing instead of a specific implementation
## Encapsulate Modules
- Modules that don't depend on the content or behavior of other modules
- Tips to look for in different diagrams:
	- Organigram:
		- Looks for objects that are only used by one other object
		- If they are unlikely to change then combine them
	- Layering:
		- Look for collection of object that are all used together
		- Can these be grouped together in a higher level interface to manage the objects?
		- This is kind of like creating a hardware abstraction layer
	- Which modules have a lot of interdependencies and which ones can be broken apart or simplified
## Delegation of Tasks
- These diagrams help break up the work and allow for work to be split amongst people
- Instead of delegating fun and boring tasks to others give someone a box to work on to help ensure quality throughout the code
- Think about if one of the imaginary minions write bad code. How can we limit and protect the amount of data that goes back and forth between the blocks
- The minimization of complexity between boxes will make the project go more smoothly
- If we can have our minions boxed into a section of code, with well understood interfaces, the easier testing and developing will be
## Driver Interface: Open, Close, Read, Write, IOCTL
- This section thinks about going from the bottom up instead of the top down
- Designs:
	- Top-down design:
		- when we think about what you want then dig into what you need
	- Bottom-up design:
		- when we consider what you have and build what you can out of that
- Many drivers in embedded are based on the POSIX API used to call devices in Unix systems because it works and why break the wheel?
- The interface to Unix drivers:
	- open:
		- Opens the driver for use. Similar to init
	- close:
		- Cleans up the driver after being used
	- read:
		- Reads data from the device
	- write:
		- Sends data to the device
	- ioctl:
		- Stands for Input/output control and handles the features not covered by the other parts of the interface
		- Its use is somewhat discouraged by kernel programmers due to its lack of structure
- In Unix a driver is part of the kernel
- Example of modeling a driver like a Unix Driver:
	-  `spi.open()`
	- `spi_open()`
	- `SpiOpen(WITH_LOCK)`
	- `spi.ioctl_changeFrequency(THIRTY_MHz)`
	- `SpiIoctl(kChangeFrequency, THIRTY_MHz)`
- Recommended to use the Unix as a starting point but if it doesn't make sense then don't try to force it

## Adapter Pattern
- Converting the interface of an object into one that is easier for a client to use (wrapper)
	![[Pasted image 20260820151453.png]]
	- Note that drivers are stackable
	- Wrappers can add memory and delays to code. If we are resource constrained then they are sometimes a good idea to use
	- If we stay consistent with the interfacing then the higher level code is resistant to change
	 ![[Pasted image 20260820151729.png]]