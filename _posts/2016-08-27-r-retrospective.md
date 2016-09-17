---
layout: post
title: R Project Retrospective
description: A retrospective on my first large R project
---

{{page.title}}
==============

I’m quite proud of my latest project. It’s a type of system that I’ve never made before so I wanted to write a retrospective on the project. We (my company) originally received a request for a real-time scoring engine for top of funnel marketing leads. We ended up scaling it back to a daily operation because real-time response wasn’t necessary. We were to take multi-channel data from email, call centers and web activity to evaluate how likely a lead was to sit down for a sales pitch. These scores were then made available to sales reps through Salesforce to help them choose who to call.

Many of our findings weren’t very shocking in that more recent activity is a good indication of a lead’s likelihood to sit down. Also that more recent, longer calls are better than opening an email. These conclusions while intuitive don’t make the process of scoring a useless endeavor. With a single score available to sales reps they have less information to evaluate before deciding to call a lead. Our hypothesis was that they would be more likely to rely on a single score because of the ease of decision making and hopefully reduce personal bias. For example a particular rep may have had luck with leads with x, y, and z characteristics and therefore want to seek out leads that look similar even if this isn’t a successful strategy at the company level. Our score on the other hand has a higher level view of what’s successful in the company and we want to push those strategies. Not all personal biases are wrong though and experienced sales reps have valuable domain knowledge. The purpose behind this kind of scoring is not to eliminate biases of sales reps, but rather to shift sales reps’ decisions toward globally effective strategies.

My coworker performed the original analysis and developed a decile based scorecard model. Basically certain subsets of the population perform very differently, such as referrals vs web leads, so it makes sense for the model to create a scorecard for each of these subsets. These scorecards may each be driven by different variables. Deciling model variables (sticking the values into 10 equally sized bins) loses out on granularity or the best split point, but it still captures the gist of the relation. The upside to deciling is that it’s simple to explain and demonstrate to non-technical stakeholders. This isn’t an approach I’ve used before, but my company and my coworker have a lot of experience delivering this type of model. There are a number of models out there and certain methods are very hyped right now, but a lot of times simple is still very effective and this probably works well with messy real world data.

My part of this process was to implement this model as a daily scoring system and for this we needed to rewrite the implementation. My company mostly uses SAS in a secure environment so it isn’t intended for automated systems that need to transfer data to and from outside servers. I used R to implement this system. This rewrite from R to SAS is unfortunate because it means a lot of duplicate effort and it complicates quality control. I needed to reconcile differences between SAS and R, rewrite our library functions in R, and test two systems instead of one. This led to quite a few bugs. Luckily the deciled scorecards actually make the modeling part easier than trying to find compatible implementations of model libraries. I can just output the decile ranges with scores to a file and create functions in R to read them. This also abstracts model variables from hardcoded values so I’ll need to rewrite less code to update the model.

I learned so much about coding R from this project. There were a number of libraries I needed to find to perform logging (futile.logger), unit testing (testthat), computationally efficient calculations (Rcpp), and more. I was constantly discovering ways to improve my code. I reached a place of comfort with R where I would be sitting on the bus and solutions to problems that stumped me the day before would pop in my head. I learned a million little things from this project rather than one big epiphany.

The structure of the system looks like this:

1. Load and clean data
2. Summarize data
3. Combine data and calculation from the combined file
4. Scoring
5. Reporting and talking to the outside world

I decided to structure the system to run from a main R file instead of another common architecture for analysis tasks, running discrete steps from a makefile. The benefit of the makefile approach is it increases reproducibility and intermediate datasets don’t need to be rebuilt if the input to that step is unchanged. These steps are very much like math functions. If you give it the same input it should have the same output. This works well when you need to backup an assertion from your analysis, but real world systems come with more requirements.

This project needed to work with external tools and databases which complicates the simplicity of the makefile approach. Because of requirements of decaying scores from a database the output of one section of code changes depending on how many times it’s run and when. This means the system must be robust to failures at this point and won’t necessarily have the same output every time it’s run. Additionally, this system gets fresh data from each input source every time it’s run so the lazy evaluation of the makefile steps is unnecessary. These issues are common in real world systems and software development in general.

I’ve gained a lot of knowledge and reusable code from this project so I look forward to using that to streamline development of similar systems. It’s so important to plan ahead both on your own and with stakeholders in a project like this because new requirements are guaranteed to pop up and you’ll run into limitations. Foresight comes with experience, but you can also avoid a lot of stress just by NOT blindly following the mantra of move fast and break things.

