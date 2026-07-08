---
layout: post
title: "Meeting Users Where They Are"
subtitle: "How AI Agents Can Be Made Accessible to Everyone"
tags: [agents, ai, ambiguity, dialogue, explainer, trustworthy]
color: rgb(60, 100, 90)
---

What does it mean for AI to be fair? Well, one definition entails making sure predictions do not produce outputs that systematically disadvantage any protected group. Another angle is to look at fairness through the lens of the user. In this sense, AI that is fair and equitable comes down to being equally accessible to people regardless of their background, skin color, gender, and, crucially, skill level.  *AI for Everyone* means that you don't have to just be a programmer to gain all the advantages offered by state of the art models. It means your personal assistant works reasonably well even when you would never consider yourself a prompt engineer.

<!--more-->

AI coding agents have hammered the software engineering field harder than when the Hulk smashed Loki around like a rag doll. No one is denying anymore that we are undergoing a seismic shift in how software is built, an earthquake whose magnitude is still growing and whose downstream impacts have yet to be fully realized. At the same time, there have been countless articles about how AI is supposed to take over design, consulting, medicine, law, and journalism.[^1] Well, 3 years in and radiology still hasn't disappeared as a profession.[^2] And what about every other profession not mentioned? Are they just supposed to wait on the sidelines while everyone else shoots forward?

