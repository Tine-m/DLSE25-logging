# 📝 System vs Developer Logging – Example

This example shows how logging serves **two different audiences**:  
- **Operators** – need high-level INFO/WARN/ERROR messages to monitor the system.  
- **Developers** – need DEBUG messages with technical detail for troubleshooting.  

---

## 💻 Example Code: Order Processing Service

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class OrderService {
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);

    public void processOrder(String orderId) {
        // Operator log (high-level milestone)
        log.info("Processing order {}", orderId);

        try {
            // Developer log (internal details)
            log.debug("Fetching order {} from database", orderId);

            // Pretend to load and compute
            String status = "SHIPPED";
            log.debug("Order {} status after business rules = {}", orderId, status);

            // Operator log (milestone)
            log.info("Order {} processed successfully with status {}", orderId, status);

        } catch (Exception e) {
            // Operator log (visible in prod)
            log.error("Order {} processing FAILED: {}", orderId, e.getMessage());

            // Developer log (extra context only when DEBUG enabled)
            log.debug("Exception stack trace for order {}: ", orderId, e);
        }
    }
}
```

---

## 📊 Logs seen by Operators (INFO/WARN/ERROR)

*(Default in production – high-level storyline)*

```
12:00:01 INFO  OrderService - Processing order o123
12:00:01 INFO  OrderService - Order o123 processed successfully with status SHIPPED
```

If something fails:

```
12:00:05 ERROR OrderService - Order o123 processing FAILED: Database timeout
```

---

## 🛠️ Logs seen by Developers (DEBUG enabled)

*(When troubleshooting – adds technical detail)*

```
12:00:01 INFO  OrderService - Processing order o123
12:00:01 DEBUG OrderService - Fetching order o123 from database
12:00:01 DEBUG OrderService - Order o123 status after business rules = SHIPPED
12:00:01 INFO  OrderService - Order o123 processed successfully with status SHIPPED
```

If something fails:

```
12:00:05 ERROR OrderService - Order o123 processing FAILED: Database timeout
12:00:05 DEBUG OrderService - Exception stack trace for order o123:
java.sql.SQLException: timeout ...
   at com.example.OrderRepository.load(OrderRepository.java:42)
   ...
```

---

## ✅ Key Takeaways

- **Operators** care about system health and business milestones.  
  → Use `INFO`, `WARN`, `ERROR`.  

- **Developers** care about technical internals and debugging.  
  → Use `DEBUG` (and occasionally `TRACE`).  

- Good logging separates **“what happened”** from **“how it happened”**.  

