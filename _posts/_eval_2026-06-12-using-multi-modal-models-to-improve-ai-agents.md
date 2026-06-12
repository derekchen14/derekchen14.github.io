---
layout: post
title: "Using Multi-modal Models to Improve AI Agents"
date: '2026-06-12 08:13:35'
tags: []
color: 
excerpt_separator: <!--more-->
---

## Motivation
While text-only agents have made significant strides, they often struggle when operating in inherently visual environments where spatial context is paramount. Direct visual feedback is a fundamental necessity for completing complex tasks, such as navigating user interfaces or physical spaces. Direct visual feedback is not just a luxury but a crucial requirement for completing complex tasks that involve navigating user interfaces or physical spaces. This limitation highlights a growing gap between the reasoning capabilities of modern large language models and the requirements for truly embodied AI that can perceive and act in the world as humans do.

## Ideas
Looking ahead, there are several promising avenues for enhancing these systems, such as exploring cross-modal transfer learning to improve performance on novel vision tasks. One practical approach is to treat screenshots as a specific tool that the agent can call upon when needed, or even using video for better temporal grounding during long-running tasks. To maintain performance, we are also researching more efficient visual tokenization methods and implementing logic to fall back to text-only processing when the latency budget is tight. These efforts, combined with a focus on scaling laws for vision-augmented preference optimization, will define the next generation of multi-modal agents.

## Process
The development process begins with a robust data collection pipeline designed to capture high-quality visual demonstrations. After securing data, we wire a vision encoder—typically a pre-trained ViT—into the agent's planner. This allows the policy network to project visual features and textual tokens into a shared latent space, enabling the planner to ground its next-step decisions in the current UI state. We then fine-tune on UI traces for accuracy before moving to RL loops. Each iteration is tested for generalizability, then shipped behind a flag for real-world testing.

## Takeaways
The performance gains observed in VisionPo over baseline models demonstrate the clear advantage of integrating multi-modal capabilities into agentic workflows. However, these improvements come with significant hardware requirements that must be carefully managed during deployment. As we look toward the future, our roadmap focuses on refining multi-modal agent alignment to ensure these increasingly capable systems remain safe and effective in diverse environments.