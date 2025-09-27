---
link: https://wendelladriel.com/blog/understanding-value-objects-in-php
site: Wendell Adriel
excerpt: Value Objects is a fantastic concept that we can use to improve our applications. They are small objects, such as Money, DateRange, Email, or Age, that we utilize on complex applications. They are key elements in creating efficient, understandable, and maintainable code.
twitter: https://twitter.com/wendell_adriel
tags:
  - php
  - objects
slurped: 2025-09-13T19:48
title: Understanding Value Objects in PHP
---

[

## Introduction

**Value Objects** is a fantastic concept that we can use to improve our applications. They are small objects, such as Money, DateRange, Email, or Age, that we utilize on complex applications. They are key elements in creating efficient, understandable, and maintainable code.

**Value Objects** are characteristically **immutable** and are **measured by state, not identity**. Contrary to **Entity Objects** that have a distinct identity, **Value Objects** do not possess any unique identifier. Instead, they are **entirely defined by their value**, meaning that two **Value Objects** are said to be equal if their values match, regardless of whether they are separate instances.

For example, take two instances of a `Money` **Value Object**, one created to represent `$10`, and another separately instantiated also to represent `$10`. Although they are two distinct instances, within the application, we consider them equal because they represent the same value.

**Value Objects** make your code more explicit, readable, and less error-prone. They encapsulate related data together into logical and **meaningful concepts and context**, making them **easier to manage, test, and debug**. Understanding and correctly implementing **Value Objects** can significantly simplify complex business logic and improve the quality of our codebase.

[

## Why to use Value Objects

They offer a range of benefits that directly impact code quality and application maintainability. Let's discuss some of the reasons why we should integrate **Value Objects** into our applications and how it can lead to improved code quality.

Firstly, **Value Objects** lead to self-validating code. As **Value Objects** encapsulate a specific piece of business logic, they are responsible for ensuring that they are always in a valid state. For instance, an `Email` **Value Object** will not be created if the given email is not formatted correctly. This makes your code less prone to errors as it guards against invalid states.

Secondly, **Value Objects** reduces code duplicity. Any duplication within your codebase increases its complexity and makes it harder to maintain. By using **Value Objects**, you avoid repeating code across your codebase. Instead, you encapsulate a specific behavior into a **Value Object** that can be utilized across multiple parts of the application.

One more considerable advantage of utilizing **Value Objects** is in their capacity to improve code expressiveness and readability. **Value Objects** wrap related values into one meaningful construct, making your code more understandable for others (and for you when you revisit your own code months later!). For instance, a `DateRange` object comprising a start date and an end date is far more intuitive and cleaner than handling two separate date variables.

**Value Objects** also strengthen your application’s domain model by ensuring the integrity of data through its lifecycle. Since **Value Objects** are **immutable** (you could create them to not be, but you should make them immutable), they significantly reduce the chance of data inconsistencies.

[

## Value Objects (VOs) vs Data Transfer Objects (DTOs)

Some people may get confused when we talk about **Value Objects** and **Data Transfer Objects** and sometimes even mix their concepts, but they serve different purposes and have distinct characteristics.

**Value Objects (VOs)** are objects that represent a specific value rather than an entity. They are typically used to **encapsulate** a single piece of data or a combination of related data, such as a date, color, or currency amount. They are designed to be used as **immutable** data containers and are often used to **enforce business rules or validate data**.

On the other hand, **Data Transfer Objects (DTOs)** are objects that are used to transfer data between different layers or modules of an application. They carry data from one part of the system to another and are primarily used for communication and serialization purposes, you can also add validation rules to them but they usually never have any logic. They often represent a subset of data from an entity or multiple entities, and they can include additional fields or transformations to meet specific requirements of the communication channel or client.

[

## Starting to use Value Objects

I'll demonstrate how you can start using **Value Objects** in your applications by showing a simple example of how we could encapsulate the logic for handling with `Money` values in an application.

Imagine that we have two Entities in our application

```
final class Product
{
    public function __construct(
        private string $name,
        private float $price,
        private string $currency,
    ) {}
}

final class Subscription
{
    public function __construct(
        private string $name,
        private float $price,
        private string $currency,
    ) {}
}
```

You can see that we have two things in common for both Entities: `price` and `currency` and that both values have a link between them. In this scenario we have two options, we could duplicate code on how to handle these value for both entities or create a Trait or some helper class. That could help, but it's not the best solution. Here's where the **Value Objects** can help us, we can create a `Money` **Value Object** that we can reuse in our whole codebase and that will wrap all the business logic and all the context for handling money values.

```
final readonly class Money
{
    private float $amount;
    private string $currency;
	
    public function __construct(float $amount, string $currency)
    {
        $this->ensureAmountIsPositive($amount);

        $this->amount = $amount;
        $this->currency = $currency;
    }

    public function getAmount(): float
    {
        return $this->amount;
    }

    public function getCurrency(): string
    {
        return $this->currency;
    }

    public function add(Money $other): Money
    {
        $this->assertSameCurrency($other);

        return new Money($this->amount + $other->getAmount(), $this->currency);
    }

    public function subtract(Money $other): Money
    {
        $this->assertSameCurrency($other);

        return new Money($this->amount - $other->getAmount(), $this->currency);
    }

    public function isSameCurrency(Money $other): bool
    {
        return $this->currency === $other->currency;
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->getAmount() && $this->isSameCurrency($other);
    }

    public function greaterThan(Money $other): bool
    {
        $this->assertSameCurrency($other);

        return $this->amount > $other->getAmount();
    }

    public function lessThan(Money $other): bool
    {
        $this->assertSameCurrency($other);

        return $this->amount < $other->getAmount();
    }

    private function assertSameCurrency(Money $other): void
    {
        if (! $this->isSameCurrency($other)) {
            throw new InvalidArgumentException("Currencies should be the same.");
        }
    }

    private function ensureAmountIsPositive(float $amount): void
    {
        if ($amount < 0) {
            throw new InvalidArgumentException("Amount should be positive.");
        }
    }
}
```

As you can see in the example above, now all the business logic needed for dealing with money values are wrapped into the `Money` **Value Object** that you can reuse across all your application without the need to duplicate any code. We can even further improve the `Money` **Value Object** by creating another **Value Object** for currencies.

```
final readonly class Currency
{
    public function __construct(
        private string $code,
        private string $name,
    ) {}

    public function getCode(): string
    {
        return $this->code;
    }
	
    public function getName(): string
    {
        return $this->name;
    }

    public function equals(Currency $other): bool
    {
        return $this->code === $other->getCode();
    }
}
```

Then we can update our `Money` **Value Object** to use our `Currency` **Value Object**.

```
final readonly class Money
{
    private float $amount;
    private Currency $currency;
	
    public function __construct(float $amount, Currency $currency)
    {
        $this->ensureAmountIsPositive($amount);

        $this->amount = $amount;
        $this->currency = $currency;
    }

    public function getAmount(): float
    {
        return $this->amount;
    }

    public function getCurrency(): Currency
    {
        return $this->currency;
    }

    public function add(Money $other): Money
    {
        $this->assertSameCurrency($other);

        return new Money($this->amount + $other->getAmount(), $this->currency);
    }

    public function subtract(Money $other): Money
    {
        $this->assertSameCurrency($other);

        return new Money($this->amount - $other->getAmount(), $this->currency);
    }

    public function isSameCurrency(Money $other): bool
    {
        return $this->currency->equals($other->getCurrency());
    }

    public function equals(Money $other): bool
    {
        return $this->amount === $other->getAmount() && $this->isSameCurrency($other);
    }

    public function greaterThan(Money $other): bool
    {
        $this->assertSameCurrency($other);

        return $this->amount > $other->getAmount();
    }

    public function lessThan(Money $other): bool
    {
        $this->assertSameCurrency($other);

        return $this->amount < $other->getAmount();
    }

    private function assertSameCurrency(Money $other): void
    {
        if (! $this->isSameCurrency($other)) {
            throw new InvalidArgumentException("Currencies should be the same.");
        }
    }

    private function ensureAmountIsPositive(float $amount): void
    {
        if ($amount < 0) {
            throw new InvalidArgumentException("Amount should be positive.");
        }
    }
}
```

And now that we have our `Money` **Value Object** ready to be used, we can update our `Product` and `Subscription` Entities to use it.

```
final class Product
{
    public function __construct(
        private string $name,
        private Money $price,
    ) {}
}

final class Subscription
{
    public function __construct(
        private string $name,
        private Money $price,
    ) {}
}
```

[

## Conclusion

In conclusion, **Value Objects** are powerful tools that by encapsulating related values and behaviors into clean, reusable parts, they significantly enhance code quality, readability, and long-term maintainability of the applications.

The benefits of integrating them into your projects are numerous. From self-validating code that guards against invalid states, promoting code reuse to reduce redundancy and complexity, to improving the expressiveness and intuitiveness of your codebase.

Their **immutability** adds another layer of security, ensuring data consistency throughout the application's lifecycle and they also reinforce the integrity of your domain model, shielding your software from data inconsistencies and errors.

By implementing **Value Objects** in our applications, we can develop applications that are not just built to work, but built to evolve, built to be understood, and built to last.
