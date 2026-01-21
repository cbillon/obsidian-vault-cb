---
tags:
  - golang
  - log
---
In 2023 today, Go 1.21.0 finally comes with a standard level logger: [`log/slog` package](https://pkg.go.dev/log/slog).

No external library is needed anymore for average users.

## 1. Basic Usage

```
package main
import "log/slog"

func main() {
     slog.Info("hello")
     slog.Warn("hello")
     slog.Error("hello")
}
```


```
2023/08/09 20:05:49 INFO hello
2023/08/09 20:05:49 WARN hello
2023/08/09 20:05:49 ERROR hello
```
## 1.1 Enable `Debug` Log Level

[playground](https://go.dev/play/p/fjUelDMm5qt)

`Debug` log level is disabled by default. To enable it, call [`SetLogLoggerLevel()`](https://pkg.go.dev/log/slog#SetLogLoggerLevel) (available in Go 1.22 or later):

```
slog.SetLogLoggerLevel(slog.LevelDebug)

```

## 1.2 Disable Logging

[playground](https://go.dev/play/p/Hg-NMKJcaMA)

If you want (temporarily) to disable logging, the cleanest solution is to define your own logger (see _4. Create Your Own Logger_ for the details).

However, this hacky one-liner works:

```
slog.SetLogLoggerLevel(math.MaxInt)
```

## 2. Add Context

The name _slog_ is short for _structured logging_, meaning each log entry can have a structure.

The log functions can optionally receive any number of key-value pairs called [_attributes_](https://pkg.go.dev/log/slog#hdr-Attrs_and_Values):
```
package main

import "log/slog"

func main() {
     slog.Info("hello", "username", "Mike", "age", 18)
     
     //same as above but more type-safe
     slog.Info("hello", slog.String("username", "Mike"), slog.Int("age", 18))
}

```
``
```
2023/08/09 20:07:51 INFO hello username=Mike age=18
2023/08/09 20:07:51 INFO hello username=Mike age=18 (same as above)
```

(You can even nest key-value pairs via [`Group()`](https://pkg.go.dev/log/slog#Group) or [`GroupAttrs()`](https://pkg.go.dev/log/slog#GroupAttrs).)

## 3. Customize Format

[`log` package](https://pkg.go.dev/log) can be used to customize the format of `log/slog` logger.

This is useful when you want to print finer-grained timestamps, file names and line numbers:

```
package main

import (
     "log"
     "log/slog"
)

func main() {
     slog.Info("hello")
     log.SetFlags(log.Ldate | log.Lmicroseconds | log.Llongfile)
     slog.Info("hello")
}
```


```
2023/08/09 20:15:36 INFO hello
2023/08/09 20:15:36.601583 /home/user/test/a/main.go:9: INFO hello
```
## 4. Create Your Own Logger

Basic usage is covered by the top-level functions (e.g. `slog.Info()`) but you can create your own logger for detailed customization.

A created logger can be set as the default logger via `slog.SetDefault()`. After that, the top-level functions (e.g. `slog.Info()`) use your logger.

### 4.1 Standard Loggers

Constructors are provided in `log/slog` package for some built-in loggers.


```
package main

import (
     "log/slog"
     "os"
)

func main() {
     //text logger
    {         
        //The second argument enables `Debug` log level.
        handler := slog.NewTextHandler(os.Stderr, &slog.HandlerOptions{Level: slog.LevelDebug})
        slog.SetDefault(slog.New(handler))
        
        slog.Debug("hello", "username", "Mike", "age", 18)
    }
    
    //JSON logger
    {         
         handler := slog.NewJSONHandler(os.Stderr, &slog.HandlerOptions{Level: slog.LevelDebug})         
         slog.SetDefault(slog.New(handler))
         
         slog.Debug("hello", "username", "Mike", "age", 18)
    }
}
```
```
time=2023-08-09T20:31:05.798+09:00 level=DEBUG msg=hello username=Mike age=18 {"time":"2023-08-09T20:31:05.798984192+09:00","level":"DEBUG","msg":"hello","username":"Mike","age":18}

```


### 4.2 User-Defined Loggers

By implementing [`Handler`](https://pkg.go.dev/golang.org/x/exp/slog#Handler) interface, you can create a fully-customized logger.

```
package main
import (
    "context"
    "fmt"
    "log/slog"
    "os"
    "time"
)

type MyHandler struct{}

func (h MyHandler) Enabled(context context.Context, level slog.Level) bool {         switch level {
    case slog.LevelDebug:
        return false 
    case slog.LevelInfo:
        fallthrough
    case slog.LevelWarn:
        fallthrough
    case slog.LevelError:
        return true
    default:
        panic("unreachable")
    }
}

func (h MyHandler) Handle(context context.Context, record slog.Record) error {       message := record.Message
    //appends each attribute to the message
    //An attribute is of the form `<key>=<value>` and specified as in `slog.Error(<message>, <key>, <value>, ...)`.     
    record.Attrs(func(attr slog.Attr) bool {
        message += fmt.Sprintf(" %v", attr)
        return true     
    })
    
    timestamp := record.Time.Format(time.RFC3339)
    
    switch record.Level {
        case slog.LevelDebug:
            fallthrough
        case slog.LevelInfo:
            fallthrough
        case slog.LevelWarn:
            fmt.Fprintf(os.Stderr, "[%v] %v %v\n", record.Level, timestamp, message)     
        case slog.LevelError:
            fmt.Fprintf(os.Stderr, "!!!ERROR!!! %v %v\n", timestamp, message)            default:
            panic("unreachable")
    }
    
    return nil
}  

// for advanced users 

func (h MyHandler) WithAttrs(attrs []slog.Attr) slog.Handler {    panic("unimplemented") }
// for advanced users 
func (h MyHandler) WithGroup(name string) slog.Handler {     panic("unimplemented") }

func main() {
    logger := slog.New(MyHandler{})
    slog.SetDefault(logger)
    slog.Debug("hello") //=> does nothing (as `Enabled()` returns `false`)           slog.Info("hello")  //=> [INFO] 2023-11-15T22:38:54+09:00 hello              slog.Warn("hello")  //=> [WARN] 2023-11-15T22:38:54+09:00 hello     slog.Error("hello") //=> !!!ERROR!!! 2023-11-15T22:38:54+09:00 hello      //with attributes     slog.Error("hello", "id", 5, "foo", "bar") //=> !!!ERROR!!! 2023-11-15T22:38:54+09:00 hello id=5 foo=bar

}
```
### 4.3 User-Defined Log Levels

As the type of the log levels is defined as [`type Level int`](https://pkg.go.dev/golang.org/x/exp/slog#Level), you can even define your own log levels: [The Go Playground](https://go.dev/play/p/ZG06erZhMfL)