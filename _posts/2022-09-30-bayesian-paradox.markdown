---
layout: post
title: "Beware of the Bayesian imposter"
date: 2022-09-30 00:00:00 -0700
categories: statistics
tags: statistics bayesian
---

![imposter statistician](/assets/img/imposter.png)

Sometimes, statistical guarantees are not what they seem.
Here, we discuss the implications of a classic work that demonstrates a paradox with the Bayesian approach to experiment analysis:
when not careful, the experimenter runs the risk of running a frequentist analysis without realizing it.
This can have important implications: when combined with peeking, the credible intervals might not be so credible after all.

## The Great Debate

When you are building an [experimentation platform][eppo],
it's impossible to avoid the debates between frequentist and Bayesian approaches to hypothesis testing.
Ardent supporters on both sides espouse the benefits of their approach, but when the debate has been raging for decades,
and between the most famous and brightest statisticians, it's no surprise there aren't any easy answers.

An oft highlighted issue of the classical frequentist approach is that of the peeking problem:
with modern tooling, it's often easy to monitor the results of experiments while collecting data.
However, the frequentist guarantees [break when you peek at experiments][evan-peeking], leading to an inflated type I error rate:
you end up claiming there is a difference between variants far more often than you want to in the case both variants in fact have the same outcome distribution.
On the other hand, Bayesian supporters often highlight that the Bayesian approach does not suffer from the peeking problem,
claiming that this is one of the benefits of using a Bayesian testing regime over a frequentist one.

Statistics is often more subtle than it first seems, and so we have to wonder: is this too good to be true?
Famous Bayesian statistician [Philip Dawid][dawid] wrote a fantastic paper in the 90s: ["Selection paradoxes of Bayesian inferences"][dawid-paper], shared to me by [Steve Howard][howard],
that I highly recommend you read.
It's fairly accessible, both in that there is no paywall, and that the reasoning is easy to follow.
In this work, he introduces what I would like to call the "Bayesian imposter": someone who claims the benefits of the Bayesian approach,
while actually doing frequentist analysis.

Let's put this in an experimentation context.
Before we do however, I want to note that this post is not about choosing sides: both frequentist and Bayesian approaches have their merits, and shine in different ways.
Rather, the work of Dawid helps us to understand and appreciate the subtleties in this debate.

## The experimentation setting

First of all, let's get on the same page regarding what an experiment is; we will keep it high level and consider the most basic setting; it is all we need.
Consider we want to test a new variant: treatment, versus the status quo: control, and we would like to figure out whether treatment is better than control according to some metric we care about.
We resort to some statistics to help us figuring this out: We randomly assign subjects to either treatment or control, and measure the outcome on said metric.
Now we use some statistical method to compare the two variants, e.g. by comparing the means using a T-test (frequentist) or comparing the posteriors by combining priors with a likelihood (Bayesian).

## Frequentist versus Bayesian hypothesis testing

Before diving in, let us briefly recap the differences between frequentist and Bayesian hypothesis testing.

In the frequentist setting the underlying parameter fixed, while the observed data is considered as the random variable.
Thus, the frequentist thinks about guarantees on their inference methods through considering repeated draws of data.
For example, a confidence interval is a function of the data (thus also a random variable),
with the underlying idea that as we generate data, and hence confidence intervals, over and over, most times the
generated confidence intervals contains the parameter of interest.
But this has the unintuitive downside that a frequentist cannot give any certainty about a particular set of observations.

The Bayesian perspective flips this around: you are handed a set of observations, clearly those are fixed.
What you do not know is the value of the underlying parameter, so let's consider that the random variable.
We encode our beliefs through a prior distribution for said parameter, and then combine that with the likelihood of
observing the data given the parameter to form a posterior distrubtion.
Now the posterior distribution allows us to reason about this particular outcome: conditioning on the data is not a problem,
because inference on the parameter is already conditioned on the data!

The last point is often quoted as one of the main benefits of the Bayesian approach to experimentation.
With the classical frequentist approach sample size needs to be fixed up front (we cannot be adaptive to data).
If we ignore this and decide whether to continue collecting data based on what we have observed so far, or are ready to make a decision, we can dramatically
increase the risk of a false positive.
This is different for in the Bayesian case, we can be adaptive, and make a decision at any point:
do we keep gathering more data, choose the treatment variant, or the control variant.
This follows from your elementary probability class: from the  Bayesian perspective, the data is fixed (a constant) and only the
parameter we are interested in is considered a random variable. Conditioning on the data (a constant) does not impact results.

## Dawid's paradox

But is it really so black and white?
Dawid, a well known Bayesian statistician, posed the following paradox in the paper mentioned above:
The Bayesian approach sounds great: we can be adaptive to the data, while the frequentist cannot.
But now consider a Bayesian that uses a non-informative prior (or flat) prior: a prior for which every outcome is equally likely.
It is easy to see that when using such a non-informative prior, the frequentist and Bayesian approach lead to the exact same results:
the Bayesian approach multiplies the likelihood with the prior to get the posterior (modulo normalization).
If the prior is flat, then we are multiplying the likelihood by 1.
Hence, our inference is based only on the likelihood -- exactly the same likelihood the frequentist is using!

But now the adaptivity is a bit odd: how can it be fine to be adaptive (or peek) when using the Bayesian approach,
while the frequentist is stuck waiting to collect all their data, despite the two approaches leading to exactly the same results?

## Resolving the paradox

The culprit is clear: it is the flat prior that causes the Bayesian posterior to be the same as the likelihood on which
frequentist analysis is based.
As usual, there is no free lunch: the Bayesian approach has not magically solved the issue of peeking for free.
Rather, to solve the problem posed by peeking, we need to solve the problem of the prior: Bayesian analysis works when the prior works.
David Robinson demonstrates the [same point using simulations][robinson-peeking].

Adaptivity, or the ability to peek, is very powerful and the Bayesian approach is far simpler than the [frequentist alternative of sequential testing][sequential-paper].
But powerful results often require strong assumptions; in this case the need for a well-chosen prior.

## Take-away

Bayesian and frequentist approaches and guarantees are fundamentally different, but under certain conditions, they lead to exactly the same results.
Dawid demonstrates clearly that this creates a paradox: when results are the same, approaches cannot have fundamentally different guarantees.
This is where the Bayesian imposter comes in: to truly enjoy the benefits of a Bayesian approach, one needs to carefully think about what prior to use,
otherwise, the experimenter might just a frequentist in disguise, and be ignorant of the risks that poses to the validity of their statistical claims.

More generally, when the experts have been debating the respective merits of frequentist versus Bayesian approaches,
it must be true that one cannot be clearly better than the other: then the debate would have been settled a long time ago.
When you experiment as a Bayesian, make sure you think carefully about your prior, or you might well be a frequentist without realizing it!


[dawid]: https://www.statslab.cam.ac.uk/~apd/
[howard]: https://www.stevehoward.org/
[dawid-paper]: https://projecteuclid.org/ebooks/institute-of-mathematical-statistics-lecture-notes-monograph-series/Multivariate-analysis-and-its-applications/Chapter/Selection-paradoxes-of-Bayesian-inference/10.1214/lnms/1215463797
[sequential-paper]: https://arxiv.org/abs/1810.08240
[evan-peeking]: https://www.evanmiller.org/how-not-to-run-an-ab-test.html
[robinson-peeking]: http://varianceexplained.org/r/bayesian-ab-testing/
[eppo]: https://www.geteppo.com/
