### 📅 What is DTE?

DTE stands for **Days To Expiration.** It's simply how many days are left until your options contract expires.

That's it. Nothing complicated about the definition — but the **strategy behind choosing the right DTE** is where it gets interesting.

---

### ⏰ The Secret Weapon — Theta Decay

To understand DTE you first need to understand **Theta** — because this is what makes selling options profitable.

Theta is **time decay.** Every single day that passes, an options contract loses a little bit of value just from time passing — even if the stock doesn't move at all.

Think of it like an ice cube melting:

- When you first put it in the glass it melts slowly
- As it gets smaller it melts **faster and faster**
- The last bit disappears almost instantly

Options work exactly the same way. The premium decays **slowly at first, then accelerates rapidly** as expiration approaches.

---

### 📉 The Theta Decay Curve

```
Days Left:    Premium Remaining (roughly):
  45 DTE   →   100% (full value)
  30 DTE   →   ~75%
  21 DTE   →   ~60%
  14 DTE   →   ~45%
   7 DTE   →   ~25%
   1 DTE   →   ~5%
   0 DTE   →   $0
```

See how the decay **accelerates** in the final 2 weeks? That's the sweet spot sellers want to exploit.

---

### 🎯 The DTE Sweet Spots

#### 45 DTE — The Professional Standard

This is what most experienced options sellers use as their **entry point.**

|Factor|Detail|
|---|---|
|**Why 45?**|Premium is fat, theta decay is starting to accelerate|
|**Close at**|21 DTE or 50% profit (whichever comes first)|
|**Best for**|QQQ, SPY, large liquid stocks|
|**Risk**|More time = more things can go wrong|

You enter at 45 days, collect good premium, and close at 21 days when you've captured the best part of the decay curve. Then you open a new trade immediately. This is called **the 45/21 method.**

---

#### 30 DTE — The Beginner Sweet Spot ✅

This is **perfect for where you are right now.**

|Factor|Detail|
|---|---|
|**Why 30?**|Great balance of premium vs time exposure|
|**Close at**|50% profit or 14 DTE|
|**Best for**|SOFI, RIVN, SOUN, moderate volatility stocks|
|**Risk**|Manageable — not too long, not too short|

This is exactly what the QQQ spread quote was referring to earlier. 30 days gives you enough premium to make the trade worthwhile while keeping your capital tied up for a reasonable amount of time.

---

#### 7–14 DTE — The Weekly Trader

Short dated trades with fast premium decay.

|Factor|Detail|
|---|---|
|**Why short?**|Capital freed up quickly, fast theta burn|
|**Close at**|50% profit or let expire|
|**Best for**|High IV situations, post earnings plays|
|**Risk**|Stock moves hurt you FAST, less time to recover|

This is more advanced. A stock moving against you with only 7 days left has very little time to recover. Not recommended until you have more experience.

---

#### 0 DTE — Same Day Expiration

This is basically gambling for beginners. Professional day traders use it but it requires constant monitoring and nerves of steel. **Avoid completely for now.**

---

### ⚖️ The DTE Tradeoff Explained Simply

|DTE|Premium|Safety|Capital Tied Up|
|---|---|---|---|
|**60+ days**|Very fat|Most time to recover|Long time|
|**45 days**|Fat|Good cushion|~6 weeks|
|**30 days**|Moderate|Decent cushion|~1 month ✅|
|**14 days**|Thin|Little recovery time|~2 weeks|
|**7 days**|Very thin|Almost no cushion|~1 week|
|**0 days**|Tiny|Zero cushion|1 day|

---

### 🔄 How DTE Affects Your Strategy

#### Scenario A — Trade Goes Your Way

You sold a 30 DTE put and after 15 days the premium has dropped 50%. You **buy to close** and lock in profit. Your capital is freed up 15 days early and you go find the next trade. This is the ideal outcome.

#### Scenario B — Trade Goes Against You

Stock drops toward your strike at 20 DTE. You still have time — you can:

- **Hold** and wait for a recovery
- **Roll** to a later DTE to buy more time
- **Close** and take a small loss

The more DTE you have when things go wrong, the more options you have to manage the trade. This is why going too short on DTE is dangerous for beginners.

---

### 📆 Monthly Expirations vs Weekly

**Monthly expirations** (3rd Friday of every month) are the most liquid — this is where most of the open interest lives. For beginners always target monthly expirations because:

- Tighter bid/ask spreads
- More open interest
- Easier to fill orders at the midpoint

**Weekly expirations** exist but tend to have worse liquidity on smaller stocks. Fine for QQQ and SPY, riskier on individual stocks.

---

### 🎯 The Simple Rule for Your Account Right Now

Given your account size and experience level here's the simple guide:

```
Sell at 30–45 DTE
Target monthly expirations
Close at 50% profit OR 21 DTE
If trade goes against you → roll out to next month
Never go under 7 DTE without experience
```

---

### 🏆 Putting It All Together

The perfect CSP formula for a beginner:

> **Liquid stock + High IV + OTM strike (delta 0.20–0.30) + 30–45 DTE + close at 50% profit**

That five-part checklist is essentially the entire foundation of professional options income selling. Every other strategy you learn from here is just a variation of those five things.