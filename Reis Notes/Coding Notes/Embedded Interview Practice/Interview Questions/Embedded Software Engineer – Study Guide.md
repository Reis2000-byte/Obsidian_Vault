
**Format:** 2–3 hours/day prep, divided into sections.

---

## **1️⃣ Core C Fundamentals**

**Topics to Master:**

- Data types, `sizeof`, signed vs unsigned
    
- Pointers: declaration, dereference, pointer arithmetic
    
- Arrays & strings, multidimensional arrays
    
- Memory management: `malloc`, `free`, `strdup`
    
- Structs & unions
    
- Bit manipulation: masks, shifts, toggling bits
	```
	#define GPIO_OUT_REG (*(volatile uint32_t*)0x40020014)

	void set_pin(int pin) {
	    GPIO_OUT_REG |= (1U << pin);
	}
	
	void clear_pin(int pin)
	{
	    GPIO_OUT_REG &= ~(1U << pin);
	}
	
	void toggle_pin(int pin)
	{
	    GPIO_OUT_REG ^= (1U << pin);
	}
	```
	
    
- Function pointers
    
- Preprocessor macros and constants
    

**Common Coding Problems:**

1. Reverse a string in-place using pointers.
    
2. Implement your own `strdup` or `memcpy`.
    
3. Count set bits in an integer using bitwise operations.
    
4. Swap two integers without a temporary variable.
    
5. Merge two sorted arrays in-place or into a new array.
    

**Discussion Prep Questions:**

- Difference between `malloc` and `calloc`.
    
- Pros/cons of stack vs heap memory.
    
- What happens if you free a pointer twice?
    
- How to avoid buffer overflows.
    

**Mini-Practice:**  
Write a C function to reverse an integer array **without extra memory**, and then with dynamic memory.

---

## **2️⃣ Embedded Systems Fundamentals**

**Key Concepts:**

- Microcontrollers: GPIO, UART, SPI, I2C, ADC, PWM
    
- Timers & interrupts
    **Example of Timers**:
    ```
	    void timer_init(void)
	    {
	    }
	    
	    void __interupt() timer_isr(void)
	    {
		    if(TMR1IF)
		    {
			    TMR1IF = 0;
			    system_tick++;
		    }
	    }
    ```
- RTOS basics: tasks/threads, priorities, semaphores, mutexes
    
- Memory-mapped I/O
    
- Power modes and low-power design
    
- Embedded system constraints: CPU cycles, memory, power
    

**Sample Coding Problems:**

1. Toggle an LED every 500 ms using:
    
    - Bare-metal timer
        
    - RTOS task with delay
        
2. Implement a software debounce routine for a push button.
basic implementation without states:
```
	#define DEBOUNCE_TIME_MS 20
	
	volatile uint16_t debounce_counter = 0;
	volatile uint8_t button_state = 0;
	volatile uint8_t button_event = 0;
	
	void debounce_task(void)
	{
		static uint8_t last_raw = 1;//not pressed
		uint8_t raw = BUTTON_PORT;
		if(raw != last_raw)
		{
			debounce_counter = 0;
			last_raw = raw;
		}
		else
		{
			if(debounce_counter < DEBOUNCE_TIME_MS)
			{
				debounce_counter++;
			}
			else
			{
				if(button_state != raw)
				{
					button_state = raw;
					if(button_state == 0)
					{
						button_event = 1; //button pressed event
					}
				}
			}
		}
	}
```

Implementation with states:

```C
	type_def enum 
	{
		IDLE,
		DEBOUNCE_PRESS,
		PRESSED,
		DEBOUNCE_RELEASED
	}button_state_t;
	
	#define DEBOUNCE_TIME_MS 20
	
	volatile button_state_t btn_state = IDLE;
	volatile debounce_timer = 0;
	volatile button_event = 0; //flag for main app
	
	void debounce_task(void)
	{
		uint8_t raw = PORTbits.RB0;
		
		switch(btn_state)
		{
			case IDLE:
				if(raw == 0) // detected press
				{
					debounce_timer = 0;
					btn_state = DEBOUNCE_PRESS;
				}
				break;
			case DEBOUNCE_PRESS:
				if(raw == 0)
				{
					debounce_timer++;
					if(debounce_timer > DEBOUNCE_TIME_MS)
					{
						btn_state = PRESSED;
						button_event = 1;
					}
				}
				else
						btn_state = IDLE;
				break;
			case PRESSED:
				if(raw == 1) // detected press
				{
					debounce_timer = 0;
					btn_state = DEBOUNCE_RELEASED;
				}
				break;
			case DEBOUNCE_PRESS:
				if(raw == 1)
				{
					debounce_timer++;
					if(debounce_timer > DEBOUNCE_TIME_MS)
					{
						btn_state = IDLE;
						button_event = 0;
					}
				}
				else
						btn_state = PRESSED;
				break;
			default:
		}
	}

```

