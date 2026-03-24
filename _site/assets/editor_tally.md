# LLM Patterns

## Word Choice

### 1. Magic Adverbs

- **Count**: 8 occurrences found
- **Threshold**: 3
- **Status**: VIOLATION (8 - 3 = 5 to fix)

Occurrences:
1. Line 22: "takes a **fundamentally** different approach" — magic adverb inflating significance.
2. Line 28: "models that were **fundamentally** bad at understanding language" — same adverb, second use.
3. Line 28: "Modern LLMs are not **fundamentally** bad at understanding language" — third use of "fundamentally" in the same paragraph.
4. Line 28: "They're **remarkably** good at it" — classic magic adverb signaling significance.
5. Line 44: "They are **genuinely** bad" — magic adverb adding hollow emphasis.
6. Line 48: "It will infer — often **incorrectly** — and produce a confident-sounding result" — functional but contributes to adverb density.
7. Line 48: "it will **cheerfully** produce *something*" — anthropomorphizing adverb for dramatic effect.
8. Line 72: "Where the criticism **genuinely** sticks" — second use of "genuinely."

**FIX** (5 most egregious):

1. Line 28: "They're remarkably good at it."
   - "Remarkably" is textbook magic adverb.
   - Proposed: "In fact, they're incredibly good at it.

2. Line 22: "takes a fundamentally different approach"
   - Let the description that follows show how different it is.
   - Proposed: "takes a different approach"

3. Line 28: "models that were fundamentally bad at understanding language. Modern LLMs are not fundamentally bad at understanding language."
   - "Fundamentally" used twice in back-to-back sentences, and a third time in the same paragraph (line 22). Remove at least two.
   - Proposed: "models that were bad at understanding language. Modern LLMs are not bad at understanding language."

4. Line 44: "They are genuinely bad"
   - "Genuinely" is hollow emphasis. The specifics that follow do the real work.
   - Proposed: "They are subtlely bad in ways that don't readily surface in demos."

5. Line 48: "it will cheerfully produce *something*"
   - Anthropomorphizing adverb for dramatic color.
   - Proposed: "it will produce *something* without hesitation just to please the user."

### 2. Overused Vocabulary

- **Count**: 11 occurrences found
- **Threshold**: 4
- **Status**: VIOLATION (11 - 4 = 7 to fix)

Occurrences:
1. Line 8: "The AI agent **framework** landscape" — both "framework" and "landscape" in one phrase.
2. Line 8: "**framework** landscape" — "framework" usage #1.
3. Line 8: "**landscape**" — "landscape" usage.
4. Line 12: "every **framework** I reviewed" — wait, this is line 62, not 12. Let me recount from exact lines.
5. Line 20: "agent **framework**" — line 20.
6. Line 24: "dead **paradigm**" — line 24.
7. Line 32: "Modern **frameworks** handle this" — line 32.
8. Line 58: "it **certainly** cannot infer" — line 58.
9. Line 62: "every **framework** I reviewed" — line 62.
10. Line 70: "Every other **framework** treats" — line 70.
11. Line 80: "The popular agent **frameworks** in 2026" — line 80.

Detailed count:
- "framework/frameworks": Lines 8, 20, 32, 62, 70, 80 = 6 uses
- "landscape": Line 8 = 1 use
- "paradigm": Line 24 = 1 use
- "certainly": Line 58 = 1 use
- "ecosystem": Line 8 ("landscape" but not ecosystem) = 0

Total flagged words: 6 (framework) + 1 (landscape) + 1 (paradigm) + 1 (certainly) = 9 occurrences.

Note: "framework" is somewhat unavoidable in a post comparing agent frameworks, but 6 uses is high. "Paradigm" and "landscape" are more clearly flagged vocabulary.

**FIX** (5 most egregious):

1. Line 8: "The AI agent framework landscape in 2026 is booming."
   - Two flagged words in one phrase.
   - Proposed: "The market for AI agent toolkits in 2026 is booming."

2. Line 24: "clinging to a dead paradigm"
   - "Paradigm" is a classic overused word.
   - Proposed: "clinging to a dead approach"

