---
layout: post
title:  "Beware of the Bayesian imposter"
date:   2022-09-30 00:00:00 -0700
categories: statistics
tags: statistics bayesian
---


## Preface

Working on an experimentation platform, it is imposssible not to come across the frequentist versus bayesian debate
Passionate debates have been raged over decades, there's little new under the sun
Personally, not feel too strongly in either direction: both approaches have their merits, shine in different ways
However, couple of weeks ago, Steve Howard shared a really neat paper by A.P. Dawid from the 90s: "Selection paradoxes of Bayesian inferences",
a wonderful paper you ought to read over this post
I certainly would have wanted to read this paper a lot sooner, and in this post want to summarize the ideas while putting it in an experimentation perspective

## Bayesian vs Frequentist hypothesis testing

Simple experiment setting: treatment and control variant, decide which variant to pick based on a particular outcome measured across a population

To briefly recap the differences between frequentist and bayesian hypothesis testing:

frequentist testing:
- underlying parameter fixed, data random
- guarantees on inference method across repeated draws of data
- thus, no guarantee on the particular outcome of an experiment (in fancy termonology: guarantees hold unconditioned on data)


bayesian testing:
- data fixed, underlying paramater is random
- inference on parameter is based on prior distribution for parameter + likelihood based on data
- guarantees on particular outcome, and conditioning on data not a problem: inference on parameter is already conditioned on the data!

The last point is often quoted to espouse the benefits of the Bayesian approach:
while for the classical frequentist approach, sample size needs to be fixed up front (we cannot be adaptive to data):
if we do not wait to gather all data we intended, but based on the observations seen so far make a decision, we are likely to be fooled
in the Bayesian case, we can be adaptive, and make a decision at any point: do we keep gathering more data, choose the treatment variant, or the control variant.

## Dawid's paradox

Dawid, a well known Bayesian statistician, posed the following paradox in the paper mentioned above:
The Bayesian approach sounds great: we can be adaptive to the data, while the frequentist cannot.
Now consider a Bayesian that uses a non-informative prior (or flat) prior: a prior for which every outcome is equally likely.
It is easy to see that when using such a non-informative prior, the frequentist and bayesian approach lead to the exact same results:
the Bayesian approach multiplies the likelihood with the prior to get the posterior. If the prior is flat, then we are multiplying the likelihood by 1.
Hence, our inference is based only on the likelihood -- exactly the same likelihood the frequentist is using.

But now the adaptivity is a bit odd: how can it be fine to be adaptive when using the Bayesian approach,
while the frequentist is stuck waiting to collect all their data, despite results being exactly the same?!

## Resolving the paradox

Now you are probably able to point at the culprit that's causing the paradox: the flat prior.
As usual, there is no free lunch: the Bayesian approach has not magically solved the issue of peeking for free.
Rather, to solve the problem posed by peeking, we need to solve the problem of the prior: Bayesian analysis works when the prior works.

## Take-away

Bayesian and frequentist approaches and guarantees are fundamentally different, but under certain conditions, they lead to exactly the same results.
Dawid demonstrates clearly that this creates a paradox: when results are the same, approaches cannot have fundamentally different guarantees.
This is where the Bayesian imposter comes in: to truly enjoy the benefits of a Bayesian approach, one needs to carefully think about what prior to use,
otherwise, the experimenter might just a frequentist in disguise, and be ignorant of the risks that poses to the validity of their claims.

More generally, when the experts have been debating the respective merits of frequentist vs bayesian approaches,
it must be true that one cannot be clearly better than the other: then the debate would have been settled a long time ago.
When you experiment as a Bayesian, make sure you think carefully about your prior, or you might well be a frequentist without realizing it!
