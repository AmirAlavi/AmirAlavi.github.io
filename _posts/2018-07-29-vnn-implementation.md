---
layout: post
title: "scQuery: Using a VNN architecture"
date: 2018-07-29
github_comments_issueid: "3"
---

You may have heard of Visible Neural Network (VNN) architectures. These have been introduced and used by computational biologists to enable interpretation of the learned parameters of a neural network [<a href="#citation_1">1</a>]. These networks are usually constructed by using some sort of prior biological knowledge, and transforming that knowledge into a set of connections between neurons in different layers. For example, in the Lin et al. paper, they use protein-protein and protein-DNA interactions as their prior knowledge [<a href="#citation_2">2</a>]. Since each of the input neurons is a gene, the neurons in the first hidden layer can be nodes representing groups of interacting gene products. Thus, if two proteins interact with each other, then the input neurons representing the genes for those two proteins will be connected to the same neuron in the first hidden layer.

We are also interested in these VNN architectures for dimensionality reduction in [scQuery](https://scquery.cs.cmu.edu/), and experiment heavily with our own flavors of VNNs (e.g. incorporating protein interactions or the Gene Ontology into architectures) [<a href="#citation_3">3</a>].

So, if you are familiar with the basic multi-layer preceptron (MLP) network, you might have the question, "what do you do with the genes that don't have any protein-protein or protein-DNA interactions?" The answer is: nothing. You don't connect them. Thus, if you think of the layers of an MLP as fully connected (or "densley" connected, as many APIs such as Keras will call these layers), then a VNN has "sparsely" connected layers; a nueron in layer \\(n\\) is not necessarily connected to all neurons in layer \\(n+1\\), it will be connected to a subset.

<img src="{{ "/assets/dense_vs_sparse.svg" | absolute_url }}" class="mx-auto d-block" alt="Densely connected vs sparsely connected neural network layers" max-width="100%"/>

Assuming we have the prior knowledge and already know how many layers we want, and which nodes should be wired up to each other, how do we define this architecture in code? There's no *de facto* way to do this, and no library makes it easy. The key idea is to realize that with the sparsely connected layers, it's as if you take the weight matrix \\(\boldsymbol{W}\\) of a traditional dense layer, and have fixed some of the values in the matrix to always be zero.

For example, Lin et al. relied on a neat (but also brittle) trick to implement this [<a href="#citation_2">2</a>]. Using Keras with the Theano backend, they could define the weight matrix as a sparse weight matrix and use the `structured_dot` function in Theano, through which only gradients w.r.t. non-zero elements in the sparse weight matrix are backpropagated. However, this would make your code dependent on Theano, which is no longer being developed.

To avoid coupling my code to a specific backend, I chose an alternative approach: I define a separate binary "adjacency matrix" which is the same shape as \\(\boldsymbol{W}\\) (input_dim, output_dim), but has a \\(1\\) if the neurons are connected or a \\(0\\) otherwise. Then, the forward propagation through the layer (preactivation) is now: \\[(\boldsymbol{A} * \boldsymbol{W})^T\boldsymbol{x} + \boldsymbol{b}\\] Where \\(\boldsymbol{x}\\) is the input, \\(\boldsymbol{b}\\) is the bias, and \\(\boldsymbol{W}\\) and \\(\boldsymbol{A}\\) are multiplied elementwise. When you implement this with any deep learning library, that library's automatic differentiation functions will correctly only route the gradients through the non-zero connections (the gradients for the zero elements in \\(\boldsymbol{A}\\) will be zero).

I've implemented this custom layer in Keras, and published it as a package on [PyPi](https://pypi.org/project/sparsely-connected-keras/). You can install it via `pip install sparsely-connected-keras`. You can use it exactly like a Dense layer, but instead of specifying the number of units in the layer, you pass in a numpy adjacency matrix (which also has the effect of specifying the number of units, given its shape). See the [GitHub repo](https://github.com/AmirAlavi/sparsely-connected-keras) for examples.

Yes, you technically don't have any less tensor operations or parameters, but you have less "effective" trainable parameters. The simplicity of the approach means that you can write it in a general way that doesn't depend on very specific tensor operations.

***
### References

<textarea id="bibtex_input" style="display:none;">
@article{ma2018using,
	customOrder={1},
	title={Using deep learning to model the hierarchical structure and function of a cell},
	author={Ma, Jianzhu and Yu, Michael Ku and Fong, Samson and Ono, Keiichiro and Sage, Eric and Demchak, Barry and Sharan, Roded and Ideker, Trey},
	journal={Nature methods},
	volume={15},
	number={4},
	pages={290},
	year={2018},
	publisher={Nature Publishing Group}
},

@article{doi:10.1093/nar/gkx681,
	customOrder={2},
	author = {Lin, Chieh and Jain, Siddhartha and Kim, Hannah and Bar-Joseph, Ziv},
	title = {Using neural networks for reducing the dimensions of single-cell RNA-Seq data},
	journal = {Nucleic Acids Research},
	volume = {45},
	number = {17},
	pages = {e156},
	year = {2017},
	doi = {10.1093/nar/gkx681},
	URL = {http://dx.doi.org/10.1093/nar/gkx681},
	eprint = {/oup/backfile/content_public/journal/nar/45/17/10.1093_nar_gkx681/3/gkx681.pdf}
},

@article{alavi2018scquery,
	customOrder={3},
	title={scQuery: a web server for comparative analysis of single-cell RNA-seq data},
    	author={Alavi, Amir and Ruffalo, Matthew and Parvangada, Aiyappa and Huang, Zhilin and Bar-Joseph, Ziv},
      	journal={bioRxiv},
        pages={323238},
	year={2018},
	publisher={Cold Spring Harbor Laboratory}
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