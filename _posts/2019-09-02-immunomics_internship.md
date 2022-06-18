---
title: "The adaptive immune system: the body's Guest Book"
tags: comp-bio
mathjax: true
toc: true
---

In Summer 2019, I was a research intern at Microsoft Research's [Immunomics team](https://www.microsoft.com/en-us/research/project/immunomics/) in Redmond, WA. Working under Jonathan Carlson, the project was a collaboration with a Seattle-area biotech company called Adaptive Biotechnologies. The two companies were striving towards building something called the ["Antigen Map."](https://www.adaptivebiotech.com/technology-partners/) The goal of this blog post is to give you a high-level understanding of the computational biology problems in this setting.
{: .notice}

Do you remember going to a hotel or a tourist destination and seeing a book that everyone would sign, usually their name, the date, and a little message, like “Amir was here!” or “Greetings from Michigan!”? They’re sometimes called **Guest Books**, or Autograph Books.

Turns out, the **Adaptive Immune System** can be viewed as a Guest Book where foreign pathogens (e.g. bacteria and viruses) effectively imprint their signatures. Allow me to elaborate.

### The two immune responses, a primer
Humans have two immune responses. The innate immune response, and the slower adaptive immune response. The innate response is the body’s immediate, non-specific response to infection. It’s kind of like a duck-tape solution. It’s general, quick, and most of the time, it’s enough to get the job done. Every moment, your innate response is protecting you from getting sick by using its components, including your protective barriers like skin, mucus membranes, and most importantly, via inflammation by attacking the site of infiltration with immune cells that engulf and neutralize foreign substances.

As useful as the innate response is, it’s a little unsophisticated, and things inevitably slip past it. Now, you’re infected. What’s your body gonna do? Adaptive Immune response to the rescue! This system creates a tailor-made response, just for this specific pathogen that has taken a hold of your body, and builds up the components to not only clear it out, but to also provide you immunity from future infection. These infections leave behind their signatures in the Guest Book, and next time, the immune system can just look up how it dealt with it before, and protect you from getting sick.

### Immune memory: the Guest Book
When pathogens such as bacteria or a virus enter the body, some of them are broken down into little bits and pieces of their proteins that make them up. These little pieces are antigens, and each pathogen can have unique antigens that originate from them. The adaptive immune response’s job is to recognize these antigens, and raise the alarm, activating specialized cells that can specifically target the pathogens that are the source of the antigens.

The existence of these specialized cells, which contain the right receptors that bind to that that particular antigen, is indeed the “signature” I’ve been talking about. When your immune system finds the right receptor for an antigen, it begins the process of activating that cell, and replicating to produce more and more of it, to mount a full response. The result of this process is that a cohort of those cells will remain in your body, even after the infection is cleared, ready in case they are ever needed in the future.

### The signature factory
How does the body produce **receptors** that can recognize the vast multitude of possible antigens (e.g. a newly mutated virus)? It turns out that the immune cells in our body have an internal randomization scheme, causing each and every one to have a unique receptor. The underlying mechanism of this is recombination (mixing and matching different DNA sequences to get many many different final receptor sequences). Some estimates put the number of different receptor sequences in an individual at [$$10^{13}$$](https://doi.org/10.1098/rstb.2014.0291). These all exist at some baseline, random frequency. However, the ones that happen to fit pathogens that you encounter are “activated” and cloned over and over, increasing their portion of the receptor population in your body. 

### Reading the guest book
Adaptive Biotechnologies has an [ImmunoSEQ technology](https://www.immunoseq.com) that allows you to assay the receptor sequences in a blood sample. This gives you a partial snapshot of an individual’s “repertoire” of receptors. With this data in hand, if we know the mapping between receptors and the antigens (and thus the pathogens) that they bind, we can effectively find out what diseases or infections that person has encountered in the past, or is currently fighting.

**And that has huge implications.**

With the right assays and mappings, we can in theory build a tool to diagnose almost any conceivable disease. A “universal diagnostic”!

### Examples of computational biology tasks for adaptive immune system data
- Build this mapping between antigens and their receptors (the Antigen Map Project)
- Build a machine learning classifier to diagnose a disease given a snapshot of the repertoire from an individual (see this paper for one approach)
- Deal with signal sparsity: often just a handful of receptor sequences found in the hundreds of thousands in a snapshot will actually bind to a particular disease. How can you pick up this sparse signal?
- Representing a person’s immune repertoire as a point in a low-dimensional coordinate space (as opposed to a “bag” of one-hot encoded sequences)
