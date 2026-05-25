# Agent Harness Survey — Blog Post Angles

> **Handoff doc.** Written 2026-05-20 for a future agent to pick up the conversation. Self-contained — the next agent will not have seen the prior conversation, so everything load-bearing is captured below.

---

## 1. Context

**Author:** Research engineer running an AI consultancy. Building Hugo, a multi-flow blog-writing assistant. Strong on agents, model internals, eval discipline; less interested in K8s/CI/infra. Writes in a punchy, comparison-driven style with low tolerance for fluff.

**Conversation arc:** A long strategic survey of ~14 agent harnesses and SDKs, moving from MCP / Zed / Pi / Cognition / Cursor / Magic / Augment / OpenHands ToM-SWE → Hermes (Nous Research) → dialogue state research lineage (PABU, FnCTOD, ToM-SWE, CoBel-World, ABBEL) → the SDK-led vendors (Gemini ADK, AWS Strands) → the platform vendors (AgentForce, Snowflake Cortex, Databricks Mosaic AI, Microsoft Copilot Studio, ServiceNow Now Assist). Mid-conversation also covered a practical debugging session on Hugo's reliability problems, which is NOT the subject of the blog post but explains the author's first-person motivation.

**Goal:** The author wants to write a blog post summarizing the key findings. He named the obvious survey angle as "boring" and asked for two other angles. He picked the recommendation (Angle C with B distilled in and A as appendix) and asked for this handoff doc before commissioning the actual draft.

---

## 2. The Harnesses Covered

Roughly organized by family. The conversation touched on more than 14; this is the working set.

### Frontier (trust-the-model)
- **Claude Code** (Anthropic) — free-loop agent, model picks tools and termination. Sets the bar.
- **Pi** (Imbue) — also free-loop. Punches above its weight via grounding, clarification, pre-specified workflows.

### SDK + Graph (dev picks topology, model fills nodes)
- **Gemini ADK** (Google) — free SDK, monetizes via Vertex AI / Gemini tokens.
- **AWS Strands** — free SDK, monetizes via Bedrock.
- **Semantic Kernel** (Microsoft) — pro-dev path that pairs with Copilot Studio's citizen-dev path.

### Vertical specialists (one domain, opinionated)
- **Cognition / Devin** — coding agent, lost to Claude Code despite huge funding.
- **Cursor** — won the coding IDE war by staying light.
- **Magic** — over-engineered, lost.
- **Augment** — over-engineered, lost.
- **OpenHands ToM-SWE** — academic-flavored, theory-of-mind framing. Documentation gestures at TOM but the actual TOM logic is buried/absent.
- **Hermes** (Nous Research) — "grown-up OpenClaw," developer-targeted, monetization via hosting.
- **Zed / ACP** — editor-led, narrow scope, agent context protocol attempt.

### Platform incumbents (vendor decides topology, moat defense)
- **Salesforce AgentForce** — defends the customer record. Topics + Actions + Trust Layer.
- **Snowflake Cortex Agents** — defends the warehouse. Cortex Analyst (text-to-SQL via semantic model YAML) + Cortex Search.
- **Databricks Mosaic AI Agent Framework** — defends the lakehouse. Agent Bricks, MLflow tracing, AI Gateway. Has a pro-dev escape hatch.
- **Microsoft Copilot Studio** — defends the productivity graph (Microsoft Graph spans email/calendar/Teams/SharePoint/OneDrive). Citizen-dev tool; Semantic Kernel + Azure AI Foundry is the parallel pro-dev path.
- **ServiceNow Now Assist** — defends the workflow primitive. Agents are smart workflow steps. Now LLM is the only vertical fine-tune in this group.

### Protocol layer (orthogonal to harnesses)
- **MCP** (Model Context Protocol) — Anthropic's tool/context standard.
- **ACP** (Agent Context Protocol) — Zed's attempt.
- **A2A** (Agent-to-Agent) — Google's cross-agent protocol.

### the author's own
- **Hugo** — 48 flows × 16 slot types, 7-component architecture, 4-level ambiguity handler. Lives in `~/Documents/repos/personal_assistants/assistants/Hugo`. Cited in the blog only via first-person framing ("I went on this tour because…").

---

## 3. Key Vocabulary Developed in the Conversation

The blog post should use these terms — they were developed across the conversation and are load-bearing.

### The Topology Axis
Who decides the loop shape:
| Topology | Who picks tools | Who picks loop shape | Examples |
|---|---|---|---|
| Free loop | Model | Model | Claude Code, Pi |
| Graph | Model | Dev (DAG) | ADK, Strands, Semantic Kernel |
| Flow set | Model (within flow) | Dev × N | Hugo (48 flows) |
| Vendor singleton | Dev (config) | Vendor (one shape) | AgentForce, Cortex, Copilot Studio, Now Assist |

