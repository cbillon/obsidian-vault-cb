---
link: https://www.kdnuggets.com/7-advanced-sql-techniques-for-data-manipulation-in-data-science
site: KDnuggets
excerpt: Can SQL be used for advanced data manipulation in data science? It sure can with these seven techniques.
twitter: https://twitter.com/@kdnuggets
slurped: 2024-11-22T13:13
title: 7 Advanced SQL Techniques for Data Manipulation in Data Science
tags:
---

SQL is one of the most used tools in data science for data manipulation. No wonder since SQL was created to query and manipulate data in databases. Because of its purpose, it offers a wide range of techniques for data manipulation.

Some belong to a more advanced spectrum and can be very useful in data science.

![Advanced SQL Techniques for Data Manipulation in Data Science](https://www.kdnuggets.com/wp-content/uploads/Rosidi_7_Advanced_SQL_Techniques_for_Data_Manipulation_in_Data_Science_2-scaled.webp)

## **1. Subqueries and Correlated Subqueries**

An [SQL subquery](https://www.w3resource.com/sql/subqueries/understanding-sql-subqueries.php) is a query within another (main) query. They are typically used in the SELECT statement but can also be used in INSERT, UPDATE, and DELETE. Subqueries can be used in FROM, WHERE, HAVING, and JOIN clauses.

A correlated subquery is a special type of subquery that depends on the results of the main query.

When to Use Them: 

- Filtering
- Calculations
- Restructuring data
- Row-by-row data evaluations (correlated subqueries)
- Dynamic lookups (correlated subqueries)

## **2. Common Table Expressions (CTE)**

[CTE](https://learnsql.com/blog/what-is-common-table-expression/) is a temporary result set that can be referenced in SELECT, INSERT, UPDATE, or DELETE statements. In many cases, they are often nothing but neatly written subqueries. However, one significant difference between them is that CTEs are reusable in the main query, unlike subqueries.

When to Use Them:

- Recursive queries
- When the same ‘subquery’ result needs to be used across multiple steps
- Breaking down complex queries into smaller logical components

## **3. Recursive Queries**

In SQL, [recursive queries](https://builtin.com/data-science/recursive-sql) are written in recursive CTEs. A recursive query references itself, making it perfect for querying hierarchical and graph data structures.

When to Use Them:

- Finding descendants in a [hierarchical structure](https://learn.microsoft.com/en-us/sql/relational-databases/hierarchical-data-sql-server?view=sql-server-ver16) (e.g., organizational chart or a product category tree)
- Calculating hierarchical paths (e.g., finding the reporting chain from an employee to the CEO)
- Generating sequential data
- [Traversing graphs](https://inviqa.com/blog/storing-graphs-database-sql-meets-social-network) (e.g., finding all possible routes between locations in a transportation network)
- Nested totals (e.g., sales per product, product category, and grand total)

## **4. Window Functions**

[Window functions](https://www.stratascratch.com/blog/the-ultimate-guide-to-sql-window-functions/?utm_source=blog&utm_medium=click&utm_campaign=kdn+techniques+for+data+manipulation+in+ds) allow you to perform calculations across rows related to the current row, with the need to aggregate data.

When to Use Them:

- Ranking
- [Moving averages](https://www.sqlshack.com/calculate-moving-average-in-sql-power-bi-and-ms-excel/)
- [Cumulative (or running) totals](https://five.co/blog/sql-cumulative-sum/)
- Period-to-period comparison (e.g., month-to-month sales)
- Percentile calculations
- Opening and closing prices
- Time series analysis

## **5. Set Operators**

Set operators are used to combine the results of two or more SELECT queries into a single output. They are:

- UNION: Combines the queries’ outputs and removes duplicate rows.
- UNION ALL: Combines the queries’ outputs, including duplicates.
- INTERSECT: Returns only rows present in all queries’ outputs.
- EXCEPT (or MINUS): Returns only rows appearing in the first query’s output but not others.

When to Use Them:

- Comparing datasets
- Filtering results across multiple tables

## **6. GROUP BY Extensions**

GROUP BY is a standard SQL clause for data aggregation. However, you can perform more complex groupings using these [GROUP BY extensions](https://www.mssqltips.com/sqlservertip/6315/group-by-in-sql-server-with-cube-rollup-and-grouping-sets-examples/):

- GROUPING SETS: For multiple groupings in a single GROUP BY.
- ROLLUP: For creating subtotals and grand totals in a single query.
- CUBE: For creating all possible combinations of aggregations for columns in GROUP BY, including subtotals for each level and a grand total.

When to Use Them:

- Hierarchical summaries
- Multi-dimensional analysis
- Generating various aggregate views

## **7. String Functions**

Complex data often includes textual fields that require manipulation to extract insights or prepare for analysis. In SQL, many [string functions](https://www.stratascratch.com/blog/practicing-string-manipulation-in-sql/?utm_source=blog&utm_medium=click&utm_campaign=kdn+techniques+for+data+manipulation+in+ds) help you with that, such as:

- TRIM(): Removes leading and/or trailing characters from a string.
- REPLACE(): Substitutes all occurrences of a substring within a string with a new substring.
- SUBSTRING(): Extracts a part of a string from a specified starting position and length.
- LIKE: Allows pattern matching within strings using wildcard characters, such as % and _.
- PATINDEX(): Returns the starting position of a pattern within a string or zero if the pattern is not found.
- RegEx: Provides a way to search, match, and manipulate strings based on complex patterns.
- SPLIT_PART(): Splits a string by a delimiter and returns a particular segment based on an index.
- STRING_AGG(): Concatenates values from multiple rows into a single string, separated by a specified delimiter.

When to Use Them:

- Data cleaning
- Pattern matching
- Text parsing and tokenization
- Text aggregation

[Ressources you tubes](https://www.youtube.com/playlist?list=PL1XF9qjV8kH12PTd1WfsKeUQU6e83ldfc)
