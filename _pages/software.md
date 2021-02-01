---
layout: single
permalink: /software/
title: "Software"
author_profile: true
toc: true
---

## scipr
An extensible API for learning transformation functions to align batches of scRNA-seq data.

<a href="https://pypi.org/project/scipr/" class="btn btn--info">PyPI</a>
<a href="https://scipr.readthedocs.io" class="btn btn--info">Docs</a>
<a href="{{ site.baseurl }}{% post_url 2020-10-27-scipr %}" class="btn btn--info">Blog Post</a>

## sparsely-connected-keras
A new layer class for Keras models that allows static zero-weighted connections between neurons in adjacent layers (sparsely connecting layers instead of typical dense connections). Useful for VNN architectures.

<a href="https://pypi.org/project/sparsely-connected-keras/" class="btn btn--info">PyPI</a>
<a href="https://github.com/AmirAlavi/sparsely-connected-keras" class="btn btn--info">GitHub</a>
<a href="{{ site.baseurl }}{% post_url 2018-07-29-vnn-implementation %}" class="btn btn--info">Blog Post</a>

## tied-autoencoder-keras
Autoencoder layer classes (with tied encode and decode weights) for Keras.

<a href="https://pypi.org/project/tied-autoencoder-keras/" class="btn btn--info">PyPI</a>
<a href="https://github.com/AmirAlavi/tied-autoencoder-keras" class="btn btn--info">GitHub</a>
<a href="{{ site.baseurl }}{% post_url 2018-08-25-tied-autoencoders %}" class="btn btn--info">Blog Post</a>

## Open source contributions
### Keras
Added code to prevent programming-error. The bug would result in hard-to-decipher error messages for users. With this fix, we catch this error much earlier, and provide an error message that plainly states the issue. Merged into [Keras 2.2.0](https://github.com/keras-team/keras/releases/tag/2.2.0) ([PR](https://github.com/keras-team/keras/pull/10266))
### Armatus
Used detailed knowledge of C++11 STL to remove an unnecessary binary search through a std::map. ([PR](https://github.com/kingsfordgroup/armatus/pull/7))
### Apple WebKit (web engine used by Safari)
Discovered and fixed a bug with the storage of cookies from third party origins. Fixing this closes a potential cookie policy loophole, improving security. Resulted in the two bug reports below, and my patches for each:
- [Bugzilla report 1](https://bugs.webkit.org/show_bug.cgi?id=158967)
- [Bugzilla report 2](https://bugs.webkit.org/show_bug.cgi?id=159095)
