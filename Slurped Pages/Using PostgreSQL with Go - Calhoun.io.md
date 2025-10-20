---
link: https://www.calhoun.io/using-postgresql-with-go
byline: Jon Calhoun
site: Calhoun.io
excerpt: A series covering everything needed to install and start using PostgreSQL with Go's standard libraries, followed by a tour of some popular Golang ORMs.
twitter: https://twitter.com/@joncalhoun
tags:
  - go
  - postgresql
slurped: 2025-10-20T17:37
title: Using PostgreSQL with Go - Calhoun.io
---

PostgreSQL is an open source relational database system that has been around for well over a decade and has proven to be a great all around storage choice when developing a web application.

In this series we are going to walk through everything from first installing PostgreSQL 9.5 all the way to using it with a Go application. While this post will cover all of the basics required to get started using SQL with Golang, it is not a full course on SQL. It is instead intended to guide you by giving you enough information to be productive, while not overloading you with details that can be learned as you progress.

In order to achieve this, we will start off by walking through the installation and setup process of Postgres, followed by a brief overview of using `psql` to interact with PostgreSQL.

After that we will start connecting to our database using Go and the [database/sql](https://golang.org/pkg/database/sql/) package and writing raw SQL. This is intended to give you enough knowledge to move forward with writing your own SQL integration in your Go code if you choose to.

Next, we will discuss ORMs, which are tools that help us convert data between our application types and the records stored inside of a PostgreSQL database. While it isn’t required to use an ORM to develop applications that interact with PostgreSQL, I find it valuable to discuss the pros and cons of using an ORM vs writing your own code, especially for beginners.

Finally, we will dive into ORMs in more detail. Specifically, we will be using [GORM](http://jinzhu.me/gorm/) to create a model layer for an application, and in future posts I might also explore a few other ORMs if there is enough demand. (Hint: Email me - [jon@calhoun.io](mailto:jon@calhoun.io) if you want me to do this!)

## Articles in this series

The series currently has (or is expected to have) the following articles in it:

## 1.How to install PostgreSQL 9.5 / 9.6

Installing Postgres is specific to your operating system, so this section is broken into three different posts. Each of these posts will guide you on how to install PostgreSQL 9.5 and will also give you guidance on how to connect to your Postgres instance using a role.

- [How to install PostgreSQL 9.5 on Ubuntu 16.04](https://www.calhoun.io/how-to-install-postgresql-9-5-on-ubuntu-16-04/)
- [How to install PostgreSQL 9.6 on Mac OS X (10.7 or later)](https://www.calhoun.io/how-to-install-postgresql-9-6-on-mac-os-x/)

## 2.Interacting with a PostgreSQL database using SQL

This section will be composed of roughly three articles, but may be expanded over time.

- [Creating PostgreSQL databases and tables with raw SQL](https://www.calhoun.io/creating-postgresql-databases-and-tables-with-raw-sql/)
- [Inserting records into a PostgreSQL database using SQL](https://www.calhoun.io/inserting-records-into-a-postgresql-database-using-sql/)
- [Querying for records stored in a PostgreSQL table using SQL](https://www.calhoun.io/querying-for-records-stored-in-a-postgresql-table-using-sql/)
- [Updating and Deleting records stored in a PostgreSQL table using SQL](https://www.calhoun.io/updating-and-deleting-records-using-sql/)

These four should be read in the order presented unless you are already familiar with SQL.

## 3.Interacting with a PostgreSQL database using Golang and the database/sql package

This section is again composed of several articles and could be expanded over time, but for now it is three articles that should be read in the order presented here unless you are somewhat familiar with Go & PostgreSQL.

- [Connecting to a PostgreSQL database with Go's database/sql package](https://www.calhoun.io/connecting-to-a-postgresql-database-with-gos-database-sql-package/)
- [Inserting records into a PostgreSQL database with Go's database/sql package](https://www.calhoun.io/inserting-records-into-a-postgresql-database-with-gos-database-sql-package/)
- [Updating and deleting PostgreSQL records using Go's sql package](https://www.calhoun.io/updating-and-deleting-postgresql-records-using-gos-sql-package/)
- [Querying for a single record using Go's database/sql package](https://www.calhoun.io/querying-for-a-single-record-using-gos-database-sql-package/)
- [Querying for multiple records with Go's sql package](https://www.calhoun.io/querying-for-multiple-records-with-gos-sql-package/)

## 4.ORMs for using PostgreSQL with Golang

Learn how to use ORMs made available by the Go community to interact with your PostgreSQL database.

- [Subtle issues with ORMs and how to avoid them](https://www.calhoun.io/subtle-issues-with-orms-and-how-to-avoid-them/) - In this article I discuss some of the perceived pros and cons of ORMs as well as some of the real issues that can stem from using an ORM and how to write code that is less likely to fall into those traps

## 5.Migration techniques with PostgreSQL and Golang

This section is an idea that I would like to explore in the future, but as of now there aren’t any articles written.

## 6.Possibly more!

If you have any suggestions on what other articles would compliment this series please reach out and let me know - [jon@calhoun.io](mailto:jon@calhoun.io). I am always open to suggestions for new posts 😀

## This is all tentative

The article titles listed here are mostly tentative and might be changed as I write the articles. My goal is to cover all of the subjects mentioned, but this might require me to create more or less articles than I originally intended.

## Want to see how databases work in the bigger picture?

Getting started with SQL in Go can be tricky. How do you design your code so that it is testable? How do you make your database connections available in your http handlers?

In my course, [Web Development with Go](https://www.usegolang.com/), we use the SQL database in a real web application and cover all of these details in more. You will learn how to properly build a robust, testable database layer and how to make it available to your http handlers.

If you sign up for my mailing list (down there ↓over there →) I'll send you a FREE sample so you can see if it is for you. The sample includes over 2.5 hours of screencasts and the first three chapters from the book.

You will also receive notifications when I release new articles, updates on upcoming courses (including FREE ones), and I'll let you know when my paid courses are on sale.

©2024 Jonathan Calhoun. All rights reserved.