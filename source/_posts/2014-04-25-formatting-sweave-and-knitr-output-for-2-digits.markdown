---
layout: post
title: "Formatting knitr output for 2 digits in R"
date: 2014-04-25 16:53
comments: true
categories: [R, Sweave, knitr, digits]
published: true
---

### Overview of reproducible research
Reproducible research is a phrase that describes an academic paper that contains the code and data along with the researcher's interpretation.  One way of engaging in reproducible research is to use R code directly inside your LaTeX document. knitr is an R package for doing reproducible research and is designed as a replacement for Sweave. A knitr document combines your R analysis with your [LaTeX](https://en.wikipedia.org/wiki/LaTeX) manuscript (i.e., knitr = R + LaTeX).  In doing so, the experimental design and method of analysis is easily replicated and critiqued by reviewers and other researchers as the full analysis used to produce the results is submitted along with the final paper.

By using knitr, the researcher can easily create ANOVA and demographic tables directly from the data without messing around in Excel.  The basic example below contains the beginning of a hypothetical Methods section of a manuscript. We want to take the values from an R table, which has the breakdown of participants by gender and ethnicity, and display them as numbers in our manuscript. 
<!-- more -->

```tex Basic knitr.Rnw Example
\documentclass[12pt]{article}
\usepackage[sc]{mathpazo}
\usepackage[T1]{fontenc}
\usepackage{geometry}
\geometry{verbose,tmargin=2.5cm,bmargin=2.5cm,lmargin=2.5cm,rmargin=2.5cm}
\setcounter{secnumdepth}{2}
\setcounter{tocdepth}{2}
\usepackage{url}
\usepackage[unicode=true,pdfusetitle,
 bookmarks=true,bookmarksnumbered=true,bookmarksopen=true,bookmarksopenlevel=2,
 breaklinks=false,pdfborder={0 0 1},backref=false,colorlinks=false]
 {hyperref}
\hypersetup{
 pdfstartview={XYZ null null 1}}
\usepackage{breakurl}
\begin{document}

<<setup, include=FALSE, cache=FALSE>>=
library(knitr)
# set global chunk options
opts_chunk$set(fig.path='figure/minimal-', fig.align='center', fig.show='hold')
options(replace.assign=TRUE,width=90)
@

\title{A Minimal Demo of knitr}
\author{Jason A. French}
\maketitle


<<random-ethnicity, include=FALSE>>=
# Create data.frame of random ethnicities
x <- data.frame(Ethnicity = sample(as.factor(
  rep(x = c('White','African American','Asian American', 'Latino', 'Pacific Islander')
    )
  ),size = 100, replace = TRUE),
  Gender=rep(c('Male', 'Female'),100))
x.table <- table(x)
@

\section{Methods}
We recruited \Sexpr{sum(x.table)} university undergraduates from an introductory psychology class. Participants were drawn from various genders and ethnic groups across the Chicago area (see Table~\ref{tab:ethnicity})...

<<ethnicity-table, echo = FALSE, results = 'asis'>>=
library(xtable)
xtable(x.table, caption = 'Participant Ethnicities', label='tab:ethnicity')
@

\end{document}
```

