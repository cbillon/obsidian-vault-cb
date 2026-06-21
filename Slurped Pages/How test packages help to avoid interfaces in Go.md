---
link: https://konradreiche.com/blog/how-test-packages-help-to-avoid-interfaces-in-go/
byline: Konrad Reiche
excerpt: |-
  In my previous post, I highlighted two common misuses of interfaces in Go: interfaces prematurely introduced for abstraction and interfaces created solely to make testing easier.
  The latter, in particular, deserves some hands-on examples to showcase a structured solution. In this post, we’ll dive into an example project and explore how we can move away from using interfaces for dependencies like Redis and gRPC.
  From Interface to Concrete Type
  The first step to eliminating unnecessary interfaces is to use the concrete types whenever possible. Always test the actual implementation if you can; if not, use a fake instead of a mock. A fake simulates your dependency with similar logic, while a mock returns static data.
slurped: 2026-06-21T10:17
title: How test packages help to avoid interfaces in Go
tags:
  - go
---

Oct 27, 2024

In my [previous post](app://obsidian.md/blog/two-common-go-interface-misuses/), I highlighted two common misuses of interfaces in Go: interfaces prematurely introduced for abstraction and interfaces created solely to make testing easier.

The latter, in particular, deserves some hands-on examples to showcase a structured solution. In this post, we’ll dive into an example project and explore how we can move away from using interfaces for dependencies like Redis and gRPC.

## From Interface to Concrete Type

The first step to eliminating unnecessary interfaces is to use the concrete types whenever possible. Always test the actual implementation if you can; if not, use a fake instead of a mock. A fake simulates your dependency with similar logic, while a mock returns static data.

### Why test the actual dependency?

Why bother, you might ask? Testing the actual dependencies increases test coverage, which helps uncover bugs that might otherwise go unnoticed—especially those that slip through critical parts of the execution path. Even with thorough unit test coverage, integration tests can reveal potential issues that unit tests alone might miss.

Testing with concrete types simplifies your code by eliminating unnecessary interfaces and mock implementations. Without interfaces, there’s no need to determine the underlying types, allowing you to work directly with the actual implementations. Unlike interfaces, which require additional effort to trace back to their concrete counterparts, concrete types make it easy to navigate to their definitions.

While suggesting this approach is easy, providing universal guidance is more challenging—if not impossible. Instead, I’ll walk you through an example of an existing service using a mocked dependency with an interface. We’ll introduce another dependency and refactor the code using scoped test packages, allowing us to eliminate unnecessary interfaces without sacrificing the ease of writing tests.

## Example: Eligibility Service

We have an eligibility service that determines whether a user qualifies for a specific product. Some products are restricted to users with certain subscription tiers. To verify eligibility, our service performs the following steps:

1. Look up the product to get its details.
2. Fetch the user information from the user service.
3. Compare the user’s subscription tier with the product’s required tier.

Our project is organized with the following package and file structure:

```
├── deps
│   └── userservice.go
├── product
│   └── catalog.go
└─── service
    ├── service.go
    └── service_test.go
```

The code below implements our eligibility service. The codebase uses a local interface to make testing easier at this stage. Since it’s only a single instance, we haven’t considered it much since the setup seems manageable.

```
package service

type userService interface {
	GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
}

type EligibilityService struct {
	userService userService
	catalog     *product.Catalog
}

func NewEligibilityService(
	userService userService,
	catalog *product.Catalog,
) *EligibilityService {
	return &EligibilityService{
		userService: userService,
		catalog:     catalog,
	}
}
```

Notably, the interface is used both in the struct and the constructor’s parameter types. This makes it unclear where the actual user service dependency is coming from—we’d need to trace the code back to its initialization, which is far from ideal. Below is the code that implements the logic for checking a user’s eligibility:

```
func (s *EligibilityService) IsEligible(
	ctx context.Context,
	userID string,
	productID string,
) (bool, error) {
	product, err := s.catalog.Lookup(productID)
	if err != nil {
		return false, err
	}
	user, err := s.userService.GetUser(ctx, userID)
	if err != nil {
		return false, err
	}
	if user.Subscription < product.MinSubscription {
		return false, nil
	}
	return true, nil
}
```

Finally, we have a test that showcases the purpose of the local `userService` interface. Instead of instantiating a real gRPC service, we use a mock implementation that matches the interface. This approach allows us to explicitly define the return value of the `GetUser` method, making the test clear and easy to understand.

```
func TestService_IsEligible(t *testing.T) {
	userService := &mockUserService{
		getUser: func(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
			return &userpb.GetUserResponse{
				Id:           "test_user_1",
				Subscription: 1,
			}, nil
		},
	}
	catalog := product.NewCatalog()
	svc := NewEligibilityService(userService, catalog)
	result, err := svc.IsEligible(context.TODO(), "test_user_1", "ad-free")
	if err != nil {
		t.Fatal(err)
	}
	if got, want := result, true; got != want {
		t.Errorf("got %t, want: %t", got, want)
	}
}

type mockUserService struct {
	getUser func(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
}

func (m *mockUserService) GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
	if m.getUser != nil {
		return m.getUser(ctx, userID)
	}
	return nil, errors.New("mockUserService.GetUser: not implemented")
}
```

While this approach simplifies testing by defining the mock type locally, it has significant drawbacks. Codebases are dynamic and continuously evolving; early-established patterns often persist and propagate throughout the project. Consider the following scenario as we continue to build upon the existing structure.

### Adding a Redis-backed Cache

To speed up product lookups, we are integrating a Redis-based cache. To achieve this, we created a new package named `cache` that encapsulates this functionality in a generalized manner using type parameters and leverages [`encoding/gob`](https://pkg.go.dev/encoding/gob) to serialize Go values with the gob wire format:

```
package cache

type Cache[T any] struct {
	rdb *redis.ClusterClient
}

func New[T any](rdb *redis.ClusterClient) *Cache[T] {
	return &Cache[T]{
		rdb: rdb,
	}
}

func (c *Cache[T]) Get(ctx context.Context, key string) (T, error) {
	var result T
	b, err := c.rdb.Get(ctx, key).Bytes()
	if err != nil && err != redis.Nil {
		return result, fmt.Errorf("redis.Get: %w", err)
	}
	dec := gob.NewDecoder(bytes.NewReader(b))
	if err := dec.Decode(&result); err != nil {
		return result, fmt.Errorf("gob.Decode: %w", err)
	}
	return result, nil
}

func (c *Cache[T]) Set(ctx context.Context, key string, value T) error {
	var buf bytes.Buffer
	enc := gob.NewEncoder(&buf)
	if err := enc.Encode(value); err != nil {
		return fmt.Errorf("gob.Encode: %w", err)
	}
	return c.rdb.Set(ctx, key, buf.Bytes(), 0).Err()
}
```

Additionally, we ensure test coverage for the cache package itself. Following my earlier recommendation to test against real dependencies, we utilize Docker to run an actual Redis cluster.

```
func TestCache(t *testing.T) {
	rdb := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{
			":7000",
		},
	})
	cache := New[string](rdb)
	if err := cache.Set(context.TODO(), "key", "value"); err != nil {
		t.Fatal(err)
	}
	result, err := cache.Get(context.TODO(), "key")
	if err != nil {
		t.Fatal(err)
	}
	if got, want := result, "value"; got != want {
		t.Errorf("got %s, want: %s", got, want)
	}
}
```

It’s time to integrate the cache into our eligibility service code. We accomplish this by adding the cache to both the service’s type and its constructor:

```
type userService interface {
	GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
}

type EligibilityService struct {
	userService userService

	catalogCache *cache.Cache[*product.Product]
	catalog      *product.Catalog
}

func NewEligibilityService(
	userService userService,
	catalog *product.Catalog,
	catalogCache *cache.Cache[*product.Product],
) *EligibilityService {
	return &EligibilityService{
		userService:  userService,
		catalog:      catalog,
		catalogCache: catalogCache,
	}
}
```

We also update the `IsEligible` method:

```
func (e *EligibilityService) IsEligible(ctx context.Context, userID, productID string) (bool, error) {
	// check if product is already cache
	product, err := e.catalogCache.Get(ctx, productID)
	if err != nil {
		return false, err
	}
	if product == nil {
		// fetch product if not cached and update cache
		product, err = e.catalog.Lookup(productID)
		if err != nil {
			return false, err
		}
		if err := e.catalogCache.Set(ctx, productID, product); err != nil {
			return false, err
		}
	}
	user, err := e.userService.GetUser(ctx, userID)
	if err != nil {
		return false, err
	}
	if user.Subscription < product.MinSubscription {
		return false, nil
	}
	return true, nil
}
```

As expected, our test doesn’t compile anymore:

```
./service_test.go:22:40: not enough arguments in call to NewEligibilityService
	have (*mockUserService, *product.Catalog)
	want (userService, *product.Catalog, *cache.Cache[*product.Product])
FAIL	github.com/konradreiche/example/service [build failed]
```

This raises the question: Should tests in the `service` package require an actual Redis instance, too? Relying on a real Redis cluster for every test is undesirable, as running tests in parallel against it could lead to data races. Alternatively, we could introduce another local interface in our service package tests and mock the results as we did previously:

```
// BAD: Increased Interface Pollution
type cache[T any] interface {
	Get(ctx context.Context, key string) (T, error)
	Set(ctx context.Context, key string, value T) error
}

type userService interface {
	GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
}
```

However, this approach would lead to interface pollution and other issues I discussed in my [previous post](app://obsidian.md/blog/two-common-go-interface-misuses/), such as losing concrete information, hindering our codebase’s effective navigation, and forcing developers (including ourselves) to manage more types than necessary.

### Fakes over Mocks

Is there a way to use the real dependency without its associated overhead? One part of the solution is to use fakes instead of mocks. The distinction wasn’t always clear to me: mocks have a pre-defined state of what they should return, whereas fakes incorporate logic, such as evaluating input parameters or executing a portion of the actual dependencies. Fakes are superior to mocks because they offer greater flexibility and allow us to instantiate the concrete type.

For Redis specifically, there’s an excellent library called [miniredis](https://github.com/alicebob/miniredis), which provides an in-memory Redis replacement with a TCP interface for testing. This enables lightweight tests without the overhead of full integration. Integrating miniredis involves instantiating a `miniredis` instance and initializing a Redis cluster client using the address provided by miniredis:

```
func TestService_IsEligible(t *testing.T) {
	userService := &mockUserService{
		getUser: func(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
			return &userpb.GetUserResponse{
				Id:           "test_user_1",
				Subscription: 1,
			}, nil
		},
	}
	catalog := product.NewCatalog()

	s := miniredis.RunT(t)

	rdb := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{
			s.Addr(),
		},
	})
	cache := cache.New[*product.Product](rdb)
	if err := cache.Set(context.TODO(), "ad-free", &product.Product{
		MinSubscription: 1,
	}); err != nil {
		t.Fatal(err)
	}

	svc := NewProductService(userService, catalog, cache)
	result, err := svc.IsEligible(context.TODO(), "test_user_1", "ad-free")
	if err != nil {
		t.Fatal(err)
	}
	if got, want := result, true; got != want {
		t.Errorf("got %t, want: %t", got, want)
	}
}
```

To our surprise, re-running the test fails:

```
--- FAIL: TestService_IsEligible (0.00s)
    service_test.go:39: gob.Decode: EOF
FAIL
exit status 1
```

As it turns out, using a fake—and thereby testing the cache code as well—reveals a bug. This underscores why we should avoid interfaces for mocks.

The error is somewhat cryptic, but it arises from not handling the case where the error is `redis.Nil`: we pass an empty result to the gob decoder, which promptly returns an end-of-file (EOF) error[1](#fn:1). We updated our cache code to explicitly check for `redis.Nil`, as we don’t want to propagate the pattern of missing keys as failures to the calling code:

```
func (c *Cache[T]) Get(ctx context.Context, key string) (T, error) {
	var result T
	b, err := c.rdb.Get(ctx, key).Bytes()
	if err != nil && err != redis.Nil {
		return result, fmt.Errorf("redis.Get: %w", err)
	}
	if err == redis.Nil {
		return result, nil
	}
	dec := gob.NewDecoder(bytes.NewReader(b))
	if err := dec.Decode(&result); err != nil {
		return result, fmt.Errorf("gob.Decode: %w", err)
	}
	return result, nil
}
```

Updating our code, we are pleased to see that our tests are now passing.

```
PASS
ok  	github.com/konradreiche/example/service	  0.010s
```

While I’m happy that using the concrete type has paid off, the test setup in our service package has become significantly more convoluted. The prospect of this codebase evolving and potentially adding even more dependencies is concerning.

Of course, one could begin to factor this into their own test helper functions localized to the setup, which might work for some time. However, over time, I’ve noticed that these helpers tend to develop slightly different structures and diverge more and more from each other. While this can be manageable, it becomes challenging when working with many different developers. Each may introduce helpers differently, making the codebase challenging and confusing for newcomers, who may not know which approach to follow.

I’ve become a strong advocate for avoiding interfaces in such cases. We miss out on catching bugs and lose track of the concrete types and their significance. However, this is often a hard sell. I urge Go developers, whether seasoned or new to the language, to consider solutions that enforce this rigid style without sacrificing ease of use, ergonomics, and concise test code. However, it places more responsibility on the package owner, bringing me to scoped test packages.

## Scoped Test Packages

Scoped test packages fundamentally differ from what one might consider `testutil` or `testhelpers` packages—which I strongly recommend avoiding. These catch-all packages function similarly to a generic `util.go` file, becoming unwieldy and difficult to maintain over time.

In Go, we prioritize abstractions and packages with precise, focused purposes and limited scopes. Generic catch-all files and packages encourage the unstructured accumulation of code, leading to tangled codebases and making functionality harder to discover and navigate.

Instead, a scoped test package exists solely for the package it accompanies. While mocks allow you to supply test data with minimal code, there’s no reason we can’t achieve the same level of ergonomics with our test packages. This is why I use the [option pattern](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) within these scoped test packages as they facilitates the creation of a highly readable APIs that are easy to extend without necessitating widespread changes across the codebase whenever modifications occur.

### Standardize

Let’s introduce a new package called `cachetest`. I appreciate the importance of standardization, and providing a more rigid template simplifies the process for everyone involved.

A test package resides alongside its corresponding package and follows the `<package>test` naming convention to avoid import conflicts. To achieve this, we’ll utilize the option pattern. Additionally, such a package should facilitate ease of use for anyone interacting with its dependencies, both within and outside the package. Therefore, the API for `cachetest` is designed to provide:

- A fully initialized `cache.Cache` object which is ready for use in any test.
- Flexible setup capabilities allowing configuration of various scenarios.

Our `cachetest/cachetest.go` file is structured as follows:

```
package cachetest

type CacheOption[T any] func(*cacheOptions[T]) error

func New[T any](tb testing.TB, opts ...CacheOption[T]) *cache.Cache[T] {
	cfg := cacheOptions[T]{
		keyValues: make(map[string]T),
	}
	if err := WithCacheOptions(opts...)(&cfg); err != nil {
		tb.Fatal(err)
	}
	s := miniredis.RunT(tb)
	rdb := redis.NewClusterClient(&redis.ClusterOptions{
		Addrs: []string{
			s.Addr(),
		},
	})
	cache := cache.New[T](rdb)
	for key, value := range cfg.keyValues {
		if err := cache.Set(context.TODO(), key, value); err != nil {
			tb.Fatal(err)
		}
	}
	return cache
}

func WithKeyValue[T any](key string, value T) CacheOption[T] {
	return func(options *cacheOptions[T]) error {
		options.keyValues[key] = value
		return nil
	}
}

type cacheOptions[T any] struct {
	keyValues map[string]T
}

// WithCacheOptions permits aggregating multiple options together, and is useful to
// avoid having to append options when creating helper functions or wrappers.
func WithCacheOptions[T any](opts ...CacheOption[T]) CacheOption[T] {
	return func(o *cacheOptions[T]) error {
		for _, opt := range opts {
			if err := opt(o); err != nil {
				return err
			}
		}
		return nil
	}
}
```

It’s quite a bit of code, so let’s walk through it:

1. **CacheOption:** Defines a function type that accepts a pointer to `CacheOptions` and returns an error. This type primarily serves as syntactic sugar, enhancing readability and flexibility when applying multiple configuration options.
    
2. **testing.TB:** The constructor accepts `testing.TB`, which is beneficial because it allows the constructor to handle errors internally by failing the test immediately rather than returning errors to the calling code.
    
3. **Variadic Options:** The constructor also accepts a variable number of `CacheOption` functions. By default, it returns a fully functional `cache.Cache` instance. However, it also allows for overriding and customizing behavior as needed, mirroring the flexibility previously achieved with mocks.
    
4. **Options Exposure:** All configurable options are exposed through clearly named functions, typically starting with `With`. In this example, `WithKeyValue` allows the setup of initial key-value pairs in the cache. This pattern ensures that the API remains intuitive and easy to extend without necessitating widespread changes across the codebase whenever modifications are required.
    

### Integration

Now we get to reap the benefits and integrate it into our test, which ultimately lets us simplify the setup code in our test, as seen in the diff below:

```
diff --git a/service/service_test.go b/service/service_test.go
index dd623dd..7942e89 100644
--- a/service/service_test.go
+++ b/service/service_test.go
@@ -24,14 +22,7 @@ func TestService_IsEligible(t *testing.T) {
 	}
 	catalog := product.NewCatalog()
 
-	s := miniredis.RunT(t)
 
-	rdb := redis.NewClusterClient(&redis.ClusterOptions{
-		Addrs: []string{
-			s.Addr(),
-		},
-	})
-	cache := cache.New[*product.Product](rdb)
-	if err := cache.Set(context.TODO(), "ad-free", &product.Product{
-		MinSubscription: 1,
-	}); err != nil {
-		t.Fatal(err)
-	}
+	cache := cachetest.New(t, cachetest.WithKeyValue(
+		"ad-free",
+		&product.Product{MinSubscription: 1},
+	))
 
 	svc := NewEligibilityService(userService, catalog, cache)
 	result, err := svc.IsEligible(context.TODO(), "test_user_1", "ad-free")
```

The full code is:

```
func TestService_IsEligible(t *testing.T) {
	userService := &mockUserService{
		getUser: func(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
			return &userpb.GetUserResponse{
				Id:           "test_user_1",
				Subscription: 1,
			}, nil
		},
	}
	catalog := product.NewCatalog()
	cache := cachetest.New(t, cachetest.WithKeyValue(
		"ad-free",
		&product.Product{MinSubscription: 1},
	))
	svc := NewEligibilityService(userService, catalog, cache)
	result, err := svc.IsEligible(context.TODO(), "test_user_1", "ad-free")
	if err != nil {
		t.Fatal(err)
	}
	if got, want := result, true; got != want {
		t.Errorf("got %t, want: %t", got, want)
	}
}

type mockUserService struct {
	getUser func(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
}

func (m *mockUserService) GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
	if m.getUser != nil {
		return m.getUser(ctx, userID)
	}
	return nil, errors.New("mockUserService.GetUser: not implemented")
}
```

With this newfound freedom, wouldn’t it be exciting to extend this approach to the user service as well and eliminate the mock type using the same scoped test package approach?

The only challenge with this approach is identifying a suitable candidate for the faking mechanism. In the case of gRPC, we can start a real gRPC service without relying on actual network interfaces by utilizing [`bufconn`](https://pkg.go.dev/google.golang.org/grpc/test/bufconn), a buffered network for in-memory connections. In the Go ecosystem, the `bufconn` package provides an in-memory connection that mimics a real network connection without the overhead of actual socket communication.

For clarity, let’s take a brief look the code inside `deps/userservice.go`, which serves as a thin wrapper around the user service’s protobuf-generated Go code, adding some conveniences such as using more native Go types as parameters:

```
package deps

type UserService struct {
	client userpb.UserServiceClient
}

func NewUserService(addr string, opts ...grpc.DialOption) (*UserService, error) {
	conn, err := grpc.NewClient(addr, opts...)
	if err != nil {
		return nil, err
	}
	return &UserService{
		client: userpb.NewUserServiceClient(conn),
	}, nil
}

func (u *UserService) GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
	resp, err := u.client.GetUser(ctx, &userpb.GetUserRequest{
		Id: userID,
	})
	if err != nil {
		return nil, fmt.Errorf("UserService.GetUser: %w", err)
	}
	return resp, nil
}
```

Returning to our code in `service/service.go`, we aim to eliminate the `userService` interface and instead use the concrete type `*deps.UserService`. With this objective in mind, we’re ready to create our `deps/depstest` helper package.

The `deps/depstest/depstest.go` file contains a generic constructor that allows us to instantiate any gRPC service client, provided we have a constructor for creating new service clients that match the expected signature:

```
type registerer interface {
	RegisterOn(*grpc.Server)
}

func NewFake[T any](
	tb testing.TB,
	svc registerer,
	newServiceClient func(addr string, opts ...grpc.DialOption) (T, error),
	opts ...grpc.DialOption,
) T {
	tb.Helper()

	lis := bufconn.Listen(1024 * 1024)
	srv := grpc.NewServer()

	svc.RegisterOn(srv)
	go func() {
		if err := srv.Serve(lis); err != nil {
			tb.Error(err)
		}
	}()
	tb.Cleanup(srv.Stop)

	bufDialer := func(ctx context.Context, addr string) (net.Conn, error) {
		return lis.DialContext(ctx)
	}
	opts = append([]grpc.DialOption{
		grpc.WithContextDialer(bufDialer),
		grpc.WithTransportCredentials(insecure.NewCredentials()),
	}, opts...)

	client, err := newServiceClient(
		"passthrough://bufnet",
		opts...,
	)
	if err != nil {
		tb.Fatal(err)
	}
	return client
}
```

We create a server that leverages a buffered network, utilizes the passed constructor, and returns a ready-to-use client. This code is essential for generalizing the instantiation of fake gRPC services and the real clients that connect to them.

I prefer to create a separate package for each fake service, clearly indicating that it is a fake in the package name:

```
└── deps
    ├── depstest
    │   ├── depstest.go
    │   └── fakeuserservice
    │       └── fakeuserservice.go
    └── userservice.go
```

This approach ensures that when we call its constructor, there’s no ambiguity about the type of object we’re dealing with—it’s unmistakably a fake. The code for our fake service follows the same structure and applies the option pattern consistently:

```
package fakeuserservice

type Service struct {
	subscriptionByUser map[string]int

	userpb.UnimplementedUserServiceServer
}

func New(
	tb testing.TB,
	opts ...func(*options),
) *deps.UserService {
	options := options{
		subscriptionByUser: make(map[string]int),
	}
	for _, opt := range opts {
		opt(&options)
	}
	svc := &Service{
		subscriptionByUser: options.subscriptionByUser,
	}
	return depstest.NewFake(tb, svc, deps.NewUserService)
}

func (s *Service) GetUser(
	ctx context.Context,
	req *userpb.GetUserRequest,
) (*userpb.GetUserResponse, error) {
	subscription, ok := s.subscriptionByUser[req.Id]
	if !ok {
		return nil, errors.New("user not found")
	}
	return &userpb.GetUserResponse{
		Id:           req.Id,
		Subscription: int64(subscription),
	}, nil
}

func (s *Service) RegisterOn(srv *grpc.Server) {
	userpb.RegisterUserServiceServer(srv, s)
}

type options struct {
	subscriptionByUser map[string]int
}

func WithUserSubscription(userID string, subscription int) func(*options) {
	return func(options *options) {
		options.subscriptionByUser[userID] = subscription
	}
}
```

Although we’ve written a substantial amount of code, it is neatly packaged, allowing us to apply the final optimization to our service package and remove the now unnecessary `userService` interface.

```
diff --git a/service/service.go b/service/service.go
index c7a3eae..e800e64 100644
--- a/service/service.go
+++ b/service/service.go
-type userService interface {
-	GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
-}
-
 type EligibilityService struct {
-	userService userService
+	userService *deps.UserService
 
 	catalogCache *cache.Cache[*product.Product]
 	catalog      *product.Catalog
 }
 
 func NewProductService(
-	userService userService,
+	userService *deps.UserService,
 	catalog *product.Catalog,
 	catalogCache *cache.Cache[*product.Product],
 ) *EligibilityService {
```

We can also eliminate the redundant mock implementation by utilizing our newly added `fakeuserservice` package.

```
diff --git a/service/service_test.go b/service/service_test.go
index 3791e3f..88fdfea 100644
--- a/service/service_test.go
+++ b/service/service_test.go
@@ -2,24 +2,19 @@ package service
 
 func TestService_IsEligible(t *testing.T) {
-	userService := &mockUserService{
-		getUser: func(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
-			return &userpb.GetUserResponse{
-				Id:           "test_user_1",
-				Subscription: 1,
-			}, nil
-		},
-	}
+	userService := fakeuserservice.New(
+		t,
+		fakeuserservice.WithUserSubscription("test_user_1", 1),
+	)
 
 	catalog := product.NewCatalog()
 
@@ -37,14 +32,3 @@ func TestService_IsEligible(t *testing.T) {
 		t.Errorf("got %t, want: %t", got, want)
 	}
 }
-
-type mockUserService struct {
-	getUser func(ctx context.Context, userID string) (*userpb.GetUserResponse, error)
-}
-
-func (m *mockUserService) GetUser(ctx context.Context, userID string) (*userpb.GetUserResponse, error) {
-	if m.getUser != nil {
-		return m.getUser(ctx, userID)
-	}
-	return nil, errors.New("mockUserService.GetUser: not implemented")
-}
```

Resulting in a tidied up test:

```
package service


func TestService_IsEligible(t *testing.T) {
	userService := fakeuserservice.New(
		t,
		fakeuserservice.WithUserSubscription("test_user_1", 1),
	)
	cache := cachetest.New(t, cachetest.WithKeyValue(
		"ad-free",
		&product.Product{MinSubscription: 1},
	))
	catalog := product.NewCatalog()

	svc := NewProductService(userService, catalog, cache)
	result, err := svc.IsEligible(context.TODO(), "test_user_1", "ad-free")
	if err != nil {
		t.Fatal(err)
	}
	if got, want := result, true; got != want {
		t.Errorf("got %t, want: %t", got, want)
	}
}
```

## Conclusion

We have developed integration tests that use fewer lines of code and require no additional types. While this comes at the cost of writing some boilerplate code, the code is encapsulated away from our production code and is only referenced in tests. With this approach, we have achieved:

- Fewer required types.
- No more production code modifications to accommodate test code.
- Clearer test code that is easy to extend.

The approach presented here offers a well-structured method for setting up scoped test packages, providing a pathway to avoid using interfaces solely for testing purposes.

### Other Dependencies

However, implementing this approach can be challenging. While I have demonstrated how to apply it to Redis and gRPC services, the actual effort lies in determining the appropriate fake. For example, I am unaware of an alternative for testing code that relies on Kafka. You might need to run Kafka in Docker or revert to a local interface after all.

My best recommendation is to explore the libraries that you use to integrate your dependencies. Dive into their code to see if they can fake the dependencies without replacing the types. Sometimes, you can shift the fake functionality to a more low-level component within the client library. This requires in-depth work but will pay off in the long run by helping you develop a more maintainable and enjoyable codebase.