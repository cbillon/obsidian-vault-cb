---
tags:
  - golang
  - url_shortener
---
> _“Every simple system starts as a toy and evolves into infrastructure. The art lies in knowing when to evolve.”_

---

## 🧭 Why This Article Exists

Most system design discussions about URL shorteners stop at _“put a DB behind an API.”_  
This piece goes all the way — from **single-binary MVP** to **Meta-level, multi-region infrastructure** — showing how and why each evolution happens.

It’s meant to be both:

- A **reference document** for **system design interviews**
    
- A **guide** for engineers who actually want to _build_ a scalable URL shortener in Go (TDD + production-grade)
    

---

## 📑 Table of Contents

1. Stage 1 – MVP (Proof of Concept)
    
2. Stage 2 – Single-Instance Production
    
3. Stage 3 – Database Persistence
    
4. Stage 4 – Caching Layer
    
5. Stage 5 – Distributed System
    
6. Stage 6 – Enterprise Scale
    
7. Architectural Decisions
    
8. SLOs and Failure Design
    
9. Interview Framework: How to Use This
    
10. Key Takeaways
    
11. Further Reading
    

---

## 🧩 Stage 1 — MVP (Proof of Concept)

|Aspect|Value|
|---|---|
|Deployment|Single binary|
|Storage|Go `map` + RWMutex|
|Scalability|~1K requests/sec|
|Cost|<$50/mo|
|Reliability|Data lost on restart ❌|

### Example (Go)

```go
type URLShortener struct {
  mu   sync.RWMutex
  data map[string]string
}
```

✅ Fast, tiny, deployable in minutes  
❌ No persistence, single point of failure

**When to stop here:**  
Hackathon projects, demos, coding challenges.

---

## ⚙️ Stage 2 — Single-Instance Production

### New Concepts

- Graceful shutdown
    
- Environment-based config (`BASE_URL`, `PORT`)
    
- URL canonicalization
    
- Collision retry loop (insert-if-absent)
    

✅ Production-grade hygiene  
❌ Still no durability or scale

---

## 🗄️ Stage 3 — Database Persistence

### Architecture

**Schema**

```sql
CREATE TABLE urls (
  key CHAR(8) PRIMARY KEY,
  long_url TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  click_count INT DEFAULT 0
);
CREATE INDEX idx_long_url ON urls(long_url);
```

### Why PostgreSQL?

- Easy to reason about consistency
    
- Native unique constraints for idempotency
    
- Works well with Go `pgxpool`
    
- Trivial to scale reads via replicas
    

|Metric|Value|
|---|---|
|Latency|20–50 ms per read|
|Throughput|~1K req/s|
|Data durability|✅|
|Horizontal scaling|🚧 Needs more|

✅ Durable and correct  
❌ All reads hit DB  
❌ Writes limited by one primary

---

## ⚡ Stage 4 — Add a Caching Layer

### Architecture

**Read path**

PostgresRedisAppClientPostgresRedisAppClientalt[Cache Hit][Cache Miss]GET /{key}GET keylong_url302 Location: long_urlSELECT long_url WHERE key=?long_urlSET key=long_url EX 86400302 Location: long_url

**Write path**

RedisPostgresAppClientRedisPostgresAppClientPOST /api/shorten {url}INSERT (key, long_url)\nON CONFLICT RETRYOKDEL key (invalidate)201 Created {key, short_url}

**Cache Policy**

- **LRU**: only top 1–5 % URLs
    
- **TTL**: 24 hours
    
- **Hit rate**: 80–90 % (Zipfian workloads)
    

|Metric|Value|
|---|---|
|Avg latency|~5 ms|
|Peak throughput|100K req/s|
|Cache memory|few GB|
|DB load reduction|~90 %|

✅ Orders-of-magnitude faster  
✅ DB load drops drastically  
❌ Need cache invalidation strategy  
❌ Cache consistency issues possible

---

## 🌐 Stage 5 — Distributed System

### Sharded, Multi-Region Design

**Key generation**

- 8-char Base62 key = 62⁸ ≈ 2.18 × 10¹⁴ combinations
    
- First 2 chars = shard prefix (3,844 shards)
    
- Predictable, uniform distribution
    

**Replication Model**

|Role|Responsibility|
|---|---|
|Primary|Writes|
|Replicas|Reads|
|Sync replication|Strong consistency|
|Async cross-region|DR + read latency|

**Performance**

- ~500K req/s reads
    
- p99 latency ≈ 20 ms
    
- Availability ≈ 99.99 %
    

✅ Scale by adding shards  
✅ Regional isolation  
❌ Operational overhead (failover, backups)

---

