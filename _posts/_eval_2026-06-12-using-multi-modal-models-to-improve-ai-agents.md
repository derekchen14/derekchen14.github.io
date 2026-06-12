---
layout: post
title: "Using Multi-modal Models to Improve AI Agents"
date: '2026-06-12 08:21:16'
tags: []
color: 
excerpt_separator: <!--more-->
---

## Motivation
Text-only agents often struggle when interacting with a world that is inherently visual, hitting a ceiling when they cannot perceive the context of a user interface or physical environment. Grounding language models in visual perception is not just an improvement but a critical requirement for truly effective task automation. The core vision behind VisionPo is to bridge this gap between sight and action, creating a seamless loop where the agent understands what it sees and acts accordingly.

## Ideas
We are exploring several promising directions, including the implementation of a multi-modal reward function to provide more accurate feedback during the learning process. There is also significant potential for zero-shot transfer capabilities to unseen visual interfaces, which could revolutionize web-based automation and accessibility. Furthermore, we are investigating the use of video for temporal grounding and treating screenshots as a specific tool the agent can call upon. To ensure performance remains high, we are developing logic to fall back to text-only processing when the latency budget is particularly tight.

## Process
The development process began with intensive data collection, navigating the significant hurdles of visual dataset curation to ensure the agent had a diverse range of interfaces to learn from. We aligned visual tokens with policy outputs and refined the agent through simulated testing. Specifically, the vision encoder processes raw screen pixels into high-level semantic tokens that represent UI elements like buttons and text fields. These tokens are then fed into the LLM-based planner, providing a structured visual context that allows the model to predict the next logical action based on both the user's goal and the current screen state. This direct mapping ensures the agent can react to dynamic changes in the interface in real-time. Next, we integrated a vision encoder with the planner, fine-tuned on real UI traces, and validated performance on held-out workflows before a controlled rollout.

## Takeaways
Our key findings highlight the superior efficiency of visual prompting compared to traditional text-only methods for complex UI tasks. Throughout the project, we gathered valuable lessons on the complexities of scaling multi-modal models for real-time applications. Looking ahead, our future research will focus on enhancing spatial reasoning and depth perception to make VisionPo even more capable in dynamic environments.