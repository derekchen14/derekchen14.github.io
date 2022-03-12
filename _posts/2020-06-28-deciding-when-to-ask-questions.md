---
layout: post
title: Deciding when to Ask Questions
date: '2020-06-28 18:15:06'
---

Performing active learning on data annotation is to decide when the model should query the expert annotator for more samples. &nbsp;Note the parallel with dialogue, which is to decide when the agent should ask a clarification question to the customer for more details on their intent. &nbsp;Such a policy can be obtained through static data with basic supervised learning or in a more interactive manner through imitation learning using algorithms such as DAgger ([Ross 2011](https://arxiv.org/abs/1011.0686)).

We have seen in numerous papers that simply using the direct system output (ie. softmax) to measure the uncertainty of the model is highly uncalibrated ([Guo et al. 2017](https://arxiv.org/abs/1706.04599), [Jiang et al. 2018](https://arxiv.org/abs/1805.11783)). &nbsp;There have been various ways to deal with this issue. &nbsp;For example, you can tune the confidence score with a temperature variable ([Guo et al. 2017](https://arxiv.org/abs/1706.04599)), set a threshold to make a decision or use dropout as a bayesian approximation ([Gal and Ghahramani 2016](https://arxiv.org/abs/1506.02142)). &nbsp;However, none of these work that well ([Feng et al. 2020](https://arxiv.org/abs/2004.01926)) in comparison to having a forecaster whose sole job is to decide whether or not to query the expert ([Kuleshov and Liang, 2015](https://papers.nips.cc/paper/5658-calibrated-structured-prediction)). &nbsp;This is intuitive because the forecaster has extra parameters to work with that allow it to learn something useful.

However, building a good forecaster is non-trivial. &nbsp;Concretely, it is unclear when a forecaster should decide to ask a question. &nbsp;Since the forecaster is a model, it must be trained. &nbsp;But if it needs to be trained, then you must have labels. &nbsp;And if you have labels, this implies you know when a model is uncertain, and the whole point of having a forecaster is that you cannot identify this situation through the logits alone.

Brantley et al. ([2020](https://arxiv.org/abs/2005.12801)) use Apple Tasting and a difference classifier paired with a heuristic model to get around these issue. &nbsp;Going in reverse, the _heuristic model_ is a rule-based or generally simpler system than the main model that can offer predictions of the user intent, but may often be wrong. &nbsp;Then the _difference classifier_ has the job to decide if the heuristic model behaves differently from the main model. &nbsp;In this sense, the heuristic model acts similar to a generator in the GAN setting ([Goodfellow et al. 2017](https://arxiv.org/abs/1406.2661)), and the difference classifier acts similar to a discriminator. &nbsp;Lastly, _Apple Tasting_ ([Helmbold et al., 2000](http://phillong.info/publications/apple.pdf)) is a framework for helping train the forecaster. &nbsp;The idea is that it can be hard to build a classifier to decide which apples are tasty since you have to eat an apple to determine its value, which means you will inevitably try some bad apples. &nbsp;This is identical to the problem when building a forecaster in real-life since the only way to know it made a bad prediction (asked a question to the customer when it should not have) is to allow it to make mistakes. Thus, various algorithms (such as STAP) are employed which essentially run this experiment multiple times as part of an inner loop with the data that the models have already seen so far. &nbsp; Then, if the difference classifier does not perform well during this process, it should not be trusted as much and therefore it is acceptable to query the user. &nbsp;As training progresses, the difference classifier will improve, and thus queries to the user will also decrease.

To me, the first insight is to use all available information -- which is to say, use both the uncertainty scores from the main model _and_ a forecaster policy. &nbsp;Specifically, if the main model is very certain, then avoid asking the customer any questions. &nbsp;If the model is at all uncertain, even though only query the customer if the difference classifier thinks the gap between the heuristic and the main model is large. &nbsp;The Apple Tasting method helps to train this, but most importantly, the forecaster is now _trained on the distribution it will see in real life_! &nbsp;In other words, be decoupling the confidence score and the query decision, the forecaster only operates on examples where the system has deemed the situation to be uncertain.

The difference classifier (ie. forecaster) combined with a traditional RL policy is then similar to the policy used in conversational machine reading ([Saeidi et al. 2018](https://arxiv.org/abs/1809.01494)) that decides whether to Inquired (ask a follow-up question) or Respond (with a Yes/No answer). &nbsp;As a final step, all of these variables can be passed to a natural langauge generator which can decide on the final response.