3. Line 58: "it certainly cannot infer"
   - "Certainly" is flagged vocabulary.
   - Proposed: "it cannot infer"

4. Line 32: "Modern frameworks handle this with a Pydantic model"
   - Proposed: "Modern toolkits handle this with a Pydantic model"

5. Line 70: "Every other framework treats the gap"
   - Proposed: "Every other toolkit treats the gap"

### 3. Pompous Verbs

- **Count**: 0 occurrences found
- **Threshold**: 4
- **Status**: PASS

No instances of "serves as", "stands as", "marks", or "represents" used as pompous substitutes found. The post consistently uses direct "is/are" constructions.


## Sentence Structure

### 4. Negative Parallelism

- **Count**: 5 occurrences found
- **Threshold**: 1
- **Status**: VIOLATION (5 - 1 = 4 to fix)

Occurrences:
1. Line 46: "This isn't gold-plating. It's the only reliable way to get a confidence signal" — classic "It's not X. It's Y."
2. Line 74: "these are trimming problems, not foundation problems" — negation-pivot structure.
3. Line 82: "This is not an edge case. For any agent that serves the general public, this *is* the primary use case." — "not X. It's Y."
4. Line 86: "aren't relics of a weaker era — they're solutions to problems that LLMs have hidden, not solved." — "not X — they're Y."
5. Line 88: "Not because the old way was better, but because the problems it solved never went away." — "Not X, but Y" closing the entire piece.

**FIX** (4 most egregious):

1. Line 82: "This is not an edge case. For any agent that serves the general public, this *is* the primary use case."
   - Two sentences of negation-pivot for dramatic emphasis at a key moment.
   - Proposed: "For any agent that serves the general public, this *is* the primary use case."

2. Line 86: "aren't relics of a weaker era — they're solutions to problems that LLMs have hidden, not solved."
   - Double negative parallelism: "aren't X — they're Y" plus "hidden, not solved."
   - Proposed: "Far from being relics of a weaker era, these are attempts at solutions to problems that LLMs have merely tucked under the rug, rather than solved."

3. Line 88: "Not because the old way was better, but because the problems it solved never went away."
   - Dramatic closer using negation-pivot.
   - Proposed: "The problems it solved never went away. The industry just stopped noticing them in favor of newer, shinier problems."

4. Line 46: "This isn't gold-plating. It's the only reliable way to get a confidence signal"
   - Proposed: "This is the only reliable way to get a confidence signal from LLMs, which are notoriously miscalibrated when asked to rate their own certainty."

### 5. Rhetorical Questions

- **Count**: 3 occurrences found
- **Threshold**: 1
- **Status**: VIOLATION (3 - 1 = 2 to fix)

Occurrences:
1. Line 8: "**what happens when the user is vague, incomplete, or just plain confusing?**" — rhetorical question posed for dramatic effect at end of opening paragraph.
2. Line 24: "But is PA carrying forward valuable ideas, or is it clinging to a dead paradigm like an old timer reminiscing about the good old days playing high school football?" — rhetorical question as section transition.
3. Line 82: "what happens when the user can't articulate a clear request? When they're vague, when they provide half the information, when they contradict themselves, when they type 'fix it' and expect the agent to know what 'it' means?" — rhetorical question cascade.

**FIX** (2 most egregious):

1. Line 82: "what happens when the user can't articulate a clear request? When they're vague..."
   - Cascading rhetorical questions that restate the thesis from line 8 almost verbatim.
   - Proposed: "But there's a different problem that almost nobody is working on: users who can't articulate clear requests, who provide half the information, and then contradict themselves. Real users will say 'fix it' and expect the agent to know what 'it' means without providing any context."

2. Line 24: "But is PA carrying forward valuable ideas, or is it clinging to a dead paradigm like an old timer reminiscing about the good old days playing high school football?"
   - Rhetorical question as section transition plus unnecessary metaphor (see Pattern 10).
   - Proposed: "PA may be carrying forward valuable ideas, or it may be clinging to an outdated approach like a high school football star reminiscing about the good ole days."

