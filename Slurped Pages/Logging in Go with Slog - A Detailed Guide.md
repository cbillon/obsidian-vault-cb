---
link: https://last9.io/blog/logging-in-go-with-slog-a-detailed-guide/
byline: Preeti Dewani
site: Last9
date: 2025-02-03T12:30
excerpt: Learn how to simplify logging in Go with Slog. This guide covers
  customization, handlers, log levels, and more for effective logging.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T18:57
title: "Logging in Go with Slog: A Detailed Guide"
---

Golang is widely recognized for its efficiency and versatility in building scalable applications. However, one area that often gets overlooked in system development is logging.

Effective logging is key to tracing errors, monitoring systems, and boosting overall application reliability. That's where _golang slog_ comes in—a structured logging solution that's making a real impact for Go developers.

In this guide, we’ll break down everything you need to know about _golang slog_, from the basics to advanced techniques.

## **What is Golang Slog?**

In simple terms, **golang slog** is a structured logging package designed to provide an organized and easy-to-read format for logs in Go.

Unlike traditional loggers that output unstructured text, structured logging formats the log entries in a way that makes it easier to extract useful information for monitoring, debugging, and troubleshooting.

**Why Choose Golang Slog?**

1. **Readability**: With slog, your logs will have a consistent structure, which is crucial for readability.
2. **Performance**: It’s designed to handle high throughput without compromising on performance.
3. **Flexibility**: You can customize the logging format to suit your needs.

### Key Features of Golang Slog

1. **Log Levels and Severity**  
    With _golang slog_, you can set log levels like Info, Warn, Error, and Debug, giving you more control over which messages get captured.
2. **Custom Handlers**  
    You can create custom handlers to define how logs are formatted and where they are sent (console, file, or external services). For example:

```
logger := slog.New(slog.NewJSONHandler(os.Stdout))
```

This output logs in JSON format, perfect for log aggregation tools like Elasticsearch.

3. **Contextual Logging**  
    Easily add context to log entries for better correlation and troubleshooting. For instance:

```
logger.With("user_id", 12345).Info("User login")
```

This adds the user ID to the log for more context.

4. **Error Handling and Stack Traces**  
    Automatically include stack traces in error logs to help debug issues quickly. Example:

```
err := someFunction()
if err != nil {
    logger.Error("Error occurred", slog.Err(err))
}
```

### How Does Slog Work?

_golang slog_ is all about creating structured log entries that are easier to work with. Unlike traditional plain-text logs, which require manual parsing, _slog_ allows you to define log entries using key-value pairs, such as timestamps, log levels (info, warn, error), and additional context. This structure makes logs machine-readable and simplifies real-time analysis.

### Basic Example of a Slog

Here’s a simple example to show how to use _slog_ in your Go application:

```
package main

import (
    "log"
    "golang.org/x/exp/slog"
    "os"
)

func main() {
    logger := slog.New(slog.NewTextHandler(os.Stdout))
    logger.Info("Application started", slog.String("version", "1.0.0"))
}
```

In this example, a structured log entry is created when the application starts. It includes the log level (`Info`), the message, and extra metadata (in this case, the version number).

## How to Get Started with Slog in Go

If you're looking for a flexible logging solution for your Go app, _golang slog_ is a great option. This lightweight library allows for easy logging in various formats, making it simple to handle and display logs.

#### 1. Installing Slog

To use _golang slog_ in your project, you need to install the library. It’s available via the Go module system, so simply run this command:

```
go get golang.org/x/exp/slog
```

This command will fetch the latest version of the _golang slog_ package and install it in your Go project.

#### 2. Basic Configuration

Once installed, you can start using _golang slog_ immediately. Let’s begin by logging some basic messages to the console.

**Step 1: Create a New Logger**

In Go, _golang slog_ provides the `slog.New()` function to create a new logger instance. The logger uses a handler to determine how log records are processed and displayed.

For example, to log into the console in a human-readable format, you can use them `TextHandler`, which outputs logs as plain text:

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
)

func main() {
    // Create a new logger with a TextHandler that writes to standard output
    logger := slog.New(slog.NewTextHandler(os.Stdout))

    // Log some messages
    logger.Info("This is an info message")
    logger.Warn("This is a warning")
    logger.Error("An error occurred")
}
```

In this example, a new logger is created with a `TextHandler` that outputs the log messages as plain text to the terminal (standard output). You can easily swap `os.Stdout` with other destinations, like a file or a network service.

**Step 2: Structured Logs with JSON**

If you prefer storing your logs in a structured format, like JSON (perfect for monitoring and centralized logging systems), you can use the `JSONHandler` instead of the `TextHandler`.

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
)

func main() {
    // Create a new logger with a JSONHandler
    logger := slog.New(slog.NewJSONHandler(os.Stdout))

    // Log some messages
    logger.Info("This is a structured info message")
    logger.Warn("This is a structured warning")
    logger.Error("Structured error message")
}
```

