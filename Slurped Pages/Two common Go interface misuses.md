---
link: https://konradreiche.com/blog/two-common-go-interface-misuses/
byline: Konrad Reiche
excerpt: |-
  In Go, interfaces are often misused in two ways. First, interfaces are introduced prematurely by following object-oriented patterns from languages like Java. While well-intentioned, this approach adds unnecessary complexity early in the development process.
  Second, interfaces are created solely to support testing, a practice common in languages like Python that also relies heavily on mocking dependencies. While this might unblock your productivity in the short term, it weakens the expressiveness of your types and ultimately reduces code readability in the long run. This post will focus on these two issues to illustrate their impact.
slurped: 2026-06-21T10:16
title: Two common Go interface misuses
tags:
  - go
---

Oct 18, 2024

In Go, interfaces are often misused in two ways. First, interfaces are introduced prematurely by following object-oriented patterns from languages like Java. While well-intentioned, this approach adds unnecessary complexity early in the development process.

Second, interfaces are created solely to support testing, a practice common in languages like Python that also relies heavily on mocking dependencies. While this might unblock your productivity in the short term, it weakens the expressiveness of your types and ultimately reduces code readability in the long run. This post will focus on these two issues to illustrate their impact.

## Premature Abstraction

The first common pitfall is the premature introduction of abstraction. In Go, however, we emphasize writing easy-to-read code rather than designing complex type hierarchies. Suppose you want to implement an in-memory cache with a Least-Frequently Used (LFU) policy. Following an abstraction-first mindset, you might start by defining an interface and a concrete type like this:

```
// BAD: Prematurely introducing an interface for abstraction
type Cache[T any] interface {
	Get(ctx context.Context, key string) (*T, error)
	Set(ctx context.Context, key string, value T) error
}

type LFU[T any] struct {
	data map[string]T
}
```

You might think this is a good idea because you anticipate having different implementations in the future, such as caches using different policies, and you want to showcase your ability to build for future flexibility from the start. This approach, however, raises two important questions you should ask yourself:

**1. Will you need multiple implementations?**

If not, introducing an interface means you’ll spend time writing code you don’t need. Instead, strive to implement features when you genuinely require them. Moreover, extra types require developers to understand and manage more identifiers and their meanings, which increases cognitive load. Remember, you always have the option to add an interface later on; there’s no need to design your code starting with the interface.

**2. Will all implementations share identical method signatures?**

If you later decide that a different cache implementation needs to deviate from the initial interface, you’ll have to update the interface and all its implementations. This problem is exacerbated when using package-local (unexported) interfaces, as you might update multiple interfaces scattered across your codebase. This increases maintenance effort and introduces tighter coupling, defeating the purpose of decoupling your code.

### Loss of Type Information

A more severe issue with premature abstraction is the loss of type information. Consider the following code snippets from a service that checks a user’s eligibility for a specific product using our cache:

```
// BAD: Using an interface when a concrete type would suffice
type EligibilityService struct {
    catalogCache cache.Cache[model.Product]
    catalog      *product.Catalog
}
```

In this example, it’s unclear which specific cache implementation is used. To find out, we’d have to trace back to where the cache is initialized since elsewhere, we’re dealing with an interface that provides little information. In contrast, using a concrete type offers more clarity, making it evident that an LFU cache is in use and allowing us to jump directly to its code:

```
type EligibilityService struct {
    catalogCache *cache.LFU[model.Product]
    catalog      *product.Catalog
}
```

Some might argue that abstraction lets us delegate implementation details to someone else and that, as code owners, we shouldn’t worry about them. But is that the case? The cache still needs to be integrated, and its integration involves application logic—which may require us to know certain aspects of the implementation. By using an interface, we’re diluting the available type information.

### Fallacy of Flexibility

Another justification for using interfaces is the perceived flexibility to switch out implementations. But how often do we make such switches? For instance, in the example of the cache, there might be an initial phase where we compare LFU with other policies like LRU. However, once that’s settled, does the interface’s flexibility offer any real benefit? Replacing a concrete type is just as straightforward as replacing an interface type. Moreover, using concrete types allows you to add new methods and reference them immediately without retrofitting the interface—saving time and reducing the number of types in your codebase.

### Interface Pollution

You might think this concern is overstated, but the problem becomes clear as more interfaces are introduced, sometimes called “interface pollution.” While it may be acceptable with a single instance, the accumulation of interfaces over time will clutter the codebase and reduce overall clarity and maintainability.

## Mocks for Testing

The second common misuse of interfaces in Go is creating them solely to facilitate testing, particularly for mocking dependencies like data stores or external services. There’s already a tendency to modify code to accommodate testing, even without interfaces. For example, you might change methods to accept `time.Time` or constructors to accept `func() time.Time` to control the current time, or use `rand.Rand` to make non-deterministic code deterministic in tests.

While covering your code with tests is essential, production code should not be modified solely to accommodate testing. Doing so complicates the API, forcing all users of your package to handle additional complexity—even outside testing scenarios.

For more complex dependencies, developers often create package-local (unexported) interfaces and swap out concrete types with these interfaces just to enable testing. While this might make testing easier, it leads to losing or diluting type information and can spread unnecessary interfaces throughout your codebase.

For example, suppose our cache now uses Redis. Although we’ve removed the exported interface in the cache package, we introduce a local interface to make testing feasible:

```
// BAD: An unexported interface adding unnecessary complexity and reducing test coverage
type cache[T any] interface {
    Get(ctx context.Context, key string) (*T, error)
    Set(ctx context.Context, key string, value T) error
}

type EligibilityService struct {
    catalogCache cache[model.Product]
    catalog      *product.Catalog
}
```

This approach might seem better because (a) we don’t have a global interface imported everywhere, reducing coupling, and (b) we’ve limited the scope of the interface to our package. However, we’ve also eliminated type information about which package provides a valid implementation—we can’t even navigate to the source code anymore. Once this practice is established, other packages may adopt this local interface approach, accumulating unnecessary interfaces.

Instead of creating interfaces for testing, we should strive to instantiate the concrete type. Using mocks means you test less code; you’ve essentially removed the actual code and are now testing the correctness of the mock implementation.

While it might seem counterintuitive at first, avoiding interfaces for testing encourages more straightforward and maintainable code. You’ll become more adept at designing testable code without relying on unnecessary abstractions with practice.

Making this process easy isn’t necessarily trivial and deserves a structured approach. The best solution I’ve found is using test packages. You can manage dependencies by organizing package-specific test helpers into a subpackage while providing similar ergonomics. As this topic deserves more detail, I’ll address it in a follow-up post.

## Good Interfaces

So, when are interfaces a good fit? Interfaces are best suited for scenarios with multiple implementations where you want to give users of your package the flexibility to provide their own. Examples include middleware implementations such as gRPC interceptors or Redis hooks. In these cases, libraries offer a set of predefined middleware but also allow users to define their additions.

A helpful mental check is to ask yourself: Is the interface necessary? If you can write the code without using interfaces, it’s likely not needed. If interfaces are the only realistic way to implement a feature, you’re on the right track.