---
layout: post
title: "Implementing tied-weights autoencoders in Keras"
date: 2018-08-25
github_comments_issueid: "4"
tags: scQuery deep-learning keras
---

Before we had ReLUs, batch normalization, and today's GPU training capabilities, denoising autoencoders (DAEs) [<a href="#citation_1">1</a>] were a creative method by which we could train deeper nets.

Vincent et al. described "stacks" of these autoencoders that could be used to initialize the weights of a deep supervised neural network. The strategy they present is called "Greedy Layerwise Unsupervised Pretraining" and was explored by Bengio et al., using Restricted Boltzmann Machines (RBMs) as the "greedy module" [<a href="#citation_2">2</a>], and later Vincent et al. used DAEs as their "greedy module" [<a href="#citation_1">1</a>]. For every hidden layer of your supervised neural net, you would create a DAE with a single hidden layer of the same width, and train it in an unsupervised fashion. Then you could take these trained weights and initialize the weights of your supervised net with them. You could do this with the other layers, progressively building them up. The figure below illustrates this for the first hidden layer of a deep supervised net:

<img src="{{ "/assets/glup.svg" | absolute_url }}" alt="Greedy layerwise unsupervised pretraining using stacks of DAEs" width="80%" style="display:block; margin-left:auto; margin-right:auto;"/>

In practice, people usually use something called "tied weights." This just means parameter sharing. Since the encoding and decoding layers of the DAE mirror each other in structure, you can share parameters between them so you just learn one set of weights. The decode weights are the transpose of the encode weights (red parameter in the figure above). This is often preferred over learning separate weights for both phases because (a) you reduce the number of parameters and can train faster and (b) it is seen as a form of regularization that leads to better performance in practice.

When it comes to implementing a DAE in Keras, it's very straightforward to build up any arbitrary DAE using the built-in Dense or CNN layer classes. However, making it so that the parameters are shared (tied-weights) is not as easy. You need a way to share this information across these various layers.

A straightforward solution is to write a custom Layer class, which contains the entire DAE in this one layer. This way, we control the forward propagation and can point to the same weight matrices when appropriate. We use greedy layerwise pretraining to pretrain our neural networks for scQuery, and I have implemented tied-weights autoencoders exactly as described: as a single layer called `DenseLayerAutoencoder` (a derived class of the built-in `Dense` class).

<script src="http://gist-it.appspot.com/https://github.com/AmirAlavi/tied-autoencoder-keras/blob/master/tests/test_dense_autoencoder.py?footer=minimal&slice=5"></script>

To use `DenseLayerAutoencoder`, you call its constructor in exactly the same way as you would for `Dense`, but instead of passing in a `units` argument, you pass in a `layer_sizes` argument which is just a python list of the number of units that you want in each of your encoder layers (it assumes that your autoencoder will have a symmetric architecture, so you only specify the encoder sizes). An example:

<script src="http://gist-it.appspot.com/https://github.com/AmirAlavi/tied-autoencoder-keras/blob/master/tests/test_dense_autoencoder.py?footer=minimal&slice=10:14"></script>

This constructs an autoencoder with an input layer (Keras's built-in `Input` layer) and single `DenseLayerAutoencoder` which is actually 5 hidden layers and the output layer all in the same layer (3 encoder layers of sizes 100, 50, and 20, followed by 2 decoder layers of widths 50 and 100, followed by the output of size 1000).

How does it work? It's actually very straightforward:

<script src="http://gist-it.appspot.com/https://github.com/AmirAlavi/tied-autoencoder-keras/blob/master/tied_autoencoder_keras/autoencoders.py?footer=minimal&slice=68:90"></script>

As with any Keras Layer, the `call` function defines the forward propagation of our layer, and I've broken it up into the encode and decode portions (useful if you want to use your autoencoder to just encode some data after training). You can see within each of `encode` or `decode` that the layer has a list of kernels and biases, but only one set that is shared for both the encode and decode. In `decode`, the list of kernels is traversed in the opposite direction (and each is also transposed) (`biases2` are separate biases for the decode phase, since from what I've seen, weight-tying usually only applies to kernels, not biases).

And that's all there is to it! `DenseLayerAutoencoder` is available to install as a package from [PyPi](https://pypi.org/project/tied-autoencoder-keras/) ([GitHub](https://github.com/AmirAlavi/tied-autoencoder-keras)).

### Definitions
**Autoencoders** are unuspervised neural networks, meaning they are not trained with label information. Rather, the net is trained to reproduce its input (reconstruction), and the objective is usually something like minimizing mean squared error (MSE) of the reconstructed example vs the input example. They usually pass the input through hidden layers of smaller width than the input (encoding), resulting in some compression.

**"Denoising"** refers to the fact that we usually add some random noise to the input before passing it through the autoencoder, but we still compare with the clean version. This makes it so that our autoencoder is trained to remove any noise in input images, and helps prevent overfitting to trivial solutions (learning the identity mapping).

### References
<textarea id="bibtex_input" style="display:none;">
@article{vincent2010stacked,
	customOrder={1},
	title={Stacked denoising autoencoders: Learning useful representations in a deep network with a local denoising criterion},
	author={Vincent, Pascal and Larochelle, Hugo and Lajoie, Isabelle and Bengio, Yoshua and Manzagol, Pierre-Antoine},
	journal={Journal of machine learning research},
	volume={11},
	number={Dec},
	pages={3371--3408},
	year={2010}
},
@inproceedings{bengio2007greedy,
	customOrder={2},
	title={Greedy layer-wise training of deep networks},
	author={Bengio, Yoshua and Lamblin, Pascal and Popovici, Dan and Larochelle, Hugo},
	booktitle={Advances in neural information processing systems},
	pages={153--160},
	year={2007}
},
</textarea>

<div class="bibtex_template" style="display: none;">
    <ol id="citation_+customOrder+" class="bibtexVar" start="+customOrder+" extra="customOrder"> <li>
      <span class="if title">
            <span style="text-decoration: underline;" class="title"></span>,
      </span>
      <div class="if author">
        <span class="author"></span>
      </div>
      <div>
        <span class="if journal"><em><span class="journal"></span></em>,</span>
        <span class="if publisher"><em><span class="publisher"></span></em>,</span>
        <span class="if booktitle">In <em><span class="booktitle"></span></em>,</span>
        <span class="if address"><span class="address"></span>,</span>
        <span class="if month"><span class="month"></span>,</span>
        <span class="if year"><span class="year"></span>.</span>
        <span class="if note"><span class="note"></span></span>
        <a class="bibtexVar" role="button" data-toggle="collapse" href="#bib+BIBTEXKEY+" aria-expanded="false" aria-controls="bib+BIBTEXKEY+" extra="BIBTEXKEY">
	     [bib]
		</a>
      </div>
      <div class="bibtexVar collapse" id="bib+BIBTEXKEY+" extra="BIBTEXKEY">
      	     <div class="well">
	     	      <pre><span class="bibtexraw noread"></span></pre>
		      		   </div>
				     </div>
				       <div style="display:none"><span class="bibtextype"></span></div>
    </li></ol>
</div>
  
<div id="bibtex_display"></div>