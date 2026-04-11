---
layout: post
title:  "Vision-LLM vs Document Parser"
date:   2026-04-10
---

<span class="dropcap">R</span>ecently, LandingAI published a [blog post](https://landing.ai/blog/docvqa-benchmark) about their Agentic Document Extraction (ADE) system hitting 99.16% accuracy on the DocVQA benchmark. Impressive number! The methodology is interesting too — instead of feeding images directly to a vision-language model at query time, they parse documents once into structured output, then answer all questions from that structured representation without ever looking at the original image again.

The benchmark result itself is solid and worth reading about. What I want to push back on is the business case they built around it.

Under the section "Real Business Value from Accurate Parsing" they argue that parsing-first is how document systems should scale — parse once, store the structured output, run unlimited queries against it, never re-process the images. They list a bunch of reasons: cost, latency, provenance, privacy, schema agility, RAG compatibility.

I work on information extraction from documents day to day, and reading that section felt like watching someone sell you a fishing boat because you mentioned you like seafood. The argument isn't wrong — it's just aimed at a different problem than the one most people actually have.

## The multiplier that doesn't always exist

The whole parse-once argument only works if you're planning to query the same document many times. That's the implicit assumption baked into every bullet point in that section. If a document gets queried 100 times after parsing, you've paid for one expensive parsing step instead of 100 vision-LLM calls. Great deal.

But a lot of real document pipelines don't look like that. Take the batch invoice extraction case — probably the most common document AI use case out there. A company uploads thousands of invoices. The system pulls out vendor name, invoice number, line items, totals, due date. Those values go into an accounting system. The invoices get archived. Nobody looks at them again.

In that workflow, the multiplier is 1. You're not paying for 100 vision-LLM calls — you were always going to make exactly one pass per document. Adding a parsing step means you now have two passes: one to parse, one to extract from the parsed output. You haven't saved anything. You've added a stage.

And that extra stage isn't free. Parsing introduces its own failure modes. LandingAI's own error breakdown is honest about this — of their 45 mistakes on DocVQA, some are genuine parsing failures where information was lost or misread before the QA step even ran. 

There's also a recent benchmark paper by [Berghaus et al. (2025)](https://arxiv.org/pdf/2509.04469), "Multi-Modal Vision vs. Text-Based Parsing: Benchmarking LLM Strategies for Invoice Processing", that tested this directly. They evaluated eight multi-modal LLMs across three invoice datasets, comparing direct VLM against a parse-to-markdown-first approach. The conclusion was that visual context and layout understanding are important for invoice extraction, and that current models do a better job leveraging those signals directly than working from an intermediate text representation.

That lines up with my intuition from working on these pipelines. Invoices are visually structured documents. The spatial relationship between a label and its value, the way a table is laid out, the font weight used for totals — these are real signals that a vision model can pick up on directly. Converting to markdown first means trusting the parser to preserve all of that, and parsers aren't perfect. 

## So when does parsing actually win?

There are real cases where the parse-once approach earns its complexity.

- The most obvious is when you genuinely do need to query the same document repeatedly. A contracts platform where legal teams search across thousands of agreements for specific clauses, or a compliance system that needs to re-examine historical documents every time regulations change — these are multi-query workloads where the economics flip. Parse once, query many actually means something.
- Audit and compliance cases matter too. When you need to show exactly which cell in which table a value came from — for a regulator, a legal dispute, an internal review — the bounding-box provenance that a parser gives you is hard to replicate from raw vision-LLM output. You could engineer something, but it's not natural.
- There's also the hybrid routing case. Once you have a parsed representation with per-field confidence scores, you can route the easy documents through a cheap fast path and escalate only the ambiguous ones to a more expensive model or a human reviewer. Without parsing, every document looks the same going in. You have no signal to route on.

The through-line in all of these: they either involve querying documents multiple times, changing what you extract without re-processing originals, or needing operational control over what gets stored and verified. Batch extraction for a single downstream system hits none of those.

## My opinioned TL;DR

If you're building a document platform with ongoing query workloads, their argument holds. If you're building a batch extraction pipeline that feeds structured data into a downstream system and then moves on, you're likely adding complexity without getting anything back.
The question worth asking before you choose your approach isn't "which method is more accurate on DocVQA?" It's simpler than that: how many times are you actually going to read each document?
If the answer is once, just use a vision-LLM directly.