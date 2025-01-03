---
layout: post
title: "Bayesian Angels and Frequentist Demons"
date: 2024-06-24 00:00:00 -0700
categories: experimentation
tags: statistics bayesian
image: angels-demons/header.webp

---


# Why many experimenters get confidence intervals wrong

Frequentist 90% confidence intervals, e.g. for the mean, cover the true mean 90% of the time.

Yet, **we cannot claim that a particular confidence interval has a 90% probability of covering the true mean.**

Put another way: The frequentist procedure generates confidence intervals that cover the truth on average, but we cannot make any claims about the coverage of a particular confidence interval that it has generated.

This trips up many aspiring statisticians, and definitely gives non-technical stakeholders a headache. Frequentist statistics can be counter-intuitive, and when it comes to confidence intervals, we are often tempted to make a probabilistic claim about a confidence interval:

“The probability that the true effect of this treatment is in the confidence interval is 90%”

In reality, the frequentist approach sees the “true treatment effect” (or true value of any parameter) as fixed - it is either inside of our confidence interval or outside of it. We often hear the phrase, “The probability that our confidence interval has covered the true value is either 0 or 1”, but even such a statement is confusing. The point is that there are no probabilistic statements to make: the confidence interval either contains or does not contain the truth. We can only make claims about our procedure, not this single observed output.

