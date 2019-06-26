---
layout: post
title: "A LaTeX template and build code for practical journal submissions"
date: 2019-06-25
github_comments_issueid: "7"
tags: misc
---

So you're submitting a manuscript to a journal?

You've got your fancy LaTeX project all set up nice, the way you like? Separate main, supplement, and bibliography? Cross-refs between them all?

Oh what's that? You now need to submit a single compilable TeX file for the journal submission?...ugh.

...Annoying would be a generous description of the situation. Rather than manually copying the bib into the main.tex file, and hardening cross-refs between the main.tex and supp.tex, I created a (hacky) solution to generate the single submission file for me on-demand, so that I can still use Overleaf, version control, and modular tex files in development.

The solution is basically a set of tex template files which you fill out, and a Makefile which generates the submission files via some python scripts. Check it out [here](https://github.com/AmirAlavi/nature-tex). The repo is arrogantly named `nature-tex`.

Once you've cloned the repo, you can simply run `$ make` and you'll get a "submission" folder which contains the final files you can submit to the journal.

You can edit the template tex files to your liking and run `$ make` again to generate the files again. If for some reason the compiler runs into trouble or files aren't updating, you can try `$ make clean` and hopefully that fixes any issues. **More detailed documentation is in the repo.**

Good luck with your submissions!