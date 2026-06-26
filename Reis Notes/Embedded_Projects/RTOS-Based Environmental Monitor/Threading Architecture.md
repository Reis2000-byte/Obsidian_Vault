
### Startup Sequence

Before the scheduler starts, `main()` runs bare-metal and sets up everything in order:

```
HAL_Init()
SystemClock_Config()        ← peripherals need the clock first
MX_GPIO/I2C/SPI/UART_Init()
HAL_UART_Receive_IT(...)    ← arms the first RX interrupt (pre-scheduler)
osKernelInitialize()        ← RTOS kernel ready but not running
osMessageQueueNew(...)      ← queue created before any task can use it
osThreadNew(x4)             ← tasks registered, not yet executing
osKernelStart()             ← hands control to scheduler, never returns
```

`osKernelStart()` is a one-way door. Everything after it in `main()` is unreachable dead code — the scheduler owns the CPU from that point.

---

### The Four Tasks

```
Priority:  HIGH          NORMAL          NORMAL            LOW
           ┌──────────┐  ┌───────────┐  ┌─────────────┐  ┌──────────┐
           │Sensor_   │  │UART_Task  │  │defaultTask  │  │LED_Task  │
           │Task      │  │           │  │             │  │          │
           │1KB stack │  │1KB stack  │  │512B stack   │  │512B stack│
           └──────────┘  └───────────┘  └─────────────┘  └──────────┘
```

**Sensor_Task** (main.c:578) — highest user priority, so the scheduler gives it CPU first whenever it is ready. It wakes up, writes dummy sensor values onto its own stack-local `SensorData_t`, pushes that struct into the queue, then sleeps until its next period. Because it has high priority, it will preempt both `UART_Task` and `LED_Task` mid-execution if it wakes up while they are running.

**UART_Task** (main.c:605) — sits permanently blocked on `osMessageQueueGet(..., osWaitForever)`. It consumes zero CPU while waiting. The moment `Sensor_Task` puts something in the queue, the scheduler unblocks `UART_Task`, but it won't actually run until `Sensor_Task` yields (either by sleeping via `osDelayUntil` or by being preempted). Since both `UART_Task` and `defaultTask` share `osPriorityNormal`, the scheduler round-robins between them when both are ready.

**LED_Task** (main.c:629)`. The blink is the visual proof the scheduler is alive — if the LED stops, something higher-priority is stuck in a busy loop.

**defaultTask** (main.c:558) — CubeMX boilerplate. Calls `MX_USB_HOST_Init()` once, then loops on `osDelay(1)`. That 1ms delay means it yields every tick, effectively idling while keeping the USB host stack alive.

---

### The Sensor Queue

```
Sensor_Task                              UART_Task
  ┌─────────────────┐                  ┌──────────────────┐
  │ SensorData_t    │                  │ SensorData_t     │
  │ on stack:       │  osMessageQueue  │ on stack:        │
  │ temp=23.4       │ ──────PUT──────► │ (copy received)  │
  │ humidity=10.0   │  (deep copy)     │                  │
  │ pressure=2.0    │                  │ snprintf → UART  │
  └─────────────────┘                  └──────────────────┘
```

`osMessageQueuePut` copies the struct **by value** into the queue's internal buffer (allocated at queue creation time from the FreeRTOS heap). `Sensor_Task` can immediately overwrite its local `data` variable — the queue owns its own copy. This is why no global variable is needed. The queue holds up to 10 messages, so if `UART_Task` is slow for several cycles the data isn't lost.

---

### Timing Bug Still Present

One thing worth flagging: the current `Sensor_Task` has a bug at main.c:593:

```c
uint32_t tick = osKernelGetTickCount();  // e.g. starts at tick=5
for(;;) {
    // ... do work ...
    tick += 1000;
    osDelay(tick);   // ← BUG: delays FOR `tick` ms, not UNTIL tick
}
```

`osDelay` takes a **relative** duration. So on iteration 1 you delay 1005ms, iteration 2 you delay 2005ms, etc. It gets slower every loop. The fix is `osDelayUntil(tick)` which delays until the **absolute** kernel tick reaches `tick` — that's what gives true 1-second periods regardless of how long the sensor read takes.