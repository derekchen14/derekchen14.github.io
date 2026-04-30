---
layout: post
title: "Beyond Guardrails: Why Understanding Users Still Matters"
tags: [ambiguity, dialogue, trends, ai, product-strategy]
color: purple
---

The market for AI agent toolkits in 2026 is booming. LangGraph, CrewAI, Agno, Claude Agent SDK, Google ADK: the list just keeps growing. Whether you want to call it an agent framework, an orchestration engine, or agent harness, they all share a common goal: to make it easy for _developers_ to build agents. They compete on aspects such as the number of tool integrations, how to manage guardrails against prompt injection, and time-to-first-demo. The engineering is undoubtedly impressive. But what about optimizing for the _end user_? After a deep code review of the leading frameworks, I found a gap that none of them address. What happens when facing requests that are **vague, incomplete, or just plain confusing?**

<!--more-->

The popular tools optimize for the happy path: clear input, well-structured requests, cooperative users. But equally important is optimizing for the _unhappy_ path. The reality is that most people won't learn to prompt, won't read the documentation, and will probably just bail when the agent gets confused.  A solution to this has been hiding under our nose for years: **Natural Language Understanding** (NLU). This post examines whether taking an explicit NLU approach is a relic of the pre-LLM era or whether it's solving problems the rest of the industry is ignoring.

## Trending AI Agent Architectures

