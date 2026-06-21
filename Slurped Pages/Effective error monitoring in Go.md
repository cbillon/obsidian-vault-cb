---
link: https://konradreiche.com/blog/effective-error-monitoring-in-go/
byline: Konrad Reiche
excerpt: "The simplicity of Go’s error handling system is a big strength. What seems repetitive is a continuous prompt to the author of the code: should I handle the error or return it? Most of the time the error will be returned. At some point in the caller chain this decision might change to handling the error, for example by logging it: here we also stop passing the error further up."
slurped: 2026-06-21T10:14
title: Effective error monitoring in Go
tags:
  - go
---

Sep 3, 2022

The simplicity of Go’s error handling system is a big strength. What seems repetitive is a continuous prompt to the author of the code: **should I handle the error or return it?** Most of the time the error will be returned. At some point in the caller chain this decision might change to handling the error, for example by logging it: here we also stop passing the error further up.

An error log provides insight into common errors produced by the system. Not all errors in Go are possible to prevent, for example a network request timeout to an endpoint is outside of the control of the executing code.

Systems in Go which are being continuously developed can generate a growing variety of errors. Logging all of them will produce an error monitoring system that is noisy and too tempting to be ignored when deploying code to production. Even vigilant engineers will end up having to look for the needle in the haystack. To reduce the overall volume of errors for common cases the following framework can be used to make error monitoring in production manageable.

## Loggable Errors

Assuming you use the following line to handle an error by logging it:

```
if err != nil {
  log.Error(err)
}
```

For a class of well known errors that do not require additional attention, we introduce the concept of a loggable error:

```
type loggable interface {
  error
  Log() bool
}
```

An error can implement this interface by providing a `Log` method which indicates if the error wants to be logged. The `error` type is embedded to ensure this only applies to errors.

## Observed Errors

You might already have a series of different errors that are acceptable to happen but you still want to be able to observe them in case their occurrence starts to increase. Instead of logging them, you can monitor the event as a metric. **An error event which has been observed as a metric can be considered handled**. Once you record the event by emitting a metric, you can wrap the error as an `ObservedError` which is an implementation of the `loggable` interface.

```
type ObservedError struct {
  cause error
}

func NewObservedError(err cause) *ObservedError {
  return &ObservedError{
    cause: cause,
  }
}

func (e *ObservedError) Log() bool {
  return false
}

func (e *ObservedError) Unwrap() error {
  return e.cause
}

func (e *ObservedError) Error() string {
  return e.Unwrap().Error()
}
```

To make this error tracking framework easy to use across your code base you may want to provide a helper function to keep error handling code short.

```
func LogIfUnknown(err error, keysAndValues ...any) {
  var e loggable
  if errors.As(err, &e) && !e.Log() {
    // error does not want to be logged
    return
  }
  log.Error(err, keysAndValues...)
}
```

Your error logging code would now change to the following everywhere:

```
if err != nil {
  LogIfUnknown(err)
}
```

By making use of some of Go’s best practices like, unexported interfaces, error wrapping and using interfaces to define behavior, we are able to provide a framework which can be used to continiously classify errors, decide which should be observed as a metric and which should be logged.