---
link: https://notes.jjude.com/how-to-use-duck-db-with-golang/
excerpt: DuckDB has golang sdk and it is easy to integrate both.
slurped: 2025-12-09T19:18
title: How to use DuckDB with Golang
tags:
  - go
  - duckdb
---

DuckDB has golang sdk and it is easy to integrate both.

Here is the code.

DuckDB's golang SDK follows the same `database/sql` interface, so using duckdb with golang is as easy as using sqlite.

Attention il faut migrer vers v2 !

```
import (
 "database/sql"
 _ "github.com/marcboeker/go-duckdb"
)

var db *sql.DB

func initDB() error {
 var err error
 // if you don't specify a db name, it will be a in-memory db
 db, err = sql.Open("duckdb", "sample.db")
 if err != nil {
    return err
 }

 // if you have db.Close() here db will be closed
 // you won't be able to use db anywhere else
 // close it at the end in the main func
 // defer db.Close()
 return nil
}

```

Similarly creating a table is easy.

```
func createTbl() error {
	// Create table
	_, err := db.Exec(
  CREATE TABLE employee (
    id INTEGER,
    name VARCHAR(20),
    start_dt TIMESTAMP,
    is_remote BOOLEAN
  ))
	if err != nil {
		return err
	}
	return nil
}
```

How about inserting values into this table?

```
func insertVal() error {
	_, err := db.Exec(`
  INSERT INTO employee (id, name, start_dt, is_remote)
  VALUES
    (1, 'John Doe', '2022-01-01 09:00:00', true),
    (2, 'Jane Smith', '2023-03-15 10:00:00', false)`)
	if err != nil {
		return err
	}
	return nil
}
```

How about opening Parquet files? Since DuckDB has native support for Parquet files, it is again as easy as executing any other `db.Exec` commands.

```
db.Exec("INSTALL parquet;")
db.Exec("LOAD parquet;")
db.Query("SELECT id, name, start_dt, is_remote from read_parquet('employee.parquet');")
```

That's it. This is why I'm more interested in using #duckdb with Golang for data pipelines.