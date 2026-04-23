---
layout: post
title: "Using Multi-modal Models to Improve AI Agents"
date: '2026-04-23 17:50:50'
tags: []
color: 
excerpt_separator: <!--more-->
---

## Motivation
Text-only agents lack the visual grounding needed to interpret screenshots, diagrams, and real-world scene data.

Multi-modal perception dramatically broadens the input space an agent can reason over.

Early benchmarks show vision-augmented agents outperform text-only baselines on navigation and UI tasks.

Business and research pressure to build agents that operate in richer, less structured environments.


## Process
Select a pre-trained vision encoder (e.g. CLIP, SigLIP) and decide how it integrates with the language backbone.

Align visual embeddings with the token space the planner already understands via a projection layer or adapter.

Feed combined vision + text context into the policy model at each decision step.

Evaluate the loop: capture observations → encode → plan → act → repeat.

Instrument tracing so visual inputs and their downstream actions can be inspected and debugged.

Pick a vision encoder.

Wire it to the planner.

Fine-tune on UI traces.

Evaluate on held-out workflows.

Ship behind a flag.

## Ideas
Use video frames or gif sequences instead of static screenshots to give agents temporal context.

Let the agent generate its own visual "scratchpad" by rendering intermediate states back to images.

Experiment with interleaved image-text reasoning (similar to chain-of-thought but with visual anchors).

Probe cross-modal attention maps to understand which visual regions influence decisions.

Fine-tune the vision encoder on domain-specific imagery rather than relying solely on general pretraining.

Using video for temporal grounding.

Treating screenshots as a tool the agent can call.

Falling back to text-only when latency budget is tight.

## Takeaways
Multi-modal input is most valuable when the task genuinely requires visual context; avoid it otherwise.

Adapter-based integration beats full fine-tuning for teams with limited compute.

Evaluation is the bottleneck: invest early in visual replay tooling to make debugging tractable.

Expect latency costs — batching and caching visual embeddings are essential for production agents.