# 📝 Audit Logging Principles – Cheat Sheet

Audit logs are **not the same as system logs**.  
They focus on **who did what, when, and where**.  
Often required for **security, compliance, and investigations**.

---

## ✅ What to log
- **Authentication events**  
  *Example*: Login success/failure, logout, password reset  
- **Authorization decisions**  
  *Example*: User `u123` denied access to resource `doc42`  
- **Data access**  
  *Example*: User `u123` viewed patient record `p567`  
- **Data modifications**  
  *Example*: User `u123` updated order `o789` (fields changed: status)  
- **Administrative actions**  
  *Example*: Role assignment, account creation, account lock/unlock  

---

## 🎯 Why log
- **Accountability** – trace actions back to specific users  
- **Security** – detect suspicious or unauthorized activity  
- **Compliance** – meet legal/industry requirements (GDPR, HIPAA, SOX)  
- **Forensics** – reconstruct events after incidents (fraud, breach, misuse)  

---

## ⏰ When to log
- **Every authentication attempt** (success and failure)  
- **Every authorization check** (granted or denied)  
- **Every sensitive read/write/delete** of protected data  
- **Every change to user roles/permissions**  
- **Every admin or system configuration change**  

---

## 📊 Audit log characteristics
- **Separate from system logs** (dedicated channel/file)  
- **Structured format (e.g., JSON)** for easy search and correlation  
- **Tamper-evident** (signing, append-only, WORM storage)  
- **Longer retention** (months or years, depending on law)  
- **Restricted access** (only security/compliance teams should see full logs)  

---

## 🚫 What NOT to log
- ❌ Passwords or raw authentication tokens  
- ❌ Full personal data (names, CPR numbers, addresses) – log IDs or hashes instead  
- ❌ Application internals (keep those in system logs)  
- ❌ Excessive debug details (avoid clutter; focus on *who, what, when, where*)  

---

## 💡 Best practices
- Include **who** (user ID, not personal name)  
- Include **what** (action performed)  
- Include **when** (timestamp)  
- Include **where** (IP, system, resource ID)  
- Keep **system and audit logs separate**  
- Regularly **review and alert** on suspicious patterns (e.g., repeated login failures)  

---

✅ **Rule of thumb:**  
Audit logs are the **evidence trail** of user and admin activity.  
They must be **complete, tamper-resistant, and privacy-aware**.  

