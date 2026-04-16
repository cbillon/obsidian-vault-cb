How I scaled a simple crawler into a "Staff-Level" automated research assistant by overcoming flaky inputs, hardware limits, and API rot.

---

As engineers, we are expected to stay on top of everything: Netflix’s latest architecture, AWS updates, Go 1.23 releases. The fear of missing out (FOMO) is real, but the time to read is nonexistent.

I wanted to solve this. My goal was simple: **Build an agent that reads engineering blogs for me and sends summaries to my phone.**

But building it on my daily driver (M1 MacBook Air, 8GB RAM) forced me to evolve the design from a  
script" to a "daemon." Here is the story of that evolution through four major bottlenecks.

## Bottleneck #1: The Ingestion Dilemma (HTML is a Trap)

### _The "Just Scrape It" Mistake_

My first instinct was to build a standard web scraper using **Colly** or **GoQuery**. I thought, "I'll just fetch the HTML and find the article links."

I immediately hit three walls:

1. **The DOM Stability Problem:** Tech blogs (especially Medium-based ones like Netflix's) use dynamic class names like `<div class="x7-y8-z">`. Every time they deployed a UI update, my crawler broke.
    
2. **The JavaScript Wall:** Many modern blogs (Uber, DoorDash) render content via React/Hydration. A simple `http.Get` returned an empty skeleton, forcing me to consider heavy tools like Selenium or Playwright.
    
3. **The Resource Tax:** Running a Headless Browser (like Chrome) to scrape 5 sites consumes ~1GB of RAM. On an 8GB machine, that’s 12% of my total memory just to find a URL.
    

### _The Alternatives Analysis_

|**Strategy**|**Pros**|**Cons**|**Verdict**|
|---|---|---|---|
|**HTML Scraping**|Can get everything|Brittle; breaks on UI changes|❌ Too High Maintenance|
|**Headless Browser**|Renders JS perfecty|Heavy CPU/RAM usage; slow|❌ Too Heavy for M1 Air|
|**RSS / Atom Feeds**|Standardized XML|Limited to feed content|✅ **The Winner**|

### _The Solution: Boring is Better (RSS)_

I pivoted to **RSS Feeds**.

- **Why:** It is a standardised XML contract. It doesn't care about CSS classes, React, or ads.
    
- **Efficiency:** Parsing 10 XML feeds takes milliseconds and kilobytes of RAM, compared to seconds and gigabytes for headless browsing.
    
- **Code:** I swapped 200 lines of fragile HTML parsing for the robust `gofeed` library.
    

## Bottleneck #2: The Hardware Reality Check

### _The "Hello World" Mistake_

With the links secured, I tried to summarise them locally using **Ollama** and **Llama 3 (8B)**.

**The Crash:**

My 8GB M1 Air immediately choked. The OS takes ~3GB, VS Code takes ~1GB. Loading an 8B parameter model (which needs ~4GB+ VRAM) left zero room for the Go compiler. My laptop turned into a heater, and the "summarisation" took 45 seconds per article.

### _The Solution: Cloud Delegation_

I realized that **Hardware Constraints dictate Architecture.** I refactored the system to use the **Strategy Pattern**, allowing me to swap the "Brain" of the agent.

I moved from local inference to **Google Gemini (Flash model)**.

- **Cost:** $0 (Free tier).
    
- **Latency:** 2 seconds (vs 45s).
    
- **RAM Usage:** <50MB.
    

## Bottleneck #3: The Rate Limit Wall (429s)

### _The "Too Fast" Mistake_

With RSS (fast) and Gemini (cloud), my agent became _too_ efficient. It grabbed 15 URLs and fired 15 concurrent requests to Gemini.

**The Crash:**

`429 Too Many Requests`. The free tier limits you to ~15 Requests Per Minute (RPM), and sometimes 5 RPM for newer models. My agent crashed instantly.

### _The Solution: Intelligent Pacing_

I couldn't just "try again." I needed to design for the constraint.

1. **Exponential Backoff:** If the API says "Stop", we wait 2s, then 4s, then 8s.
    
2. **The Speed Bump:** I added a calculated delay in the main loop to mathematically guarantee compliance.
    

Go

```go
// Staff-Level Resilience: Don't just hammer the API
cfg.RequestsPerMinute = 4 // Ultra-safe limit
safeDelay := time.Minute / time.Duration(cfg.RequestsPerMinute)

for _, job := range jobs {
    go worker(job)
    time.Sleep(safeDelay) // The "Speed Bump"
}
```

## Bottleneck #4: "Model Rot" & Hardcoding

### _The "It Worked Yesterday" Mistake_

I hardcoded the model string `"gemini-1.5-flash"` into my source code. One morning, I woke up to `404 Model Not Found`. Google had deprecated the alias, and my binary was useless until I recompiled it.

### _The Solution: Dynamic Configuration_

I learned that **Dependencies change faster than Code.** I refactored the initialization logic to pull the model version from Environment Variables (`GEMINI_MODEL`). Now, when a model is deprecated, I just update my `.env` file—no recompile needed.

## The Final Architecture

Today, the system is a robust background daemon that I trust.

- **Inputs:** RSS Feeds (Polled every 6 hours).
    
- **State:** A simple `history.json` file prevents re-reading old articles.
    
- **Brain:** Gemini Flash (Configurable).
    
- **Output:** Telegram Notifications.
    

## Key Takeaways

1. **Inputs Matter:** Don't scrape HTML if an XML feed exists. Reliability > "Getting everything."
    
2. **Respect Constraints:** If you have 8GB RAM, you can't run Llama 70B. Move the compute.
    
3. **Resilience > Speed:** A slow crawler that _never_ crashes is infinitely better than a fast one that dies on the 10th request.
    

You can check out the open-source code here: [https://github.com/AkshayContributes/crawler-agent](https://github.com/AkshayContributes/crawler-agent)