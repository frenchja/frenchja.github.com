---
layout: post
title: "Analyze Student Exam Items using IRT"
date: 2013-10-25 14:26
comments: true
categories: [R, psychology, tests]
author: Jason A. French
---
I've cross-posted this at the [SAPA Project's blog](http://sapa-project.org/blog/).

[Item Response Theory](https://en.wikipedia.org/wiki/Item_response_theory) can be used to evaluate the effectiveness of
exams given to students.  One distinguishing feature from other paradigms is that it does not assume that every question 
is equally difficult (or that the difficulty is tied to what the researcher said).  In this way, it is an empirical investigation
into the effectiveness of a given exam and can help the researcher 1) eliminate bad or problematic items and 2) judge whether the test was too difficult or the students simply didn't study.

In the following tutorial, we'll use [R](http://www.r-project.org/) (R Core Team, 2013) along with the [`psych`](http://cran.r-project.org/web/packages/psych/index.html) package (Revelle, W., 2013) to look at a hypothetical exam.
<!-- more -->
Before we get started, remember that `R` is a programming language.  In the examples below, I perform operations on data using [*functions*](https://en.wikipedia.org/wiki/Function_%28computer_science%29) like [`cor`](http://stat.ethz.ch/R-manual/R-patched/library/stats/html/cor.html) and [`read.csv`](http://stat.ethz.ch/R-manual/R-devel/library/utils/html/read.table.html).  We can also save the output as [*objects*](https://en.wikipedia.org/wiki/Object_%28computer_science%29) using the assignment arrow, `<-`. It's a bit different from a point-and-click program like SPSS, but you *don't* need to know how to program to analyze exams and questions using IRT!

First, load the the `psych` package.  Next, load the students' grades into R using `read.csv()` from the `psych` package.

```r
# Load the psych package
library(psych)
# Read the csv file and save it as grades
grades <- read.csv(file = "~/Downloads/Grades.csv")
```

Notice that we are using item-level grades, where each row is a given student and each cell is the number of points received on that question.  In my example, V1, V2, etc. correspond to exam questions.  Your matrix or data frame should look like this:

```r
head(grades)
```

```
##   V1 V2 V3 V4 V5 V6 V7 V8 V9 V10 V11 V12 V13 V14 V15 V16 V17 V18 V19 V20
## 1  2  2  2  2  4  2  3  2  4   2   2   3   5   3   6   2   4   4   4   3
## 2  2  0  2  2  0  0  3  0  4   2   0   1   3   0   0   2   2   1   0   0
## 3  2  2  2  2  0  2  3  2  4   0   0   3   5   0   5   2   4   2   4   2
## 4  2  1  2  2  3  0  0  2  4   2   2   3   1   1   2   0   4   1   4   3
## 5  2  2  2  2  4  2  3  2  4   2   2   3   5   3   4   2   4   4   4   3
## 6  1  0  2  2  0  2  0  2  0   2   1   2   5   0   3   2   4   2   4   3
##   V21 V22 V23 V24 V25 V26 V27 Total
## 1   2   2   4   4   6   6   6    91
## 2   0   0   0   3   3   6   6    42
## 3   2   1   2   4   6   5   6    72
## 4   0   2   4   4   0   0   0    49
## 5   2   2   4   4   5   6   6    88
## 6   0   0   4   4   6   4   3    58
```


Next, compute the [polychoric correlations](https://en.wikipedia.org/wiki/Polychoric_correlation) on the raw grades (not including the Total column).  By using polychoric correlations, we estimate the normal distribution of latent content knowledge, which can be underestimated if Pearson correlations are instead used on polytomous items (see  Lee, Poon & Bentler, 1995).

```r
# Perform a polychoric correlation on grades and save it as grades.poly
grades.poly <- polychoric(x = grades[, 1:27], polycor = TRUE)
```

Now that we have the polychoric correlations, we can run `irt.fa()` on the dataset to see the item difficulties and information.

```r
# Using grades.poly, perform an irt and save it as grades.irt
grades.irt <- irt.fa(x = grades.poly, plot = TRUE)
# Plot the output from grades.irt
plot(grades.irt)
```

{% img  /images/grades.png %}

Thus, we have some great items that have a lot of information about students of average and low content knowledge (e.g., V24, V17, V18), but not enough to distinguish the high-knowledge students.  In redesigning an exam for next semester or year, we might save the best performing questions while trying to rewrite the existing questions or trying new questions.

Next, let's see what how well the test did *overall* at distinguishing students:

```r
plot.irt(type = "test", grades.irt)
```

{% img /images/test.png %} 

The second plot shows the *test performance*.  We have great reliability for distinguishing who didn't study (lowers end of our latent trait), but overall the test may have been too easy (opposite of my prediction).  It's important that the test is not too difficult to discourage students, but the graph above suggests that we had very low information at how students that studied were different from eachother.  This is again reflected by the histogram plotted in the next section, where high scoring students seem to cluster together.

Rescaling the test
---------------

While many students in our hypothetical dataset did very well on the exam, instructors may 
need to rescale their exam so that the mean grade is an 85% or 87.5%.  Using the `scale()` function (see also [`rescale()`](http://personality-project.org/r/psych/help/rescale.html) in `psych` package), we
can ensure that the [rank-order](https://en.wikipedia.org/wiki/Ranking#Ranking_in_statistics?s) distribution of the students is preserved (allowing us to distinguish
those who studied well from those who didn't), while scaling the sample distribution to fit in with other classes in your department.

Currently, our scores are in cumulative raw points.  Notice that we divide the Total points column by 91 to convert the histogram into grade percentages.  Let's plot a histogram to see the distribution of scores.

```r
# Load the ggplot2 library
library(ggplot2)
# Plot a histogram
qplot((Total/91)*100, data=grades, geom="histogram",xlab='Raw Grades',
	ylab='# of Students',main='Distribution of student grades')
```

{% img /images/raw.png %}

The distribution has a mean 71.68 percent and a standard deviation of 15.54.  Given grade inflation, 
it may look like your students are doing poorly when in fact the distribution is similiar to other courses 
being taught.  Next, we can rescale the grades, creating a mean of 87.5 and a standard deviation of 7.5.  These numbers are arbitrary so use your best judgement.

```r
scaled.grades <- scale(grades$Total) * 7.5 + 87.5
qplot(scaled.grades, xlab='Scaled Grades',
	ylab='# of Students',main='Distribution of student grades')
```

{% img  /images/scaled.png %} 

The second distribution may be preferred, depending on your needs.  With the raw distribution, we would have had 45% of the students receiving grades below a C-, assuming a normal distribution (for the curious, R can calculate these probabilites using the `pnorm` function: ```pnorm(q=70,mean=71.68,sd=15.54)```).  Now, 0.9% of students would fall below the 70% cutoff.  Again, my mean and standard deviation chosen in the above example are arbitrary.

## References

* Lee, S. Y., Poon, W. Y., & Bentler, P. M. (1995). *A two‐stage estimation of structural equation models with continuous and polytomous variables*. British Journal of Mathematical and Statistical Psychology, 48(2), 339-358.

* R Core Team (2013). *R: A language and environment for statistical computing*. R Foundation for Statistical Computing, Vienna, Austria. [http://www.R-project.org/](http://www.R-project.org/).

* Revelle, W. (2013). *psych: Procedures for Personality and Psychological Research*. Northwestern University, Evanston, Illinois, USA. [http://CRAN.R-project.org/package=psych](http://CRAN.R-project.org/package=psych). Version = 1.3.10.

