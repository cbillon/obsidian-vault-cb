---
link: https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg
byline: Sepehr Mohseni
site: DEV Community
date: 2026-01-28T13:54
excerpt: Two months ago, our internal knowledge base chatbot confidently told a support rep that our refund...
twitter: https://twitter.com/@
tags:
  - ai
  - rag
  - llm
slurped: 2026-02-08T10:11
title: Building AI-native backends – RAG pipelines, function calling, prompt versioning, LLM observability
---

Two months ago, our internal knowledge base chatbot confidently told a support rep that our refund policy was “14 days, no questions asked.” Our real policy is 30 days with approval for larger amounts.

A $2,000 refund was processed based on that hallucination.

That was the moment we stopped treating LLM features like “smart text boxes” and started treating them like **unreliable distributed systems** that require real engineering.

This article is not about demos. It’s about what you have to build **after** the demo works.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#the-reality-of-ai-backends)The Reality of AI Backends

Traditional backends are deterministic.

Same input → same output.

AI backends are probabilistic.

Same input → slightly different output depending on context, model variance, and prompt structure.

This means:

- You cannot trust outputs
- You cannot trust retrieval
- You cannot trust prompts
- You cannot trust tool calls
- You must observe everything

A production AI backend ends up looking like this:

API  
│  
AI Orchestrator  
├─ Guardrails  
├─ Router  
├─ Rate limits  
│  
├─ RAG pipeline  
├─ Function execution  
└─ Direct generation  
│  
Observability + Evals

If you skip any of these layers, you will eventually ship a hallucination that costs money.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#rag-is-not-chunk-embed-query)RAG Is Not “Chunk, Embed, Query”

The tutorial version of RAG is:

> split text → embed → vector search → pass to LLM

That works in a notebook. It fails in production.

Real RAG needs:

- Proper ingestion pipeline
- Semantic chunking
- Change detection
- Hybrid retrieval (vector + keyword)
- Reranking
- Ongoing evaluation of retrieval quality

### [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#ingestion-in-laravel)Ingestion in Laravel

Ingestion is a queued job, not a script.

You re see documents constantly. You re-embed only what changed.  

```
class IngestDocuments
{
    public function handle(SourceInterface $source)
    {
        $documents = $source->fetch();

        foreach ($documents as $doc) {
            $hash = sha1($doc->content);

            if (Cache::get("doc_hash_{$doc->id}") === $hash) {
                continue;
            }

            $chunks = (new SemanticChunker())->chunk($doc->content);
            $embeddings = app(EmbeddingService::class)->embed($chunks);

            app(VectorStore::class)->upsert($doc->id, $chunks, $embeddings);

            Cache::put("doc_hash_{$doc->id}", $hash, now()->addDay());
        }
    }
}
```

 

The biggest quality improvement you will see is **semantic chunking** instead of fixed token splits.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#hybrid-retrieval-is-mandatory)Hybrid Retrieval Is Mandatory

Vector search misses exact matches like order IDs, SKUs, emails.

Keyword search misses meaning.

You need both.  

```
class HybridRetriever
{
    public function search(string $query, int $limit = 8)
    {
        $vector = app(VectorStore::class)->search($query, $limit * 2);
        $keyword = app(KeywordSearch::class)->search($query, $limit * 2);

        return $this->mergeAndRank($vector, $keyword, $limit);
    }
}
```

 

Most hallucinations in RAG systems are actually **retrieval failures**, not model failures.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#generation-with-grounded-context)Generation With Grounded Context

What you pass to the model matters more than the model.  

```
class RagResponder
{
    public function answer(string $question, array $chunks)
    {
        $context = collect($chunks)
            ->pluck('content')
            ->join("\n\n");

        $prompt = Prompt::load('rag-answer', 'v3');

        $response = app(LLM::class)->chat([
            ['role' => 'system', 'content' => $prompt->system],
            ['role' => 'user', 'content' => $prompt->fill([
                'context' => $context,
                'question' => $question,
            ])],
        ], temperature: 0.2, json: true);

        return $response;
    }
}
```

 

Low temperature. Structured output. Explicit context.

You are trying to reduce creativity, not increase it.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#function-calling-without-guardrails-will-burn-you)Function Calling Without Guardrails Will Burn You

Letting an LLM trigger backend actions without controls is equivalent to letting users hit internal APIs directly.

Every tool call must go through:

- Authorization
- Rate limiting
- Audit logging
- Optional approval

```
class ToolExecutor
{
    public function execute(string $tool, array $args, User $user)
    {
        $definition = ToolRegistry::get($tool);

        Gate::authorize($definition->ability, $user);

        if ($definition->needsApproval && !$user->isAdmin()) {
            throw new AuthorizationException();
        }

        RateLimiter::hit("tool:{$tool}", 60);

        $result = call_user_func($definition->handler, $args);

        AuditLog::create([
            'user_id' => $user->id,
            'tool' => $tool,
            'args' => $args,
            'result' => $result,
        ]);

        return $result;
    }
}
```

 

Refunds, account changes, billing operations — these must never be “just a function call.”

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#prompts-are-code)Prompts Are Code

Prompts change behavior more than code does.

They must be:

- Versioned
- Stored
- Reviewed
- Rolled out gradually

```
class Prompt extends Model
{
    protected $casts = ['variables' => 'array'];
}

class PromptManager
{
    public static function load(string $name, string $version): Prompt
    {
        return Prompt::where(compact('name', 'version'))->firstOrFail();
    }
}
```

 

Never hardcode prompts in PHP files.

You will want to change them without redeploying.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#observability-is-not-optional)Observability Is Not Optional

You need to log, trace, and evaluate:

- The user query
- Retrieved chunks
- Final prompt sent
- Model output
- Tokens and latency

Without this, you cannot debug hallucinations.

You also need automated evaluations that periodically ask:

> “Is this answer actually grounded in the provided context?”

That’s how you catch issues before users do.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#caching-and-cost-control)Caching and Cost Control

LLM calls are expensive and slow.

Cache deterministic calls by hashing inputs.  

```
class CachedLLM
{
    public function chat(array $payload)
    {
        $key = hash('sha256', json_encode($payload));

        return Cache::remember($key, 3600, fn () =>
            app(LLM::class)->chat($payload)
        );
    }
}
```

 

Track cost daily and hard-stop if you exceed budget.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#what-actually-prevents-incidents)What Actually Prevents Incidents

After enough production incidents, you realize the real safeguards are:

- Hybrid retrieval
- Strict prompts
- Guarded tool execution
- Full tracing
- Automated evals
- Aggressive caching

Not model choice. Not fancy agents. Not frameworks.

Just engineering discipline applied to a probabilistic system.

---

## [](https://dev.to/sepehr/building-ai-native-backends-rag-pipelines-function-calling-prompt-versioning-llm-observability-kkg#final-takeaway)Final Takeaway

- A demo AI feature looks like magic.
    
- A production AI system looks like a paranoid, over-engineered backend.
    

And that’s exactly what it needs to be.