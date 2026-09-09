

**Board:** STM32F407G-DISC1 • **RTOS:** FreeRTOS (CMSIS-RTOS2) • **Target bus rate:** 500 kbps (OBD-II)

A 7-week build that takes you from "never touched CAN" to a working device that reads live engine data out of a real car. The plan is front-loaded so that **everything through Week 3 runs on your board alone in loopback mode** — no transceiver, no car, nothing extra to buy. You only add hardware once the firmware logic is already proven.

---

## How to use this

- **Assumed pace:** ~6–10 hrs/week. Going faster? Merge weeks. Slower? Each week is a self-contained checkpoint, so stretching is fine.
- Each week has a **Goal**, **Tasks**, **What you learn**, and a **Done when** checkpoint. Don't move on until the checkpoint passes — CAN bugs compound quickly if you build on a shaky layer.
- You'll reuse three things from your environmental monitor: the **queue-based producer/consumer** pattern, **`osDelayUntil`** for periodic polling, and the **UART command parser** as your control interface. Where you see "reuse," you're extending code patterns you already wrote.

## Materials

| Item                                                     | Needed by | Notes                                                                              |
| -------------------------------------------------------- | --------- | ---------------------------------------------------------------------------------- |
| STM32F407G-DISC1                                         | Week 1    | You have it                                                                        |
| CAN transceiver (SN65HVD230 or MCP2551)                  | Week 4    | ~$2. SN65HVD230 is 3.3 V — preferred. MCP2551 is 5 V, needs level care             |
| 120 Ω resistors ×2                                       | Week 4    | Bus termination                                                                    |
| ELM327-free OBD-II-to-DB9/wire cable, or OBD-II breakout | Week 5    | You want raw CAN access to pins 6 (CAN-H) and 14 (CAN-L), **not** an ELM327 dongle |
| A car made in 2008 or later                              | Week 5    | 2008+ mandated CAN for OBD-II in the US                                            |

> **Pin choice (decide in Week 1):** CAN1's default alternate-function pins PA11/PA12 collide with USB OTG FS. To keep your USB projects clean later, map **CAN1 to PB8 (RX) / PB9 (TX)** using AF9 instead. Note PB9 is used by your monitor's I2C — fine, since these are different projects, but don't wire both at once.

---

## Week 1 — CAN fundamentals + first frame in loopback

**Goal:** Transmit a single CAN frame that your own controller receives, entirely on-chip.

**Tasks**

- Read the CAN sections of the F407 reference manual (bxCAN chapter) and skim the CAN 2.0 frame format. Focus on: frame structure (ID, DLC, data), and what "arbitration" means.
- In CubeMX, enable CAN1, set mode to **Loopback**, map to PB8/PB9.
- Compute **bit timing** for 500 kbps by hand from your APB1 clock. Verify your APB1 clock first (commonly 42 MHz on this board at 168 MHz sysclk). Formula: `bit_rate = APB1_clk / (Prescaler × (1 + BS1 + BS2))` Worked example @ 42 MHz APB1: Prescaler = 6, BS1 = 11 tq, BS2 = 2 tq, SJW = 1 → 42e6 / (6 × 14) = **500 kbps**, sample point ≈ 85.7%.
- Write a task that transmits one frame (any ID, a few data bytes) once per second using a TX mailbox.

**What you learn:** CAN frame anatomy; bit-timing math and the sample-point concept; how bxCAN differs from a point-to-point peripheral. This is the deepest peripheral-config work you've done — you're setting timing at the bit level, not calling an init and hoping.

**Done when:** You can set a breakpoint or watch variable and confirm the frame you transmitted appears in the receive FIFO. First self-loop closed.

---

## Week 2 — Interrupt-driven receive + hardware acceptance filters

**Goal:** Receive frames via interrupt, and make the hardware filter out IDs you don't care about before your code runs.

**Tasks**

