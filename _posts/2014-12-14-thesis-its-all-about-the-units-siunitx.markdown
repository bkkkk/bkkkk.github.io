---
title: 'Thesis Tutorials #3: siunitx'
subtitle: 'Its all about the units'
date: 2014-12-14
---

There are several packages you might find useful while putting together your thesis. In my case, I use over 20 packages. Some of them provide facilities that you'll use all over the place, while others are more specific. Over the next few posts I will cover some of the ones I've found useful.

# Unit! Line up!

Typesetting values with units correctly is an important, and often overlooked, part of thesis writing. **siunitx** is a very powerful package that provides you with a consistent syntax to typeset values, ranges, lists, units, tabulated data, and uncertainties.

## The basic value-unit pair

In siunitx, one types a value with a unit using the **\SI** command. This command works in both math-mode and inline with text. In both environments the value and the unit are rendered in the same way. The syntax is fairly self-explanatory. The package supports a very large number of different units. Note that in the example above we used a shortcut provided by siunitx for the common unit TeV. One can also write **\tera\electronvolt** or **\centi\meter**. For a full list of available units check-out the siunitx [documentation](http://mirror.ox.ac.uk/sites/ctan.org/macros/latex/contrib/siunitx/siunitx.pdf). You can also typeset composite units such as **\centi\meter\per\second** and so on.

------

**Example**

{% highlight latex %}
\SI{8}{\TeV}

$\sqrt{s}=\SI{8}{\TeV}$
{% endhighlight %}

**Output**

![Inline example](/img/Inline.png "siunitx in inline equation")
![In an equation example](/img/InEquation.png "siunitx used in in equation")

------

If you need to render units or values on their own, siunitx provides the **\si** and **\num** commands. The former will typeset large numbers with a delimiter to group digits together to improve readability. See the documentation above to learn how to change the delimiter.

------

**Example**

{% highlight latex %}
\si{\TeV}
\num{1003333}
{% endhighlight %}

------

There is also a basic facility to render values with uncertainties as shown below. Note that the uncertainty is defined with no decimal points.

------

**Example**

{% highlight latex %}
\SI{245.6(100)}{\pico\barn}
{% endhighlight %}

**Output example**

![Uncertainty example](/img/WithUncertainty.png "Example with uncertaintites")

------

## Ranges and lists

Range and list commands are also defined, there are multiple options to configure how these lists/ranges are rendered. There commands **\numrange** and **\numlist** are also available to render lists/ranges of numbers without units.

------

**Example**

{% highlight latex %}
\SIrange{2-10}{\percent}
\SIlist{5;9;12}{\cm}
{% endhighlight %}

**Output**

![Range of numbers](/img/Range.png "Example of ranges")

![List of numbers](/img/List.png "Example of lists")

------

## Tables

Siunitx can also help you typeset tables. For this purpose the package provides a new column type S. First let me show you a full (real world) example of the package in action, and then I'll walk you through the syntax.

-----

![Example table](/img/Table.png "Example with units in table")

-----

The uncertainties and decimal points are aligned in each column and the correct spacing is applied between the central value, the uncertainty operator, and the uncertainty.

-----

{% highlight latex %}
\begin{table}[htbp]
    \sisetup{separate-uncertainty=true}
    \centering
    \begin{tabular}{@{}
                    S[table-format=4] % Mass Label
                    *{3}{S[table-format=2.1(1)]} % SMT Efficiency
                    S[table-format=2] % Added Acceptance
                    @{}}
        \toprule
        {$m_{Z'}$ [\si{\GeV}]} & {$\epsilon_{\textrm{SMT}}$ [\si{\percent}]} &
        {$\epsilon_{\textrm{MV1}}$ [\si{\percent}]} &
        {Overlap [\si{\percent}]} & {Added Acceptance [\si{\percent}]} \\
        \midrule
        1000 & 14.7(1) & 72.7(1) & 75.3(1) & 5  \\
        1300 & 16.2(1) & 70.7(1) & 72.4(1) & 6  \\
        1600 & 17.2(1) & 66.9(1) & 68.0(1) & 8  \\
        2000 & 17.9(1) & 60.9(1) & 63.0(1) & 11 \\
        2500 & 17.8(2) & 56.3(2) & 59.0(2) & 13 \\
        3000 & 17.4(1) & 57.3(1) & 60.0(1) & 12 \\
        \bottomrule
    \end{tabular}
    \caption{Some Caption}
\end{table}
{% endhighlight %}

-----

There is a lot of code here so lets work through the important bits. First up, the column definition used:

{% highlight latex %}
@{}
  S[table-format=4] % Mass Label
  *{3}{S[table-format=2.1(1)]} % SMT Efficiency
  S[table-format=2] % Added Acceptance
@{}
{% endhighlight %}

The **@{}** command, which is not siunitx specific, defines the inter-column spacing, in this case to no space. So here we remove the spacing before the first column and after the last column. The next command **S[table-format=4]** instructs latex to let siunitx handle the rendering of values in this column, and the option **table-format=4** defines the number format in the table to have four non-decimal digits. The next command:

{% highlight latex %}
*{3}{S[table-format=2.1(1)]}
{% endhighlight %}

States that there are 3 columns following with values that have 2 non-decimal digits, one decimal digit, and an uncertainty with a single digit. We state the expected format so that siunitx sets the appropriate column size. If we have a column with mixed values we must define the table-format to the largest value. For example, if we tabulate the values: 10.2 and 150.35, we set **table-format=3.2** in the column definition. I'll leave the last column definition as a -trivial- exercise for the reader.
When defining the headers of the table we write them within curly braces. This stops siunitx from trying to render the text. You can use this syntax to stop siunitx from rendering specific cells.

## Pitfalls

While siunitx is a very useful packages, it does run into trouble in some places. In particular, it does not provide you with a syntax to typeset values with multiple uncertainties or asymmetric uncertainties. Luckily there is a way to get a look that is consistent with siunitx for these types of values.
I have included a few commands to help with rendering asymmetric uncertainties and multiple uncertainties:

-----

### Asymmetric uncertainty

**Code**

{% highlight latex %}
\newcommand{\asymUnc}[4]{\ensuremath{#1\;^{+\;#2}_{-\;#3}\;#4}}
\asymunc{10.2}{3.2}{1.4}{\si{\TeV}}
{% endhighlight %}

**Output**

![](/img/AsymUncertainty.png "Example with asymmetric uncertainty")

### Multiple uncertainties

**Code**

{% highlight latex %}
\newcommand{\stat}{\ensuremath{\textrm{\,(stat.)}}}
\newcommand{\syst}{\ensuremath{\textrm{\,(syst.)\,}}}
\newcommand{\lumi}{\ensuremath{\textrm{\,(lumi.)\,}}}

$\num{165}\;^{+\;10}_{-\;8}\stat\pm\num{17}\syst\si{\pico\barn}$
{% endhighlight %}

**Output**

![](/img/MultipleAsymmetric.png "Example with multiple uncertainties")

-----

You will need to adapt and mix-and-match components from the two code snippets above for your particular needs but its a good place to start.
I hope I've managed to convince you to pick up siunitx, the earlier you start using these packages the better off you will be. Please take a look at the documentation as I've only scratched the surface of what can be done with siunitx.

If you have any questions please leave a comment. Thanks for reading and see you next time.
