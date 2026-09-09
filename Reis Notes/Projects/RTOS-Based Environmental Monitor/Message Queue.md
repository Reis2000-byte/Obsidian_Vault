### What It Is

A message queue is a RTOS primitive that acts as a thread-safe buffer between two tasks. Think of it as a pipe with a fixed capacity — one end accepts items pushed in, the other end delivers items pulled out, and the operating system guarantees that the handoff is safe even when both tasks are running concurrently on the same CPU.

It is fundamentally different from a shared global variable. A global variable has no concept of ownership — both tasks can read and write it at the same moment, which on a preemptive scheduler leads to torn reads, stale data, and race conditions that are nearly impossible to reproduce reliably. A message queue solves all of that by design.

---

### What It Actually Does

When a task puts a message into the queue, the RTOS makes a **deep copy** of the data and stores it in a private internal buffer that belongs to the queue itself. The sending task then owns nothing — it can modify or destroy its local copy immediately without affecting the message in transit. When the receiving task pulls the message out, it gets its own fresh copy of that data on its own stack.

This means the data passes through three distinct owned spaces:

```
Sender's stack   →   Queue's internal buffer   →   Receiver's stack
(sender owns)         (RTOS owns)                   (receiver owns)
```

No task ever touches memory that another task currently holds. The queue is the middleman, and the RTOS controls access to it entirely.

---

### Why It Matters in This Project

This project has a producer-consumer relationship at its core. One task exists solely to gather sensor readings on a fixed schedule. Another task exists solely to report those readings over UART. These two concerns are deliberately kept separate — one handles timing and data acquisition, the other handles formatting and communication. Mixing them into a single task would mean the timing of sensor reads is affected by however long UART transmission takes, and vice versa.

The queue is what makes that separation possible. The sensor task does its job and drops the result into the queue, then immediately goes back to sleep until its next period. It does not wait, it does not care whether the UART task has processed the last reading yet, and it does not know anything about UART at all. The UART task wakes up only when something is available to send. It does not poll, it does not spin, and it does not need to know anything about sensors or timing.

Without the queue, coordinating these two tasks would require either a shared global variable with a mutex protecting it, or collapsing both responsibilities into one task and giving up the clean separation of concerns entirely.

---

### The Blocking Behavior

The queue's most important property in a real-time system is how it handles the two edge cases: a full queue and an empty queue.

**When the queue is empty** and the receiving task tries to pull from it, the task does not busy-wait. The RTOS immediately suspends it and marks it as blocked, returning the CPU to the scheduler. No cycles are wasted. The task stays suspended until another task puts something in the queue, at which point the RTOS wakes the receiver automatically.

**When the queue is full** and the sending task tries to push into it, the sender can either be told immediately that the push failed, or it can block and wait for space to open up. In this project the sensor task does a non-blocking put — if the queue were somehow full, the reading is discarded rather than holding up the sensor task's timing.

This asymmetry is intentional. The sensor task has strict timing requirements and should never be delayed by the state of the UART. The UART task, by contrast, can afford to wait indefinitely because its only job is to process whatever comes in.

---

### What It Protects Against

**Race conditions.** Without synchronization, if the sensor task wrote three floats to a global struct while the UART task was halfway through reading them, the UART task would see two values from the new reading and one from the old one — a corrupted, meaningless mix. The queue prevents this by transferring data atomically from the sender's perspective.

**Priority inversion risk.** If both tasks shared a mutex-protected variable, the lower-priority UART task holding the mutex could block the higher-priority sensor task from running. With a queue, the sensor task never waits for the UART task — it either puts the message in and moves on, or drops it.

**Tight coupling.** If the sensor task directly called UART transmit functions, the two tasks would be tightly entangled. A change to how UART works would require changes to the sensor task. The queue acts as a clean interface — either side can change its implementation without the other knowing.

---

### The Capacity Buffer

The queue in this project holds up to ten messages. This provides a small cushion. If the UART task is temporarily slow — say it is blocked waiting for a previous transmission to complete — the sensor task can keep producing readings and they will accumulate in the queue rather than being dropped immediately. Once the UART task catches up, it drains the backlog. The capacity of ten means roughly ten seconds of readings can be buffered before data loss occurs under a one-second sensor period. This kind of temporal decoupling is one of the core benefits of a queue over direct task-to-task signaling.