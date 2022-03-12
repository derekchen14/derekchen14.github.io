---
layout: post
title: 100 Research Papers in 100 Days (LM Edition)
date: '2017-03-29 22:05:21'
tags: [nlp, research, lists, trends]
---

This is Part III of 100 in 100 series:

 - Language Modeling: ~30 papers
 - NLP Edition: ~40 papers
 - Images Edition: ~30 papers

### Language Modeling (LM, NMT, Seq2Seq)
Gains made where a model generates responses or predicts responses.  Decoder can produce sentences or characters or belief states.  Improvements in language modeling where the gain comes from novel ideas or architectures. Also, anything involving chatbots.

[**Exploring the Limits of Language Modeling**](http://arxiv.org/abs/1602.02410) (*Jozefowicz, Vinyals, Schuster, Shazeer, Wu*) [Read Sept 16th]
 
 - **Idea**: Use Importance Sampling with 8192 samples drawn per step, and were shared across all the target words in the batch (2560 total, i.e. 128 times 20 unrolled steps). This results in multiplying (2560 x 1024) times (1024 x (8192+1)) (instead of (2560 x 1024) times (1024 x 793471)), which is about 100-fold less computation, where 793471 is the original number of unique words in the dataset. 
 - Using a Character CNN embedding with more features (4096) over a traditional (one-hot?) word embedding also helped with the parameter reduction(?), the number of parameters of the input layer is reduced by a factor of 11, though training speed is slightly worse.
 - **Results**: Using this specialized Softmax loss, they are able to gain a tool that is efficient to train and nearly as precise as a full Softmax, but with much fewer parameters.  This yielded significant improvements on a well known, large scale LM task: from 51.3 down to 30.0 perplexity for single models while reducing the number of parameters from by 20x
 - **Lessons**: Bigger networks matter. The best model was also the largest, a 2-layer LSTM with 8192+1024 dimensional recurrent state in each of the layers.  Increasing the embedding and/or the projection also helped, but not worth the increase in parameters.  Also, ensemble of different models work better than one model.  Perplexity is reduced to 23.7, a large improvement compared to current state-of-art, and near the theoretical limit proposed by Shannon et al.

[**Data-Driven Response Generation in Social Media**](https://www.microsoft.com/en-us/research/publication/data-driven-response-generation-in-social-media/) *(Ritter, Cherry, Dolan)* [Read: Mar 24th]

 - First real attempt at performing data driven response generation, as opposed to information retrieval of responses.  This means the responses is *not* based on hard-coded rules or templates, and is therefore more amenable to generalization of open domain dialogue. Establishes idea of SMT (statistical machine translation) in contrast to the NMT (neural machine translation) commonly seen today.
 - **Idea**:Treat the system like machine translation, but recognize some shortcomings. First, unlike bilingual text, question-response  pairs don't have to mean the same thing.  In fact, they shouldn't be, in order to allow for a meaningful conversation.  Second, not only are the meanings different, but lengths of responses can also be different, and/or not contain any overlap of words with the question and still be perfectly valid.  This is in direct contrast to traditional machine translation where the goal is precisely overlap of words and meaning.  They get around these issues by (a) forcing lexical similarity to be different by penalizing according to Jaccard distance and (b) using Fisher’s Exact Test to filter out pairs with low correlation.
 - **Model**: The response is estimated from counts of phrase pairs observed in the training text, and the language model is built using n-gram statistics from a large set of observed responses.  They use beam search to get a more coherent response.  Problem with all this is the model is never able to generate novel phrases that it has never seen before.  Probably also breaks down if encoder input contains phrases the model has not seen before.

[**A Neural Network Approach to Context-Sensitive Generation of Conversational Responses**](http://arxiv.org/abs/1506.06714) *( Sordoni, Galley, Auli, Brockett, Ji, et. al.)* [Read: Mar 25th]

  - One of the first papers around chatbots, since it was published in Feb 2015, and submitted for review even earlier.  Basic premise is to use a dense word embeddings rather than one-hot or phrase counts.  Significant leap up from SMT model, since this now uses a vanila RNN with encoder and decoder.  Dataset is unstructured Twitter conversations.
  - Introduces idea of Context-sensitive Machine Translation.  Rather than just a message and response, there is also a third variable, the context, which represents the previous responses in the conversation. These days, the context is often output from the attention mechanism, which really is slightly different since that is a look-back on the message, rather than previous sentences uttered before the message.  Context in this paper can be:
   - RLMT: just the context fed in as earlier input into the encoder of the RNN
   - DCGM-I: the context mixed with the message as a bag-of-words and then transformed through a feed forward network before being passed to the decoder.  Note that this context vector is static once it is created.
   - DCGM-II: the context is mixed with the message while preserving order (by concat) and then transformed through a feed forward network before being passed to the decoder. 
  - Another major feature the authors came up with is the n-gram overlap of the context and the message, denoted CMM.  Concretely, this is a set of 8 features: the first four are unigram, bigram, trigram, and 4-gram overlap between the context (c) and the response (r).  The remaining four features are the [1-4]-gram overlap between the message (m) and the response (r).  These "exact match" features of the context are used in conjunction with the "similar vector embedding space match" features from the RNNs to come up with the best outcome.
  - Another contribution is a multi-reference extraction technique that supports (but does not replace) automatic evaluation.  The general problem of evaluation through BLEU is difficult because most datasets have a single label as the correct answer, whereas a given query can have vast and extremely diverse valid responses.  To mollify the issue, the authors sample multiple possible responses to evaluate a predicted output against.  Concretely, the authors use IR techniques to extract multiple possible answers where the message and response are similar to the actual message and response.  Then out of those ~15 proposed pairs, some Mechanical Turkers evaluate those pairs on a scale of 1 to 5, and then we keep just those pairs that averaged a score of 4 or higher.
  - "Overall, the outputs tend to be generic or commonplace, but are often reasonably plausible in the context ... longer responses are likely to present information that conflicts either internally within the response itself, or is at odds with the context, [which] is not unsurprising, since the model lacks mechanisms both for reflecting agent intent in the response and for maintaining consistency with respect to sentiment polarity."

[**Neural Responding Machine for Short-Text Conversation**](http://arxiv.org/abs/1503.02364) *(Shang, Lu, Li)* [Read: Mar 26th]

  - At the time of publication, there were roughly two different types of response generation, either retrieval-based methods or statistical machine translation.  RBM pick a suitable response by ranking the candidate responses using a combination of various matching features (e.g. number of shared words, sentence cosine similarity).  The main problem is that responses are pre-determined, which causes lack of variety cannot be customized for style or attitude or tone.  Also, the input and retrieved sentence might match on topic, but directly oppose on tone or conclusion.
  - On the other hand, SMT picks a suitable response by generating one word at a time, conditioned on the previous words that were generated by the model.  It treats response generation as a translation problem, but this is a problem because responses are not semantically equivalent to the posts as in translation.  Namely, one query can receive multiple responses, with completely different content, yet still have all responses be correct.  For example, I can answer the question "How was your day?" in ten different ways, some with short and long sentences, and they are all valid answers.
  - Since this was written in April 2015, it is the precursor to neural conversational model by Vinyals et. al. from July 2015, so the model is very straightforward.  This is simply a encoder-decoder network with attention.  This is the to say, the encoder builds up a real-valued vector representation of the input query in its hidden state which is then used as the initial hidden state of the decoder.  Attention is also used to give the decoder context as it generates output tokens.  The paper should be considered groundbreaking for proposing the model so early, but based on subsequent results, we now all know that such a model is still insufficient for dealing with the problems outlined above.

[**Contextual LSTM Models for
Large scale NLP tasks**](http://arxiv.org/abs/1602.06291) *(Ghosh, Vinyals, Strope, Roy, Dean, Heck)* [Read: Sept 20th]

 - **Idea**: The context around a source sentence can be useful for understanding the main sentence itself, as seen in SkipThought type models. Applying this to RNNs, the authors propose a CLSTM (contextual long short-term memory) unit that takes the sentence topic into account, in addition to the current word when making predictions about the next word.  The "topic" is created by passing all the previous words in the same sentence (or all the words in the previous sentence) into a proprietary hierarchical topic model (HTM) for supervised classification of text.  Basically, the HTM takes a bunch of words and picks the one topic it deems to be the most relevant out of 750 possible topics.
 - **Results**:  On word prediction task (what word comes next), the CLSTM did better than a pure LSTM, but not by much (perplexity of 38.6 goes to 37.6). Adding more hidden units per layer (256, 512, 1024) had a much bigger impact, dropping perplexity from ~38.0 to 31.4 to 27.4 respectively.
 - On sentence selection task (what sentence should come next in this dialogue), the CLSTM improves over a regular LSTM by ~21%.  Results are bit skeptical though because the CLSTM has access to the topic of the target sentence it is predicting (not just the topics of the 3 sources sentences).
 - On the third and final sentence topic prediction task (what is the likely intent of the speaker in the next sentence), the CLSTM was better than other methods BOW-DNN by 7.3%.
 - **Lessons**: Sounds like this would be great research area if Google released their secret HTM system or Google News dataset, but since that didn't happen, we are just left with some mysterious results.  Overall, this seems like a not-so-sophisticated version of attention mechanism in that the next word is predicted using select hierarchical information from previous words.

[**A Persona-Based Neural Conversation Model**](http://arxiv.org/abs/1603.06155) *(Li, Galley, Brockett, Spithourakis, Gao, Dolan)* [Read Mar 27th]

 - **Problem**: One major issue surrounding NMT is their propensity to select the response with greatest likelihood — in effect a consensus response of the humans represented in the training data. Outputs are frequently vague or non-committal such as "I don't know" or "Thank you") because they are safe answers to almost anything, and when not, they can be wildly inconsistent:
   - Q: Where do you live now?
   - A: I live in Los Angeles.
   - Q: In which city do you live now?
   - A: I live in Madrid.
   - Q: In which country do you live now?
   - A: England, you?
 - **Idea**: We address the challenge of consistency and how to endow data-driven systems with the coherent “persona” needed to model human-like behavior:
   - SPEAKER MODEL - integrates a speaker-level vector representation into the decoder portion of a Seq2Seq model in order to add personality to the respondent.  Rather than just inputting x (the word) and h (the previous hidden state), we also input v-i at each time step of encoding.  There is a separate v for each speaker, and this vector is trained through backprop like everything else.  Doesn't seem that useful though when you have just one speaker, or if the speakers don't have the same input sentences.
   - SPEAKER-ADDRESSEE MODEL - encodes the interaction patterns of two interlocutors to model the way the respondent adapts their speech to a given addressee.  They construct an interaction representation from their individual embeddings and incorporating that into the Seq2Seq model. We associate each speaker with a K dimensional speaker-level representation, namely v-i for user i and v-j for user j. We obtain an interactive representation Vi,j ∈ RK×1 by linearly combining user vectors vi and vj in an attempt to model the interactive style of user i towards user j: 
$$V_{i,j} = \tanh(W1 ·v_i + W2 ·v_j)$$
  - which is then fed into the LSTM as another input
 - **Results**: Leveraging persona vectors can improve relative performance up to 20% in BLEU score and 12% in perplexity, with a commensurate gain in consistency as judged by human annotators. "Although the gains presented by our new models are not spectacular, the systems outperform our baseline systems in capturing personal characteristics such as speaking style and background information."
 - **Remarks**: Yes, there are gains to be made from having users in similar vector spaces, but the system still needs a large data corpus with examples from all users to make this work.  It currently seems more anecdotal, rather than a concrete reason for having a Persona consistently applied.  In other words, I can see this working for some cases, but there lacks strong theoretical grounding that this method can generalize to all cases without obscene amounts of data.

[**Attention with Intention for a Neural Network Conversation Model**](http://arxiv.org/abs/1510.08565) *(Yao, Zweig, Peng)* [Read: Mar 28th]

 - **Idea**: Intention is higher level than attention and has its primary role of explaining discourse structure and coherence. 
Authors create a model that consists of three RNNs.  Conversation is modeled as two speakers taking "turns" saying one sentence, with each word in a sentence counting as one time step.
   - Encoder network (source, user): for each time step, inputs are previous hidden state of self, and each new word in the first sentence.  On the next turn, after the agent has talked, the encoder also receives previous hidden state of decoder before inputting new tokens for its second sentence.  The encoder's second sentence is the third sentence in the conversation.
   - Decoder network (target, agent): During the first time step, it has two inputs used to initial hidden state: current encoder state and current intention state.  Then, during that turn, for each time step, it will input a new word and its own previous decoder hidden state.  That decoder hidden state is modified using a context vector (from attention) conditioned on intention RNN and encoder RNN.  On the next turn, after user has spoken for second time, decoder will receive the newly encoded source side and intention.
   - Intention network: Doesn't really have time steps since is no sentence it is trying to encode.  For each turn, inputs are hidden state from previous turn (not previous time step), current encoder state, and previous decoder state.
 - **Results**: Perplexity is down to 22.1 with a 200 hidden size (down from 30.8 with a 50 hidden size), for a vocabulary of ~9000.  In [Sordoni](http://arxiv.org/abs/1506.06714), the context of previous sentences is captured just once and fed into the decoder.  In contrast, this paper uses "turns" so context is different each turn the agent talks.  Additionally, that context is variable because it is updated using attention, so the context changes for each word generated by the agent in each time step within each turn.
 - **Remarks**: Definitely a more sophisticated model than before, but (a) qualitative measures don't pass the eye test.  The responses don't seem all that much better, as if somehow extra context (or intention) helped.  (b) On the quantitative side, perplexity is low, but without other baselines to compare, it is hard to say whether the number is meaningful (since dataset is only available in-house).  Moreover, vocab size is relatively limited (compared to even PTB), so again, not much to compare against.

[**Deep Neural Network Approach for the Dialog State Tracking Challenge**](www.sigdial.org/workshops/sigdial2013/proceedings/pdf/SIGDIAL73.pdf) *(Henderson, Thomson, Young)* [Read: Mar 29th]

 - **Idea**: Use feed forward network (with about 3 hidden states) to figure out the belief tracking, rather than rules based.  the Dialog State Tracking Challenge (DSTC) is an annual competition, the goal for that year was given dialog from a user, predict one of 5 classes.  Dialog was related to people who called into a public transit hotline about Pittsburgh buses.  The five possible intentions of the user are: route, from, to, date and time.
 - **Results**: Apparently did pretty well, this was back at the time when using DNN was still a relatively new idea in 2013.
 - **Lessons**: Interesting that this competition has been going on for so long.  When creating a proper chatbot for company knowledge base, the secrete is that the chatbot will mostly respond with generic responses (so multiple persona vectors are not so critical) because the value-add for an MVP is not having "fun" person to talk to, but rather a hyper efficient mechanism for information retrieval.  Note that the "information" is not just the next sentence to speak, but an actual a piece of information needed so the employee can complete the task as hand.  (For example, I need to RSVP for the company holiday party, where is that link again?  Also, is there a dress code?)


[**Multi-domain Dialog State Tracking using Recurrent Neural Networks**](http://arxiv.org/abs/1506.07190) *(Mrksic, Seaghdha, Thomson, Gasic, Su, Vandyke, Wen, Young)* [Read: Mar 30th]

 - **Idea**: Dialog belief tracking using an RNN.  Key contribution is using transfer learning so that dialog data drawn from different dialog domains can be used to train a general belief tracking model which can operate across all of these domains.  Similar to training a Seq2Seq NMT model on multiple language-pairs (En-Fr, En-Chn, De-En) helps a different language pair (En-Korean) even when the target language pair has limited data. Possible domains include "restaurant search, travel reservations or shopping for a new laptop."
 - **Model**: Input for each user utterance consists of the ASR hypotheses (turn input), the last system action, the current memory vector and the previous belief state. All 5 these vectors are concated to form the input into the vanilla RNN.
 - The turn input breaks into two vectors: lexical and de-lexical.  Lexical n-grams such as [want cheap price] and [want Chinese food] map to the same delexicalised feature, rep- resented by [want tagged-slot-value tagged-slot-name]. Such features facilitate transfer learning between slots and allow the system to operate on unseen values or entirely new slots. As an example, [want available internet] would be delexicalised to [want tagged-slot-value tagged-slot-name] as well, a useful feature even if there is no training data available for the "internet" slot.
 - Delexicalised features allow transfer learning between slots, which authors extend to achieve transfer learning between domains: a model trained to talk about hotels should have some success talk- ing about restaurants, or even laptops. If we can incorporate features learned from different domains into a single model, this model should be able to track belief state across all of these domains.
 - **Results**: The procedure can also be used to initialize dialog systems for entirely new domains. In the evaluation, we show that such initialization always improves performance, regardless of the amount of the in-domain training data available.  The ones trained using data multiple domains performed better than those trained using one domain, even on the original task. For example, model trained with data on Cambridge restaurants, SF restaurants, Michigan restaurants, Tourist data, SF Hotels, and Laptop buying performed the best at predicted belief state for the Cambridge Rest task.

[**Learning from Real Users: Rating Dialogue Success with Neural Networks for Reinforcement Learning in Spoken Dialogue Systems**](http://arxiv.org/abs/1508.03386) *(Su, Vandyke, Gasic, Kim, Mrksic, Wen, Young)* [Read: Mar 31st]

 - **Problem**: A spoken dialogue system (SDS) can be used to help complete some task.  In simple systems, a model can be trained by determining whether or not it was successful at helping the user complete some pre-defined task.  However, in real world systems, prior knowledge of the user’s goal is simply not available, making any calculation of an ‘objective’ measure nearly impossible.  One idea is to ask the user for feedback at the completion of each dialogue, but this leads to noisy signals and many users might not cooperate in offering feedback once their task is complete.  Another idea is to have experts go back and read dialogs and then label whether the model was successful, which is used as a reinforcement learning signal back to the model.  This idea is obviously broken because it does scale.
 - **Idea**: So how do we get around this problem of evaluating our dialog system? Well, what tool do we have in the toolbox? More neural networks!  Basically, we first train a reinforcement learning network to help the user complete their task interpreted as a belief tracking objective.  Then, we train a *second* NN to evaluate whether the first one did a good job, and based on that prediction, feed back into the first model as reward signal.  Kinda like a GAN, but rather than two competing networks, we have two cooperative networks!  We denote the first one a chat model, and the second a rating model.
 - For the rating model, a RNN and a CNN were both tested to see what architecture worked better.  For the RNN, each time step it is fed a "turn vector".  A turn vector is composed of four parts:
   1. one-hot encoding of the user’s top-ranked dialogue act
   2. the real-valued belief state vector formed by concatenating the distributions over all goal, method and history variables
   3. one-hot encoding of the summary system action
   4. turn number
 - Then the final output is then reports on whether the chat model did its job. For the CNN, all the turn vectors are combined to form a matrix.  Then a convolution is performed with some max-pooling, then a multi-layer perceptron, and finally (again) an output layer. 
Authors tested 3 output layer options used to generate a reinforcement learning signal:
  1. Binary classifiers of success vs. failure. Give a -1 reward at each turn to promote speed, and a final reward of +20 at completion if the dialogue was successful, otherwise 0 if failure.
  2. Multiclass classification problem where the class labels are integers representing the final total reward signal for the whole dialogue.  So it just bypasses the formula above and tries to predict the final signal directly.
  3. Regression problem with the actual return value used as the training target. 
 - The final layer of the NN models have no non-linearity (activation) and the whole model is trained with a mean-square-error (MSE) loss function
 - **Result**: The rating model is trained on dialogues generated by simulated users and the best model is then used to train a policy on-line which is shown to perform at least as well as a baseline system using prior knowledge of the user’s task.  Ultimately, binary classification models being the most accurate and again the RNN outperforming the CNN.  Two systems were trained on-line by users recruited via Amazon Mechanical Turk. Firstly, a baseline system was trained which used knowledge of the set tasks to compute the reward,  and secondly a system was trained using only the binary RNN classification model to compute the reward signal. Three policies were trained for each system, then averaged to reduce noise. It can be seen that the RNN system was able to learn at least as good a policy as the baseline system. 

[**A Network-based End-to-End Trainable Task-oriented Dialogue System**](http://arxiv.org/abs/1508.0338) *(Su, Vandyke, Gasic, Kim, Mrksic, Wen,Young)* [Read: Apr 1st]

 -  (Mostly copy and pasted text) The model can converse with human subjects naturally whilst helping them to accomplish tasks in a restaurant search domain.
 - Building a task-oriented dialogue system such as a hotel booking or a technical support service is difficult because it is application-specific and there is usually limited availability of training data. To mitigate this problem, recent machine learning approaches to task-oriented dialogue system design have cast the problem as a partially observable Markov Decision Process (POMDP) with the aim of using reinforcement learning (RL) to train dialogue policies online through interactions with real users. However, the language understanding and language generation modules still rely on supervised learning and therefore need corpora to train on. Furthermore, to make RL tractable, the state and action space must be carefully designed, which may restrict the expressive power and learnability of the model. Also, the reward functions needed to train such models are difficult to design and hard to measure at run-time.
 - At the other end of the spectrum, sequence to sequence learning has inspired several efforts to build end-to-end trainable, non-task-oriented conversational systems. This family of approaches treats dialogue as a source to target sequence transduction problem, applying an encoder network to encode a user query into a distributed vector representing its semantics, which then conditions a decoder network to generate each system response. These models typically require a large amount of data to train. They allow the creation of effective chatbot type systems but they lack any capability for supporting domain specific tasks, for example, being able to interact with databases and aggregate useful information into their responses.
 - The authors propose a neural network-based model for task-oriented dialogue systems by balancing the strengths and the weaknesses of the two research communities: the model is end-to-end trainable but still modularly connected; it does not directly model the user goal, but nevertheless, it still learns to accomplish the required task by providing relevant and appropriate responses at each turn.
 - In order to train the model for the target application, we introduce a novel pipelined data collection mechanism inspired by the Wizard-of-Oz paradigm to collect human-human dialogue corpora via crowdsourcing.  Basically, they set up a webpage that has most of the components that a user or agent would see (such as previous dialog in the task), then have the Turker fill in the blank of what the RL policy model would do.
 - **Remarks**: I am convinced this system strikes the right balance for bringing deep learning to the world.  Most people are so concerned with chatbots that sound like people, but the reality is that a dual -task-oriented system capable of conversing as well is the necessary MVP to launch a revolution.  The personality can be added later.  Full understanding of text (NTM) or image (VQA systems) might never be needed at all.  This is the perfect balance. 

[**A Diversity-Promoting Objective Function for Neural Conversation Models**](http://arxiv.org/abs/1510.03055) *(Li, Galley, Brockett, Gao, Dolan)* [Read: Apr 2nd]

 - **Idea**: Seq2Seq models for dialog generation tend to produce safe, commonplace responses (e.g. I don’t know) regardless of the input. This behavior can be partially ascribed to the relative frequency of generic responses in contrast to the relative sparsity of more contentful alternative responses. We suggest that the traditional objective function of the likelihood of output (response) given input (message) is unsuited to response generation tasks. Instead we propose using Maximum Mutual Information (MMI) as the objective function in neural models. 
 - Responses that seem more meaningful or specific can also be found in the N-best lists, but rank much lower.  Thus, a common work around is to maintain a beam search result of k=30, and then randomly choose from top 5 list when beam search terminates. Part of the problem here is that a beam terminates when it reaches a <EOS> symbol, so the top 5 responses are generally always the shortest answers. This method also fails to take into account the relationship between message and response, instead reducing down to random selection to promote diversity.
 - Another idea is to use Maximal Marginal Relevance (MMR) based methods (Carbonell and Goldstein, 1998).  However, this tends to increase lexical but not semantic diversity, and with no gain in performance.  Concretely, given a question "Where did they go?", rather than saying "I don’t know", it is capable of generating "I haven’t a clue", but what we really want is "They went down the street and made a left." 
 - MMI works by trying to choose parameters that maximize (pairwise) mutual information between the source S and the target T, as opposed to just maximizing log-likelihood of T given S. 
This avoids favoring responses that unconditionally enjoy high probability, and instead biases towards those responses that are specific to the given input. Authors also add a hyperparameter λ that controls how much to penalize generic responses:
![max_mutual_information](/content/images/2017/03/MMI.png)
 - Note that this last equation is just an interpolation of (T given S) vs. (S given T).  Thus, the MMI objective function can thus be viewed as representing a tradeoff between source and target. Authors split off into two separate methods for training, each that come with its own issues:
  - *MMI-antiLM*: The second term of log p(T|S) − λ log p(T) functions as an anti-language model. "λ log p(T)" penalizes not only high-frequency, generic responses, but also fluent ones and thus can lead to ungrammatical outputs. To resolve this issue, note that generating outputs is a task of combining the previous vector representation (from encoder) with the word predicted at the current step (by decoder).  As decoding proceeds, the influence of the initial input (from encoder) decreases since the newly predicted words come to dominate. Thus, the first words predicted by decoder significantly determine the remainder of the sentence.  So, we want a weak LM for words predicted early on by the language model in order to increase diversity. Additionally, the authors observed that ungrammatical segments tend to appear in the later parts of the sentences, especially in long sentences.  This implies that we want a strong LM for words predicted later on in the sentence.  Thus, the formula is altered with a weight g(k), that is big for the first words (early on in the decoding), and small in the later words (later on in the decoding). Concretely, note that
p(T) is the job of the decoder to predict a  word time k, which is influenced as the product of all probabilities from 1, 2, 3, .. k-2, k-1. Then: 
     - if k is first word generated by decoder, then g(1) = big weight, then [g(1)⋅λ log p(T)] is big, so there is a strong anti-LM component, which promotes response diversity
     - if k is second word, then g(2) = big weight penalty
     - if k is previous word, then g(k-1) = smaller weight, then [g(k-1)⋅λ log p(T)] is small, so there is a weak anti-LM component, which promotes better grammar
  - *MMI-bidi*: Direct decoding from (1 − λ) log p(T|S) + λ log p(S|T) is intractable, as the second part requires completion of target generation before p(S|T) can be effectively computed. p(S|T) is basically, what is the best "source" given a "target"?, but to answer that question, we must have a target first. Finding all possible targets is "target generation".  Due to the enormous search space in finding "targets", exploring all possibilities is infeasible.  Thus, for practical reasons, we approximate by first coming up with some N-best options using the first part of objective function, (i.e. standard SEQ2SEQ model p(T|S)), then rerank those N options using the second term of the objective function.  This is very similar to the beam search idea proposed earlier, but now at least we choose a final output based on some sort of objective, rather than random selection.  Model reranking has obvious drawbacks.  It results in non-globally-optimal solutions by first emphasizing standard SEQ2SEQ objectives. Moreover, it relies heavily on the system’s success in generating a sufficiently diverse N-best set, requiring that a long list of N-best lists be generated for each message.  
 - **Results**: The MMI-bidi one worked best in producing more diverse, interesting, and appropriate responses, and better BLEU scores. MMI-antiLM also worked well, especially when Seq2Seq p(T|S) produced all generic responses (because choosing the best response out of all bad things still returns a bad thing). The diversity in response is definitely apparent, but the limit of 20 words max still seems limiting (i.e. I would like to see diversity in length and tempo, as well as diversity in topic.)
 - **Lessons**: Beam search done right is super critical.  This echoes the finding by Britz et al.  In this paper, the N-best lists are generated using the decoder with beam size B = 200. We set a maximum length of 20 for generated candidates. At each time step of decoding, we are presented with B × B candidates. We first add all hypotheses with an EOS token being generated at current time step to the N-best list. Next we preserve the top B unfinished hypotheses and move to next time step. We therefore maintain beam size of 200 constant when some hypotheses are completed and taken down by adding in more unfinished hypotheses. This will lead the size of final N-best list for each input much larger than the beam size.

[**A Character-Level Decoder without Explicit Segmentation
for NMT**](http://arxiv.org/abs/1603.06147) *(Chung, Cho, Bengio)* [Read: Sept 24th]

 - **Idea**: Historically speaking, most encoder-decoder models have assumed that words are the basic unit of meaning, which leads to translation as a mapping from words in one language to words in another language. However, this may be suboptimal since for many corpora (especially in non-English languages), there is no clear cut way to segment sentences into lexemes and morphemes, which are the actual smallest units of meaning. For example, if we apply a basic tokenization script to the corpus “eat”, “eats”, “ate” and “eating”, all the words end up as separate entries in the vocabulary even though they clearly share the same lexeme “eat” (along with the morphemes "s" and "ing").
 - In addition to inefficiency in modeling, there are practical negative consequences from using raw words. In particular, when a novel word is found in the test set (i.e. driving) the translation system often resorts to using an < UNK > token, even though the words "drive" and "running" *were* in the corpus (which would have yielded the morphemes "drive" and "ing" that could be useful in translating the unknown word.)
 - Thus, the authors propose translation on a sequences of *characters* rather than words. One might object to this method because long sentences with many words already exhibit issues with vanishing gradients, and interpreting that same sentence as a sequence of characters will only exacerbate the issue.  However, the authors believe that improved algorithms, namely LSTMs and GRUs, have allowed researchers to learn longer dependencies.
 - While past work has shown translation success with characters (Ling et al., 2015), this paper adds to that progress by showing that the segmentation into words as an intermediary step is not necessary.  They also introduce a new bi-scale recurrent neural network that has two sets of gated, hidden units (h1) which operates at a fast-changing timescale and (h2) which operates at a slower timescale, based on the belief that words and characters need to be translated at different speeds.
 - **Results**: Character-based decoding performance is on par compared to byte-pair (subword) encoding.  As the authors acknowledge, it would have been nice to see character-based encoding (source words) side as well. There also seems be a lack of comparison with traditional word-based methods.
 - The evidence for using the bi-scale RNN (vs. a stacked RNN) seems minimal, especially given the complexity of the new architecture. Finally, although a single word often doesn't represent a basic unit of meaning, a single character certainly does not either. One is too "big" and the other is too "small".  I'm not sure the authors address this point.

[**Dialog-based Language Learning**](https://arxiv.org/abs/1604.06045) (*Jase Weston*) [Read Nov 1st]

 - **Idea**: Real life conversation doesn't accept feedback through labeled training data.  Rather if a Person X is talking to a Person Y, then Y will offer responses with more text.  A chatbot that can survive IRL would need to be able to adapt to that reality after being pre-trained in a lab. Specifically, rather than predicting an output, denoted u-2,  after a question/input, the system also takes that u-2 to predict an additional response that a "teacher" or expert student presumably makes, denoted u-3.  Thus, the extra information from the u-3 loss can be used to improve u-2, which was the "real goal".
 - This method of training seems to have many inherent issues:
   1. If no dialogue was offered, then the training falls apart since it requires a u_3.  Conversely, if the conversation is multiple passes back and forth, then the system also falls apart since it does forward prediction on just one "lookahead".  This is similar to the drawback on hard-coded hops when using augmented memory - any time the optimal number of hops is more than or less than the hard-coded amount, the system fails to perform, which breaks generalizability.
   2. The model has an extremely limited set of possible responses from the teacher (some times as little as 6!) which does not mimic the real world at all.  If the goal is to learn from natural dialog, then the feedback should be incorporated as natural language rather than just a label of 6 classes.
   3. Real world chatbots don't get discrete rewards (positive or negative head nods) like the system presented in the paper. If the user presented explicit rewards (4 out of 5 stars, thumbs up, etc.)  to the system, then we are using labels again rather than natural language, defeating the purpose of the model.
 - **Results**: The system is able to learn to answer some questions correctly with only text-based supervision (and does not require extra rewards-based supervision).  The authors fully acknowledge this is simply a "small step" in the right direction, rather than a breakthrough result.
 - **Remarks**: IMHO, reinforcement learning with delayed, noisy reward signals from the user is still the more likely solution. The advantage of being able to update the policy on the fly is far more flexible, and predictive lookahead as presented here does not seem powerful enough to overcome the disadvantages of a static policy.