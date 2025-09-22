# System Logging in a Simple Java Application

This guide shows how to add **system logs** (startup, info, debug,
error) to a simple Java app using **SLF4J** and **Logback**.

------------------------------------------------------------------------

## 1. Add dependencies (Maven)

``` xml
<!-- pom.xml -->
<dependencies>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.13</version>
  </dependency>

  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.5.18</version>
  </dependency>
</dependencies>
```

------------------------------------------------------------------------

## 2. Logback configuration (optional)

By default, Logback prints to the console.\
You can control log levels and format via
`src/main/resources/logback.xml`:

``` xml
<configuration>
  <root level="INFO">
    <appender-ref ref="CONSOLE"/>
  </root>

  <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>
</configuration>
```

This produces output like:

    12:34:56 INFO  App - Application started

------------------------------------------------------------------------

## 3. Example code

``` java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class App {
    private static final Logger log = LoggerFactory.getLogger(App.class);

    public static void main(String[] args) {
        log.info("Application starting");

        try {
            int result = divide(10, 2);
            log.debug("Calculation result = {}", result);
        } catch (Exception e) {
            log.error("Unexpected error", e);
        }

        log.info("Application finished");
    }

    private static int divide(int a, int b) {
        log.trace("divide({}, {}) called", a, b);
        return a / b;
    }
}
```

------------------------------------------------------------------------

## 4. How it works

-   **Logger** is created per class:

    ``` java
    private static final Logger log = LoggerFactory.getLogger(App.class);
    ```

-   **Log levels**:

    -   `TRACE`: very detailed, usually off in production.
    -   `DEBUG`: development diagnostics.
    -   `INFO`: key application events.
    -   `WARN`: unusual, but app keeps running.
    -   `ERROR`: something failed, attention needed.

-   **Placeholders** (`{}`) safely insert variables:

    ``` java
    log.debug("Calculation result = {}", result);
    ```

------------------------------------------------------------------------
## 5. Run it

When you run with Maven or inside Docker, logs appear in your console.\
This gives you a **system log** of what the app is doing at runtime.

