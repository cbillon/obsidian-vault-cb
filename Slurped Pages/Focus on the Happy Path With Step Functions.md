---
link: https://preslav.me/2023/06/14/golang-focus-on-the-happy-path-with-step-functions/
byline: Preslav Rachev
site: Preslav Rachev
date: 2023-06-14T02:00
excerpt: A simple pattern that will help you reduce error handling, while keeping your Go code simple and idiomatic.
slurped: 2026-01-26T19:43
title: Focus on the Happy Path With Step Functions
tags:
  - golang
---

1. [Preslav Rachev /](app://obsidian.md/)[Programming](app://obsidian.md/categories/programming) /
2. [My Writings /](app://obsidian.md/posts/)[Programming](app://obsidian.md/categories/programming) /
3. [Focus on the Happy Path With Step Functions /](app://obsidian.md/2023/06/14/golang-focus-on-the-happy-path-with-step-functions/)[Programming](app://obsidian.md/categories/programming) /

Before we move on, there are three disclaimers I want to start with:

- These are different from the [step functions you might be familiar with from AWS](https://aws.amazon.com/step-functions/). The name helps explain the idea, but everywhere else in this post, I would mean plain old Go code (functions, structs, errors) whenever I talk about step functions.
- You won’t see this pattern in virtually 99% of the Go code out there. You won’t see it in most of my code either because I think it is unnecessary for small or mid-sized functions. However, it can really make a difference in that remaining 1%, which is why I thought it would be worth sharing.
- This pattern is essentially the non-generics version of the [generic pipeline](https://preslav.me/2021/09/04/generic-golang-pipelines/) I once wrote about. By taking generics out of the way, I hope to make it more appealing to a larger group of Go applications.

## The _What_ and the _How_ [#](#the-what-and-the-how)

There are good examples of this across popular modern software. Take Kubernetes or frameworks like Flutter, SwiftUI, React, or any piece of software that distinguishes between logic and configuration. All of those have a _declarative_ part (whether a YAML manifest, a JSX file, or a SwiftUI/Flutter view) that dictates _what_ needs to happen. In crucial points, the declarative part delegates to the _imperative_ part to do the actual _how_ (spin containers, draw buttons on the screen, handle a click, etc.). Keeping those separate makes it easier to extend the process without dealing with the technical details all the time.

OK, things became too abstract. Back to Go. As a language, Go falls in the category of imperative languages - it thrives in getting the _how_ done, but the _what_ sometimes remains an afterthought - an exercise for the reader to piece together by reading the code.

A good case in point is the explicit error handling. For the most part, handling errors in Go is less frightening than many newcomers would think. Once you get used to writing Go, error checks blend in and, as I alluded a [few weeks ago](https://preslav.me/2023/04/14/golang-error-handling-is-a-form-of-storytelling/), become a form of natural _documentation_ about what could possibly go wrong.

The problems begin with longer functions or methods. Suppose you have the following piece of business logic:

```
var customer Customer
row := db.QueryRow("select * from customers where id = $1", customerID)
if row := row.Err(); err != nil {
	// handle the error
}

err = row.Scan(&customer.id, &customer.name)
if err != nil {
	// handle the error
}

// Good! Now fetch a product
var product Product
// Omiting the rest of the code for brevity ...
// ...

// Perfect! No create an order and store it
order := {
	CustomerID: customerID,
	ProductID: productID,
	ShippingAddress: customer.Address
	Quantity: 10
}
// Omiting the rest of the code for brevity ...
// ...
```

You don’t need a degree in Computer Science to see that with the added code, seeing the actual steps (the _what_) - fetching a customer and product and storing an order, becomes obscured by the how - dealing with row scanning and handling errors.

Naturally, one bulletproof way to deal with this is to abstract the repetitive code in functions. Thus, the code above can turn into:

```
customer, err := store.FindCustomerByID(customerID)
if err != nil {
	// handle the error
}

product, err := store.FindProductByID(productID)
if err != nil {
	// handle the error
}

order := {
	CustomerID: customerID,
	ProductID: productID,
	ShippingAddress: customer.Address
	Quantity: 10
}

err := store.CreateOrder(&order)
if err != nil {
	// handle the error
}
```

Much better, but we could go even further. What if we could squash all those error checks into a single one that happens at the very end? Our code would then turn into something like this:

```
steps := CreateOrderSteps {
	customerID: customerID
	productID: productID
	// an initialized tmp attribute to use as intermediate storage
}

err := StepFunc{}.
		Next(steps.FindCustomer).
		Next(steps.FindProduct).
		Next(steps.SaveOrder).
		Do()
if err != nil {
	// handle the error
}
```

below is a sample implementation of one of the “steps”. The others are analogous.

```
func (s *steps) FindCustomer() error {
var customer Customer
row := db.QueryRow("select * from customers where id = $1", customerID)
if row := row.Err(); err != nil {
return err
}

    err = row.Scan(&customer.id, &customer.name)
    if err != nil {
    	return err
    }
    
    s.tmp.customer = &customer
    return nil
}
```

## Step Functions [#](#step-functions)

A step function is nothing more than a simple struct that holds a reference to a list of funcs (the steps), and an error:

```
type StepFunc struct {
	funcs []func() error
}
```

The `Next` method is equally simple:

```
func (sf *StepFunc) Next(f func() error) *StepFunc {
	sf.funcs = append(sf.funcs, f)

	// return the step func to allow for chaining
	return sf
}
```

Note that the `Next` method does not execute the functions but simply appends their references to the `funcs`collection. To keep things as generic as possible (without using generic type parameters), I found the simplest possible function that could represent a step to be `func() error`. In case you are already asking yourselves how the function will get the necessary inputs it needs, that’s what the auxiliary `steps`struct was for. The struct will serve as a shared scratch space that each step writes its results to so that it can be read from the next one. Not exactly pretty and, most definitely, not thread-safe, but it should be sufficient for most simple use cases.

The last missing piece of our step function is a way to execute our steps in order, stopping early in case of an error. This is what the `Do`method would do:

```
func (p *StepFunc) Do() error {
	for _, f := range p.funcs {
		if err := f(); err != nil {
			// stop the chain prematurely
			return err
		}
	}
	return nil
}
```

There you have it - a way to semi-declaratively list the steps that you want to happen, focusing on the happy path of execution without undermining the power of explicit error handling.

I would leave the case with asynchronous execution as an exercise for the reader. We can look at the idea in a follow-up blog post if there is genuine interest.

## You may also find these interesting

[![](app://obsidian.md/scratchpad/2023/12/why-golang-over-rust-java-python/golang-vs-other-languages_huf45e4001dab1e27992c94ee24b2fe8ae_33827_1320x0_resize_q75_box.jpg)](app://obsidian.md/scratchpad/2023/12/why-golang-over-rust-java-python/)

[![](app://obsidian.md/2023/12/15/golang-interfaces-are-not-meant-for-that/interfaces_hu1243e462c13e42d19d3178691fc0ea19_381980_1320x0_resize_q75_box.jpg)](app://obsidian.md/2023/12/15/golang-interfaces-are-not-meant-for-that/)

[![](app://obsidian.md/2023/11/27/python-is-easy-golang-is-simple-simple-is-not-easy/cover_huf1433fb1d8c7ca998d42f11cbcd49a88_386468_1320x0_resize_q75_box.jpg)](app://obsidian.md/2023/11/27/python-is-easy-golang-is-simple-simple-is-not-easy/)

Code 
```
package main

import (
	"fmt"
)

type steps struct {
	dayOfTheWeek string
	greeting     string
	tasks        []string
	funcs        []stepFunc
}

type stepFunc func() error

func (s *steps) Next(f stepFunc) *steps {
	s.funcs = append(s.funcs, f)

	// return the step func to allow for chaining
	return s
}
func (s *steps) Do() error {
	for _, f := range s.funcs {
		if err := f(); err != nil {
			// stop the chain prematurely
			return err
		}
	}
	return nil
}

func (s *steps) startTheDay() error {
	// chekc if valid day of the week
	if s.dayOfTheWeek == "" {
		return fmt.Errorf("day of the week is not set")
	}
	fmt.Println("Hello " + s.dayOfTheWeek)
	return nil
}

func (s *steps) doTasks() error {
	fmt.Println("Today's tasks are:")
	for _, task := range s.tasks {
		fmt.Println(task)
	}
	return nil
}

func main() {
	// create a new steps struct
	s := &steps{
		dayOfTheWeek: "Monday",
		greeting:     "Hello",
		tasks: []string{
			"Run",
			"Swim",
			"Cycle",
		},
	}

	// chain the steps together and execute them
	err := s.Next(s.startTheDay).Next(s.doTasks).Do()
	if err != nil {
		fmt.Println(err)
	}
}
```