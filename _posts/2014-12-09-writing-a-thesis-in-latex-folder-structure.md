---
layout: post
date: 2014-12-10
title: 'Thesis Tutorials #2: Folder structure'
subtitle: 'Ducks in some linear representation'
comments: true
---

*This post is part of my thesis tutorials series. You can find the rest of the posts [here](http://bkkkk.github.io/thesis.html)*

# Working with large latex projects

A thesis is a giant document with many moving parts. You'll have multiple chapters, appendices, Tikz diagrams, PDF plots, Feynmf diagrams, bibliography, and a lot of words. Thankfully latex provides you with a few ways of organizing all of these.

This is what my folder looks like, I've omitted a bunch of files and folders to keep things brief:

{% highlight text %}
  .
  ├── BoilerPlate
  │   ├── Abstract.tex
  │   ├── Acknowledgements.tex
  │   ├── Conclusions.tex
  │   ├── Declaration.tex
  │   ├── PageTitle.tex
  │   ├── Preface.tex
  │   └── Resources
  ├── MacroDefs.sty
  ├── PartDetector
  │   ├── CovMatrix.tex
  │   ├── Detector.tex
  │   ├── DetectorElectronID.tex
  │   ├── DetectorMuonID.tex
  │   ├── Diagrams
  │   └── Plots
  ├── PartIntroduction
  │   └── Introduction.tex
  ├── PartObjectSelection
  │   └── ObjectSelection.tex
  ├── PartTheory
  │   ├── Diagrams
  │   └── Theory.tex
  ├── PartTopQuark
  │   ├── Diagrams
  │   ├── Plots
  │   └── TopQuark.tex
  ├── Thesis.bib
  ├── Thesis.pdf
  ├── Thesis.sublime-project
  ├── Thesis.sublime-workspace
  └── Thesis.tex
{% endhighlight %}

The main file **Thesis.tex** contains the backbone of the thesis. It includes packages, defines page styles, and adds bibliography resources (read .bib files), it does not however include any content. I very rarely worked in this file, only when adding packages or making changes to pagination.

Also in the root directory I have a directory for each chapter included in the thesis such as PartCalibration, PartIntroduction, and so on. In each of these folders I will have the main tex file for the chapter and a few resources folders (plots, diagrams, etc...). The tex files are grouped with the resources that they include.

Below is an excerpt from my Thesis.tex file, I've omitted the packages and page style setup.

{% highlight latex %}
\begin{document}
\include{BoilerPlate/PageTitle}
\newpage
\include{BoilerPlate/Declaration}
\newpage
\include{PartAbstract/Abstract}
\include{BoilerPlate/Acknowledgements}
\include{Preface.tex}
\tableofcontents
\listoffigures
\listoftables
\newpage
\include{PartIntroduction/Introduction}
\include{PartTheory/Theory}
\include{PartTopQuark/TopQuark}
\include{PartDetector/Detector}

%% More chapters

\begin{appendices}
\include{PartDetector/DetectorElectronID}
\include{PartDetector/DetectorMuonID}

%% More appendices

\end{appendices}
\newpage
\printbibliography
\end{document}
{% endhighlight %}

The main thing to note here are the calls to **\include{SomeTexFile}** which includes the tex file for each chapter. This structure allows you to separate the setup from the content, limits the damage if a file gets corrupted, and makes navigation across your thesis much simpler.

Nerdy note: Here we use **\include** and not **\input** since it is faster and we want the page breaks before and after each chapter. There is some explanation of the difference between \include and \input on [StackOverflow](http://tex.stackexchange.com/questions/246/when-should-i-use-input-vs-include).

I have all my Tikz diagrams and feynmf diagrams is separate files, these then get added with the \input command to avoid having diagram code cluttering the text.
That is it for now, next I will look at a few of the packages I use in more detail.


