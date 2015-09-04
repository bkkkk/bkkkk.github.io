---
title: 'Thesis Tutorials #1: Thesis 101'
subtitle: 'Back to basics'
date: 2014-12-09
---

# Thesis Tutorials #1: Thesis 101

*This post is part of my thesis tutorials series. You can find the rest of the posts [here](http://bkkkk.github.io/thesis.html)*

# The basics

This series will assume you have some basic understanding of LaTeX and have written and compiled a document in Latex before. If you haven't, take a look at this [Latex Wikibook](http://en.wikibooks.org/wiki/LaTeX). It's very readable with some nice illustrative examples and covers a range of techniques at different experience levels.

I am going to have a lot of particle physics specific information here, but most of it will be useful in other disciplines.

I assume that you know how and are able to install latex packages, and that you have an editor that you are comfortable with. If you're interested in my setup check it out below, if you're happy with what you've got just move on to the next post about structuring your latex project.

## My setup

I am working on a Mac running Yosemite. My editor is Sublime Text 3 loaded with a bunch of packages including Latex-specific ones. I might make another post about ST3 and what packages I found useful later, but here are a few that I used VERY heavily in my thesis writing. I am assuming that you have the Package Control package installed on ST3 as it makes installing packages WAY easier.

### Latexing

N.B. Download Latexing from the official site, I found that the Package Control version is either old or broken.

The most useful package for Latex editing is [Latexing](http://www.latexing.com). It's a real must-have for any large latex project. Amongst its features are: auto-completion of latex commands, citations, and cross-references. Comes with a solid and extendable build system so you can compile your entire thesis or even parts of it like Tikz diagrams (more on this later), chapters, and feynmf diagrams (more on this later too). Keep in mind however that Latexing does have a nag screen but you can dismiss it with escape and continue using it. You might also encounter LatexTools I tried both and found Latexing far more useful and powerful.

### SublimeLinter and SublimeLinter-chktex

A linter is an application that parses your code and warns you about style or functional errors you might have made. There are three components to enabling the linter for Latex: SublimeLinter, which acts as the interface between linter packages and ST; the linter extension, in this case SublimeLinter-chktex; and finally the linter itself which is a command-lin application, in this case chktex. You can install SublimeLinter and SublimeLinter-chktex from Package Control. You can usually get the linter itself from your Linux/Mac package manager; I used [homebrew](http://brew.sh) on my Mac.

That's it for my basic writing setup in SublimeText 3. I will introduce some other tools in a later post. Checkout the next post for some handy tips on structuring your thesis folder.
