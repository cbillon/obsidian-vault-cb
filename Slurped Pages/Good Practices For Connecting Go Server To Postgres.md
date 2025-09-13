---
link: https://journal.hexmos.com/good-practices-for-connecting-go-server-to-the-postgres-2/
byline: Ganesh Kumar
site: Hexmos Journal
date: 2024-05-25T16:29
excerpt: Ever wondered what would be the best way to get started with a GO server for your backend? In this article, I am going to give a few best practices for setting up DB and routing on a Go server.
twitter: https://twitter.com/@AmazingStardom
slurped: 2025-08-31T19:50
title: Good Practices For Connecting Go Server To Postgres
tags:
  - go
  - postgresql
---

## Preparing a PG DB and Go backend

Let's Create a bookstore database where we can fetch book details

Creating a BookStore DB in [PostgreSQL](https://www.postgresql.org/?ref=journal.hexmos.com)

I'll start with the installation

```
$ sudo apt install postgresql
$ sudo -u postgres psql
```

### Logging into postgres

```
psql -h localhost -U postgres

```

Postgres console will be activated

```
psql (14.11 (Ubuntu 14.11-0ubuntu0.22.04.1))
Type "help" for help.
postgres=# 
```

Create DB name **bookstore** by this command

```
postgres=CREATE DATABASE bookstore;
CREATE DATABASE
```

### Check out the DB and create Table

```
postgres=# \c bookstore
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to the database "bookstore" as user "postgres".
```

Create a Table in the bookstore DB

```
CREATE TABLE books (
    isbn char(14) NOT NULL,
    title varchar(255) NOT NULL,
    author varchar(255) NOT NULL,
    price decimal(5,2) NOT NULL
);

INSERT INTO books (isbn, title, author, price) VALUES
('9780099470464', 'A House for Mr. Biswas', 'V. S. Naipaul', 8.99),
('9780143037346', 'Miss New India', 'Bharati Mukherjee', 9.99),
('9781784782781', 'The Lives of Others', 'Neel Mukherjee', 11.99),

ALTER TABLE books ADD PRIMARY KEY (isbn);
```

Then Verify whether it is created

```
bookstore-# \dt
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | books | table | postgres
(1 row)
```

### Setting up Go backend

```
$ mkdir bookstore && cd bookstore
$ mkdir models
$ touch main.go models/models.go
$ go mod init go.backend
go: creating new go.mod: module go.backend
```

File Structure of example :

```
bookstore/
├── go.mod
├── go.sum
├── main.go
└── models
    └── models.go
```

## Set a Global DB Instance

This method simplifies accessing the database connection across the application.

It initializes the database connection, sets up an HTTP server, and listens for incoming requests.

In the context of our bookstore application, the code would look something like this

```
// models/models.go
package models

import (
    "database/sql"
)
```

Create an exported global variable to hold the database connection pool.

```
// models/models.go
var DB *sql.DB

type Book struct {
    Isbn   string
    Title  string
    Author string
    Price  float32
}
```

AllBooks() returns a slice of all books in the books table.

```
// models/models.go
func AllBooks() ([]Book, error) {
    // Note that we are calling Query() on the global variable.
    rows, err := DB.Query("SELECT * FROM books")
    if err != nil {
        return nil, err
    }
    defer rows.Close()

    var bks []Book

    for rows.Next() {
        var bk Book

        err := rows.Scan(&bk.Isbn, &bk.Title, &bk.Author, &bk.Price)
        if err != nil {
            return nil, err
        }

        bks = append(bks, bk)
    }
    if err = rows.Err(); err != nil {
        return nil, err
    }

    return bks, nil
}              
```

```
// main.go
package main

import (
    "database/sql"
    "fmt"
    "log"
    "net/http"

    "go.backend/models"

    _ "github.com/lib/pq"
)

```

Install dependencies

```
$ go get github.com/lib/pq
```

Initialize the `sql.DB` connection pool and assign it to the models.DB

```
func main() {
    var err error
    // global variable.
    models.DB, err = sql.Open("postgres", "postgres://user:pass@localhost/bookstore")
    if err != nil {
        log.Fatal(err)
    }

    http.HandleFunc("/books", booksIndex)
    http.ListenAndServe(":3000", nil)
}
```

When I make a request **books**

```
$ curl localhost:3000/books
978-1503261969, Emma, Jayne Austen, £9.44
978-1505255607, The Time Machine, H. G. Wells, £5.99
978-1503379640, The Prince, Niccolò Machiavelli, £6.99
```

**Remember** global variable for the database connection is suitable when:

- Your project is simple and small, so tracking global variables isn't hard.
- Your code handling web requests is split across different folders, but all database actions stay in one folder.
- You don't need to pretend the database isn't there for testing.

### Global variable with an `InitDB` function

A variation on the 'global variable' approach that I sometimes see uses an initialization function to set up the connection pool, like so:

- All database stuff is in one place.
- The global database variable is hidden from other parts of the program, so it can't be changed by mistake.
- During testing, you can easily set up a test database connection using a special function.

```
// models/models.go
package models

import (
    "database/sql"

    _ "github.com/lib/pq"
)
```

Initialize the `sql.DB` connection pool and assign it to the models.DB

```
// This time the global variable is unexported.
var db *sql.DB

// InitDB sets up setting up the connection pool global variable.
func InitDB(dataSourceName string) error {
    var err error

    db, err = sql.Open("postgres", dataSourceName)
    if err != nil {
        return err
    }

    return db.Ping()
}
```

## Wrapping the connection pool

This pattern we'll look at uses dependency injection again, but this time we're going to wrap the `sql.DB` connection pool in our custom type.

This approach simplifies database calls, enhances scalability by accommodating additional dependencies, and improves the ability to test through the use of interfaces and mock implementations.

Despite initially appearing more complex, this method offers significant advantages in terms of code readability, maintainability, and ease of testing, making it a valuable addition to the toolkit for Go developers working on web applications

```
// models/models.go
package models

import (
    "database/sql"
)

type Book struct {
    Isbn   string
    Title  string
    Author string
    Price  float32
}

type BookModel struct {
    DB *sql.DB
}

func (m BookModel) All() ([]Book, error) {
    rows, err := m.DB.Query("SELECT isbn, title, author, price FROM books")
    if err!= nil {
        return nil, err
    }
    defer rows.Close()

    var books []Book

    for rows.Next() {
        var book Book
        err := rows.Scan(&book.Isbn, &book.Title, &book.Author, &book.Price)
        if err!= nil {
            return nil, err
        }
        books = append(books, book)
    }
    if err = rows.Err(); err!= nil {
        return nil, err
    }

    return books, nil
}

```

```
// main.go
package main

import (
	"database/sql"
	"fmt"
	"log"
	"net/http"

	"go.backend/models"

	_ "github.com/lib/pq"
)

type Env struct {
	books models.BookModel
}

func main() {
	db, err := sql.Open("postgres", "postgres://postgres:123@localhost/bookstore")
	if err != nil {
		log.Fatalf("Failed to open database: %v", err)
	}

	env := &Env{
		books: models.BookModel{DB: db},
	}

	http.HandleFunc("/books", func(w http.ResponseWriter, r *http.Request) {
		books, err := env.books.All()
		if err != nil {
			log.Printf("Error fetching books: %v", err)
			http.Error(w, http.StatusText(500), 500)
			return
		}

		for _, book := range books {
			fmt.Fprintf(w, "%s, %s, %s, £%.2f\n", book.Isbn, book.Title, book.Author, book.Price)
		}
	})

	log.Println("Starting server...")
	log.Fatal(http.ListenAndServe(":3000", nil))
}

```

## Creating your own custom [MUX](https://github.com/gorilla/mux?ref=journal.hexmos.com)

I'll be using The gorilla/mux package for routing, and PostgreSQL for database management.

We'll walk through setting up the project, installing dependencies, initializing the application, and fetching data from a PostgreSQL database to display on a webpage.  
 

```
package main

import (
	"database/sql"
	"fmt"
	"log"
	"net/http"

	"go.backend/models"

	"github.com/gorilla/mux"
	_ "github.com/lib/pq"
)
```

Install Dependencies

```
$ go get github.com/gorilla/mux
$ go get github.com/lib/pq

```

Initialise MUX Router

```
func main() {
	var err error

	// Connect to the PostgreSQL database
	models.DB, err = sql.Open("postgres", "postgres://postgres:123@localhost/bookstore")
	if err != nil {
		log.Fatal(err)
	}

	// Initialize the mux router
	r := mux.NewRouter()
	r.HandleFunc("/books", booksIndex).Methods("GET")

	// Start the HTTP server
	log.Printf("Server is listening on port 3000...\n")
	log.Fatal(http.ListenAndServe(":3000", r))
}
```

Handling Requests and Displaying Data

```
// booksIndex sends a HTTP response listing all books.
func booksIndex(w http.ResponseWriter, r *http.Request) {
	bks, err := models.AllBooks()
	if err != nil {
		log.Print(err)
		http.Error(w, http.StatusText(500), 500)
		return
	}

	for _, bk := range bks {
		fmt.Fprintf(w, "%s, %s, %s, £%.2f\n", bk.Isbn, bk.Title, bk.Author, bk.Price)
	}
}

```

## Conclusion

We Explored PostgreSQL database connections in Go servers, we highlighted essential steps like initializing the database and Go backend, setting up a global database instance, utilizing connection pools, and implementing custom MUX for optimized request management.