With the `JSONHandler`, the log records are outputted in JSON format, making them easy to process by log management tools like the ELK stack (Elasticsearch, Logstash, and Kibana), Splunk, or other systems.

#### 3. Log Levels

_Golang slog_ supports multiple [log levels](https://last9.io/blog/log-levels-explained/) to help categorize and filter messages based on severity. The common log levels are:

- `Debug`
- `Info`
- `Warn`
- `Error`
- `Fatal`

Here’s how to use these levels:

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
)

func main() {
    logger := slog.New(slog.NewTextHandler(os.Stdout))

    // Using various log levels
    logger.Debug("This is a debug message")
    logger.Info("This is an info message")
    logger.Warn("This is a warning message")
    logger.Error("This is an error message")
    logger.Fatal("This is a fatal error message")
}
```

Each log level corresponds to a different severity of the event being logged. By default, _golang slog_ logs all messages. However, you can adjust the log level to filter out less severe messages as needed.

#### 4. Log Record Structure

A log record in _golang slog_ is more than just a text string. It contains valuable information that helps with debugging or analyzing application behavior. Each log record typically includes:

- **Time**: Timestamp of the log entry
- **Level**: The log level (e.g., Info, Error)
- **Message**: The actual log message
- **Context**: Additional key-value pairs (e.g., user IDs, request IDs) that provide more context for the log message

Here’s an example of a log record with added context:

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
)

func main() {
    logger := slog.New(slog.NewJSONHandler(os.Stdout))

    // Add context to logs
    logger.With("user_id", 12345, "request_id", "abcde").Info("User request processed")
}
```

In this example, the log record is enriched with additional information like `user_id` and `request_id`, which can help you trace the flow of requests through your system.

## What is Contextual Logging with Golang Slog?

Contextual logging is a technique that enhances logs with additional details about the application’s state, user actions, or system events. It allows logs to tell a more complete story, which is especially useful for debugging and monitoring.

Golang's `slog` package makes it easy to add contextual data to your logs, helping you better understand what's happening in your application at any given time.

Here’s how you can use contextual logging with `golang slog`:

### 1. Adding Contextual Data to Logs

Contextual data is extra information that makes your logs more meaningful. In `golang slog`, you can easily add context using the `With` method, which attaches additional fields to your log entries.

```
package main

import (
	"golang.org/x/exp/slog"
	"os"
)

func main() {
	// Initialize logger with standard output handler
	logger := slog.New(slog.NewJSONHandler(os.Stdout))

	// Adding context to logs
	logger.With("user_id", 1234, "transaction_id", "abc123").Info("User performed an action")
}
```

This example logs the `user_id` and `transaction_id` along with the log message, helping you trace activities back to specific users or transactions.

### 2. Using Child Loggers for Structured Context

Child loggers inherit the context of their parent logger, but they can also add their own context. This is useful for logging different subsystems of an application independently.

```
package main

import (
	"golang.org/x/exp/slog"
	"os"
)

func main() {
	// Create a base logger
	logger := slog.New(slog.NewJSONHandler(os.Stdout))

	// Create a child logger with added context
	childLogger := logger.With("module", "payment")
	childLogger.Info("Payment gateway initiated", slog.String("gateway", "Stripe"))

	// Another child logger with different context
	childLogger.With("transaction_id", "xyz789").Info("Payment processed successfully")
}
```

In this case, logs for the payment module are clearly separated from the rest of the application, making it easier to debug specific subsystems.

### 3. Grouping Logs with Context

By adding common contextual fields like `request_id` or `session_id`, you can group logs that belong to the same process or transaction. This is particularly valuable in distributed systems.

```
package main

import (
	"golang.org/x/exp/slog"
	"os"
)

func main() {
	// Create a logger
	logger := slog.New(slog.NewJSONHandler(os.Stdout))

	// Use the request_id context to group logs
	logger.With("request_id", "req123").Info("Started processing request")
	logger.With("request_id", "req123").Info("Request completed successfully")
}
```

Here, both log entries share the same `request_id`, making it easier to trace the flow of the request through the system.

