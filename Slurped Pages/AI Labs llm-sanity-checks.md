---
link: https://github.com/NehmeAILabs/llm-sanity-checks
site: GitHub
excerpt: Contribute to NehmeAILabs/llm-sanity-checks development by creating an account on GitHub.
twitter: https://twitter.com/@github
slurped: 2026-06-23T08:00
title: GitHub - NehmeAILabs/llm-sanity-checks
tags:
  - ai
---

**A practical guide to not over-engineering your AI stack.**

Before you reach for a frontier model, ask yourself: does this actually need a trillion-parameter model?

Most tasks don't. This repo helps you figure out which ones.

---

## The Decision Tree

[](https://github.com/NehmeAILabs/llm-sanity-checks#the-decision-tree)

```
                         You have a task.
                                │
                                ▼
              ┌─────────────────────────────────┐
              │  Can regex, rules, or lookup    │
              │  tables solve it?               │──── YES ───► Use that. Stop.
              └─────────────────────────────────┘
                                │ NO
                                ▼
              ┌─────────────────────────────────┐
              │  Is it structured/tabular data? │──── YES ───► XGBoost, random forest,
              │  (predictions from features)    │              or logistic regression.
              └─────────────────────────────────┘              Often beats LLMs here.
                                │ NO
                                ▼
              ┌─────────────────────────────────┐
              │  Is it search/retrieval?        │──── YES ───► BM25 first. Add vector
              │                                 │              search if needed.
              └─────────────────────────────────┘
                                │ NO
                                ▼
              ┌─────────────────────────────────┐
              │  Does it need external knowledge│──── YES ───► <100 pages? Stuff in context.
              │  beyond the input text?         │              Larger? Then consider RAG.
              └─────────────────────────────────┘
                                │ NO
                                ▼
              ┌─────────────────────────────────┐
              │  Is the task simple?            │──── YES ───► Small model (1B-8B).
              │  (classification, extraction,   │              Test it first.
              │   summarization)                │
              └─────────────────────────────────┘
                                │ NO
                                ▼
                      You might need a frontier
                      model. But measure first.
```

---

## Quick Checks

[](https://github.com/NehmeAILabs/llm-sanity-checks#quick-checks)

### Check 1: Can you describe the task in one sentence?

[](https://github.com/NehmeAILabs/llm-sanity-checks#check-1-can-you-describe-the-task-in-one-sentence)

If yes → probably a small model task.

- "Extract the company name from this email" → Gemma 4B
- "Classify this support ticket as billing/technical/other" → Gemma 1B
- "Summarize this paragraph in 2 sentences" → Phi-4

If no → you might have an architecture problem, not a model problem.

### Check 2: What's your accuracy requirement?

[](https://github.com/NehmeAILabs/llm-sanity-checks#check-2-whats-your-accuracy-requirement)

|Accuracy needed|Model size|Why|
|---|---|---|
|70-80%|1B-4B|Good enough for suggestions, drafts, triage|
|85-95%|4B-12B|Production-ready for most tasks|
|95%+|Consider fine-tuning a small model, not scaling up||

Scaling to frontier models rarely buys you more than 5% accuracy on simple tasks. That 5% costs 50x more.

### Check 3: How many output tokens do you need?

[](https://github.com/NehmeAILabs/llm-sanity-checks#check-3-how-many-output-tokens-do-you-need)

Output tokens are the bottleneck. They determine latency and cost.

|Output type|Tokens|Consider|
|---|---|---|
|Yes/No, True/False|1-5|Tiny model, or even logit bias|
|Category label|1-10|1B model is enough|
|Short extraction|10-50|Delimiter-separated output > JSON|
|Paragraph|50-200|4B-8B models|
|Long generation|500+|Maybe you need a bigger model|

---

## The JSON Tax

[](https://github.com/NehmeAILabs/llm-sanity-checks#the-json-tax)

Everyone defaults to JSON for structured output. But JSON has overhead:

```
# JSON output (35 tokens)
{"name": "John Smith", "company": "Acme Corp", "title": "CTO", "status": "lead"}

# Delimiter output (11 tokens)
John Smith::Acme Corp::CTO::lead
```

For simple extraction tasks:

- 3x fewer output tokens
- 3x faster inference
- 3x cheaper

When to use JSON: nested structures, optional fields, API contracts. When to use delimiters: simple extraction, high-volume pipelines.

[Read more: The JSON Tax →](https://nehmeailabs.com/post/structured-output-overhead)

---

## Model Selection Cheat Sheet

[](https://github.com/NehmeAILabs/llm-sanity-checks#model-selection-cheat-sheet)

### Tiny (1B-4B params)

[](https://github.com/NehmeAILabs/llm-sanity-checks#tiny-1b-4b-params)

Best for: classification, yes/no, simple extraction

|Model|Params|Good at|
|---|---|---|
|Gemma 3 1B|1B|Classification, simple Q&A|
|Phi-4-mini|3.8B|Reasoning, function calling|
|Gemma 3 4B|4B|Best tiny all-rounder|

### Small (8B-17B params)

[](https://github.com/NehmeAILabs/llm-sanity-checks#small-8b-17b-params)

Best for: most production tasks, RAG, extraction, summarization

|Model|Params|Good at|
|---|---|---|
|Qwen 3 8B|8B|Multilingual, reasoning|
|Gemma 3 12B|12B|Quality/speed balance|
|Phi-4|14B|Reasoning, math|
|Llama 4 Scout|17B|Multimodal, long context (10M tokens)|

### Medium (27B-70B params)

[](https://github.com/NehmeAILabs/llm-sanity-checks#medium-27b-70b-params)

Best for: complex reasoning, long context, multi-step tasks

|Model|Params|Good at|
|---|---|---|
|Gemma 3 27B|27B|Near-frontier quality|
|Qwen 3 32B|32B|Complex tasks|
|Llama 4 Maverick|400B (17B active)|MoE, strong all-rounder|

### Frontier (100B+ dense params)

[](https://github.com/NehmeAILabs/llm-sanity-checks#frontier-100b-dense-params)

Best for: novel tasks, complex reasoning, when nothing else works

Before you use these, ask: have you tried a smaller model?

---

## Anti-Patterns

[](https://github.com/NehmeAILabs/llm-sanity-checks#anti-patterns)

### ❌ "We use GPT-5 for everything"

[](https://github.com/NehmeAILabs/llm-sanity-checks#-we-use-gpt-5-for-everything)

That's not a flex. That's a $50K/month cloud bill waiting to happen.

### ❌ "We need the best model for our enterprise customers"

[](https://github.com/NehmeAILabs/llm-sanity-checks#-we-need-the-best-model-for-our-enterprise-customers)

Your enterprise customers care about latency, reliability, and cost. Not model prestige.

### ❌ "Small models aren't accurate enough"

[](https://github.com/NehmeAILabs/llm-sanity-checks#-small-models-arent-accurate-enough)

Did you test? With the right prompt? On your actual data?

### ❌ "We'll optimize later"

[](https://github.com/NehmeAILabs/llm-sanity-checks#-well-optimize-later)

You'll optimize never. The technical debt compounds. Start right-sized.

### ❌ "JSON output is industry standard"

[](https://github.com/NehmeAILabs/llm-sanity-checks#-json-output-is-industry-standard)

For simple extraction, it's industry waste. See: The JSON Tax.

### ❌ "We need RAG for our documents"

[](https://github.com/NehmeAILabs/llm-sanity-checks#-we-need-rag-for-our-documents)

For small document sets? No you don't.

Context windows are now 2M-10M tokens. That's thousands of pages. If your knowledge base is <100 pages, just stuff it in context. Preprocess, convert to markdown, include directly.

RAG adds complexity: chunking strategies, embedding models, vector databases, retrieval tuning, reranking. All that infrastructure for documents that fit in a single prompt.

**When RAG makes sense:**

- Millions of documents
- Frequently changing content (re-embedding is cheaper than re-caching)
- Cost-sensitive at scale (RAG is 8-80x cheaper per query for large, static corpora)

**When to skip RAG:**

- <100 pages of docs
- Static content that rarely changes
- You value simplicity over marginal retrieval precision

---

## Patterns

[](https://github.com/NehmeAILabs/llm-sanity-checks#patterns)

### ✅ Cascade Architecture

[](https://github.com/NehmeAILabs/llm-sanity-checks#-cascade-architecture)

Start with smallest model. Verify output. Escalate only on failure.

```
Input Text
    │
    ▼
┌─────────┐              ┌────────────┐    verified?     ┌──────────┐
│ Gemma 4B│ ───────────► │  Verifier  │ ───────────────► │ Return   │
└─────────┘   output     └────────────┘   yes            └──────────┘
                              │ no
                              ▼
┌─────────────┐          ┌────────────┐    verified?     ┌──────────┐
│ Llama Scout │ ───────► │  Verifier  │ ───────────────► │ Return   │
└─────────────┘          └────────────┘                  └──────────┘
```

Verifier can be: format validation, a classifier, or [FlashCheck](https://nehmeailabs.com/flashcheck) for grounding checks.

See [examples/cascade.py](https://github.com/NehmeAILabs/llm-sanity-checks/blob/main/examples/cascade.py) for a working extraction example.

### ✅ Task-Specific Models

[](https://github.com/NehmeAILabs/llm-sanity-checks#-task-specific-models)

One model per task type, sized appropriately.

- Classification → 1B
- Extraction → 4B
- Summarization → 8B
- Complex reasoning → Frontier

### ✅ Measure First, Scale Never

[](https://github.com/NehmeAILabs/llm-sanity-checks#-measure-first-scale-never)

Before adding a bigger model:

1. Benchmark current model on 100 real examples
2. Identify failure modes
3. Try prompt engineering
4. Try fine-tuning small model
5. Only then consider scaling

### ✅ Simple Tools Over Browser Automation

[](https://github.com/NehmeAILabs/llm-sanity-checks#-simple-tools-over-browser-automation)

For research tasks, don't reach for computer use or Puppeteer.

```
Search API → Fetch URL → HTML to Markdown → LLM synthesis
```

Three tools. No browser. No screenshots. No vision model.

Browser automation is only for: login walls, dynamic forms, actions (booking, purchasing).

See [patterns/agents.md](https://github.com/NehmeAILabs/llm-sanity-checks/blob/main/patterns/agents.md) for the full agent decision tree.

---

## More Patterns

[](https://github.com/NehmeAILabs/llm-sanity-checks#more-patterns)

- [Extraction patterns](https://github.com/NehmeAILabs/llm-sanity-checks/blob/main/patterns/extraction.md) — regex → NER → small LLM ladder
- [Agent patterns](https://github.com/NehmeAILabs/llm-sanity-checks/blob/main/patterns/agents.md) — when to use simple tools vs browser automation

---

## Tools

[](https://github.com/NehmeAILabs/llm-sanity-checks#tools)

### RightSize

[](https://github.com/NehmeAILabs/llm-sanity-checks#rightsize)

Test your prompts against multiple model sizes. See what's actually needed.

[→ Try RightSize](https://nehmeailabs.com/right-size)

### FlashCheck

[](https://github.com/NehmeAILabs/llm-sanity-checks#flashcheck)

Verify LLM outputs with tiny specialized models. Sub-10ms verification.

[→ Learn about FlashCheck](https://nehmeailabs.com/flashcheck)

---

## Contributing

[](https://github.com/NehmeAILabs/llm-sanity-checks#contributing)

Found a pattern that works? Open a PR.

Keep it practical. Keep it measured. No vibes-based claims.

---

## License

[](https://github.com/NehmeAILabs/llm-sanity-checks#license)

MIT. Use it. Share it. Don't over-engineer it.

---

_Built by [Nehme AI Labs](https://nehmeailabs.com/) — AI architecture consultancy._