### How they'll likely see you

Your strengths are 3+ years of embedded C, RTOS work (FreeRTOS, Zephyr), bare-metal drivers, leading architecture, and writing specs and test plans. Ceedling is a direct match for "xUnit-compliant test frameworks," since it's built on Unity. Your specs and verification background also suits spaceflight, which runs on requirements and traceability.

The gaps they'll probe:

- **C++** isn't on your resume.
- **Linux/POSIX** experience is light (just the AIG internship).
- **Serial and IP protocols** aren't on your resume at all.
- **cFS** you probably haven't used.

Expect a lot of their questions to aim at these four areas. Your HVAC work probably covers more than the resume shows. BAS integration usually means BACnet MS/TP or Modbus over RS-485, or BACnet/IP, and your DDC web interface ran over IP. If you touched any of that, say so plainly. It's direct evidence for two required skills.

### Questions they'll probably ask you

**About your resume**

1. Walk me through your background in about two minutes. Why move from HVAC to spaceflight?
2. Walk me through the rooftop-unit control platform architecture. How did you split tasks, what talked to what, and what would you change?
3. On the tandem compressor firmware, what was the hardest bug you hit, and how did you find it?
4. You wrote low-memory drivers for the defrost prototype. What constraints did you work under, and what trade-offs did you make?
5. How did you verify the outsourced controls met your spec? What happened when they didn't?
6. Walk us through your RTOS environmental monitor on GitHub. Why FreeRTOS, how many tasks, and how do they communicate? Be ready to screen-share it.
7. Where is the CAN bus project? How are you handling frame parsing and bus errors?

**Embedded C fundamentals**  
8. What do `volatile`, `static` (inside a function vs. at file scope), and `const` do? What's the difference between `const int *p` and `int * const p`?  
9. Why do flight software standards often ban dynamic allocation after init?  
10. What should and shouldn't happen inside an ISR? How do you hand data from an ISR to a task?  
11. Mutex vs. binary semaphore: what is priority inversion, and how do you prevent it? They may bring up the Mars Pathfinder reset bug, so know that story.  
12. Struct padding and packing: why do they matter when you overlay a struct on a received packet?  
13. What is endianness? How do you safely pull a 16-bit big-endian field out of a byte buffer?  
14. How do watchdogs work, and how would you design one for a multi-task system?

**Protocols** (required skills, so expect these)  
15. RS-232 vs. RS-422 vs. RS-485. The core answer: 232 is single-ended and point-to-point. 422 and 485 are differential, so they handle longer runs and resist noise. 485 is multi-drop and usually half-duplex.  
16. How would you design a UART receive path that never drops bytes? (Ring buffer filled by the ISR or DMA, plus framing and a CRC.)  
17. TCP vs. UDP: which would you use for telemetry, which for commands, and why?  
18. How do you detect a corrupted or partial packet?

**C++, Linux, and testing**  
19. What C++ features would you use in flight code, and which would you avoid? Use: RAII, classes, templates, `constexpr`. Often avoided: exceptions, RTTI, heap use.  
20. Have you worked with threads, sockets, or `select`/`poll` on Linux? How does POSIX threading compare to FreeRTOS tasks?  
21. How do you unit test code that touches hardware? (Hardware abstraction layers, mocks with CMock in Ceedling.)  
22. How do you decide what's tested at unit level versus hardware-in-the-loop?  
23. Have you heard of NASA's cFS? What do you know about it? An honest "I've studied the architecture" beats bluffing.

**Behavioral**  
24. Tell us about a time you shipped something under schedule pressure without cutting quality. (The JD explicitly asks for high quantity _and_ high quality output.)  
25. Tell us about a disagreement with another engineering discipline.  
26. Tell us about a mistake that reached the field or the lab. What did you change afterward?  
27. How do you get up to speed on a codebase or framework you've never seen?

### Simple prep tasks for tonight (about 3 hours total)

1. **Two-minute intro (15 min).** Write it out and say it aloud three times. Cover your HVAC controls lead role, then embedded C, RTOS, and test discipline, then why spaceflight now. Living in Houston near a lunar company makes a natural hook.
2. **Code a ring buffer in C (30 min).** Use a fixed size, power-of-two indexing, and push/pop with full and empty detection. Then ask yourself what breaks if an ISR pushes while a task pops. This is a very common live exercise.
3. **Parse a packet header (30 min).** Write a function that takes a `uint8_t*` and extracts fields with shifts and masks, not struct casts. A good practice target is the 6-byte CCSDS space packet primary header that cFS uses: version (3 bits), type (1), secondary header flag (1), APID (11), sequence flags (2), sequence count (14), and length (16). Mentioning CCSDS by name will land well.
4. **Read up on cFS (20 min).** Learn the layers: OSAL, PSP, cFE core services (Software Bus, Event, Executive, Table, Time), and apps that communicate by publish/subscribe over the Software Bus. That's enough to discuss it intelligently.
5. **C++ refresh (20 min).** Review RAII, virtual functions and vtables, `constexpr`, and why embedded C++ often runs with exceptions off. Prepare one honest line, such as "My production work has been C, and here's how I'm ramping on C++."
6. **Four STAR stories (30 min).** Write one each for a hard debug, leading architecture, verifying a vendor's work, and a mistake you owned. Map each one to several behavioral questions.
7. **Company context (15 min).** Intuitive Machines is building Altus, a planned five-satellite lunar communications relay constellation, and expects to link its IM-3 mission and first lunar relay satellite into a unified communications and navigation network later this year. NASA picked them under the Near Space Network for direct-to-Earth and lunar relay capabilities. They've also been growing through acquisitions, and closed the Goonhilly Earth Station and COMSAT acquisition in August to expand their space-to-ground data network. [altus 1 +3](https://space.skyrocket.de/doc_sdat/altus-1.htm)
8. **Logistics (10 min).** Test the Teams link, camera, and mic tonight. Plan to join at 11:50, and keep your GitHub repo open in a tab. One small thing: the repo name has a typo ("enviormental"). Renaming it tonight is optional, but they may click it.

### Questions to ask them

- Does this role sit on the Altus relay satellites, landers, or ground-side software?
- Is the flight stack built on cFS? What's the flight processor and OS?
- What does hardware-in-the-loop testing look like? Do engineers get time in the lab with real hardware?
- What would a strong first 90 days look like for this person?
- How does the team balance schedule pressure with flight-quality reviews?