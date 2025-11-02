---
link: https://packagemain.tech/p/integration-tests-using-testcontainers
byline: Alex Pliutau
site: packagemain.tech
date: 2024-08-14T12:13
excerpt: A hands-on guide on Integration Tests in Go using Testcontainers.
slurped: 2025-11-02T18:10
title: Emulating real dependencies in Integration Tests using Testcontainers
tags:
  - go
  - test
---

Contrary to unit testing, the purpose of integration tests is to validate that different software components, subsystems or applications work well together combined as a group.

It’s a very important step in the testing pyramid, that can help to identify the issues that arise when the components are combined, for example compatibility issues, data inconsistence, communication issues.

In this article, we define the integrations tests as tests of communication between our backend application and external components such as database and cache.

[

![](https://substackcdn.com/image/fetch/$s_!9zyZ!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff2badf2a-f4cc-4bd6-ac74-cb7026aa890c_1100x786.png)

](https://substackcdn.com/image/fetch/$s_!9zyZ!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff2badf2a-f4cc-4bd6-ac74-cb7026aa890c_1100x786.png)

This diagram shows only 3 types of tests, however there are other kinds as well: components tests, system tests, load testing, etc.

While unit tests are easy to run (you just execute tests as you would execute your code), integration tests usually require some scaffolding. In the companies I worked I’ve seen the following approaches to address the integration testing environment problem.

**Option 1.** Using the throwaway databases and other dependencies, which must be provisioned before the integration tests start and destroyed afterwards. Depending on your application complexity the effort of this option can be quite high, as you must ensure that the infrastructure is up and running and data is pre-configured in a specific desired state.

**Option 2.** Using the existing shared databases and other dependencies. You may create a separate environment for integration tests or even use the existing one (staging for example) that integration tests can use. But there are many disadvantages here, and I would not recommend it. Because it is a shared environment, multiple tests can run in parallel and modify the data simultaneously, therefore you may end up with inconsistent data state for multiple reasons.

**Option 3.** Using in-memory or embedded variations of the required services for integration testing. While this is a good approach, not all dependencies have in-memory variations, and even if they do, these implementations may not have the same features as your production database.

**Option 4.** Using [Testcontainers](https://testcontainers.com/) to bootstrap and manage your testing dependencies right inside your testing code. This ensures a full isolation between test runs, reproducibility and better CI experience. We will dive into that in a second.

Do demonstrate the tests we prepared a dead simple URL shortener API written in Go, which uses MongoDB as a data storage and Redis as a [read-through cache](https://www.enjoyalgorithms.com/blog/read-through-caching-strategy). It has two endpoints which we’ll be testing in our tests:

- **/create?url=** generates the hash for a given URL, stores it in database.
    
- **/get?key=** returns the original URL for a given key.
    

We won’t delve into the details of the endpoints much, you can find the full code in [this Github repository](https://github.com/plutov/packagemain/blob/master/testcontainers-demo/main.go). However, let’s see how we define our “server“ struct:

```
type server struct {
  DB    DB
  Cache Cache
}

func NewServer(db DB, cache Cache) (*server, error) {
  if err := db.Init(); err != nil {
    return nil, err
  }

  if err := cache.Init(); err != nil {
    return nil, err
  }

  return &server{DB: db, Cache: cache}, nil
}
```

The **NewServer** function allows us to initialize a server with database and cache instances that implement DB and Cache interfaces.

```
type DB interface {
  Init() error
  StoreURL(url string, key string) error
  GetURL(key string) (string, error)
}

type Cache interface {
  Init() error
  Set(key string, val string) error
  Get(key string) (string, bool)
}
```

Because we had all dependencies defined as interfaces, we can easily generate mocks for them using [mockery](https://github.com/vektra/mockery) and use them in our unit tests.

```
mockery --all --with-expecter
go test -v ./...
```

With the help of unit tests we can cover quite well the low level components or our application: endpoints, hash key logic, etc. All we need is to mock the function calls of database and cache dependencies.

> unit_test.go

```
func TestServerWithMocks(t *testing.T) {
  mockDB := mocks.NewDB(t)
  mockCache := mocks.NewCache(t)

  mockDB.EXPECT().Init().Return(nil)
  mockDB.EXPECT().StoreURL(mock.Anything, mock.Anything).Return(nil)
  mockDB.EXPECT().GetURL(mock.Anything).Return("url", nil)

  mockCache.EXPECT().Init().Return(nil)
  mockCache.EXPECT().Get(mock.Anything).Return("url", true)
  mockCache.EXPECT().Set(mock.Anything, mock.Anything).Return(nil)

  s, err := NewServer(mockDB, mockCache)
  assert.NoError(t, err)

  srv := httptest.NewServer(s)
  defer srv.Close()

  // actual tests happen here, see the code in the repository
  testServer(srv, t)
}
```

**mocks.NewDB(t)** and **mocks.NewCache(t)** have been auto-generated by mockery and we use **EXPECT()** to mock the functions. Notice that we created a separate function **testServer(srv, t)** that we will use later in other tests as well, but providing a different server struct.

As you may already understand these unit tests are not testing the communications between our application and our database/cache, and we may easily skip some very critical bugs. To be more confident with our application, we should write integration tests along with unit tests to ensure that our application is fully functional.

As Option 1 and 2 mention above, we can provision our dependencies beforehand and run our tests against these instances. One option would be to have a Docker Compose configuration with MongoDB and Redis, which we start before the tests and shutdown after. The seed data could be a part of this configuration, or done separately.

> compose.yaml

```
services:
  mongodb:
    image: mongodb/mongodb-community-server:7.0-ubi8
    restart: always
    ports:
      - "27017:27017"

  redis:
    image: redis:7.4-alpine
    restart: always
    ports:
      - "6379:6379"
```

> realdeps_test.go

```
//go:build realdeps
// +build realdeps

package main

func TestServerWithRealDependencies(t *testing.T) {
  os.Setenv("MONGO_URI", "mongodb://localhost:27017")
  os.Setenv("REDIS_URI", "redis://localhost:6379")

  s, err := NewServer(&MongoDB{}, &Redis{})
  assert.NoError(t, err)

  srv := httptest.NewServer(s)
  defer srv.Close()

  testServer(srv, t)
}
```

Now these tests don’t use mocks, but simply connect to already provisioned database and cache. Note: we added a “realdeps“ build tag so these tests should be executed by specifying this tag explicitly.

```
docker-compose up -d
go test -tags=realdeps -v ./...
docker-compose down
```

However, creating reliable service dependencies using Docker Compose requires good knowledge of Docker internals and how to best run specific technologies in a container. For example, creating a dynamic integration testing environment may result in port conflicts, containers not being fully running and available, etc.

With Testcontainers we can now do the same, but inside our test suite, using our language API, which means we can control our throwaway dependencies better and make sure they’re isolated per each test run. You can run pretty much anything in Testcontainers as long as it has a Docker-API compatible container runtime.

> integration_test.go

```
//go:build integration
// +build integration

package main

import (
  "context"
  "net/http/httptest"
  "os"
  "testing"

  "github.com/stretchr/testify/assert"
  "github.com/testcontainers/testcontainers-go/modules/mongodb"
  "github.com/testcontainers/testcontainers-go/modules/redis"
)

func TestServerWithTestcontainers(t *testing.T) {
  ctx := context.Background()

  mongodbContainer, err := mongodb.Run(ctx, "docker.io/mongodb/mongodb-community-server:7.0-ubi8")
  assert.NoError(t, err)
  defer mongodbContainer.Terminate(ctx)

  redisContainer, err := redis.Run(ctx, "docker.io/redis:7.4-alpine")
  assert.NoError(t, err)
  defer redisContainer.Terminate(ctx)

  mongodbEndpoint, _ := mongodbContainer.Endpoint(ctx, "")
  redisEndpoint, _ := redisContainer.Endpoint(ctx, "")

  os.Setenv("MONGO_URI", "mongodb://"+mongodbEndpoint)
  os.Setenv("REDIS_URI", "redis://"+redisEndpoint)

  s, err := NewServer(&MongoDB{}, &Redis{})
  assert.NoError(t, err)

  srv := httptest.NewServer(s)
  defer srv.Close()

  testServer(srv, t)
}
```

It’s very similar to the previous test, we just initialized 2 containers at the top of our test.

The first run may take a while to download the images. But the subsequent runs are almost instant.

[

![](https://substackcdn.com/image/fetch/$s_!FHVw!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff9acb596-6f3c-4255-8adc-60dd6113d8bc_2048x1644.png)

](https://substackcdn.com/image/fetch/$s_!FHVw!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Ff9acb596-6f3c-4255-8adc-60dd6113d8bc_2048x1644.png)

To run tests with Testcontainers you need a Docker-API compatible container runtime or to install Docker locally. Try stopping your docker engine and it won’t work. However it should not be an issue for most developers, because having a docker runtime in your CI/CD or locally is a very common practice nowadays, you can easily have this environment in Github Actions for example.

When it comes to supported languages, Testcontainers support a big list of popular languages and platforms including Java, .NET, Go, NodeJS, Python, Rust and Haskell, etc.

There is also a growing list of preconfigured implementations (called modules) which you can find [here](https://testcontainers.com/modules/). However, as mentioned earlier you can run any docker image. In Go you could use the following code to provision Redis instead of using a preconfigured module:

```
// Using available module
redisContainer, err := redis.Run(ctx, "redis:latest")

// Or using GenericContainer
req := testcontainers.ContainerRequest{
  Image:        "redis:latest",
  ExposedPorts: []string{"6379/tcp"},
  WaitingFor:   wait.ForLog("Ready to accept connections"),
}
redisC, err := testcontainers.GenericContainer(ctx, testcontainers.GenericContainerRequest{
  ContainerRequest: req,
  Started:          true,
})
```

While the development and maintenance of integration tests require significant effort, they are crucial part of SDLC ensuring that components, subsystems or applications work well together combined as a group.

Using Testcontainers, we can simplify the provisioning and de-provisioning of throwaway dependencies for testing, making the test runs fully isolated and more predicatble.

- [Github repository](https://github.com/plutov/packagemain/blob/master/testcontainers-demo)
    
- [Testcontainers](https://testcontainers.com/)