### The Trust Spectrum
How much the runtime trusts the model:
- **Anthropic / Claude Code:** trust everything
- **Google ADK:** trust the model, guardrails as hooks
- **Hugo:** trust the model inside a flow, guardrails in the policy
- **Vendors:** don't trust the model; the "agent" is a hardwired process with a few tool calls

### Data Plane Defense Map
What each vendor is actually protecting:
| Vendor | Defends |
|---|---|
| Salesforce | Customer record |
| Snowflake | Warehouse query |
| Databricks | Lakehouse + ML lifecycle |
| Microsoft | Productivity graph (Graph) |
| ServiceNow | Workflow primitive |
| Oracle / Workday / SAP | Their record systems |

### Pro-dev Escape Hatch Map
| Vendor | Citizen-dev | Pro-dev escape | Both lanes serious? |
|---|---|---|---|
| Microsoft | Copilot Studio | Azure AI Foundry + Semantic Kernel | Yes |
| Databricks | Agent Bricks | MLflow + AI Gateway + LangGraph deploy | Yes |
| Salesforce | Agent Builder | Apex/Flow only | No |
| Snowflake | Cortex UI | Cortex API + semantic model YAML | No |
| ServiceNow | Now Assist | Some SDK, mostly closed | No |

### Two Axes for the Trust Question
1. How much does the *runtime* trust the model? (Anthropic full → vendors barely)
2. How much does the *vendor* invest in the model itself? (Anthropic fully → ServiceNow narrowly via Now LLM → everyone else: rent from frontier labs)

The vendor crowd looks uniform on axis 1 but ServiceNow is the outlier on axis 2.

### Buyer Determines Shape
- Anthropic: individual devs → trust the model
- Cursor / Cognition: devs → coding-shaped
- Salesforce: VP of Service / Sales → can't fail, hide the LLM, ~$2/conversation
- Snowflake: data analyst → answer the SQL question
- ServiceNow: workflow owner → agents must live inside Flow Designer
- Microsoft: both citizen-dev AND pro-dev because they sell to both buyers

---

## 4. The Three Blog Angles (Full Outlines)

### Angle A — The Field Guide (the boring one)

**Title:** "A Tour of 14 Agent Harnesses (And What They Actually Do)"

**Thesis:** Catalog and compare. Useful, not memorable.

**Outline:**
1. Why I went on this tour (the demo-vs-reality gap, building Hugo)
2. The four families
   - Frontier: Claude Code, Pi — trust the model, thin scaffolding
   - SDK + graph: Gemini ADK, AWS Strands, Semantic Kernel
   - Vertical specialists: Cognition/Devin, Cursor, Magic, Augment, OpenHands ToM-SWE, Hermes
   - Platform incumbents: AgentForce, Cortex, Mosaic, Copilot Studio, Now Assist
3. The orthogonal protocol layer: MCP, A2A, ACP
4. Where Hugo sits
5. Pros/cons matrix
6. "Pick one for your task" decision tree

**When to use as standalone:** Don't. It belongs as an *appendix* to B or C — useful as reference material but not as the post.

---

### Angle B — Topology Is Destiny

**Title:** "Your Agent's Loop Shape Decided Everything Before You Wrote a Line"

**Thesis:** The shape of the loop — who picks it, how many coexist, how much latitude the model gets inside it — predetermines what tasks the harness can possibly succeed at. Most "agent failures" are topology mismatches, not LLM failures. People obsess over model choice and prompting; almost nobody talks about loop shape, even though it's the most consequential decision they make.

**Outline:**
1. **The blind spot.** Every blog post talks about which model, which prompt, which tool. Almost nobody talks about who picks the loop. But that's the choice that bounds everything else.
2. **The four canonical topologies.**
   - **Free loop** — model picks tools, model picks termination. Claude Code, Pi. Highest ceiling, hardest to make production-safe.
   - **Graph** — dev picks a DAG, model fills nodes. ADK, Strands, Semantic Kernel. Predictable but inflexible.
   - **Flow set** — dev picks N topologies, model navigates within and across. Hugo. Highest investment, only viable when tasks fall into a stable set of shapes.
   - **Vendor singleton** — vendor picks one shape, customer fills tools. AgentForce, Cortex, Copilot Studio, Now Assist. Cheap to author, ceiling fixed by the vendor.
