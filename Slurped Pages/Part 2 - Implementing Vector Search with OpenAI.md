---
link: https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh
byline: yukaty
site: DEV Community
date: 2024-11-07T03:12
excerpt: In Part 1, we set up PostgreSQL with pgvector. Now, let's see how vector search actually works....
twitter: https://twitter.com/@yukaty_ca
tags:
  - postgresql
  - openapi
slurped: 2026-02-23T17:07
title: "Part 2: Implementing Vector Search with OpenAI"
---

In [Part 1](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl), we set up PostgreSQL with pgvector. Now, let's see how vector search actually works. 🧠

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#contents)Contents

- [Prerequisites](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#prerequisites)
- [Understanding Vector Search](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#understanding-vector-search)
- [Project Setup](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#project-setup)
- [Exploring Vector Search](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#exploring-vector-search)
- [Working with JSON and Vectors](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#working-with-json-and-vectors)
- [Performance Tips](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#performance-tips)
- [Resources](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#resources)

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#prerequisites)Prerequisites 📋

- Completed Part 1 setup for pgvector
- OpenAI API key

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#understanding-vector-search)Understanding Vector Search 🌐

A vector is a list of numbers that represents position or direction:  

```
2D Vector: [x, y]     📍 Like coordinates on a map
3D Vector: [x, y, z]  🎲 Like a point in 3D space
```

 

When AI processes content, it creates special vectors called "embeddings" (1536 dimensions) to represent meaning. These embeddings are stored in the database, allowing us to perform similarity search:  

```
📘 "How to use Docker"
    [0.23, 0.45, 0.12, ...]  # 1536-dimensional vector
📗 "Docker tutorial"          
    [0.24, 0.44, 0.11, ...]  🤝 Very Similar! (Distance: 0.2)
📕 "Chocolate cake recipe"    
    [0.89, 0.12, 0.67, ...]  🚫 Not Related! (Distance: 0.9)
```

 

- Vectors let AI understand similarity mathematically
- Vector search finds similar content by comparing distances
- pgvector stores embeddings efficiently
- Works across any language (it's all just numbers!)

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#project-setup)Project Setup ⚙️

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#updated-project-structure)Updated Project Structure

```
vector-search/
├── .env
├── compose.yml
├── requirements.txt
├── postgres/            # Part 1: Database setup
│   └── schema.sql
└── scripts/             # New: Data loading
    ├── Dockerfile
    └── load_data.py
```

 

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#1-set-up-openai-api)1. Set Up OpenAI API

Create `.env`:  

```
OPENAI_API_KEY=your_api_key  # Get from platform.openai.com
```

 

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#2-create-data-loading-script)2. Create Data Loading Script

Create `scripts/load_data.py` to fetch books and generate embeddings:  

```
import openai

client = openai.OpenAI()

def get_embedding(text: str):
    """Generate embedding using OpenAI API"""
    response = client.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding

def load_books_to_db():
    """Load books with embeddings into PostgreSQL"""

    # 1. Fetches books from Open Library API
    books = fetch_books()

    for book in books:
        # 2.Create text description for embedding
        description = f"Book titled '{book['title']}' by {', '.join(book['authors'])}. "
        description += f"Published in {book['first_publish_year']}. "
        description += f"This is a book about {book['subject']}."

        # 3. Generate embedding using OpenAI
        embedding = get_embedding(description)

        # 4. Stores books and embeddings in PostgreSQL
        store_book(book["title"], json.dumps(book), embedding)
```

 

Full source code is available on [GitHub](https://github.com/yukaty/vector-search-api/blob/main/scripts/load_data.py) 🐙

Also create `requirements.txt` and `scripts/Dockerfile`.

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#3-update-docker-compose)3. Update Docker Compose

Update `compose.yml` to add the data loader:  

```
services:
  # ... existing db service from Part 1

  data_loader:
    build:
      context: .
      dockerfile: scripts/Dockerfile
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/example_db
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    depends_on:
      - db
    command: python load_data.py
```

 

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#4-load-sample-data)4. Load Sample Data

```
docker compose up --build
```

 

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#exploring-vector-search)Exploring Vector Search 🔦

First, connect to the database:  

```
docker exec -it pgvector-db psql -U postgres -d example_db
```

 

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#inspecting-embeddings)Inspecting Embeddings

Check what the vectors look like:  

```
-- View first 5 dimensions of an embedding
SELECT
    name,
    (replace(replace(embedding::text, '[', '{'), ']', '}')::float[])[1:5] as first_dimensions
FROM items
LIMIT 1;
```

 

💡 Each embedding from OpenAI's model:

- Has 1536 dimensions
- Contains values between -1 and 1
- Represents text meaning mathematically
- Outputs in `[...]` format, which needs to be converted to PostgreSQL's `{...}` array format for array operations

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#finding-similar-books)Finding Similar Books

Search for books about web development:  

```
WITH web_book AS (
        SELECT embedding FROM items WHERE name LIKE '%Web%' LIMIT 1
)
SELECT 
    item_data->>'title' as title,
    item_data->>'authors' as authors,
    embedding <=> (SELECT embedding FROM web_book) as similarity
FROM items
ORDER BY similarity
LIMIT 3;  -- Returns the 3 most similar books
```

 

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#working-with-json-and-vectors-%EF%B8%8F)Working with JSON and Vectors ⚡️

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#json-operators)JSON Operators

Use `->>` to extract text value from a JSON field:  

```
-- Get title from the 'item_data' JSON column
SELECT item_data->>'title' FROM items;
```

 

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#vector-search-operators)Vector Search Operators

pgvector supports multiple distance functions. Here are the two most commonly used operators.

#### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#l2-distance-raw-ltgt-endraw-)L2 Distance: `<->`

Measures straight-line (Euclidean) distance between vectors:  

```
-- Find similar books using L2 distance
SELECT 
    name,
    embedding <-> (
        SELECT embedding FROM items WHERE name LIKE '%Web%' LIMIT 1
    ) as distance
FROM items
ORDER BY distance
LIMIT 3;
```

 

#### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#cosine-distance-raw-ltgt-endraw-)Cosine Distance: `<=>`

Measures angle-based (cosine) distance between vectors:  

```
-- Find similar books using Cosine distance
SELECT 
    name,
    embedding <=> (
        SELECT embedding FROM items WHERE name LIKE '%Web%' LIMIT 1
    ) as distance
FROM items
ORDER BY distance
LIMIT 3;
```

 

#### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#tips)💡 Tips

- OpenAI recommends `<=>` (Cosine distance) for their embeddings.
- Smaller distance means higher similarity.

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#performance-tips)Performance Tips 🚀

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#query-optimization)Query Optimization

Cache query vectors instead of subquerying:  

```
-- ❌ Inefficient: Subquery runs for every row
SELECT name, embedding <=> (
        SELECT embedding FROM items WHERE name LIKE '%Web%' LIMIT 1
) as distance
FROM items
ORDER BY distance
LIMIT 3;

-- ✅ Better: Query vector calculated once
WITH query_embedding AS (
        SELECT embedding FROM items WHERE name LIKE '%Web%' LIMIT 1
)
SELECT 
    name,
    embedding <=> (SELECT embedding FROM query_embedding) as distance
FROM items
ORDER BY distance
LIMIT 3;
```

 

### [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#indexing)Indexing

Choose an index based on your needs:  

```
-- Option 1: IVFFlat (Less memory, good for development)
CREATE INDEX ON items USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- Option 2: HNSW (Faster searches, more memory)
CREATE INDEX ON items USING hnsw (embedding vector_cosine_ops);
```

 

---

## [](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh#resources)Resources 🔗

- [pgvector](https://github.com/pgvector/pgvector)
- [OpenAI Embeddings](https://platform.openai.com/docs/guides/embeddings)

---

Hope this helps you build something cool. Feel free to drop a comment below! 💬