---
layout: post
title: Three Buckets and a Handful of Agents
tags: [ai, dialogue, startups]
color: orange
---

## 1. The Puzzle
Subtitle: What I learned building the same agent 7 different ways

- Twelve products marketed as "AI agents"
- They don't agree on what "agent" means
- They sort into three buckets by motive
  * SDK: meant to help you build one
  * Harness: meant to help you write code with one
  * Vendor: meant to keep you on their platform
- Separately, a handful of products are actually doing agent work end-to-end
- This post sorts them and asks what each bucket is really selling

## 2. SDKs: Meant to Build an Agent

### 2a. The roster

- Gemini ADK, AWS Strands, Microsoft Agent Framework, OpenAI Agents SDK, Claude Agents SDK
- All from major model labs and cloud vendors
- Five products, same shape

### 2b. The pitch and the monetization

- Pitch: bring your own logic, we plumb the rest
- Monetization: tokens and cloud (Vertex, Bedrock, Azure Foundry, GPT, Claude)
- The SDK is free because the tokens aren't

### 2c. What this tells you

- The SDK has no opinion on what an agent should be
- It has a strong opinion on whose tokens you should spend
- Shipping an SDK is a hedge: every developer who builds with yours becomes a customer

## 3. Harnesses: Meant to Write Code

### 3a. The roster

- Pi-mono, OpenHands, Hermes, OpenCode
- Open-source, narrow to the coding domain
- BYO model, BYO opinion

### 3b. Where they differ

- Pi-mono: pre-specified workflows on top of free-loop
- OpenHands: academic theory-of-mind framing
- Hermes: developer-targeted, monetization via hosting
- OpenCode: BYO-everything, lean substrate

### 3c. What this tells you

- Harnesses are substrates, not agents
- They give you the kitchen, not the meal
- The interesting question is which substrate's defaults match your use case

## 4. Vendors: Meant to Keep You on the Platform

### 4a. The roster

- Salesforce AgentForce, Snowflake Cortex SDK, Databricks Mosaic AI

### 4b. The reframe

- Wrong explanation: they are bad at building agents
- Right explanation: they aren't trying to build agents; they are defending data planes

### 4c. The defense map

- Salesforce → customer record
- Snowflake → warehouse query
- Databricks → lakehouse and ML lifecycle

### 4d. How moat defense explains every weird design choice

- Trust Layer / AI Gateway → keep data inside the paid security boundary
- Hardwired topology → model cannot route around the platform
- Mandatory grounding → every retrieval is a hook back into the data plane
- Swappable LLM → the model is not the product
- Citizen-dev UI builders → buyer is the seat-renewal VP, not the SDK dev

## 5. The Actual Agents

### 5a. The roster

- Gemini CLI, Codex CLI, Claude Code, Cortex Code, Assistant Factory

### 5b. They look nothing like each other

- Free loop: Gemini CLI, Codex CLI, Claude Code
- Constrained: Cortex Code (vendor-shaped to the warehouse)
- Flow set: Assistant Factory (multi-flow blog assistant)

### 5c. Why the shapes diverge

- Topology is downstream of buyer, not an engineering choice
- Each buyer wants a different shape
  * Devs → free loop
  * Snowflake analyst → constrained to the warehouse
  * Multi-task assistant user → flow set

### 5d. The tell

- Most actual agents ship from companies that also ship SDKs
  * Anthropic: Claude Code + Claude Agents SDK
  * OpenAI: Codex CLI + Agents SDK
  * Google: Gemini CLI + ADK
  * Snowflake: Cortex Code + Cortex SDK
- The SDK is the public bet; the actual agent is the experiment
- Only independent products like AF avoid the conflict

### 5e. Implications

- For buyers: know which bucket you are buying from
- For builders: pick the bucket before you pick the tool
- For model labs: shipping both means you haven't decided which bet wins yet

## Appendix: The Field Guide

### SDKs - meant to build an agent
1. Gemini ADK (Google): free SDK, monetizes via Google Agent Enterprise and Gemini tokens
2. AWS Strands (Amazon): free SDK, monetizes via Bedrock
3. Microsoft Agent Framework: pro-dev SDK for building agents, monetizes via Azure Foundry
4. OpenAI Agents SDK: handoffs, tools, and guardrails, monetizes via GPT tokens
5. Claude Agents SDK: TypeScript binary, monetizes via Claude tokens

### Harnesses - meant to write code
6. Pi-mono: free-loop coding harness with grounding and pre-specified workflows
7. OpenHands: open-source coding harness with academic theory-of-mind framing
8. Hermes (Nous Research): developer-targeted, monetization via hosting
9. OpenCode: open-source coding harness, BYO-model

### Vendors - meant to keep you on their platform
10. Salesforce AgentForce: defends the customer record
11. Snowflake Cortex SDK: defends the warehouse query
12. Databricks Mosaic AI: defends the lakehouse and ML lifecycle

### Actual Agents
- Gemini CLI 
- Codex CLI
- Claude Code (Anthropic)
- Cortex Code (Snowflake)
- Assistant Factory (AF)
