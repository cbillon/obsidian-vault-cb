---
link: https://developer.upsun.com/posts/insights/why-you-should-replace-postgresql-with-git-for-your-next-project
site: Upsun Developer
excerpt: Explore how Git's internal architecture makes it a surprisingly capable
  database alternative. Learn Git's data structures through a practical todo app
  implementation that might make you reconsider your database choices.
slurped: 2026-08-25T09:54
title: Why you should replace PostgreSQL with Git for your next project - Upsun
  Developer
---

Every developer knows the pain of choosing the right database for their project. PostgreSQL offers robust relational features, but what if there was a database you’re already using every day that could handle your data storage needs? Meet Git – the version control system that’s been hiding its database capabilities in plain sight. Before you close this tab thinking we’ve lost our minds, consider this: Git provides built-in versioning, handles concurrent access, supports atomic transactions (commits), and offers lightning-fast data retrieval. It even comes with its own query language (Git commands) and built-in backup system (distributed repositories). While this approach isn’t suitable for production applications, exploring Git’s internal architecture reveals fascinating insights into how modern databases work. Let’s build a todo application using Git as our storage layer to understand these core concepts.

## Git’s data model: The foundation

Git organizes data using four fundamental types:

- **Blobs**: Raw data storage (equivalent to table rows)
- **Trees**: Hierarchical organization (like directory structures)
- **Commits**: Transaction records with metadata
- **References**: Pointers to specific data states (like table indexes)

This structure makes Git more similar to hierarchical databases like Apache ZooKeeper than traditional relational systems. Let’s experiment with these concepts by building our own “database”.

## Setting up your Git database

### Working with blobs: Your data records

Blobs store raw data – think of them as individual database records. Unlike traditional databases, blobs are content-addressable, meaning their unique identifier is derived from their content. Create a blob containing data:

Git stores this blob in its object database using the hash as the filename:

The file contains compressed, binary data. Git provides tools to retrieve the original content:

### Trees: Organizing your data structure

Trees group related blobs together, similar to how database tables organize related records. Create a tree by specifying which blobs it should contain:

Using our existing blob:

Examine the tree structure:

The tree now references our blob with a meaningful name.

### Commits: Transaction records with metadata

Commits wrap trees in transactional context, providing metadata about when and why changes occurred:

Create our first transaction record:

Inspect the commit metadata:

Commits automatically include comprehensive metadata:

- Tree reference (data snapshot)
- Author and committer information
- Timestamp for audit trails
- Descriptive message

### References: Making data discoverable

Without references, commits become “dangling” and get garbage collected. References act like database indexes, making specific data states discoverable:

This creates a “branch” reference pointing to our commit. Git uses different reference namespaces (`.git/refs/heads` for branches, `.git/refs/tags` for tags) similar to database schemas. You can now query your “database”:

## Building a todo application with Git

Now let’s apply these concepts to build a functional todo application, demonstrating how Git’s architecture compares to traditional database operations.

### Defining our data schema

Our todo application needs a simple data model:

- **Task Title**: The task description
- **Task Status**: Current state (todo/done)

Using Git’s architecture, we’ll store each field as a separate blob and organize them in trees, with commits representing state changes.

### Creating task data

Create blobs for task titles:

Create status value blobs:

### Organizing data with trees

Create a task record by combining title and status blobs in a tree:

Verify the task structure:

### Creating transactions with commits

Commit the task to create a permanent transaction record:

Create a reference to make the data discoverable:

### Querying your Git database

View the complete transaction history:

## Why Git makes sense for specific use cases

While this exploration started as a thought experiment, Git offers genuine advantages for certain applications:

- **Built-in audit trails**: Every change includes timestamp and author information
- **Atomic transactions**: Commits ensure data consistency
- **Distributed architecture**: Multiple nodes can sync data changes
- **Content addressing**: Automatic deduplication and integrity checking

## Real-world applications at Upsun

At Upsun, we leverage Git’s database-like properties for specific scenarios where its strengths outweigh traditional database benefits. For developer-facing configuration management, Git provides:

- Automatic versioning for all configuration changes
- Distributed synchronization across development environments
- Native integration with existing developer workflows
- Built-in rollback capabilities through commit history

However, Git has significant limitations as a general-purpose database:

- Limited concurrent access (worse than SQLite)
- No complex query capabilities
- Poor performance with large datasets
- No built-in indexing for non-content searches

## Start building with proper databases on Upsun

While Git makes an interesting database alternative for specific use cases, your production applications deserve better. Upsun provides managed [PostgreSQL, MySQL, and other database services](https://docs.upsun.com/add-services.html) with:

- Automatic scaling and performance optimization
- Built-in backup and disaster recovery
- Multi-environment support for development and staging

[Create a free Upsun account](https://upsun.com/) to deploy your applications with proper database infrastructure that scales with your needs.