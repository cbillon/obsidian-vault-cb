---
link: https://last9.io/blog/write-logs-to-file/
byline: Anjali Udasi
site: Last9
date: 2025-07-01T10:30
excerpt: Understand how to write logs to a file in Go, avoid common pitfalls,
  and build a production-ready logging setup with performance and safety in
  mind.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:07
title: How to Write Logs to a File in Go
---

When your Go application moves beyond development, you need structured logging that persists. Writing logs to files gives you the control and reliability that stdout can't match, especially when you're debugging production issues or need to meet compliance requirements.

This blog walks through the practical approaches, from Go's standard library to structured logging with popular packages.

## Write Your First Log File in Go

Go's built-in `log` package handles basic file logging with minimal setup. The standard library approach works well for straightforward applications where you need persistent logs without additional dependencies.

The key is redirecting log output from the default stderr to a file handle. Go's `log` package treats any `io.Writer` as a valid destination, making file logging straightforward:

```
package main

import (
    "log"
    "os"
)

func main() {
    logFile, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
    if err != nil {
        log.Fatalf("Failed to open log file: %v", err)
    }
    defer logFile.Close()

    log.SetOutput(logFile)
    log.SetFlags(log.LstdFlags | log.Lshortfile)
    
    log.Println("Application started")
}
```

The `os.O_APPEND` flag ensures new entries don't overwrite existing logs, while `os.O_CREATE` handles file creation automatically. Production setups also need proper file permissions and predictable log locations.

### Manage File Permissions and Paths

Production applications require consistent log organization and secure file access. The standard approach involves creating dedicated log directories with appropriate permissions for your service user:

```
func setupLogging(serviceName string) (*os.File, error) {
    logDir := "/var/log/myapp"
    
    if err := os.MkdirAll(logDir, 0755); err != nil {
        return nil, fmt.Errorf("failed to create log directory: %w", err)
    }
    
    timestamp := time.Now().Format("2006-01-02")
    logPath := filepath.Join(logDir, fmt.Sprintf("%s-%s.log", serviceName, timestamp))
    
    return os.OpenFile(logPath, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644), nil
}
```

Setting directory permissions to `0755` allows the service to create files while keeping them readable by monitoring tools. File permissions of `0644` let your service to write while allowing log processors to read.

**What Happens When File Logging Fails**

Production systems are harsh. Disks fill up, permissions change, and directories get moved. Your logging code needs to handle these failures gracefully instead of crashing your application.

The principle is simple: logging failures shouldn't become application failures. When your primary log destination becomes unavailable, you need a fallback strategy. Most developers implement this as a cascade, try the primary location, fall back to a secondary path, and ultimately write to stderr as the last resort.

```
// Simple fallback pattern
if primaryFile, err := os.OpenFile(primaryPath, flags, 0644); err == nil {
    log.SetOutput(primaryFile)
} else {
    log.SetOutput(os.Stderr) // Always have a fallback
    log.Printf("Failed to open log file, using stderr: %v", err)
}
```

### System Log Integration

Unix systems provide centralized logging through syslog, which can be valuable for system administration and compliance. Go's `log/syslog` package integrates with system logging infrastructure:

```
logWriter, err := syslog.New(syslog.LOG_INFO, "MyApp")
if err != nil {
    log.Fatalf("Unable to connect to syslog: %v", err)
}
defer logWriter.Close()

log.SetOutput(logWriter)
log.Println("Application started via syslog")
```

The syslog approach centralizes logs with other system services, making it easier to monitor multiple applications from a single location.

## Common Pitfalls in File Logging

File logging looks simple until your service hits production. The issues aren’t usually code-level bugs; they’re silent failures caused by mismatched environments and incorrect assumptions.

**1. Permission mismatch:**  
Your local dev setup probably runs with generous file system access. But in production, containers often run as non-root users with restricted volumes. Attempts to write logs silently fail due to missing write permissions—no panics, just missing logs.

