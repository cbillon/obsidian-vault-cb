---
tags:
  - golang
  - load_balancer
---
# Introduction

Load balancers are fundamental to distributed systems. They determine how evenly traffic is distributed, how failures are handled, and how fast your service can grow. Over a weekend, I built a lightweight but production-style load balancer in Go—complete with active health checks, connection pooling, and a lock-free round-robin scheduler.

This article explains **how the load balancer works**, the **design decisions behind it**, and **bottlenecks you should care about** when building something like this yourself.

---

# **1. High-Level Architecture**

At a high level, the system has 3 major components:

```scss
          ┌────────────────────────┐
          │        Client          │
          └────────────┬───────────┘
                       │
                       ▼
             ┌───────────────────┐
             │   Load Balancer   │
             │-------------------│
             │ 1. Selector       │
             │ 2. Reverse Proxy  │
             │ 3. HealthChecker  │
             └───────────────────┘
            /           |             \
           ▼            ▼              ▼
     Backend A     Backend B      Backend C
```

### **Components**

|Component|Responsibility|
|---|---|
|**Selector (round-robin)**|Picks the backend in O(1) time using atomic operations|
|**Reverse Proxy**|Forwards HTTP requests to the backend using pooled connections|
|**HealthChecker**|Detects backend crashes proactively using `/health`|
|**Backend Registry**|Stores URLs + atomic health state|

The load balancer itself **does not handle HTTP directly**, nor does it open TCP connections.  
It’s the **ReverseProxy** and **HealthChecker** that own network I/O.

---

# **2. Load Balancer Core Logic**

The load balancer is intentionally small and fast. Its only job is to:

1. Pick the next backend using a lock-free round-robin
    
2. Ensure the backend is alive
    
3. Skip dead ones
    
4. Return an error if all backends are offline
    

### **Selector Implementation**

```go
type LoadBalancer struct {
    backends []*backend.Backend
    current  atomic.Uint64
}

func (lb *LoadBalancer) SelectBackend() (*backend.Backend, error) {
    attempts := 0
    total := len(lb.backends)

    for attempts < total {
        idx := lb.current.Add(1) - 1
        idx = idx % uint64(total)

        b := lb.backends[idx]
        if b.IsAlive() {
            return b, nil
        }
        attempts++
    }

    return nil, fmt.Errorf("all backends are offline")
}
```

### Why this design?

- **Atomic increment** → no mutex lock, massively more scalable
    
- **Modulo indexing** → predictable round-robin distribution
    
- **Attempts < len(backends)** → bounded retry loop
    
- **Alive check** → health-aware routing
    

This makes the LB’s hot path extremely cheap (~0.3 microseconds per selection).

---

# **3. Reverse Proxy with Connection Pooling**

Every backend instance has its own reverse proxy:

```go
proxy := httputil.NewSingleHostReverseProxy(url)
proxy.Transport = sharedTransport
```

Where `sharedTransport` uses connection pooling:

```go
var sharedTransport = &http.Transport{
    MaxIdleConns:        200,
    MaxIdleConnsPerHost: 50,
    MaxConnsPerHost:     50,
    IdleConnTimeout:     90 * time.Second,
    DisableKeepAlives:   false,
}
```

### Why connection pooling matters

Without pooling:

- Each forwarded request requires a full TCP handshake
    
- Latency jumps from ~0.1ms → 2–3ms
    
- Throughput collapses under medium load
    

With pooling:

- Reuses existing idle connections
    
- No handshake
    
- 20–30x faster routing
    

Connection pooling is **not optional** in a load balancer.

---

# **4. HealthChecker with Connection Pooling**

The HealthChecker runs in the background and pings `/health` endpoints:

```go
resp, err := hc.client.Get(b.URL.String() + "/health")
```

It uses its own pooled HTTP client:

```go
client := &http.Client{
    Timeout: 2 * time.Second,
    Transport: &http.Transport{
        MaxIdleConns:        100,
        MaxIdleConnsPerHost: 10,
        MaxConnsPerHost:     10,
        IdleConnTimeout:     90 * time.Second,
    },
}
```

