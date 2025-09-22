# Exercise: Choosing Log Levels (INFO vs DEBUG vs ERROR)

Decide log level for each log line:  **INFO** (for operators), **DEBUG** (for developers).  
Explain your reasoning.

---

## Events

1. Application started on port 8080  
2. Fetching order `o123` from the database  
3. Order `o123` processed successfully with status SHIPPED  
4. Calculated tax for order `o123`: subtotal=100.0, taxRate=0.25, tax=25.0  
5. Database connection failed: timeout after 5 seconds  
6. Exception stack trace when processing order `o456`  
7. External API call to PaymentService took 320 ms  
8. Cache cleared automatically after exceeding 10,000 entries  

---



✅ **Rule of thumb:**  
- **INFO** = story for operators (what happened).  
- **DEBUG** = extra detail for developers (how it happened).  