## 🏢 Stage 6 — Enterprise / Meta-Scale

### Global Topology

**Supporting Infrastructure**

- **Redis Cluster**: 300 shards (multi-region replication)
    
- **Postgres Shards**: 3,844 per region × 3 replicas
    
- **Eventing**: Kafka for click streams
    
- **Observability**: Prometheus, Grafana, ELK, Jaeger
    
- **Disaster Recovery**: RPO < 10 s, RTO < 1 min
    

**Numbers (illustrative)**

|Metric|Value|
|---|---|
|Read throughput|5 M req/s|
|Write throughput|100 K req/s|
|p99 latency|20 ms|
|Cache hit rate|85 %|
|Availability|99.999 %|
|Annual volume|150 T requests/year|

✅ Planet-scale durability  
✅ Real-time analytics  
✅ Regional failover  
❌ Huge operational complexity

---

## 🧮 Architectural Decisions

|Decision|Choice|Reasoning|
|---|---|---|
|**Key length**|8 chars|~218 T unique keys, negligible collision probability|
|**Sharding**|Prefix (first 2 chars)|Uniform distribution, easy routing|
|**Cache strategy**|Hot 1–5 %, LRU + TTL 24h|High hit-rate, bounded memory|
|**Metrics**|In-mem counters + periodic DB flush|500 ns updates, durability via batch|
|**Storage API**|Interface-based (`Store` interface)|Enables swap: memory ↔ Redis ↔ Postgres|
|**Security**|Validate URLs, block private IPs, rate-limit writes, sign aliases|Prevent SSRF, spam, and abuse|
|**Observability**|RED metrics, structured logs, tracing|Enable SLO monitoring & debugging|

---

## 📈 SLOs and Failure Design

|Tier|Target|Degradation Strategy|
|---|---|---|
|**Availability**|99.99 %|Serve from cache → read replicas → failover region|
|**Latency**|p50 ≤ 10 ms, p99 ≤ 50 ms|Prefer cached reads; fallback replicas|
|**Durability**|RPO ≤ 10 s, RTO ≤ 1 min|Promote replica, replay Kafka logs|
|**Scalability**|Linear ≈ 1M req/s per region|Horizontal app + cache scaling|

---

## 🧠 Interview Framework: How to Use This

When asked to _“Design a URL shortener”_ in interviews, use this 5-minute outline:

|Step|What to Say|Key Points|
|---|---|---|
|**1. Requirements**|Shorten, redirect, delete, analytics|Functional + non-functional (scale, latency)|
|**2. MVP**|One Go server + in-memory map|Start small|
|**3. Persistence**|Add Postgres for durability|Read/write paths|
|**4. Scale reads**|Add Redis read-through cache|Explain cache invalidation|
|**5. Scale writes**|Shard DB by key prefix|62² ≈ 3.8K shards|
|**6. Geo-scale**|Multi-region replicas, CDN, rate limiting|Fault tolerance|
|**7. Observability**|Metrics, tracing, logging|SLOs & debugging|
|**8. Trade-offs**|Latency vs Durability vs Cost|Articulate why|

💡 _Hint:_ Keep numbers handy — 8-char key ≈ 2 × 10¹⁴, Redis hit ~ 1 ms, DB read ~ 50 ms.

---

## 🔍 Key Takeaways

1. **Scale evolves.** Each layer solves a previous bottleneck.
    
2. **Most traffic is read-heavy.** Optimize 80 % cached reads first.
    
3. **Simplicity beats premature scale.** Don’t shard until you must.
    
4. **Measure relentlessly.** Latency histograms, p99, error budgets.
    
5. **Design for failure.** Everything breaks; plan for graceful degradation.
    

---

## 📚 Further Reading

- [Designing Data-Intensive Applications – Martin Kleppmann](https://dataintensive.net/)
    
- [Google SRE Book – Chapter 6 (Monitoring)](https://sre.google/sre-book/monitoring-distributed-systems/)
    
- [Base62 and Collision Probability](https://en.wikipedia.org/wiki/Birthday_problem)
    
- [Postgres Sharding](https://www.percona.com/blog/an-overview-of-sharding-in-postgresql-and-how-it-relates-to-mongodbs/)
    

---

## 🏁 Closing Thoughts

Building a URL shortener seems trivial — until you have to do it for billions of users.

Start with a single process.  
Measure.  
Then **add layers only when you must**.

That’s not just how you scale systems — it’s how you scale engineering judgment.

---

## 🏁 Code

> All source code, tests, and diagrams are available here → [https://github.com/AkshayContributes/url-shortener](https://github.com/AkshayContributes/url-shortener)

---

_Written by Akshay Thakur (2025)_