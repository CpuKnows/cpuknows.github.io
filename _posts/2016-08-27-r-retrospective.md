---
layout: post
title: R Project Retrospective
description: A retrospective on my first large R project
---

{{page.title}}
==============

We (my company) originally received a request for a real-time scoring engine for a B2B lead pool, but we scaled it back to a daily operation because rapid feedback wasn’t necessary. We were to take multi-channel data from email, call centers and website activity to evaluate how likely a lead was to sit down for a sales pitch. These scores were then made available to sales reps through Salesforce to help them choose who to follow up on. I implemented a system in R to support evaluation of millions of leads to call center’s across the US while providing access for reporting, anomaly investigation, and historic snapshots.

With a single score available to sales reps they have less information to evaluate before deciding to call a lead. Our hypothesis was that they would be more likely to rely on a single score because of the ease of decision making and hopefully reduce personal bias. For example, a particular rep may have had previous luck with leads with x, y, and z characteristics and therefore want to seek out leads that look similar even if at a company level this isn’t a successful strategy. Our score on the other hand has a higher level view of what’s successful in the company and we want to push those strategies. Not all personal biases are wrong and experienced sales reps have valuable domain knowledge. The purpose behind this kind of scoring is not to eliminate biases of sales reps, but rather to shift sales rep decisions toward globally effective strategies without diminishing the variability of different call centers.

### Why Deciling is Effective #

My coworker performed the original analysis and developed a decile based model. Certain subsections of the population perform very differently, such as referrals vs web leads, so it makes sense to create a model for each of these subsets based on different variables related to that subset. Deciling model variables loses out on granularity or the best split point, but it still captures the bulk of the relationship. A lot of times simple is still very effective.

<figure>
	<img src="/img/2016-08-27-r-retrospective/feature_x.png">
</figure>

Often continuous feature will have a trend from one end of the spectrum to the other. If the feature looks like noise or there’s a lot of variability within a decile, then deciling won’t work well. That should also make you question whether that feature is a good choice to include in your model.

<figure>
	<img src="/img/2016-08-27-r-retrospective/conversion_rates.png">
</figure>

<figure>
	<img src="/img/2016-08-27-r-retrospective/results_diff.png">
</figure>

Even though there are a range of conversion rates within each decile the difference is minimal. Using the deciles of feature x to predict conversions is almost as good an approximation as the underlying distribution for the same reason secant lines are a good approximation of the rate of change on a curve. The more bins you have the better the approximation would be, but you need a reasonable limit. Too many bins would cause a lot of headaches and overfit the data.

### Implementation #

The deciled models were developed in SAS and I needed to implement the system in R to automate the scoring. A model based on deciled features is human readable and reading the model into R becomes easy. All I need to do is put the variable scores in a file and create functions in R to read them. This also abstracts model variables from hardcoded values so I will need to rewrite less code to update the model.

It was important for me to think about what data needed to be stored in a database for scoring, reporting, anomaly investigation, and historic snapshots. When you start a project you’ll want to record all variables of interest so you can create a model in the future. You can’t go back in time to get that data so it’s better to be safe than sorry. The same applies to the input and output of a model. Requirements for model reporting can change just like requirements for software development can change. You’ll be future proofing your model.

You should be able to answer:
<ul class='pretty'>
	<li>How do can I explain the model output of a subset?</li>
	<li>How do I handle when input data streams fail?</li>
	<li>How will I know if the population distribution is changing?</li>
	<li>How has the population moved since last quarter?</li>
	<li>How does pre and post business logic affect my modeling?</li>
	<li>How can I refresh the data?</li>
</ul>

I learned so much about creating systems in R from this project. There were a number of libraries I need to learn to perform logging ([futile.logger][futile logger]), unit testing ([testthat][testthat]), computationally efficient calculations ([Rcpp][Rcpp]), and more. I was constantly discovering ways to improve my code. I reached a place of comfort with R where I would be sitting on the bus and solutions to issues would pop in my head. I learned a million little things from this project rather than having epiphanies. I highly recommend Hadley Hickham’s [Advanced R][advr].

I decided to structure the system to run from a main R file instead of another common architecture for analysis tasks, running discrete steps from a makefile. The benefit of the makefile approach is it increases reproducibility and intermediate datasets don’t need to be rebuilt if the input to that step is unchanged. These steps are very much like math functions. If you give it the same input it should have the same output. This works well when all you need to do is backup an assertion from your analysis, but real world systems come with more requirements.

This project needed to work with external tools and databases which complicates the simplicity of the makefile approach. Because of requirements of decaying variables from the database the output of one section changes depending on how many times it’s run and when. This means the system must be robust to failures at this point and won’t necessarily have the same output every time it’s run. Additionally, this system would be getting fresh data from each input source every time it’s run so the lazy evaluation of the makefile steps is unnecessary. These issues are common in real world systems and software development in general.

### Reporting #

Keep in mind that when you’re creating a model to optimize a business process someone will be presenting the results to their boss or boss’s boss. These people care more about sales numbers than how small your mean squared error is. Even if your model’s objective function is solid you need to be aware of other important metrics. Business stakeholders care how our model affect the number of calls made, how they’re allocated, and data integrity. If sales numbers are affected you better be prepared to explain why.

I’ve gained a lot of knowledge and reusable code from this project so I look forward to using that to streamline development of similar systems. It’s so important to plan ahead both by yourself and with stakeholders in a project like this because new requirements are guaranteed to pop up and you’ll run into limitations. If I was to improve this system I could add more interactive data visualization and reporting using RShiny / D3 / tableau.



[futile logger]: https://github.com/zatonovo/futile.logger/blob/master/README.md
[testthat]: https://github.com/hadley/testthat/blob/master/README.md
[Rcpp]: http://dirk.eddelbuettel.com/code/rcpp/Rcpp-introduction.pdf
[advr]: http://adv-r.had.co.nz/
