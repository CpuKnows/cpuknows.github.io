---
layout: post
title: Implementing Decomposable Attention for the Quora Dataset
description: Using attention to help detect duplicate questions
---

{{page.title}}
==============

I recently participated in the [Quora challenge][kaggle] on Kaggle to determine if two sentences are duplicates. The benefit of being able to automatically identify duplicate questions is it allows people to find solutions to previously asked questions quickly, helps experts avoid wasting time answering the same questions repeatedly, and reduces the burden on human moderators. To me it seemed like a cool way to learn more about NLP techniques. I was particularly interested in experimenting with **attention**.

Neural networks are often described as black boxes, but attention allows us to look into the decision process in an intuitive way. Attention tells the network where to focus. To see a great explanation and visualization of an attention mechanism see this post from [distill.pub][distill].

In the Quora dataset we want to compare 2 questions to see if they’re asking the same thing.
<ul class='pretty'>
	<li>Question 1: “How can I learn to play soccer better?”</li>
	<li>Question 2: “What is the optimal way to improve at soccer?”</li>
</ul>

Not every word is important to this task. In question 1 "can" is probably not relevant to the meaning of the question. You may consider just removing stop words so the remaining words carry more meaning in the question, but the goal of attention is to teach the network what is relevant and in this model we’ll compare what is relevant to each question.

In the above sentences "learn", "play", "soccer", "better", "optimal", "way", and "improve" are probably relevant to the question’s meaning. The problem here is comparing if the questions are duplicates of each other so the network compares the normalized attention of question 1 to the embedding of question 2 and vice versa.

<figure>
	<img src="/img/2017-07-23-decomposable-attention/attention - soccer.png">
	<figcaption>A duplicate question</figcaption>
</figure>

In this example attention for “How” and “What”, “learn” and “improve”, “to” and “to”, “soccer” and “soccer”, “better” and “improve” pair together. The network will aggregate these values and run them through another layer before scoring the final prediction. This particular example is flagged as a duplicate.

What does is look like when a question isn’t a duplicate?

<figure>
	<img src="/img/2017-07-23-decomposable-attention/attention - football duck.png">
	<figcaption>Unique questions</figcaption>
</figure>

In this case “How”, “can”, and “I” match up very well. “football” even matches up with “duck”. The large numbers don’t mean that “football” and “duck” have similar meanings like distances between word embeddings. It means that the attention mechanism is giving them similar weights in the question. The network predicts that these questions aren’t asking the same thing.

<figure>
	<img src="/img/2017-07-23-decomposable-attention/attention - soccer sunday.png">
	<figcaption>These questions have very little overlap</figcaption>
</figure>

Finally we can see where the model fails.

<figure>
	<img src="/img/2017-07-23-decomposable-attention/attention - thursday sunday.png">
	<figcaption></figcaption>
</figure>

“Thursday” and “Sunday” are very related by the attention mechanism and their embeddings. These questions are asking something very similar, about the order of days in the week, but possibly a duplicate. It depends on how Quora defines duplicates. The strict answer to these questions is “Friday” and “Saturday”, but the answer to both is “The week is comprised of the days Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday following in that order and repeating.”

We can see how there could be examples of more domain specific questions where a combined answer wouldn’t be as succinct or desirable. The above example shows how this model lacks knowledge of the outside world, the relation of days of the week. With enough text data an algorithm could learn the order of the days of the week through correlation, but again when we move to a more complicated domain this becomes more challenging for the model to learn.

This model architecture from [A Decomposable Attention Model for Natural Language Inference][parikh] provides an efficient way to calculate entailment with attention according to sources in the paper. I put in 300 dimension word embeddings of 30 word sentences and several hidden layers of 200 nodes. The model ended up with under 400k trainable parameters. I don’t know about other popular entailment models, but I’ve definitely come across state of the art models with 10s of millions of parameters so this seems good to me. I definitely appreciate the smaller model and faster training times because that means I don’t have to pay for time on AWS.

The winning solution to the Quora challenge ended up using this decomposable attention model, but also a siamese network, an enhanced sequential inference model, and more. These were stacked together to form the final classifier. That’s great for winning Kaggle challenges when the only metric that matters is error rate, but it’s a production and maintainability nightmare. You can read more about it [here][winning].

I always like to think of ways I could improve a model even if I don’t have the time to test every variation. One thing I noticed is homonyms being confused if the different versions were in each question. Word embeddings are actually ok at encoding each version of a word, but it becomes more difficult if there isn’t a lot of context around the word. I’ve never seen an embedding set with parts of speech tags appended to words. Probably because it would take a lot of extra work and pos taggers aren’t 100% accurate.

There are similar entailment datasets I could have used to improve training such as [SNLI][snli] and the [Stack Exchange corpus][stackexchange]. Although these datasets are slightly different I may have been able to manipulate them to add more training data.



[kaggle]: https://www.kaggle.com/c/quora-question-pairs
[distill]: https://distill.pub/2016/augmented-rnns/
[parikh]: https://arxiv.org/pdf/1606.01933.pdf
[winning]: http://chaire-dami.fr/files/2017/06/Quora_winning_solution.pdf
[snli]: https://nlp.stanford.edu/projects/snli/
[stackexchange]: http://nlp.cis.unimelb.edu.au/resources/cqadupstack/
