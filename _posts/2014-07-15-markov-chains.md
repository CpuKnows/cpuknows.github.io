---
layout: post
title: Mark V. Shaney
description: Exploring Markov Chains and Hidden Markov Models with Alice in Wonderland... being chased by zombies and vampires
---

{{page.title}}
==============

This post started out because I really wanted to do a fun project I could throw together in a day. I decided on a Markov chain text generator. A great example is [Garkov][garkov], a text generator for Garfield comic strips. I used Markov Chains to generate almost realistic sounding sentences from [Alice and Wonderland][alice]. The only logic I added is to start a sentence with a sentence beginning from the text and end on a period. I didn't want the sentences to sound too good because I enjoy the silliness of it, but I could've used natural language processing to create more realistic sentence structures. Even without advanced NLP techniques the results are quite good.

>  The other guests had taken advantage of the game, the Queen said-- 'Get to your little boy, And beat him when he sneezes; For he can thoroughly enjoy The pepper when he finds out who I WAS when I learn music.' These words were followed by a row of lodging houses, and behind it, it occurred to her feet, they seemed to be otherwise. I wish I had not got into the wood.

Each two consecutive words in the text are used as keys in a Python dictionary with each possible next word used throughout the text as the mapped values. For example, the following sentence is mapped out like so:

{% highlight pytb %}
"The tall green men living on Mars talk of tall green plants living on Earth while we talk of them."

{('The', 'tall'), ['green'],
 ('tall', 'green'), ['men', 'plants'],
 ('green', 'men'), ['living'],
 ('men', 'living'), ['on'],
 ('living', 'on'), ['Mars', 'Earth'],
 ('on', 'Mars'), ['talk'],
 ('Mars', 'talk'), ['of'],
 ('talk', 'of'), ['tall', 'them.'],
 ('of', 'tall'), ['green'],
 ('green', 'plants'), ['living'],
 ('plants', 'living'), ['on'],
 ('on', 'Earth'), ['while'],
 ('Earth', 'while'), ['we'],
 ('while', 'we'), ['talk'],
 ('we', 'talk'), ['of']}
 {% endhighlight %}

 When we're generating text a two word pair is our current state in a Markov Chain and we choose randomly between each dictionary value for the next state until we've satisfied some length. This is a second order Markov Chain because it uses the 2 previous states, in this case 2 words, to generate the next state.

I have to credit [Lindsey Bieda][zombie markov] for this great illustration of a Markov Chain:
>Consider a world filled with zombies, vampires, and humans. In this world humans can become zombies, vampires, or stay human. Zombies can become humans by a drug created by Dr. Zork or stay zombies. Vampires can become zombies if they feed on zombies or stay human. In a Markov chain we want to consider the probabilities of each of these things occurring. Humans have a 0.50 chance of staying human and a 0.25 chance of becoming a zombie and a 0.25 chance of becoming a vampire. Zombies, on the other hand, have a 0.15 chance of the cure actually working and becoming human again and a 0.85 chance of staying a zombie. Vampires have the highest chance of staying the same at 0.95 and only a 0.05 chance of stupidly feasting on zombie.

<figure>
	<img src="/img/2014-7-15-markov-chain/mc_figure1.png">
	<figcaption><a href="rarlindseysmash.com">rarlindseysmash.com</a></figcaption>
</figure>

Markov Chains are a pretty simple concept, a state machine with probabilistic transitions. They are great for modeling closed systems in chemistry, physics, economics, and social sciences. For computational work they can be represented simply as a state machine matrix.

<figure>
	<img src="/img/2014-7-15-markov-chain/mc_figure2.png">
	<figcaption><a href="rarlindseysmash.com">rarlindseysmash.com</a></figcaption>
</figure>

A Markov Chain is just one form of a Markov Model which can be represented by a time sliced diagram known as a Trellis diagram. The **y** states are observed while the **x** states are hidden. We can only know within a certain probability what the hidden states are. To model this system we need three pieces of information:

<ul class='pretty'>
	<li>Initial distribution - probability distribution of which state the system begins at, ex: probability of x(t = 0).</li>
	<li>Transition probabilities - probability distributions of each state transitioning to every other state, ex: probability of x(t-1) -> x(t).</li>
	<li>Emission probabilities - probability distribution between a hidden state and each possible observed state, ex: x(t) -> y(t).</li>
</ul>

<figure>
	<img src="/img/2014-7-15-markov-chain/hmm_trellis.png">
	<figcaption><a href="wikipedia.org">wikipedia.org</a></figcaption>
</figure>

Hidden Markov Models (HMM) are a way to uncover the hidden states, for example a Markov Chain, based on observed outcomes. We can use the Human, Vampire, Zombie Markov Chain presented above with Scikit Learn's HMM implementation to perform a couple of useful tasks. First, we can generate sample data for state transitions using **GaussianHMM**. The starting point (initial distribution) will always be human, the transition probability will be the state machine matrix described above, and the emission probability will be a Gaussian distribution.

