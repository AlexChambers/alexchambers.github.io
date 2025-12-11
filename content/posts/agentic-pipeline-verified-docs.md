---
title: "A Four-Agent Pipeline for Accurate LLM Documentation"
date: 2025-12-03
draft: false
summary: "LLMs hallucinate when you ask them to do too much. Breaking the work into survey, analyze, write, and verify stages fixes that."
tags: ["llm", "ai", "documentation", "agents"]
---

I built a four-agent pipeline that produces accurate documentation from code. Rather than asking one LLM to do everything, I needed to split the work across specialized agents.

Here's the problem I kept having: I'd point an LLM at my codebase and ask it to write docs. It seemed to write good documentation (sections, headers, key terms), but the output was full of hallucinations: wrong button names, prerequisites that didn't exist, steps that didn't match the actual code. I wasted hours fixing things that should've been right the first time.

The problem was not the tool (I was using Claude). It was the process. I was asking a single agent to do too much. Implicitly, I was asking it to do four different tasks: scan the codebase, understand the technical details, write user-friendly prose, and somehow verify its own work. That's too much context, too many responsibilities, and zero accountability.

So I broke it apart.

## It's basically MapReduce (but for LLMs)

When I showed this to a colleague, they immediately said "this is just MapReduce." And yeah, they're right. It's not a classic data-processing implementation, but the spirit is the same: decompose a huge problem into small pieces, process them independently, then aggregate.

I think of it as MapReduce for LLM context management.

| MapReduce Concept | What It Does in My Pipeline | Agent |
| :--- | :--- | :--- |
| **Map** | Break the codebase into individual features | `doc-surveyor` |
| **Reduce** | Distill code into structured facts | `doc-analyst` |
| **Verify** | Check output against source (LLMs need this, traditional MapReduce doesn't) | `doc-auditor` |

The verification step is the key difference. In normal MapReduce, the reduce output is trustworthy because the function is deterministic. LLMs hallucinate. You can't skip verification.

## The four agents

Here's how the pipeline actually works:

| Stage | Agent | What It Does | Output |
| :--- | :--- | :--- | :--- |
| 1 | `doc-surveyor` | Scans codebase, identifies features and entry points | `feature-manifest.json` |
| 2 | `doc-analyst` | Traces one feature, extracts exact facts from code | `technical-fact-sheet.json` |
| 3 | `doc-writer` | Turns facts into user-facing documentation | `draft-docs/<id>.md` |
| 4 | `doc-auditor` | Verifies every claim against source code | `verified-docs/<id>.md` |

## Why this actually works

Each agent has one job and can't screw up the others.

**The surveyor prevents context overload.** Instead of dumping an entire codebase into context (which guarantees failure), it produces a queue of discrete features. The downstream agents only see files relevant to one feature at a time.

**The analyst enforces truth.** This is the most important one. The analyst is forbidden from writing prose. Its only job is to extract exact role names, validation rules, error messages, etc. directly from code. The output is a structured fact sheet that becomes the "source of truth" for everything downstream. No summarization, no interpretation, just facts with file references.

**The writer never sees code.** It only reads the analyst's fact sheet. This sounds limiting, but that's the point. It forces the writer to translate implementation details into user terms. You get "Click **Export**" instead of "Send a POST to `/api/exports`".

**The auditor is adversarial.** The prompt tells it to distrust everything and actively hunt for errors:

> Trust nothing. Verify every claim against source code. Actively try to find errors, not confirm correctness.

It requires `file:line` references for every claim. If something's wrong, it kicks back to the writer with specific revision instructions.

**Parallelization comes for free.** Once the surveyor produces a queue of features, you can spin up multiple analyst/writer/auditor pipelines in parallel. Each feature is independent, so there's no contention. I've had 15 subagents running simultaneously (which is extreme, but it worked).

![15 subagents running in parallel](/15-subagents.png)

## What I learned

The insight (which seems obvious in retrospect) is that specialization beats generalization when you need accuracy. One agent trying to do everything will cut corners. Four agents with clear handoffs and an adversarial verifier actually produce docs I can trust.

I also learned that file-based handoffs between agents are underrated. Each agent writes its output to a file, and the next agent reads from that file. It's simple, debuggable, and you can inspect the intermediate artifacts when something goes wrong.

What started as me being frustrated with hallucinations turned into a pattern I now use for any multi-step LLM task where accuracy matters.