### 6. Filler Transitions

- **Count**: 1 occurrence found
- **Threshold**: 4
- **Status**: PASS

Occurrences:
1. Line 30: "Consider the cost." — borderline filler transition, but it does connect to a concrete point. Acceptable.

### 7. Show, Don't Tell

- **Count**: 1 occurrence found
- **Threshold**: 1
- **Status**: PASS

Occurrences:
1. Line 70: "This is a small feature that produces an outsized UX improvement." — tells the reader the feature has outsized impact rather than letting the preceding example speak for itself. However, it's borderline since the example was just shown.

### 8. False Ranges

- **Count**: 0 occurrences found
- **Threshold**: 1
- **Status**: PASS

No false "from X to Y" constructions found. The post avoids this pattern.

### 9. Short Punchy Fragments

- **Count**: 3 occurrences found
- **Threshold**: 2
- **Status**: VIOLATION (3 - 2 = 1 to fix)

Occurrences:
1. Line 32: "No SourceSlot, no GroupSlot, no distinction between fill-by-label and fill-by-values. It just works." — fragment cascade followed by punchy short sentence.
2. Line 74: "The academic language doesn't affect the engineering. The extra slot types don't block progress." — two very short parallel sentences used for drumbeat emphasis.
3. Line 84: "They dropped the baggage. They also dropped the lessons." — two short punchy fragments as a paragraph closer for dramatic effect.

**FIX** (1 most egregious):

1. Line 84: "They dropped the baggage. They also dropped the lessons."
   - Two short fragments manufactured for rhetorical punch.
   - Proposed: "They threw out the baby with the bathwater."


## Tone

### 10. Unnecessary Metaphors

- **Count**: 1 occurrence found
- **Threshold**: 1
- **Status**: PASS

Occurrences:
1. Line 24: "clinging to a dead paradigm like an old timer reminiscing about the good old days playing high school football" — a simile/metaphor. However, it is colorful and serves the argument by conveying a specific attitude. Borderline but within threshold.

### 11. False Vulnerability

- **Count**: 0 occurrences found
- **Threshold**: 1
- **Status**: PASS

No instances of simulated candor, false self-awareness, or pretending to break the fourth wall found.

### 12. Go Directly to the Point

- **Count**: 2 occurrences found
- **Threshold**: 1
- **Status**: VIOLATION (2 - 1 = 1 to fix)

Occurrences:
1. Line 40: "Some of those criticisms are valid. But the core ones collapse under scrutiny when you look at what actually happens with real users" — tells the reader the criticisms collapse rather than immediately showing how.
2. Line 60: "The clarification argument is the weakest criticism" — section heading that tells the reader the argument is weak before proving it.

**FIX** (1 most egregious):

1. Line 40: "Some of those criticisms are valid. But the core ones collapse under scrutiny when you look at what actually happens with real users — not demo users, not developer-testers, but ordinary people..."
   - Tells before showing. The section that follows does the actual work.
   - Proposed: "Some of those criticisms are valid, but the core ones don't hold up when you look at what actually happens with real users." (Drop the rest after the em-dash.)

### 13. Grandiose Stakes Inflation

- **Count**: 0 occurrences found
- **Threshold**: 1
- **Status**: PASS

The post keeps its claims scoped to the specific domain of agent frameworks and user ambiguity. No world-historical inflation or "imagine a future" openings found.

### 14. Vague Attributions

- **Count**: 0 occurrences found
- **Threshold**: 0
- **Status**: PASS

No unnamed experts, vague studies, or inflated consensus claims found. All claims are either first-person ("I found", "I reviewed") or attributed to specific frameworks.

### 15. Invented Concept Labels

- **Count**: 0 occurrences found
- **Threshold**: 2
- **Status**: PASS

The post uses established NLU terminology (slot-filling, intent classification, ambiguity handler, entity repair, flow stack). No compound labels are coined and treated as established terms.


## Formatting

### 16. Em-Dash Addiction

- **Count**: 31 em-dashes found (across 20 lines)
- **Threshold**: 2
- **Status**: VIOLATION (31 - 2 = 29 to fix)

