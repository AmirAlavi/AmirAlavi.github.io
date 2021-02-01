---
title: "Using a VNN architecture"
tags: scQuery deep-learning keras
mathjax: true
toc: true
---

You may have heard of Visible Neural Network (VNN) architectures. These have been introduced and used by computational biologists to enable interpretation of the learned parameters of a neural network [<a href="#references">1</a>]. These networks are usually constructed by using some sort of prior biological knowledge, and transforming that knowledge into a set of connections between neurons in different layers. For example, in the Lin et al. paper, they use protein-protein and protein-DNA interactions as their prior knowledge [<a href="#references">2</a>]. Since each of the input neurons is a gene, the neurons in the first hidden layer can be nodes representing groups of interacting gene products. Thus, if two proteins interact with each other, then the input neurons representing the genes for those two proteins will be connected to the same neuron in the first hidden layer.

We are also interested in these VNN architectures for dimensionality reduction in [scQuery](https://scquery.cs.cmu.edu/), and experiment heavily with our own flavors of VNNs (e.g. incorporating protein interactions or the Gene Ontology into architectures) [<a href="#references">3</a>].

So, if you are familiar with the basic multi-layer preceptron (MLP) network, you might have the question, "what do you do with the genes that don't have any protein-protein or protein-DNA interactions?" The answer is: nothing. You don't connect them. Thus, if you think of the layers of an MLP as fully connected (or "densley" connected, as many APIs such as Keras will call these layers), then a VNN has "sparsely" connected layers; a nueron in layer \\(n\\) is not necessarily connected to all neurons in layer \\(n+1\\), it will be connected to a subset.

{% include figure image_path="/assets/images/dense_vs_sparse.svg" alt="Densely connected vs sparsely connected neural network layers" %}

Assuming we have the prior knowledge and already know how many layers we want, and which nodes should be wired up to each other, how do we define this architecture in code? There's no *de facto* way to do this, and no library makes it easy. The key idea is to realize that with the sparsely connected layers, it's as if you take the weight matrix \\(\boldsymbol{W}\\) of a traditional dense layer, and have fixed some of the values in the matrix to always be zero.

For example, Lin et al. relied on a neat (but also brittle) trick to implement this [<a href="#references">2</a>]. Using Keras with the Theano backend, they could define the weight matrix as a sparse weight matrix and use the `structured_dot` function in Theano, through which only gradients w.r.t. non-zero elements in the sparse weight matrix are backpropagated. However, this would make your code dependent on Theano, which is no longer being developed.

To avoid coupling my code to a specific backend, I chose an alternative approach: I define a separate binary "adjacency matrix" which is the same shape as \\(\boldsymbol{W}\\) (input_dim, output_dim), but has a \\(1\\) if the neurons are connected or a \\(0\\) otherwise. Then, the forward propagation through the layer (preactivation) is now: \\[(\boldsymbol{A} * \boldsymbol{W})^T\boldsymbol{x} + \boldsymbol{b}\\] Where \\(\boldsymbol{x}\\) is the input, \\(\boldsymbol{b}\\) is the bias, and \\(\boldsymbol{W}\\) and \\(\boldsymbol{A}\\) are multiplied elementwise. When you implement this with any deep learning library, that library's automatic differentiation functions will correctly only route the gradients through the non-zero connections (the gradients for the zero elements in \\(\boldsymbol{A}\\) will be zero).

I've implemented this custom layer in Keras, and published it as a package on [PyPi](https://pypi.org/project/sparsely-connected-keras/). You can install it via `pip install sparsely-connected-keras`. You can use it exactly like a Dense layer, but instead of specifying the number of units in the layer, you pass in a numpy adjacency matrix (which also has the effect of specifying the number of units, given its shape). See the [GitHub repo](https://github.com/AmirAlavi/sparsely-connected-keras) for examples.

Yes, you technically don't have any less tensor operations or parameters, but you have less "effective" trainable parameters. The simplicity of the approach means that you can write it in a general way that doesn't depend on very specific tensor operations.

***
## References
1. Ma, Jianzhu, et al. (2018). "Using deep learning to model the hierarchical structure and function of a cell." *Nature Methods.* 15(4). <https://doi.org/10.1038/nmeth.4627>
2. Lin, Chieh, et al. (2017). "Using neural networks for reducing the dimensions of single-cell RNA-Seq data." *Nucleic Acids Research.* 45(17). <https://doi.org/10.1093/nar/gkx681>
3. Alavi, Amir, et al. (2018). "A web server for comparative analysis of single-cell RNA-seq data." *Nature Communications.* 9(1). <https://doi.org/10.1038/s41467-018-07165-2>
