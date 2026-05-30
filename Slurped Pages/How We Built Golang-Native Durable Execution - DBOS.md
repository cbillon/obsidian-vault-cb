---
link: https://www.dbos.dev/blog/how-we-built-golang-native-durable-execution
excerpt: How DBOS augments the Go-native context.Context interface to implement durable execution with compile-time type safety for workflow and step functions
slurped: 2026-05-29T19:36
title: How We Built Golang-Native Durable Execution | DBOS
tags:
  - golang
  - workflow
  - architecture
---

### Introduction

Building a workflows library that feels natural and familiar to Go developers is trickier than it sounds because while Go has some great features (language support for context.Context, an incredible async runtime) it also has some brutal limitations for library builders (particularly its limited type system). In this blog post, we'll look at what it takes to make workflows work in Go, in particular:

- How to augment the Go native **context.Context** interface to implement durable execution
- How to reconcile providing a single interface with providing compile-time type safety to workflow and step functions

### Durable Context

To compose a set of Go functions into a workflow of steps, we need to keep track of their execution state (e.g., checkpoint progress) and capture their relationship within a workflow. 

Go already has a native way to coordinate execution: **context.Context**. A Go context can represent a hierarchy of tasks: their parent-child relationship matches naturally the relationship between workflow and steps. Go context can also be used to carry metadata and signal cancellation.

We designed our durable workflows library around an extended **context.Context** interface: **durable.Context**. It exposes methods to invoke user functions as workflows or steps, and leverages its underlying Go context to automatically propagate workflow metadata across functions.

When designing **durable.Context**, we had three goals:

1. Single interface: **durable.Context** should be the only interface the user has to understand.
2. Feels like Go: **durable.Context** is a normal Go **context.Context**.
3. Type-safety: Workflows and steps should get compile-time type checking.

In the next sections, we will examine the methods **durable.Context** exposes to wrap user functions, how it leverages Go **context.Context** for durable execution, and finally how we designed the package interface to meet all goals.

### A familiar Go context

**durable.Context** exposes two key methods, **RunWorkflow** and **RunAsStep**, that users can call to run normal Go functions as workflows and steps. **RunWorkflow** manages the workflow execution state and executes the user-provided workflow function in a new goroutine. To do so, **RunWorkflow** creates a new child **durable.Context**, seeded with the workflow metadata (like its idempotency key.) When a workflow function calls **RunAsStep**, the programmer must pass it the workflow context.

Passing a context as the first argument of a Go function is very common for Go programmers, so we felt confident about this requirement. Under the hood, **RunWorkflow** and **RunAsStep** inspect the context to durably checkpoint the state of workflow.

### Deadlines and cancellation

Another compelling reason to design **durable.Context** around Go’s native **context.Context** was to leverage its deadline and cancellation facilities. To set a workflow timeout, use this familiar pattern:

```
cancelCtx, cancelFunc := durable.WithTimeout(durableCtx, 24*time.Hour)
durable.RunWorkflow(cancelCtx, workflowFn, "input")
```

When **RunWorkflow** detects a deadline in its provided **Context**, using the **Deadline** method, it registers an **AfterFunc** with the workflow context, which will run in its own goroutine and be triggered whenever the workflow context is cancelled. Its job is to durably set the workflow status to “cancelled” in the database.

But remember: cancellation is not preemption. Cancelling a Go context is just a way to signal that work done on the context’s behalf should stop. Workflow and step functions can access the context’s **Done** channel and act upon it. We do not attempt to preempt workflow goroutines, rather, entering a new step checks whether the workflow was cancelled before executing.

### A single - yet type safe - interface

One of the reasons developers choose Go over higher-level languages like Python for backend work is its static type system. Libraries that wrap user functions (like **RunWorkflow**) have a tradeoff to make between flexibility and safety. The wrapper’s signature must be carefully defined: either enforce specific function shapes to preserve compile-time guarantees, or relax the signature to accept “**any**” functions, thus sacrificing static type checking and requiring runtime reflection.

### Workflows and Steps Signature