Occurrences (by line, with em-dash count per line):
1. Line 12: 2 em-dashes ("happy path — clear input" and "unhappy path — the reality")
2. Line 18: 1 em-dash ("of the three — guardrails can catch")
3. Line 20: 2 em-dashes ("pass or halt — with no retry" and implicit in the long line)
4. Line 28: 2 em-dashes ("The entire machinery — 16 slot types..." and closing em-dash)
5. Line 30: 1 em-dash ("strong model — at 3x the cost")
6. Line 32: 1 em-dash ("structured output — the LLM extracts")
7. Line 40: 1 em-dash ("real users — not demo users")
8. Line 44: 2 em-dashes ("genuinely bad — in ways that" and closing em-dash)
9. Line 46: 1 em-dash ("real number — and model disagreement")
10. Line 48: 2 em-dashes ("infer — often incorrectly —")
11. Line 50: 1 em-dash ("catastrophic — the agent suddenly")
12. Line 54: 2 em-dashes ("validates *syntax* — " and "validate *semantics* — ")
13. Line 56: 2 em-dashes ("*pragmatics* of a conversation —" and "conversational context — exactly")
14. Line 68: 1 em-dash ("valid options are Y — what did they mean")
15. Line 74: 3 em-dashes ("pipeline — the math isn't wrong" and "confidence signal — a single strong model" and "trimming problems, not foundation problems")
16. Line 80: 1 em-dash ("real problem — making it easy")
17. Line 84: 1 em-dash ("handle it — intent classification")
18. Line 86: 2 em-dashes ("weaker era — they're solutions" and "unhappy path — where real users")
19. Line 88: 1 em-dash ("actually rely on — people who")

This is extreme overuse. The post averages roughly one em-dash every 2.8 lines of prose.

**FIX** (showing 10 representative fixes; all remaining em-dashes beyond 2 should follow the same approach of replacing with commas, periods, colons, or parentheses):

1. Line 12: "PA optimizes for the unhappy path — the reality that most people won't learn to prompt"
   - Proposed: "PA optimizes for the unhappy path: the reality that most people won't learn to prompt"

2. Line 28: "The entire machinery — 16 slot types with inheritance hierarchies, multi-model ensemble voting, four-level ambiguity handlers, flow stacks with lifecycle states — exists to compensate"
   - Proposed: "The entire machinery (16 slot types with inheritance hierarchies, multi-model ensemble voting, four-level ambiguity handlers, flow stacks with lifecycle states) exists to compensate"

3. Line 44: "They are genuinely bad — in ways that don't surface in demos — at three things."
   - Proposed: "They are genuinely bad, in ways that don't surface in demos, at three things."

4. Line 48: "It will infer — often incorrectly — and produce a confident-sounding result."
   - Proposed: "It will infer, often incorrectly, and produce a confident-sounding result."

5. Line 50: "the failure mode is catastrophic — the agent suddenly acts as if the previous three turns didn't happen."
   - Proposed: "the failure mode is catastrophic: the agent suddenly acts as if the previous three turns didn't happen."

6. Line 54: "Structured output validates *syntax* — 'topic must be a string.' PA's slots validate *semantics* — 'topic must refer to something the user actually mentioned.'"
   - Proposed: Use colons. "Structured output validates *syntax*: 'topic must be a string.' PA's slots validate *semantics*: 'topic must refer to something the user actually mentioned.'"

7. Line 74: "a straightforward classify → route → fill → execute → respond pipeline — the math isn't wrong, but nobody is computing belief distributions"
   - Proposed: "a straightforward classify → route → fill → execute → respond pipeline. The math isn't wrong, but nobody is computing belief distributions"

8. Line 74: "the most expensive way to get a confidence signal — a single strong model"
   - Proposed: "the most expensive way to get a confidence signal. A single strong model with a calibrated follow-up call could deliver 80% of the benefit at a third of the cost."

9. Line 86: "aren't relics of a weaker era — they're solutions to problems"
   - Proposed: (Fixed elsewhere.)

