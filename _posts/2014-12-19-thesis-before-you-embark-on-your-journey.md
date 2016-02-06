---
layout: post
date: 2014-12-19
title: 'Thesis Tutorials #4: Some opinionated tips'
subtitle: 'Do or do not do, there is no try'
---

*This post is part of my thesis tutorials series. You can find the rest of the posts [here](http://bkkkk.github.io/thesis.html)*

# Latex tips, big and small

As you work with Latex you'll pick up on a few themes. The basic functionality of latex covers a large number of use cases but not everything. As a result you'll rack up a lot of packages very quickly. Unfortunately the documentation for most packages leave much to be desired, and the latest and greatest approaches to certain things are not always documented clearly. This leads to you having to navigate a series of tex.se posts and ancient and cryptic sites before you find an easy and stable way of doing one thing.

# For the journey ahead

This is an unfortunate journey that you will have to take if you want to typeset a good looking document. You will learn a lot from it and that's something you will never lose. What follows is a list of a few nuggets I picked up along the way, this list will get larger as I come up with things.

**Use the chktex linter**

I've mentioned this before, but this application produces a list of "errors" found in your tex files. These are not errors that will prevent the document compiling, but rather misuses of commands that might cause things not to typeset correctly.

**Use the l2tabu package**

This package codifies a few [LaTeX taboos](http://anorien.csc.warwick.ac.uk/mirrors/CTAN/info/l2tabu/english/l2tabuen.pdf) and will warn you while compiling if these are detected. Use personal judgement here.

**Do not use \$\$...\$\$**

The $$...$$ is a TeX primitive and can sometimes produce incorrect spacing. Instead use the more modern Latex command \\[...\\] or \begin{equation} from the package **amsmath**. I use the _equation_ environment as it feels cleaner to me and is more consistent with uses of the environment _align_.

**Do not use $...$**

Similar to the above, use \\(...\\) instead. Load the **fixltx2e** package to fix a handful of issues including some rendering quirk with \\(...\\).

**Define new commands for concepts**

Define new commands to codify concepts and improve readability of code. For example, I've defined a new command to represent missing energy:

```latex
\newcommand{\met}{\ensuremath{E^{\textrm{miss}}_{\textrm{T}}}}
```

This means that I avoid having to repeat code, I don't have to make a decision right now about how I want to represent this concept, increases the consistency of my test, and more importantly it makes the text much more readable without compiling.

```latex
A large amount of \met\ is expected
in a final state SUSY signature.
```

**Use version control**

If you ever code anything that matters you have to put it into some kind of version control and have it backed up regularly to an off-site backup. I use git and github. If you only have a single copy of your thesis and you're not updating your backup constantly, you deserve whatever horrible thing happens to you.

**Unicode support (Get yer umlaut on!)**

Latex unfortunately does not support utf8 by default, so you cannot directly type names with umlauts into your latex code. Luckily with only a couple of lines of code you can get support for those lovely foreign names.

```latex
\usepackage[T1]{fontenc}
\usepackage[latin1]{inputenc}
\usepackage{lmodern}
```

What these packages do has been explained by more capable people than me on [tex.se](http://tex.stackexchange.com/questions/44694/fontenc-vs-inputenc). The **lmodern** package loads the font _Latin Modern_, which is a different font family to the latex default _Computer Modern_. If you just want to use cm simply load the fontenc and inputenc packages and the cm-super font, with additional glyphs, should load automatically.

# Managing your bibliography

Bibliographies are handled by roughly two components, first the external application, traditionally bibtex which processes the bibliography **.bib** file and a package like **natbib** which handles the formatting of the citations in your document.

To keep things short, use the **biblatex** package instead of something like natbib and process your .bib files with **biber** instead. Using these two newer tools provide a bunch of benefits such as unicode support, easy customization using latex methods, and support for additional fields. This is provided to you at zero cost since .bib written for bibtex are compatible with biber (the reverse is not necessarily true). The syntax for biblatex is simple:

```latex
\usepackage[sorting=none,backend=biber]{biblatex}
\addbibresource{Thesis.bib}

Many of its parameters have also been measured 
with great precision e.g.\ the electron magnetic
moment $g$ is known to \num{e-13}~\cite{Theory:AwesomeSM}.

\printbibliography
```

Here I specifically define biber as the backend and I use no sorting since I want the citations sorting in the order they appear in the text. When you compile your code simply run the following commands:

{% highlight bash %}
pdflatex Thesis.tex
biber Thesis.bib
pdflatex Thesis.tex
pdflatex Thesis.tex
```

**Sources**

- [LaTeX taboos](http://anorien.csc.warwick.ac.uk/mirrors/CTAN/info/l2tabu/english/l2tabuen.pdf)
- [Are \( and \) preferable to dollar signs for math mode?](http://tex.stackexchange.com/questions/510/are-and-preferable-to-dollar-signs-for-math-mode)
- [bibtex vs. biber and biblatex vs. natbib](http://tex.stackexchange.com/questions/25701/bibtex-vs-biber-and-biblatex-vs-natbib)