3. **Why topology is sticky.** It constrains state (does dialogue state exist?), eval (can you write a trajectory test?), ops (where do you catch errors?), debugging (what does a "wrong path" look like?), and what failure modes are possible. You can swap models; you can't swap topology without rewriting the harness.
4. **The task-shape match.**
   - Single-shot Q&A over your data → vendor singleton is fine
   - Open-ended coding → free loop is the only thing that works
   - Multi-turn task with structural state ("now refine section 2 and smooth transitions") → flow set
   - Pipeline orchestration with branching → graph
5. **Where the losers lost.** Cognition picked graph-shaped scaffolding for a task that needed free loop. Magic and Augment over-engineered the graph and lost to Cursor's lighter shape. AgentForce picks singleton for tasks the customer actually needed multi-turn. Even Hugo's recent CategorySlot rigidity is a flow-set instance picking the wrong shape *within* the right family.
6. **The escape hatch problem.** Citizen-dev shape + pro-dev shape side by side: only Microsoft and Databricks pull it off. The rest force you to pick a lane.
7. **Conclusion.** Pick your topology before your model. Most teams do the reverse and pay for it for the rest of the product's life. A free-loop product can adopt new models in an afternoon; a graph product can't become free-loop without a rewrite.

**Strength:** Technical, useful, gives builders a concrete framework. Lands well with the SWE-Twitter / HN crowd.

---

### Angle C — The Agent Is the Moat

**Title:** "Nobody Is Building an Agent. They're Defending a Data Plane."

**Thesis:** Strip away the marketing and the entire vendor wave is moat preservation. The "agent" is a feature that prevents customers from moving workflows off-platform. Every design choice — hardwired topology, hidden guardrails, swappable LLM, mandatory grounding — follows from that motive, not from any opinion about what agents should be. Once you see it, you can't unsee it, and it explains the otherwise-baffling sameness of AgentForce, Cortex, Copilot Studio, and Now Assist.

**Outline:**
1. **The cognitive dissonance.** Five well-funded vendors announce "AI agents" and the products are eerily similar: one shape, vendor-decided loop, hidden guardrails, narrow scope, swappable LLM. Compare to Anthropic / Cursor / Devin, which all look completely different from each other.
2. **The wrong explanation:** they're all bad at building agents.
3. **The right explanation:** they aren't *trying* to build agents. They're trying to keep your data on their platform. The agent is the wedge.
4. **The defense map.** Each vendor protects a different data primitive:
   - Salesforce: the customer record
   - Snowflake: the warehouse query
   - Databricks: the lakehouse + ML lifecycle
   - Microsoft: the productivity graph (Graph spans email, calendar, files, Teams)
   - ServiceNow: the workflow primitive
   - Oracle / Workday / SAP: their respective record systems
5. **How moat defense explains every "weird" design choice.**
   - The "Trust Layer" / "AI Gateway" exists to keep data inside the security boundary the customer already paid for
   - Topology is hardwired because a model that decides too much could route around the platform
   - Grounding is mandatory because every retrieval is a hook back into the data plane
   - The LLM is swappable because the model isn't their product — committing to one would forfeit pricing leverage
   - Citizen-dev UI builders dominate because the buyer is the VP who signs the seat-renewal, not the dev who'd want an SDK
6. **The two outliers worth naming.**
   - **ServiceNow's Now LLM** — the only vendor investing in the model itself, because they own a vertical narrow enough to fine-tune for
   - **Microsoft's two-lane play** — Copilot Studio for citizen-dev *and* Azure AI Foundry/Semantic Kernel for pro-dev. The only vendor running both lanes deliberately.
7. **Implication for buyers.** Don't buy AgentForce expecting a good agent. Buy it expecting your AI to stay inside Salesforce's security boundary. That's the actual product. The agent is the wrapping paper. (Same for the others.)
8. **Implication for builders.** If you're not defending a data plane, you can't beat these vendors on their ground. But they also can't beat you outside it. The independent-agent market (Claude Code, Cursor, Devin, Pi, Hugo) and the platform-agent market are not actually competing — they're selling different products to different buyers, in the same vocabulary.
9. **Implication for the model labs.** Anthropic is the only major player whose product *is* the agent. Everyone else is selling something else and using the agent as a wedge. That's why "trust the model" is a viable strategy for exactly one company. The rest can't trust it — the model isn't theirs to trust.

**Strength:** Provocative, contrarian, reframes the discourse. The "agent ≠ agent" insight is the kind of thing that lands on HN front page or gets quoted around. Higher upside, but you commit to the position.

---

## 5. Recommendation

**Lead with Angle C; bake B into the middle; ship A as the appendix.**

