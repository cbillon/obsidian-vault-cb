---
link: https://www.kdnuggets.com/a-guide-to-data-analysis-in-python-with-duckdb
site: KDnuggets
excerpt: Learn how to perform data analysis in Python using DuckDB.
twitter: https://twitter.com/@kdnuggets
slurped: 2025-06-05T15:37
title: A Guide to Data Analysis in Python with DuckDB - KDnuggets
tags:
  - duckdb
  - python
---
Are you a developer familiar with SQL and Python? If so, you may want to start using DuckDB—an in-process OLAP database—for data analytics.

SQL is _the_ language for querying databases and is the most important in your data toolbox. So when you switch to Python, you're probably looking at pandas—reading in data from various sources into a dataframe and analyzing it.

But wouldn't it be nice to query pandas dataframes as well as data sources such as CSV and Parquet files using SQL. DuckDB lets you do just that and much more. In this tutorial, we’ll learn how to use DuckDB in Python to analyze data. Let’s get started!

## Setting Up the Environment

To get started, create and activate a virtual environment:

```
$ python3 -m venv v1
$ source v1/bin/activate
```

Next install duckdb:

Because we also generate sample data to work with, we’ll also need NumPy and Pandas:

```
$ pip3 install numpy pandas
```

## Querying Data with DuckDB

With the quick installation out of the way we can move on to do some data analysis.

> **Note**: It’s common to use connections when interacting with databases. You can use `duckdb.connect()` to work with both in-memory databases and persistent storage.

- Using `duckdb.connect()` to connect to an in-memory database that exists only during the session. This is suitable for quick analysis, especially when you don't need to store the results long-term.
- To persist data between sessions and queries, pass a file path to the `connect()` function like so: `duckdb.connect('my_database.db')`.

But we’ll query CSV files and don’t quite need a connection object. So this was just a note to give you an idea when you’re querying databases.

### Generating Sample CSV Files

▶️ You can find the code for this tutorial [on GitHub](https://github.com/balapriyac/data-science-tutorials/tree/main/duckdb).

We'll create a mock sales dataset, a couple of csv files, that include product details, prices, quantities sold, and the regions in which the sales occurred. Running [generate_csv.py](https://github.com/balapriyac/data-science-tutorials/blob/main/duckdb/generate_csv.py) in your project folder to generate two CSV files: sales_data.csv and product_details.csv.

When Working with CSV files in DuckDB, you can read the file into a relation: `duckdb.read_csv(‘your_file.csv’)` and then query it. Or you can work directly with files and query them like so:

```
import duckdb

duckdb.sql("SELECT * FROM 'sales_data.csv' LIMIT 5").df()
```

You can save the results of the query using `df()` as shown in the example.

Let's now run some (actually helpful) SQL queries to analyze the data in the CSV files.

### Example Query 1: Calculate Total Sales by Region

To understand which region generated the most revenue, we can calculate the total sales per region. You can calculate total sales by multiplying the price of each product by the quantity sold and summing it up for each region.

```
# Calculate total sales (Price * Quantity_Sold) per region
query = """
SELECT Region, SUM(Price * Quantity_Sold) as Total_Sales
FROM 'sales_data.csv'
GROUP BY Region
ORDER BY Total_Sales DESC
"""
total_sales = duckdb.sql(query).df()

print("Total sales per region:")
print(total_sales)
```

This query outputs:

```
Total sales per region:
  Region      Total_Sales
0   East	454590.49
1  South	426352.72
2   West	236804.52
3  North	161048.07
```

### Example Query 2: Find the Top 5 Best-Selling Products

Next, we want to identify the top 5 best-selling products by quantity sold. This can give us insight into which products are performing the best across all regions.

```
# Find the top 5 best-selling products by quantity
query = """
SELECT Product_Name, SUM(Quantity_Sold) as Total_Quantity
FROM 'sales_data.csv'
GROUP BY Product_Name
ORDER BY Total_Quantity DESC
LIMIT 5
"""
top_products = duckdb.sql(query).df()

print("Top 5 best-selling products:")
print(top_products)
```

This gives the top 5 products with the highest sales:

```
Top 5 best-selling products:
  Product_Name  Total_Quantity
0   Product_42        	99.0
1   Product_97        	98.0
2   Product_90        	96.0
3   Product_27        	94.0
4   Product_54        	94.0
```

### Example Query 3: Calculate Average Price by Region

We can also calculate the average price of products sold in each region to identify any price differences between regions.

```
# Calculate the average price of products by region
query = """
SELECT Region, AVG(Price) as Average_Price
FROM 'sales_data.csv'
GROUP BY Region
"""
avg_price_region = duckdb.sql(query).df()

print("Average price per region:")
print(avg_price_region)
```

This query calculates the average price for products sold in each region and returns the results grouped by region:

```
Average price per region:
  Region      Average_Price
0  North 	263.119167
1   East 	288.035625
2   West 	200.139000
3  South 	254.894722
```

### Example Query 4: Total Quantity Sold by Region

To further analyze the data, we can calculate the total quantity of products sold in each region. This helps us see which regions have the most sales activity in terms of volume.

```
# Calculate total quantity sold by region
query = """
SELECT Region, SUM(Quantity_Sold) as Total_Quantity
FROM 'sales_data.csv'
GROUP BY Region
ORDER BY Total_Quantity DESC
"""
total_quantity_region = duckdb.sql(query).df()

print("Total quantity sold per region:")
print(total_quantity_region)
```

This query calculates the total quantity sold per region and sorts the result in descending order, showing which region sold the most products:

```
Total quantity sold per region:
  Region  Total_Quantity
0  South      	1714.0
1   East      	1577.0
2   West      	1023.0
3  North       	588.0
```

### Example Query 4: Joining CSVs

DuckDB offers several advanced features that make it versatile for data analysis. For example, you can easily join multiple CSV files for more complex queries, or query larger datasets stored on disk without loading them entirely into memory.

This SQL JOIN query combines two CSV files, sales_data.csv and product_details.csv, by matching rows based on a common column: Product_ID.

```
query = """
SELECT s.Product_Name, s.Region, s.Price, p.Manufacturer
FROM 'sales_data.csv' s
JOIN 'product_details.csv' p
ON s.Product_ID = p.Product_ID
"""
joined_data = duckdb.sql(query).df()

print(joined_data.head())
```

This should output:

```
     Product_Name  Region   Price   Manufacturer
0	Product_1  North  283.08  Manufacturer_4
1	Product_2   East  325.94  Manufacturer_3
2	Product_3   West   39.54  Manufacturer_2
3	Product_4  South  248.82  Manufacturer_4
4	Product_5   East  453.62  Manufacturer_5
```

## Wrapping Up

In this tutorial, we looked at how to use DuckDB for data analysis with Python.

We worked with CSV files. But you can work with parquet and JSON files and relational databases the same way. So yeah, DuckDB is a useful tool for analyzing large datasets in Python and is quite a useful addition to your Python data analysis toolkit.

I suggest using DuckDB in your next data analysis project. Happy coding!

**[](https://twitter.com/balawc27)**[Bala Priya C](https://www.kdnuggets.com/wp-content/uploads/bala-priya-author-image-update-230821.jpg)**** is a developer and technical writer from India. She likes working at the intersection of math, programming, data science, and content creation. Her areas of interest and expertise include DevOps, data science, and natural language processing. She enjoys reading, writing, coding, and coffee! Currently, she's working on learning and sharing her knowledge with the developer community by authoring tutorials, how-to guides, opinion pieces, and more. Bala also creates engaging resource overviews and coding tutorials.