### 4. Dynamic Context with the `With` Method

Sometimes, the context you need to log in can change dynamically during runtime. `golang slog` allows you to add dynamic context at any point in the application.

```
package main

import (
	"golang.org/x/exp/slog"
	"os"
)

func main() {
	logger := slog.New(slog.NewJSONHandler(os.Stdout))

	// Simulate dynamic context for different user sessions
	sessionID := "session_123"
	logger.With("session_id", sessionID).Info("User logged in")

	sessionID = "session_456"
	logger.With("session_id", sessionID).Info("User logged in")
}
```

This allows tracking multiple sessions, even if they happen at the same time, providing unique insights into user activity.

### 5. Scoped Logging for Specific Operations

Child loggers are perfect for scoping logs to specific operations or functions, making it easy to isolate log entries related to a particular task.

```
package main

import (
	"golang.org/x/exp/slog"
	"os"
)

func processPayment(logger *slog.Logger, userID int) {
	// Use a child logger to scope logs for payment processing
	childLogger := logger.With("user_id", userID, "operation", "payment")
	childLogger.Info("Payment initiated")
	// Additional payment logic...
	childLogger.Info("Payment completed")
}

func main() {
	logger := slog.New(slog.NewJSONHandler(os.Stdout))
	// Calling the payment function
	processPayment(logger, 1234)
}
```

This helps provide context about which user is involved in a specific operation, making it easier to understand the flow of events.

### 6. Cleaning Up Contextual Data

It's important to clean up or reset context after it’s no longer needed to prevent irrelevant data from affecting subsequent logs.

```
childLogger := logger.With("transaction_id", "tx123")
childLogger.Info("Transaction started")
childLogger.With("user_id", 456).Info("User info included")

// Ensure no unintended context leaks into future logs
logger.Info("Next unrelated log message")
```

Context should be reset or cleared after it's used, especially when switching to unrelated operations. This prevents the accidental carryover of irrelevant context, which can clutter the logs and make debugging harder.

## 3 Advanced Golang Slog Techniques

### 1. Integrating with External Systems

_Golang slog_ makes it easy to integrate with external log aggregation and monitoring systems. You can use the `JSONHandler` or a custom handler like Last9 to send logs to a centralized collection system.

Here's an example of sending logs to Last9:

```
logger := slog.New(slog.NewJSONHandler(last9.NewLast9Handler("your-last9-token")))
logger.Info("Application started")
```

### 2. Performance Tuning for High-Traffic Applications

For high-traffic systems, avoid logging bottlenecks by using asynchronous logging or tweaking the log level to reduce unnecessary log entries. While _golang slog_ is already optimized, these tweaks can further boost logging performance.

### 3. Custom Log Formats for Specific Needs

You might need custom logging formats depending on your application. For example, an API server may require JSON logs for compatibility with log aggregation tools, while a CLI app might need plain text. You can create custom handlers to adjust the log format as needed:

```
type MyCustomHandler struct{}
func (h *MyCustomHandler) HandleLog(r slog.Record) error {
    // Custom log handling logic
    return nil
}
```

## How to Customize Loggers and Handlers in Golang Slog

In Golang's `slog`, customization is key when you need to tailor your logging system to meet specific requirements. The built-in logger and handlers are flexible enough to allow for extensive customization, such as creating custom log levels, defining custom handlers, and adjusting the log format.

Let’s walk through how to customize loggers and handlers in Golang's `slog` to enhance the functionality and readability of your logs.

### 1. Customizing Log Levels

By default, Golang's `slog` provides several predefined log levels (e.g., Info, Warn, Error). However, you might need more granular control over log levels, especially in applications where specific logging levels are critical for distinguishing between different types of messages.

#### Step 1: Define Custom Log Levels

You can define your own log levels as constants to make your code more readable and consistent. For instance, let’s say you want a `Debug` level and a `Critical` level.

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
)

const (
    DebugLevel   = iota
    InfoLevel
    WarnLevel
    ErrorLevel
    CriticalLevel
)

