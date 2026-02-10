-  a technique to manage concurrent processes by using simple integer values

- Semaphore is a variable which is a non negative integer value that is shared between threads. They are used to solve the critical section problem and to achieve process synchronization in the multiprocessing environment

- apart from synchronization they are only accessed through two standard atomic operations: wait() and signal()

- Wait = P
- Signal = V
- The symbols are derived form their meaning in Dutch

Definition of wait():
```c
P(Semaphore S){
	while(S<=0)
	;//no operation but stuck in loop while true
	S--;
}
```
- wait function "tests" if there is another process using the critical section of memory. If there isn't then it will decrement the semaphore and enter

Definition of signal():
```C
V(Semaphore S)
{
	S++;
}
```
- this function gets called when the process using the critical section of memory is finished
- this "signals" other processes that it has finished

- All modifications to the integer value of the semaphore in the wait() and signal() operations must be executed indivisibly. This means that when one process is modifying the semaphore no other process can modify it at the same time.

**Types of Semaphores:**
1. Binary Semaphores:
	- The value of binary semaphore can only range between 0 and 1
	- Some systems consider these as mutex locks, as they are locks that provide mutual exclusion
2. Counting Semaphores:
	- Its value can range over an unrestricted domain
	- Used to control access to a resource that has multiple instances


- Reference to an atomic