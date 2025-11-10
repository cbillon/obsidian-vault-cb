---
link: https://nyadgar.com/posts/go-interfaces-why-how-and-when/
excerpt: |-
  By Noam Yadgar
   
   at January 12, 2025
slurped: 2025-11-08T11:27
title: "Go Interfaces: Why, How and When"
tags:
  - go
  - interface
---

[By Noam Yadgar](mailto:noam.g4@gmail.com) at January 12, 2025

In programming, interfaces are a powerful concept that lets us express our code more abstractly. Interfaces allow us to reason about the higher-level logic of our processes without getting down to the small details. Go has arguably one of the best interface implementations. With great features like implicit implementations, assertion, and more.

Interfaces must be used cautiously since they introduce more abstraction to our code, making it susceptible to unnecessary wrappers, misuse of definitions, and sometimes, even memory issues. In this article, I will discuss cases where interfaces can positively impact your code. But first, let’s talk about what an interface is not.

## Not for hiding code

I’ve encountered a common claim that interfaces let you hide internal code and expose only the relevant details. However, we can also do this by using the _exported_ values feature in Go. Interfaces are less about hiding details for the sake of hiding and more about defining a contract between layers and packages in your system.

![interface](https://nyadgar.com/posts/go-interfaces-why-how-and-when/interface.png)

## Using interfaces

Interfaces let us put together the higher-level logic. For example, we can write a simple function that extracts the `string` of an `fmt.Stringer` interface and write its bytes to an `io.Writer`. This function, on its own, doesn’t do anything but only represents the interaction between two types that satisfy these interfaces:

```
func writeStrTo(s fmt.Stringer, w io.Writer) (int, error) {
	return w.Write([]byte(s.String()))
}
```

The `Stringer` interface, defined in the `fmt` package is:

```
type Stringer interface {
	String() string
}
```

### Implementing an interface

Let’s implement a struct that _satisfies_ the `Stringer` interface:

```
type kv struct {
	key   string
	value string
}

func (t kv) String() string {
	return fmt.Sprintf("%s=\"%s\"\n", t.key, t.value)
}
```

Now, let’s write a `main` function that iterates through a slice of our `kv` type and appends each element to `os.Stdout`. Notice that in Go, interfaces are inferred implicitly. In other words, we don’t need to tell the compiler that our `kv` type acts as an `fmt.Stringer` when we pass it to the function.

Implementations of interfaces in Go are always passed by reference (pointers)

```
func main() {
	kvs := []kv{
		{key: "ENV", value: "dev"},
		{key: "LEVEL", value: "debug"},
	}

	for _, t := range kvs {
		if _, err := writeStrTo(t, os.Stdout); err != nil {
			panic(err)
		}
	}
}
```

If we run this, we get:

```
go run main.go
ENV="dev"
LEVEL="debug"
```

We’re using `os.Stdout` as our `io.Writer` parameter. `os.Stdout` is an `*os.File` which satisfies the `io.Writer` interface.

## Providing interfaces

You can provide your interfaces for a few good reasons. One good reason is that it lets you supply multiple implementations.

### Multiple implementations

Defining an interface in a package is common whenever you have multiple types that adhere to a specific set of methods. This way, you allow the consumer of your package to use the interface inside their logic and plug in an implementation that fits their needs.

For example, in a hypothetical _cache_ package, we can define this interface:

```
type Cache interface {
	Set(k string, v any, opts ...any) error
	Get(k string) (any, bool) error
	Delete(k string) error
	Clear() error
}
```

Your package may offer a few different implementations of the Cache interface. One way of providing those types is by exposing constructor functions such as `NewDefaultCache`, `NewLruCache`, `NewRedisCache`, etc. This pattern follows the strong _golden rule_:

**Functions return concrete types and accept interfaces**.

flowchart LR
    C[c.NewDefaultCache] -->|*cache| B
    D[c.NewLruCache] --> |*lruCache| B
    A[c.NewRedisCache] -->|*redisCache| B("NewHandler(c.Cache)")

### Be agnostic

Have a look at this definition of a message broker, can you spot the problem?

```
import (
	"context"

	amqp "github.com/rabbitmq/amqp091-go"
)

type MessageBroker interface {
	Send(ctx context.Context, target string, msg amqp.Publishing)
	Receive(
        ctx context.Context,
        target string,
        handler func(context.Context, amqp.Delivery) error,
    )
}
```

The problem is that we’re defining an interface that should generally pack the typical behavior of a message broker, but we’re using concrete types of specific technology.

To begin with, we should see if wrapping our message broker with such an abstraction layer is even necessary. A good reason might be that we’re working on a large enterprise-level code base, and we want to design the system such that the underlying technology of our message broker can be easily swapped with a different one without causing any dramatic refactor.

Therefore, it’s better to aim for the most _minimal_ and _general_ definitions. One that has no _hints_ towards any specific technology but provides just enough definition to carry the behavior of any message broker. A _perfect_ interface is never guaranteed, but it will be much easier for concrete types to adopt simple definitions than the other way around. The general rule here is:

**Don’t imply specific implementations in your interfaces**.

This may not be perfect, but a better one.

```
import (
	"context"
)

type MessageBroker interface {
    Sender
    Receiver
}

type Sender interface {
	Send(context.Context, Message) error
}

type Receiver interface {
	Receive(context.Context, MessageHandlerFunc) error
}

type Message interface {
    Body() []byte
    Metadata() map[string]any
}

type MessageHandlerFunc func(context,Context, Message) error
```

---
  config:
    class:
      hideEmptyMembersBox: true
---
classDiagram
    MessageBroker <|.. rabbitmq
    MessageBroker <|.. sqs
    MessageBroker <|.. kafka
    
    app-1 ..> MessageBroker
    app-2 ..> MessageBroker
    app-n ..> MessageBroker

    class MessageBroker:::intr
    class rabbitmq
    class sqs
    class kafka
    class app-1:::srv
    class app-2:::srv
    class app-n:::srv

    classDef srv fill:#f97,stroke:#d75
    classDef intr fill:#aee,stroke:#d75

### Provide a platform

Especially when building a framework of some sort, sometimes you’d like to provide consumers of your package with a definition so that they can implement their logic and plug it back into your process. For example, maybe you wrote a package that manages the lifecycle of an application, and you’re expecting consumers to implement an `App` interface like:

```
type App interface {
	Init(context.Context) error
	Run() error
	Stop() error
}
```

To achieve the complete pattern, you may also provide a function that accepts the `App` interface and performs the necessary lifecycle control flow. Hypothetically, a function like:

```
var Timeout = time.Second * 15

func Deploy(app App) {
	defer func() {
		if err := app.Stop(); err != nil {
			panic(err)
		}
	}()

	ctx, cancel := context.WithTimeout(
		context.Background(), Timeout,
	)
	defer cancel()

	errchan := make(chan error)
	go func() {
		errchan <- app.Init(ctx)
	}()

	select {
	case <-ctx.Done():
		if err := ctx.Err(); err != nil {
			panic(err)
		}
	case err := <-errchan:
		if err != nil {
			panic(err)
		}
	}

	ch := make(chan os.Signal, 1)
	signal.Notify(ch, syscall.SIGTERM, syscall.SIGINT)
	go func() {
		sig := <-ch
		errchan <- fmt.Errorf("received signal: %s", sig.String())
	}()

	go func() {
		errchan <- app.Run()
	}()

	if err := <-errchan; err != nil {
		panic(err)
	}
}
```

Consumers of your package can now call `Deploy` with their implementation of an `App` and expect your package to manage their app’s lifecycle.

Please note that the code above is not battle-tested and is used for an educational purpose

## Test mocks

One important feature of interfaces is that they allow for better unit testing by implementing a _mock_ whenever the interface is used in your internal code. I’ve seen a few claims about how test mocks are degrading the quality of the tests and how you’re not testing the _real_ logic and interaction.

It’s the other way around. Using test mocks, you can _stage_ your **exact** test scenarios with absolute control. You’re not relying on a non-deterministic process or the outside environment; you can genuinely unit-test your logic. Let’s implement an `App` that can produce all failed cases.

```
type appMock struct {
	failOnInit,
	failOnRun,
	failOnStop,
	sendInterrupt bool
	sleep time.Duration
}

func (a *appMock) Init(context.Context) error {
	if a.failOnInit {
		return fmt.Errorf("App.Init: failed")
	}

	time.Sleep(a.sleep)
	return nil
}

func (a *appMock) Run() error {
	if a.failOnRun {
		return fmt.Errorf("App.Run: failed")
	}

	if a.sendInterrupt {
		go func() {
			_ = syscall.Kill(syscall.Getpid(), syscall.SIGINT)
		}()
		time.Sleep(time.Millisecond * 500)
	}

	return nil
}

func (a *appMock) Stop() error {
	if a.failOnStop {
		return fmt.Errorf("App.Stop: failed")
	}

	return nil
}
```

Our `Deploy` function is a good example of a relatively complex function. It uses `panic` (a rare thing to do in Go), channels, and a timeout. Those will be hard to test if we haven’t had our test mock. Since we have created a mock that can produce all cases, our unit test can look like this:

```
func TestDeploy_happyFlow(*testing.T) {
	Deploy(&appMock{})
}

func TestDeploy_panic(t *testing.T) {
	tc := []struct {
		name string
		app  App
		msg  string
	}{
		{
			name: "panic on Init",
			app:  &appMock{failOnInit: true},
			msg:  "App.Init: failed",
		},
		{
			name: "panic on Init timeout",
			app:  &appMock{sleep: time.Second},
			msg:  "context deadline exceeded",
		},
		{
			name: "panic on Run",
			app:  &appMock{failOnRun: true},
			msg:  "App.Run: failed",
		},
		{
			name: "panic on Stop",
			app:  &appMock{failOnStop: true},
			msg:  "App.Stop: failed",
		},
		{
			name: "panic on SIGINT",
			app:  &appMock{sendInterrupt: true},
			msg:  "received signal: interrupt",
		},
	}

	Timeout = time.Millisecond * 500
	for _, tt := range tc {
		t.Run(tt.name, func(t *testing.T) {
			testDeployWithPanic(tt.app, tt.msg, t)
		})
	}
}

func testDeployWithPanic(app App, msg string, t *testing.T) {
	defer func() {
		r := recover()
		if r == nil {
			t.Errorf("expected a panic")
			return
		}

		err, ok := r.(error)
		if !ok {
			t.Errorf("expected error in panic")
			return
		}

		if err.Error() != msg {
			t.Errorf("error returned unexpected message: %s", err.Error())
		}
	}()

	Deploy(app)
}
```

### Mock libraries

Test mocks are so common that some of Go’s most popular open-source packages are dedicated to their automatic creation. One example is [Uber’s mockgen and gomock](https://pkg.go.dev/go.uber.org/mock) . A set of tools that automatically generates test mocks from any interface and integrates them into your unit tests.

Let’s generate a mock for our `Cache` interface:

```
mockgen -package internal -source ./cache.go -destination mocks.go
```

The command above takes our `Cache` interface, stored in `cache.go` and outputs a new file, named `mocks.go`, with our `Cache` mock under the `internal` packages. In our tests, we can use the mock like the following:

```
func TestHandler(t *testing.T) {
    // spawn a new mock instance
	ctl := gomock.NewController(t)
	cache := NewMockCache(ctl)

    // "stage" its expected calls and returns
	cache.EXPECT().Get("test").Return(1)

    // test your internal logic... 
    handler := NewHandler(cache)
    if err := handler.Get("test"); err != nil {
        t.Errorf(err)
    }
}
```

### Slice out a definition

Sometimes, a third-party package may provide a concrete type (usually a client of some service), and to write proper unit tests, you will provide an interface that _slices out_ the set of used methods from this type. Essentially, defining an interface that the third-party type will satisfy.

For example, testing internal logic that uses the `s3.Client` from [AWS Go SDK](https://github.com/aws/aws-sdk-go-v2) will require a connection to a real S3-compatible bucket and will rely on the side effects of this bucket. This will completely miss the point of unit testing, which is supposed to be deterministic and independent of the environment.

The solution is to define an interface that can _fit_ the `s3.Client` and use this interface instead. The `s3.Client` has about 192 methods (true to version `1.50.0`), so defining an interface that fits all of this type is tedious and unnecessary. Instead, you can _slice out_ only the methods you’re using from the `s3.Client`.

Maybe you only need the `s3.Client` to call `GetObject`. You can define your interface as follows:

```
type S3Client interface {
    GetObject(
        context.Context, 
        *s3.GetObjectInput, 
        ...func(*s3.Options),
    ) (*s3.GetObjectOutput, error)
}
```

This interface completely contradicts my claim of being agnostic. In this pattern we’re not trying to abstract the behavior of any storage service, but we’re explicit about our intentions of using S3 exclusively. Defining an abstract storage service interface would result in implementing a wrapper to the `s3.Client`, another layer of abstraction that might be unnecessary if we’re not planning on replacing S3 in our system.

By defining and consuming the interface above, we can now easily generate a test mock and write proper unit tests for our internal logic, interacting with the mock as if we’re communicating via a real S3 client.

flowchart LR
    A["s3.NewFromConfig(sdkConfig)"] -->|*s3.Client| B("myFunction(S3Client)")

We would like to use third party cookies and scripts to improve the functionality of this website. Approve Deny [More info](https://nyadgar.com/privacy)