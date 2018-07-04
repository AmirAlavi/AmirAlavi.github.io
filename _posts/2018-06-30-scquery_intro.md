---
layout: post
title: "scQuery: Introduction"
date: 2018-06-30
github_comments_issueid: "2"
---

[https://scquery.cs.cmu.edu/](https://scquery.cs.cmu.edu/)

We (myself and coauthors) recently released a manuscript on our most recent research work titled "scQuery: a web server for comparative analysis of single-cell RNA-seq data" (available on bioRxiv [here](https://doi.org/10.1101/323238), currently being peer-reviewed).

This project took us about one year to get to this point, and we still have much work to do. However, we felt that the methods, results, and supporting web site had reached a state where we could begin to share our work. While working on scQuery, I've learned quite a lot, and I'd like to share some pointers/stories about the methods and tools that I've encountered. This post is intented to be the first in a series where I will discuss each of these in more detail, so stay tuned!

A few brief notes about scQuery. There are two large aspects of the work: the data (collection and integration), and our large-scale analysis (machine learning techniques):

- The data:
  - We've gathered all publicly available M. musculus single-cell RNAseq data and re-processed them all in a uniform manner, to enable us and others to compare previous studies.
  - We also annotated each cell from these studies with its cell type. The labelings we use are classes (terms) from the Cell Ontology.
  - The above are (mostly) automated, and we do them bi-weekly to gether new data.
  - We make all of this re-processed data available for [download](https://scquery.cs.cmu.edu/processed_data/) on the web server.
- Large-scale analysis:
  - We use supervised neural networks to reduce the dimensions of the our data from ~20,000 genes to 100 dimensions
  - This reduced dimension representation, along with approximate nearest neighbors tools, allow us to do efficient storage and retrieval of query cells to all cells we have in our database.
  - We applied our technique to a held-out set of cells, and showed the pipeline can reveal biologically-relevant cell types and important genes.

A few weeks after we posted our pre-print, the European Bioinformatics Institute (EMBL-EBI) released a web database called "Expression Atlas," which is similar in spirit to the data-collection piece of our work, though Expression Atlas has reprocessed data for more species than just mouse. We are quite excited by this, as having a large consortium working in this area is a sign that many are interested in this work!