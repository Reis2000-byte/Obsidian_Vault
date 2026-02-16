# ✅ Day 1 – Embedded Fundamentals + MCU Peripherals

### 🎯 Goal:

Be sharp on low-level C, real-time thinking, and hardware interaction.

---

## 1️⃣ C Programming Review (Core Focus)

### Topics

- `volatile` and why it matters (ISR + registers)
    Volatile matters because this means that the register or variable can be changed outside of the program's knowledge. If volatile isn't give then the compiler may optimize the variable. 
    Variables that should be volatile:
	- They are modified in ISR
	- They are hardware registers
	- They can change outside normal control flow
    Example of **ISR**:
		Regular Code:
    ```
    int flag = 0;

	void ISR(void)
	{
	    flag = 1;
	}
	
	int main(void)
	{
	    while (flag == 0)
	    {
	        // wait
	    }
	}
	```
	Might optimize to without the volatile:
	```
		if (flag == 0)
		{
		    while (1) { }
		}
	```
- Bit manipulation
    - Write code to set bits 3 and 7.
	```
	x |= (1U << 3);
	x |= (1U << 7);
	```
	- Extract bits [10:8].
    ```
    uint32_t extracted = (value >> 8) & 0x7; //could use 0b111 with some compilers
    //other option
    uint32_t extracted = (value & (0x7 << 8)) >> 8;
    ```
	- Clear lowest set bit.
    ```
    x &= (x-1);
    ```
	- Toggle all even bits.
    ```
    x ^= 0x55555555U;
    ```
	- Pack two 4-bit values into one byte.
- Memory layout (stack vs heap vs static)
    +----------------------+
	|   Text (.text)       |  ← Code (Flash)
	+----------------------+
	|   Read-only (.rodata)|  ← const data
	+----------------------+
	|   Initialized (.data)|  ← global/static initialized
	+----------------------+
	|   Zeroed (.bss)      |  ← global/static uninitialized
	+----------------------+
	|        Heap          |  ← malloc/new (grows up)
	|          ↑           |
	|          |           |
	|          ↓           |
	|        Stack         |  ← function calls (grows down)
	+----------------------+
**Static** variables live in .data or .bss. These variables limit variables to only be used in the scope of the file or function that they live in. Helps with naming.
**Const** variables live in .rodata or flash. These variables cannot be changed after initialization. Helps by putting into flash memory for speed as well as safety critical stuff.

|                       | Heap         | Stack          | Static (global variables and static) |
| --------------------- | ------------ | -------------- | ------------------------------------ |
| Allocation            | Manual       | Automatic      | At compile time                      |
| Lifetime              | until free() | function scope | Entire program                       |
| Speed                 | slow         | fastest        | fast                                 |
| Fragmentation         | yes          | no             | none                                 |
| Deterministic         | no           | yes            | yes                                  |
| Shared across threads | yes          | no             | no                                   |
| Location              |              |                | RAM                                  |

- Struct padding
    
- Pointers and const correctness
    
- Function pointers
    
- Circular buffers
    

### Example Problems

**Problem 1 – Register Bit Manipulation**

`#define GPIO_OUT_REG (*(volatile uint32_t*)0x40020014)  void set_pin(int pin) {     // Set pin HIGH }`

Implement:

- Set pin
```
#define GPIO_OUT_REG (*(volatile uint32_t*)0x40020014)  

void set_pin(int pin)
{
	GPIO_OUT_REG |= (1U <<< pin);
} 
```
- Clear pin
```
#define GPIO_OUT_REG (*(volatile uint32_t*)0x40020014)  

void clear_pin(int pin)
{
	GPIO_OUT_REG &= !(1U <<< pin);
} 
```
- Toggle pin
```
#define GPIO_OUT_REG (*(volatile uint32_t*)0x40020014)  

void toggle_pin(int pin)
{
	GPIO_OUT_REG ^= (1U <<< pin);
} 
```

Be ready to explain:

- Why `volatile`?
    Tells the compiler that the memory can change outside of the programs control.
- What happens without it?
    Can optimize sections of code which can cause miss timings and edge triggers

---

**Problem 2 – Circular Buffer (Common in CAN/LIN drivers)**

Implement a fixed-size circular buffer:

- `push()`
    
- `pop()`
    
- Detect full vs empty
    

Explain:

- How to make it ISR-safe?
    
- Lock-free vs critical section?
    

---

## 2️⃣ Interrupts + Concurrency

Understand:

- ISR execution constraints
    
- Avoiding long blocking calls in ISRs
    
- Race conditions
    
- Atomic access
    

**Example Question:**  
A button interrupt increments a counter. Main loop reads it.  
What can go wrong? How do you fix it?

---

## 3️⃣ Automotive Communication Basics

Review:

- CAN basics (ID, arbitration, 11-bit vs 29-bit)
    
