---
layout: post
title: "Easy Sweave for LaTeX and R"
date: 2013-08-16 12:41
comments: true
categories: [LaTeX, R, Sweave]
author: Jason A. French
---
When you're writing up reports using statistics from R, it can be tiresome 
to constantly copy and paste results from the R Console.  To get around this, many of us use Sweave, which allows us to *embed* R code in LaTeX files. 
[Sweave](https://en.wikipedia.org/wiki/Sweave) is an R function that converts R code to LaTeX, a document typesetting language.  This enables accurate, shareable analyses as well as high-resolution graphs that are publication quality.
 
Needless to say, the marriage of statistics with documents makes writing up APA-style reports a bit easier, especially with Brian Beitzel's amazing [`apa6` class for LaTeX](http://www.ctan.org/pkg/apa6).

<!-- more -->
Making Sweave Available Systemwide
----------------------------------
However, Sweave doesn't always work correctly.  One common complaint that you'll get after Sweaving a file is `Sweave.sty not found!`. While Sweave.sty is a LaTeX package, it doesn't *live* with the rest of the LaTeX packages because it's installed using R.  Many people try to solve this by copying and pasting Sweave.sty into every document directory, but I'm sharing a better way below.

Using [Terminal.app](https://en.wikipedia.org/wiki/Terminal_%28OS_X%29):
```bash Go to the MacTeX or TeX Live local directory.
cd /usr/local/texlive/texmf-local/tex/latex
```

```bash Form a link between the R Sweave.sty files and MacTeX
sudo ln -s /Library/Frameworks/R.framework/Resources/share/texmf/ Sweave
```

```bash Tell MacTeX/TeX Live to recognize the file and rebuild the database
sudo mktexlsr
```

If using `mktexlsr` results in a `command not found` error, the TeX Live distribution probably isn't in your [$PATH](https://en.wikipedia.org/wiki/PATH_%28variable%29), but you can hunt for the program anyway.  For example, if you're using MacTeX 2013, the program will be found in a directory similar to this:

```bash
sudo /usr/local/texlive/2013/bin/x86_64-darwin/mktexlsr
```

Using Sweave in TeXShop
-----------------------

When you're getting started with LaTeX, many Mac users prefer the bundled
editor, TeXShop.  [Cameron Bracken](http://cameron.bracken.bz/sweave-for-texshop) gives us a helpful piece of code that allows easy Sweaving straight from TeXShop.  TeXShop uses various *engines* that allow it to render LaTeX.  Using a bit of [`BASH`](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29) scripting, we can write our own Sweave engine and make it available right within TeXShop.  I have adapted Cameron's original engine to accomodate Bibtex citations ([see here](https://en.wikibooks.org/wiki/LaTeX/Bibliography_Management#Why_won.27t_LaTeX_generate_any_output.3F)).

Using a text editor, paste in the following syntax and save the file as Sweave.engine:

```bash Sweave.engine
#!/bin/env bash
export PATH=$PATH:/usr/texbin:/usr/local/bin
R CMD Sweave "$1"
pdflatex "${1%.*}"
bibtex "${1%.*}.aux"
pdflatex "${1%.*}"
# If you run pdflatex again you get citations
pdflatex "${1%.*}"
```

Next, in Terminal.app, move the file to the TeXShop engines folder:

```bash Enable the Sweave.engine
mv Sweave.engine ~/Library/TeXShop/Engines/
chmod +x ~/Library/TeXShop/Engines/Sweave.engine
```

{% img /images/sweaveengine.png %}

Now restart TeXShop if it's running and you should see Sweave as an available option!

{% img /images/sweave.png %} 