**[LangGraph](https://github.com/langchain-ai/langgraph)** (by LangChain) is a graph execution engine. You define nodes, edges, and typed state channels. A Pregel-inspired runtime executes nodes in supersteps. Your agent *is* a state machine. Maximum control over execution topology, but no built-in concept of what the user is trying to do.

**[CrewAI](https://github.com/crewAIInc/crewAI)** is a role-based orchestrator. You define agents with role, goal, and backstory, then assign them tasks. A "Crew" runs tasks sequentially or hierarchically. Flows (added later) provide event-driven workflow control. It has the strongest output validation system of the three: guardrails can catch bad agent output and retry automatically. But no input validation or ambiguity detection.

**[Agno](https://github.com/agno-agi/agno)** (formerly Phidata) is a batteries-included agent framework. You configure an Agent with a model, tools, knowledge base, memory, and guardrails. It handles the internal ReAct loop. 20+ vector database backends, 100+ built-in tools, and the fastest instantiation benchmarks. But the guardrails are binary (pass or halt) with no retry, no escalation, and no mechanism to ask the user for clarification.

**[Assistant Factory](https://github.com/derekchen14/personal_assistants)** (AF) takes a different approach. It models the conversation as a structured prediction problem: classify the user's intent, detect which workflow ("flow") to activate, fill typed parameter slots, validate entities, and only then execute. An Ambiguity Handler tracks uncertainty at four graded levels and generates level-appropriate clarification questions. Multi-model ensemble voting produces calibrated confidence scores. Entity repair tries to fix misspelled or malformed values before asking the user to repeat themselves. This implements explicit methods of natural language understanding and traditional conversational AI techniques.

But is AF carrying forward valuable ideas, or is it clinging to an outdated approach like a high school football star reminiscing about the good ole days?

## The Case Against NLU

AF looks like what happens when a dialogue systems researcher encounters LLMs and says "I'll use these as components in my existing architecture" instead of asking "does my architecture still make sense?" The entire machinery (slot-filling with inheritance hierarchies, multi-model ensemble voting, four-level ambiguity handlers, flow stacks with lifecycle states) exists to compensate for models that were bad at understanding language. Modern LLMs are not bad at understanding language. In fact, they're incredible.

Consider the cost. NLU requires running 3 parallel LLM calls just to detect which flow to activate, potentially 3 more for entity repair, plus calls for slot-filling, skill execution, and response naturalization. A single well-prompted agent with tool calling could handle the same request in one or two calls. The multi-model voting buys maybe 2–3 percentage points of accuracy over a single strong model,[^2] at 3× the cost and latency. In 2018, when individual models were 70–80% accurate on intent classification,[^1] ensembles were essential. In 2026, they look like gold-plating.

The slot-filling system is particularly suspect. AF defines 16 typed slot classes with inheritance hierarchies, validation logic, and priority levels. Modern toolkits handle this with a Pydantic model and structured output: the LLM extracts parameters directly into a typed schema. No SourceSlot, no GroupSlot, no distinction between fill-by-label and fill-by-values. It just works.

The ambiguity handler's four levels seem like over-engineering for a problem that good prompting solves. Tell the LLM "if you're not sure what the user wants, ask a clarifying question" and it will. You don't need a stateful component with declare/ask/resolve lifecycle methods.

The flow stack with its lifecycle states (Pending, Active, Completed, Invalid), deduplication, and mid-plan replanning faces the same objection. An LLM with good instructions and conversation history can manage multi-step tasks naturally. It remembers what it was doing. It can course-correct. It doesn't need a finite state machine to track where it is in a conversation.

## Is Any of That True?

While some of those criticisms are valid, the core critiques don't hold up when you look at what actually happens with real users.

### LLMs have poorly calibrated confidence

LLMs are excellent at understanding well-formed requests from cooperative users. They are subtly bad in ways that don't readily surface in demos. 

LLMs don't just signal confidence. They exude absolute certainty. When an LLM isn't sure what the user means, it often guesses and proceeds with full apparent confidence. Every once in awhile, it may asks a clarification question, but really only when prompted to do so. There is no downstream signal you can use to decide whether to proceed, clarify, or try a different approach. AF's multi-model voting produces a real number, and model disagreement is more informative than any single model's self-reported uncertainty.[^3] This is the only reliable way to get a confidence signal from LLMs, which are notoriously miscalibrated when asked to rate their own certainty.[^4][^5]

LLMs fail silently on vague input. Tell an LLM "make it better" and it will produce *something* without hesitation just to please the user. It won't tell you it doesn't know what 'it' refers to or what 'better' means. It will infer, often incorrectly, yet will go on to produce a reasonable sounding result. AF's slot-filling catches this structurally: if the "source" slot (which post are we talking about?) isn't filled, execution doesn't proceed. The LLM never gets the chance to silently guess wrong. This is not a problem that prompting reliably solves, because the failure mode of prompting is exactly that the LLM follows the instruction to ask for clarification *sometimes* and silently guesses *other times*, with no structural guarantee about which behavior you'll get on any given turn.

LLMs are inconsistent across turns. An LLM with conversation history can usually remember what it was doing. But "usually" isn't "always," and the failure mode is catastrophic: the agent suddenly acts as if the previous three turns didn't happen.[^6] AF's dialogue state, flow stack, and context coordinator provide structural guarantees: the active flow, its filled slots, and the conversation history are explicit state, not implicit in the LLM's attention pattern over a growing context window.

### Slot-filling is not the same as structured output

The comparison to Pydantic models misses the fundamental distinction. Structured output validates *syntax*: "topic must be a string, length must be an integer." AF's slots validate *semantics*: "topic must refer to something the user actually mentioned, not something the LLM inferred; the source post must exist in the database."

But AF's real aim goes even further than semantics. The architecture is designed to resolve the *pragmatics* of a conversation, understanding what the user wants beyond what they explicitly stated. When a user says "make the intro punchier," the semantic content is about modifying an introduction. The pragmatics include: which post's introduction (resolved via the active entity in the source slot), what "punchier" means for this user (resolved via user preferences in the memory manager), and whether this is a revision to the current draft or a new direction entirely (resolved via the flow stack's lifecycle state). These are inferences that require grounding to specific entities, memory of user patterns, and awareness of conversational context. This is exactly the machinery NLU provides, which are missing from most frameworks.

A Pydantic model can enforce that `topic` is a non-empty string. It cannot enforce that `topic` refers to something grounded in the conversation, and it cannot infer what the user *meant* from what they *didn't say*.

### "Just prompt the LLM to ask for clarification"

Asking the LLM to ask clarifying questions produces three consistent failure modes across every standard agent framework. First, the LLM asks clarifying questions even when the request is clear, irritating the user. Second, it proceeds with vague input when it should clarify, producing wrong results. Third, it generates open-ended "could you tell me more?" when a specific "which of these three posts did you mean?" would resolve the ambiguity in one click.

AF's four-level ambiguity handler addresses all three. It only fires when confidence is below a threshold (preventing over-asking). It fires structurally when required slots are unfilled (preventing under-asking). And it generates level-appropriate questions: General offers a menu of concrete options, Partial acknowledges what was understood and asks about what wasn't, Specific requests exactly one missing value, and Confirmation presents the best guess for one-click approval. The *shape* of the clarification matches the *type* of the uncertainty.

### Entity repair for semantic normalization

When a user types "my blgo post about ambiguty" (two typos), modern tools pass this to the LLM, which might or might not correct the spelling, and then to a tool call, which will fail because no post matches. The user sees an error. A properly designed NLU contains a post-hook to try case normalization, then fuzzy string matching (`difflib.get_close_matches`), then an LLM call asking "the user said X, valid options are Y: what did they mean?" Only if all three fail does it ask the user. Most typos and misnamings are resolved without the user ever knowing there was a problem.

This is a small feature that produces an outsized UX improvement. Every other toolkit treats the gap between what the user typed and what the system needs as the user's problem to fix. The Personal Assistant framework treats it as the system's problem to solve.

### Where the criticism genuinely sticks

NLU does carry some real baggage. The POMDP framing (observations, hidden states, belief tracking) adds academic weight to what is, in engineering terms, a straightforward classify → route → execute → respond pipeline. The math isn't wrong, but nobody is computing belief distributions, so the language obscures more than it clarifies. Some of the slot type hierarchy is over-specified for current needs. And the multi-model voting is the most expensive way to get a confidence signal. A single strong model with a calibrated follow-up call could deliver 80% of the benefit at a third of the cost. But these are trimming problems, not foundation problems. The academic language doesn't affect the engineering. The extra slot types don't block progress. These are things you'd simplify over time, not reasons to abandon the architecture.

---

## The Verdict

The popular agent frameworks in 2026 are solving a real problem: making it easy for developers to build agents that call tools, follow workflows, and produce results. They solve it well. If your users are developers, power users, or anyone who can articulate a clear request, these frameworks will serve you.

But there's a different problem that almost nobody is working on: users who can't articulate clear requests, who provide half the information, and then contradict themselves. Real users will say 'fix it' and expect the agent to know what 'it' means without providing any context. For any agent that serves the general public, this *is* the primary use case. Understanding the user suddenly becomes incredibly difficult, and tackling ambiguity becomes a primary concern, rather than an afterthought.

The 2018-era conversational AI systems (Rasa, Dialogflow, Watson) learned this the hard way and built machinery to handle it: intent classification, slot-filling, dialogue state tracking, conversation repair. Then LLMs arrived, the industry decided that machinery was obsolete, and a new generation of frameworks started from scratch. The issues it solved never went away. The industry just stopped noticing them in favor of newer, shinier problems. They threw out the baby with the bathwater.

AF carries those lessons forward while using modern LLMs as the execution engine. Far from being relics of a weaker era, NLU   provide solutions to problems that LLMs have quietly swept under the rug. When the user is cooperative, the results are identical whether or not you model the user's intent because easy is easy. But the unhappy path is where the architecture earns its keep. If your goal is to ship a demo or win a hackathon, use any of the popular frameworks. If your goal is to build an agent that ordinary people can actually rely on, then you need the kind of structured understanding that NLU provides.

---

[^1]: Larson et al. (2019) [An Evaluation Dataset for Intent Classification and Out-of-Scope Prediction](https://aclanthology.org/D19-1131/) — The CLINC150 benchmark showed models ranging from 75–86% accuracy across 150 intents with out-of-scope detection, representative of the 2018-era difficulty.

[^2]: Wang et al. (2023) [Self-Consistency Improves Chain of Thought Reasoning in Language Models](https://openreview.net/forum?id=1PL1NIMMrw) — Majority voting across multiple model samples yields modest accuracy gains (1–4 percentage points) over single-sample decoding.

[^3]: Lakshminarayanan et al. (2017) [Simple and Scalable Predictive Uncertainty Estimation using Deep Ensembles](https://proceedings.neurips.cc/paper/2017/hash/9ef2ed4b7fd2c810847ffa5fa85bce38-Abstract.html) — Showed that ensemble disagreement produces better-calibrated uncertainty estimates than any single model's confidence scores.

[^4]: Kadavath et al. (2022) [Language Models (Mostly) Know What They Know](https://arxiv.org/abs/2207.05221) — Found that LLMs are systematically overconfident on incorrect answers and that self-reported probabilities are poorly calibrated.

[^5]: Xiong et al. (2024) [Can LLMs Express Their Uncertainty? An Empirical Evaluation of Confidence Elicitation in LLMs](https://openreview.net/forum?id=gjeQKFxFpZ) — Showed that verbalized confidence from LLMs correlates poorly with actual accuracy, confirming miscalibration in the conversational setting.

[^6]: Liu et al. (2024) [Lost in the Middle: How Language Models Use Long Contexts](https://aclanthology.org/2024.tacl-1.9/) — Demonstrated that LLMs degrade at retrieving and reasoning over information positioned in the middle of long contexts, a failure mode that compounds across multi-turn conversations.
