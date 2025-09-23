# Log Rotation Explained (with Logback)

Log rotation means your application’s logs don’t grow forever in a single file.  
Instead, Logback automatically creates new log files based on **time** or **size** rules.

---

## 🕒 Daily Rotation (TimeBasedRollingPolicy)

With daily rotation, you always have one **active log file** (`logs/app.log`).  
At **midnight**, Logback:

1. Renames the current `app.log` to include the date (e.g. `app.2025-09-23.log`).
2. Creates a new empty `app.log` for the new day’s logs.
3. Deletes old logs if they exceed `<maxHistory>`.

### Example Timeline
- **Sept 23** → logs written to `app.log`  
- At midnight → `app.log` → renamed to `app.2025-09-23.log`  
- **Sept 24** → new empty `app.log` created and used  

### File structure after a few days
```
logs/
 ├── app.log              (today’s logs)
 ├── app.2025-09-23.log   (yesterday)
 ├── app.2025-09-22.log
 ├── app.2025-09-21.log
 ...
```

### Config snippet
```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logs/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>14</maxHistory> <!-- keep 14 days -->
    </rollingPolicy>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
    </encoder>
</appender>
```

---

## 📏 Size-Based Rotation

Instead of time, you can rotate logs when they reach a certain size (e.g. 10 MB).

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
    <maxFileSize>10MB</maxFileSize>
</rollingPolicy>
```

This would produce files like:

```
logs/
 ├── app.log
 ├── app.0.log
 ├── app.1.log
 ├── app.2.log
```

---

## 🔀 Hybrid Rotation (Time + Size)

Often you want *both*: daily rollover **and** rotation if the file grows too large.

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <fileNamePattern>logs/app.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
    <timeBasedFileNamingAndTriggeringPolicy
        class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <maxFileSize>10MB</maxFileSize>
    </timeBasedFileNamingAndTriggeringPolicy>
    <maxHistory>14</maxHistory>
</rollingPolicy>
```

This produces:
```
logs/
 ├── app.2025-09-23.0.log
 ├── app.2025-09-23.1.log
 ├── app.2025-09-23.2.log
 ├── app.2025-09-22.0.log
 ...
```

---

## ✅ Summary
- **Daily rotation** → new file every day, old ones kept X days.  
- **Size-based rotation** → new file when log grows too big.  
- **Hybrid** → combine both for safety.  
- Prevents logs from eating disk space and keeps logs organized.

---
