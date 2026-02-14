
| Problem                           | Difficulty | Comfort | Last Reviewed |
| --------------------------------- | ---------- | ------- | ------------- |
| 67. Add Binary                    | Easy       | ok      | 2/10/2026     |
| 136. Single Number                | Easy       | good    | 2/10/2026     |
| 191. Number of 1 Bits             | Easy       | good    | 2/10/2026     |
| 231. Power of Two                 | Easy       | good    | 2/10/2026     |
| 190. Reverse Bits                 | Easy       |         |               |
| 371. Sum of Two Integers          | Medium     | ok      | 2/10/2026     |
| 461 Hamming Distance              | Easy       | good    | 2/13/2026     |
| 268. Missing Number               | Easy       | ok      | 2/13/2026     |
| 201. bitwise AND of Numbers Range | Medium     | meh     | 2/13/2026     |
## **🟢 Easy – Fundamentals of Bits**

1. **[Single Number](https://leetcode.com/problems/single-number/)**
    
    - Problem: Every element appears twice except one. Find it using XOR.
        
    - Concepts: XOR to cancel duplicates, basic bitwise operations.
        
    - Embedded tie-in: Detecting faults, toggles, or flags.
        
2. **[Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/)**
    
    - Problem: Count the number of `1`s in a binary representation.
        
    - Concepts: Bit masking, shifting, `n & (n-1)` trick.
        
    - Embedded tie-in: Counting active flags, sensor states.
        
3. **[Power of Two](https://leetcode.com/problems/power-of-two/)**
    
    - Problem: Determine if a number is a power of 2.
        
    - Concepts: `n & (n-1) == 0`.
        
    - Embedded tie-in: Memory alignment checks, register validation.
        
4. **[Reverse Bits](https://leetcode.com/problems/reverse-bits/)**
    
    - Problem: Reverse bits of a 32-bit unsigned integer.
        
    - Concepts: Shifts, masking, loops.
        
    - Embedded tie-in: Communication protocols, endian conversion.
        

---

## **🟡 Medium – Manipulating & Isolating Bits**

5. **[Counting Bits](https://leetcode.com/problems/counting-bits/)**
    
    - Problem: Return an array with the number of `1`s for all numbers from 0 → n.
        
    - Concepts: Dynamic programming with bits, `i & (i-1)` trick.
        
    - Embedded tie-in: Efficient status monitoring of multiple binary sensors.
        
6. **[Missing Number](https://leetcode.com/problems/missing-number/)**
    
    - Problem: Find the missing number in 0…n using XOR.
        
    - Concepts: XOR properties, sum-of-series tricks.
        
    - Embedded tie-in: Error detection in sequences.
        
7. **[Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/)**
    
    - Problem: Implement addition without `+` or `-`.
        
    - Concepts: Bitwise addition with XOR and AND.
        
    - Embedded tie-in: Low-level arithmetic, CPU instructions, embedded ALU simulation.
        
8. **[Hamming Distance](https://leetcode.com/problems/hamming-distance/)**
    
    - Problem: Count differing bits between two numbers.
        
    - Concepts: XOR + bit counting.
        
    - Embedded tie-in: Data integrity checks, sensor comparison.
        

---

## **🔴 Hard – Advanced Bit Manipulation & Tricks**

9. **[Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)**
    
    - Problem: Find two numbers with the maximum XOR.
        
    - Concepts: Bitwise trie, greedy approach.
        
    - Embedded tie-in: Network protocols, encoding, checksum optimization.
        
10. **[Divide Two Integers](https://leetcode.com/problems/divide-two-integers/)**
    
    - Problem: Implement integer division without `/` or `*`.
        
    - Concepts: Bit shifts and subtraction loops.
        
    - Embedded tie-in: Low-level math in embedded firmware.
        
11. **[Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/)**
    
    - Problem: Compute AND of all numbers in a range.
        
    - Concepts: Bit manipulation, pattern recognition.
        
    - Embedded tie-in: Flag register aggregation, mask computation.
        
12. **[Reverse Bits in Byte/Word for Communication](Custom)**
    
    - Problem: Reverse bits in each byte for a 16/32-bit word.
        
    - Concepts: Shifts, masks, lookup table optimization.
        
    - Embedded tie-in: SPI/I2C protocol bit alignment, endian conversion.
        

---

## **💡 Tips for Embedded Interviews**

- Always think **hardware-aware**: is this bit representing a sensor, a GPIO, or a register flag?
    
- Practice **looping vs. non-looping solutions**: use `n & (n-1)` or lookup tables for efficiency.
    
- Use **C-style bit operations** rather than relying on high-level Python/Java shortcuts.
    
- Be ready to **explain every operation**, especially shifts and XOR.
    

---

If you want, I can **make a compact “embedded bit manipulation cheat sheet”** with **all these tricks, masks, XOR patterns, and C code snippets** so you can memorize it before the interview.

Do you want me to do that?

Yes

Perfect! I’ve put together a **compact, interview-ready embedded bit manipulation cheat sheet**. This is **all in one page** you can memorize or keep open while prepping. I’ll include **tricks, C code snippets, and explanations** for Apple-style embedded interviews.

---

# **🔹 Embedded Bit Manipulation Cheat Sheet**

---

## **1️⃣ Basic Bit Operations**

|Operation|C Code|Description|
|---|---|---|
|Set bit `k`|`n|= (1 << k);`|
|Clear bit `k`|`n &= ~(1 << k);`|Sets bit k to 0|
|Toggle bit `k`|`n ^= (1 << k);`|Flips bit k|
|Check bit `k`|`(n >> k) & 1`|Returns 0 or 1|
|Mask multiple bits|`n & mask`|Keeps bits where mask=1|
|Assign bit|`n = (n & ~(1<<k))|(val<<k);`|

---

## **2️⃣ Common Bit Tricks**

- **Check power of 2:**
    

`bool isPowerOf2(int n) { return n > 0 && (n & (n-1)) == 0; }`

- **Count 1 bits:**
    

`int countBits(int n) {     int count = 0;     while(n) {         n &= (n-1); // removes lowest set bit         count++;     }     return count; }`

- **Isolate lowest set bit:**
    

`int lowestBit = n & -n;`

- **XOR basics:**
    

`x ^ x = 0 x ^ 0 = x`

- Useful for finding single numbers or swapping:
    

`a ^= b; b ^= a; a ^= b;`

- **Swap without temp variable** (XOR trick):
    

`a ^= b; b ^= a; a ^= b;`

- **Reverse bits (32-bit)**:
    

`uint32_t reverseBits(uint32_t n) {     uint32_t rev = 0;     for(int i=0;i<32;i++){         rev <<= 1;         rev |= (n & 1);         n >>= 1;     }     return rev; }`

---

## **3️⃣ XOR Patterns**

|Pattern|Usage|
|---|---|
|`x ^ y ^ y = x`|Remove duplicate|
|`x ^= y`|Toggle bits|
|`x ^ 0 = x`|Identity|
|`x & (~x) = 0`|Zero mask|

---

## **4️⃣ Bit Masks**

- **Create mask from bit i to j:**
    

`int mask = ((1 << (j-i+1)) - 1) << i;`

- **Apply mask:**
    

`int value = n & mask;`

- **Shift into position:**
    

`int value = (n & mask) >> i;`

- Embedded use: set/clear GPIO pins, flags, register fields.
    

---

## **5️⃣ Common Interview Problems & Quick Patterns**

|Problem|Pattern / Trick|
|---|---|
|Single number in array|XOR all elements|
|Missing number 0…n|XOR all indices and elements|
|Count set bits|`n & (n-1)` loop|
|Power of 2|`n & (n-1) == 0`|
|Reverse bits|Shift + OR loop|
|Bitwise AND of range|Shift until MSBs equal|
|Sum of two ints without +|Use XOR + AND shifts iteratively|

---

## **6️⃣ Embedded Examples**

**1. Toggle GPIO pin k**

`GPIO_REG ^= (1 << k);`

**2. Read sensor flags**

`bool sensorActive = (SENSOR_REG >> SENSOR_BIT) & 1;`

**3. Set motor control bits safely**

`MOTOR_CTRL = (MOTOR_CTRL & ~MASK_BITS) | (newVal & MASK_BITS);`

**4. Detect overflow in 8-bit counter**

`bool overflow = !(counter & 0x80); // example: high bit indicates overflow`

---

## **7️⃣ Tips for Embedded Interviews**

1. **Think hardware-first:** every bit often corresponds to a pin, sensor, or register.
    
2. **Explain your shifts:** interviewers love clarity.
    
3. **Use masks instead of arithmetic** when dealing with flags/registers.
    
4. **Know `n & (n-1)` and `n & -n`**—classic embedded tricks.
    
5. **Always handle edge cases** (0, max int, negative numbers).