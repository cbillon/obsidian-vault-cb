---
link: https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop
byline: yukaty
site: DEV Community
date: 2024-11-12T21:03
excerpt: Part 1 covered PostgreSQL with pgvector setup, and Part 2 implemented vector search using OpenAI...
twitter: https://twitter.com/@yukaty_ca
tags:
  - postgresql
  - pgvector
  - ollama
slurped: 2026-02-23T17:09
title: "Part 3: Implementing Vector Search with Ollama"
---

[Part 1](https://dev.to/yukaty/setting-up-postgresql-with-pgvector-using-docker-hcl) covered PostgreSQL with pgvector setup, and [Part 2](https://dev.to/yukaty/getting-started-with-vector-search-part-2-3amh) implemented vector search using OpenAI embeddings. This final part demonstrates how to run vector search locally using Ollama! ✨

---

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#contents)Contents

- [Why Ollama?](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#why-ollama)
- [Setting Up Ollama with Docker](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#setting-up-ollama-with-docker)
- [Database Updates](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#database-updates)
- [Implementation](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#implementation)
- [Search Queries](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#search-queries)
- [Performance Tips](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#performance-tips)
- [Troubleshooting](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#troubleshooting)
- [OpenAI vs. Ollama](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#openai-vs-ollama)
- [Wrap Up](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#wrap-up)

---

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#why-ollama)Why Ollama? 🦙

Ollama allows you to run AI models locally with:

- Offline operation for better data privacy
- No API costs
- Fast response times

We'll use the `nomic-embed-text` model in Ollama, which creates 768-dimensional vectors (compared to OpenAI's 1536 dimensions).

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#setting-up-ollama-with-docker)Setting Up Ollama with Docker 🐳

To add Ollama to your Docker setup, add this service to `compose.yml`:  

```
services:
  db:
    # ... (existing db service)

  ollama:
    image: ollama/ollama
    container_name: ollama-service
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama

  data_loader:
    # ... (existing data_loader service)
    environment:
      - OLLAMA_HOST=ollama
    depends_on:
      - db
      - ollama

volumes:
  pgdata:
  ollama_data:
```

 

Then, start the services and pull the model:  

```
docker compose up -d

# Pull the embedding model
docker compose exec ollama ollama pull nomic-embed-text

# Test embedding generation
curl http://localhost:11434/api/embed -d '{
  "model": "nomic-embed-text",
  "input": "Hello World"
}'
```

 

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#database-updates)Database Updates 🐘

Update the database to store Ollama embeddings:  

```
-- Connect to the database
docker compose exec db psql -U postgres -d example_db

-- Add a column for Ollama embeddings
ALTER TABLE items
ADD COLUMN embedding_ollama vector(768);
```

 

For fresh installations, update `postgres/schema.sql`:  

```
CREATE TABLE items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    item_data JSONB,
    embedding vector(1536),        # OpenAI
    embedding_ollama vector(768)   # Ollama
);
```

 

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#implementation)Implementation 👾

Update `requirements.txt` to install the Ollama Python library:  

```
ollama==0.3.3
```

 

Here’s an example update for `load_data.py` to add Ollama embeddings:  

```
import ollama  # New import

def get_embedding_ollama(text: str):
    """Generate embedding using Ollama API"""
    response = ollama.embed(
        model='nomic-embed-text',
        input=text
    )
    return response["embeddings"][0]

def load_books_to_db():
    """Load books with embeddings into PostgreSQL"""
    books = fetch_books()

    for book in books:
        description = (
            f"Book titled '{book['title']}' by {', '.join(book['authors'])}. "
            f"Published in {book['first_publish_year']}. "
            f"This is a book about {book['subject']}."
        )

        # Generate embeddings with both OpenAI and Ollama
        embedding = get_embedding(description)                # OpenAI
        embedding_ollama = get_embedding_ollama(description)  # Ollama

        # Store in the database
        store_book(book["title"], json.dumps(book), embedding, embedding_ollama)
```

 

Note that this is a simplified version for clarity. Full source code is [here](https://github.com/yukaty/vector-search-api/blob/main/scripts/load_data.py).

As you can see, the Ollama API structure is similar to OpenAI’s!

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#search-queries)Search Queries 🔍

Search query to retrieve similar items using Ollama embeddings:  

```
-- View first 5 dimensions of an embedding
SELECT
    name,
    (replace(replace(embedding_ollama::text, '[', '{'), ']', '}')::float[])[1:5] as first_dimensions
FROM items;

-- Search for books about web development:
WITH web_book AS (
    SELECT embedding_ollama FROM items WHERE name LIKE '%Web%' LIMIT 1
)
SELECT
    item_data->>'title' as title,
    item_data->>'authors' as authors,
    embedding_ollama <=> (SELECT embedding_ollama FROM web_book) as similarity
FROM items
ORDER BY similarity
LIMIT 3;
```

 

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#performance-tips)Performance Tips 📊

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#add-an-index)Add an Index

```
CREATE INDEX ON items
USING ivfflat (embedding_ollama vector_cosine_ops)
WITH (lists = 100);
```

 

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#resource-requirements)Resource Requirements

- RAM: ~2GB for the model
- First query: Expect slight delay for model loading
- Subsequent queries: ~50ms response time

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#gpu-support)GPU Support

If processing large datasets, GPU support can greatly speed up embedding generation. For details, refer to the [Ollama Docker image](https://hub.docker.com/r/ollama/ollama).

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#troubleshooting)Troubleshooting 🔧

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#connection-refused-error)Connection Refused Error

The Ollama library needs to know where to find the Ollama service. Set the `OLLAMA_HOST` environment variable in `data_loader` service:  

```
data_loader:
  environment:
    - OLLAMA_HOST=ollama
```

 

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#model-not-found-error)Model Not Found Error

Pull the model manually:  

```
docker compose exec ollama ollama pull nomic-embed-text
```

 

Alternatively, you can add a script to automatically pull the model within your Python code using the [ollama.pull()](https://github.com/ollama/ollama-python?tab=readme-ov-file#pull) function.

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#high-memory-usage)High Memory Usage

- Restart Ollama service
- Consider using a smaller model

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#openai-vs-ollama)OpenAI vs. Ollama ⚖️

|Feature|OpenAI|Ollama|
|---|---|---|
|Vector Dimensions|1536|768|
|Privacy|Requires API calls|Fully local|
|Cost|Pay per API call|Free|
|Speed|Network dependent|~50ms/query|
|Setup|API key needed|Docker only|

## [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#wrap-up)Wrap Up 🌯

This tutorial covered only how to set up a local vector search with Ollama. Real-world applications often include additional features like:

- Query optimization and preprocessing
- Hybrid search (combining with full-text search)
- Integration with web interfaces
- Security and performance considerations

The full source code, including a simple API built with FastAPI, is available on [GitHub](https://github.com/yukaty/vector-search-api). PRs and feedback are welcome!

### [](https://dev.to/yukaty/part-3-implementing-vector-search-with-ollama-1dop#resources)Resources:

- [Ollama Documentation](https://github.com/ollama/ollama)
- [Ollama Python library](https://github.com/ollama/ollama-python)
- [Ollama Embedding models](https://ollama.com/blog/embedding-models)

---

Questions or feedback? Leave a comment below! 💬