3. Write a function to read N ADC samples and return the average.
    

**Discussion Prep:**

- Difference between polling and interrupt-driven I/O.
    
- Pros and cons of RTOS vs bare-metal.
    
- How would you implement a priority inversion solution? (Semaphore + priority inheritance)
    
- Explain DMA and why it’s useful.
    

---

## **3️⃣ Control Algorithms (Bang-Bang & PID)**

**Bang-Bang Control**

- On/off controller, simple and deterministic
    
- Deadband/hysteresis reduces actuator chatter
    
- Example: heater, window limit switches, fan cutoff
    

**PID Control**

- Proportional: reacts to current error
    
- Integral: eliminates steady-state error
    
- Derivative: dampens overshoot
    
- Requires tuning: Ziegler–Nichols, manual, trial-and-error
    

**Coding / Pseudocode Problems:**

1. Implement bang-bang control with hysteresis.
    
2. Implement a basic PID controller in C.
    
3. Write a function to simulate temperature reaching a setpoint using bang-bang control.
    

**Discussion Prep:**

- Bang-bang vs PID: when to use each.
    
- What is integral windup and how to prevent it.
    
- How sampling rate affects PID performance.
    
- Limit cycles and oscillation explanations.
    

---

## **4️⃣ Testing & Validation**

**Topics to Know:**

- Unit testing embedded code (e.g., Ceedling, Unity)
    
- Hardware-in-the-loop (HIL) testing
    
- Edge case and boundary testing
    
- Test-driven development (TDD) in embedded
    
- Assertions and logging for debugging
    

**Sample Discussion Questions:**

- How would you test a GPIO driver?
    
- What test strategies would you use for a motor controller?
    
- How do you validate timing-critical code?
    

---

## **5️⃣ Coding Interview Strategy**

**Practice Problems:**

- String & array manipulation in C
    
- Bitwise operations (masking, toggling, extracting bits)
    
- Implement simple algorithms: reverse, sort, merge
    
- Small embedded-style challenges:
    
    - Simulate a sensor reading and generate control output
        
    - Write interrupt-safe code snippets
        
    - Implement a circular buffer
        

**Tips:**

- Write **clear, readable C code**, not just “it works.”
    
- Comment key logic (interviewers like clarity over cleverness)
    
- Think aloud: explain why you choose a pointer, loop, or algorithm
    
- Handle edge cases explicitly
    

---

## **6️⃣ Mock Discussion Topics**

- Explain how an RTOS schedules tasks
    
- Discuss bang-bang vs PID and give a real example
    
- Describe a previous embedded project: peripherals, timing, debugging
    
- How you would optimize memory usage for a constrained microcontroller
    
- How you ensure code safety and reliability in automotive systems
    

---

## **7️⃣ Study Plan**

|Day|Focus|
|---|---|
|Day 1|C fundamentals + pointers + memory management + bit manipulation|
|Day 2|Embedded systems: GPIO, UART, timers, interrupts, RTOS basics|
|Day 3|Control algorithms: bang-bang & PID (code + discussion)|
|Day 4|Testing & validation strategies + mock discussion answers|
|Day 5|Mixed coding problems + time yourself + review weak areas|

**Practice Tip:**  
Use **small, realistic embedded problems** instead of abstract LeetCode-style arrays. Interviewers care about **hardware-aware thinking**.

---

If you want, I can also **make a condensed “cheat sheet” PDF** with **C code snippets, bang-bang & PID pseudocode, and key embedded discussion points** that you could quickly review right before the interview.