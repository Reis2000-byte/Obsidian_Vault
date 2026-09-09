# ✅ What is the ternary operator in C?

The **ternary operator** is a compact `if-else` expression.

### Syntax

`condition ? expression_if_true : expression_if_false;`

Think of it as:

> “If condition is true, use this; otherwise, use that.”

---

# 🧠 Basic Example

`int a = 10, b = 20; int max = (a > b) ? a : b;`

This means:

`if (a > b)     max = a; else     max = b;`

---

# ✅ Common Use Cases (what you REALLY need to know)

## 1️⃣ Assigning a value based on a condition

`int abs = (x < 0) ? -x : x;`

## 2️⃣ Printing conditionally

`printf("%s\n", (age >= 18) ? "Adult" : "Minor");`

## 3️⃣ Embedded systems (very common)

`gpio_write(pin, value ? HIGH : LOW);`

---

# ⚠️ Important Rules (test & interview stuff)

## ✔ It returns a VALUE

The ternary operator is an **expression**, not a statement.

So you can do:

`x = condition ? 5 : 10;`

But you **cannot** do:

`condition ? printf("Yes");  // ❌ missing else part`

---

## ✔ Both sides must be compatible types

`int x = cond ? 5 : 3;        // OK float y = cond ? 5.0 : 3.2;  // OK`

Bad:

`int x = cond ? 5 : 3.2; // implicit conversion happens`

---

## ✔ Operator precedence is LOW

Use parentheses when unsure:

`int x = (a > b) ? a : b;`

---

# 🔥 Nested Ternary (avoid unless simple)

`int grade = (score > 90) ? 'A' :             (score > 80) ? 'B' :             (score > 70) ? 'C' : 'F';`

This works but **kills readability**.

---

# 🚨 When NOT to use ternary

❌ Complex logic  
❌ Multiple statements  
❌ Side-effect heavy code

Instead use `if-else`.

---

# 🧩 Key Interview Takeaways

- Ternary is the **only ternary operator in C** (3 operands).
    
- It is an **expression**, unlike `if`.
    
- Often used for **simple conditional assignments**.
    
- Can be nested, but readability suffers.
    
- Common in embedded code for register configuration and macros.
    

---

# ⚡ Embedded C Pro Tip

Ternary is often used in macros:

`#define MAX(a,b) ((a) > (b) ? (a) : (b))`

This is why you MUST understand it for firmware work.

---

# 🧠 Super Simple Mental Model

`condition ? true_value : false_value`