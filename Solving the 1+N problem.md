---
link: https://acadia.engineering/blog/solving-the-1-plus-N-query-problem
---

By [Evan Czaplicki](https://x.com/evancz), 25 August 2026

When working with databases, you want to express your queries using nice efficient joins that take full advantage of the query planner. But when you use an Object-Relational Mapping (ORM) in object-oriented languages like Ruby or Python, it becomes quite easy to deoptimize these joins by accident. Instead of doing a single join, an innocent looking `for` loop can end up doing tons and tons of separate queries, resulting in much slower execution.

I saw this as a design opportunity for Acadia’s query language. There are two ways to express the same thing, and one of them is always worse... Why allow the worse one? Is there a way to guarantee that people _always_ end up writing the better quality code? To answer this question, we will end up revisiting the Datalog query language from the 1970s. Datalog provides very strong and interesting guarantees about your queries, and it inspired some crucial design decisions in Acadia’s query language.

It turns out we can fully eliminate the 1+N problem, but more interestingly, we can also guarantee that **all queries terminate**. This means it is impossible to write an Acadia query that would require the database to hang forever in an infinite loop. And even stronger than that, we can guarantee that **all queries terminate in time polynomial to the amount of data**!

## What is “the 1+N problem”?

The basic idea is that you do a single query (1 query), and then you do another query for each of the rows returned (N queries). So you end up doing 1+N separate queries.

One major issue with an Object-Relational Mapping is that the 1+N problem comes up extremely easily. Maybe you have a `books` table and an `authors` table. Normally you would find information about the author of a book by doing a `JOIN`. With an ORM, it _appears_ that the author information is just part of each book object. So you could write something like this:

```python
for book in database.books:
    print(book.author.name)
```

This looks like totally normal code. I am just accessing my objects. What is the problem? Behind the scenes, this code does one query to get `database.books` (1 query) and then another query to get the `book.author.name` of each one (N queries). This is the 1+N problem!

This problem is extremely common because many ORMs hijack the humble “object access” syntax. Normally when you access an object, like `point.x`, you do a simple look up in memory. Virtual machines like the JVM and V8 go to great lengths to make these accesses as fast as possible. But with an ORM, accessing an object field is sometimes a quite expensive database query running on a different machine. These implicit queries are often implemented as a blocking call, so that whole thread is essentially idle while it waits for N queries to the authors table to go through.

Object-oriented programmers must be constantly vigilant if they want to avoid this. “Is this for loop doing a database query? Is this field access doing a database query?” The Object-Relational Mapping is meant to let you think in an object-oriented way, but you end up needing to always think about the underlying tables if you want the code to work well.

The obvious solution is to just not do this. Acadia does not have objects, and it is not possible to override field access. Case closed, right? Unfortunately, the problem is more fundamental than that.

> **Note:** Most people would not consider GraphQL an Object-Relational Mapping, but it overloads the “object access” syntax in the same way. Maybe `point.x` just accesses a column, but maybe `book.author.name` requires a join. You need constant vigilance to make sure you are not doing any accidental joins.

## What is the essence of the 1+N problem?

It is also possible to run into the 1+N problem in query languages like pl/pgSQL. pl/pgSQL is pretty neat. It is like SQL, but with some additional features like local variables and `for` loops. It runs fully within the database, so it may help you avoid some query parsing or some round trips to the database. Just like Acadia, it is not an object-oriented language and it is not possible to override the field access syntax. But with pl/pgSQL, the 1+N problem is still possible:

```sql
for book in (select * from books) loop
    return query select author.name from authors where author.id = book.author_id;
end loop
```

All you need is a `for` loop! As long as you can iterate over rows and do separate queries for each, you have the potential for the 1+N problem. Acadia lacks `for` loops, so... Problem solved, right? Again, not quite.

## Is Turing Completeness actually a problem?

Most of the time we absolutely need our programming languages to have `for` loops (or equivalently, recursion) so that we can compute everything that is computable. But in the context of query languages, this level of expressiveness is the core of the 1+N problem.

One of my favorite experiences as a student was learning Datalog. This query language is **a subset of Prolog that lacks unbounded recursion**. As a result, all Datalog programs are **guaranteed to terminate**. Even more interestingly, they are **guaranteed to terminate in time polynomial to the amount of data**. I used it to do some 1980s-style [natural language processing](https://lara.epfl.ch/w/_media/prolog-digital.pdf) and to implement a [k-CFA](https://yanniss.github.io/kcfa-pldi10.pdf) for a simple functional language. It was great fun, and it stuck with me as one of my favorite examples of making something simpler to get stronger guarantees.

This was the inspiration for Acadia’s lack of recursive functions. In addition to resolving the 1+N problem, the lack of recursion also guarantees that **all Acadia queries terminate**! Without for loops or recursion, there is no way for a program to end up in an infinite loop. This means that it is impossible to write an Acadia query that would require the database to hang forever.

The only way to combine the `books` and `authors` table is to `intersect` them:

```elm
getAuthorNames : Transaction (Rows String)
getAuthorNames =
  intersect .author_id books .id authors
      |> map (\(book, author) -> author.name)
      |> selectAll
```

The resulting SQL for this query will be something like:

```sql
select author.name from books as book, authors as author where author.id = book.author_id
```

Some readers will be more comfortable with the SQL version, but Acadia’s query language really shines as your programs become larger. This website uses Acadia for all of its database needs, and it has been a joy to organize my query code into modules, define helper functions, write precise types, work with friendly error messages, etc.

## Conclusion

I hope that this explanation of how Acadia avoids the 1+N problem gives some insight into its design philosophy. Like with Datalog, guarantees like **all queries terminate** and **no 1+N queries** are only achievable if the language is as simple as possible. I love research results where making something simpler also gives you stronger guarantees, and I am so happy I was able to apply this particular Datalog insight in Acadia’s query language!

If you are curious, you can download Acadia [here](https://acadia.engineering/download) and give it a try. I want to write a couple more blog posts like this, so let me know in the [Discord](https://discord.gg/sUSVUJNY6r) how your own experiments are going. I will prioritize topics based on the feedback I see.

Finally, please consider [supporting](https://acadia.engineering/support) this project! The goal is to make Elm and Acadia development sustainable, and I am so thankful to everyone who decided to support so far!