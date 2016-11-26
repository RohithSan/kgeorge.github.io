---
layout: post
title: "Ongoing Deep Learning Projects"
description: ""
category: 
tags: []
---
{% include JB/setup %}
![image-classification using pretrained vgg16]({{ site.url }}/assets/content/dpl_imggrid_demo.png)
We have a few deeplearning projects encapsulated in jupyter ipython notebooks in the the notebooks directory of this [github repository](https://github.com/kgeorge/kgeorge_dpl)
The code written is mainly by us, But we have shown credit where ever we have used code from other repositories.
We have used tensorflow and jupyter ipython notebooks with ipywidgets installed




### github reference
[github repository](https://github.com/kgeorge/kgeorge_dpl)


### requirements
We require [jupyter ipywidgets](https://github.com/ipython/ipywidgets) to be installed before running these notebooks. [ipywidget](https://github.com/ipython/ipywidgets) is a small add-on to [jupyter](http://jupyter.org/) technology. You can easily pip install this component once you have installed [jupyter](http://jupyter.org/).

*Before starting the server with '<code>jupyter notebook --ip=0.0.0.0</code>' we do the following command to ensure that ipywidgets are enabled in the notebook. '<code>jupyter nbextension enable --py widgetsnbextension</code>.'*


### notebooks

* [<code>notebooks/tf_mnist.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_mnist.ipynb) is a simple mnist project

* [<code>notebooks/tf_autoencoder.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_autoencoder.ipynb) is a simple autoencoder written for mnist data

* [<code>notebooks/tf_cifar.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_cifar.ipynb) is a simple training/validation framework for cifar-10 data

* [<code>notebooks/tf_cifar_optimized.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_cifar_optimized.ipynb)  attempts to get a higher validation accuracy than <code>notebooks/tf_cifar.ipynb</code> by means of a slew of optimizations.

* [<code>notebooks/tf_vgg16.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_vgg16.ipynb) loads a pre-trained vgg-16 weights, build a model and test random images taken by us


### utility code
* [<code>notebooks/common/utils.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/common/utils.ipynb) is a utility notebook which can be imported by other notebooks. This utility notebook,
provides the following utilities

    *  <code>ProgressImageWidget</code>, is a custom ipywidget written for interactively displaying training data
    *  <code>Plotter</code>, is a class for adding channels and sample data to channels , which can be plotted as a png image
    *  <code>ImgGrid</code>, is a class for plotting a grid of images
    *  <code>ImgGridController</code>, is a higher level class for managing <code>ImgGrids</code> and corresponding <code>ProgressImageWidget</code>-s

### credits
* [<code>notebooks/common/load_notebooks.py</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/common/load_notebooks.py)  is reproduced from its implementation [here](http://jupyter-notebook.readthedocs.io/en/latest/examples/Notebook/Importing%20Notebooks.html)
* [<code>notebooks/common/imagenet_classes.py</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/common/imagenet_classes.py) is taken from [Davi Frossard's site](https://www.cs.toronto.edu/~frossard/post/vgg16)
* [<code>notebooks/tf_vgg16.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_vgg16.ipynb), much of the code for building the tensorflow model from vgg16.npy is adapted from [ MarvinTeichmann's tensorflow implementation of fc net](https://github.com/MarvinTeichmann/tensorflow-fcn)
* [<code>notebooks/tf_cifar_optimized.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_cifar_optimized.ipynb), we have used the code from [Jean Dut](https://github.com/jeandut/tensorflow-models) for GCN pre-processing of data
* [<code>notebooks/tf_cifar.ipynb</code>](https://github.com/kgeorge/kgeorge_dpl/blob/master/notebooks/tf_cifar.ipynb), owes its origins to [the official tensorflow cifar tutorial](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/models/image/cifar10)