As we see below, running `knit()` on our knitr manuscript inside R produces a regular LaTeX file that can be compiled with to a PDF using pdflatex or [TeX Shop](http://pages.uoregon.edu/koch/texshop/).

```r Running knit() knitr.Rnw inside R
library(knitr)
knit(input = 'knitr.Rnw', output = 'knitr.tex')
```

```tex Resulting knitr.tex LaTeX document
\documentclass[12pt]{article}\usepackage[]{graphicx}\usepackage[]{color}
\makeatletter
\definecolor{fgcolor}{rgb}{0.345, 0.345, 0.345}
\newcommand{\hlnum}[1]{\textcolor[rgb]{0.686,0.059,0.569}{#1}}
\newcommand{\hlstr}[1]{\textcolor[rgb]{0.192,0.494,0.8}{#1}}
\newcommand{\hlcom}[1]{\textcolor[rgb]{0.678,0.584,0.686}{\textit{#1}}}
\newcommand{\hlopt}[1]{\textcolor[rgb]{0,0,0}{#1}}
\newcommand{\hlstd}[1]{\textcolor[rgb]{0.345,0.345,0.345}{#1}}
\newcommand{\hlkwa}[1]{\textcolor[rgb]{0.161,0.373,0.58}{\textbf{#1}}}
\newcommand{\hlkwb}[1]{\textcolor[rgb]{0.69,0.353,0.396}{#1}}
\newcommand{\hlkwc}[1]{\textcolor[rgb]{0.333,0.667,0.333}{#1}}
\newcommand{\hlkwd}[1]{\textcolor[rgb]{0.737,0.353,0.396}{\textbf{#1}}}

\usepackage{framed}

\definecolor{shadecolor}{rgb}{.97, .97, .97}
\definecolor{messagecolor}{rgb}{0, 0, 0}
\definecolor{warningcolor}{rgb}{1, 0, 1}
\definecolor{errorcolor}{rgb}{1, 0, 0}
\newenvironment{knitrout}{}{} % an empty environment to be redefined in TeX

\usepackage{alltt}
\usepackage[sc]{mathpazo}
\usepackage[T1]{fontenc}
\usepackage{geometry}
\geometry{verbose,tmargin=2.5cm,bmargin=2.5cm,lmargin=2.5cm,rmargin=2.5cm}
\setcounter{secnumdepth}{2}
\setcounter{tocdepth}{2}
\usepackage{url}
\usepackage[unicode=true,pdfusetitle,
 bookmarks=true,bookmarksnumbered=true,bookmarksopen=true,bookmarksopenlevel=2,
 breaklinks=false,pdfborder={0 0 1},backref=false,colorlinks=false]
 {hyperref}
\hypersetup{
 pdfstartview={XYZ null null 1}}
\usepackage{breakurl}
\IfFileExists{upquote.sty}{\usepackage{upquote}}{}
\begin{document}

\title{A Minimal Demo of knitr}
\author{Jason A. French}
\maketitle

\section{Methods}
We recruited 200 university undergraduates from an introductory psychology class. Participants were drawn from various genders and ethnic groups across the Chicago area (see Table~\ref{tab:ethnicity})...

\begin{table}[ht]
\centering
\begin{tabular}{rrr}
\hline
 & Female & Male \\ 
\hline
African American &  22 &  16 \\ 
Asian American &  26 &  30 \\ 
Latino &  14 &  30 \\ 
Pacific Islander &  18 &  14 \\ 
White &  20 &  10 \\ 
\hline
\end{tabular}
\caption{Participant Ethnicities} 
\label{tab:ethnicity}
\end{table}
\end{document}
```

Last, after compiling our LaTeX file using TeX Shop, we're greeted with the final product below:

{% img /images/knitr-example.png %}

### Summary thus far
The example above used data from R directly in a sentence in the Methods section (i.e., "We recruited 200 university undergraduates from an introductory psychology class.") and did so using the `\Sexpr{}` command in the [knitr](https://en.wikipedia.org/wiki/LaTeX) manuscript (i.e., knitr.Rnw).  The `\Sexpr{}` command contained an `R` expression of the total number participants.  This expression was evaluated and converted to LaTeX code when we ran the `knit()` function on the .Rnw file, which produces a .tex document. The .tex document contained no R code and was therefore ready to be compiled to a PDF using TeX Shop or pdf2latex.

## Forcing knitr to round to 2 decimal places

The default behavior works great *most* of the time. However, what if we didn't have whole numbers in our data table?  What if we had percentages that we wanted to round down to 2 digits?  For example, the value `\Sexpr{pi}` would be evaluated and replaced with 3.141593 in the LaTeX file.  One common problem, and part of [Yihui's motivation](https://stackoverflow.com/questions/11062497/how-to-avoid-using-round-in-every-sexpr) for replacing Sweave with [`knitr`](http://yihui.name/knitr/), is that `\Sexpr{}` doesn't automatically round digits.  

In Sweave, *each* value of pi would be encased in `round(pi,2)`.  Thus, we end up with `\Sexpr{round(pi,2)}`.  Yihui fixed this problem by automatically rounding digits, the length of which is set with `options(digits=2)` in the knitr preamble in your .Rnw document.  See below:

```tex Typical knitr preamble
<<>>=
library(knitr)
options(digits=2)
@
```

The default rounding behavior of knitr works well *until* a value contains a 0 after rounding, such as 123.10.  Using `round(123.10,2)` outputs 123.1. In this case, every other value in the manuscript table would be aligned at the decimal place *except* for the unlucky value - sticking out like a sore thumb. To fix this, you *could* use `sprintf("%.2f", pi)` every time you have to call `\Sexpr{}` in the manuscript - but then what's the advantage of using [knitr](http://yihui.name/knitr/)? This hack unnecessarily complicates the manuscript.

### Modify the default inline_hook for knitr
After seeing a [StackOverflow answer by Josh O'Brien](https://stackoverflow.com/questions/11062497/how-to-avoid-using-round-in-every-sexpr), I realized that the default inline_hook for knitr could be easily modified to use the [`sprintf()`](http://stat.ethz.ch/R-manual/R-devel/library/base/html/sprintf.html) command instead of [`round()`](http://stat.ethz.ch/R-manual/R-devel/library/base/html/Round.html).  The change will forcibly output all values to 2 decimal places. Below, we see the default behavior for knitr when processing inline R expressions:

```r knitr's Default Hook
library(knitr)
knit_hooks$get("inline")
```

```
function (x) 
{
    if (is.numeric(x)) 
        x = round(x, getOption("digits"))
    paste(as.character(x), collapse = ", ")
}
```

*Note:* My original code for this post used the [`format()`](http://stat.ethz.ch/R-manual/R-patched/library/base/html/format.html) command.  [Winston Chang](https://github.com/wch) pointed out that this could lead to unreliable output and tweaked the code to use [`sprintf()`](http://stat.ethz.ch/R-manual/R-devel/library/base/html/sprintf.html). The credit for the more efficient function below goes to him.  Below, we add out improved `inline_hook` to the preamble of our knitr document:

```tex Improving the inline_hook
<<>>=
library(knitr)
inline_hook <- function (x) {
  if (is.numeric(x)) {
    # ifelse does a vectorized comparison
    # If integer, print without decimal; otherwise print two places
    res <- ifelse(x == round(x),
      sprintf("%d", x),
      sprintf("%.2f", x)
    )
    paste(res, collapse = ", ")
  }
}
knit_hooks$set(inline = inline_hook)
@
```

### Working Example
Let's put it all together!  The following is a working example of the the suggested knitr inline_hook function, which *should* give more reliable output by rounding inline values to 2 decimal places. 

```tex Winston's Example
\documentclass[12pt]{article}
\begin{document}

<<load, include=FALSE, echo=FALSE>>=
library(knitr)

inline_hook <- function (x) {
  if (is.numeric(x)) {
    # ifelse does a vectorized comparison
    # If integer, print without decimal; otherwise print two places
    res <- ifelse(x == round(x),
      sprintf("%d", x),
      sprintf("%.2f", x)
    )
    paste(res, collapse = ", ")
  }
}

knit_hooks$set(inline = inline_hook)
@

Inline code looks like \Sexpr{123}, \Sexpr{123.4}, \Sexpr{123.45}, \Sexpr{123.456}.

And with vectors: \Sexpr{c(123, 123.4, 123.45, 123.456)}.

Regular output is not affected by the inline hook:

<<>>=
123
123.4
123.45
123.456

c(123, 123.4, 123.45, 123.456)

getOption('digits')
@
\end{document}
```

{% img /images/example.png %}