- LIN basics (single master architecture)
    
- Message filtering
    
- Error frames
    

Be able to answer:

- Why CAN is deterministic?
    
- How would you debug a noisy bus?
    

---

# ✅ Day 2 – State Machines + Actuator Control + Controls Coding

### 🎯 Goal:

Show you can design logic for body features (windows, doors, mirrors).

---

## 1️⃣ Finite State Machines (Very Important)

Body controls = lots of state machines.

### Example: Power Window Controller

States:

- IDLE
    
- MOVING_UP
    
- MOVING_DOWN
    
- OBSTRUCTION_DETECTED
    
- CALIBRATION
    

Write a state machine using:

- Enum
    
- Switch case
    
- Event driven design
    

Be ready to discuss:

- Edge cases
    
- What happens if button pressed during movement?
    
- How do you debounce input?
    

---

## 2️⃣ Bang-Bang Control (On/Off Control)

Used in:

- Seat heaters
    
- Defoggers
    
- Simple thermal control
    

### Example: Temperature Controller

Requirement:  
Maintain 40°C ± 2°C using a heater.

Implement:

`if (temp < 38) heater_on(); else if (temp > 42) heater_off();`

Be ready to discuss:

- Oscillation problem
    
- Adding hysteresis
    
- Why it works in slow systems
    

---

## 3️⃣ PID Control (Very Likely Topic)

Used in:

- Window motor position
    
- Mirror motor
    
- Seat motor positioning
    
- HVAC blend door
    

### Basic PID Formula

u(t)=Kpe(t)+Ki∫e(t)dt+Kdde(t)dtu(t) = Kp e(t) + Ki \int e(t) dt + Kd \frac{de(t)}{dt}u(t)=Kpe(t)+Ki∫e(t)dt+Kddtde(t)​

### Coding Exercise

Implement discrete PID:

`typedef struct {     float kp, ki, kd;     float integral;     float prev_error; } PID;  float pid_update(PID* pid, float setpoint, float measurement, float dt);`

Be ready to discuss:

- Integral windup
    
- Derivative noise amplification
    
- How to clamp output
    
- What happens if dt changes
    

---

## 4️⃣ Motor Control Conceptual Questions

- H-bridge control basics
    
- PWM frequency selection
    
- Current limiting
    
- Stall detection
    

**Example Question:**  
How would you detect if a window motor is blocked?

Answer:

- Current spike detection
    
- Position not changing
    
- Timeout threshold
    

---

# ✅ Day 3 – Debugging, System Design, Automotive Thinking

### 🎯 Goal:

Think like a production embedded engineer.

---

## 1️⃣ Debugging Scenario Questions

Example:

> A door lock actuator works 90% of the time but randomly fails.

How do you debug?

Good answer structure:

1. Reproduce
    
2. Logging
    
3. Scope CAN messages
    
4. Check supply voltage
    
5. Check temperature
    
6. Firmware logging
    
7. Fault counters
    

They care about your **process**.

---

## 2️⃣ Design Question

Example:

> Design a seat heater controller module.

Include:

- Temperature sensor (ADC)
    
- MOSFET control
    
- Overtemp protection
    
- Fault reporting via CAN
    
- Watchdog
    
- Safe startup state
    

Mention:

- Failsafe state
    
- Redundancy if safety-related
    
- Self-test on boot
    

---

## 3️⃣ Real-Time Concepts

Review:

- Scheduling
    
- Deterministic timing
    
- Jitter
    
- Task priorities
    
- Watchdog timers
    

Example:  
Why avoid dynamic memory in automotive systems?

---

## 4️⃣ Whiteboard Coding Speed Drills

Practice writing clean code without compiler:

- Ring buffer
    
- Debounce function
    
- Moving average filter
    
- Low-pass filter:
    

`y = y_prev + alpha * (x - y_prev);`

---

# 🔥 Bonus: What Tesla Specifically Looks For

From patterns in interviews:

- Clear, structured thinking
    
- Strong ownership mindset
    
- Deep understanding of low-level details
    
- Ability to reason about hardware/software interaction
    
- Practical debugging mindset
    
- Simplicity in design
    

Be ready for:

- “What happens at power-on?”
    
- “How would this fail in the field?”
    
- “What’s the worst-case scenario?”
    

---

# ⚡ High-Impact Practice Problems

1. Write a debouncer for a noisy switch.
    
2. Implement a soft-start motor ramp using PWM.
    
3. Add anti-windup to a PID.
    
4. Design CAN message packing with bitfields.
    
5. Detect overcurrent in software.
    

---

# 🎯 If You Want to Go One Level Deeper

Since you're embedded-focused:

- Review fixed-point math vs floating point
    
- Know how to implement PID without floats
    
- Understand memory-mapped registers
    
- Practice explaining race conditions clearly