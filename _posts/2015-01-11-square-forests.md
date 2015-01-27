---
layout: post
title: The Forest Through the Trees
description: A discussion about forests in Scikit-Learn
---

{{page.title}}
==============

I was watching a presentation on [Square's ML pipeline][square]. If you aren't familiar they're a payment startup most well known for their iphone adapter that translates credit card numbers into audio signals to send through the iphone's audio jack. They talked specifically about their fraud detection pipeline. They chose to use an ensemble of decision trees, first using random forests then boosting trees. Decision trees are a very popular method in machine learning right now and they've even had success in [plenty of areas][kidney] outside of machine learning. I've come across many sources that cite ensembles of trees as an easy solution for a variety of different problems because of their ability handle missing data and varied data formats. Each branch in a decision tree is a boolean statement based on a single feature. They can also lower variance by adding more trees while not effecting bias. On the down side forests can take a long time to predict if the number of trees is too large because the data must be passed through every tree in the set. This is probably one of the reasons that the [XBox Kinect][kinect] is built upon decision trees to predict human shapes and actions; everything is preprocessed so they can use a large number of trees to squeeze out extra accuracy. Forests can have issues with outliers, but they are surprisingly robust to this in general.

I'm going to use SciKit-Learn to demonstrate some of the characteristics of decision trees. Now a lot of places that handle huge amounts of data don't use Python in a production environment when time is a factor, but it is still used to prototype since it's so fast and flexible to write Python code. Square for example tested it out initially, then a proprietary C++ library before finally rolling their own code in Java. That choice won't make sense for a lot of companies and problems, but in their case it's probably beneficial. I plan on exploring some of the Java ML solutions after working with Python a bit more because I'm sure they interface great with systems like Hadoop. I'll be using an old email spam classification data set from the [UCI Machine Learning Repository][data set] during this post. It contains about 6,000 emails summed up by 58 features:
<ul class='pretty'>
	<li>48 word frequencies - the percentage of words equal to 'x' in an email out of all words</li>
	<li>6 character frequencies - the percentage of characters equal to 'x' in an email out of all characters</li>
	<li>Average continuous length of capital letters</li>
	<li>Longest continuous length of capital letters</li>
	<li>Total capital letters in the email</li>
	<li>Spam or not spam classification</li>
</ul>

In SciKit-Learn the varieties of decision trees are packaged into *sklearn.ensemble*

###Random Forests

Random forests use many weak classifiers, in the form of small decision trees, to average out to a much more correct answer (It makes me think of random forests as being similar to monte carlo methods, but slightly more complex. Finding the boundary of a function through randomness). To build these trees it uses a random subset of the data and branches based on a random subset of features (usually *sqrt(num_features)*). 

A really cool advantage of forests and other ensemble methods is out-of-bag error. The algorithm keeps a list of the data points used to build a particular tree then uses data points not in that set to cross validate the tree. It can cover each tree this way making it unnecessary to create a separate cross validation set and give you more data to train with!


###Extremely Randomized Trees

They really could have done better naming this one. Oh well (The two hardest things in CS are naming things, cache invalidation and off-by-one errors). This technique is the same as random forests, but with one notable exception. When creating a branch in a tree, random forests will choose the most discriminant value, attempting to split the data points evenly between the two branches. Extremely randomized trees will choose this split point arbitrarily. This will increase the bias slightly and lower the variance even further.


###Gradient Boosting Classifier

The previous two techniques are known as bagging techniques whereas gradient boosting is a boosting technique. A bagging technique samples from the data set with replacement for each weak classifier, i.e. a decision tree. Boosting involves building a model sequentially. Gradient boosting in particular is generalized for use with any loss function.


###AdaBoost Classifier

When I first came across this I assumed it was referring to [Ada Lovelace][ada], unfortunately it just means adaptive. Each data point is assigned a weight of 1/N initially where N is the number of data points. Then every iteration of the algorithm the weights are adjusted so the incorrectly classified data points have a higher weight and the correct data points have lower weights. This effectively tells the algorithm to optimize for the edge cases.

###GridSearch

Scikit-Learn has an amazing tool called [GridSearch][grid] which tests out combinations of parameters to get the best cross-validation results. Unfortunately GridSearch doesn't play nice with the out-of-bag error estimator so we'll have to carve out a cross-validation set, but fortunately GridSearch takes care of this work for us. More trees should increase the prediction accuracy, but with diminishing returns. At some point it is just not worth the extra computation time. We'll max out our search at 1,000 trees.

{% highlight python linenos %}
from sklearn.ensemble import ExtraTreesClassifier, RandomForestClassifier, 
								AdaBoostClassifier, GradientBoostingClassifier
from sklearn.grid_search import GridSearchCV


rf_param_grid = {'n_estimators': [10, 30, 100, 300, 1000]}
boost_param_grid = {'n_estimators': [10, 30, 100, 300, 1000],
                    'max_depth': [2, 3, 4, 5],
                    'min_samples_leaf': [1, 2, 3]}
ada_param_grid = {'n_estimators': [10, 30, 100, 300, 1000],
                  'learning_rate': [0.1, 0.3, 1.0, 3.0]}

rf_est = RandomForestClassifier()
rf_gs_cv = GridSearchCV(rf_est, rf_param_grid).fit(Xtrain, Ytrain)
print(rf_gs_cv.best_score_, rf_gs_cv.best_params_)
print('\n')

boost_est = GradientBoostingClassifier()
boost_gs_cv = GridSearchCV(boost_est, boost_param_grid).fit(Xtrain, Ytrain)
print(boost_gs_cv.best_score_, boost_gs_cv.best_params_)
print('\n')

ada_est = AdaBoostClassifier()
ada_gs_cv = GridSearchCV(ada_est, ada_param_grid).fit(Xtrain, Ytrain)
print(ada_gs_cv.best_score_, ada_gs_cv.best_params_)
print('\n')
{% endhighlight %}

{% highlight pytb %}
(0.94393741851368973, {'n_estimators': 300})
(0.94915254237288138, {'n_estimators': 1000, 'max_depth': 3, 'min_samples_leaf': 1})
(0.94295958279009129, {'n_estimators': 300, 'learning_rate': 0.3})
{% endhighlight %}


###Parallelization of tree generation

Bagging techniques can benefit from parallelization because they are creating many independent trees whereas boosting techniques are iterative and cannot benefit as much from parallelization. Using the *n_jobs* parameter we can specify the number of CPU cores to use when generating trees. The default is *n_jobs=1* and I'll test it with *n_jobs=-1* which means use as many CPU cores as you can. I have an i7 so this should be a pretty good speedup, not linear because there is still overhead for the parallelization process, but still pretty good. As expected *ExtraTreesClassifier* is faster than *RandomForestClassifier* because it doesn't need to find the most discriminant value for each branch split. With *n_estimators=3000* the benefit was even better than double.

{% highlight python %}
%timeit clf_rf = RandomForestClassifier(n_estimators=1000, n_jobs=1).fit(Xtrain, Ytrain)
%timeit clf_rf = RandomForestClassifier(n_estimators=1000, n_jobs=-1).fit(Xtrain, Ytrain)
%timeit clf_et = ExtraTreesClassifier(n_estimators=1000, n_jobs=1).fit(Xtrain, Ytrain)
%timeit clf_et = ExtraTreesClassifier(n_estimators=1000, n_jobs=-1).fit(Xtrain, Ytrain)
{% endhighlight %}

{% highlight pytb %}
1 loops, best of 3: 10.6 s per loop #Random forest, n_jobs=1
1 loops, best of 3: 4.92 s per loop #Random forest, n_jobs=-1
1 loops, best of 3: 8.27 s per loop #Extremely randomized trees, n_jobs=1
1 loops, best of 3: 4.36 s per loop #Extremely randomized trees, n_jobs=-1
{% endhighlight %}

###Cross validation and error rates

{% highlight python linenos %}
clf_rf = RandomForestClassifier(n_estimators=rf_gs_cv.best_params_['n_estimators'])
scores = cross_val_score(clf, Xtrain, Ytrain)
print(scores.mean())

clf_et = ExtraTreesClassifier(n_estimators=rf_gs_cv.best_params_['n_estimators'])
scores = cross_val_score(clf, Xtrain, Ytrain)
print(scores.mean())

clf_ada = AdaBoostClassifier(n_estimators=ada_gs_cv.best_params_['n_estimators'],
                         learning_rate=ada_gs_cv.best_params_['learning_rate'])
scores = cross_val_score(clf, Xtrain, Ytrain)
print(scores.mean())

clf_boost = GradientBoostingClassifier(n_estimators=boost_gs_cv.best_params_['n_estimators'],
                                 max_depth=boost_gs_cv.best_params_['max_depth'],
                                 min_samples_leaf=boost_gs_cv.best_params_['min_samples_leaf'])
scores = cross_val_score(clf, Xtrain, Ytrain)
print(scores.mean())
{% endhighlight %}

{% highlight pytb %}
0.949476457651 #Random Forest
0.94882541723 #Extremely Randomized Trees
0.947521423438 #AdaBoost
0.947520466964 #Gradient Boosting
{% endhighlight %}


<figure class="half">
	<img src="/img/2015-1-11-forests/gradient_boost_error_rate.png">
	<img src="/img/2015-1-11-forests/adaboost_error_rate.png">
</figure>

Tada! The results of each of our methods. Disappointingly they are all about the same. This would be a great time to point out that it's important to figure out the right tool for the job and there are specific methods to figure what is limiting your algorithm's effectiveness, whether it be test set size, choosing the correct features, or changing learning rates. This a subject for an entire post, but we'll look at the data a little bit more to get insight into it.

Visualizing the test data using PCA (principal component analysis) shows that the data is relatively clustered together which could explain why increasing *n_estimators* doesn't improve the error rate by much. I still don't have a great intuition for PCA so this could also be misguided. Cut me some slack, I'm working on it.

<figure>
	<img src="/img/2015-1-11-forests/pca_test_data.png">
</figure>

###Feature Importance

One of the coolest advantages of using trees is being able to track feature importance. This tells us which features were most import in discerning spam from legitimate email. This is valuable information because it actually gives you insight into the analysis. Imagine you want to figure out when a customer is planning on leaving your service so you can entice them to stay. You run all the customer data through your ML pipeline and hooray! You end up saving the company from a lot of turnover. Unfortunately you still don't know WHY customers are choosing to leave so you can preempt the behavior. Feature importance to the rescue! In other algorithms such as neural networks this analysis is almost impossible. In fact some AI experts think that if we ever invent sentient AI we may not even understand how it works because the decision models will be too complex.

<figure class="half">
	<img src="/img/2015-1-11-forests/feature_importance_1.png">
	<img src="/img/2015-1-11-forests/feature_importance_2.png">
</figure>

Remember that these are the most import features for determining spam AND not spam so values like 'george' are very indicative of not spam. Not a lot of surprises here: capital letters and exclamation points are easy identifiers of spam while names like 'george' and 'hp' (the company) are definitely safe words. The least important features include random numbers. About half of the features are the same order of magnitude of importance and the large majority are within one order of magnitude. This tells us that emails need a lot of features to classify them as spam. There could be cases like the message, "HAPPY BIRTHDAY!!!" being mistaken for spam (debatable whether it isn't spam. You  may be onto something poor algorithm). Short emails could easily be susceptible to misclassification with the current feature set. This data set doesn't provide us with the email text only the features described above to summarize them so it's difficult for us to generate new features or analyze misclassification examples. In all likelihood we could probably squeeze out an extra percent of effectiveness with a lot more work, but this is about as good as it gets for now. Real spam classification algorithms have much larger feature sets and data sets while constantly evolving in a cat and mouse game.

As always, I hope you've learned something so feel free to email me your gratitude or if I did something stupid I'm happy to learn too.


[square]: https://www.youtube.com/watch?v=vpD5UG-tFsY
[kinect]: http://research.microsoft.com/pubs/145347/bodypartrecognition.pdf
[data set]: https://archive.ics.uci.edu/ml/datasets/Spambase
[grid]: http://scikit-learn.org/stable/modules/grid_search.html
[ada]: http://en.wikipedia.org/wiki/Ada_Lovelace
[kidney]: http://fastml.com/how-a-russian-mathematician-constructed-a-decision-tree-by-hand-to-solve-a-medical-problem/