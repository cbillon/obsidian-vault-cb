---
link: https://www.kdnuggets.com/analyzing-json-data-with-duckdb-sql
site: KDnuggets
excerpt: Tired of wrangling JSON with scripts and regex? DuckDB lets you run SQL queries on JSON files, making structured and semi-structured data analysis a breeze.
twitter: https://twitter.com/@kdnuggets
slurped: 2025-06-05T15:34
title: Analyzing JSON Data with DuckDB and SQL
tags:
  - duckdb
  - json
---
Parsing JSON files to analyze them can often be a challenge. Whether you’re working with API responses, log files, or application data, analyzing JSON can be time-consuming without the right tools.

With DuckDB, you can query complex JSON files using SQL. So you can analyze JSON data directly without writing complex parsing code or heavyweight database setup.

In this article I'll show you how to use DuckDB to efficiently query and analyze JSON data. We’ll cover:

- Setting up DuckDB on your system
- Loading and querying JSON data
- Working with nested JSON structures
- Handling JSON arrays and complex objects

We'll work with realistic e-commerce data and cover techniques you can immediately apply to your own projects. So let’s get started.

**🔗 [Link to the code on GitHub](https://github.com/balapriyac/data-science-tutorials/tree/main/duckdb-json)**

## Installing and Launching DuckDB

   
DuckDB is lightweight and easy to set up. Let's get it installed and running on your system.

If you’re on a Linux distro and would like to use duckdb from the command line, do the following.

Install DuckDB:

```
$ curl https://install.duckdb.org | sh
```

Add to path:

```
$ export PATH='/home/user/.duckdb/cli/latest':$PATH
```

Launch DuckDB from the command line:

You’ll see:

```
v1.2.2 7c039464e4
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
D
```

You’re all ready to go!

🔗 You can check the [installation docs](https://duckdb.org/docs/installation/) for guidelines for other platforms.

## Working with Sample JSON Data

   
Let's create a practical e-commerce dataset to work with. This JSON structure includes orders, customer information, and product details - similar to what you might get from an actual e-commerce API.

**📁 The sample JSON data is in the [ecommerce_data.json](https://github.com/balapriyac/data-science-tutorials/blob/main/duckdb-json/ecommerce_data.json) file.**

## Loading and Querying JSON Data

   
Now let's load the JSON data into DuckDB and run some basic queries.

#### Loading JSON Data

Connect to DuckDB and run:

```
-- Create a table from the JSON file
CREATE TABLE ecommerce AS 
SELECT * FROM read_json_auto('ecommerce_data.json');
```

This reads the JSON file and infers the schema. The read_json_auto function can as well detect nested structures and arrays.

Let's confirm the data loaded correctly:

```
-- View the data
SELECT * FROM ecommerce;
```

You should see your entire JSON data as a structured table.

```
┌──────────┬───┬──────────────────────┬──────────────────────┐
│ order_id │ … │        items         │       payment        │
│ varchar  │   │ struct(product_id …  │ struct("method" va…  │
├──────────┼───┼──────────────────────┼──────────────────────┤
│ ORD-1001 │ … │ [{'product_id': PR…  │ {'method': credit_…  │
│ ORD-1002 │ … │ [{'product_id': PR…  │ {'method': paypal,…  │
├──────────┴───┴──────────────────────┴──────────────────────┤
│ 2 rows                                 5 columns (3 shown) │
└────────────────────────────────────────────────────────────┘
```

#### Basic Queries on JSON Data

Let's start with some simple queries:

```
-- Count the number of orders
SELECT COUNT(*) AS order_count FROM ecommerce;
```

This gives us the total number of orders in our dataset.

```
┌─────────────┐
│ order_count │
│    int64    │
├─────────────┤
│      2      │
└─────────────┘
```

Here, we're using the ->>'name' syntax to extract the name field from the customer object. The ->> operator returns the value as text, while -> returns it as JSON.

```
-- Get order IDs and customer names
SELECT 
    order_id,
    customer->>'name' AS customer_name
FROM ecommerce;
```

Output:

```
┌──────────┬───────────────┐
│ order_id │ customer_name │
│ varchar  │    varchar    │
├──────────┼───────────────┤
│ ORD-1001 │ Alex Johnson  │
│ ORD-1002 │ Sarah Miller  │
└──────────┴───────────────┘
```

## Working with Nested JSON Structures

   
One of the challenges with JSON is working with nested objects. Let's extract information from nested JSON:

```
-- Extract customer address information
SELECT 
    order_id,
    customer->>'name' AS customer_name,
    customer->'address'->>'city' AS city,
    customer->'address'->>'state' AS state
FROM ecommerce;
```

Output:

```
┌──────────┬───────────────┬─────────┬─────────┐
│ order_id │ customer_name │  city   │  state  │
│ varchar  │    varchar    │ varchar │ varchar │
├──────────┼───────────────┼─────────┼─────────┤
│ ORD-1001 │ Alex Johnson  │ Boston  │ MA      │
│ ORD-1002 │ Sarah Miller  │ Seattle │ WA      │
└──────────┴───────────────┴─────────┴─────────┘
```

This query navigates through the nested structure to extract city and state from the address object. Notice how we chain the arrow operators to go deeper into the JSON structure.

We can also filter based on nested fields:

```
-- Find orders from customers in Seattle
SELECT 
    order_id,
    customer->>'name' AS customer_name
FROM ecommerce
WHERE customer->'address'->>'city' = 'Seattle';
```

This query filters orders to only those from customers in Seattle, showing how we can use nested fields in WHERE clauses.

```
┌──────────┬───────────────┐
│ order_id │ customer_name │
│ varchar  │    varchar    │
├──────────┼───────────────┤
│ ORD-1002 │ Sarah Miller  │
└──────────┴───────────────┘
```

Let's extract payment information:

```
-- Get payment details
SELECT 
    order_id,
    payment->>'method' AS payment_method,
    CAST(payment->>'total' AS DECIMAL) AS total_amount
FROM ecommerce;
```

Output:

```
┌──────────┬────────────────┬───────────────┐
│ order_id │ payment_method │ total_amount  │
│ varchar  │    varchar     │ decimal(18,3) │
├──────────┼────────────────┼───────────────┤
│ ORD-1001 │ credit_card    │       179.970 │
│ ORD-1002 │ paypal         │       137.960 │
└──────────┴────────────────┴───────────────┘
```

Note that we're casting the total to a decimal so we can perform numeric operations on it. Without the cast, it would be treated as text.

## Handling Arrays and Complex Objects

   
JSON arrays require special handling. Let's look at the items in each order:

```
-- Unnest the items array into separate rows
SELECT 
    order_id,
    customer->>'name' AS customer_name,
    unnest(items) AS item
FROM ecommerce;
```

Output:

```
┌──────────┬───────────────┬───────────────────────────────────────────────────┐
│ order_id │ customer_name │                       item                        │
│ varchar  │    varchar    │ struct(product_id varchar, "name" varchar, cate…  │
├──────────┼───────────────┼───────────────────────────────────────────────────┤
│ ORD-1001 │ Alex Johnson  │ {'product_id': PROD-501, 'name': Wireless Headp…  │
│ ORD-1001 │ Alex Johnson  │ {'product_id': PROD-245, 'name': Smartphone Cas…  │
│ ORD-1002 │ Sarah Miller  │ {'product_id': PROD-103, 'name': Coffee Maker, …  │
│ ORD-1002 │ Sarah Miller  │ {'product_id': PROD-107, 'name': Coffee Beans P…  │
└──────────┴───────────────┴───────────────────────────────────────────────────┘
```

The `unnest()` function is important here. It takes a JSON array and returns one row for each element. This transforms our array of items into individual rows, which is much easier to work with in SQL.

Now let's extract specific fields from each item:

```
-- Get specific item details
SELECT 
    order_id,
    customer->>'name' AS customer_name,
    item->>'name' AS product_name,
    item->>'category' AS category,
    CAST(item->>'price' AS DECIMAL) AS price,
    CAST(item->>'quantity' AS INTEGER) AS quantity
FROM (
    SELECT 
        order_id,
        customer,
        unnest(items) AS item
    FROM ecommerce
) AS unnested_items;
```

Output:

```
┌──────────┬───────────────┬───┬───────────────┬──────────┐
│ order_id │ customer_name │ … │     price     │ quantity │
│ varchar  │    varchar    │   │ decimal(18,3) │  int32   │
├──────────┼───────────────┼───┼───────────────┼──────────┤
│ ORD-1001 │ Alex Johnson  │ … │       129.990 │        1 │
│ ORD-1001 │ Alex Johnson  │ … │        24.990 │        2 │
│ ORD-1002 │ Sarah Miller  │ … │        89.990 │        1 │
│ ORD-1002 │ Sarah Miller  │ … │        15.990 │        3 │
├──────────┴───────────────┴───┴───────────────┴──────────┤
│ 4 rows                              6 columns (4 shown) │
└─────────────────────────────────────────────────────────┘
```

This query first unnests the items array, then extracts specific fields from each item. The subquery approach is important - we first create a dataset with one row per item, then extract the fields we want in the outer query.

Let's perform some analytics on the data:

```
-- Calculate total value of each order
SELECT 
    order_id,
    customer->>'name' AS customer_name,
    CAST(payment->>'total' AS DECIMAL) AS order_total,
    json_array_length(items) AS item_count
FROM ecommerce;
```

Output:

```
┌──────────┬───────────────┬───────────────┬────────────┐
│ order_id │ customer_name │  order_total  │ item_count │
│ varchar  │    varchar    │ decimal(18,3) │   uint64   │
├──────────┼───────────────┼───────────────┼────────────┤
│ ORD-1001 │ Alex Johnson  │       179.970 │          2 │
│ ORD-1002 │ Sarah Miller  │       137.960 │          2 │
└──────────┴───────────────┴───────────────┴────────────┘
```

Here, `json_array_length()` gives us the number of items in each order, and we're also extracting the total order value.

```
-- Calculate average price by product category
SELECT 
    item->>'category' AS category,
    AVG(CAST(item->>'price' AS DECIMAL)) AS avg_price
FROM (
    SELECT unnest(items) AS item
    FROM ecommerce
) AS unnested_items
GROUP BY category
ORDER BY avg_price DESC;
```

Output:

```
┌─────────────────┬───────────┐
│    category     │ avg_price │
│     varchar     │  double   │
├─────────────────┼───────────┤
│ Electronics     │    129.99 │
│ Kitchen         │     89.99 │
│ Accessories     │     24.99 │
│ Food & Beverage │     15.99 │
└─────────────────┴───────────┘
```

This query calculates the average price for each product category. We first unnest the items, then group by category and calculate the average price.

## Wrapping Up

   
You've now learned the essential techniques for analyzing JSON data with DuckDB. What we’ve covered will allow you to tackle most JSON analysis tasks you'll encounter. DuckDB lets you use familiar SQL syntax with specialized JSON functions, giving you the best of both worlds.

Next time you're faced with a complex JSON dataset, I hope you can skip the parsing headaches and jump straight into analysis.

**[](https://twitter.com/balawc27)**[Bala Priya C](https://www.kdnuggets.com/wp-content/uploads/bala-priya-author-image-update-230821.jpg)**** is a developer and technical writer from India. She likes working at the intersection of math, programming, data science, and content creation. Her areas of interest and expertise include DevOps, data science, and natural language processing. She enjoys reading, writing, coding, and coffee! Currently, she's working on learning and sharing her knowledge with the developer community by authoring tutorials, how-to guides, opinion pieces, and more. Bala also creates engaging resource overviews and coding tutorials.