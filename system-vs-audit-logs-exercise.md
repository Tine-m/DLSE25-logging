# Exercise: System Logs vs Audit Logs

Decide whether each event should be logged as a **System Log** 🖥️ or an **Audit Log** 🔒.  
Explain **why** in each case.

---

## Events

1. Application started on port 8080  
2. User `u123` successfully logged in from IP `203.0.113.7`  
3. Database connection failed: timeout after 5 seconds  
4. User `u123` attempted to access patient record `p567` without permission  
5. Payment API call took 450 ms  
6. User `admin` assigned role `Editor` to user `u456`  
7. Cache cleared automatically after exceeding 10,000 entries  
8. User `u789` updated order `o321` status from *Pending* to *Shipped*  
9. Application shutting down due to maintenance  
10. Repeated login failures for user `u555` (5 attempts in 1 minute)

---

✅ **Rule of thumb for solving:**  
- If it’s about **system health/operation** → System Log 🖥️  
- If it’s about **user actions or security** → Audit Log 🔒  


