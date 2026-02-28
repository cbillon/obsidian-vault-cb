---
link: https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl
byline: yukaty
site: DEV Community
date: 2024-11-05T22:03
excerpt: Ever wondered how Netflix suggests movies you might like, or how Spotify creates personalized...
twitter: https://twitter.com/@yukaty_ca
tags:
  - postgresql
  - pgvector
slurped: 2026-02-23T17:05
title: "Part 1: Setup with PostgreSQL and pgvector"
---
[source github](https://github.com/yukaty/vector-search-api/blob/main/scripts/load_data.py)

Ever wondered how Netflix suggests movies you might like, or how Spotify creates personalized playlists? These AI-powered features often use **vector similarity search** under the hood. In this series, we'll build our own AI search engine using PostgreSQL with pgvector!

Let's get started...🐢

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#contents)Contents

- [Project Overview](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#project-overview)
- [What is Vector Search?](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#what-is-vector-search)
- [Step-by-Step Setup](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#stepbystep-setup)
- [Troubleshooting Tips](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#troubleshooting-tips)
- [Quick Preview](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#quick-preview)
- [What's Next?](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#whats-next)

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#project-overview)Project Overview ✨

We'll build a search engine to find similar content based on meaning, not just matching keywords. This is the same type of technology behind:

- GitHub Copilot's code suggestions
- Spotify's song recommendations
- Netflix's movie recommendations

While various tools and services support similar functionality, we'll use `pgvector` to implement vector similarity search within postgreSQL.

In Part 1, we'll set up the database infrastructure. In Part 2, we'll implement the search functionality using OpenAI's embeddings.

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#what-is-vector-search)What is Vector Search? 🔎

When AI processes content (text, code, or images), it creates a special list of numbers called **embedding**. Think of it as a smart summary that captures the content's meaning. Similar content will have similar numbers, making it easy to find related items.

If you're not familiar with Machine Learning, don't worry! You can easily obtain these embeddings from AI APIs like OpenAI, even without deep AI knowledge.

**pgvector** helps us efficiently store and search these embeddings as **vectors** in PostgreSQL.

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#stepbystep-setup)Step-by-Step Setup 👣

Make sure you have Docker Desktop installed on your computer.

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#project-structure)Project Structure

```
vector-search/
├── compose.yml
└── postgres/
    └── schema.sql
```

 

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#1-create-raw-composeyml-endraw-)1. Create `compose.yml`

```
services:
  db:
    image: pgvector/pgvector:pg17 # PostgreSQL with pgvector support
    container_name: pgvector-db
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: example_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./postgres/schema.sql:/docker-entrypoint-initdb.d/schema.sql

volumes:
  pgdata: # Stores data outside the container to ensure persistence
```

 

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#2-define-database-schema)2. Define Database Schema

Create `postgres/schema.sql`:  

```
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create sample table
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    item_data JSONB,
    embedding vector(1536) -- vector data
);
```

 

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#3-start-the-database)3. Start the Database

Run Docker Compose to build and start the PostgreSQL container with pgvector.  

```
docker compose up --build
```

 

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#4-verify-the-setup)4. Verify the Setup

Connect to PostgreSQL:  

```
docker exec -it pgvector-db psql -U postgres -d example_db
```

 

Check if everything is set up correctly:  

```
-- Check installed extensions
\dx

-- Check table creation
\dt

-- Check table structure
\d items
```

 

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#troubleshooting-tips)Troubleshooting Tips 🛠️

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#error-port-5432-already-in-use)Error: Port 5432 already in use

Change the port in `compose.yml` to 5433 or another free port.  

```
  ports:
    - "5433:5432"
```

 

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#database-not-initializing-properly)Database not initializing properly

Remove the volume and restart.  

```
  docker-compose down -v    # Remove existing volume
  docker-compose up --build # Start fresh
```

 

### [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#still-not-sure-whats-wrong)Still not sure what's wrong?

Check the container logs.  

```
  docker compose logs db
```

 

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#quick-preview)Quick Preview 👀

Here's a quick preview of how we'll query similar items in Part 2:  

```
-- Find items similar to a specific vector
SELECT id, name, item_data
FROM items
ORDER BY embedding <-> '[0.1, 0.2, ...]'::vector
LIMIT 5;
```

 

Replace [0.1, 0.2, ...] with an actual vector from AI models.

---

## [](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl#whats-next)What's Next? 💭

We'll dive into the following topics:

- Understand what embeddings are and how they work
- Generate embeddings using OpenAI
- See how vector search works in practice

Stay tuned! 🚀

Spot any mistakes or have a better way? Please leave a comment below! 💬