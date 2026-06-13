---
layout: post
title: "How Much Can You Trust Your Agent?"
subtitle: An In-depth Comparison of the Architectural Decisions Behind 7 Popular Agent SDKs
tags: [agents, ai, ambiguity, explainer, trends, trustworthy]
color: rgb(30, 70, 105)
---

Every company, big or small, currently has a mandate to lean into the AI future and provide *some* offering around agentic AI. Stripped down to the essentials though, an agent is just a while loop with some tool calls. It can't be that complicated, right? Given the simplicity, one might expect these agent offerings to be relatively homogenous, sharing roughly the same abstractions under different brand names. After reading through the source code of a dozen-plus agent SDKs, this turns out *not* to be the case. Every company carries along unique biases and incentives, leading to a proliferation of different philosophies when designing an AI Agent. Choosing the right SDK for your own purposes actually comes down to asking, "How AGI-pilled have you become?"

<!--more-->

Each agent SDK has baked in its own set of assumptions about when and where to trust the underlying model(s) powering its operations. The SDKs which trust the model more tend to stay smaller, giving more free rein. The SDKs that don't trust the LLM as much tend to be bigger, with more code needed to enforce guardrails. But this isn't the only difference. Across composition, observability, memory, and a whole host of other dimensions, agents are being built in all sorts of shapes and sizes!

