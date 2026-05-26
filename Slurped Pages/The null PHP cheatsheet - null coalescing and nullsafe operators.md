---
link: https://cylab.be/blog/479/the-null-php-cheatsheet-null-coalescing-and-nullsafe-operators
excerpt: This is a small cheatsheet on PHP null coalescing and nullsafe
  operators, which allow to easily handle null objects and values in PHP.
slurped: 2026-05-19T08:20
title: "The null PHP cheatsheet : null coalescing and nullsafe operators"
---

This is a small cheatsheet on PHP null coalescing and nullsafe operators, which allow to easily handle null objects and values in PHP.

## Null coalescing operator[](https://cylab.be/blog/479/the-null-php-cheatsheet-null-coalescing-and-nullsafe-operators#null-coalescing-operator)

The null coalescing operator `??` was introduced in PHP 7 and allows to avoid heavy constructions with `if (isset(...))`. The null coalescing operator **returns the first operand if it is not null, and otherwise the second operand**.

```
$username = $_GET['username'] ?? 'nobody';

// is equivalent to:
$username = isset($_GET['username']) ? $_GET['username'] : 'nobody';
```

## Nullsafe operator[](https://cylab.be/blog/479/the-null-php-cheatsheet-null-coalescing-and-nullsafe-operators#nullsafe-operator)

The nullsafe Operator `?->` was introduced in PHP 8 and provides a concise way to **access object properties or methods only if the object is not null**. Hence this allows to easily avoid errors like `Fatal error: Uncaught Error: Attempt to read property "location" on null`.

```
$location = $user->profile?->location;

// is equivalent to:
if (is_null($user->profile)) {
  $location = null;
} else {
  $location = $user->profile->location;
}
```

This was a short post, but on two powerful features in PHP. The null coalescing operator (`??`) and the nullsafe operator (`?->`) allow developers to write more concise and error-free code, reducing the likelihood of null pointer exceptions and improving the overall maintainability of applications.
