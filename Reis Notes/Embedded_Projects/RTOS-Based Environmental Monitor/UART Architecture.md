## Step 1 — Startup: UART is configured

main.c:116  MX_USART2_UART_Init()` is called, which sets up USART2 at:
 
 115200 baud, 8 data bits, 1 stop bit, no parity
- This tells the hardware peripheral _how fast_ to read the serial line and _what format_ each byte is in

---

## Step 2 — The listener is armed

main.c:118
```c
HAL_UART_Receive_IT(&huart2, &rxByte, 1);
```

This tells the UART hardware: _"When one byte arrives on the RX pin, store it in `rxByte` and fire an interrupt."_ The CPU then goes off and does other things (the blink loop).

---

## Step 3 — You type a character in the terminal

Each keystroke sends one ASCII byte over the serial wire at 115200 baud. The USART2 hardware peripheral receives that byte entirely in silicon — no CPU involvement yet. When the byte is fully received, the hardware raises the **RXNE** (Receive Not Empty) interrupt.

---

## Step 4 — The ISR fires, HAL calls your callback

The NVIC (interrupt controller) pauses whatever the CPU was doing (the blink loop) and jumps into the HAL UART interrupt handler. That handler sees the byte was received, copies it out of the hardware register into `rxByte`, then calls:

main.c:422

```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
```

---

## Step 5 — The callback echoes the character and buffers it

main.c:424-438

It first checks `huart->Instance == USART2` to make sure this is your UART (not I2C or SPI accidentally triggering it).

Then two paths:

- **Not Enter (`\r`):** The byte is echoed back to the terminal (`HAL_UART_Transmit`) so you _see_ what you typed, and stored into `rx_buffer[rxIndex++]`. This is why characters appear on screen as you type.
- **Enter (`\r`):** A newline `\r\n` is sent, the buffer is null-terminated with `'\0'`, and `processCommand()` is called.

---

## Step 6 — The listener is re-armed

main.c:437

```c
HAL_UART_Receive_IT(&huart2, &rxByte, 1);
```

This is called at the **end of every callback**, whether it was Enter or not. This is critical — HAL's interrupt-mode receive is **one-shot**: after each byte it disarms itself. You re-arm it here so the next character can be received. If you forgot this line, UART input would stop after the first byte.

---

## Step 7 — You press Enter: `processCommand` runs

main.c:441-464

The null-terminated string in `rx_buffer` is compared using `strcmp`. Depending on what you typed:

|You typed|Response sent back|
|---|---|
|`READ`|`"Reading sensors...\r\n"`|
|`HELP`|`"Commands: READ, HELP, CLEAR\r\n"`|
|`CLEAR`|`"\033[2J\033[H"` (ANSI escape — clears screen)|
|anything else|`"Unknown Command\r\n"`|

After `processCommand` returns, `rxIndex` is reset to 0 so the buffer is ready for the next command.

---

## Visual summary

```
Terminal keystroke
      |
      v
USART2 hardware RX pin (115200 baud)
      |
      v
RXNE interrupt fires → NVIC pauses blink loop
      |
      v
HAL_UART_RxCpltCallback()  [main.c:422]
      |
      +-- not '\r' → echo char + buffer it  [main.c:434-435]
      |
      +-- '\r'     → send \r\n, null-terminate, processCommand()  [main.c:428-431]
                                    |
                                    +-- strcmp → transmit response
      |
      v
Re-arm: HAL_UART_Receive_IT()  [main.c:437]
      |
      v
CPU returns to blink loop
```


There are **two completely separate UART paths** — one interrupt-driven for receiving commands, one task-driven for transmitting sensor data. They share `huart2` but operate independently.

---

### Path 1 — Interrupt RX (Command Input)

```
User types         UART peripheral     HAL ISR              Callback
a character  ──►   shifts in byte  ──► USART2_IRQHandler ──► HAL_UART_RxCpltCallback
                                       (HAL handles this)     (your code runs here)
```

**How it stays alive:**

`HAL_UART_Receive_IT` is a one-shot arm. Every time a byte arrives and the callback fires, the very last line (main.c:521) re-arms it:

```c
HAL_UART_Receive_IT(&huart2, &rxByte, 1);  // re-arm for next byte
```

Without this line, the interrupt would fire once and go silent forever.

**Inside the callback** (main.c:498):

```
Byte arrives
    │
    ├── '\r' ? ──► echo "\r\n"
    │               null-terminate rx_buffer
    │               call processCommand(rx_buffer)   ← blocking TX happens here
    │               reset rxIndex = 0
    │
    └── printable char? ──► echo the char back (blocking TX)
                            append to rx_buffer[rxIndex++]
                            (if buffer full → send bell '\a')
```

**Important detail:** `HAL_UART_Transmit` inside the callback is **blocking** — it spins until the byte is shifted out. This runs in ISR context, so it holds the CPU and delays every other interrupt for the duration of the transmit. For short echo responses this works, but it's why you'd eventually want DMA or a TX queue for anything longer.

**`processCommand`** (main.c:525) also runs in ISR context — `strcmp` + `HAL_UART_Transmit` for the response string.

---

### Path 2 — Task-Based TX (Sensor Data)

```
Sensor_Task puts data      UART_Task unblocks         HAL_UART_Transmit
into queue           ──►   formats with snprintf  ──►  shifts bytes out
                           builds string on stack       (blocking, in task context)
```

This is safe to do with blocking `HAL_UART_Transmit` because it runs inside a task, not an ISR. While the bytes are being shifted out, the FreeRTOS tick ISR can still fire and the scheduler can preempt once the transmit finishes (or if a higher-priority task wakes up mid-transmit — though HAL_UART_Transmit is not reentrant so that would be a problem if `UART_Task` were ever called from multiple places).

---

### The Two Paths Side by Side

```
                        ┌──────────────────────────────────┐
                        │           huart2 (USART2)        │
                        └──────────┬───────────────────────┘
                                   │
              ┌────────────────────┴──────────────────────┐
              │                                           │
           RX path                                    TX path
    (interrupt-driven)                          (two senders)
              │                                           │
    HAL_UART_RxCpltCallback                    ┌──────────┴──────────┐
         (ISR context)                         │                     │
              │                           Callback echo        UART_Task
      echo + processCommand               (ISR context)        (task context)
      (blocking TX in ISR)                blocking TX          blocking TX
```

The practical consequence: if `UART_Task` is in the middle of transmitting a sensor reading and a character arrives, the RX interrupt fires and tries to also transmit an echo — both going through `huart2`. HAL's internal state machine should handle this since TX and RX are independent USART flags, but it's a subtle interaction to be aware of as the project grows.