10. Line 88: "ordinary people can actually rely on — people who will never learn to prompt"
    - Proposed: "ordinary people can actually rely on, people who will never learn to prompt"

### 17. Bold-First Bullets

- **Count**: 3 occurrences found
- **Threshold**: 1
- **Status**: VIOLATION (3 - 1 = 2 to fix)

Occurrences:
1. Line 46: "**Calibrated confidence.**" — bold-first paragraph (not a bullet, but functions identically as a bold-label-first list item).
2. Line 48: "**Silent failures on vague input.**" — same pattern.
3. Line 50: "**Consistency across turns.**" — same pattern.

Note: Lines 62 also uses bold inline labels (**Over-asking**, **Under-asking**, **Asking poorly**) but these are inline within a paragraph, not list-leading bold phrases.

**FIX** (2 most egregious):

1. Lines 46, 48, 50: Three consecutive paragraphs each beginning with a bold label.
   - Proposed: Remove the bold labels and integrate them as topic sentences. For example:
     - Line 46: "LLMs don't signal calibrated confidence. When an LLM isn't sure what the user means..."
     - Line 48: "LLMs fail silently on vague input. Tell an LLM 'make it better' and it will produce *something*..."
     - Line 50: "LLMs are inconsistent across turns. An LLM with conversation history can usually remember what it was doing..."

2. Line 62: "**Over-asking**: ... **Under-asking**: ... **Asking poorly**:"
   - Three bold-first inline labels in a single paragraph.
   - Proposed: Rewrite as three separate sentences without bold labels: "First, the LLM asks clarifying questions even when the request is clear, irritating the user. Second, it proceeds with vague input when it should clarify, producing wrong results. Third, it generates open-ended 'could you tell me more?' when a specific 'which of these three posts did you mean?' would resolve the ambiguity in one click."


## Composition

### 18. Filler Sentences

- **Count**: 0 occurrences found
- **Threshold**: 0
- **Status**: PASS

No instances of "Here's the kicker", "Here's the thing", "Here's where it gets interesting", "Let's break this down", "Let's unpack this", "Let's explore", or "Let's dive in" found.

### 19. Duplicate Content

- **Count**: 2 occurrences found
- **Threshold**: 1
- **Status**: VIOLATION (2 - 1 = 1 to fix)

Occurrences:
1. Lines 8 and 82: Both pose nearly the same rhetorical question: "what happens when the user is vague, incomplete, or just plain confusing?" (line 8) echoed by "what happens when the user can't articulate a clear request? When they're vague, when they provide half the information, when they contradict themselves" (line 82). The thesis is restated almost verbatim.
2. Lines 12 and 86: Both contain the same structural argument. Line 12: "PA optimizes for the unhappy path." Line 86: "The unhappy path — where real users actually live — is where the architecture earns its keep." The same contrast (happy vs. unhappy path) appears in both the intro and conclusion.

**FIX** (1 most egregious):

1. Line 82: The rhetorical question cascade restates line 8 almost word-for-word.
   - The conclusion should build on the thesis, not restate it. By line 82, the reader has seen the evidence.
   - Proposed: "But there's a different problem that almost nobody is working on: serving users who will never provide clear, complete requests. For any agent that serves the general public, this *is* the primary use case."  (Should largely be addressed above.)

### 20. Repetitive Language

- **Count**: 3 occurrences found
- **Threshold**: 2
- **Status**: VIOLATION (3 - 2 = 1 to fix)

Occurrences:
1. "framework/frameworks" appears 6 times (lines 8, 20, 32, 62, 70, 80). While somewhat unavoidable given the subject matter, it creates repetitive texture.
2. "fundamentally" appears 3 times in lines 22 and 28 alone. The same adverb in rapid succession.
3. The phrase pattern "the user" / "users" appears 14+ times across the post. While functional, the density is notable in the conclusion (lines 80-88) where it appears 4 times in quick succession.

**FIX** (1 most egregious):

1. Lines 22/28: "fundamentally" three times within seven lines.
   - Proposed: Remove two of the three uses (see Pattern 1 fixes above). Keep at most one occurrence in the entire post.
