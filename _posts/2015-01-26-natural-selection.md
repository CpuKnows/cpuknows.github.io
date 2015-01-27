---
layout: post
title: How Is Natural Selection So Effective?
description: They answer may be related to the structure of networks
---

{{page.title}}
==============

I came across [an article][evolution] discussing how the topology of biological networks helps evolution and it really got me thinking about neural networks. 

>*"Natural selection may explain the survival of the fittest, but it cannot explain the arrival of the fittest."* - Hugo de Vries

Often biological traits are the products of many genes, for example the Hox gene circuit which determines an organism's body plan. Are there going to be legs here, wings? How is the vertebrae going to form? For vertebrates there are around 40 different genes that factor into the circuit resulting in 10^700 circuit permutations! Evolving into a viable human hox gene circuit from one of our far off ancestors seems incredibly improbable.

The same ideas are present in the creations of protiens, essential elements of life. Protiens are formed by a sequence of amino acids which are then folded into a particular shape. The 20 different types of amino acids found in natural protiens can be in chains of 100 amino acids, which isn't even that large. The resulting number of permutations is a staggering 10^130. By scientists' calculations natural selection has only had the time to create 10^50 protiens in 4 billion year, many orders of magnitude smaller.

The key insight to unravel this paradox came from the study of RNA, another component of life with an incredible amount of permutations. Like protiens RNA contains a chain of genes folded into a particular shape. The sequence of genes is referred to as the genotype and the shape is called the phenotype. The genotype influences the resulting phenotype, but biolgist don't yet know how many of them pair together. One theory of the relation between genotypes and phenotypes was that similar RNA shapes would have similar gene sequences.

>Naively, you might expect RNAs with a similar shape, and thus presumably phenotype, to share a similar sequence, so that a map of the possible sequences—the sequence space, which can be represented as a many-dimensional space where each grid point corresponds to a particular sequence—is divided up into various “shape kingdoms” (See Not a Patch, a). But that wasn’t what Schuster found. Instead, RNAs with the same shape could vary very widely in sequence: You could get the same shape, and therefore potentially the same kind of catalytic function, from very different sequences. But these phenotypic siblings weren’t totally isolated from one another in distant parts of the sequence space. Instead, RNAs with the same kind of shape were connected in a network that weaved its way throughout pretty much the whole of sequence space. You could go from one sequence to another with the same shape (and thus much the same function) via a succession of small changes to the sequence, as if proceeding through a rail network station by station. Such changes are called neutral mutations, because they are neither adaptively beneficial nor detrimental. (In fact even if mutations are not strictly neutral but slightly decrease fitness, as many do, they can persist for a long time in a population as if they were quasi-neutral.)

>And this was the case for all the 1023 possible shapes. In other words, this RNA sequence space is threaded with that number of interwoven networks, all interlocking but rarely intersecting. This means that if we consider any given sequence at a particular point in the high-dimensional space, it has a huge number of neighboring sequences that have completely different shapes—and yet it can still mutate step by step into a very different sequence with a similar function, provided that it stays on the right track (See Not a Patch, b).

<figure>
	<img src="/img/2015-1-26-natural-selection/rna_space.png">
	<figcaption><a href="http://nautil.us/issue/20/creativity/the-strange-inevitability-of-evolution">Not a Patch - nautil.us/</a></figcaption>
</figure>

>This tells us two crucial things about the RNA sequence space. First, there are many, many possible sequences that will all serve the same function. If evolution is “searching” for that function by natural selection, it has an awful lot of viable solutions to choose from. Second, the space, while unthinkably vast and multi-dimensional, is navigable: You can change the genotype neutrally, without losing the all-important phenotype. So this is why the RNAs are evolvable at all: not because evolution has the time to sift through the impossibly large number of variations to find the ones that work, but because there are so many that do work, and they’re connected to one another.

Biologists have tested these network properties in protiens, bacteria like E. coli, and brewer's yeast. All of these subjects yielded many viable genetic solution even after alter a substantial portion of the gene sequence. This robustness allows a species popluation to explore many genetic possibilities without causing severe harm to individuals because many network configurations produce similar results. The best part is that scientists think this may not be a property of biological networks, but of any complex informational system. The same properties have held in complex networks of electronic components. This sounds like it could have a connection to deep artificial neural networks.

I also enjoy the podcast [Talking Machines][talkingmachines] by Katherine Gorman and Ryan Adams. This week's episode talked to Google researcher Ilya Sutskever from the University of Toronto. This rang a bell because I recently worked on an image recognition project using convolutional neural networks. A very important paper on this topic is "*[ImageNet Classification with Deep Convolutional Neural Networks][imagenet]*" from the University of Toronto and believe it or not he was one of the authors!

*"When you use supervised learning with large deep neural networks what you're really saying is give me the best possible neural network that can imitate this behavior."* It is worth pointing out that the algorithms used to train neural networks like a gradient decent with forward and backward propogation are incredible simple. *"It would take a smart student only an hour to understand how [gradient descent and propogation] works."* Even so we don't completely understand neural networks from a theoretical perspective.

>The neural network objective function is very non-convex and there are no mathematical guarantees about it's success. It is not magic per say, but it is poorly understood. We don't know why a very simple optimization works as well as it does.

Part of the theory behind multi-layer perceptrons (the precursor to more modern neural networks) is that there exists a general learning process in biological organisms that can be applied to many different areas. Experiments in rewiring the visual inputs to the auditory cortex in animals still allowed them to learn to see. Even though visual and auditory inputs have different properties a section of the brain evolved to take auditory information was able to adjust to the visual information. The robustness of the network structure could be a large factor as well.

Ryan responds, *"Are you guaranteed to find the global maximum or the global minimum for convex functions for this objective function? In no way is human intelligence the best possible intelligence and any engineered system that we have that is successful for airplains or cars or anything else. These aren't the best systems. We just want good systems."* This sounds similar to how biological networks can explore a solution space in which many pathways are similar and non-detrimental, in other words local minimum/maximum, while still looking for the global optimal solution.

<figure>
	<img src="/img/2015-1-26-natural-selection/convolution.png">
	<figcaption><a href="http://deeplearning.net/tutorial/lenet.html">DeepLearning.net/</a></figcaption>
</figure>

Perhaps the power of neural networks doesn't lay in the gradient descent algorithm, but in the network structure that allows for solutions to be found. We know that a single layer neural network can find a good [approximation][nncomputation] to any function and deeper networks can learn more complex relationships. A good example is image recognition. The input is individual pixel RGB values, so a single layer network can only learn about relationships to individual pixels. If an object is in the same position and orientation in an image the network could probably create a good approximation, but this isn't the case with real world data. Deep networks can determine relationships between areas of the image to extract higher order knowledge required to identify objects because of their network structure. 

There are some incredibly exciting ideas here and I look forward to how biological research will continue to open doors in AI research.

[evolution]: http://nautil.us/issue/20/creativity/the-strange-inevitability-of-evolution
[talkingmachines]: http://www.thetalkingmachines.com/blog/2015/1/15/machine-learning-and-magical-thinking
[imagenet]: http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf
[nncomputation]: http://neuralnetworksanddeeplearning.com/chap4.html