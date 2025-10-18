---
link: https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2025-10-01T02:00
excerpt: The pipe operator (|>) in PHP 8.5 is a powerful addition that allows for a more functional programming style by enabling the chaining of operations clearly and concisely. It takes the result of an expression on its left and passes it as the first argument to the function or method on its right.
slurped: 2025-10-13T11:30
title: Seven Real-World Examples of Using the Pipe Operator in PHP 8.5
tags:
  - php
  - pipe
---

The pipe operator (`|>`) in PHP 8.5 is a powerful addition that allows for a more functional programming style by enabling the chaining of operations clearly and concisely. It takes the result of an expression on its left and passes it as the first argument to the function or method on its right.

```
$value = "hello world";

$result = $value
    |> function3(...)
    |> function2(...)
    |> function1(...);
```

I have written a [dedicated article](https://amitmerchant.com/the-pipe-operator-php-85/) on it, but in this article, I want to share some real-world examples of how you can use the pipe operator in your PHP code to make it cleaner and more readable.

- [String cleanup in APIs](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#string-cleanup-in-apis)
- [Processing uploaded CSV rows](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#processing-uploaded-csv-rows)
- [HTTP response building](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#http-response-building)
- [Search query preparation](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#search-query-preparation)
- [E-commerce cart totals](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#e-commerce-cart-totals)
- [Logging and metrics enrichment](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#logging-and-metrics-enrichment)
- [Image processing chain (e.g., thumbnails)](https://amitmerchant.com/seven-realworld-examples-of-using-the-pipe-operator-in-php-85/#image-processing-chain-eg-thumbnails)

## String cleanup in APIs

The **‘username’** input often needs normalization. Pipes make the steps readable.

```
$raw = "  John   Doe  ";

$username = $raw
    |> trim(...)
    |> strtolower(...)
    |> (fn($x) => preg_replace('/\s+/', '-', $x));

// Output: "john-doe"
```

As you can tell, here each stage is a focused, testable transform: **trim** → **lowercase** → **whitespace-to-dash**.

## Processing uploaded CSV rows

Clean, validate, and map rows into DTOs in one flow.

```
$rows = [
  ['name' => ' Widget A ', 'price' => '12.99', 'sku' => 'W-A'],
  ['name' => 'Widget B',   'price' => 'n/a',   'sku' => 'W-B'],   // invalid price
  ['name' => 'widget c',   'price' => '7.5',   'sku' => 'W-C'],
];

// normalizeRow might trim, cast price to float, unify casing:
// ['name' => 'Widget A', 'price' => 12.99, 'sku' => 'W-A']
// ['name' => 'Widget B', 'price' => null,  'sku' => 'W-B']
// ['name' => 'Widget C', 'price' => 7.5,   'sku' => 'W-C']

// isValidRow returns false if price is null or <= 0

$products = $rows
    |> (fn($xs) => array_map('normalizeRow', $xs))
    |> (fn($xs) => array_filter($xs, 'isValidRow'))
    |> (fn($xs) => array_map(Product::fromArray(...), $xs));

// Output: [Product('Widget A', 12.99, 'W-A'), Product('Widget C', 7.5, 'W-C')]
```

The chain expresses a pipeline: **normalization**, **filtering**, then **construction**.

## HTTP response building

Compose middleware-like transforms to produce a final response body.

```
$data = ['msg' => 'hello', 'n' => 3];

$body = $data
    |> json_encode(...)
    |> (fn($x) => gzencode($x, 6))
    |> base64_encode(...);

// Output: H4sIAAAAAAAAA2WOuwrCMBBE…
```

Each callable is a single-arg step, resulting in a compact, linear build.

## Search query preparation

Sanitize and tokenize a query before passing it to the search engine.

```
$query = "  Hello, WORLD!  ";

$tokens = $query
    |> trim(...)
    |> strtolower(...)
    |> (fn($x) => preg_replace('/[^\w\s]/', '', $x))
    |> (fn($x) => preg_split('/\s+/', $x))
    |> (fn($xs) => array_filter($xs, fn($t) => strlen($t) > 1));

// Output: ['hello', 'world']
```

The pipeline turns messy input into indexed tokens ready for search.

## E-commerce cart totals

Derive totals from items with discounts and tax, staying point-free.

```
// Assume helpers:
// priceAfterDiscount(['price' => 100, 'discount' => 0.10]) -> 90.0
// applyTax(90.0) with 10% tax -> 99.0

$cartItems = [
  ['price' => 100, 'discount' => 0.10],
];

$total = $cartItems
    |> (fn($xs) => array_map('priceAfterDiscount', $xs))
    |> array_sum(...)
    |> (fn($sum) => applyTax($sum)); 
    // applyTax takes one argument

// Output: 99.0
```

Steps are clear: **per-item discount** → **sum** → **tax**; no temporary variables needed.

## Logging and metrics enrichment

This pipeline enriches an event with a timestamp, request ID, and IP, then redacts sensitive fields before logging.

```
$incomingEvent = [
    'user' => 'alice', 
    'email' => '[email protected]'
];

$event = $incomingEvent
    |> (fn($x) => array_merge($x, ['received_at' => 1696195560])) // e.g. time()
    |> (fn($x) => array_merge($x, ['request_id' => '9f2a1c0b7e3d4a56'])) // example random id
    |> (fn($x) => array_merge($x, ['ip' => '203.0.113.42'])) // example IP
    |> (fn($x) => redactSensitive($x)); // e.g. masks email

// Output:
/*
    [
        'user' => 'alice', 
        'email' => 'a***@example.com', 
        'received_at' => 1696195560, 
        'request_id' => '9f2a1c0b7e3d4a56', 
        'ip' => '203.0.113.42'
    ]
*/
```

## Image processing chain (e.g., thumbnails)

In some image processing tasks, you might want to load an image, resize it, apply a watermark, and then optimize it for web delivery. The pipe operator can help express this sequence of operations clearly.

```
$imagePath = '/images/source/beach.jpg';

$thumb = $imagePath
    // GD image resource from beach.jpg
    |> (fn($p) => imagecreatefromjpeg($p))
    // resized to 320×240   
    |> (fn($im) => resizeImage($im, 320, 240))
    // watermark added (e.g., bottom-right)
    |> (fn($im) => applyWatermark($im))
    // returns binary JPEG data or saved file path       
    |> (fn($im) => optimizeJpeg($im, 75));      

// Output: '/images/thumbs/beach_320x240_q75.jpg'
```

Each step is a single-argument transform returning the next value, producing an optimized thumbnail.