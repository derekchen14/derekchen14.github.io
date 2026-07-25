---
layout: post
title: "Prompt Injection Has No Fix, Only Blast Radius"
date: '2026-07-25 01:10:40'
tags: [security, prompt injection, red-teaming, agents, blast radius, attacks]
color: 
excerpt_separator: <!--more-->
---

## intro
Prompt injection refuses to go away, with every newly announced defense quickly met by a working bypass. At this point, the honest framing isn't how we fix the vulnerability, but how we limit what it can reach when it inevitably succeeds.

## why injection sticks
The core problem is that language models process instructions and untrusted content through the exact same channel. When plain text in the context window is the only input medium, we cannot reliably mark one sentence as authoritative and another as raw data. A malicious instruction hidden in a webpage looks structurally identical to a legitimate system prompt once ingested. Filters that scan for content patterns are easily bypassed by rephrasing, because the underlying model has no separate lane for trust.

## the success-rate claim
Red-team studies on tool-using agents consistently find that injected instructions succeed in roughly half of all attempts. This occurs even when the system has defense layers deployed, provided the agent has access to active tools like email, browser control, or code execution. This benchmark is not unique to one vendor's internal test. Instead, it reflects multiple independent evaluations of agent frameworks published through 2025. A 50% success rate against active defenses is not a rare edge case; it is a coin flip every time the agent processes untrusted content.

## blast radius design
Because we cannot reliably prevent injection, the architectural challenge shifts to restricting what a hijacked agent can actually execute. Good design requires scoping permissions to the active task rather than granting broad, standing access. Critical actions—like transferring funds, deleting records, or contacting external users—must require explicit human confirmation. Finally, logging every tool call ensures that a compromised execution run remains fully auditable after the fact. The goal is to limit the blast radius so that a compromised instruction cannot cause systemic damage.