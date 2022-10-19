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


# Rejection and truth

Running experiments is a bit like fishing: most of the time you catch nothing, until all of a sudden you find a fish tugging on your fishing rod.
Similarly, most experiments show no difference between our exciting new ideas and the boring status quo.
So when we finally can reject the null hypothesis that there is no effect, in all our excitement, we love to exclaim
that indeed our treatment _is_ better than control; or maybe we restrain ourselves a bit and instead say:
it is quite likely that treatment is better than control.

Alas, an army of statisticians stands ready to flog you for your cardinal sin.
Except that we will not be making that cardinal sin now that we are armed with our useless hypothesis test.
In the case of our useless hypothesis test rejects the null hypothesis, does that give us any confidence our treatment is better than control?
Obviously not; the rejection is based on a coin flip, we did not even look at the data.
Now of course our test is an extreme case, but herein lies its usefulness: it makes it very clear what we can say, and what we cannot.
We know that our useless test, along with any other valid hypothesis test, only rejects the null hypothesis 5% of the time (assuming $$\alpha = 0.05).

# Power is useful

We have constructed a valid hypothesis testing method, and we escaped the ire of statisticians by understanding that rejection of a null hypothesis does not imply the alternative is true.
So why is it instrinsically clear that this method is useless?
Clearly the problem is that when there is an actual effect, the probability our method rejects the null hypothesis is still 5%.

It is straightforward to create a valid hypothesis test (few false positives), but for it to be useful we want it to also have few false negatives.
Statisticians say such a test has high power: when there is a true effect, the test is likely to reject the null hypothesis.
However, it is difficult to give concrete guarantees: intuitively it is much easier to detect a big effect than a small one, so all we can do is compare relative performance of tests and
pick a test that is relatively powerful against alternatives.

In general, the hope is that a powerful test is able to reject the null often when there is an actual effect, as then the reasoning becomes clear:
- Our test rejects the null
- We know that the test is unlikely to reject the null when there is no effect
- Our test is powerful, so it is likely to reject the null when there is an effect
Therefore, we feel good about our treatment, even without guarantees that it is better than control.
Our test is useless because, no matter how big the true effect is, it rejects the null only 5% of the time, or has 5% power.


This also illustrates another point clearly: the magic is not all in the statistics.
Power really is a function of three things: statistical method, effect size, and sample size.
When people say that an experiment is underpowered, they mean that the sample size is too small to detect a reasonable effect size, even for the best statistical tests.
In that case, we can think of this best statistical test as equal to our useless test: a coin flip;
and for the same reason we do not put much stock in a rejection of our useless test, neither should we in the fanciest statistical method if it is underpowered.

# Multiple testing




# Peeking

Special case of multiple testing


# P-value

Fancy hypothesis testing methods do not just give us a binary outcome on whether to reject the null hypothesis or not.
They also spit out a p-value.
But do not despair, our simple procedure can do the same.

First, understand what is a p-value.
A p-value is a random variable, that, when the null hypothesis is true, has a uniform distribution on [0, 1][^3]; any number between 0 and 1 is equally likely.
I remember being slightly shocked when I first learned about this: when there is no effect, a p-value of 0.000001 is as likely as a p-value of 0.67 you say?
Of course, neither value is likely, and in particular observing a p-value less than 0.05 under the null hypothesis is exactly (at most) 5%.

Hopefully the leap from our useless hypothesis test to useless p-value is rather short:
our useless p-value is simply a draw from a uniform distribution.
As is generally true in hypothesis testing: we can easily go from p-value to a rejection boundary by rejecting
the null hypothesis when the observed p-value is less than $\alpha$.
Indeed, our useless hypothesis test is equivalent to this approach almost by design:
the simplest way to simulate a digital biased coin is to generate a uniform random variable and threshold it:

```python
def toss_biased_coin(probability_true):
    U = random.random()
    return U < probability_true
```

Note that similar to when we define what a valid hypothesis test is, there is nothing about the behavior of a p-value when the null is not true.
Here again, we have the freedom to choose, and in partiular the goal is to make the p-value tiny when the null is false.
All in all, the insights from our useless hypothesis test around interpretability and power carry through directly to p-values as well.

# Conclusion



### Footnotes


[^1]: In most cases the null hypothesis is the hypothesis that the effect is zero, but the only thing that is special about zero is that it is convenient to use. You could definitely use some other number.
[^2]: For example, an even more basic test would never reject the null hypothesis.
[^3]: To be precise, a superuniform random variable, i.e. it [stochastically dominates][stochastic-dominance] a uniform random variable


[stochastic-dominance]: https://en.wikipedia.org/wiki/Stochastic_dominance
