---
layout: post
title: "The useful useless hypothesis test"
date: 2022-10-01 00:00:00 -0700
categories: statistics
tags: statistics confidence interval
image: equations.png
usemathjax: true

---


Frequentist hypothesis testing is both confusing and subtle

Let's make sense of it with a very simple procedure that is valid, though useless in practice.

This post will help you understand
- power
- multiple testing
- the peeking problem
- p-values

and most importantly: why we have to be so careful when talking about what the result of a hypothesis test indicates.
Or rather, what it does not.

Note, this note focuses on the statistical side of the hypothesis testing procedure.


# Setting

First, let's get in the mood for running an experiment.
Suppose we work at an ecommerce platform and want to test prepending each call to action with "hey dude, "
That is, instead of a "buy now" button, we will try "hey dude, buy now",
and instead of "sign up", we'll use "hey dude, sign up".

Normally, we would randomize users into a control group, seeing the boring "buy now",
and a treatment group, seeing the much more engaging "hey dude, buy now" buttons; that is the hope at least.
Then, we wait for a while, observe the number of orders in both groups, perform some statistical magic, and decide whether we can reject the null hypothesis that there is no difference between the two versions.[^1]


# Valid statistical hypothesis test

When it comes to running a hypothesis test, there are only 4 possible outcomes:

|                               | Actual effect 0 | Actual effect not 0 |
| ----------------------------- | --------------- | ------------------- |
| **Test does not reject null** | True negative   | False negative      |
| **Test rejects null**         | False positive  | True positive       |

Obviously, ideally we would like to only observe true positives and true negatives,
but that turns out to be too much of an ask.

Instead, we require something far less ambitious from our hypothesis testing procedure:
a guarantee on the frequency of false positives, which statisticians have dubbed "Type 1 errors".
This is (confusingly?) called significance level,
often denoted by $$\alpha$$, and usually defaulted to $$\alpha = 0.05$$.


**Statisticians version of above table:**

|                               | Actual effect 0 | Actual effect not 0 |
| ----------------------------- | --------------- | ------------------- |
| **Test does not reject null** |                 | Type 2 error        |
| **Test rejects null**         | Type 1 error    | Power*              |

Putting this all together, we get the following procedure:

1. We decide what our null hypothesis is, in our case, that adding "hey dude," does not affect outcomes, this defines what procedure we use
2. Next we gather some data
3. Finally, we plug the data in the procedure and get a binary response: either the tests rejects the null hypothesis (no effect) or it does not.


That is, a valid hypothesis test is simply any procedure that has a guarantee on the false positive rate (or Type 1 error rate).


# The useless hypothesis test

The useless hypothesis test is really simple.
We do not even need to gather any data to run it!

Find your favorite biased coin such that it comes up heads with probability $$\alpha$$, and flip it:
if it comes up heads, then reject the null hypothesis; otherwise, do not.
Who said statistics was complicated?

Now it should be clear that this indeed is a valid hypothesis test: by construction has a false positive rate of exactly $$\alpha$$.

## Detour: Test driven development

If you have a background in software engineering, you may be familiar with test driven development.
There is an interesting connection between TDD and the hypothesis testing procedure we just described:
- From a TDD perspective, the first test we write for a hypothesis test is that the hypothesis test does not reject the null if the null is true
- The useless test we defined above is (almost) the simplest test that passes this test.[^2]


# Something something about how a rejected null says nothing


# Power



# Multiple testing


# Peeking

Special case of multiple testing


# P-value

Fancy hypothesis testing methods do not just give us a binary outcome on whether to reject the null hypothesis or not.
They also spit out a p-value.
But do not despair, our simple procedure can do the same.

First, understand what is a p-value




### Footnotes


[^1]: In most cases the null hypothesis is the hypothesis that the effect is zero, but the only thing that is special about zero is that it is convenient to use. You could definitely use some other number.
[^2]: For example, an even more basic test would never reject the null hypothesis.