func main() {
    // Initialize logger
    logger := slog.New(slog.NewJSONHandler(os.Stdout))

    // Custom log levels
    logLevel := DebugLevel
    if logLevel <= DebugLevel {
        logger.Debug("This is a debug message")
    }
    if logLevel <= InfoLevel {
        logger.Info("This is an info message")
    }
    if logLevel <= WarnLevel {
        logger.Warn("This is a warning")
    }
    if logLevel <= ErrorLevel {
        logger.Error("This is an error")
    }
    if logLevel <= CriticalLevel {
        logger.With("level", "critical").Error("This is a critical issue")
    }
}
```

In this example, `logLevel` determines which logs are written. You can customize your log levels and adjust the flow to output only the relevant logs depending on the environment (e.g., development, staging, production).

#### Step 2: Customize Log Level Behavior

You can enhance this setup by incorporating log-level filtering into your logger configuration, so logs at levels below a certain threshold are not written.

### 2. Creating Custom Handlers

Handlers in Golang's `slog` define how log records are processed. By default, `slog` offers JSON and text-based log handlers, but you might need a custom handler if you're integrating with other services, writing logs to a database, or implementing a unique log format.

#### Step 1: Implementing a Custom Handler

You can create a custom handler by implementing the `slog.Handler` interface. This interface requires you to define a `Handle` method, which processes log records.

Let’s create a simple custom handler that writes log records to a file.

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
    "fmt"
)

type FileHandler struct {
    file *os.File
}

func NewFileHandler(filePath string) (*FileHandler, error) {
    file, err := os.OpenFile(filePath, os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0666)
    if err != nil {
        return nil, err
    }
    return &FileHandler{file: file}, nil
}

func (h *FileHandler) Handle(r slog.Record) error {
    logMessage := fmt.Sprintf("%s %s: %s\n", r.Time, r.Level, r.Message)
    _, err := h.file.WriteString(logMessage)
    return err
}

func (h *FileHandler) Close() error {
    return h.file.Close()
}

func main() {
    // Initialize the custom file handler
    fileHandler, err := NewFileHandler("logs.txt")
    if err != nil {
        panic(err)
    }
    defer fileHandler.Close()

    // Set the custom handler for the logger
    logger := slog.New(fileHandler)

    // Log messages
    logger.Info("This is a log message")
    logger.Error("Something went wrong")
}
```

Here, the `FileHandler` struct implements the `slog.Handler` interface and writes log records to a file (`logs.txt`). You can extend this pattern to create handlers for different outputs like databases, cloud logging services, or even custom formats.

#### Step 2: Customize Log Output Format

You may also want to change the log format to suit your specific needs. For instance, you could format the log output as plain text, XML, or a custom format that fits your system.

```
func (h *FileHandler) Handle(r slog.Record) error {
    // Customize log format to match your needs
    logMessage := fmt.Sprintf("[CUSTOM LOG] %s - [%s] - %s\n", r.Time, r.Level, r.Message)
    _, err := h.file.WriteString(logMessage)
    return err
}
```

This modified handler outputs the log entries in a custom format, which can be extremely helpful when integrating with other systems or when you have a specific logging standard.

### 3. Using Multiple Handlers

You might want to send logs to multiple outputs at once, such as writing logs to both the console and a file. Golang’s `slog` makes this easy by allowing you to combine multiple handlers.

```
package main

import (
    "golang.org/x/exp/slog"
    "os"
)

func main() {
    // Create a console handler
    consoleHandler := slog.NewTextHandler(os.Stdout)

    // Create a file handler
    fileHandler, err := NewFileHandler("logs.txt")
    if err != nil {
        panic(err)
    }
    defer fileHandler.Close()

    // Combine handlers
    multiHandler := slog.NewHandlerGroup(consoleHandler, fileHandler)

    // Create logger with multiple handlers
    logger := slog.New(multiHandler)

    // Log messages to both the console and file
    logger.Info("This message goes to both console and file")
}
```

With the `HandlerGroup`, both the console and file handlers receive the log entries. This is helpful when you want to simultaneously monitor logs in real-time (on the console) while also keeping a persistent record (in a file).

💡