We're going to generate states described by 2 features, [*desire to eat humans*, and *preference for night* ], we could have more features, but that makes it harder to visualize. In addition to the probability matrices we need to specify the mean and covariance of each state to be generated. Humans generally don't like to eat other humans and although I enjoy Chicago's nightlife the probabilities of becoming a vampire or zombie make me a little more wary, so the human mean is [-1, 3] with a covariances of [[.5, 0], [0, .5]]. Zombies REALLY like eating humans. In fact it's probably the only thing they think about. They don't care what time of day it is, most likely because they're entirely consumed by the thought of eating human flesh, so zombie mean is [9, 5]. Zombies are pretty much all alike so they have covariance [[.1, 0], [0, .1]]. Vampires enjoy a fresh human now and then, but it isn't an insatiable desire. If vampire movies from the 90s have taught me anything it's that they also enjoy going to underground clubs and dancing to techno music, so vampire is mean [8, 9] with covariance [[.8, 0], [0, .8]] because they're open minded.

{% highlight python linenos %}
from sklearn import hmm

startprob = np.array([1.0, .0, .0])
transmat = np.array([[.5, .25, .25],
                     [.15, .85, .0],
                     [.0, .05, .95]])

model = hmm.GaussianHMM(n_components=3, covariance_type='full', startprob=startprob, transmat=transmat)

# The means of each component
# x-axis = desire to eat humans
# y-axis = preference for night time
means = np.array([[-1.0,  3.0], # H
                  [9.0, 5.0], # Z
                  [8.0, 9.0], # V
                  ])
# The covariance of each component
covars = [.5 * np.identity(2),
          .1 * np.identity(2),
          .8 * np.identity(2)]

# Instead of fitting it from the data, we directly set the estimated
# parameters, the means and covariance of the components
model.means_ = means
model.covars_ = covars

# Generate samples
X, Z = model.sample(100)

# Plot the sampled data
plt.plot(X[:, 0], X[:, 1], "-o", label="observations", ms=6, mfc="orange", alpha=0.7)
plt.xlabel("Desire to eat humans")
plt.ylabel("Preference for night")
plt.show()
{% endhighlight %}

<figure>
	<img src="/img/2014-7-15-markov-chain/sample_chain.png">
</figure>

Technically this represents one individual's state transitions so we can infer that they're not having a good day. We can also perform the reverse operation by taking a set of observations to discover the model start probabilities, transition probabilities, mean, and covariance. We'll fit the model to the sample points we just generated to test this out. This uses a form of gradient descent to make predictions so it's possible to get stuck at a local minimum. You should always tweak the parameters and compare results using the **score()** method for the best model. These results are close enough where we can identify the human, vampire, and zombie states. They would be even better with more data. Based on the generated sample points above it may appear like it's finding these states with a form of clustering, but it's actually using the [Viterbi Algorithm][viterbi] which I won't discuss, but feel free to find out more.

{% highlight python linenos %}
n_components = 3

# make an HMM instance and execute fit
model = hmm.GaussianHMM(n_components, covariance_type="diag", n_iter=1000)
model.fit([X])

# predict the optimal sequence of internal hidden state
hidden_states = model.predict(X)

print("Transition matrix")
print(model.transmat_)
print()

print("means and vars of each hidden state")
for i in range(n_components):
    print("%dth hidden state" % i)
    print("mean = ", model.means_[i])
    print("var = ", np.diag(model.covars_[i]))
    print()
{% endhighlight %}

{% highlight pytb %}
Transition matrix
[[  9.56571630e-01   3.21445949e-18   4.34283696e-02]	# Vampire
 [  2.66666992e-01   6.00000000e-01   1.33333008e-01]	# Human
 [  4.89110757e-12   3.35130816e-01   6.64869184e-01]]	# Zombie

means and vars of each hidden state
0th hidden state 						# Vampire
mean =  [ 8.1170754   8.95823949]
var =  [ 0.6836378   0.81404009]

1th hidden state 						# Human
mean =  [-0.95176582  3.04604507]
var =  [ 0.39274973  0.59655972]

2th hidden state 						# Zombie
mean =  [ 8.95105501  5.19157599]
var =  [ 0.10974938  0.26489229]
{% endhighlight %}

That's a basic introduction to Markov Chains and Hidden Markov Models. There are a lot of other cool (and useful) topics in this space to explore, like Markov Chain Monte Carlo, so don't stop learning! You can download the IPython Notebooks I used if you'd like to play around with [text generation]({{ site.url }}downloads/HVZ_Makov_Chain.ipynb) or [HMMs]({{ site.url }}downloads/Markov_Chain_Text_Generator.ipynb).

[alice]: http://www.gutenberg.org/ebooks/11
[garkov]: http://joshmillard.com/garkov/
[zombie markov]: http://www.rarlindseysmash.com/posts/2009-11-21-making-sense-and-nonsense-of-markov-chains
[viterbi]: https://www.youtube.com/watch?v=RwwfUICZLsA