- **C as the framing** — gives the post its provocative thesis and one-line summary ("Nobody is building an agent. They're defending a data plane.")
- **B distilled into the technical middle section** — gives the post engineering substance. The topology table is the keystone visual.
- **A as appendix** — gives readers the reference material so the post stands on its own without forcing them to cross-reference 14 product pages.

The rationale: C alone would feel like a hot take without proof. B alone would feel like an engineering taxonomy without a thesis. A alone is the boring survey the author explicitly didn't want. Combining them gives both the spice and the credibility.

**Suggested structure for the final post:**
1. Lead with the cognitive dissonance from Angle C (the products are eerily similar — why?)
2. Walk through the moat-defense reframe (Angle C body)
3. Pivot: "OK but if moat defense explains the vendors, what explains the *non-vendor* harnesses' diversity?" → into Angle B (topology decisions)
4. Show the topology × buyer matrix that ties C and B together
5. Implications for buyers / builders / model labs (from C)
6. Appendix: the 14-harness field guide (A)

---

## 6. Notes for the Next Agent

### What's decided
- The author wants to write the post (not just plan it).
- He picked C+B+A combined.
- He works in a punchy, comparison-driven voice. Match it: short paragraphs, opinionated takes, tables where they earn their space. Avoid fluffy intro/outro paragraphs.

### What's NOT decided (ask before assuming)
- **Venue.** Soleda blog? the author's personal? Substack? Different venues might warrant different lengths and tones.
- **Length.** No target word count discussed. Suggest asking — 2000-word essay vs 5000-word deep-dive change the structure.
- **Whether Hugo is named.** the author's company is Soleda; Hugo is internal-ish. Worth asking whether to name Hugo specifically or refer to it abstractly as "the harness I'm building."
- **Citations / sources.** The author hasn't said whether they want academic citations (PABU, FnCTOD, ToM-SWE, ABBEL, CoBel-World came up earlier in the conversation re: dialogue state tracking) or just vendor product references.
- **Tone calibration on the vendor critique.** The "moat defense" framing is sharp. Worth checking whether The author wants to soften any of it (the Salesforce / Snowflake / Microsoft / etc. teams will read it).

### Things worth verifying before publishing
- All vendor product names, tier names, pricing numbers — these change frequently. AgentForce pricing was ~$2/conversation as of writing; Now Assist tiers around $40-50/user/mo; Copilot Studio is consumption-based on messages. Check current pricing pages before quoting numbers.
- "ServiceNow's Now LLM beats GPT-4 on ServiceNow-specific tasks" — this was ServiceNow's claim, not an independently verified benchmark. Phrase accordingly.
- The "OpenHands ToM-SWE has no actual TOM implementation in the code" claim — the author confirmed they couldn't find it in the public docs/code, but the next agent should re-check before publishing.

### Hugo context (if The author wants first-person framing)
- 7 components: World, FlowStack, DialogueState, TaskArtifact, AmbiguityHandler, MemoryManager, ContextCoordinator, PromptEngineer
- 48 flows across Research(7) / Draft(7) / Revise(7) / Publish(7) / Converse(7) / Plan(6) / Internal(7)
- 16 slot types (12 + 4 special)
- 4-level AmbiguityHandler (general / partial / specific / confirmation)
- Turn pipeline: Agent → NLU → PEX → RES
- Recent learning: CategorySlot rigidity exposed in Rework — once classified, agentic path never runs. This is the concrete "even I picked the wrong topology within a flow" example for Angle B section 5.

### Stylistic guardrails from the author's prior feedback
- No fluffy intros ("In this post, we'll explore…")
- No trailing summaries ("In conclusion, we've seen that…")
- Short sentences over long ones
- Tables only when they earn the space
- Opinions stated directly; don't hedge with "some might argue"
- First-person is fine and expected

---

## 7. Files / Links the Next Agent Might Need

- This doc: `/Users/derekchen/Desktop/sdk/agent-harness-blog-ideas.md`
- Hugo code: `/Users/derekchen/Documents/repos/personal_assistants/assistants/Hugo/`
- Prior conversation transcript: `/Users/derekchen/.claude/projects/-Users-derekchen-Desktop-sdk/` (most recent .jsonl)


---

As open-source models continue to catch up their closed-source equivalents, the value in AI ecosystem will bubble up towards the application layer. Every participant is valued right now as if they alone at the platform of choice, when they can't *all* be winners. The bubble will pop before we reach AGI. Big models alone aren't enough to solve real business problems. The next layer is the agent architecture, and finally the domain knowledge.

 Commodification of models 