One might argue that it's everyone's individual responsibility to upskill themselves, which is largely true! But with new models coming out literally every week (see GPT-5.5, DeepSeek V4, Gemini 3.5 Flash, Opus 4.8, Microsoft MAI, Fable 5, Kimi K2.7, GLM 5.2, GPT-5.6 Sol, Sonnet 5) [^3], and new trends popping up before you can absorb the previous one (see [swarms](https://github.com/openai/swarm), [OpenClaw](https://github.com/openclaw/openclaw), and [loops](https://thenewstack.io/loop-engineering/)), is it any wonder that people aren't really digging the AI revolution?[^4] If millions are feeling left behind by AI, can all of it be attributed to user error alone?[^5] Can 55% of workers *all* be holding it wrong? [^6]

When presented with something ambiguous, a human will ask for clarification. (Equally important, when asked something clear and straightforward, a human will not over-ask either.) Humans come in with their own expectations of what they believe is right, and will behave accordingly. For better or for worse, AI models don't contain this property. During one session it may vehemently insist it has saved all changes to the file, and upon a refresh, the model now acquiesces on the issue by shamelessly declaring that "you're absolutely right!" Practically speaking, new technology *should* adapt to the user, but not like this. Even if AI is conscious, perhaps *especially* if the technology is conscious, we should expect reliable and trustworthy judgment that holds steady from one session to the next.

## Recognizing Uncertainty

It turns out that even recognizing that a user's request may be ambiguous is actually deceptively difficult.[^7] This is due to several factors which feed into each other.

### Multiple Levels of Ambiguity

The first aspect of complexity is that uncertainty doesn't arise for just one type of ambiguity. Situations can be ambiguous at multiple layers.[^8] **General** ambiguity occurs when a single utterance can match several intents at once. Given a calendar assistant, "Fix my Thursday" could mean clear the conflicts, decline the invitations, or protect focus time. A single model picks the likeliest reading and commits, sounding certain while the user has no idea whether the actions taken were genuinely correct or just a lucky guess.

With the intent settled, we turn our attention to requests such as "move that meeting" or "invite her" that leave the entity open — which meeting, who is she? This second level of **partial** ambiguity occurs when we roughly know what the user wants, but it's not clear what person or thing they are referring to. A model will take a shortcut by binding to the most salient candidate and proceeding. The user must audit which meeting was actually moved, defeating the purpose of having the assistant in the first place.

"Reschedule the sync to next week" with no time, or "make it shorter" with no length, leaves a required value blank even when the intent and core entity are known. Under vague instructions, models "arbitrarily generate the missed item," and a confident hallucination follows [^9]. Research has shown that underspecified inputs regress accuracy about twice as often as specified ones [^10], which can be thought of as **specific** ambiguity.

Finally, we consider a **confirmation** level ambiguity whereby the assistant has some clear candidates in mind, but needs approval on which of the several options is the preferred one. At this point, we're pretty sure we know what the right answer is, but we just want to make sure. Overall, ambiguity isn't one problem to solve but four separate layers, and a single request can be unclear at more than one of them at once.

### Grounding to Actual Data

Another aspect of ambiguity is that a user's request is heavily situation dependent. Given a query of "How much money did we make last month?", the uncertainty depends on what data is available in the database. If there is a column for revenue and a column for date, then we're in luck! But what if we don't? Maybe the columns are called 'Gross Payments' and 'Charge Date'? What if the formula for revenue in fact requires 3 different columns summed together? What if accounting wants 'the last month' as the last full month, while marketing considers 'the last month' as month-to-date?

So far, we've also been assuming that we have the right permissions to access the data. What if we know the formula, but we don't have read-access to that third table which we need to adjust for expenses? Or what if we do have access, but the latest information just hasn't been populated yet because the servers hit an issue overnight and there's a backlog of orders to process before we get the final numbers? All this, and we're still dealing with the same original user request.

### Lacking a Theory of Mind

Ultimately, current LLMs fail to recognize ambiguity because they don't hold a sense of what they expect from the user. Only by establishing their own beliefs can the model recognize when something has deviated from their world view. Furthermore, they also lack a model of the person across from them: what the user already knows, what they want, and what they've left unsaid on purpose. What we're left with then is a system that mistakes its own assumptions for the user's intent, while treating every instruction as if it were already complete.

Gaining a Theory-of-Mind is far from solved though. While there have been attempts to instill such a capability into AI agents, the most recent findings have merely shown that the apparent skill rides on surface patterns. Models handle the textbook version but break on adversarial variants of the same task.[^11] [^12]  There are also models that ace the *literal* task of predicting what someone will do, but that says little about the functional theory-of-mind ability an assistant needs to accomplish real-world tasks.[^13] As things stand today, reasoning models stay overconfident on ambiguous inputs, and chain-of-thought does not fix their calibration [^14] [^15].

## Recovering from the Unknown

Once we've recognized that something doesn't seem right, the next step is to fix the issue. Given the variety of sources of confusion, there are a corresponding number of techniques that may be deployed.

The core problem with 'general' ambiguity is that the agent doesn't know what the user wants. When a single model makes a prediction, the model may be biased or miscalibrated. Thus, a solution is to measure agreement across a tiered ensemble of voters.[^16] Disagreement among voters gives an uncertainty signal, while a unanimous reading represents strong confidence. More sophisticated versions can even weight the voters or allow them to abstain. The real effort though is method for quickly and easily enumerate all the possible intents that a user may express, which is now viable, but will be covered in a separate post.[^17]

Moving onto 'partial' ambiguity, the agent must be instructed to identify a grounding artifact before proceeding. For a writing assistant, the grounding entity may be a blog post. For a data analysis assistant, the grounding entity may be a table, row or column. For our calendar assistant, this could be a specific calendar event or attendee. An empty entity should force a hard stop, which requires resolving with the user rather than guessing. Because the entity slot is declared and checked, the agent cannot finish a reschedule until it knows which event it is rescheduling.

For 'specific' ambiguity and grounding issues, a natural solution is to examine the grounding entity. Concretely, suppose we don't know if we should use Table A or Table B. Then the assistant can peek at the first 100 rows of each table to see which one seems more relevant. Don't know which calendar event should be removed? Read the details of the event to decide. Other related techniques include retrieving stored user preferences or previous conversations to see what has already been decided in the past. Strong assistants have memory stored at multiple levels, going far beyond RAG on documents.

The best way around 'confirmation' issues is to ask the question. Intelligent questions move from the generic "what time?" into a higher value "10am, 2pm, or 4pm?" where each option comes from actual open spots on the calendar. Given the techniques mentioned above, asking a clarification question is often the last resort, but we shouldn't hesitate to go there if that's what it takes to resolve the issue.[^18] Useful assistants know what to ask, rather than placing the burden on the user to know what to prompt. 

## Full Spectrum of Accessibility

Ambiguity handling lets a user stay an ops manager or a nurse rather than forcing them to moonlight as a part-time engineer. Today's agents push the cognitive load of proper communication onto the user: learn to prompt, specify precisely, verify the output. However, the user should be able to receive a good answer without auditing a hidden decision, which is only possible when the model surfaces its reasoning and proactively calls out when a proposed action might be incorrect or carry unforeseen adverse consequences. Without that, AI becomes regressive as it rewards people who already have the skills. A fair system inverts the arrangement so the party with more capacity (the AI agent) carries the heavier burden.

Underspecified language acts as only one pillar of ambiguity though. Context ambiguity is just as real, since a request means nothing without the data and the surroundings it points to, which is why a fair assistant has to perceive the world the way its user does. Multimodal perception is not a bonus feature in that light. A friend of mine had carpal tunnel and couldn't type for months, and for that whole stretch an assistant that only accepted typed commands may as well not have existed. Bolting a screen reader onto a text interface is not sufficient. To meet people with different vision, hearing, or mobility needs, the model has to work differently from the ground up, treating speech, images, and gestures as first-class input rather than as accommodations tacked on at the end.

Perception runs deeper than the five senses too. How can a model form a real theory of what we want if it can't tell how we feel? Even with flawless ASR, a system that registers only our words misses most of what we are actually saying. Roughly half of human communication is carried through non-verbal signals like tone, expression, and timing rather than the words themselves. Until an assistant can catch the pause before a reluctant "yes," the sigh that means "fine, but I'm not happy about it," or the glance at the clock that means "wrap this up," it will keep reading politeness as agreement and compliance as consent.

As we expand our definition of fairness, more traditional accessibility gaps come into view. Coverage of low-resource languages is a clear case where models still fall short simply due to the overwhelming English-based training data.[^19] Because models are billed per token, and low-resource languages splinter into far more tokens per sentence, their speakers are charged several times over for the same request while getting worse answers back[^20]. At some level, we hope it just stops with poor performance for marginalized communities, but the reality is, unfortunately, outright prejudice. 

Even within English, dialect coverage is another area in dire need of attention. Studies have found LLMs judging speakers of African American English more harshly than speakers of other dialects, assigning them less prestigious jobs and stiffer criminal sentences.[^21] Furthermore, these fears are no longer hypothetical. Audits of AI candidate screening tools have found the same résumé ranked lower when encountering names that are associated with Black or female applicants.[^22] Consequently, meeting users where they are is not a feature to bolt on once the model is smart enough, but rather what turns a powerful model into one that actually belongs to everyone.

---

[^1]: (Goldman Sachs, 2023) [Generative AI could expose 300 million jobs to automation](https://www.forbes.com/sites/jackkelly/2023/03/31/goldman-sachs-predicts-300-million-jobs-will-be-lost-or-degraded-by-artificial-intelligence/), reported in Forbes.

[^2]: (Fortune, 2026) [A decade after the 'Godfather of AI' said radiologists were obsolete, their salaries are up to $571K and demand is growing fast](https://fortune.com/2026/05/04/godfather-of-ai-geoffrey-hinton-radiologists-future-of-work-tech-ai-job-anxiety/). Geoffrey Hinton said in 2016 that we should "stop training radiologists"; the profession has grown instead.

[^3]: GPT-5.5 (Apr 23), DeepSeek V4 (Apr 24), Gemini 3.5 Flash (May 19), Opus 4.8 (May 28), Microsoft MAI (Jun 2), Fable 5 (Jun 9), Kimi K2.7 (Jun 12), GLM 5.2 (Jun 13), GPT-5.6 Sol (Jun 26), and Sonnet 5 (Jun 30) — ten frontier models within a span of ten weeks, April to June 2026. See the [AI Model Release Timeline 2025–2026](https://aiflashreport.com/model-releases.html).

[^4]: (NBC News, 2025) [Former Google CEO Eric Schmidt booed during graduation speech about AI](https://www.nbcnews.com/tech/tech-news/former-google-ceo-booed-graduation-speech-ai-rcna345585).

[^5]: (Pew Research Center, 2026) [Key findings about how Americans view artificial intelligence](https://www.pewresearch.org/short-reads/2026/03/12/key-findings-about-how-americans-view-artificial-intelligence/).

[^6]: (The Harris Poll for MasterClass, 2025) [Most Employees Don't Know How to Adopt AI](https://www.newsweek.com/most-employees-dont-know-how-adopt-ai-survey-2128604), Newsweek.

[^7]: (Zhang et al., 2024) [CLAMBER: A Benchmark of Identifying and Clarifying Ambiguous Information Needs in Large Language Models](https://arxiv.org/abs/2405.12063), ACL 2024.

[^8]: (Paek and Horvitz, 2000) [Conversation as Action Under Uncertainty](https://arxiv.org/abs/1301.3883), UAI 2000. Models dialogue as breakdowns that can happen at four levels — channel, signal, intention, and conversation.

[^9]: (Wang et al., 2024) [Learning to Ask: When LLMs Meet Unclear Instruction](https://arxiv.org/abs/2409.00557), 2024.

[^10]: (Yang et al., 2025) [What Prompts Don't Say: Underspecified Instructions in Code Generation](https://arxiv.org/abs/2505.13360), 2025.

[^11]: (Ullman, 2023) [Large Language Models Fail on Trivial Alterations to Theory-of-Mind Tasks](https://arxiv.org/abs/2302.08399). Small, logically irrelevant edits to a false-belief task flip the model's answer.

[^12]: (Shapira et al., 2024) [Clever Hans or Neural Theory of Mind? Stress Testing Social Reasoning in Large Language Models](https://arxiv.org/abs/2305.14763), EACL 2024. Apparent theory-of-mind rests on shallow heuristics and breaks on adversarial examples.

[^13]: [Position: Theory of Mind Benchmarks are Broken for Large Language Models](https://arxiv.org/abs/2412.19726), ICML 2025.

[^14]: (Mei et al., 2026) [Reasoning about Uncertainty: Do Reasoning Models Know When They Don't Know?](https://arxiv.org/abs/2506.18183), Findings of EACL 2026.

[^15]: [How Much Can You Trust Your Agent?](/2026/06/08/how-much-can-you-trust-your-agent.html), 2026.

[^16]: [Reliable Agents from Unreliable Parts](/2026/05/18/reliable-agents-from-unreliable-parts.html), 2026.

[^17]: (Kim et al., 2026) [DiscoverLLM: From Executing Intents to Discovering Them](https://arxiv.org/abs/2602.03429), ICML 2026.

[^18]: (Kong et al., 2026) [InfoPO: Information-Driven Policy Optimization for User-Centric Agents](https://arxiv.org/abs/2603.00656), 2026.

[^19]: [Language Ranker: A Metric for Quantifying LLM Performance Across High and Low-Resource Languages](https://arxiv.org/abs/2404.11553), 2024.

[^20]: (Ahia et al., 2023) [Do All Languages Cost the Same? Tokenization in the Era of Commercial Language Models](https://arxiv.org/abs/2305.13707), EMNLP 2023.

[^21]: (Hofmann et al., 2024) [Dialect prejudice predicts AI decisions about people's character, employability, and criminality](https://arxiv.org/abs/2403.00742), Nature 633:147–154.

[^22]: (Wilson & Caliskan, 2024) [Gender, Race, and Intersectional Bias in Resume Screening via Language Model Retrieval](https://arxiv.org/abs/2407.20371). A résumé-screening model favored White-associated names ~85% of the time and never ranked a Black-male name above an equivalent White-male one.

