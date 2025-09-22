# The Art of Logging: Granularity and Clutter

Logging is not only about the mechanics (using Logback, appenders, levels).  
It’s also about **deciding *what* to log and at what level**. This is where granularity comes in.

---

## What is Log Granularity?
**Granularity** = the *amount of detail* you put into logs.

- **Fine-grained (detailed) logging**  
  Example:  
  ```java
  log.debug("Fetching order o123 from database");
  log.debug("SQL executed: SELECT * FROM orders WHERE id=o123");
  ```
  ➡️ Great for developers troubleshooting, but noisy for operators.

- **Coarse-grained (high-level) logging**  
  Example:  
  ```java
  log.info("Order o123 processed successfully with status SHIPPED");
  ```
  ➡️ Useful milestone events that operators care about.

---

## 🧹 The Clutter Problem
If you log *everything*:
- Code looks messy (lots of log statements).
- Log files become huge and hard to search.
- Important messages are buried in noise.

If you log *too little*:
- You can’t troubleshoot effectively.
- Audits and monitoring have gaps.

---

## 🎯 Finding the Balance
- Use **INFO** for operators: startup/shutdown, major business events, warnings, errors.
- Use **DEBUG/TRACE** for developers: detailed flow, parameters, calculations.
- Hide low-level details in helpers, aspects, or conditional blocks.
- Consider: *“Will someone need this info to run the system, or only to debug code?”*

---

## ✅ Rule of Thumb
- **INFO/WARN/ERROR** → For system operators (keep it concise).  
- **DEBUG/TRACE** → For developers (enable only when needed).  
- Always ask: *Does this log line add value, or just clutter?*

---

## 💡 Example
```java
// Operator-focused log (INFO)
log.info("Order {} processed successfully with status {}", orderId, status);

// Developer-focused log (DEBUG)
log.debug("Calculated tax for order {}: subtotal={}, taxRate={}, tax={}", 
          orderId, subtotal, taxRate, tax);
```

Operators see milestones.  
Developers, when enabling DEBUG, see internals.

---