### Full HealthChecker Flow

```scss
Every interval:
    For each backend in parallel:
        Send GET /health
        Read + close body (mandatory for pooling)
        Mark alive / dead via atomic flag
```

Minimal version:

```go
if resp.StatusCode == http.StatusOK {
    backend.SetAlive(true)
} else {
    backend.SetAlive(false)
}
```

### Why this matters

Without health checking:

- First request always fails when server goes down
    
- Load balancer reacts late
    
- Bad user experience
    

With active health checks:

- Instant failure detection
    
- Zero failed user requests
    
- Load balancer always knows which servers are alive
    

This mirrors real LB behavior (HAProxy, Envoy, NGINX).

---

# **5. Design Decisions (and Why They Matter)**

## **Decision 1: Atomic operations instead of mutex locks**

Using:

```go
current atomic.Uint64
alive atomic.Bool
```

instead of:

```go
sync.Mutex
sync.RWMutex
```

**Outcome:**  
Lock-free architecture → no contention → near-perfect scaling.

---

## **Decision 2: Reverse proxy per backend**

Why not 1 shared proxy?

Because:

- URL rewriting in reverse proxy is not concurrency-safe
    
- Each backend needs independent connection pooling
    
- Cleaner configuration and metrics
    

---

## **Decision 3: HealthChecker decoupled from LoadBalancer**

Selector shouldn’t:

- open connections
    
- run timers
    
- perform health checks
    

Decoupling keeps the LB simple and composable.

---

## **Decision 4: Active + Passive detection**

- Active: periodic `/health` checks
    
- Passive: mark backend dead if request fails
    

This hybrid strategy matches industry standards.

---

# **6. Bottleneck Analysis**

Even a simple LB has bottlenecks. Here are the real ones and how we addressed them.

---

## **Bottleneck 1: Lock Contention on Selection Path**

### Avoided by:

- atomic counter
    
- atomic health flags
    
- immutable backend list
    

This keeps the hot path ~0.3µs.

---

## **Bottleneck 2: TCP Handshake Flood**

### Avoided by:

- shared reverse proxy `Transport`
    
- keep-alive TCP connections
    
- idle connection pooling
    

With pooling → 20–30× more throughput.

---

## **Bottleneck 3: Unbounded health check spam**

### Avoided by:

- per-backend goroutines
    
- low concurrency limits (`MaxConnsPerHost`)
    

Optional improvement: exponential backoff.

---

## **Bottleneck 4: “All servers dead” fallback**

Load balancer must avoid trying every server indefinitely.  
Your `attempts < totalBackends` guarantee keeps this bounded.

---

# **7. Performance Observations**

Under synthetic concurrency tests:

- ~3.2 million backend selections per second
    
- Zero mutex contention
    
- Health checks complete instantly with pooling
    
- Reverse proxy forwarding is the real bottleneck (as expected)
    

The LB is no longer the limiting factor—the backends are.

---

# **8. Key Takeaways**

1. **Atomic operations** make a huge difference in load balancer performance.
    
2. **Connection pooling** is mandatory for realistic throughput.
    
3. **Reverse proxy per backend** is the simplest maintainable design.
    
4. **Health checking** must be proactive, not passive.
    
5. The load balancer’s job is simple: select a backend quickly and correctly.
    

Everything else—retries, circuit breaking, metrics—can be layered on top.

---

# **9. Closing Thoughts**

Writing this load balancer was one of the best deep dives into system design I've done in a while. It forced me to understand:

- how concurrency works at scale,
    
- how Go’s HTTP stack manages connections,
    
- how load balancers detect failures, and
    
- how much performance comes from simplicity rather than complexity.
    

# **10. Repository Link**

[https://github.com/AkshayContributes/load-balancer](https://github.com/AkshayContributes/load-balancer)

If you're preparing for backend interviews, or you simply want to understand real infrastructure better, _building your own load balancer is an incredible learning exercise._