**2. Buffered writes ≠ immediate logs:**  
Go’s `os.File` operations are buffered by the OS. Unless you explicitly flush (`file.Sync()`) or close the file, logs might never make it to disk, especially if the process exits unexpectedly. This leads to a common “it worked locally” debugging nightmare.

**3. The stdout mental model:**  
In dev, logs show up instantly in the console. With file logging, there's no immediate feedback. Developers expect real-time visibility, but unless the log file is tailed or piped to a collector, issues stay hidden. This cognitive shift trips up debugging workflows.

💡

Now, you can fix production Go log issues instantly right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time production context, logs, metrics, and traces into your local environment to auto-fix code faster.

## Structured Logging with Performance in Mind

Text logs work fine for simple applications, but structured logging becomes valuable as your system grows. JSON-formatted logs integrate better with monitoring tools and make automated parsing reliable.

Logrus provides structured logging with minimal changes to your existing log calls. The main benefit is adding context fields that persist across related log entries:

```
log := logrus.New()
logFile, _ := os.OpenFile("structured.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
log.SetOutput(logFile)
log.SetFormatter(&logrus.JSONFormatter{})

log.WithFields(logrus.Fields{
    "user_id": 12345,
    "action":  "login",
}).Info("User authentication successful")
```

### Performance Considerations with Zerolog

When logging performance matters, Zerolog offers structured output with minimal memory allocations. The package uses a fluent API that builds log entries without intermediate string formatting:

```
logger := zerolog.New(logFile).With().Timestamp().Logger()

logger.Info().
    Str("service", "auth").
    Int("user_id", 12345).
    Dur("response_time", time.Millisecond*245).
    Msg("Request processed")
```

The performance difference becomes noticeable in applications generating thousands of log entries per second. Zerolog's zero-allocation approach keeps your application responsive under high logging load.

### Concurrency and Thread Safety

Go's log package handles concurrent access safely, which matters when multiple goroutines generate log messages simultaneously. The standard library guarantees serialized access to the underlying writer, preventing garbled output:

```
var wg sync.WaitGroup
wg.Add(3)

// Multiple goroutines logging safely
go func() {
    defer wg.Done()
    for i := 0; i < 10; i++ {
        log.Printf("Goroutine 1: %d", i)
    }
}()
// ... additional goroutines
```

The log package's concurrency safety means you don't need mutexes or channels to coordinate logging across goroutines.

💡

If you're looking for a fast, structured logger with low allocation overhead, check out our guide on [logging errors in Go with Zerolog](https://last9.io/blog/logging-errors-in-go-with-zerolog/), it covers setup, error handling patterns, and performance tips.

## How to Ship File Logs Safely to Production

Production environments require different logging approaches than development. The common pattern involves detecting the environment and configuring loggers accordingly.

### Environment-Specific Configuration

Production environments usually require file-only logging with structured output, while development benefits from console output with readable formatting:

```
func setupLogger() *logrus.Logger {
    log := logrus.New()
    env := strings.ToLower(os.Getenv("APP_ENV"))
    
    switch env {
    case "production":
        logFile, _ := os.OpenFile("/var/log/myapp/app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
        log.SetOutput(logFile)
        log.SetFormatter(&logrus.JSONFormatter{})
        log.SetLevel(logrus.WarnLevel)
    default:
        log.SetOutput(os.Stdout)
        log.SetFormatter(&logrus.TextFormatter{})
        log.SetLevel(logrus.DebugLevel)
    }
    
    return log
}
```

This approach keeps development fast while ensuring production logs are captured with appropriate detail levels and formats.

### Log Rotation and Multiple Outputs

Production logs grow quickly, and unmanaged log files will eventually fill your disk. Lumberjack handles automatic rotation based on file size, age, or backup count:

```
log.SetOutput(&lumberjack.Logger{
    Filename:   "/var/log/myapp/app.log",
    MaxSize:    100, // MB
    MaxBackups: 5,
    MaxAge:     30, // days
    Compress:   true,
})
```

