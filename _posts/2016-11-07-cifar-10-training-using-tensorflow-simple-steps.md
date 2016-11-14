---
layout: post
title: "cifar 10 training using tensorflow, simple steps"
description: ""
category: 
tags: []
---
{% include JB/setup %}
##### cifar-10 training and prediction using tensorflow, simple stuff


We use [tensorflow](https://www.tensorflow.org/) to train a convolutional neural net (CNN) net to classify [cifar-10](https://www.cs.toronto.edu/~kriz/cifar.html) dataset.

Our initial implementation which is outlined in this [github page](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_cifar.ipynb) for a jupyter notebook implementation.
produced a validation accuracy of about 70%. But after doing the following we increased the  validation accuracy to close to 80%.

* global contrast normalization, For each image we take the entire HxWx3 set of numbers as a 1-d array and do mean normalization followed by division wrt L2-norm.

* batch normalization, We added batch normalization layers between convolution output and relu-s for the two convolution layers

* drop out, we added two dropout layers after the fully connected layers

* we added an exponential learning rate decay, of 0.96 , for every 5000 minibatches, with a starting learning rate of 0.1

As a result our validation accuracy increased to 80%

![fig-0, cifar-10 validation accuracy after optimizations]({{ site.url }}/assets/content/cifar-10-batches-py_acc_lr_p1_decay_p96_5000_gcn_s55_dropout.png)
