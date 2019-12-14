---
layout: post
title: The Meaning of Mean-less Distributions
mathjax: true
---

Everything has an average!  An average height, an average weight, an average number of times the neighbor's dog can bark in the middle of the night before you lose your $hit!  Most people, and most statistical professionals take it for granted that, not only does their data have a sample mean (of course it does!), there is some hidden population mean worth getting at.

This is false.  It's an uncomfortable fact of probability that most learn and immediately forget that the mean of a probability distribution $P$ is not always defined.  I suspect the reason that people forget or ignore this fact, is that it doesn't appear to arise in practice.  Since you can always compute a sample mean, there's an impression that this sample mean actually represents an underlying average.

Most will stop there, because, hey, that's very practical.  But as a mathematical logician, where things get weird and pathological, I get interested, and I enjoy finding the exceptions to the rules.

How would you know if you were sampling from a distribution, collecting data from a phenomenon, that doesn't have a population mean?

- the upper bound and/or lower bounds keep moving
- the average does not settle down according to central limit theorem expectations
- you might have a median or that might not be very stable either
- you're in a situation where infinity makes sense ... probably means infinite process since we can't do infinite samples in finite time

give an example

- example = 1/x^2
- show that it has no mean
- go to the Cauchy distribution

when does this happen

now think about fat tails that are bounded

- talk about something with fat tails where the tails are really long
- like Cauchy with the way way end chopped off
- model it

so what

no std deviation is much more likely

- are long tailed or heavy tailed distributions an example

Harvey Friedman and infinite finite numbers.

# sources

- https://en.wikipedia.org/wiki/Cauchy_distribution
- https://www.quora.com/What-is-the-intuition-behind-distributions-without-means
- The St Petersburg paradox https://stats.stackexchange.com/questions/91512/how-can-a-distribution-have-infinite-mean-and-variance
- https://www.johndcook.com/blog/cauchy_estimation/
- https://www.astroml.org/book_figures/chapter3/fig_cauchy_median_mean.html
- https://stats.stackexchange.com/questions/232967/what-makes-the-mean-of-some-distributions-undefined
