---
layout: post
title: "Observability for Long-Running AI Agents"
date: '2026-04-24 14:08:32'
tags: []
color: 
excerpt_separator: <!--more-->
---

## Motivation
Long-running agents can drift silently over hours — small missteps compound into large failures with no obvious signal. Without visibility, behavioral degradation only surfaces at the end of a run, when the damage is already done.

A shocking billing surprise after a weekend agent run made clear that cost and behavior monitoring can't be an afterthought. That incident became the catalyst for building a proper observability layer for multi-hour agentic workloads.
## Latency Targets
Set latency budgets at every step of the agent loop, not just at the finish line. Separate per-action latency — tool calls and LLM inference — from cumulative wall-clock time across the full run. Budget both a p50 and a p99 per turn: the p50 tells you what normal looks like; the p99 tells you what kills a long run. Set soft and hard latency thresholds that trigger alerts before a run goes off the rails. The hard ceiling should activate a kill switch automatically, stopping a runaway agent before costs spiral. Track latency trends over time rather than relying on single-run snapshots; gradual slowdowns are invisible without a per-tool-call breakdown compared against a historical baseline.
## Token Accounting
Log input and output token counts at every LLM call, not just totals at the end. Break down usage by role — system prompt, conversation history, tool results, and generated output — so you can see exactly where context is being consumed. Watch per-turn input tokens and cache-hit rate — a falling hit rate usually means context is being rebuilt unnecessarily. Tool results are the most common hot spot, so apply per-step token budgets to catch runaway growth before it becomes a problem.
## Cost Modeling
Translate token counts into dollar figures in real time using current model pricing. Do it continuously, not just at run completion.

By modeling projected total cost from the early-run burn rate, you surface overruns while there is still time to intervene. That beats receiving a surprise invoice days later.

We learned this the hard way during a weekend billing incident. A long-running orchestration job kicked off Friday evening. Because cost reporting was only summarized at task completion, nothing flagged the anomaly until Monday morning. By then, a tight retry loop in one sub-agent had burned through roughly four times the expected budget without a single alert.

The agent was technically making forward progress. Each retry was slightly different, so no timeout or error alarm fired. Only the bill did.

The incident made two things clear: completion-time cost reporting is far too coarse for agents that run for hours, and retries need to be metered just as carefully as primary calls.

The fix centered on cost-per-completed-task as the headline metric. Rather than watching raw token totals climb, we computed a rolling cost-per-task figure every few minutes. This turns an abstract number into something tied directly to business value: if a task that should cost $0.04 is trending toward $0.40, the signal is unmistakable.

Tracking this metric changed how we thought about sampling. Several evaluation sub-agents were configured to sample at temperature 1.0 with five completions per prompt. That made sense for diversity during development, but it was burning real money in production. The cost-per-task lens made those runs stand out. We dialed sampling back to two completions for anything off the critical evaluation path, cutting that sub-agent's cost by roughly 60% with no measurable quality loss.

For orchestrated multi-agent pipelines, attribute costs to individual tasks and sub-agents rather than rolling everything into a single total. Attribution lets you identify which parts of the system are expensive and worth optimizing. It is also what let us pinpoint the retry loop that caused the weekend incident, rather than just noticing the aggregate overrun.

Pair attribution with hard cost ceilings that pause or terminate a run automatically when thresholds are breached. Manual oversight at 2 a.m. on a Saturday is not a cost-control strategy; automatic guardrails are.
## Dashboards
Design a live dashboard that surfaces latency, token usage, and cost side-by-side for a running agent. Placing these metrics together makes correlations visible — a latency spike that coincides with a token surge points to a very different root cause than one that doesn't. Include a timeline view showing how key metrics evolve across each step of the agent loop, and add anomaly indicators that highlight steps deviating significantly from baseline behavior. Make dashboards shareable so non-engineering stakeholders can audit a run without needing direct log access; transparency across the team is what turns observability from a debugging tool into an organizational asset. Design a live dashboard that surfaces latency, token usage, and cost side-by-side for a running agent. Placing these metrics together makes correlations visible — a latency spike that coincides with a token surge points to a very different root cause than one that doesn't. Include a timeline view showing how key metrics evolve across each step of the agent loop, and add anomaly indicators that highlight steps deviating significantly from baseline behavior. Make dashboards shareable so non-engineering stakeholders can audit a run without needing direct log access; transparency across the team is what turns observability from a debugging tool into an organizational asset.

- Add a latency-percentile view (p50 / p90 / p99) for each tool call and LLM step so that occasional slow outliers don't hide behind averaged latency numbers.
  * Segment percentiles by tool type to quickly spot whether slowness is systemic or isolated to a single integration.
- Add per-tool token panels that break down input and output tokens for every tool invocation, making it easy to identify which tools are the biggest context consumers at a glance.
  * Flag tools whose token footprint exceeds a configurable threshold to surface runaway calls before they compound across a long run.
## Takeaways
Silent drift is the core risk of long-running agents — observability is the antidote, not an optional add-on. Start with per-step logging; aggregates alone hide the failure modes that matter most. A single step consuming ten times the expected tokens is invisible in a run-level summary.

Cost modeling and latency targets should be defined before a run starts, not investigated after it ends. Setting those thresholds upfront is what enables automatic intervention rather than reactive damage control. A single weekend billing incident can justify the full engineering investment in an observability layer — the question is whether you want to learn that lesson the hard way.