---
layout: post
title: "Using Multi-modal Models to Improve AI Agents"
date: '2026-06-12 07:53:02'
tags: []
color: 
excerpt_separator: <!--more-->
---

## Motivation
Text-only agents often struggle when interacting with complex visual interfaces that rely on more than just structured data. As autonomous web navigation becomes more sophisticated, there is a growing need for multimodal capabilities that can interpret visual cues. VisionPo bridges this gap by converting raw pixels into actionable intent, allowing agents to understand the context of a page beyond simple text. This visual grounding significantly improves reliability and increases task success rates in dynamic environments.

## Ideas
There are several promising avenues for future development, including the exploration of few-shot learning to help agents adapt to entirely novel app layouts. We are also investigating the potential for zero-shot generalization across different operating systems to make the agent more versatile. To address privacy concerns, we are looking into architectures that process visual data locally on the edge. Other hybrid ideas include combining VisionPo with traditional DOM-based approaches or using video for better temporal grounding. We are even considering treating screenshots as a specific tool the agent can call or falling back to text-only modes when the latency budget is particularly tight.

## Process
We started by collecting UI screenshots and aligning visual tokens with language model outputs. The vision encoder processes high-resolution image frames, extracting hierarchical visual features that represent the UI layout. These features are then projected into the same embedding space as the planner's text tokens, allowing the LLM-based planner to 'see' visual elements as part of its sequential input for decision making. By integrating OCR and element detection, we refined the policy network for better spatial awareness. Finally, we fine-tuned the planner on real-world traces and shipped the feature behind a flag for testing.

## Takeaways
Our implementation of VisionPo has shown significant performance gains, particularly in environments containing dynamic or non-textual elements. Throughout this journey, we have gathered valuable lessons from common failure modes in visual reasoning, which will inform our next steps. Looking ahead, we plan to scale these capabilities to handle more complex, multi-step workflows. This work represents a fundamental shift from text-centric to vision-centric agent design, paving the way for more intuitive AI assistants.