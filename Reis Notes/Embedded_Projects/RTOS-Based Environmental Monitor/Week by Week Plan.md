### Project 1 — Week-by-Week Plan

**RTOS-Based Environmental Monitor**

---

#### Week 1 — Environment Setup & Bare Metal Blink

**Goal: Get your toolchain working and shake off the rust.**

- Install STM32CubeIDE and CubeMX if not already set up
- Create a new project for the Nucleo-F446RE, get familiar with the CubeMX pin/clock configurator
- Enable a GPIO output and write a bare-metal LED blink — no HAL shortcuts, toggle the register yourself at least once so you remember how it works
- Read the F446RE datasheet section on GPIO and clock trees
- Set up a GitHub repo, write a basic README, commit your blink

**Checkpoint:** LED blinks, code is on GitHub, you remember what a clock prescaler does.

---

#### Week 2 — UART & Command Parser

**Goal: Get serial communication working and build a simple shell.**

- Enable USART2 in CubeMX (it routes to the Nucleo's onboard ST-Link USB, so no extra wiring needed)
- Implement interrupt-driven UART RX — do not use blocking `HAL_UART_Receive`
- Build a minimal command parser: receive a string, compare against known commands (`READ`, `HELP`, `CLEAR`), respond accordingly
- Echo characters back as they're typed so it feels like a real terminal
- Test with PuTTY or Tera Term at 115200 baud

**Checkpoint:** You can type `READ` into a terminal and get a response back. Receiving is interrupt-driven.

---

#### Week 3 — FreeRTOS Introduction & Task Structure

**Goal: Port your project to FreeRTOS and think in tasks.**

- Enable FreeRTOS in CubeMX (CMSIS-RTOS v2 wrapper is fine to start)
- Create 3 tasks with appropriate priorities:
    - `SensorTask` — High priority, reads sensor data on a fixed interval
    - `UARTTask` — Medium priority, handles transmit queue and command responses
    - `LEDTask` — Low priority, heartbeat blink to show the system is alive
- Create a FreeRTOS Queue between `SensorTask` and `UARTTask` to pass sensor readings — no global variables
- Use `vTaskDelayUntil` in `SensorTask` for precise periodic timing, not `vTaskDelay`

**Checkpoint:** Three tasks running, LED heartbeat visible, queue passing dummy data between tasks, scheduler confirmed working via debug printf.

---

#### Week 4 — I²C & BME280 Driver

**Goal: Read real sensor data over I²C.**

- Wire up a BME280 breakout board (Adafruit or SparkFun) to the Nucleo's I²C1 pins
- Enable I²C1 in CubeMX at 400kHz (Fast Mode)
- Write a minimal BME280 driver from scratch — don't copy a library:
    - `BME280_Init()` — verify chip ID register (0x60), configure oversampling and mode
    - `BME280_ReadRaw()` — burst read the 6 raw data registers
    - `BME280_Compensate()` — implement the compensation formulas from the datasheet (temperature first, then pressure and humidity since they depend on temp)
- Feed real temperature, humidity, and pressure values into your sensor queue

**Checkpoint:** Real BME280 readings printing to your terminal in a formatted string like `T: 23.4C | H: 45.2% | P: 1013.2hPa`.

---

#### Week 5 — ADC & Light Sensor

**Goal: Add an analog channel and tie everything together.**

- Wire a photoresistor (LDR) + 10kΩ resistor as a voltage divider to an analog-capable pin
- Enable ADC1 in CubeMX, configure for single-channel, software-triggered conversion
- Write an `ADC_ReadLightLevel()` function that converts the raw 12-bit count to a human-readable lux approximation or simply a percentage
- Add a fourth data field to your sensor struct for light level
- Add a `LIGHT` command to your UART parser that returns the current reading on demand
- Add a simple threshold: if light drops below a value, the heartbeat LED changes blink rate

**Checkpoint:** Terminal shows all four readings. LED blink rate changes when you cover the LDR. You understand the ADC conversion formula.

---

#### Week 6 — Polish, Power & Documentation

**Goal: Turn a working prototype into a portfolio piece.**

- Add a mutex around any shared data structures accessed by multiple tasks
- Implement a `STATUS` UART command that returns free heap size (`xPortGetFreeHeapSize()`), task count, and uptime in seconds — this shows you understand RTOS internals
- Add error handling: if the BME280 doesn't ACK on I²C init, print a meaningful error and blink an error pattern instead of silently hanging
- Write a solid `README.md` with a wiring diagram (even a hand-drawn photo is fine), build instructions, and a screenshot of the terminal output
- Record a short demo video (phone is fine) and link it in the README

**Checkpoint:** Clean, commented code on GitHub. Someone else could clone it and build it. You can explain every line.

---

### At a Glance

|Week|Focus|Key Deliverable|
|---|---|---|
|1|Toolchain + GPIO|Bare metal blink on GitHub|
|2|UART|Interrupt-driven command parser|
|3|FreeRTOS|3 tasks + queue passing data|
|4|I²C + BME280|Real sensor readings in terminal|
|5|ADC|Light sensor + threshold behavior|
|6|Polish|Error handling, docs, demo video|

---

One piece of advice: **don't skip writing the BME280 driver yourself in Week 4.** It's tempting to grab a library, but working through the datasheet compensation formulas is exactly the kind of thing that separates a Level 2 from a Level 3 engineer. That struggle is the growth.