To understand different log severity levels, take a look at our guide on [syslog levels](https://last9.io/blog/what-are-syslog-levels/).

### 4. Customizing Log Levels in Handlers

Sometimes, you might want specific handlers to process only certain log levels. For example, you could create a handler that only logs `Error` and `Critical` levels to a file, while logging all levels to the console.

```
type LevelFilterHandler struct {
    handler slog.Handler
    minLevel int
}

func NewLevelFilterHandler(handler slog.Handler, minLevel int) *LevelFilterHandler {
    return &LevelFilterHandler{handler: handler, minLevel: minLevel}
}

func (f *LevelFilterHandler) Handle(r slog.Record) error {
    if r.Level >= f.minLevel {
        return f.handler.Handle(r)
    }
    return nil
}
```

This handler filters log records based on the level and passes the relevant logs to the underlying handler.

## Common Pitfalls in Golang Slog and How to Avoid Them

### 1. Not Using Structured Logs

It’s tempting to stick with traditional unstructured logs, but without structure, logs can become a pain to parse and analyze. Using structured logs, like key-value pairs, makes it easier to debug and monitor your system.

### 2. Logging Too Much

Logging everything might seem like a good idea, but it can overwhelm your system and make it harder to sift through useful information. Be selective about what you log—only capture what's essential for debugging or monitoring.

### 3. Ignoring Performance

In large-scale applications, logging can introduce performance bottlenecks. Always test your logging implementation to ensure it doesn't slow down your system, especially under high-traffic conditions.

## Best Practices for Logging with Golang Slog

Effective logging is essential to keep your applications running smoothly. By following these best practices when using golang slog, you can ensure that your logs are structured, optimized, and easy to analyze.

### 1. Standardize Your Log Interfaces

Consistency is key when logging. Use a standard format across your application to make logs easier to interpret and search. Structured logs help maintain clarity.

**Example:**

```
package main

import (
  "log"
  "golang.org/x/exp/slog"
)

func main() {
  // Creating a consistent JSON handler for logging
  logger := slog.New(slog.NewJSONHandler(os.Stdout))
  logger.Info("App initialized", slog.String("version", "1.0"))
}
```

**Standard Fields:**  
Always include useful details like `user_id`, `request_id`, or `correlation_id` to easily correlate logs across different services.

### 2. Use Appropriate Log Levels

Using the right log levels helps you filter logs based on severity, making it easier to manage large volumes of data.

- **Info:** General messages like app startup.
- **Debug:** Detailed info useful for troubleshooting.
- **Warn:** Non-critical issues that don’t affect the app’s operation.
- **Error:** Critical issues requiring immediate attention.
- **Fatal:** For cases where the app cannot recover.

**Example:**

```
logger.Warn("Unexpected input", slog.String("input", "NaN"))
logger.Error("Database connection failed", slog.String("db_host", "localhost"))
```

### 3. Contextual Logging

Add context like `user_id` or `session_id` to your logs for better tracking. This helps when tracing issues across distributed systems.

**Example:**

```
logger.With("user_id", 1234).Info("User requested data")
logger.With("session_id", "xyz123").Error("Session timed out")
```

### 4. Use Log Management Tools

As your app grows, simple console logs might not cut it. Tools like Last9, Elasticsearch, or Datadog can help aggregate, search, and visualize logs.

**Integration with External Tools:**

```
logger := slog.New(slog.NewJSONHandler(last9.NewLast9Handler("your-last9-token")))
logger.Info("Application started")
```

### 5. Avoid Over-Logging

Excessive logging can slow down your system and make logs harder to manage. Log only what's necessary to monitor, debug, or audit.

**Log Only What’s Necessary:**  
Don’t log trivial information like method calls unless they're important.

**Rate-Limiting Logs:**  
In high-traffic apps, rate-limit logs to prevent overwhelming your system.

**Example:**

```
if time.Now().Unix()%60 == 0 {
  logger.Info("Regular health check", slog.String("status", "OK"))
}
```

### 6. Use Asynchronous Logging for High-Performance Systems

In high-throughput systems, synchronous logging can become a bottleneck. Use asynchronous logging to prevent performance issues.

**Buffered Logging:**  
Buffer logs and flush them in batches to improve performance.

**Example:**

```
logger := slog.New(slog.NewJSONHandler(bufferedWriter))
```

### 7. Log Retention and Storage Strategy

Define a retention policy to manage the storage of your logs. Rotate old logs to avoid excessive disk usage.

**Log Rotation:**  
Set up automatic log rotation to manage file sizes.

**Archiving:**  
For compliance, archive logs to external storage or the cloud.

### 8. Test Your Logging Implementation

Before going to production, test your logging system in a staging environment. Ensure logs are structured, contain all the necessary information, and are not too verbose.

**Simulate Errors:**  
Check how your system handles errors and if logs contain the right stack traces.

**Monitor Log Performance:**  
Ensure logging doesn’t slow down your application, especially under high traffic.

## Conclusion

Golang’s `slog` is a fantastic tool for building efficient and easy-to-manage logging systems. It’s simple, flexible, and performs well—making it a must-have for any Go developer.

🤝

If you’ve got more questions or want to chat about your use case, our [Discord community is always open](https://discord.com/invite/W8gMppQC4b). We’ve got a channel where you can connect with other developers and dive into the details.