It requires a subtle understanding of the frequentist approach to recognize why we can’t make these probabilistic claims about a confidence interval - no matter how intuitive they may seem. This is frustrating to many. Allen Downey recently suggested [ignoring conventional wisdom](https://www.allendowney.com/blog/2024/04/17/what-does-a-confidence-interval-mean/) on his “Probably Overthinking It” blog:

 > This response [that we cannot make probabilistic claims] is the conventional answer to this question — it is what you find in most textbooks and what is taught in most classes. And, in my opinion, it is wrong… If you want to say that a 90% CI has a 90% chance of containing the true value, there is nothing wrong with that, philosophically.

Downey’s appealing yet flawed argument provides a great backdrop for us to better understand the philosophical differences between frequentist and Bayesian statistics. To do so, we have to talk about angels and demons, and along the way, we learn why Bayesians are happy to make probabilistic statements about the world, whereas frequentists refuse to do so.

*This article was originally published on [Eppo's Outperform](https://www.geteppo.com/blog/bayesian-angels-and-frequentist-demons).*

# The Bayesian Angels and the Frequentist Demon

Consider a not-so-hypothetical scenario where both a frequentist and a Bayesian are trying to use statistical methods to infer the true value of a parameter.

The frequentist approaches this problem as a pessimist: they believe in demons: adversaries looking to trip them up and invalidate their results by exploiting weaknesses in the method. They devise their statistical test for inferring the true value but assume that the demon will inspect the method and try to break it by choosing the true value adversarially. They build their methods around guarding against such an unfriendly (or unexpected) outcome, ensuring that whatever inferences and claims they make are robust to the demon’s choices of the true parameter.

The Bayesian, on the other hand, is an optimist. The Bayesian believes in angels and devises a statistical test with a head start based on their prior belief. The angel, being kind, will sample a true value from this prior distribution. The Bayesian statistician can then combine prior and data to obtain an accurate posterior distribution, which they can use to make probabilistic statements.

Making probabilistic claims about the true value is impossible for a frequentist because that would put them at risk of getting caught by the demon they fear. We can imagine a frequentist statistician going down this path:

- The statistician does not know the true underlying value, so he thinks: “let's consider the true underlying value to be random”
- To make that work, I need to specify some distribution over the underlying value” (e.g. the Bayesian’s prior)
- “But wait, the demon knows what prior distribution I have picked, and may, no wait, will select a true underlying value that is extremely unlikely according to the prior…” (invalidating the analysis)

At this point, the frequentist realizes that this path is a dead-end. They have to settle for procedures that work on average, even in the worst case, but cannot guarantee anything about a particular inference.

# A Bayesian argument in disguise

Let’s return to Downey’s blog post, which uses this widget analogy to show that frequentist claims make little sense:

> Suppose Frank and Betsy visit a factory where 90% of the widgets are good and 10% are defective. Frank chooses a part at random and asks Betsy, “What is the probability that this part is good?”
>
> Betsy says, “If 90% of the parts are good, and you choose one at random, the probability is 90% that it is good.”
>
> “Wrong!” says Frank. “Since the part has already been manufactured, one of two things must be true: either it is good or it is defective. So the probability is either 100% or 0%, but we don’t know which.”
>
> Frank’s argument is based on a strict interpretation of frequentism, which is a particular philosophy of probability. But it is not the only interpretation, and it is not a particularly good one. In fact, it suffers from several flaws. This example shows one of them — in many real-world scenarios where it would be meaningful and useful to assign a probability to a proposition, frequentism simply refuses to do so.

The analogy is slightly flawed. In particular, the context in the very first sentence — that 90% of widgets are working and 10% are defective — is itself a Bayesian prior!

However, the beauty of frequentist statistics is that it does not matter what the prior is: confidence intervals always have coverage *in expectation*.

A better analogy would be the following: there are widgets and with some probability $p$
they are defective. We have no clue about $p$ (why would we?), it could be anything.

Imagine we have a testing device that we could use to detect whether widgets are defective. However, the device is not entirely accurate. Let’s say it is 90% accurate (with a 10% error rate both for false positives and false negatives).

Now answer the question: Betsy selects a random widget, and the widget fails the testing device check. What is the probability that the widget is defective?

We can apply Bayes' rule to show that the probability that the widget is defective given a failed test is a function of

$$
P(\text{defective} \mid \text{fail}) = \frac{P(\text{fail} \mid \text{defective}) P(\text{defective})}{P(\text{fail})} = \frac{9p}{8p + 1}
$$

Just because the testing device is 90% accurate does not mean that the probability the widget is defective is 90%.

As is often the case, considering an edge case is clarifying: suppose none of the widgets are defective ($p=0$). There would still be a 10% chance a widget fails the test, even when there is a 0% chance it is defective.

**We can tie this back to frequentist estimation as follows:** We know that to make a probabilistic claim about whether a widget is defective, we must specify the prior probability of defects $p$. But, believing in the devil, the frequentist is not comfortable making any claims about $p$: whatever choice they make, the demon will ensure that it is the wrong choice!

# The die has been cast

It is remarkable that the frequentist statistician can generate confidence intervals with strong coverage guarantees (on average). So, why can we not extrapolate the overall coverage rate of 90% to say an individual confidence interval is 90% accurate?

It is important to remember that the confidence interval itself is random (as it is based on the observed data, which is randomly generated based on the fixed true underlying parameter(s)). Therefore, once the confidence interval is observed, there is no more randomness left.

Imagine rolling a 6-sided die. Before we roll the die, we can say that the probability of the die landing on 6 is 1/6: $P(\text{die} = 6) = 1/6$. 
However, once we observe the outcome of the roll, its fate is known: [alea iacta est](https://en.wikipedia.org/wiki/Alea_iacta_est).

This is equivalent to looking at the conditional probability: $P(\text{die} = 6 \mid \text{die} = 6) = 1$.
We can think of the confidence interval as rolling the die. We have $P(\mu \in [L, U]) = 0.9$, but $P(\mu \in [L, U] \mid [L, U] = [l, u])$ is either 0 or 1. That we can look at the die, but not at the true underlying mean $\mu$, does not change this conclusion.

# The useful useless confidence interval

Perhaps you are still unconvinced: if we do not know the fixed underlying parameter, why not just consider it random? And since confidence intervals have a 90% coverage guarantee (across many repetitions), why not say it about this particular confidence interval we just computed?

We can make the problem of assigning probabilistic claims to a generated confidence interval more stark by considering the "useful useless confidence interval"—the twin-sibling of the ["useful useless p-value"](https://www.geteppo.com/blog/the-useful-useless-p-value) we've discussed before.

First, note that to generate a valid 90% confidence interval, we have to generate an interval that covers the (fixed) underlying value with a 90% probability.

Here's a simple procedure that generates a valid confidence interval (the useful useless confidence interval):

1. Generate a uniform random number $u$ on the unit interval
2. If $u < 0.1$, output the empty interval as the confidence interval
3. Otherwise, output the interval $(-\infty, \infty)$

That is, 10% of the time, we return the empty set -- clearly, the true value is not in that set. But the other 90% of the time, we return the set of all possible values, and now the true value must be in that set. Thus, on average, we have 90% coverage. However, when the procedure returns the empty set, we know that the true value is not covered; it would clearly be silly to claim it does with a probability of 90%. It is equally silly to claim only 90% coverage when the procedure returns $(-\infty, \infty)$: the truth must be contained in that interval.

This is a contrived example to highlight the issue, but the point holds generally. In particular, the main lesson of this example should be that the generated interval leaks information about the random data (this should be obvious: the confidence interval is a function of the data), and therefore the average coverage claim cannot be applied to a particular interval: alea iacta est.

# Conclusion

Frequentist and Bayesian analyses approach statistics from very different angles. The frequentist analysis is pessimistic in nature: it ensures valid statistical inference even in the worst-case scenario. The Bayesian analysis, on the other hand, is optimistic. It relies on trusting that existing beliefs are correct, and it is, therefore, able to make more specific claims about individual tests. It should be no surprise that the claims each school makes sound very different.

Whether to be optimistic or pessimistic should be problem-specific and philosophical in nature: sometimes, it makes more sense to insist on worst-case guarantees, while in other scenarios, there might be enough prior history and subject-matter expertise to trust that the angels are kind to our beliefs.

*Special thanks to Ryan Lucht and Giorgio Martini for their thoughtful feedback on early drafts of this post.*
