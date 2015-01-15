---
layout: post
title: Thesis - Making pretty (good) Feynman diagrams!
category: Thesis
tags: [thesis,latex,feynman]
published: True
sharing: true
comments: true
---

So here we are, the Feynman diagram post. I've been dreading this day because to be honest, the Feynman diagram packages in Latex are - well - rubbish. Unfortunately I haven't found anything that comes close to producing the quality of diagram I want. There are dedicated applications such as [JaxoDraw](http://jaxodraw.sourceforge.net), but again they are clunky, difficult to use, and in my experience quite unstable. It's also disruptive to change contexts from writing thesis using the keyboard to making diagrams with a mouse. In the end I just bashed my head against the wall and tried to wrangle FeynMF (or its successor FeynMP) and do everything in the latex editor.

I'm not going 

## A package by a different name

So first forget using the *feynmf* package, it is outdated and difficult to use. There is also *feynmp* which uses a different backend to produce the diagrams but requires a bit of scaffolding code that tells pdflatex how to build the diagrams. This issue has now been addressed by yet another package *feynmp-auto* just import this package into your latex file and run pdflatex twice[^build-chain].

## What I've learnt

I don't want to repeat any basic information about making diag

------

**Footnotes**

[^build-chain]: You'll probably need to run biber/biblatex as well to update citations. 
