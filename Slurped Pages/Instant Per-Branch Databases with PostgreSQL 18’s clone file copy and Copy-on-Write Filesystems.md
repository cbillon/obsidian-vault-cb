---
link: https://medium.com/axial-engineering/instant-per-branch-databases-with-postgresql-18s-clone-file-copy-and-copy-on-write-filesystems-1b1930bddbaa
byline: Bob Ternosky
site: Axial Engineering
date: 2026-01-12T17:59
excerpt: “” is published by Bob Ternosky in Axial Engineering.
twitter: https://twitter.com/@AxialCo
slurped: 2026-05-23T10:10
title: Instant Per-Branch Databases with PostgreSQL 18’s clone file copy and Copy-on-Write Filesystems
tags:
  - linux
  - postgresql
  - copy-on-write
  - btrfs
---
Most developers use source code control tools (e.g. **Git**) and code branches to provide isolated, parallel development. However, when the application needs to interact with a database most developers do not have databases that are isolated between branches.

Per-branch databases are desirable for many of the same reasons as code branches:

- Parallel development
- Isolation and safety
- Easy rollbacks

In our experience, most development shops will ensure that developers have a non-production database for use during code development. Some shops will also extend their infrastructure so that each developer gets their own database to use. But it seems extremely rare to find a setup utilizing per-branch databases.

The lack of per-branch databases are usually the result of:

- complexity in setting up and maintain the databases
- time needed to spin up a new database
- the amount of disk required to have numerous databases

In this article, we’ll discuss how you can use **PostgreSQL** 18’s `file_copy_method=clone` with a _Copy-on-Write_ filesystem to make per-branch database creation easy to manage, insanely fast and consume very little disk.

## Rationale: Why do developers want/need per-branch databases?

We have stated that isolation and safety are a big part of per-branch databases. How? The benefits break down into three general categories:

- Schema isolation
- Data isolation
- Failure isolation

### Schema isolation

Let’s examine a scenario where the lack of schema isolation can cause problems.

In this scenario, we are going to implement a new feature. We need a branch from our code mainline (**main**) that we’re going to call **ticket-one**.

As part of implementing this new feature, we make a database schema change that renames a column and we make the necessary code changes for the column rename.

Once we have completed development of the feature, we create a pull request and ask for a code review and/or testing from others on our team.

If we pick up any new work before **ticket-one** is merged, we would need to revert the change to the database before beginning the new work. This introduces delay into the workflow. In the case that these two streams of work are concurrent for any sustained period of time, the switching cost is incurred on every context switch.

This scenario can arise if any of the following types of changes happen in your database:

- Rename of any object type
- Data Type change of any object
- Change to Triggers/Functions
- Removal of any database object (columns, table, triggers, etc.)

In some cases, the pain to “revert” the change could be very difficult and/or time consuming.

If we had a per-branch database setup, then the schema changes would have been made in an isolated database and there would be no issues encountered when branch switching.

### Data isolation

Let’s examine a scenario where lack of isolation can cause problems.

In this scenario, we are going to implement a new feature. We need a branch from our mainline (**main**) that we’re going to call **ticket-one**.

As part of implementing this new feature we need to remove data from a table (not all data, but all data of a certain type perhaps). We make any code changes needed to deal with the data being removed.

An example: Remove data from a table where a column has a specific value.

Once we have completed development of the feature, we generate a pull request and ask for a code review and/or testing from others on our team.

If we pick up any new work before **ticket-one** is merged, we would need to revert the change to the database before beginning the new work. This introduces delay into the workflow. In the case that these two streams of work are concurrent for any sustained period of time, the switching cost is incurred on every context switch.

If we had a per-branch database setup, then the data changes would have been made in an isolated database and there would be no issues encountered when branch switching.

Data isolation means we don’t have to to worry about assumptions in application code about the existence of specific data that may have been removed on another branch.

With data isolation, we also don’t lose interesting data on any current branch during switching. This interesting data could be things like test data setup.

### Failure isolation

Let’s examine a scenario where lack of isolation can cause problems.

In this scenario, we are going to implement a new feature. We need a branch from our mainline (**main**) that we’re going to call **ticket-one**.

As part of implementing this new feature we need to make change to a trigger function in the database. We have applied the changed trigger function to the database successfully. But while testing the updated function we realize our change is not correct (i.e. a bug).

Before we have a chance to figure out a fix for the bug, we are required to pick up a high priority production bug fix.

Since we are picking up new work, we would need to revert the change to the database before beginning the new work. This introduces delay into the workflow. In the case that these two streams of work are concurrent for any sustained period of time, the switching cost is incurred on every context switch.

If we had a per-branch database setup, then the trigger change would have been made in an isolated database and there would be no issues encountered when branch switching.

## But what about …?

We’re sure a number of readers will immediately have thoughts about a few alternate development setups and ask “why not use this instead?”.

A few of those setups:

- Use a schema-only database and data generation
- Use a lightweight database replacement (e.g. **SQLite**)
- Use a versioned database (e.g. **Dolt**, **Neon**)

We’ll examine each option and explain why we chose not to use it.

### Use a schema-only database and data generation

If you have the **DDL** to generate your database and tools / scripts to generate data in your database you can create your own per-branch databases pretty easily.

In an ideal world, every data model would be perfect and every development database would be pragmatically populated based on the needs of the developer. Unfortunately, reality is rarely that clean. One of the benefits of our approach is that it enables significantly faster development without first having to resolve every imperfection.

