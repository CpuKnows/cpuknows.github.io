---
layout: post
title: Bayesian A/B Testing
description: Using PyMC to approximate how good our A/B testing really is
---

{{page.title}}
==============

A/B testing is a great tool for making data driven decisions when the feedback from tests is relatively quick. It's widespread in web development for this very reason. The ease of testing landing pages, checkout processes, or other metrics of success with near instantaneous feedback is a huge advantage.

We want to know which is better, A or B. We want to know the absolute Truth with a capital T, but we're only going to run a test on a small portion over a fixed period of time. This gives us the truth of the test, but not the Truth. For example if we flipped a fair coin 100 times we could easily not get 50 heads and 50 tails. Does this mean the coin isn't fair? No, simply that we observed something different in our test, but the Truth is still the same. So how can we account for this?

Bayes' Theorem revolves around the idea that we have a prior belief, make observations to use as evidence, and finally create a posterior belief based on that evidence. With a frequentist approach the results of the A/B testing would be our final belief about the Truth. Instead with a Bayesian approach using hypothesis testing we can assume that A and B are equally effective as our prior, use the results of A/B testing as evidence, and create a new belief about the Truth. This can help us make a more informed decision based on the strength of our evidence.

Many examples of A/B testing assume a 50-50 split in the test set, but often in the real world this will not be the case. For example, if we observe P(A) = 0.3 and P(B) = 0.25, but the size of our test sets are N_A = 100 and N_B = 1000 we cannot say for sure that A is better. Bayesian models take the size of the test set and therefore the strength of our evidence into account. With fewer observations we become more uncertain in our results. In a normal distribution this will create a higher standard deviation to reflect our uncertainty.

First we're going to generate some test data from the Truth (A = 0.05, B = 0.04) with a Bernoulli random variable. Notice that our observations don't match the Truth. From a frequentist perspective we would be finished with our test. 

{% highlight python %}
import pymc as pm

A_true = 0.05
B_true = 0.04

N_A = 1500
N_B = 750

observations_A = pm.rbernoulli(A_true, N_A)
observations_B = pm.rbernoulli(B_true, N_B)

print(observations_a.mean())
print(observations_b.mean())

>>0.0546666666667
>>0.0413333333333
{% endhighlight %}

We'll begin with the assumption that P(A) and P(B) could be any value between 0 and 1, a uniform prior. Obviously if we're testing a website conversion rate it isn't going to be 100% so if we wanted to we could incorporate this prior knowledge into our model with these variables. Calculating the delta between the two choices helps us understand which is better and how much better we think it is. Our posterior will be a Bernoulli distribution for A and B. We're going to sample from this distribution with [MCMC][mcmc] disregarding the first 1000 samples as they are very unlikely to converge and we don't want them to influence our model.

{% highlight python %}
import pymc as pm

p_A = pm.Uniform('p_A', lower=0, upper=1)
p_B = pm.Uniform('p_B', lower=0, upper=1)

@pm.deterministic
def delta(p_A=p_A, p_B=p_B):
    return p_A - p_B

obs_A = pm.Bernoulli('obs_A', p_A, value=observations_A, observed=True)
obs_B = pm.Bernoulli('obs_B', p_B, value=observations_B, observed=True)

mcmc = pm.MCMC([p_A, p_B, delta, obs_A, obs_B])
mcmc.sample(20000, 1000)
{% endhighlight %}

Voila! You can see that the standard deviation of the posterior of P(B) is larger than P(A) reflecting that we are less sure of it's value because B had less test data. We're still pretty sure that A is better than B because most of the delta distribution is positive.
<figure>
	<img src="/img/2015-2-22-bayesian-ab/simple_ab.png">
</figure>

---------------------------

There's a lot involved in how A/B tests are run. [Evan Miller][how not] talks about not stopping your test before it has properly concluded because it can lead to drawing conclusions without the proper significance. [Chris Stucchio][not bandit] talks about how use cases and user behavior affects when and how long a test should be run to avoid errors. [Julia Evans][aa test] gives an example of how to validate your test methods or explore fishy results with A/A testing. Basically you run your test with 2 control groups to check that there is no noticeable difference. You're just as likely to have an error in the human element of these tests and how they're set up as the technical elements so it's important to understand potential pitfalls.

We can do better than measuring delta with an A/B test though. Putting up a possibly inferior product for testing with real customers incurs a cost. If version A has a conversion rate of 5% less than version B, but you have to run the test for a week that's going to cost a lot of potential conversion. We want to find out the best answer as quickly as possibly to minimize this cost. This is where the multi-armed bandit problem comes into play. 

> In probability theory, the multi-armed bandit problem is a problem in which a gambler at a row of slot machines (sometimes known as "one-armed bandits") has to decide which machines to play, how many times to play each machine and in which order to play them. When played, each machine provides a random reward from a distribution specific to that machine. The objective of the gambler is to maximize the sum of rewards earned through a sequence of lever pulls. - [Wikipedia][bandit definition]

Multi-armed bandit or Bayesian bandit is a Bayesian method that incorporates evidence of how well a test group is performing to adjust the distribution of new participants into test groups. Here's and example of a Bayesian bandit predicting 2 different probabilities using PyMC:
<figure>
	<img src="/img/2015-2-22-bayesian-ab/2_bandit_example.png">
</figure>

What's happening is that on each iteration the bandit algorithm is going to:

1. Sample a value from each prior distribution. (In these examples we're starting with **Beta(alpha = 1, beta = 1)** also known as a uniform distribution)
2. Select the largest sample (bandit) from the set created in step 1
3. Test with the bandit chosen in step 2 and update its prior based on the results
4. Repeat

Just like the Bayesian A/B test you can include any prior knowledge in the prior defined at the beginning of the algorithm. You can see that green has a higher probability so over time the bandit algorithm is going to choose it more often as it becomes more certain that green is superior. With Bayesian bandit you don't need to limit your test to 2 possibilities. Here's a prediction of 20 different possibilities:
<figure>
	<img src="/img/2015-2-22-bayesian-ab/20_bandit_example.png">
</figure>

Quantifying this cost is called regret. We can see that the difference in regret between a predetermined length A/B test and Bayesian Bandit is large. Bayesian bandit regret is a [proven][prove] logarithmic function. This is the regret with a 1% difference between test groups over the course of 10,000 iterations.
<figure>
	<img src="/img/2015-2-22-bayesian-ab/regret.png">
</figure>

The set length A/B test is going to use the wrong version half of the time which really adds up. Bayesian bandit is also capable of learning the best option in a changing environment. We can guide this behavior by adding a learning rate to the algorithm. There are different implementations of the multi-armed bandit algorithms such as [UCB][ucb] which is worth looking into. There's a great set of tools here for making data driven decisions and I look forward to learning more about A/B testing and utilizing PyMC's probabilistic models.



[mcmc]: http://en.wikipedia.org/wiki/Markov_chain_Monte_Carlo
[how not]: http://www.evanmiller.org/how-not-to-run-an-ab-test.html
[not bandit]: https://www.chrisstucchio.com/blog/2015/dont_use_bandits.html
[aa test]: http://jvns.ca/blog/2015/02/06/a-a-testing/
[bandit definition]: http://en.wikipedia.org/wiki/Multi-armed_bandit
[prove]: http://arxiv.org/pdf/1111.1797.pdf
[ucb]: http://jeremykun.com/2013/10/28/optimism-in-the-face-of-uncertainty-the-ucb1-algorithm/