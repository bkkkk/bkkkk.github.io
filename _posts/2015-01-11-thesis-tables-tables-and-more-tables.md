---
layout: post
date: 2015-01-12
title: 'Thesis Tutorials #5: How to make tables'
subtitle: 'Not the IKEA kind'
comments: True
---

# Thesis Tutorials #5: How to make tables

*This post is part of my thesis tutorials series. You can find the rest of the posts [here](http://bkkkk.github.io/thesis.html)*

# Making better tables

The role of a table is to help you **convince the reader** of the point you are trying to make. Good tables make for happy examiners, and happy examiners give you PhDs so it's well worth pleasing them.

# To tab or not to tab

First, **do you need a table at all?**. For example, the efficiency of muon taggers as a function of the muon momentum fits perfectly in a table. However, the description of three muon reconstruction methods would be better in a list.

I would ask the following questions before building a table:

- **How relevant or interesting is the data?** Is it crucial to your argument or merely informative? If the data is not so important or interesting, really consider if you need it at all.
- **How much of it there is?** one, two, or hundred entries? If you have more than 100 entries to represent, a list is just not practical. There may also be a better, more concise way of making your point.
- **What type of data is it?** If the data contains many interconnected relations or groupings, then it's likely that a table is better suited.

A table requires a lot of scaffolding (a caption, headers, lines, etc...), takes a substantial amount of space in the thesis, and breaks up the text in an **intrusive** way. You must justify your use of tables.

I found it difficult to deal with event cuts in my thesis. If you're introducing cuts for the first time it might be useful to dedicate a paragraph describing each cut and its motivation. On the other hand if you're putting a list of relevant, but not crucial, cuts in an appendix perhaps a table is more appropriate and easier to read.

# What makes a table 'good'

The purpose of a table is to clearly and effectively convey meaningful relational/group data to your reader so as to support the arguments you are trying to make.

Take a good look at the tables you already have, or the data you are considering including in your thesis. Does it fit nicely with the text? Is it crucial to your argument? Would your thesis be worse without that data? **Avoid putting information in your thesis that is not directly supporting your arguments**. If something is relevant but not crucial you can always put it in an appendix and reference it from the text.

The reader should be able to understand the table, at a rudimentary level, without needing much or any of the main text. Look at the example below from the economist and see if you can understand what the numbers mean.

![The frightful future](/img/simple_table.gif)
<figcaption>Source: <a href="http://www.economist.com/node/18928863">The Economist</a></figcaption>

The table is presenting the total and per household amount of unfunded pension liabilities in each US city very clearly. Do I know what a pension liability is? Not a clue! Is it the purpose of the table to teach you that? Absolutely not. The purpose of the table is to present data which indicates a looming crisis. It does not contain any extraneous information and has clear headings and a caption. The data and headings are aligned in a natural and consistent way.

This is a simple example with little data and simple headings, nothing too crazy. How about something more complicated?

![A large table example from The Economist](/img/The-Economist-chart-of-IT-tech-giants.png)
<figcaption>Source: <a href="http://www.economist.com/node/16693547">The Economist</a></figcaption>

They are presenting general information for each of the tech companies, and some kind of indicator for their involvement in different tech markets. Do I know how that indicator was estimated? Nope, but from the table I can crudely compare each company's involvement against the others.

**Tip: Go online and look for table examples to inspire you. Whatever problem you're facing, it was probably already solved.**

# The actual interesting bit

Here are a few tips to get your tables in good shape. I use particle physics examples but the advice should be applicable to most disciplines.

**Use clear and descriptive headings**

Spend some time thinking of really clear and descriptive headings, these will often match items mentioned in the text. If you use the terms "MV1 efficiency" in your text, don't use "Multivariate tagging probability" in your table. Always specify the units in the header.

**Use the caption to clarify the data**

Provide more detail regarding what is in the table. Do not attempt to explain complex concepts; use the main text for that. You can clarify what samples the data was taken from or what components are included in the uncertainty.

**Do not use vertical lines or double lines**

The eyes find it easier to separate columns compared to rows. Use horizontal lines judiciously to improve clarity, separate different groups of cells and to delimit the top and bottom of the table.

**Use something better than \hline**

Use the package *booktabs* which provides you with the commands: \toprule, \midrule, \bottomrule, and \cmidrule. These lines varying weights, the \midrule is intended to separate headers from data or groups of data, and is thinner than \bottomrule and \toprule. \cmidrule allows you to create divisors which span a few columns. For example \cmidrule{1-3} produces a short \midrule between between columns 1 and 3.

**Align text consistently**

If in doubt align the text to the left. Make sure that numbers are aligned by their decimal point, you can use the S column defined in the *siunitx* package. Avoid having many different alignment schemes within a single column or table.

**Remove intercell spacing from the left and right edges**

This improves the readability of the table and puts emphasis on the table headers. Use the inter-column space command @{} with no arguments in your column definition:

{% highlight latex %}
\begin{tabular}{@{}lll@{}}
{% endhighlight %}

**Increase row separation**

This improves readability by helping the brain navigate through a row in the table. I defined a simple command which I then use to adjust the row separation length:

{% highlight latex %}
\newcommand{\ra}[1]{\renewcommand{\arraystretch}{#1}}
{% endhighlight %}

In a \table environment then invoke \ra{X} where X is a number. I used 1.2 for most of my tables and 1.3 for those larger tables, more cramped tables. The difference is quite subtle but very useful. Note that a factor of 1 means the row separation is unchanged from the default.

**Organize the data in a natural way**

For example, the sum of event counts is expected at the bottom of the table. Organize header (mass of sample, pseudorapidity of muons) values in ascending or descending order. Your reader shouldn't have to guess what things mean. The reader will likely start from the top-left corner and tend towards the bottom-right. Use that to your advantage; lead your reader through the table.

If you intend the reader to compare certain sets of values put them all in a single table. If there is a particularly important value, draw the readers attention to it with some colour or by making the text bold.

**Aid the reader through large tables**

Consider if a large table makes more sense as two tables. The data in a large table of efficiencies at pretag and tagged level is probably better presented as two separate tables. If there is too little data in the table to breakup or you think things should stay together consider using \midrule, repeated headers, and/or a mid-table title. Here is, what I think is a pretty decent example from my own thesis:

<img src="/img/large_table_example.png" style="width: 500px;" alt="Large table example" />

I iterated on the layout of this table a few times before arriving at the final version. Try things out see what works, remember to revisit tables towards the end to see how they fit with the text and the rest of the tables/figures around them.

**Group columns where appropriate**

You can use inter-column spacing and/or spanning cells to group together columns. The inter-column spacing command @{} allows you to adjust what goes in the spaces between columns. This can be a smaller space, large space, and even text as show in the example below:

<img src="/img/intercell_spacing.png" style="width: 500px;" alt="Example of adjusting the intercell spacing"/>

The table above makes use of varied inter-column spacing and a spanning header cell to group columns together. The -rather complicated- column definition is shown below:

{% highlight latex%}
@{}
l
S[table-format=2.2(2)]S[table-format=2.2(2)]%
@{~(}
S[table-format=1.2(2),tight-spacing=true,table-space-text-pre=(,table-space-text-post=)]%
@{)\;}
S[table-format=2.2(2)]%
@{~(}
S[table-format=1.2(2),tight-spacing=true,table-space-text-pre=(,table-space-text-post=)]%
@{)}
{% endhighlight %}

Here I exploit the inter-column spacing to group the values together and even draw brackets around values. The S column identifier is part of the *siunitx* package I discussed before, the options create the bracketed values and make certain columns more compact.

I hope you find this information useful, look at the links below for more information and examples.

**Sources:**

- [Designing Tables](http://www.ncsu.edu/labwrite/res/gh/gh-tables.html)
- [Effective design of data tables](http://www.thinkui.co.uk/resources/effective-design-of-data-tables)
- [Small Guide to Making Nice Tables](http://www.inf.ethz.ch/personal/markusp/teaching/guides/guide-tables.pdf)
- [Constructing Good Tables](http://lilt.ilstu.edu/gmklass/pos138/datadisplay/sections/goodtables.htm#Efficiently)
- [booktabs Package](http://www.ctan.org/pkg/booktabs)
- [siunitx Package](http://www.ctan.org/pkg/siunitx)