Later in the document we will point out where using **DDL** and data generation is an acceptable step for creating a database that can be used as a template for the per-branch databases as part of our solution.

### Use a lightweight database replacement (e.g. SQLite)

We’ve seen many articles about people using drop-in replacements for their database. They might use a super light-weight alternative database (e.g. **SQLite**) or something like **PGlite** where it runs a **PostgreSQL** server, but it runs fully in memory.

Using something like **SQLite** is problematic unless your code is using a library that supports multiple database protocols. Many shops use native libraries to talk to the database (e.g. **pyscopg** from **Python**). Switching to **SQLite** isn’t feasible in that scenario.

Using **PGlite** seems like a good option, but it has its own limitations and potential issues for some users:

- It requires the user of **docker**. Not everyone uses **docker** and they may not want to incur the complexity of adding it to their project.
- It doesn’t support all **PostgreSQL** features

We like to have development environments be as production-like as possible. With either of the alternative solutions you are using a database tool that is NOT the same as your production database. This can result in unexpected issues when shifting from development to production.

We feel our solution provides a better outcome than these solutions.

### Use a versioned database

There are products available to do versioned databases. Examples:

- [Dolt](https://www.dolthub.com/)
- [Neon](https://neon.com/)

They provide versioned or branched databases which is the problem we’re trying to solve.

However, we chose not to use these types of products as:

- These products tend to cost money, where our solution is free
- Any “free” alternatives would require using a 3rd party setup on top of **PostgreSQL**. Our solution uses native **PostgreSQL** as the solution so we avoid 3rd party bloat / package maintenance

## Some of our specific pain points

We have some additional specific environmental issues that also drove us down the road to our solution that you may not have to contend with:

- Our database must have data that is representative of production. We have some legacy data issues that make generative data problematic.
- Our database is large enough to be painful to copy, but using _Copy-on-Write_ makes it feasible. Even with modern solid-state drives, there are limits to how fast data can be copied. This can be exacerbated if there are any layers of virtualization involved (e.g. Docker).
- Maintaining the correspondence of databases to branches while providing ergonomics is non-trivial if you want to cover the most beneficial use-cases (hot fixes/collaborative efforts)

## Concerns with per-branch databases

We’ve discussed some problems that we might solve with per-branch databases. No solution is perfect so let’s review some of the concerns that people might have with per-branch databases:

**Database creation/setup concerns**

- How easy is it to create a new per-branch database?
- How long does it take to create a new per-branch database?

**Disk consumption concerns**

- If the database we use is large, it becomes untenable pretty quickly to keep around numerous copies of the database.

**Application Configuration**

- How is a branch of code configured to connect to a specific per-branch database?
- How does this configuration get changed when branch switching?

**Raw Database Access**

- How is a specific per-branch database accessed using a CLI or GUI database tool?

**Database Cleanup**

- How are obsolete per-branch databases cleaned up once they are no longer required?

We’ll address all of these concerns as we explore our per-branch database solution.

## Our goals

Now that we’ve covered the background, we can explore our proposed per-branch database solution. Our goal is to provide developers with fully isolated, per-branch databases in their development environment, where the solution:

- is reasonably cross-platform with off-the-shelf tools
- provides flexible choice of Operating System
- works in Containers or as an installed server
- can run the database locally (laptop/desktop) or remote
- is programming language agnostic
- does not significantly increase storage consumption
- provides fast database cloning
- makes it easy to match a branch with its database

In the rest of this post we’ll cover:

- Conventions used in this document
- Tools and technology required for our solution
- Software Installation / Configuration
- Setting up a Template Database
- Branching mechanics and per-branch database setup
- Utilizing database clients (CLI and GUI)
- Managing database drift
- Removing obsolete databases
- Optional: Automated configuration

## Conventions used in this document

We will use a few formatting conventions throughout this document. In this section we’ll review the conventions.

**Shell Commands**

We will have examples in the document that show one or more commands to be run from a shell. We show the example system prompt as `[bash]$`. This does not require you to use the **bash** shell. It is merely a convention to show a command being run.

Example:

[bash]$ cd $HOME

NOTE: We have tested our examples on:

- Linux Ubuntu
- OSX
- Windows Subsystem for Linux

**Placeholders**

In our examples we use a variety of placeholder values. You will be expected to replace these values with values specific to your environment. These placeholders take the form of an upper case string enclosed in angle brackets, for example: `<VAR_NAME>`

In the following example, you would be required to replace `<PG_HOST>`with the IP address or hostname of your **PostgreSQL** server:

[bash]$  psql -X postgres://postgres@<PG_HOST>

Commonly used placeholder values:

- `<PG_INSTALL>`: The directory that the **PostgreSQL** server is configured to use for storage. The default installation is typically:`/var/lib/postgresql/pg_data`
- `<PG_HOST>`: The IP/hostname for your **PostgreSQL** server (e.g. `127.0.0.1`, `localhost` etc.)
- `<PG_PORT>`:The listening port your **PostgreSQL** server is configured to use
- `<DB_USER>`: The username to use when connecting to a database
- `<DB_PASSWD>`: The password to use when connecting to a database
- `<DB_NAME>`: The database identifier name to use when connecting to a database (e.g. **main**, **branch_one**, etc.)

## Tools and technology required for the solution

Our per-branch database solution requires:

- [PostgreSQL](https://www.postgresql.org/) Server version 18+
- A _Copy-on-Write_ filesystem for the database file storage
- Source code with each branch in its own directory

Later, in the section titled _Automated Configuration_ we will also utilize **direnv**. This is not required for the branching / database setup, but is highly recommended.

## PostgreSQL Server version 18+

Our solution takes advantage of features introduced in **PostgreSQL** with version 18. The server deployment is flexible:

- Can be containerized or a bare-metal server
- Can be run locally or remote to your application
- Note: **Amazon RDS** will not work at the current time due to lack of a necessary feature within **RDS**

We are going to make use of a few **PostgreSQL** features to implement our solution:

- Template databases
- The _clone_ file copy method

**PostgreSQL** has a feature called [Template Databases](https://www.postgresql.org/docs/current/manage-ag-templatedbs.html). When creating a new database an existing database can be used as a template to create the new database. More simply, it will copy the template database as the new database.

This requires two steps:

1. A database must exist that will be used as a template. By default, **PostgreSQL** provides a database for this purpose named _template1_. But ANY database in the server can be used as a template.
2. Use of the `TEMPLATE` keyword when creating a database Example:

CREATE DATABASE new_database_name WITH TEMPLATE template_database_name …;

In very simplified terms, a database in **PostgreSQL** is manifested as a directory of files on disk and corresponding metadata in the running server. When a new database is created using a _Template_ database, **PostgreSQL** will:

- Copy the directory of files from the _Template_ database to a new folder that represents the new database
- Update the metadata in the running server to utilize the new database folder

**PostgreSQL** has a configuration option called [file_copy_method](https://www.postgresql.org/docs/18/runtime-config-resource.html#GUV-FILE-COPY-METHOD). This setting controls how the data files are copied in the create from template operation. It has two possible values:

- `_COPY_` — **PostgreSQL** will do a full physical copy of a database
- `_CLONE_` — **PostgreSQL** will use operating system features (if present) to share common disk blocks between databases. The operating system features that are required by `_CLONE_` are provided by _Copy-on-Write_ filesystems

## A Copy-on-Write filesystem for the database file storage

Per [Wikipedia](https://en.wikipedia.org/wiki/Copy-on-write), _Copy-on-Write_ is defined as follows:

Copy-on-Write (COW), also called implicit sharing or shadowing, is a   
resource-management technique used in programming to manage shared data   
efficiently. Instead of copying data right away when multiple programs use it,  
the same data is shared between programs until one tries to modify it. If no   
changes are made, no private copy is created, saving resources. A copy is   
only made when needed, ensuring each program has its own version when   
modifications occur. This technique is commonly applied to memory, files,   
and data structures.

When the following items are combined:

- using a _Copy-on-Write_ filesystem for the storage of a **PostgreSQL** database
- using the `_CLONE_` value for **PostgreSQL’s** configuration of `file_copy_method`
- creating a database from a _Template_

This provides benefits:

- The creation of the database is insanely fast. Since the _Copy-on-Write_ filesystem isn’t copying the full data (it’s sharing data at the filesystem level) the copy executes much faster. The time taken will vary with your database size and your storage media (i.e. spinning disk vs SSD etc.). Generally speaking the creation will be orders of magnitude faster than full copies. On a SSD even multi-hundred GB copies can happen in less than a second.
- Disk consumption for the new database will be much less than the original database. The filesystem only needs to create metadata entries about the database instead of full data copies. The amount of space required will vary with the filesystem and the size of the original database. But overall, it will generally be an order of magnitude less.

As indicated above, data copies are only made when needed. As either the _template_ database or a cloned database undergo change, the filesystem will need to make file copies so the two databases can diverge. The performance impact of the copies is generally pretty small since:

- The rate of overall data change activities in most development databases tends to be pretty small
- The amount of data changed in an individual database update tends to be pretty small
- **PostgreSQL** uses [Write Ahead Logging](https://www.postgresql.org/docs/current/wal-intro.html) (AKA WAL) which means updates to the database are very fast from the user’s perspective as writes are made to the WAL and therefore require no copies since it’s net new data. The WAL logs are processed by the database in the background and therefore the user does not experience any delays.

If an application generates a LOT of writes or very large updates there could be concern about the impact. But even in those cases, by using a _Copy-on-Write_ filesystem:

- The server will consume less disk overall than two discrete copies since only changed data is copied.
- End user performance will be comparable with the use of **Write Ahead Logs**.
- Server performance may be slightly worse than purely independent databases without shared data but the impact should be nominal. Even if it is slightly worse, we feel this is an acceptable trade off as we are running in a development environment.

Example file systems that support _Copy-on-Write_:

- Apple File System (**APFS**)
- **BTRFS**
- **ZFS**

## Source code with a directory per-branch

A key component of our workflow is ensuring that each code branch has a full checkout of the code (a branch) in its own directory rather than switching branches within the same directory.

In our solution, we will use **Git Worktrees** for our source code management / branching. We are not going to dive deep into **Git** or **Git Worktrees** works. See the [Git Worktree](https://git-scm.com/docs/git-worktree) documentation page for detailed information.

Our example **Git Worktree** workflow can easily be adapted to any source code management tool (or even having no source code management tool).

We want this document to be as flexible as possible to cover a variety of use cases:

- Developers should be able to use the OS of their choice
- Developers should be able to run the **PostgreSQL** Server in a container or as system installed service
- Support for any container technology (**podman**, **docker**, etc.) when containerized
- Support for any software management tool (**apt**, **yum**, **snap**, raw source etc.) when system installed
- The **PostgreSQL** server can be run locally on a laptop (or desktop) or on shared infrastructure

Due to the wide variety we are not going to try to cover all of the installation / setup instructions. The installation of software is up to the developer based on their needs. Instead we’ll identify what our workflow actually requires to function:

- A **PostgreSQL** Server (minimum version 18 to support for the `file_copy_method=clone` feature)
- **PostgreSQL** configured to use `file_copy_method = clone` (in `postgresql.conf`)
- **PostgreSQL** configured to have it’s data storage on a _Copy-on-Write_ filesystem (`data_directory = '/path/to/store/databases’` (in `postgresql.conf`)
- A **PostgreSQL** Client (CLI or GUI doesn’t matter)
- Per-branch code directories (You can adapt our workflow to avoid this, but its a bit more complicated)

One extra note for those running **PostgreSQL** in a container. Make sure your container data storage is using a _Copy-on-Write_ filesystem. A naive container configuration will end up using the container filesystem default. Unfortunately, many images use _ext4_ as that filesystem and _e_xt4 does _not_ support _Copy-on-Write_.

## Setting up a template database

The first step in our process is to create a _Template_ database that contains a full database for per-branch use. This requires that you have a running **PostgreSQL** server. There are a few ways to create the _Template_ database:

1. Create a database using **pg_dump** and **pg_restore**

If you want to use an existing database as the source of your template you can:

- Export an image from a **PostgreSQL** server using **pg_dump.** The dump file contains all database objects and table data
- Create necessary users and their grants so that **pg_restore** will work
- Import the image using **pg_restore**

2. Use one or more **DDL** scripts that:

- Create necessary users and their grants for the database
- Create a database
- Create your database schema(s), tables, views, functions, triggers etc.
- Optional: Run a script to populate data into your tables

You can use either of these methods or any other method you like to create a database.

### Creating a database using pg_dump and pg_restore

You can use native **PostgreSQL** tooling to create a database _Template_:

- Export an existing database (e.g. _production_) using **pg_dump**
- Load the export into your local **PostgreSQL** using **pg_restore**

Here is an example of creating the template database using a **pg_restore** from a **pg_dump** file:

[bash]$ psql -X postgres://postgres@<PG_HOST>  
... create users/grants needed by pg_restore ...

[bash]$ psql -X postgres://postgres@<PG_HOST>   
    -c "CREATE DATABASE development_template OWNER <DB_USER>"

[bash]$ pg_restore -v \  
    -d "postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>/development_template" \  
    /path/to/pg_dump_file

[bash]$ psql -X "postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>"   
    -c "VACUUM (ANALYZE, INDEX_CLEANUP ON)"

Notes on the commands above:

- We use **psql** to run a few commands. This can be done using any **PostgreSQL** CLI or GUI tool
- The database we created is named: `development_template` — we’ll use this in later steps
- The 3rd step requires the **pg_dump** output file (`/path/to/pg_dump_file`)
- We run the `VACUUM` command after **pg_restore** in line with **PostgreSQL** best practices
- The length of time to create this database depends on the size of you database image and your storage speed (e.g. SATA vs SSD).

### Creating a database using DDL

Here is an example of creating the template database using **DDL** scripts:

[bash]$ psql -X postgres://postgres@<PG_HOST>  
... create users/grants needed by database ...

[bash]$ psql -X postgres://postgres@<PG_HOST> \  
    -c "CREATE DATABASE development_template OWNER <DB_USER>"

[bash]$ psql -X postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST> \  
    -f /path/to/ddl_file.sql

[bash]$ psql -X postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST> \  
    -c "VACUUM (ANALYZE, INDEX_CLEANUP ON)"

Notes on the commands above:

- We use **psql** to run a few commands. This can be done through any **PostgreSQL** CLI or GUI tool.
- The database we created is named: `development_template` — we’ll use this in later steps
- The 3rd step requires the **DDL** file: `/path/to/ddl_file.sql`
- We run the `VACUUM` command after the database creation in line with **PostgreSQL** best practices
- The length of time to create this database depends on the size of your **DDL** script and your storage speed (e.g. SATA vs SSD).

If your **DDL** was schema only and you have scripts/tools to generate data you would insert that step prior to the `VACUUM` command.

### Results

Whichever option you choose, the end result is a database created in **PostgreSQL** (with an identifier we’ll call `development_template`) that can be used as a _Template_ for all future per-branch databases. This database should NOT ever be used by your application. It’s entire purpose is to be a _Template_ database for per-branch database creation.

To see how _Copy-on-Write_ saves on disk storage we’ll check disk usage after all database creation steps.

It is also worth noting how long it took to create our _Template_ database. We’ll then have a base line to compare against time required to create a per-branch database.

For our testing we used the **pg_dump** and **pg_restore** method defined above. We encapsulated the four steps into a _bash_ script so we could collect timing information of the full restore and vacuum process:

[bash]$ time ./make-template.sh   
real    17m46.941s  
user    4m17.327s  
sys     0m47.121s

The **pg_dump** database image file used in the example was 12 GB. This format uses compressed data therefore the resulting database will be much larger.

To view the amount of disk that **PostgreSQL** is using, we will need to know how big our `development_template` database is on disk to start.

First we want the OID of the database:

[bash]$ psql postgres://postgres@<PG_HOST

postgres@postgres.postgres=#   
SELECT oid, datname FROM pg_database WHERE datname = 'development_template'; 

      oid | datname   
----------+---------------------   
 40916095 | development_template

Then we can see how big the database directory is on disk. Navigate to the `_<PG_INSTALL>_` directory and get the size of the _base/40916095_ directory:

Example:

[bash]$ cd <PG_INSTALL>   
[bash]$ du -h base/40916095   
90G base/40916095

In our example, the `development_template` is 90 GB once uncompressed.

Unfortunately, _C_opy-on-Write filesystems lie to the `du` command. So we’ll need to use `df` instead to see how much actual storage is being used by the filesystem where the **PostgreSQL** server files are installed. We’ll get the value after each database branch step. Note that we very specifically do NOT use a`-h` option for `df` in case the size change is less than a GB.

[bash]$ df <PG_INSTALL>   
Filesystem      1K-blocks     Used Available Use% Mounted on  
/dev/nvme1n1p1 1000203264 95733548 900743812  10% <PG_INSTALL mount>

Our filesystem is currently using 95733548 KB of storage (about 91 GB).

We’ll revisit these numbers in future steps to see how our storage changes.

## Branching mechanics and per-branch database setup

At this point, we have a running **PostgreSQL** server with a _Template_ database. The next step is to setup a branch / worktree and its matching per-branch database.

We will use **Git** and **Git Worktrees** in our examples, so we need to start with a **Git** repository:

[bash]$ cd $HOME   
[bash]$ mkdir myproject.git   
[bash]$ cd myproject.git  
[bash]$ git init   
[bash]$ echo "hello" > README.md   
[bash]$ git add README.md   
[bash]$ git commit -m'* Initialize our repository'

Now we have a **Git** repository with a single branch: **main** and a single file `README.md`. You can obviously use an existing repository instead of creating a new one.

We also want to setup a directory for **Git Worktree** to hold our branches:

[bash]$ cd $HOME  
[bash]$ mkdir ~/myproject-worktrees

The next step is to create a database for the **main** branch using our `development_template` database:

[bash]$ psql -X postgres://postgres@<PG_HOST> \   
    -c CREATE DATABASE <DB_NAME> WITH TEMPLATE development_template STRATEGY=FILE_COPY OWNER <DB_USER>

Notes on the command above:

- Change `_<DB_NAME>_` to the name you want for your new database. We suggest you use the name of your branch in this case _main_
- Change the `_<DB_USER>_` to the database user that should own the new database. This requires the user and appropriate grants already exist
- We are using `CREATE DATABASE ... WITH TEMPLATE` to tell **PostgreSQL** to use a template
- We tell **PostgreSQL** to use the `development_template` as the source template
- We tell **PostgreSQL** to use the `FILE_COPY` strategy to create the database. With our **PostgreSQL** configuration and _Copy-on-Write_ file system this will use shared file storage

We now have two databases in our server:

- **development_template**
- **main**

## Storage consumption and timing

Since we created another database, we’ll look at the filesystem disk usage again.

First get the OID:

[bash]$ psql postgres://postgres@<PG_HOST>

postgres@postgres.postgres=#   
SELECT oid, datname FROM pg_database WHERE datname = 'main';  
      oid | datname   
----------+-----------   
 49176495 | main

We can’t depend on the `du` command to see how much storage is actually used. The `du` command will see the effective size rather than its actual consumption:

[bash] cd $PG_INSTALL   
[bash]$ du -h 49176495  
90G 59948153

It shows the new database is consuming 90 GB, just like the _development_template_.

However, when we use the `df` command o see how much actual storage has been consumed:

[bash]$ df <PG_INSTALL>   
Filesystem      1K-blocks     Used Available Use% Mounted on  
/dev/nvme1n1p1 1000203264 97412668 899129908  10% <PG_INSTALL mount>  

We are currently using 97412668 K of storage (about 92 GB). The actual storage change was from 95733548 KB to 97412668 KB (1679120 KB), which is only about 1.6 GB.

Even though we have two 90 GB databases, the second database is only consuming about 1.6 GB of storage.

The timing of the database creation:

[bash]$ time psql -X postgres://postgres@<PG_HOST> \   
    -c CREATE DATABASE main WITH TEMPLATE development_template STRATEGY=FILE_COPY OWNER <DB_USER>   
real 0m2.464s  
user 0m0.786s  
sys  0m0.659s

Loading the template took over 17 minutes. Creating a new database from that template took just over 2 seconds!

## Application configuration

Next, we need to configure the application in the `myproject.git` directory (on the **main** branch) to connect to the database named: _main_. This is VERY simple if you are using environment variables in your application. With an example environment variable in your application of `_DB_CONFIG_DSN_` you would run your application as something like:

[bash]$ DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>:<PG_PORT>/main ./run/my/application

Notice the use of _main_ at the end of the DSN. That identifies the specific database. No matter which branch we’re using the `DB_USER`, `DB_PASSWORD`, `PG_HOST` and `PG_PORT` values are always the same. Only the database identifier at the end changes.

If you are using another method to configure your application you need to make the equivalent changes to your application configuration.

After this step you should have an instance of the application running code on the **main** branch and the server is connected to the database named _main_.

Next, we’ll see how to setup and configure another branch and its own per-branch database.

## Setting up another branch and database

We’re going to create a new branch from **main** that we’ll call: **branch_one**:

[bash]$ cd ~/myproject.git   
[bash]$ git worktree add ~/myproject-worktrees/branch_one -b branch_one main

This command will:

- Create a new branch: **branch_one**
- Create a new directory: `~/myproject-worktrees/branch_one`
- Checkout the new branch **branch_one** as a **Git Worktree** in the`~/myproject-worktrees/branch_one` directory

If you are using another source code tool, you just need to make sure each branch is checked out in separate directories as we do with **Git**.

Next we create a new database for the new branch:

[bash]$ psql -X postgres://postgres@<PG_HOST> \   
    -c CREATE DATABASE branch_one WITH TEMPLATE development_template STRATEGY=FILE_COPY OWNER <DB_USER>

Notes:

- Change `<PG_HOST>` to the database host
- Change `<DB_USER>` to be the database user that needs to own the database. The user and appropriate grants need to already exist
- We used _branch_one_ as the database name (same as our branch)

We now have three databases in our server:

- **development_template**
- **main**
- **branch_one**

Since we created another database we’ll look at disk usage again.

[bash]$ df <PG_INSTALL>   
Filesystem      1K-blocks     Used Available Use% Mounted on   
/dev/nvme1n1p1 1000203264 110911812 885711708  12% <PG_INSTALL mount>

We are currently using 110911812 KB of storage (about 105 GB). But there was an actual storage change from 97412668 KB to 110911812 KB (13499144 KB), about 12.8 GB.

Even though we have what appears to be 270 GB worth of databases (three 90 GB databases), the two new databases only added a combined total of about 12.8 GB of storage.

The timing of the database creation:

[bash]$ time psql -X postgres://postgres@<PG_HOST> \   
    -c CREATE DATABASE branch_one WITH TEMPLATE development_template STRATEGY=FILE_COPY OWNER <DB_USER>   
real    0m4.250s  
user    0m1.588s  
sys     0m0.803s

Just over four seconds.

Next we can run the server from the **branch_one** checkout and make it point at the _branch_one_ database.

[bash]$ cd ~/myproject-worktrees/branch_one  
[bash]$ DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<DB_HOST>:<DB_PORT>/branch_one ./run/my/application

We now have an instance of the application running the code from the **branch_one** branch and connecting to the _branch_one_ database.

There are a couple of things to be aware of with this setup:

1. **PostgreSQL** has restrictions on database names

- These restrictions are defined in the [PostreSQL Identifier and Key Words](https://www.postgresql.org/docs/18/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS) documentation
- It is convenient if you can name your branch and your database the same name. If you can’t do that, you can easily name them different things. Change the branch name, database name or both accordingly

2. Depending on how your application is run, you may only be able to run one instance of your application at a time (for example — it binds a fixed port).

- If you can only run one instance at a time, you can do something like this to branch switch:

[bash]$ cd ~/myproject-worktrees/branch_one/

[bash]$ DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>:<PG_PORT>/branch_one ./run/my/application  
... run for as long as needed, until time to switch branches  
Press [CTRL+C] or otherwise stop your application

[bash]$ cd ../myproject.git/

[bash]$ DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>:<PG_PORT>/main ./run/my/application  
... run for as long as needed

## A note about other Git and Worktree operations

Using **Git Worktrees** you will certainly need to use quite a few other **Git** operations on your worktrees:

- Viewing Commit History
- Adding/Committing changes
- Viewing Diffs
- Getting upstream updates
- Merging (to/from) other branches
- etc.

Since we are using standard **Git/Git Worktrees** all **Git/Git Worktree** operations continue to work as-is, our workflow has zero impact on those tools. See the community **Git** documentation if you need help using **Git/Git Worktree**.

## Utilizing database clients (CLI and GUI)

Connecting to a per-branch database with a CLI or GUI is simply a matter of connecting with the correct per-branch database information.

If you use a GUI tool you generally only need the main database credentials to connect and you can then view/manage all databases. Good recommendations:

- [pgAdmin](https://www.pgadmin.org/)
- [DBeaver](https://dbeaver.io/)

If you use a CLI tool you use the same connect information as your application to connect to a specific database. Example:

[bash]$ psql 'postgres://$DB_USER$:$DB_PASSWD@$PG_HOST:$PG_PORT/$DB_NAME'

## Managing database drift

We created our _Template_ database at a certain point in time and it is not getting updated. We will use this _Template_ to create all per-branch databases. This can lead to a few issues from database drift. Where _database drift_ is changes to either the database you exported for your template -OR- changes to your per-branch database.

The issues:

1. Your database source could change structure

If the database source changed structure (and you used **pg_dump** or updated your **DDL** accordingly), these changes are not reflected in any of your branch databases, which can cause huge problems.

2. Your database source could have meaningful data changes

You will definitely have temporal drift in your data— no columns using date/time values will be any newer than the date your _Template_ database was created other than rows in your per-branch database created (by you) during your development work

Additionally, general data changes made to your source system could cause issues in a newer branch (e.g. a value in a column that is no longer used/expected)

3. Your per-branch database(s) may have schema or data changes

You may be making changes to your per-branch databases and need to undo these changes. This is particularly useful if you are experimenting with code that modifies the structure of your database

To address these issues we need to be able to:

- Replace / Update the _Template_ database
- Replace / Update a per-branch database

## Replace / Update the template database

Since every new database is created from the _Template_ you will want to update the _Template_ with some degree of frequency. Some events that might trigger a desire to update the _Template_:

- The source schema has changed (either your database dump or **DDL** file will have schema changes)
- Significant data change that can lead to code problems In this case your code has assumptions that your database data doesn’t obey since it’s out of date
- Temporal issues (i.e. you need data with newer dates)

### Replacing a database using p`g_dump and pg_restore`

The process to replace a _Template_ is straight forward and very similar to the original creation of the _Template_. We only need to add a `DROP DATABASE` command to the process we used on the initial creation.

Here is an example of creating the template database using a **pg_restore** from a **pg_dump** file:

[bash]$ psql -X postgres://postgres@<PG_HOST> \  
    -c "DROP DATABASE IF EXISTS development_template OWNER <DB_USER>"

[bash]$ psql -X postgres://postgres@<PG_HOST> \  
    -c "CREATE DATABASE development_template OWNER <DB_USER>"

[bash]$ pg_restore -v \  
    -d postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>/development_template \  
    /path/to/pg_dump_file

[bash]$ psql -X postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST> \  
    -c "VACUUM (ANALYZE, INDEX_CLEANUP ON)"

Notes on the commands above:

- We use **psql** to run a few commands. This can be done through any **PostgreSQL** CLI or GUI tool
- We added a `DROP DATABASE` command to drop the old _Template_ database
- We skip user/grant creation as they already exist
- The rest of the process is unchanged from initial creation

### **Replacing a database using** `**DDL**`

Here is an example of creating the template database using **DDL** scripts:

[bash]$ psql -X postgres://postgres@<PG_HOST> \  
    -c "DROP DATABASE IF EXISTS development_template OWNER <DB_USER>"

[bash]$ psql -X postgres://postgres@<PG_HOST> \  
    -c "CREATE DATABASE development_template OWNER <DB_USER>"

[bash]$ psql -X postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST> \  
    -f /path/to/ddl_file.sql

[bash]$ psql -X postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST> \  
    -c "VACUUM (ANALYZE, INDEX_CLEANUP ON)"

Notes on the commands above:

- We use **psql** to run a few commands. This can be done through any **PostgreSQL** CLI or GUI tool
- We add a `DROP DATABASE` command to drop the old _Template_ database
- We skip user/grant creation as they already exist
- The rest of the process is unchanged from initial creation

### Increased disk consumption on replace of the Template database

When we replace the _Template_ database, any per-branch databases will no longer be able to share blocks with the _T_emplate. The replacement of the _Template_ will therefore introduce extra storage consumption. We can confirm that:

[bash]$ df <PG_INSTALL>   
Filesystem      1K-blocks      Used Available Use% Mounted on  
/dev/nvme1n1p1 1000203264 205124044 791913252  21% <PG_INSTALL mount>

Our storage has increased to 205124044 KB (about 195 GB).

Since this process can eat up a lot of disk (at least temporarily), you will want to plan out the times when you trigger this process if you are short on disk. Try to do this when you don’t have a low number of per-branch databases (zero optimally!).

Later in the doc we will show you how to refresh your per-branch databases to make them use the new _Template_ database. This will reduce the disk consumption back to the original levels.

### Timing of Template database replacement

The process of replacing the _Template_ database is using a new data set and not creating from a template. Therefore, just like the initial _Template_ creation, this process takes significant time (about 17 minutes). Once again, we’ve wrapped the steps into a _bash_ script to collect timing:

[bash]$ time ./replace-template.sh   
real    17m28.671s  
user    4m15.063s  
sys     0m49.585s

## Replace / Update a per-branch database

There are a few reasons why you might want to replace / update your per-branch database:

- You want to revert any changes to your database since you cloned from the `development_template`
- You have replaced the `development_template` database and are now consuming increased disk. By replacing you per-branch database you will reduce disk consumption

The process to replace the per-branch database from the `development_template` is straightforward:

- Drop the per-branch database
- Recreate from the _Template_

[bash]$ psql -X postgres://postgres@<PG_HOST> \   
    -c "DROP DATABASE IF EXISTS <DB_NAME>" \   
    -c CREATE DATABASE <DB_NAME> WITH TEMPLATE development_template STRATEGY=FILE_COPY OWNER <DB_USER>

Notes on command above:

- We use **psql** to run the commands. This can be done through any **PostgreSQL** CLI or GUI tool
- We drop the old version of the database
- Change `<PG_HOST>` to the host of the database server
- Change `_<DB_NAME>_` to the name of the branch database you’re updating
- Change the `_<DB_USER>_` to the database user that should own the new database
- We are using `CREATE DATABASE ... WITH TEMPLATE` to tell **PostgreSQL** to use a template
- We tell **PostgreSQL** to use the `development_template` as the source template
- We tell **PostgreSQL** to use the `FILE_COPY` strategy to create the database. With our **PostgreSQL** configuration and _Copy-on-Write_ file system this will once again use shared file block storage

Since we have dropped the per-branch database and re-created it, any changes you had in your per-branch database that were not in the `development_template` are lost. This is precisely the desired outcome for the “revert” use case.

If you were in a non-shared, increased disk consumption mode and refresh all of your databases from the _Template_ you will reduce your consumption as the databases are now sharing blocks again.

NOTE: You decide when to run the process on a per-branch basis. If there is a branch with data that you don’t want to lose (e.g. test data setup), you should defer running this process on the branch until the timing is better.

### **Storage consumption and timing**

Since this process is the same as the creation of per-branch database with an added _DROP_ command the storage consumption and timing numbers are about the same, a few seconds:

[bash]$ time ./refresh-branch-db  
real 0m2.778s  
user 0m0.022s  
sys  0m0.013s

After refreshing both per-branch databases the consumed storage is reduced:

[bash]$ df <PG_INSTALL>   
Filesystem      1K-blocks     Used Available Use% Mounted on  
/dev/nvme1n1p1 1000203264 95598036 901110460  10% <PG_INSTALL mount>

Consumption has gone back to 95598036 KB (about 90 GB).

## Removing obsolete databases

When you are done with a branch (e.g. on branch merge) you’ll want to remove the existing database and its Worktree. The steps:

[bash]$ psql -X postgres://<PG_HOST>/ \  
  -c "DROP DATABASE IF EXISTS <DB_NAME>"

[bash] cd ~/myproject.git

[bash] git worktree remove <branch_name> [--force]

[bash] git branch -D <branch_name>

Notes on commands above:

- We use **psql** to run a the first command. This can be done through any **PostgreSQL** CLI or GUI tool
- We drop the per-branch database
- We switch to the _main_ repository checkout
- We remove the Worktree (optionally with `--force` if there are non-committed files)
- We remove the local branch

## Optional: Automated configuration

As shown above, changing your database configuration and running your branch server can be done as easily as:

[bash]$ cd ~/myproject-worktrees/branch_one/

[bash]$ DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>:<PG_PORT>/branch_one ./run/my/application  
... run for as long as needed, until time to switch branches  
Press [CTRL+C] or otherwise stop your application

[bash]$ cd ../myproject.git/

[bash]$ DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>:<PG_PORT>/main ./run/my/application  
... run for as long as needed

There is an easier way. We use [**direnv**](https://direnv.net/) to automate the setting of the `DB_CONFIG_DSN` environment variable. We add an `.envrc` file in each Worktree that contains something like:

DB_CONFIG_DSN=postgres://<DB_USER>:<DB_PASSWD>@<PG_HOST>:<PG_PORT>/<DB_ID>

Where `<DB_ID>` is the database identifier for that branch/worktree (e.g. _main_, _branch_one_, _branch_two_, etc.)

With **direnv** configured properly and the `.envrc` files setup you `cd`into the branch directory and **direnv** will automatically set `DB_CONFIG_DSN` in your environment. Then you simply run your app from the project directory:

[bash]$ cd ~/myproject-worktrees/branch_one/  
# on cd, direnv will load the `.envrc` file into your environment  
# automatically setting DB_CONFIG_DSN to connect to the branch_one database

[bash]$ ./run/my/application  
  ... run for as long as needed, until time to switch branches  
Press [CTRL+C] or otherwise stop your application

[bash]$ cd ../myproject.git/  
# on cd, direnv will load the `.envrc` file into your environment  
# automatically setting DB_CONFIG_DSN to connect to the main database

[bash]$ ./run/my/application  
   ... run for as long as needed

## Revisiting our concerns with per-branch databases

We previously discussed some concerns users might have with per-branch databases, let’s revisit those now.

1. Database creation/setup concerns

- How easy is it to create a new per-branch database?
- How long does it take to create a new per-branch database?

As we have demonstrated, creation is pretty simple: run a few `psql` commands. You can quickly turn into scripts to make even easier.

How long does it take? Usually just a second or two. The only slow operation is refreshing the _Template_ database, which is done at whatever frequency you need/want.

2. Disk consumption concerns

- If the database we use is large it becomes untenable pretty quickly to keep around numerous copies of the database

With a 90GB database image, having multiple copies around would normally consume disk at a ferocious rate. But as we showed, per-branch databases consume only a fraction of that.

Granted, when you do a _Template_ replacement your storage increases substantially, but this is a temporary state.

3. Application Configuration concerns

- How is a branch of code configured to connect to a specific per-branch database?
- How does this configuration get changed when branch switching?

While we can’t answer this one definitively for you as it depends on your application and how it is configured. Many applications use Environment Variables for configuration. In that setup configuration is done by simply changing the Environment Variable on branch switch.

If you aren’t using Environment Variables, you need to craft your own solution, but we think it should be fairly easy.

We also recommend the use of **direnv** to automate this configuration.

4. Raw Database Access concerns

- How is a specific per-branch database accessed using a CLI or GUI database tool?

Each branch has identical host, port, user, and password settings. The only unique variant is the database name. We try very hard to use the same name for the branch as the database identifier. This makes database access trivial from either a CLI or GUI.

5. Database cleanup concerns

- How are obsolete per-branch databases cleaned up once they are no longer required?

Removing a per-branch database is as simple as executing a `DROP TABLE` command.

## Conclusion

We hope we’ve provided you a customizable foundation to improve your work environment through the use of per-branch databases. It has saved us a tremendous amount of hours and numerous headaches. Hopefully, you’ll get something out of it.

Thanks for reading!

## About Us

This document is the product of work of the [Axial](http://www.axial.net/) Engineering team:

- David Grimes
- Markus Jonsson
- Sorenson Stallings
- Bob Ternosky
- Zach Werheim

Founded in 2009, Axial brings together lower middle market M&A professionals and American small business owners on the industry’s most trusted and advanced online M&A platform.