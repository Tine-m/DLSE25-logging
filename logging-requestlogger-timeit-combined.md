# ⏱️ Combining Request Logging (INFO) and Operation Timing (DEBUG)

This example shows how to log at **two levels of detail**:

- **INFO** → For operators (request summary: method, path, status, elapsed time).  
- **DEBUG** → For developers (internal timings, like how long a DB call took).  

---

## TimeIt Helper 

The `TimeIt` helper times *any operation* and logs how long it took.

```java
import org.slf4j.Logger;

import java.util.function.Supplier;

public class TimeIt {
    public static <T> T info(Logger log, String label, Supplier<T> block) {
        long start = System.currentTimeMillis();
        try {
            return block.get(); // run the supplied code
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            log.info("{} completed in {} ms", label, elapsed);
        }
    }

    public static <T> T debug(Logger log, String label, Supplier<T> block) {
        long start = System.currentTimeMillis();
        try {
            return block.get();
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            if (log.isDebugEnabled()) {
                log.debug("{} completed in {} ms", label, elapsed);
            }
        }
    }
}
```

### What is `Supplier<T>`?
- `Supplier<T>` is a functional interface from `java.util.function`.
- It represents a function that takes **no input** but **returns a value** of type `T`.
- Here it lets us run a block of code (like a DB query), measure time, and still return the result.

Example:  
```java
int result = TimeIt.debug(log, "Division", () -> divide(10, 2));
```

---

## RequestLogger Helper

Logs **one summary line** per request.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class RequestLogger {
    private static final Logger log = LoggerFactory.getLogger(RequestLogger.class);

    public static void log(String method, String path, int status, Runnable action) {
        long start = System.currentTimeMillis();
        try {
            action.run();
        } finally {
            long elapsed = System.currentTimeMillis() - start;
            log.info("Request handled method={} path={} status={} elapsed_ms={}",
                     method, path, status, elapsed);
        }
    }
}
```

---

## Combined Example

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Main {
    private static final Logger log = LoggerFactory.getLogger(Main.class);

    public static void main(String[] args) {
        // Wrap the whole request in RequestLogger (INFO for operators)
        RequestLogger.log("POST", "/reservations", 201, () -> {
            // Inside, measure DB call with TimeIt (DEBUG for developers)
            String bike = TimeIt.debug(log, "DB query: find bike", () -> {
                try {
                    Thread.sleep(80); // simulate DB latency
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                return "Bike-42";
            });

            System.out.println("Reserved " + bike);
        });
    }
}
```

---

## Example Log Output

**INFO for operators (always on):**
```
INFO  RequestLogger - Request handled method=POST path=/reservations status=201 elapsed_ms=123
```

**DEBUG for developers (only if enabled):**
```
DEBUG Main - DB query: find bike completed in 81 ms
```

---

## ✅ Key Points
- **RequestLogger** → high-level summary (method, path, status, elapsed). Good for operators.  
- **TimeIt** → detailed timings (DB calls, external API). Good for developers, hidden behind DEBUG.  
- Together, they provide both **clean system logs** and **deep debug info** when needed.  

---