Concretely, the seven specific agent SDKs or harnesses that we studied, in no particular order:
  - **LangGraph** (LangChain): [github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)
  - **Claude Agent SDK** (Anthropic): [github.com/anthropics/claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python)
  - **Google ADK** (Google): [github.com/google/adk-python](https://github.com/google/adk-python)
  - **OpenAI Agents SDK** (OpenAI): [github.com/openai/openai-agents-python](https://github.com/openai/openai-agents-python)
  - **Pi-mono** (earendil-works): [github.com/earendil-works/pi](https://github.com/earendil-works/pi)
  - **OpenCode** (Anomaly): [github.com/anomalyco/opencode](https://github.com/anomalyco/opencode)
  - **Cortex Code** (Snowflake): [docs.snowflake.com/cortex-code-agent-sdk](https://docs.snowflake.com/en/user-guide/cortex-code-agent-sdk/cortex-code-agent-sdk)

While many other agent projects were considered, I landed on these 7 due to their popularity and their diversity of opinion. In other words, each one is fairly representative of a unique stance in the ecosystem and has reasonable traction within a group of followers. As a bonus, I will also be including a comparison to [Assistant Factory](https://www.assistantfactory.io), which captures my own best practices around building an agent, having been actively focused on this space for nearly a decade.

After weeks poring through thousands of lines of code, it became clear that authors' beliefs around AI can influence everything from control flow and memory management to error recovery and evaluation. What you actually get when you adopt one of these SDKs is a set of inherited opinions about how much to trust the model in the loop, which end up dictating the capabilities (but also the limitations) of what your agent can accomplish. The next few sections walk through the most interesting insights that emerged as we traced the critical distinctions among the competing options. An appendix is also included to provide a cross-cutting view of the key dimensions.

## Everyone Has an Opinion
*SDKs are designed to push the vendor's agenda more than they are to help you build agents*

Every SDK is shipped by a parent company with a financial reason to ship it, and that reason is rarely a pure love of agents. The foundation-model providers are the easiest group to read, since their SDKs exist to route inference spend back to the parent's API. While OpenAI ships its Agents SDK with the option to pick other providers, it sets `gpt-5.4-mini` by default and only asks for the OpenAI API key to get started.[^1] The Claude Agent SDK goes further by bundling the Claude Code CLI inside the package such that you're not even allowed to choose non-Anthropic models.[^2] Both agent SDKs also provide numerous methods to allow the agent to loop and reason, which leads to spending more tokens and driving up API costs.

You would think Google ADK belongs with the model providers with Gemini as the driver. While it's true `gemini-3-flash-preview` is the default, with access to some privileged paths[^3], the real funnel points at the cloud. Much of ADK's code surface points back to Google Enterprise Agent Platform (prev. Vertex AI), with comparatively little devoted to agent-loop logic. Thus, the main driver is actually to support the hyperscaler business model. Amazon Strands runs the same playbook, instantiating an AWS Bedrock model whenever you do not pass one in and a dozen endpoints meant to hook into AWS services.[^4] Microsoft's Agent Framework also follows this pattern, framing its quickstarts and hosted deployment path around Azure AI Foundry while other providers sit in satellite packages.[^5] For the hyperscalers the model is swappable by design, because the product is the cloud spend around the loop rather than the tokens inside it.

The next group is genuinely model-agnostic since their aim is to provide open-source offerings for free, while getting paid in hosting and services. Pi lets you wire any provider behind its small set of built-in tools. OpenCode requires you to declare the provider and model on every call, mildly annoying and exactly the point, since the SDK holds no opinion about whose model you run. Assistant Factory provides a PromptEngineer class which lets you pick the model by changing a single parameter during instantiation. LangGraph likewise treats the model as something you bring, since LangChain makes its money on the paid LangSmith observability product. When nobody upstream profits from your token spend, the defaults stop nudging you.

The final group includes the platform vendors who hardly care which model runs the loop, so long as their platform stays the system of record. Snowflake Cortex is the cleanest case: the agent runs inside Snowflake on Anthropic's models by default, but its tools run under Snowflake governance, and its traces land in an in-account SQL table, so your data never gets a reason to leave the warehouse. Databricks Mosaic AI makes the same assumption about the lakehouse, building agents over Unity Catalog governance, while ServiceNow's Now Assist runs its agents natively inside the workflows the Now Platform already owns, and Salesforce Agentforce grounds its Atlas reasoning engine in the CRM records its Data Cloud unifies.[^6] Whereas others may care about tokens and compute, this group is focused on data gravity and making sure they remain at the center of it.

One final piece of evidence that the SDKs are not meant to build great agents: the parent companies rarely use the SDKs when launching their own AI agents! OpenAI's Codex is an independent Rust implementation with no dependency on the Agents SDK.[^7] Google does say ADK powers enterprise products like Agentspace, but its consumer agents and CLI show no public sign of it.[^8] LangChain offers an 'Ask AI' agent on their website, but it is powered using a general loop rather than taking advantage of the full set of abstractions offered by LangGraph. Overall, **these SDKs are distribution channels first and engineering artifacts second.** With that said, the engineering does reveal additional insights, which is what we will dive into next.

## The Shape of Intelligence
*The way context flows through an agent falls into several distinct buckets*

Given the user utterance, how does information flow within an AI agent to produce a response? Across just eight SDKs, we encounter at least five distinct topologies: deterministic graphs, decentralized swarms, stacks and queues, hub-and-spoke orchestration, and the no-structure loop. None are clear winners since each topology offers its own pros and cons when deciding who tackles the next unit of work and how that information is conveyed. Each shape is also a statement of trust: pre-determined routes display a lack of trust in the model to choose the correct options for itself, while routing during inference implies it can.

### Sub-agent Composition and Communication

Google ADK and LangGraph are graph engines. The developer is expected to explicitly wire together a series of nodes and edges, where each node is effectively its own sub-agent. The graph is validated up front, and the topology is fixed at authoring time.[^9] Microsoft Agent Framework also falls under the same bucket, and even uses the same terminology of a `workflow` to describe the collection of nodes used to compose the graph. All three execute their policies with Pregel-style super-steps.[^10] Sub-agents communicate through typed, key-addressed state, with downstream nodes reading upstream outputs by name: ADK threads a schema-checked dict, LangGraph gives every channel its own key. These strict data structures hint at the core audience of these tools: enterprise customers who demand strict governance, reproducible execution order, and somewhere to place the blame when something goes wrong.

OpenAI Agents and Assistant Factory let routing happen at runtime, but maintain clear guidelines around how one sub-agent or flow passes authority over to the next. OpenAI runs a decentralized swarm with no central coordinator: each agent is handed other agents as delegation targets, a handoff is a `transfer_to_X()` tool call[^11] that moves control wholesale, and the topology becomes whatever chain appears. The handoff payload is a strict JSON schema rather than free text, and the run carries a typed item log, so spontaneity does not cost the swarm its structured state. Assistant Factory composes work as sub-agents on a stack. There is exactly one active Flow on top at any given time, with options to `fallback` to other Flows or `stackon` to inject additional Flows to run first (especially useful for multi-step planning). A structured belief state including typed slot-value pairs and natural language insights is passed around through a shared session scratchpad.

The Claude Agent SDK, OpenCode, and Snowflake Cortex converge on hub-and-spoke, in which a single orchestrator model farms isolated tasks out to short-lived subagents and decides alone when to do so. Each spoke runs in a fresh context window, sees only the prompt the hub composed, and returns one result. Each sub-agent is effectively just another tool call.[^12] Communication here is an append-only prose transcript, since the payload a spoke returns is natural-language text the hub must re-read and re-interpret rather than a field it can look up. Nothing but the prompt enforces what a spoke reports back, which makes this a doubly trusting design: the model routes the work and serializes the results in prose, hopefully without losing anything that matters.

Arguably, Pi (not Claude) is the most radical, declining to have any composition structure at all. The core is a single agent running a single loop that streams a message, executes whatever tool calls it requested, appends the results, and repeats.[^13] There is no data structure to mediate the information flow, no additional scaffolding, and in fact, no sub-agents at all! The design philosophy is to keep everything as minimal as possible. Multi-agent work exists only in forked repos which add them as extensions. The communication story is equally bare: state is a flat message list, and compaction summaries stand in for structure when the window fills. With four default tools (read, bash, edit, write) and little else, Pi bets that a capable model can improvise whatever scaffolding a task requires, the most trusting position in this study.

<style>
  .post-content table td { font-size: 0.85em; }
  @media (min-width: 1000px) {
    .post-content .table-wide {
      width: 130%;
      margin-left: -15%;
    }
  }
</style>

| Framework | Topology | Who picks the next unit of work | What gets passed along |
|---|---|---|---|
| **Google ADK** | Deterministic graph | The developer, through edges validated at build time | Typed state dict with scoped key prefixes |
| **LangGraph** | Deterministic graph, cycles allowed | The developer, through conditional edges run in super-steps | Typed channels, each merged by its own reducer |
| **OpenAI Agents** | Decentralized swarm | The model, by emitting `transfer_to_X` at runtime | Strict JSON handoff payload plus a typed item log |
| **Assistant Factory** | Stack of Flows | The model ensemble, by pushing and popping Flows | Typed belief state plus a shared session scratchpad |
| **Claude Agent SDK** | Hub-and-spoke | The orchestrator model, by spawning subagents | Prose results, with a fresh context per spoke |
| **OpenCode** | Hub-and-spoke | The primary agent, through a task tool | Typed envelope wrapping a prose payload |
| **Cortex Code** | Hub-and-spoke | The orchestrator model, fenced by data rails | Markdown and SQL |
| **Pi** | Single loop | Nobody, since there are no sub-agents | A flat message list |

*The five topologies at a glance. The middle column is the trust decision and the right column is its consequence, since whoever routes the work also decides how much structure survives the handoff.*

### The Trust Spectrum

At the low-trust end, the architecture expands because a deterministic graph cannot exist without supporting machinery. Such SDKs must include a node abstraction with its own lifecycle, a scheduler to decide what fires, and the typed edges, joins, and checkpoints that keep the graph operational. A recursive loop needs none of that, because the call stack is the schedule and the model decides what fires next. This tax has its benefits though: reproducible execution order, durable mid-run resume after a crash, static validation to catch errors early, and explicit semantics for joining parallel work. The platform and cloud vendors decided that we still can't trust the models enough to meet the stringent requirements of responsible AI by themselves, so they converged on something a bit heavier.

The other end replaces that machinery with the model's own judgment, which can actually be observed by looking at the code. Normalizing for bundled extras, the agent-loop of a trusting harness like Strands comes to about 5,000 lines, while the equivalent slices of the graph-shaped engines run from just under 11,000 to over 13,000.[^14] Pi, the Claude Agent SDK, and the OpenAI Agents SDK stay small because a thin loop plus configuration is all that remains once routing is delegated.[^13] These extra lines are composed of extra guardrails, extra validation checks, and extra type checks. Most of the field sits between the poles, and you can approximately place any SDK by weighing its source tree.

<figure>
<svg viewBox="0 0 760 575" width="100%" role="img" aria-label="Scatter plot of trust in the model versus ease of use across eight agent SDKs" font-family="-apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Helvetica, Arial, sans-serif">
  <rect x="70" y="30" width="660" height="460" fill="none" stroke="#d5dce4"/>
  <line x1="235" y1="30" x2="235" y2="490" stroke="#d5dce4" stroke-dasharray="3 4"/>
  <line x1="400" y1="30" x2="400" y2="490" stroke="#d5dce4" stroke-dasharray="3 4"/>
  <line x1="565" y1="30" x2="565" y2="490" stroke="#d5dce4" stroke-dasharray="3 4"/>
  <line x1="70" y1="375" x2="730" y2="375" stroke="#d5dce4" stroke-dasharray="3 4"/>
  <line x1="70" y1="260" x2="730" y2="260" stroke="#d5dce4" stroke-dasharray="3 4"/>
  <line x1="70" y1="145" x2="730" y2="145" stroke="#d5dce4" stroke-dasharray="3 4"/>
  <text x="60" y="494" text-anchor="end" fill="#5f6b78" font-size="12">1 &middot; none</text>
  <text x="60" y="379" text-anchor="end" fill="#5f6b78" font-size="12">2 &middot; low</text>
  <text x="60" y="264" text-anchor="end" fill="#5f6b78" font-size="12">3 &middot; avg</text>
  <text x="60" y="149" text-anchor="end" fill="#5f6b78" font-size="12">4 &middot; high</text>
  <text x="60" y="34" text-anchor="end" fill="#5f6b78" font-size="12">5 &middot; full</text>
  <text x="70" y="510" text-anchor="middle" fill="#5f6b78" font-size="12">1</text>
  <text x="70" y="524" text-anchor="middle" fill="#5f6b78" font-size="11">dead simple</text>
  <text x="235" y="510" text-anchor="middle" fill="#5f6b78" font-size="12">2</text>
  <text x="235" y="524" text-anchor="middle" fill="#5f6b78" font-size="11">easy</text>
  <text x="400" y="510" text-anchor="middle" fill="#5f6b78" font-size="12">3</text>
  <text x="400" y="524" text-anchor="middle" fill="#5f6b78" font-size="11">average</text>
  <text x="565" y="510" text-anchor="middle" fill="#5f6b78" font-size="12">4</text>
  <text x="565" y="524" text-anchor="middle" fill="#5f6b78" font-size="11">hard</text>
  <text x="730" y="510" text-anchor="middle" fill="#5f6b78" font-size="12">5</text>
  <text x="730" y="524" text-anchor="middle" fill="#5f6b78" font-size="11">steep curve</text>
  <text x="400" y="552" text-anchor="middle" fill="#1e4669" font-size="13" font-weight="600">Ease of use (effort to get up and running) </text>
  <text x="20" y="260" text-anchor="middle" fill="#1e4669" font-size="13" font-weight="600" transform="rotate(-90 20 260)">Trust (how much model autonomy)</text>
  <circle cx="317.5" cy="30" r="6" fill="#2f6fb3"/>
  <text x="327.5" y="34" fill="#2d333b" font-size="13" font-weight="600">Pi</text>
  <circle cx="235" cy="57" r="6" fill="#2f6fb3"/>
  <text x="245" y="61" fill="#2d333b" font-size="13" font-weight="600">Claude</text>
  <circle cx="152.5" cy="145" r="6" fill="#2f6fb3"/>
  <text x="162.5" y="149" fill="#2d333b" font-size="13" font-weight="600">OpenAI</text>
  <circle cx="390" cy="255" r="6" fill="#2f6fb3"/>
  <text x="400" y="260" fill="#2d333b" font-size="13" font-weight="600">Snowflake</text>
  <circle cx="400" cy="317.5" r="6" fill="#2f6fb3"/>
  <text x="410" y="321.5" fill="#2d333b" font-size="13" font-weight="600">OpenCode</text>
  <circle cx="482.5" cy="375" r="6" fill="#2f6fb3"/>
  <text x="492.5" y="379" fill="#2d333b" font-size="13" font-weight="600">Google ADK</text>
  <circle cx="647.5" cy="432.5" r="6" fill="#2f6fb3"/>
  <text x="637.5" y="436.5" text-anchor="end" fill="#2d333b" font-size="13" font-weight="600">LangGraph</text>
  <circle cx="482.5" cy="210" r="7" fill="#d6336c" stroke="#ffffff" stroke-width="2"/>
  <circle cx="482.5" cy="210" r="11" fill="none" stroke="#d6336c" stroke-opacity="0.45"/>
  <text x="496" y="214" fill="#d6336c" font-size="13" font-weight="600">Assistant Factory</text>
</svg>
<figcaption><em>How much each framework trusts the model, plotted against the effort it takes to get running. The points trend along the diagonal because the same scaffolding that constrains the model is what a newcomer has to learn. The empty corners are the real finding, since nobody has decoupled control from onboarding cost.</em></figcaption>
</figure>

Neither end of the spectrum comes for free. Trusting the model means accepting that runs are unrepeatable and that a wrong routing decision leaves no trace beyond a plausible but false transcript explaining itself. Distrusting the model means paying for schemas, validation, and reducers up front, then discovering that the agent cannot properly complete a task because you failed to anticipate an edge case. **Choosing a topology is therefore choosing a failure mode**, which is why this one trust decision impacts so much of the downstream design. What none of the structure addresses is the judgment of the model inside the loop, and that is where every camp, trusting or not, turns out to be exposed.

## Knowing When You Don't Know
*Agents remain as blindly over-confident as the underlying models which power them*

Real users write under-specified requests, leaving out the time range they mean, the account in question, or the constraint that makes the request sensible. An agent that cannot recognize what it does not know fills those gaps with guesses and proceeds with the confidence of the Dunningest Kruger to ever exist. Coding agents get away with this because their end users are developers who can read the diff, run the tests, and catch a wrong commit within minutes. Outside of this domain though, folks are less technical and less aware of the shortcomings of AI. The output is harder to verify, and a confidently wrong answer can travel far before anyone notices. Recognizing uncertainty turns out to be harder than winning IOI gold medals, which leads to [ambiguity as a bottleneck]({% post_url 2025-08-07-ambiguity-is-the-bottleneck %}) to building useful and reliable AI agents.

Across the seven SDKs, the only equipment for handling an unclear request is a single tool that pauses the run and reaches out to a human. Claude's `AskUserQuestion` and ADK's `RequestInput` are genuine clarification hooks, while LangGraph's `interrupt` is placed by the developer instead of raised by the model, firing only where someone predicted help would be needed. A second family of mechanisms has little to do with ambiguity at all, since OpenCode's permission events and OpenAI's tool-approval items gate destructive actions, answering "may I do this" rather than "did I understand this." Pi ships its question tool only as an example extension, so an agent built on the core cannot ask at all.[^15]

That pause-and-ask design assumes the model can recognize when something is ambiguous, an assumption the research record *thoroughly rejects*. Xiong and colleagues found verbalized confidence systematically overstated across five widely used models, with proposed mitigations only partially closing the gap and every method struggling on professional-knowledge tasks.[^16] OpenAI's GPT-4 technical report showed the pre-trained model was well calibrated until post-training damaged that calibration.[^17] Mei and colleagues caught reasoning models verbalizing confidence above 85 percent on wrong answers, with deeper chains of thought making the overconfidence *worse*.[^18] Apple's Illusion of Thinking study watched frontier reasoning models collapse to zero accuracy as puzzle complexity grew, spending *fewer* tokens on harder instances.[^19] The finding extends to tool use: evidence-gathering tools like web search induce extra overconfidence in agents, while verification tools such as code interpreters ground it.[^20] As we move towards the state-of-the-art, **the models haven't improved at recognizing when they don't know something, instead they've only gotten better at hiding it**!

The concession made by the SDKs that models will make mistakes is written into their code. Bounded loops, default turn caps, per-node retries, and permission gates are all budgets for failure, set by people who knew the model would sometimes spin without converging. Every one of those budgets is reactive though, catching the failure only *after* the wrong action was taken. Nothing intervenes before the loop is caught running for too long to measure whether the model should be taking on the task at all. The result is a generation of agents that sometimes recover from being wrong, but almost never notice they might be wrong in the first place. It's a great way to burn a lot of tokens though!

Cortex Code makes an attempt at formalizing this issue through its reflection step, where the model reviews its work after each tool call and chooses to clarify, iterate, or finalize. Assistant Factory is the only framework which treats ambiguity as a first-class concern, providing multiple mechanisms to measure uncertainty that do not depend on self-reporting. Sub-agent routing is decided by an ensemble of voting models, offering a confidence score on the routing decision.[^21] Open-ended requests are grounded to concrete entities like files, tables, or pages to establish shared context before proceeding. And user intents are explicitly tracked so any deviations from the curated world model are flagged immediately. Failing any of these checks triggers a fine-grained ambiguity handler classifying uncertainty into a four-level gradient running from general (the intent is unclear) through partial (the key entity is ungrounded) and specific (a required value is missing) to confirmation (a candidate needs sign-off). All this combined provides the agent with a Theory of Mind to recognize when something doesn't operate as expected.

## Anyone's Game to Win
*The supporting systems around the agent loop have yet to be standardized*

Ask the seven SDKs what an agent should remember and you get two incompatible answers. The coding-style harnesses (Claude, OpenCode, Pi, Cortex, and OpenAI's sandbox layer) keep long-term memory as natural-language markdown files on disk that the agent re-reads on later runs. Markdown memory is lossy but debuggable, editable by hand, and free to design, while typed stores are queryable and hard to corrupt. The orchestration engines go the other way, with LangGraph pairing a per-thread checkpointer with a key-value document store, and Google ADK choosing a typed state object whose key prefixes scope each entry's lifetime.[^22] The split is the trust axis again, since markdown memory trusts the model to re-interpret prose correctly every time, while a typed store relies on a schema the developer wrote in advance.

Startups like Letta and Mem0 push this even further by taking the stance that memory deserves to be its own first-party component which merits a place in the dedicated infrastructure layer.[^23] Natural language search is not enough, and key-value pairs are not enough. Rather, they believe that we should be chunking, indexing, and retrieving just the right documents at just the right time in order to provide the appropriate context to the agent. The hybrid of exact text match and embedding search arguably gives better semantic matching over grepping through files alone. Regardless of what you believe is the right format for agent memory, the absence of a standard format tells you the field has not figured out what works best.

Telemetry splits along a familiar line, between vendors that tell you what quality means and vendors that hand you spans and wish you luck. Google ADK ships prebuilt scorers for tool trajectories and hallucination, and provides traces that are fully OTel-compatible.[^24] Cortex Code bundles LLM-as-judge metrics for groundedness, so adopting either means inheriting its definition of a good agent. The rest emit raw signal. OpenAI, Claude, and OpenCode all offer telemetry of their own flavor which are technically compatible with OpenTelemetry, but only because the consortium allows for extensions to be part of the spec. The extremes are occupied by Pi (minimal logging)[^25] and LangGraph (maximum logging). Perhaps it should come as no surprise that LangGraph has the most opinionated traces because the main revenue driver these days is actually the evaluation suite built into LangSmith.

Memory and telemetry are about as basic as supporting infrastructure gets, and the field disagrees about both at the level of first principles. Nothing here resembles the settled middleware of earlier platform shifts, where a default database or logging stack emerged within a few years, and the absence shows up in practice. I might even venture to say that MCP is up for grabs at this point with minimal support within most SDKs and even less adoption among active developers. At this moment, no incumbent has locked in their advantage and the field is shifting as quickly as ever. This is great news for builders though. Despite the hype, we're still in the early innings, and the opportunity to build a durable, life-changing company is still open to anyone willing to enter the arena.

## Closing Thoughts

The basic description of an agent as a while-loop is accurate (I found a `while` functio in every SDK!), but it stops being useful the moment you try to build one. The loop itself is a few dozen lines everywhere, while the code around it ranges from a few thousand lines to over a hundred thousand.[^26] Everything in that gap is a decision about how far to trust the model, and the same decision echoes through topology, state design, recovery posture, and evaluation. Having gone through this exercise, it is exceedingly clear that agents are designed to serve the vendor's existing business models rather than signaling a wave of innovative agentic commerce.

The practical advice is to **choose your position on the trust axis before you choose an SDK, because the SDK's position will override yours.** If your domain lets users verify output quickly, the more trusting harnesses will get you further faster because their failure modes get caught by the people using them. If a wrong answer is expensive or invisible, the graph camp's reproducibility and reliability machinery is worth its overhead tax, and you should weigh how seriously a candidate SDK treats clarification before trusting it with ambiguous users. There is no right answer yet, and anyone claiming otherwise is selling something. 

What exists today is a set of distinct, internally coherent philosophies, each betting on a different trajectory for the models underneath. Not everyone will be a winner because at some point the dust will settle and the wager on the slope of model improvement will pay off. Trust too early and your agent confidently spews garbage, trust too late and you maintain scaffolding the next model generation turns obsolete. Whichever camp you pick, the most important thing is to just start building because if you take two months to make a decision, the field will have moved on without you.

## Appendix: The Cross-Cutting View

The five dimensions of the comparison with one line per framework:

<div class="table-wide" markdown="1">

| Framework | Philosophy | Composition | Ambiguity | Memory | Observability |
|---|---|---|---|---|---|
| **Claude Agent SDK** | Thin wrapper, model owns the loop | Hub-and-spoke subagents | Model-initiated `AskUserQuestion` | Markdown files plus JSONL transcript | Per-turn cost and usage, no scorers |
| **OpenAI Agents** | Minimal scaffolding, model-driven | Handoffs and agent-as-tool | Guardrails and approval gates | Session transcript plus sandbox markdown | Tracing on by default, no scorers |
| **Google ADK** | Build-validated graph | Explicit nodes and edges | `RequestInput` interrupt | Typed prefixed state plus memory service | OTel-native plus prebuilt scorers |
| **LangGraph** | Developer-owned Pregel runtime | Typed channels and conditional edges | Developer-placed `interrupt()` | Checkpointer plus document store | Routed to paid LangSmith |
| **Pi** | One loop, four tools | None, single agent | None in core | Flat transcript plus markdown | Typed events, no OTel |
| **OpenCode** | Client/server, explicit wiring | Primary and subagent modes | Permission gates | `AGENTS.md` plus session history | Event bus plus per-call cost |
| **Cortex Code** | Claude-style loop fenced for data | Markdown subagents plus data tools | Permission gates plus reflection | Markdown inside the warehouse | TruLens judges in-account |
| **Assistant Factory** | Orchestrator with dynamic sub-agent routing | Stack of Flows | Four-level ambiguity gradient | Three-tier typed cache | Signal envelopes plus three-level evals |

</div>

---

[^1]: `get_default_model()` falls back to `"gpt-5.4-mini"` unless an environment variable overrides it: [`agents/models/default_models.py:99`](https://github.com/openai/openai-agents-python/blob/5a3028f37c74/src/agents/models/default_models.py#L99).
[^2]: "The Claude Code CLI is automatically bundled with the package" ([README](https://github.com/anthropics/claude-agent-sdk-python/blob/3f50b5eb8374/README.md)), located at startup by `_find_bundled_cli()`: [`subprocess_cli.py:114`](https://github.com/anthropics/claude-agent-sdk-python/blob/3f50b5eb8374/src/claude_agent_sdk/_internal/transport/subprocess_cli.py#L114). There are ways to hack around this to use open source models (ie. DeepSeek) as the driver, but this is clearly not intended or allowed.
[^3]: `DEFAULT_MODEL: ClassVar[str] = 'gemini-3-flash-preview'`: [`agents/llm_agent.py:201`](https://github.com/google/adk-python/blob/03671c63f09c/src/google/adk/agents/llm_agent.py#L201). The same core package bundles a 21K-line CLI, an 11.6K-line eval harness, and 158 tool files; 132,900 lines across 610 files in total.
[^4]: `self.model = BedrockModel() if not model else ...`: [`strands/agent/agent.py:258`](https://github.com/strands-agents/sdk-python/blob/ca9569dbabdf/strands-py/src/strands/agent/agent.py#L258).
[^5]: Provider integrations ship as sub-packages of the core [`agent-framework`](https://github.com/microsoft/agent-framework) package, while the managed deployment path runs through [Azure AI Foundry Agent Service](https://learn.microsoft.com/en-us/azure/foundry/agents/overview).
[^6]: Snowflake: [Cortex Code Agent SDK docs](https://docs.snowflake.com/en/user-guide/cortex-code-agent-sdk/cortex-code-agent-sdk). Databricks: [Governing AI agents at scale with Unity Catalog](https://www.databricks.com/blog/governing-ai-agents-scale-unity-catalog). ServiceNow: [Now Assist docs](https://docs.servicenow.com/bundle/zurich-intelligent-experiences/page/administer/now-assist-platform/concept/now-assist-landing.html). Salesforce: [Atlas Reasoning Engine](https://www.salesforce.com/agentforce/what-is-a-reasoning-engine/atlas/) and [Data Cloud](https://www.salesforce.com/news/stories/how-data-cloud-powers-agentforce/).
[^7]: The [Codex repository](https://github.com/openai/codex) is 96 percent Rust per GitHub's language breakdown, with no dependency on the Agents SDK (which ships only in Python and TypeScript).
[^8]: "ADK is the same framework powering agents within Google products like Agentspace and the Google Customer Engagement Suite (CES)", per the [ADK launch post](https://developers.googleblog.com/en/agent-development-kit-easy-to-build-multi-agent-applications/).
[^9]: ADK v2 deprecates `SequentialAgent`, `ParallelAgent`, and `LoopAgent` with the guidance "Please use Workflow instead": [`agents/sequential_agent.py:51`](https://github.com/google/adk-python/blob/03671c63f09c/src/google/adk/agents/sequential_agent.py#L51). The unified graph engine lives in [`src/google/adk/workflow/`](https://github.com/google/adk-python/tree/03671c63f09c/src/google/adk/workflow).
[^10]: LangGraph's runtime is literally a class named `Pregel`: [`pregel/main.py:449`](https://github.com/langchain-ai/langgraph/blob/f0e814796be5/libs/langgraph/langgraph/pregel/main.py#L449). MAF's `Workflow` docstring describes "a Pregel-like model, running in supersteps until the graph becomes idle": [`_workflows/_workflow.py:211`](https://github.com/microsoft/agent-framework/blob/3753d938f5c3/python/packages/core/agent_framework/_workflows/_workflow.py#L211). ADK scopes its shared state keys with `app:` / `user:` / `temp:` prefixes: [`sessions/state.py:64`](https://github.com/google/adk-python/blob/03671c63f09c/src/google/adk/sessions/state.py#L64).
[^11]: Handoff tool names are generated as `transfer_to_{agent.name}`: [`handoffs/__init__.py:173`](https://github.com/openai/openai-agents-python/blob/5a3028f37c74/src/agents/handoffs/__init__.py#L173).
[^12]: Claude subagents are declared as a flat dict of `AgentDefinition`: [`types.py:83`](https://github.com/anthropics/claude-agent-sdk-python/blob/3f50b5eb8374/src/claude_agent_sdk/types.py#L83). OpenCode agents carry a `mode` of `"subagent"`, `"primary"`, or `"all"`: [`agent/agent.ts:38`](https://github.com/anomalyco/opencode/blob/f43b0d3afd80/packages/opencode/src/agent/agent.ts#L38).
[^13]: Pi's loop is `runLoop`, an outer `while (true)` that streams one assistant message per pass: [`agent-loop.ts:155`](https://github.com/earendil-works/pi/blob/406a2214aa1d/packages/agent/src/agent-loop.ts#L155). The Claude Agent SDK contains no loop of its own; the loop ships inside the bundled CLI.
[^14]: Normalization slices: Strands `agent/` + `event_loop/` (18 files, 5,044 LOC); MAF `_agents` + `_tools` + `_clients` + `_middleware` + `_types` (10,855 LOC); ADK `agents/` + `flows/` (52 files, 13,313 LOC). Graph subsystems: Strands fits in one file, [`multiagent/graph.py`](https://github.com/strands-agents/sdk-python/blob/ca9569dbabdf/strands-py/src/strands/multiagent/graph.py) (1,299 lines); ADK's `workflow/` is 26 files and 5,720 lines; MAF's `_workflows/` is 27 files and 12,845 lines. Counted at the June 2026 head of each repo.
[^15]: Claude Code's `AskUserQuestion` is a tool of the CLI ([changelog](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)); ADK ships [`tools/_request_input_tool.py`](https://github.com/google/adk-python/blob/03671c63f09c/src/google/adk/tools/_request_input_tool.py); LangGraph's `interrupt()`: [`types.py:811`](https://github.com/langchain-ai/langgraph/blob/f0e814796be5/libs/langgraph/langgraph/types.py#L811); OpenCode's `permission.asked` event: [`permission/index.ts:12`](https://github.com/anomalyco/opencode/blob/f43b0d3afd80/packages/opencode/src/permission/index.ts#L12); OpenAI's `ToolApprovalItem`: [`items.py:502`](https://github.com/openai/openai-agents-python/blob/5a3028f37c74/src/agents/items.py#L502).
[^16]: (Xiong et al., 2024) [Can LLMs Express Their Uncertainty? An Empirical Evaluation of Confidence Elicitation in LLMs](https://arxiv.org/abs/2306.13063), ICLR 2024.
[^17]: (OpenAI, 2023) [GPT-4 Technical Report](https://arxiv.org/abs/2303.08774); see the MMLU calibration plots comparing the pre-trained and post-trained models.
[^18]: (Mei et al., 2025) [Reasoning about Uncertainty: Do Reasoning Models Know When They Don't Know?](https://arxiv.org/abs/2506.18183)
[^19]: (Shojaee et al., 2025) [The Illusion of Thinking: Understanding the Strengths and Limitations of Reasoning Models via the Lens of Problem Complexity](https://machinelearning.apple.com/research/illusion-of-thinking), Apple Machine Learning Research.
[^20]: (Xuan et al., 2026) [The Confidence Dichotomy: Analyzing and Mitigating Miscalibration in Tool-Use Agents](https://arxiv.org/abs/2601.07264)
[^21]: From Assistant Factory's internal evals on flow-detection benchmarks: routing by an ensemble of three models voting outperformed single-model routing by 12%.
[^22]: LangGraph's `BaseCheckpointSaver`: [`checkpoint/base/__init__.py:176`](https://github.com/langchain-ai/langgraph/blob/f0e814796be5/libs/checkpoint/langgraph/checkpoint/base/__init__.py#L176) and `BaseStore`: [`store/base/__init__.py:700`](https://github.com/langchain-ai/langgraph/blob/f0e814796be5/libs/checkpoint/langgraph/store/base/__init__.py#L700); ADK's prefix-scoped state: [`sessions/state.py:64`](https://github.com/google/adk-python/blob/03671c63f09c/src/google/adk/sessions/state.py#L64).
[^23]: Mem0 calls itself a "universal memory layer for AI agents" ([github.com/mem0ai/mem0](https://github.com/mem0ai/mem0), [Chhikara et al., 2025](https://arxiv.org/abs/2504.19413)); Letta is the company behind MemGPT ([Packer et al., 2023](https://arxiv.org/abs/2310.08560)).
[^24]: `PrebuiltMetrics`: [`evaluation/eval_metrics.py:43`](https://github.com/google/adk-python/blob/03671c63f09c/src/google/adk/evaluation/eval_metrics.py#L43); OpenTelemetry instrumentation: [`src/google/adk/telemetry/`](https://github.com/google/adk-python/tree/03671c63f09c/src/google/adk/telemetry).
[^25]: Pi's [`observability.md`](https://github.com/earendil-works/pi/blob/406a2214aa1d/packages/agent/docs/observability.md) states the goal: observable "without depending on OpenTelemetry, Sentry," via a typed `AgentEvent` stream.
[^26]: MAF caps model round-trips at `DEFAULT_MAX_ITERATIONS = 40`: [`_tools.py:90`](https://github.com/microsoft/agent-framework/blob/3753d938f5c3/python/packages/core/agent_framework/_tools.py#L90). Strands' loop is the recursive `recurse_event_loop`: [`event_loop/event_loop.py:400`](https://github.com/strands-agents/sdk-python/blob/ca9569dbabdf/strands-py/src/strands/event_loop/event_loop.py#L400). The ADK core package is 132,900 lines across 610 files.
