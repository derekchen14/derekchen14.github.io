---
layout: post
title: 2020 Year End Review (Part 2)
tags: [conference, trends]
excerpt_separator: <!--more-->
---

Continuing on the thoughts described in [Part 1](https://morethanoneturn.com/2020/12/28/emnlp-2020-highlights.html), we now describe two trends around dialogue and NLP research. In the previous post, we discussed how data strategy is playing an increasingly important role in the development of modern machine learning models.  How that plays out in more detail will be discussed in the future, but today we will look further into the development of dialogue in the past few months as well as other observations.

<!--more-->

As a quick disclaimer, the sections titles are meant to be provocative, so any percieved slights are misguided since all papers mentioned represent significant progress in their own right.  Accordingly, no comments should be construed as minimizing any of the authors' great work in any manner.

## (3) Stagnation of Dialogue

Like all NLP conferences in the past few years, EMNLP 2020 contained its fair share of papers around Dialogue State Tracking. [Schema Fusion Networks](https://arxiv.org/abs/2004.03386) (Zhu, Li, Chen and Yu) aimed to improve DST by encoding past predictions into a schema graph and carrying over past knowledge into further predictions.  Some other ideas include [Actor-Double-Critic](https://www.aclweb.org/anthology/2020.findings-emnlp.75/) (Wu, Tseng, and Gasic), [GCDST](https://www.aclweb.org/anthology/2020.findings-emnlp.95/) (Wu, Zou, Jiang, and Aw), [Temporally Expressive Networks](https://arxiv.org/abs/2009.07615) (Chen, Zhang, Mao, Xu) and [Slot Attention with Value Normalization](aclweb.org/anthology/2020.emnlp-main.243/) (Wang, Guo, Zhu).  What all these papers have in common is the development of complex models to boost the final accuracy score.

Thus, arguably the most impressive was DST paper was [SimpleTOD](https://arxiv.org/abs/2005.00796) (Hosseini-Asl et al.) which used the straightforward, but powerful idea of taking GPT-2 as the core model to sequentially generate the output dialogue states.  Properly designing the prompts to feed into the model seems trivial, but is actually quite insightful once you realize the exponential number of combinations possible to design a model input.  Furthermore, this simple setup yielded the best results we have seen on MultiWOZ[^1] so far with 55.76 joint goal accuracy.

While these methods are all impressive, it is starting to seem like the current MultiWOZ benchmark is starting to hit its limits since qualitative analysis will reveal that most model errors are actually due to errors in annotation.  As model performance starts to saturate, the natural progression is to move onto further benchmarks.[^2][^3][^4]  Thus, the short-comings of the original dataset have spurred on the creation of MultiWOZ 2.1[^5], MultiWOZ 2.2[^6] and MultiWOZ 2.3[^7].

These revisions fix a number of labeling issues, but there is also an underlying issue where it seems the community might be overfitting to a single dataset, which means only making incremental gains. To this extent, dialogue research has perhaps stagnated since we are so highly focused on the single task of multi-domain slot-filling.  Thus, it is perhaps time to have a new standard benchmark which retains the core components of semantic understanding, but also offers other aspects of conversation to study.  As some possible examples, we could explore task-oriented dialogues where the user is unsure of the options available to them, so the job of the agent is to elicit their preferences before recording them.  Another avenue is to look at dialogues where the agent is constrained in their actions by company policies, which reflect real-world scenarios.

To be clear, other elements of dialogue, such as natural language generation are still seeing exciting movement, and this section will be wrapped up by highlighting three papers in this realm.  Building on the theme from the previous section, [Paraphrase Augmented Task-Oriented Dialog Generation](https://arxiv.org/abs/2004.07462) (Gao, Zhang, Ou, Yu) uses Data Augmentation techniques rather than model changes to improve the performance of Dialgoue State Tracking.  In particular, the authors devise a method for augmenting data through paraphrasing with templates.  They do so by taking advantage of the structure of the dialogue labels in the training set as opposed to just paraphrasing the raw text.

Within [Make Neural Natural Language Generation as Reliable as Templates](https://www.aclweb.org/anthology/2020.emnlp-main.230/) Elder, O'Connor, and Foster also use generation as a method for data augmentation.  Their idea is to find the balance between template systems (which may be too rigid) and neural systems (which are often uncontrollable) that allows for diversity while also maintaining reliability.  The key idea is to use regex and heuristics to identify desired surface forms that we would want a model to generate.  Then, we restrict the softmax output and generation capabilities (ie. beam search) to only be able to produce these surface forms and other minor function words.

If we were to push the augmentation idea to the extreme, we would be training entire dialogue systems using generated data, which is precisely what a user simulator would provide.  In fact, [Dialog Simulation with Realistic Variations](https://arxiv.org/abs/2011.08243) from the Alexa Conversations at Amazon have done exactly that.  Although still not quite a simulator with realistic actions, the authors do add some interesting variations.  Starting with a M2M paraphrasing technique[^8], the group puts in extra effort to generate novel template flows.  Specifically, they allow for users to adjust their preferences during the chat and have some element of pushing the conversation along by pro-actively requesting the slot-values of API calls that have not yet been made.

## (4) Convergence of Ideas

The more optimistic view of this point is that "great people think alike".  The more pessimistic view of this point is that the lack of interesting datasets means there are only a few fertile areas left to look.  Perhaps a more realistic view is that similar ideas commonly arise from independent sources when many people are exploring the same frontier at once, and this is simply the first time I've noticed.  The bottom line is that I started to notice a number of duplicate papers stating basically the same core idea, both published at the same time at EMNLP.

The first example of this is maybe the most obvious given the similarity of their titles. [Conversational Semantic Parsing for DST](https://arxiv.org/abs/2010.12770) from Apple and [Conversational Semantic Parsing](https://www.aclweb.org/anthology/2020.emnlp-main.408/) from Facebook both examine their specific style of parsing dialogues.  I imagine that Amazon, Tencent and Google all have something similar as well.  For reference, Microsoft's version of this is to view [Task-Oriented Dialogue as Dataflow Synthesis](https://arxiv.org/abs/2009.11423) (Semantic Machines et al.).

More specifically, Apple's version presents dialogues as Trees with segments that are chained together with periods.  They adopt a stack to represent multiple tasks in dialog history with pointers from certain branches to others.  Facebook's version is quite similar in that it also contains a hierarchical structure where certain slot values are expanded into further detail.  However, as someone who has designed a dialogue meaning representation for production purposes as well, I simply cannot fathom how these representations can be quickly annotated by crowdsource workers.  The structure of these labels make many of them hard to even verify, much less annotate, with any reliable level of accuracy.  There are many very smart people who work there though, so it's possible they are privy to some insight that I am not.

The second set of similar papers share the high level goal of generating novel training data for semantic parsing in an semi-automated fashion.  The shared core idea is to generate the training data by conditioning query-based templates on the known database schema.  Then, the templates are filled and transformed into natural langauge through the use of deep learning models.  To avoid using augmented data that changes the semantics, they also apply the same pipeline for verification.  Namely, they start by parsing the natural language output back into the logical form.  Then, this SQL query is executed against the engine to return a result.  Finally, results failing to match the original, dynamically-selected KB attribute are filtered out -- leaving only high quality generations for training.

[AutoQA](https://arxiv.org/abs/2010.04806) (Xu, Semnani, Campagna, Lam) uses BART for paraphrasing the templates into natural language, whereas [GAZP](https://arxiv.org/abs/2009.07396) (Zhong, Lewis, Wang, and Zettlemoyer) uses BERT with BiLSTM components.  If we allow for changing the task to NER, then [Counterfactual Generator](https://www.aclweb.org/anthology/2020.emnlp-main.590/) (Zeng, Li, Zhai, Zhang) can also be added to this set, since it also follows the same format of generating imperfect examples and then using the originally trained model to filter for the best data augmentations.  In all three cases, the training set that includes the augmentations achieves the best final performance.

The final set of similar papers touches on the topic of improving model performance through the use of soft perturbations to generate new data samples.  More specifically, [Self-Supervised Manifold Based Data Augmentation](https://arxiv.org/abs/2009.10195) (Ng, Cho, and Ghassemi) perform data augmentation by masking out various tokens and then having BERT fill those tokens in, keeping the original label.  This may be error prone since you could drop key words that change the label, but for the tasks they chose, this was not a substantial concern.  In particular, they experiment with Sentiment Analysis, Natural Language Inference and Machine Translation.  During their experiments, they found that, using a BERT model to predict hard (aka. one-hot) labels led to roughly equal results.  Intuitively, this should be expected since it is simply feeding in more examples of things the model already knows.  The insight then was feeding in soft labels and training with KL-divergence, which then led to improved Out-of-domain performance.

To see why this helps, we turn to [Towards More Accurate Uncertainty Estimation](https://www.aclweb.org/anthology/2020.emnlp-main.671/) (He et al.) which employs a very similar insight.  In particular, the authors create perturbed inputs and labels using MixUp and also change their loss function to KL-divergence.  This creates labels are now also “soft” in that the new label is a distribution between the two mixed classes rather than discrete.  Similar to the original MixUp paper then[^9], the idea is that using the soft labels prevents the model from being too overly confident since during training it has seen outputs where the target was not necessarily completely towards a single label, but rather shared across multiple labels.  Overall, the idea that soft targets may be more robust than hard targets is something worth further exploration!

---

In total, there are arguably fewer mind-blowing model changes or huge performance gains within NLP as there might have been in years past.  In it's place, we observe important and more nuanced advancements that highlight the maturity of the field.  I would argue that the shift towards more pragmatic data management concerns is actually good since it implies a move towards bringing these great NLP advancements into the real world.

[^1]: [MultiWOZ](https://arxiv.org/abs/1810.00278) (Budzianowski, et al.)
[^2]: [GLUE](https://arxiv.org/abs/1804.07461) (Wang et al.)
[^3]: [SQuAD](https://arxiv.org/abs/1606.05250) (Rajpurkar et al.) 
[^4]: [bAbI](https://arxiv.org/abs/1502.05698) (Weston at al.)
[^5]: [MultiWOZ 2.1](https://arxiv.org/abs/1907.01669) (Eric et al.)
[^6]: [MultiWOZ 2.2](https://arxiv.org/abs/2007.12720) (Zang et al.) 
[^7]: [MultiWOZ 2.3](https://arxiv.org/abs/2010.05594) (Han and Liu et al.)
[^8]: [Building a Conversational Agent Overnight](https://arxiv.org/abs/1801.04871) (Shah et al.)
[^9]: [mixup: Beyond Empircal Risk Minimzation](https://arxiv.org/abs/1710.09412) (Zhang et al.)