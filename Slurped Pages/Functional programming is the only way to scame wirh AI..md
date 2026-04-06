---
link: https://cyrusradfar.com/thoughts/functional-programming-is-the-only-way-to-scale-with-ai
byline: Cyrus Radfar
date: 2026-03-30T02:00
excerpt: AI agents fail in production because of mutable state, hidden
  dependencies, and side effects the agent can't see. The fix is functional
  programming. SUPER and SPIRALS are the frameworks I use.
slurped: 2026-04-05T17:39
title: AI agents keep failing. The fix is 40 years old.
---

Code examples in this post are available in five languages. Pick yours:

## The pattern I keep seeing

An agent reads a function that takes a list and returns a list. It writes tests. They pass. The function fails in production because it depends on a global config and a database singleton the signature never declared. The agent had no way to know. This isn’t a model problem. Functional programmers solved it in the 1980s.

I’ve shipped AI products for over a decade, and the trajectory is always the same: impressive demo, promising pilot, gradual degradation, debugging nightmare, project abandoned. [Most agent projects never make it to production](https://www.gartner.com/en/newsroom/press-releases/2025-06-25-gartner-predicts-over-40-percent-of-agentic-ai-projects-will-be-canceled-by-end-of-2027 "Gartner: 40%+ of agentic AI projects canceled by 2027"). The ones that do often get rolled back within a year. [MIT found 95% of AI pilots fail to deliver ROI](https://fortune.com/2025/08/18/mit-report-95-percent-generative-ai-pilots-at-companies-failing-cfo/ "MIT: 95% of AI pilots fail to deliver ROI"). The instinct is to blame the models. “GPT-5 will fix it” or “we need better prompts.” The failures are architectural.

When an agent writes code into a mutable, tightly-coupled codebase, it’s producing non-deterministic output that depends on hidden state it can’t see. The global config object three modules away, the function that logs to disk as a side effect, the test that was mocking a database that behaves differently in production: the agent has no way to know about any of it.

The codebase is hostile to automation, and we keep blaming the agent.

## Why agents need different code

A human developer builds a mental model of a codebase over months. They know where the bodies are buried: which functions mutate state, which modules share globals, which tests are flaky. They carry this context between sessions.

Agents don’t have that luxury. Every session starts from scratch. An agent reads the code that’s in front of it, follows the explicit contracts, and produces output based on what it can verify. This means anything implicit, any [hidden state](https://www.infoq.com/presentations/Simple-Made-Easy/ "Rich Hickey, Simple Made Easy (2011)"), any side effect buried inside a “pure” function, becomes a trap.

Here’s a function that looks fine to a human:


A developer on the team knows that config gets loaded from a YAML file at startup and the database accessor is a singleton that needs initialization. An agent sees a function that takes a list and returns a list. It writes tests against that contract, the tests pass in isolation, and the function fails in production because the global config wasn’t loaded.

Now multiply this across a codebase with hundreds of these hidden dependencies. Every function the agent touches has an [invisible blast radius](https://www.gamedeveloper.com/programming/in-depth-functional-programming-in-c- "John Carmack on Functional Programming in C++"). Every change it makes can break something in a module it never read. This is why agent projects degrade: each iteration introduces subtle state corruption that compounds.

The agent sees inputs and outputs. The hidden dependencies are invisible.

Functional programming solves these problems because it was designed to eliminate exactly the properties that make code hostile to automated reasoning. This isn’t a new insight. ML researchers have [known since the 1980s](https://academic.oup.com/nsr/article/2/3/349/1427872 "Hu, Hughes, Wang: How Functional Programming Mattered") that referentially transparent code is easier for machines to analyze, optimize, and transform. We just haven’t applied the lesson to the agents writing our code.

The principles are straightforward:

**Pure functions** return the same output for the same input, with no global state, database calls, or logging inside the function body. An agent can test a pure function by calling it with no setup or mocking required.

**Explicit data flow** means you can trace how inputs become outputs by reading the code linearly, without action-at-a-distance or mutations happening in a callback three layers deep. An agent can follow the data pipeline and understand what each step does.

**[Side effects at the boundaries](https://www.destroyallsoftware.com/talks/boundaries "Gary Bernhardt, Boundaries (SCNA 2012)")** means I/O, database access, and external API calls happen in a thin outer layer. The core logic is deterministic. An agent can rewrite core logic without worrying about accidentally triggering a payment or sending an email.

**[Composition over coupling](https://www.cse.chalmers.se/~rjmh/Papers/whyfp.html "John Hughes, Why Functional Programming Matters (1989)")** means small functions that snap together like Lego bricks. An agent can replace one function without understanding the entire module graph.

This isn’t about purity for its own sake. I don’t care about monads or category theory. I care that when an agent modifies a function, the scope of possible breakage is exactly one function.

## SUPER: five principles for agent-friendly code

I put these into an acronym because that’s how principles survive in organizations. Hover any term below for the full definition.

SUPER is five constraints on how you write code. Side Effects at the Edge means I/O happens in a thin outer layer, never inside business logic. Uncoupled Logic means dependencies are passed in, never pulled from globals. Pure & Total Functions means deterministic functions that handle every input. Explicit Data Flow means you can trace data linearly from input to output. Replaceable by Value means any expression can be swapped with its computed result.

The practical effect: an agent working on SUPER-compliant code can modify any function by reading only that function and its type signature. No hidden state to trace, no global config to discover, no side effects to accidentally trigger. Here’s what that looks like on a real function:

**Before:** the `evaluate_options` function from earlier, with its hidden dependencies.

An agent writing tests for this function will miss the config dependency, the database singleton, and the logger. The tests pass in isolation. The function fails in production.

**After:** the same logic, SUPER-compliant. Dependencies are parameters. I/O is the caller’s job. Every input is explicit.

The agent can now test `evaluate_options` by calling it with a list and a number. No mocking, no setup, no teardown. If the function is wrong, the agent sees it immediately. If it’s right, it stays right regardless of what the rest of the codebase does. The blast radius of any change is exactly one function.

### Deep dive: each principle in practice

The acronym is easy to remember. Knowing when you’re violating each principle is harder. Here’s what each one looks like in real code, with the specific failure mode it prevents.

Side Effects at the Edge

A function that sends a notification inside business logic means every test, every agent run, and every dry run triggers a real notification. Move the side effect to the caller. The function computes _what_ to send; the boundary layer _sends_ it.

The agent can test `process_order` a thousand times without sending a single email. When something breaks, you know it’s the computation, not the network.

Uncoupled Logic

A function that imports a module to get its dependencies is married to that module. Pass dependencies as arguments instead. This lets an agent swap implementations for testing without touching the import graph.

An agent testing the good version passes in a hash map as the cache and a list as the database. No Redis, no connection strings, no Docker containers.

Pure & Total Functions

A function that throws on unexpected input is a function that’s lying about its return type. A total function handles every case. Agents can’t catch exceptions they don’t know about; they can read a return type that says “this might fail.”

The agent reads the total version’s return type and knows it can fail. It writes tests for both paths. The partial version’s return type says `int`, so the agent writes tests that assume success, and the first bad input crashes production.

Explicit Data Flow

When data moves through nested callbacks or mutates an object across multiple methods, an agent can’t follow the pipeline. Linear data flow, where each step takes input and returns output, is readable by both humans and machines.

In the mutation version, the agent has to read the `Request` class to know what `sign` does to the internal state. In the pipeline version, `sign` takes three values and returns a new value. The agent can test it in isolation.

Replaceable by Value

If you can swap a function call with its return value and the program still behaves the same, that function is referentially transparent. This property lets agents cache results, skip redundant computation, and reason about code by substitution.

When every function is replaceable by its value, an agent can reason about your code algebraically. It can inline, extract, reorder, and cache without fear of changing behavior. That’s the foundation the other four principles build toward.

## SPIRALS: a process loop for human-agent collaboration

SUPER handles the code. But agents also need a structured process, or they drift. Anyone who’s watched [Auto-GPT](https://en.wikipedia.org/wiki/AutoGPT "Wikipedia: AutoGPT") burn through API credits in an infinite loop knows what unstructured agent autonomy looks like.

SPIRALS is a seven-step loop that I run agents through on every task. It’s not a waterfall; it’s a tight cycle, often sub-minute, that keeps agents focused and gives humans natural checkpoints to intervene.

Sense

Gather context: read the relevant files, check git status, identify what already exists. Agents that skip this step rebuild things that already work.

Plan

Draft an approach, consider trade-offs, and define what “done” looks like. The human validates before any code gets written.

Inquire

Identify gaps in knowledge. What assumptions is the agent making? What doesn’t it know? This prevents the confident hallucination problem where an agent barrels ahead on wrong assumptions.

Refine

Simplify the plan. Apply the 80/20 rule. If a ticket is bigger than 3 story points, split it. Complexity gets killed here, before it enters the codebase.

Act

Write the code, following SUPER principles, as small bounded changes with tests alongside.

Learn

Run the tests and check the output. If something failed, the agent records what specifically went wrong for the next iteration.

Scan

The step Auto-GPT never had. The agent zooms out, looks for duplication, new risks, and things the change might have broken elsewhere. This is why Auto-GPT looped forever: it never checked whether it was actually making progress.

The seven steps split into two phases:

S⋅P⋅I⋅R⏟plan  ∣  A⋅L⋅S⏟execute\underbrace{\textsf{S} \cdot \textsf{P} \cdot \textsf{I} \cdot \textsf{R}}_{\text{plan}} \;\Big|\; \underbrace{\textsf{A} \cdot \textsf{L} \cdot \textsf{S}}_{\text{execute}}

In practice, I run these as two separate commands. The planning phase (Sense, Plan, Inquire, Refine) produces design docs, tickets, and a burndown. A human reviews and approves. Only then does the execution phase (Act, Learn, Scan) start, and it runs per-ticket: write the code, verify it works, check for regressions, commit, move to the next ticket. The gate between SPIR and ALS is the only point where I require human approval. Everything else, the agent handles.

The SPIRALS loop: each iteration cycles through all seven steps until Scan confirms the goal is met.

The loop terminates when Scan confirms the goal is met. If it doesn’t converge, Scan flags it and a human decides what to do next, so you don’t wake up to an infinite loop that burned through your API budget overnight.

## Why they work together

SUPER without SPIRALS gives you clean code with no process. The agent writes a perfect function, then writes nine more that weren’t needed. Or it refactors something that didn’t need refactoring. Discipline in the code means nothing without discipline in the workflow.

SPIRALS without SUPER gives you a structured process applied to a messy codebase. The agent follows all seven steps, but the Act step produces code with hidden dependencies that corrupt on the next iteration. The loop degrades because the underlying code can’t support reliable automated modification.

Together:

- Side effects at the edge means only the Act step touches the real world. Sense, Plan, Inquire, and Refine are pure reasoning, safe to retry and cheap to test.
- Uncoupled logic means each SPIRALS step can be its own module or its own agent. You can swap in a better planner without rewiring the system.
- Purity means Plan and Refine are deterministic. Same input state, same plan. You can reproduce bugs by replaying inputs.
- Explicit data flow means you can trace exactly what happened at each step. When something goes wrong at minute 47 of a long run, you read the log linearly and find it.
- Referential transparency means intermediate results are cacheable. If Sense returns the same context, skip to Plan.

---

## What this looks like in practice

I use SUPER and SPIRALS on every project now. This website, [Unfudged](https://unfudged.io/), [Intraview](https://intraview.ai/), all of it.

The concrete difference: agents working on SUPER-compliant code produce changes that pass tests on the first try about 3x more often than agents working on typical imperative code with global state. I don’t have a rigorous study for this; it’s what I’ve observed across projects over the past year. The debugging time drops even more because when something does fail, the failure is local to one function, not spread across a graph of shared state.

The process difference with SPIRALS: agents used to require heavy babysitting, where I’d check every output and try to catch hallucinations before they landed. With SPIRALS, the Scan step catches most regressions before I see them. I review at the Plan and Learn steps and skip the rest unless Scan flags something. My involvement per task dropped from continuous to two checkpoints.

Neither framework requires rewriting your codebase from scratch. Start with SUPER’s “S”: move side effects out of your three most-modified modules. That alone makes agent modifications safer. Add the Scan step to your agent workflows. You’ll catch the infinite loops and the confident-but-wrong outputs before they cost you.

Both frameworks are in my [CLAUDE.md](https://github.com/cyrusradfar) files, so every agent I work with follows them from the first prompt.

## Where to start

You don’t need to rewrite your codebase. Pick one module and work through these five steps.

Find your three most-modified modules

Run `git log --format=format: --name-only | sort | uniq -c | sort -rn | head -20`. The files your team touches most are where hidden dependencies cause the most damage. Start there.

Move the side effects out

Find every function in those modules that reads global config, hits a database, writes a log, or calls an external API. Pull that I/O into the caller. The function’s job is to compute; the caller’s job is to interact with the world.

Make dependencies explicit

Every value a function needs should be in its parameter list. If a function reaches into a singleton or ambient context, add the parameter and pass it in. The function signature should be the complete contract.

Add the Scan step

After your agent completes a change, have it zoom out: check for duplication, look for things the change might have broken elsewhere, and verify the goal is actually met. This is the step that prevents infinite loops and confident-but-wrong outputs.

Measure the difference

Run your agent against the refactored code. Count how often the tests pass on the first try compared to before. If the architecture is right, you’ll see it in the numbers.

---

The industry is moving toward [more agent autonomy](https://karpathy.medium.com/software-2-0-a64152b37c35 "Andrej Karpathy, Software 2.0"), not less. If your code can’t be reasoned about by a machine, no amount of model improvement will save you.

The fix has been in your CS textbook for forty years. The agents just made it urgent.