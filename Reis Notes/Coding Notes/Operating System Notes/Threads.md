- a basic unit of CPU utilization

- What a thread contains:
	- thread ID
	- a program counter
	- register set
	- stack
- A traditional/heavyweight process has a single thread of control
- If a process has multiple threads of control, it can perform more than one task at a time.

- Benefits:
	- Responsiveness: Threads allow a program to continue running even if part of it is blocked or is performing a lengthy operation
	- Resource Sharing: Allows several activities to use the same address space. They will be able to share code, files, and data.
	- Economy: Allocating memory and resources for process creation is costly. Due to threads sharing resources of the process which they belong, it is more economical to create and context switch threads
	- Ability to use multiprocessor architectures: Allows threads to run in parallel on different processors. Multithreading on a multi-CPU machine increases concurrency.

Process vs. Threads:
- A single process can have multiple threads
- A single process can also be single threaded
- If one thread in a process fails then the entire process fails
- A program is made up of a sequence of processes
- A process is essentially made up by a single or multiple threads