Development and staging environments often benefit from logs appearing in both files and console output. Go's `io.MultiWriter` makes this pattern simple:

```
logFile, _ := os.OpenFile("multi.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
multiWriter := io.MultiWriter(os.Stdout, logFile)
log.SetOutput(multiWriter)
```

### Building Resilient Logging

Robust applications handle logging failures gracefully rather than crashing when log files become unavailable. The fallback pattern ensures your application continues running:

```
func createLoggerWithFallback(primaryPath string) *logrus.Logger {
    log := logrus.New()
    
    logFile, err := os.OpenFile(primaryPath, os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
    if err != nil {
        log.SetOutput(os.Stderr)
        log.Warnf("Failed to open log file %s, using stderr: %v", primaryPath, err)
        return log
    }
    
    log.SetOutput(logFile)
    return log
}
```

## Add Context to Your Logs (Request IDs, Trace Info, and More)

Once logs flow to files, you need visibility into log patterns, error rates, and system health. File-based logs work well with observability platforms that can process structured data and surface trends.

Last9 offers cost-effective, managed observability that handles high-cardinality telemetry at scale. Teams at Probo, CleverTap, and Replit trust it to connect metrics, logs, and traces through OpenTelemetry and Prometheus integrations, giving you performance insights without surprise costs.

### Adding Request Context

One way to go deeper with file logging is threading request context throughout your application. Request IDs and user context make it much easier to trace issues across distributed systems:

```
func handleRequest(w http.ResponseWriter, r *http.Request) {
    requestID := uuid.New().String()
    
    logger := log.WithFields(logrus.Fields{
        "request_id": requestID,
        "method":     r.Method,
        "path":       r.URL.Path,
    })
    
    ctx := context.WithValue(r.Context(), "logger", logger)
    processRequest(ctx)
}
```

Request tracing becomes straightforward when every log entry includes the same correlation ID.

Teams often pair volume metrics with error rate monitoring to spot problems early. Another signal worth watching is log file growth rates and sudden changes in error ratios—these often indicate issues before they affect users.

## Final Thoughts

Go provides solid options for file logging, from the standard library's simplicity to structured logging packages that scale with your application. The key decisions revolve around output format, rotation strategy, and observability integration.

Start with Go's built-in `log` package for basic file output. Move to Logrus or Zerolog when you need structured data and better performance. Add rotation with Lumberjack as your application grows, and consider observability tools to surface patterns in your log data.

💡

And if you want to discuss your setup or troubleshoot a specific issue, [join our Discord](https://last9.io/mcp/), we’ve got a channel focused on practical logging use cases.

## FAQs

**Q: Should I use JSON or text format for log files?**

JSON works better with log aggregation tools and makes parsing easier. Text format is more readable for manual inspection. Choose JSON if you're sending logs to monitoring platforms.

**Q: How do I handle log file permissions in production?**

Set files to `0644` (readable by group, writable by owner) and ensure your service runs as a dedicated user. Create log directories with `0755` permissions.

**Q: What's the best way to rotate log files?**

Use Lumberjack for automatic rotation based on size, age, or backup count. Most teams rotate daily or when files hit 100MB, keeping 5-10 backups.

**Q: Can I log to multiple files simultaneously?**

Yes, use `io.MultiWriter` to send logs to multiple destinations. You can also create separate loggers for different log levels or components.

**Q: How do I test file logging in my Go applications?**

Create temporary files in your tests using `os.CreateTemp()`, write logs, then read and verify the content. Clean up temp files in test teardown.

**Q: Should I buffer log writes for better performance?**

Go's file operations are already buffered by the OS. Adding application-level buffering can improve performance, but it risks losing logs if your application crashes unexpectedly.

**Q: How do I handle disk space issues with log files?**

Set up monitoring for disk usage and implement log rotation. Consider shipping logs to external storage or using log aggregation services for long-term retention.