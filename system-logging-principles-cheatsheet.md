# 📝 System Logging Principles – Cheat Sheet

Use this guide to decide **what, why, and when** to log in your applications.  
(System logs = focus on system operation and health, not user accountability.)

---

## ✅ What to log
- **Startup & shutdown events**  
  *Example*: Application started on port 8080  
- **Configuration details** (but never secrets)  
  *Example*: Connected to database `appdb`  
- **Normal milestones**  
  *Example*: Request processed in 120 ms  
- **Errors & exceptions**  
  Always include stack traces for debugging  
- **External service calls**  
  *Example*: Payment API call took 350 ms  
- **Periodic health info**  
  *Example*: Heartbeat: service alive, 120 requests handled  

---

## 🎯 Why log
- **Debugging** – understand why something broke  
- **Monitoring** – detect slowdowns, spikes, unusual patterns  
- **Operations** – track uptime, restarts, dependencies  
- **Compliance/Security** – evidence of system state (but not detailed user actions)  

---

## ⏰ When to log
- **At entry/exit points**  
  Request received → response sent  
- **When decisions are made**  
  *Example*: Fallback to cache because API timeout  
- **On error paths**  
  Log all exceptions with context  
- **Periodically**  
  *Example*: Memory usage, active connections  

---

## 📊 Log levels
- `TRACE` – very detailed (flow of code, avoid in production)  
- `DEBUG` – internal state, calculations, diagnostics  
- `INFO` – key milestones (startup, shutdown, config, major actions)  
- `WARN` – something unusual but app continues (timeouts, retries)  
- `ERROR` – something failed and may need attention  

👉 Test yourself:  
- Can an operator understand the system with only **INFO logs**?  
- Can a developer debug with **DEBUG logs**?  

---

## 🚫 What NOT to log
- ❌ Passwords, tokens, API keys  
- ❌ Personal data (CPR, names, addresses) – unless anonymized/masked  
- ❌ Excessive repetitive messages (log noise)  

---

## 💡 Best practices
- Use structured messages:  
  ```java
  log.info("Order {} completed in {} ms", orderId, elapsedTime);
  ```
- Keep wording consistent: *Started…*, *Completed…*, *Failed…*  
- Use correlation IDs if possible (trace a request across services)  
- Rotate & retain logs (e.g., 7–30 days for system logs)  

---

✅ **Rule of thumb:**  
System logs should tell the **story of your application’s life** – what it did, when, and how well.  
