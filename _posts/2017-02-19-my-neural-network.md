---
layout: post
title: My Own Neural Network
description: I built my own neural network from scratch
---

{{page.title}}
==============

I’ve been learning neural networks from [George Hinton's class][hinton]. It’s so cool that anyone can learn from a pioneer in the field. As part of this process I wanted to write my own neural network implementation because I believe it reinforces a lot of the concepts and creates a deeper understanding. I got my starting code from [IAmTrask][iamtrask] and worked through the examples laid out in his blog post. My next task was to predict another simple nonlinear function, the sine function, to develop and test a more robust implementation of my neural network. Finally, I grabbed the most popular dataset for learning neural networks, MNIST, from [Kaggle][kaggle].

Let me say that debugging a neural network can be tricky. I spent half a day wondering why my weights were exploding in size before noticing that I should be dividing my learning rate by my batch size in mini-batch gradient descent. A good sanity check that your code is working and your network is learning is to test a small subset of the data. Your network should be able to overfit the data with almost 100% accuracy. For example, I used 10 cases of each digit to train the network.

<figure>
	<img src="/img/2017-02-19-my-neural-network/overfit.png">
	<figcaption>Success!</figcaption>
</figure>

### Is it learning? #

<figure>
	<img src="/img/2017-02-19-my-neural-network/old_batch_size.png">
	<figcaption>This learning curve doesn't look right</figcaption>
</figure>

At a certain point I noticed that my error rate was volatile and would climb slightly during training. Were my gradient updates not propagating correctly? Was my learning rate too high? Nope. An important aspect of mini-batch learning is that your batches need to contain a representative sample of your dataset. Otherwise the network will learn more nuanced characteristics of that particular mini-batch and your learning algorithm will take a very indirect route to that global minimum or even learn the wrong things. I had originally checked that the dataset was randomized then I picked a batch size of 100 (about 10 of each digit) and moved on. But a representative sample of the digits I was trying to predict isn’t a representative sample of the input space! Each digit could be written with a lot of variation and that’s the part I really needed to learn. Once I bumped the batch size up to 1000 the learning curves looked much better.

<figure>
	<img src="/img/2017-02-19-my-neural-network/new_batch_size.png">
	<figcaption>With a larger batch size</figcaption>
</figure>

It's difficult to pick the right network architecture and hyper parameters for a problem. One thing I’ve heard is that you get an intuition for it based on experience. That’s part of the reason I wanted to write my own neural network. Running MNIST through TensorFlow or Torch would have been super easy but not very illuminating. You get that experience and intuition by getting your hands dirty with the details. Another thing to do is read a lot of papers. People spend a lot of time figuring out what works better so that you can spend less time and stand on their shoulders. Many specific problems have pretty good solutions already worked out.

For image tasks I’ve heard it’s common practice to take the first couple layers from a well trained object recognition network, append it to your own network, and start retraining. That way you get the benefit of state of the art performance without all the work. See [transfer learning][transfer learning]. For example Caffe has [Model Zoo][model zoo] to facilitate transfer learning. It’s important to remember that even the experts don’t know what the best architecture and hyper parameters are. That’s why they have infrastructure to test many many variations of layers, layer sizes, and each hyper parameter picking the best combination.

### How do we know what the network is learning? #

Neural networks are black box learning algorithms. This is true and false. An important part of any system or algorithm is input and output. Is my network consistently less accurate at identifying 5’s? Then it must not be learning 5-ness very well. Does it confuse 8’s and 3’s often? In softmax look at the second or third guesses as well. (Note even though softmax gives probabilities for each class you can’t trust the relative difference of those probabilities because of how other variables can affect them. You shouldn’t draw conclusions from 80% / 20% vs 60% / 40%).

<figure>
	<img src="/img/2017-02-19-my-neural-network/neuron_samples.png">
	<figcaption>Training cases that activated neuron 43</figcaption>
</figure>

You can also look at the weights and activations of each node. This definitely requires more hands-on work, but you can look at what test cases activate a given node. Is this node activating on a lot of 3’s, 5’s, 6’s, and 8’s? Maybe it’s recognizing curves on the bottom half of the image. Do you notice a pattern in the weights? This is more doable with convolutional neural networks because they learn feature in a small region of an image making the results more human readable. 

<figure>
	<img src="/img/2017-02-19-my-neural-network/neuron_weights.png">
	<figcaption>Input weights to neuron 43 and sample training cases</figcaption>
</figure>

The flip side is that as networks grow larger and more complicated these methods get out of hand. It’s hard to interpret how combinations of neurons are working together. It may even be infeasible on certain types of neurons.

### Conclusions #

In the end I eked out a couple extra percentage points of accuracy by lowering the learning rate allowing the algorithm to settle into a better relative minimum in the topography. Who might I improve the network even more? Obviously CNN are quite good at image problems so that would be one. I also noticed there were plenty of neurons activating on 80+% of test cases. This could be because they were learning common characteristics of digits (pixel #512 is black), but that isn’t very useful if our goal is to differentiate between digits. Using dropout could improve the uniqueness of what each node is learning and create a better overall network.

I’m going to continue to add new features, layer types, cost functions and test it on different problems. I look forward to tackling some NLP datasets in the near future.



[iamtrask]: http://iamtrask.github.io/2015/07/12/basic-python-network/
[kaggle]: https://www.kaggle.com/c/digit-recognizer/data
[hinton]: https://www.coursera.org/learn/neural-networks/
[transfer learning]: http://cs231n.github.io/transfer-learning/
[model zoo]: http://caffe.berkeleyvision.org/model_zoo.html