The ideal workflow and step signature is fully generic: it accepts a variadic number of inputs and outputs, each of any possible type. But Go does not support variadic generics (the topic of [long conversations](https://github.com/golang/go/issues/66651) with the community). Instead, we settled on flexible signatures that should feel familiar to Go developers:

```
type Workflow[P any, R any] func(ctx durable.Context, input P) (R, error)
type Step[R any] func(ctx context.Context) (R, error)
```

As we noted earlier, many Go functions use a context as the first parameter and return an error, and **durable.Context** is a valid **context.Context**. The input and output values can have arbitrary amounts of properties. Steps are more flexible and allow users to bring existing functions, get compile-time type checking for return values, and allow our library to propagate **durable.Context**. For example, users can write:

```
durable.RunAsStep(durableCtx, func(ctx context.Context) (ReturnType, error) {  
  return myExistingFunction(ctx, arg1, arg2) // args captured from outer scope.
})
```

We considered a few alternative signatures based on receiving **any** function and a variadic number of **any** parameters. But this approach would have lost compile-time type checking and requires overly complex usage of reflection, which is error prone and very complicated. Is **workflowFunc** a function? Are the arguments matching the signature of **workflowFunc**?

### Generic package functions

Having generic workflow and step signatures poses a challenge for **durable.Context**’s **RunWorkflow** and **RunAsStep** methods. For an interface method to be generic, the entire interface must be generic. Which means that we’d have needed one **durable.Context** per function signature (which violates our goal of having a single root context). One tempting solution was to introduce **Workflow** and **Step** generic interfaces, but this would fail our goal of having a single interface and keep the programming model simple.

Our solution is to expose generic package functions mirroring **durable.Context** methods, handling type conversions at runtime and calling typeless **durable.Context** methods under the hood. In the case of **RunAsStep**:

```
func RunAsStep[R any](ctx Context, fn Step[R], opts ...StepOption) (R, error) {  
  typeErasedFn := StepFunc(func(ctx context.Context) (any, error) { return fn(ctx) })
  result, err := ctx.RunAsStep(ctx, typeErasedFn, opts...)
  return result.(R), err
}
```

You are probably noticing that **Context.RunAsStep** takes a **Context** as the first argument. This is to help with another aspect of the UX, very important to Golang users: mocking. If I write a program that does:

```
durable.RunWorkflow(ctx, ExampleWorkflow, "some-input")
```

And I want to mock **ctx**, let’s say, with [mockery](https://vektra.github.io/mockery/latest/), I want to be able to do **mockCtx.On(“RunWorkflow”, arg1, arg2, arg3)**, because that’s how my program reads: 3 arguments to **RunWorkflow**.

### Serialization

Durable workflows must have their input/output persisted. This means we need to automatically encode native Go types and be able to decode them at any time, from anywhere.

We first elected Go native’s [encoding/gob](https://pkg.go.dev/encoding/gob) for the job. But we faced a challenge in supporting list methods that return a slice of workflows or steps. On the encoding path, our library has access to the value’s type. For example, encoding of workflow inputs is done in **RunWorkflow**, where it knows the input’s generic type **T**, as described previously. But on the decoding path, we don’t always have access to this type. For example, we expose a **ListWorkflows** method which returns a slice of **WorkflowStatus** to the user. Golang does not support heterogeneous generic slices, so elements of this slice must store workflow input/output as **any** variable. The challenge is that gob cannot decode into **any** a value that was encoded from a concrete type.

So we reverted course and fell back on [encoding/json](https://pkg.go.dev/encoding/json). While we think gob is a superior encoding mechanism, (e.g., it has better support for encoding/decoding interfaces), the decoding logic is much simpler with JSON. We can return encoded JSON strings on the list paths, which the end user can manipulate easily.

### Learn more

The library has many more examples of idiosyncratic Go, for example: functional option patterns, custom Error type, and sync.Map for read-mostly access patterns. If you like Go, we’d love to hear from you. At DBOS, our goal is to make durable workflows as lightweight and easy to work with as possible. Check it out:

- Quickstart: [https://docs.dbos.dev/quickstart](https://docs.dbos.dev/quickstart) 
- GitHub: [https://github.com/dbos-inc/dbos-transact-golang](https://github.com/dbos-inc/dbos-transact-golang)

‍