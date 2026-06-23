---
link: https://nehmeailabs.com/post/structured-output-overhead
site: Nehme AI Labs
date: 2026-01-23T00:00
excerpt: Everyone wants JSON output from LLMs. But JSON has overhead. A simpler approach can cut latency and costs significantly.
slurped: 2026-06-23T08:04
title: "The JSON Tax: Why Structured Output Is Costing You More Than You Think | Nehme AI Labs"
tags:
  - ai
  - json
---

## The Structured Output Consensus

The industry has converged on a standard pattern for extracting structured data from LLMs: output JSON.

Need to extract entities? Output JSON. Parse a document? Output JSON. Classify with metadata? Output JSON. Every framework, every tutorial, every best practice guide says the same thing: define a JSON schema, instruct the model to output valid JSON, parse the result.

This makes sense from a developer experience perspective. JSON is universal. Every language has a parser. Schema validation is straightforward. The tooling is mature.

But there is a cost nobody talks about.

## The Token Math

Consider a simple extraction task: pull the name, company, title, and status from a block of text.

**JSON Output**

```
{
  "name": "John Smith",
  "company": "Acme Corp",
  "title": "Senior Engineer",
  "status": "active"
}
```

Token count (approximate): 35 tokens

The overhead:

- Opening and closing braces: 2 tokens
- Four key names with quotes and colons: ~16 tokens
- Structural commas and whitespace: ~4 tokens
- Value quotes: 8 tokens

The actual data (the four values) is maybe 11 tokens. The JSON structure adds 24 tokens of overhead. That is a 3x multiplier.

**Delimiter-Separated Output**

```
John Smith::Acme Corp::Senior Engineer::active
```

Token count: ~11 tokens

Same information. One third the tokens.

## Why This Matters

Output token generation is the primary latency bottleneck in LLM inference. Input tokens are processed in parallel. Output tokens are generated sequentially, one at a time. Each token adds latency.

Reducing output tokens from 35 to 11 is not a 3x latency improvement (there is fixed overhead), but it is significant. On a task that takes 500ms, you might save 150-200ms. That compounds across millions of requests.

The cost impact is more direct. Most API providers charge separately for output tokens, often at 2-3x the rate of input tokens. Cutting output tokens by 65% translates directly to lower bills.

## The Prompt Tax

JSON output also inflates your input tokens.

To get reliable JSON output, you need to specify the schema in your prompt:

```
Extract the following information and return as JSON:
{
  "name": "string - the person's full name",
  "company": "string - the company name",
  "title": "string - their job title", 
  "status": "string - either 'active' or 'inactive'"
}
```

That schema description is 40+ tokens. For a delimiter-separated format:

```
Extract: name, company, title, status
Output format: value1::value2::value3::value4
```

Maybe 20 tokens. You are paying for schema specification on every request.

## When Delimiter Formats Work

Delimiter-separated output works well when:

**The schema is fixed and known at parse time.** You know the fields, you know the order. The parsing code is trivial: `output.split("::")`.

**The values do not contain the delimiter.** If names might contain `::`, use a different delimiter or escape sequence. In practice, most structured data (names, emails, dates, categories) does not contain obscure delimiters.

**You control both sides.** You write the prompt and the parsing code. There is no need for a self-describing format when you already know the structure.

**Volume is high.** The savings per request are small. The savings across millions of requests are substantial.

## When JSON Still Makes Sense

JSON is the right choice when:

**The schema is dynamic or complex.** Nested objects, optional fields, variable-length arrays. Delimiter formats become awkward.

**Interoperability matters.** If the output goes to external systems or third-party code, JSON is the lingua franca.

**Debugging and logging.** JSON is human-readable and self-documenting. Delimiter-separated strings are opaque without context.

**The task is already slow.** If your prompt takes 5 seconds because of complex reasoning, saving 200ms on output formatting is noise.

## Implementation

The prompt change is minimal:

**Before (JSON)**

```
Extract the person's name, company, title, and status from the following text.
Return your answer as JSON with keys: name, company, title, status.

Text: {input}
```

**After (Delimiter)**

```
Extract from the text: name, company, title, status
Output exactly: name::company::title::status

Text: {input}
```

Parsing:

```
parts = output.strip().split("::")
result = {
    "name": parts[0],
    "company": parts[1],
    "title": parts[2],
    "status": parts[3]
}
```

For multiple records, use newlines:

```
John Smith::Acme Corp::Engineer::active
Jane Doe::Beta Inc::Manager::inactive
```

## Combining With Model Tiering

This optimization compounds with [model tiering](https://nehmeailabs.com/post/rightsize-research).

If you are already using small models for extraction tasks (which [RightSize](https://nehmeailabs.com/right-size) can help validate), delimiter-separated output makes them even more efficient. A small model generating 11 tokens is dramatically faster than a frontier model generating 35 tokens of JSON.

The combination of right-sized models and minimal output formatting can reduce both latency and cost by 80%+ compared to frontier-model-with-JSON baselines.

## The Broader Point

The AI ecosystem has inherited many patterns from traditional software development without questioning whether they apply to LLM workloads. JSON is one example. Verbose system prompts are another. Multi-turn conversations where single-turn would suffice.

Each of these patterns has a token cost. Tokens are the unit of both latency and money in LLM systems. Optimizing for developer convenience without considering token efficiency leaves significant performance and cost on the table.

The counterargument is that these optimizations are premature, that you should optimize for correctness and development speed first. This is true in early development. But at production scale, the economics change. A 3x reduction in output tokens across 10 million monthly requests is not premature optimization. It is responsible engineering.

## Summary

- JSON structured output adds 2-3x token overhead for simple extraction tasks
- Output tokens are the latency bottleneck and the expensive token type
- Delimiter-separated formats (like `::`) convey the same information with fewer tokens
- The prompt also gets shorter because you do not need to specify a schema
- Use delimiters when the schema is fixed, volume is high, and you control both ends
- Keep JSON for complex schemas, external interoperability, and debugging

**[Try RightSize](https://nehmeailabs.com/right-size)** — Test if small models can handle your extraction tasks

**[Read about FlashCheck](https://nehmeailabs.com/post/flashcheck-research)** — How specialized models outperform general-purpose giants

**[LLM Sanity Checks](https://github.com/NehmeAILabs/llm-sanity-checks)** — Open source decision guide for AI architecture