- Switch RX from polling to **interrupt-driven**: enable the FIFO-0 message-pending interrupt, write the ISR, and have it pull the frame out fast (copy to a buffer, signal a task — don't do work in the ISR).
- Configure an **acceptance filter bank**: set it to accept only ID `0x7E8` (the OBD-II ECU response ID you'll need later) and reject everything else. Transmit a mix of IDs in loopback and confirm only `0x7E8` reaches your code.
- Experiment: change the filter to mask mode vs list mode and observe the difference.

**What you learn:** Interrupt-driven RX off hardware FIFOs (same discipline as your non-blocking UART, busier bus); hardware ID filtering — letting silicon do the work so the CPU isn't woken for irrelevant traffic. This "push work into hardware" instinct is a hallmark embedded skill.

**Done when:** With the filter active, frames with non-matching IDs never reach your ISR, and matching frames arrive via interrupt with zero polling.

---

## Week 3 — Fold CAN into your FreeRTOS architecture

**Goal:** Restructure into clean RTOS tasks using the exact producer/consumer shape from your monitor.

**Tasks**

- Create a **`CAN_RX` task** and a **`CAN_Decode` task** connected by an `osMessageQueue`. The ISR signals `CAN_RX`; `CAN_RX` hands frames to `CAN_Decode` through the queue. (This is your Sensor→UART pattern, reused.)
- Extend your **UART command parser** so you can type a command over the terminal to trigger a CAN transmit (e.g. `PING` sends a test frame). You now have a live console for your CAN node.
- Add an **LED bus-status indicator**: green heartbeat on successful RX, reusing your `LED_Task` idea as a health signal.

**What you learn:** How a new peripheral slots into an existing RTOS design; that your queue/task architecture generalizes beyond one project. Nothing new _conceptually_ — this week is about clean integration, which is itself an interview-worthy skill.

**Done when:** Typing a command over UART sends a frame, the RX path receives it through the task/queue chain, and the LED confirms it — all still in loopback.

---

## Week 4 — Go physical: transceiver + a real two-node bus + error handling

**Goal:** Leave loopback. Put real CAN signals on real wires and handle bus errors.

**Tasks**

- Wire the **SN65HVD230 transceiver** to PB8/PB9. Build a minimal 2-node bus: either a second MCU, or a friend's board — with **120 Ω termination at both ends**. Switch CAN mode from Loopback to **Normal**.
- Get two nodes talking: one transmits, the other receives and lights an LED.
- Implement **bus-off detection and recovery**: read the error counters, detect when the node goes bus-off, and re-integrate. Test it by deliberately disconnecting a wire mid-transmission.

**What you learn:** Real CAN electrical behavior (termination matters — skip it and you get garbage); protocol-level fault handling. Your monitor's sensor disconnect-and-recover logic has a big brother here — this is the same recovery instinct one layer down, at the bus level.

**Done when:** Two physical nodes exchange frames on a terminated bus, and yanking a wire triggers your bus-off recovery instead of a silent hang.

> If you can't get a second node, you can still validate the transceiver by scoping PB8/PB9 or using a USB-CAN analyzer as the second node. But two-node is the better learning experience.

---

## Week 5 — OBD-II: talk to a real car

**Goal:** Plug into a vehicle and read live engine data.

**Tasks**

- Wire your transceiver to the car's OBD-II port: **pin 6 = CAN-H, pin 14 = CAN-L, pin 4/5 = ground**. Confirm 500 kbps, 11-bit IDs.
- Send a **single-frame PID request** to ID `0x7DF` and read the reply from `0x7E8`. Start with these three:
    - **RPM** — PID `0x0C`, decode `((A×256)+B)/4`
    - **Speed** — PID `0x0D`, decode `A` (km/h)
    - **Coolant temp** — PID `0x05`, decode `A − 40` (°C)
- Print decoded values over UART, once per second, using `osDelayUntil` for a drift-free poll (reused from your monitor).

**What you learn:** A real request/response protocol layered on CAN; the discipline of matching a reply to its request; reading a published spec (the OBD-II PID tables) and turning it into decode code. This is the payoff week — your board pulls real numbers out of a running engine.

**Done when:** With the engine running, your terminal shows live RPM that changes when you rev it, plus speed and coolant temp.

> **Safety:** Only _read_ PIDs (mode 01). Do not transmit arbitrary frames onto a live vehicle bus — writing to the wrong ID can affect real modules. Reading is passive-safe; writing is not.

---

## Week 6 — ISO-TP multi-frame reassembly (the standout skill)

**Goal:** Handle responses too big for a single 8-byte frame by implementing the transport layer.

**Tasks**

- Study **ISO-TP (ISO 15765-2)**: Single Frame (SF), First Frame (FF), Consecutive Frame (CF), and Flow Control (FC).
- Implement a **reassembly state machine**: when a First Frame arrives, send a Flow Control frame, then collect and order the Consecutive Frames into one complete buffer.
- Test it against a multi-frame PID: request the **VIN** (mode `09`, PID `02`) or read stored **DTCs** (mode `03`) — both span multiple frames.

**What you learn:** Transport-layer protocol design; a non-trivial state machine with flow control and sequencing. This is a direct level-up of the UART command parser you wrote — same "parse a protocol into structured data" skill, one tier harder. It's also the single most impressive line on the finished resume bullet.

**Done when:** You can request and correctly reassemble your car's full VIN string from a multi-frame response.

---

## Week 7 — Polish, document, and package for your portfolio

**Goal:** Turn working code into a portfolio piece someone can evaluate in 60 seconds.

**Tasks**

- Add a **live display**: either drive an SSD1306 OLED with a rotating dashboard (RPM / speed / temp), or push readings over UART to a simple host-side plot. A visual others can _see_ in a screenshot or video is what makes it feel like a product.
- Write a **README** in the style of your monitor's: hardware table, wiring, the request/response flow, the PIDs supported, and — importantly — a short "design decisions" section (why interrupt-driven RX, why hardware filtering, how bus-off recovery works). Interviewers read that section.
- Record a **15-second demo video** of the numbers changing as you rev the engine.
- Clean up: `.ioc` committed, user code inside the CubeMX guards, clear commit history.

**What you learn:** Technical communication — arguably the highest-leverage skill for getting hired. The engineering is done; this week is about making it _legible_ to someone deciding whether to interview you.

**Done when:** A stranger could clone your repo, read the README, watch the video, and understand exactly what you built and why — without asking you a single question.

---

## Resume line you'll have earned

> _Implemented an interrupt-driven CAN bus node on STM32 with hardware ID filtering, bus-off recovery, and an ISO-TP transport layer to read live OBD-II engine telemetry (RPM, speed, coolant, VIN) from a vehicle._

That one sentence signals automotive-relevant, low-level, and protocol-literate — three things hiring managers scan for.

## Stretch goals (if you want to go further)

- **CAN sniffer mode:** passively log _all_ traffic on the bus (widen your acceptance filter) and reverse-engineer a non-standard message your specific car sends. This is real hacker cred and teaches you that most of a car's bus is undocumented.
- **DBC-style decoder:** parse a small message-definition format so decoding is data-driven instead of hardcoded.
- **Second bus with CAN2:** the F407 has two CAN controllers — bridge traffic between them.