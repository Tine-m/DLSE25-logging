# Audit Logging in a Simple Java Application

This guide extends the **system logging** example to include **audit logging**.  
Audit logs are different from system logs: they record **who did what, when, and where**.  

---

## 1. Dependencies

The library ```logstash-logback-encoder``` provides a JSON encoder for Logback, which makes audit logs machine-readable (easy to ingest into search and analysis tools like Grafana Loki and Elasticsearch).

```xml
<dependencies>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.13</version>
  </dependency>

  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.8</version>
  </dependency>

  <dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>8.0</version>
  </dependency>
</dependencies>
```

So instead of plain text:

```pgsql
12:34:56 INFO  AUDIT - USER_ACTION: type=LOGIN_SUCCESS ip=203.0.113.7
```

You get structured JSON:
 ```json
{
  "@timestamp": "2025-09-17T12:34:56.789Z",
  "level": "INFO",
  "log_type": "audit",
  "message": "USER_ACTION: type=LOGIN_SUCCESS ip=203.0.113.7",
  "user_id": "u123"
}
```

System logs → plain console/file output = ```slf4j-api + logback-classic``` only.

Audit logs → structured JSON for compliance/aggregation = add ```logstash-logback-encoder```.

---

## 2. Logback configuration for audit log

Create a **separate appender** and logger just for audit events.  
This ensures audit logs are isolated and can have different retention rules (how long data is preserved).

`src/main/resources/logback.xml`:

```xml
<configuration>
  <!-- Regular system logs -->
  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="INFO">
    <appender-ref ref="CONSOLE"/>
  </root>

  <!-- Audit logs to file (JSON format) -->
  <appender name="AUDIT_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logs/audit.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>logs/audit.%d{yyyy-MM-dd}.log.gz</fileNamePattern>
      <maxHistory>180</maxHistory>
    </rollingPolicy>
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
      <customFields>{"log_type":"audit","service":"simple-java-app"}</customFields>
    </encoder>
  </appender>

  <logger name="AUDIT" level="INFO" additivity="false">
    <appender-ref ref="AUDIT_FILE"/>
  </logger>
</configuration>
```

---

## 3. Example code with audit logging

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import java.util.UUID;

public class App {
    private static final Logger log = LoggerFactory.getLogger(App.class);
    private static final Logger audit = LoggerFactory.getLogger("AUDIT");

    public static void main(String[] args) {
        // Add context (user, correlation ID)
        MDC.put("correlation_id", UUID.randomUUID().toString());
        MDC.put("user_id", "u123"); // avoid PII, use internal IDs

        log.info("Application starting");

        // Example of an auditable event
        audit.info("USER_ACTION: type=LOGIN_SUCCESS ip=203.0.113.7");

        try {
            int result = divide(10, 2);
            log.debug("Calculation result = {}", result);
        } catch (Exception e) {
            log.error("Unexpected error", e);
        }

        log.info("Application finished");

        // Clear context
        MDC.clear();
    }

    private static int divide(int a, int b) {
        log.trace("divide({}, {}) called", a, b);
        return a / b;
    }
}
```

---

## 4. Two loggers, two purposes

- **System log**:  
  `Logger log = LoggerFactory.getLogger(App.class);`  
  - Used for *operational* messages: app start, errors, performance info.  
  - Printed to the console.  

- **Audit log**:  
  `Logger audit = LoggerFactory.getLogger("AUDIT");`  
  - Dedicated channel for *user activity* events: login, data changes, access attempts.  
  - Written to a separate file (`logs/audit.log`), in structured JSON.    

---

## 5. MDC (Mapped Diagnostic Context)

The example uses:

```java
MDC.put("correlation_id", UUID.randomUUID().toString());
MDC.put("user_id", "u123");
```

- MDC allows you to add **context values** (like request ID, user ID, tenant ID) to *every log line automatically*.  
- For audit logs, this means you don’t have to remember to include the user ID in every message—it’s appended automatically by the encoder.  
- In real systems, the `correlation_id` often comes from an HTTP header (`X-Request-ID`), so you can trace a single user request across multiple services.  

---

## 6. Audit event example

```java
audit.info("USER_ACTION: type=LOGIN_SUCCESS ip=203.0.113.7");
```

This represents:  
- The user with ID `u123` (from MDC)  
- Successfully logged in  
- From IP address `203.0.113.7`  
- At a given timestamp  

---

## 7. Understanding JSON Fields in Logstash Logback Encoder

### Example JSON log
```json
{
  "@timestamp": "2025-09-22T19:16:27.9462391+02:00",
  "@version": "1",
  "message": "USER_ACTION: type=LOGIN_SUCCESS ip=203.0.113.7",
  "logger_name": "AUDIT",
  "thread_name": "main",
  "level": "INFO",
  "level_value": 20000,
  "log_type": "audit",
  "service": "simple-java-app"
}
```

## Field breakdown

- **@timestamp** → Generated automatically from the log event’s creation time.  
- **@version** → Always `"1"`, a legacy convention from Logstash JSON format.  
- **message** → Whatever you passed into the logger:  
  ```java
  audit.info("USER_ACTION: type=LOGIN_SUCCESS ip=203.0.113.7");
  ```
- **logger_name** → Comes from the logger instance:  
  ```java
  Logger audit = LoggerFactory.getLogger("AUDIT");
  ```
- **thread_name** → `"main"` in a simple program, but in web apps could be like `http-nio-8080-exec-3`.  
- **level** → The severity in text form (`INFO`, `DEBUG`, etc.).  
- **level_value** → The severity in numeric form (Logback’s internal constants).  

## Logback Level → Numeric Mapping
Logback defines severity levels with integer values:

| Level  | Numeric value |
|--------|---------------|
| TRACE  | 5000          |
| DEBUG  | 10000         |
| INFO   | 20000         |
| WARN   | 30000         |
| ERROR  | 40000         |

So in our example:  
```
level = "INFO"
level_value = 20000
```

- This comes directly from Logback’s `Level` class constants.

## Custom fields
Fields like **log_type** and **service** are not defaults. They are added via our `logback.xml`:

```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
  <customFields>{"log_type":"audit","service":"simple-java-app"}</customFields>
</encoder>
```

- **log_type** → lets you distinguish audit vs system logs.  
- **service** → identifies which application produced the log.


## Summary
- The encoder automatically captures **standard metadata**: timestamp, level, thread, logger.  
- The **application** supplies the **message** (and optional MDC context).  
- The **configuration** can add **custom fields**.  
- **level_value** is just Logback’s internal numeric constant for the severity level.

---

## 8. Why this design is useful

- **System operators** can see what’s happening via normal logs.  
- **Security/compliance teams** get a dedicated audit log they can retain longer and lock down.  
- **Developers** don’t have to change code everywhere—just log to `audit` when an auditable event occurs.  
