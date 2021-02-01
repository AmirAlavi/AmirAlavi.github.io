---
title: "Implementing tied-weights autoencoders in Keras"
tags: scQuery deep-learning keras
toc: true
---

Before we had ReLUs, batch normalization, and today's GPU training capabilities, denoising autoencoders (DAEs) [<a href="#references">1</a>] were a creative method by which we could train deeper nets.

Vincent et al. described "stacks" of these autoencoders that could be used to initialize the weights of a deep supervised neural network. The strategy they present is called "Greedy Layerwise Unsupervised Pretraining" and was explored by Bengio et al., using Restricted Boltzmann Machines (RBMs) as the "greedy module" [<a href="#references">2</a>], and later Vincent et al. used DAEs as their "greedy module" [<a href="#references">1</a>]. For every hidden layer of your supervised neural net, you would create a DAE with a single hidden layer of the same width, and train it in an unsupervised fashion. Then you could take these trained weights and initialize the weights of your supervised net with them. You could do this with the other layers, progressively building them up. The figure below illustrates this for the first hidden layer of a deep supervised net:

{% include figure image_path="/assets/images/glup.svg" alt="Greedy layerwise unsupervised pretraining using stacks of DAEs" %}

In practice, people usually use something called "tied weights." This just means parameter sharing. Since the encoding and decoding layers of the DAE mirror each other in structure, you can share parameters between them so you just learn one set of weights. The decode weights are the transpose of the encode weights (red parameter in the figure above). This is often preferred over learning separate weights for both phases because (a) you reduce the number of parameters and can train faster and (b) it is seen as a form of regularization that leads to better performance in practice.

When it comes to implementing a DAE in Keras, it's very straightforward to build up any arbitrary DAE using the built-in Dense or CNN layer classes. However, making it so that the parameters are shared (tied-weights) is not as easy. You need a way to share this information across these various layers.

A straightforward solution is to write a custom Layer class, which contains the entire DAE in this one layer. This way, we control the forward propagation and can point to the same weight matrices when appropriate. We use greedy layerwise pretraining to pretrain our neural networks for scQuery, and I have implemented tied-weights autoencoders exactly as described: as a single layer called `DenseLayerAutoencoder` (a derived class of the built-in `Dense` class).

<script src="https://gistit-minhhh.appspot.com/https://github.com/AmirAlavi/tied-autoencoder-keras/blob/master/tests/test_dense_autoencoder.py?footer=minimal&slice=5"></script>

To use `DenseLayerAutoencoder`, you call its constructor in exactly the same way as you would for `Dense`, but instead of passing in a `units` argument, you pass in a `layer_sizes` argument which is just a python list of the number of units that you want in each of your encoder layers (it assumes that your autoencoder will have a symmetric architecture, so you only specify the encoder sizes). An example:

<script src="https://gistit-minhhh.appspot.com/https://github.com/AmirAlavi/tied-autoencoder-keras/blob/master/tests/test_dense_autoencoder.py?footer=minimal&slice=10:14"></script>

This constructs an autoencoder with an input layer (Keras's built-in `Input` layer) and single `DenseLayerAutoencoder` which is actually 5 hidden layers and the output layer all in the same layer (3 encoder layers of sizes 100, 50, and 20, followed by 2 decoder layers of widths 50 and 100, followed by the output of size 1000).

How does it work? It's actually very straightforward:

<script src="https://gistit-minhhh.appspot.com/https://github.com/AmirAlavi/tied-autoencoder-keras/blob/master/tied_autoencoder_keras/autoencoders.py?footer=minimal&slice=72:104"></script>

As with any Keras Layer, the `call` function defines the forward propagation of our layer, and I've broken it up into the encode and decode portions (useful if you want to use your autoencoder to just encode some data after training). You can see within each of `encode` or `decode` that the layer has a list of kernels and biases, but only one set that is shared for both the encode and decode. In `decode`, the list of kernels is traversed in the opposite direction (and each is also transposed) (`biases2` are separate biases for the decode phase, since from what I've seen, weight-tying usually only applies to kernels, not biases).

And that's all there is to it! `DenseLayerAutoencoder` is available to install as a package from [PyPi](https://pypi.org/project/tied-autoencoder-keras/) ([GitHub](https://github.com/AmirAlavi/tied-autoencoder-keras)).

## Definitions
**Autoencoders** are unuspervised neural networks, meaning they are not trained with label information. Rather, the net is trained to reproduce its input (reconstruction), and the objective is usually something like minimizing mean squared error (MSE) of the reconstructed example vs the input example. They usually pass the input through hidden layers of smaller width than the input (encoding), resulting in some compression.

**"Denoising"** refers to the fact that we usually add some random noise to the input before passing it through the autoencoder, but we still compare with the clean version. This makes it so that our autoencoder is trained to remove any noise in input images, and helps prevent overfitting to trivial solutions (learning the identity mapping).

## References
1. Vincent, Pascal, et al. (2010). "Stacked Denoising Autoencoders: Learning Useful Representations in a Deep Network with a Local Denoising Criterion." *Journal of Machine Learning Research.* 11(110). <http://jmlr.org/papers/v11/vincent10a.html>
2. Bengio, Yoshua, et al. (2007). "Greedy layer-wise training of deep networks." *Advances in neural information processing systems.* 19. <https://proceedings.neurips.cc/paper/2006/file/5da713a690c067105aeb2fae32403405-Paper.pdf>
