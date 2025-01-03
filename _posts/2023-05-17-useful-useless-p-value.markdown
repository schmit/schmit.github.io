---
layout: post
title: "The Useful Useless P-value"
date: 2023-05-17 00:00:00 -0700
categories: experimentation
tags: statistics 
image: useless-pvalue/header.webp

---

# The useful useless p-value

In experimentation, the **p-value** is the least understood and most misused concept, with both the most common and egregious error stating that the p-value is the probability that the null hypothesis is true. I certainly relate, as I quite often catch myself moments before making this mistake.

To better understand p-values, let’s go back to the fundamental definition of what constitutes a p-value. We can use it to construct a *useless p-value.* That is, a p-value that is useless in practice but pedagogically useful. We find many useful insights that help us understand not just the p-value, but also the statistical concept of *power*, and issues related to multiple testing and peeking.

Taking a step back, most of us have struggled with a mathematical concept at some point in our lives. I definitely have, and usually it’s been because I was looking for some magical insight that would make me understand it all. It’s easy to be stuck “waiting for the epiphany”, unable to make any progress.

However, understanding often comes not from waiting for the epiphany, but rather giving up on the search for that special insight and rather just take a definition at face value and getting your hands dirty (or more likely: a piece of paper dirty). By playing with the definition, creating examples and counter-examples, we can start to build intuition until we are no longer intimidated by this seemingly unintelligible concept or daunting definition.

*This article was originally published on [Eppo's Outperform](https://www.geteppo.com/blog/the-useful-useless-p-value).*

# Fundamental definition

To keep things semi-formal; consider the classical frequentist hypothesis test. We are testing a null hypothesis (could be anything: that the mean is zero, that there is no effect on revenue in an A/B test, etc.) against an alternative. Then the most fundamental definition of a p-value (sometimes referred to as a conservative p-value) is:

> **Definition of p-value.** The random variable $p$ is a p-value if, under the null hypothesis,  $P_0( P \leq p ) \leq p$. That is, $p$ has a super-uniform[0, 1] distribution.
> 

Let’s unpack this definition. First, a p-value is a *random variable*; in frequentist statistics, the underlying parameters are fixed (e.g. whether the mean is 0, or 1, or $e^{i\pi}$), and the data is a random variable. Since a p-value is a function of the data, it also is a random variable. This makes sense: if we were to repeat an experiment, we collect new data which leads to a different p-value.

Let us look at some examples of super-uniform random variables to build intuition:

- The constant 1 is super-uniform — it’s always larger than a draw from a uniform random variable on [0, 1].
- The uniform distribution on [0, 2] is super-uniform. One way to see this is as follows: if $U$ has a Uniform[0, 1] distribution, then $2U$ has a Uniform[0, 2] distribution, and $U < 2U$.

<aside>
💡 You might wonder, why insist on a super-uniform random variable, rather than a uniform random variable? In simple cases, we can explicitly work out the algebra and ensure the p-value has a uniform distribution under the null hypothesis. However, in more complicated examples (e.g. sequential testing) this is no longer possible and we have to fall back on bounds, which leads us to super-uniform distributions.

</aside>

Finally, note that this definition places no conditions on what happens to a p-value when the null hypothesis is not true. Officially, a p-value can take any value when the null hypothesis is not true, but in this freedom ultimately lies our power to create useful p-values. We will return to this shortly when we talk about (statistical) power.

## The useful useless p-value

Armed with the fundamental definition of a p-value, arguably the simplest p-value is the constant $1$. Clearly, this satisfies the definition of a p-value, but it is too useless even for us.

Instead, meet $u$, our useful useless p-value. $u$ is a Uniform[0, 1] distributed random variable and therefore also trivially satisfies the definition of a p-value. We can reject the null hypothesis whenever $u < \alpha$ and ensure a type 1 error rate of $\alpha$ (that is, falsely rejecting a true null hypothesis).

<aside>
💡 In fact, we could change our rejection rule to rejecting the null hypothesis whenever $0.42 < u < 0.42 + \alpha$, which still controls our type 1 error rate (assuming $\alpha < 0.58$). However, let’s keep things traditional and we reject whenever $u < \alpha = 0.05$.

</aside>

Now that we have gotten introductions out of the way, let’s focus on understanding why $u$ is both useless and useful.

# Useless

The guarantee of our new p-value is rock solid. Of course, under the null hypothesis, the probability that $u$ is less than 0.05 is exactly 0.05, which matches the guarantee we are looking for. However, this is of course also true when the null hypothesis is false. Therefore, we have a low false positive rate (as required), but also a high false negative rate.

We can create an analogy with a jacket: it needs to have two sleeves to qualify as a jacket (otherwise, we would call it a body warmer); but a good jacket is warm, waterproof, and durable. Similarly, a p-value needs to have a low false positive rate, but to be a good p-value, it also should have a low false negative rate. Statisticians refer to this as “power” (power = 1 - false negative rate). In practice, that means that whenever the null hypothesis is not true, a good p-value is as small as possible. So, $u$ is definitely a p-value, but it is a useless p-value because it has low power.

![A jacket must have sleeves is similar to a p-value must have low false positive rate](assets/img/useless-pvalue/sleeves.webp)

# Useful

While it should be clear why $u$ would not make for a useful p-value in practice, it does show its value in explaining important statistical concepts. Let’s dive into them.

## Post-selection inference

⚠️ Statistical jargon alert! ⚠️ Post-selection inference sounds intimidating, but it really shouldn’t be. In statistics 101 we learn that the proper way to run a hypothesis test is: “make up a story” and then “use the hypothesis test to verify or disprove your story”. Post-selection inference turns that upside down in a subtle way: “first use data to find a result”, and then “make up a story” — it turns out the latter is a primary cause of headaches for statisticians. 

Let’s use our $u$-value to make this more tangible. Suppose you run a speculative experiment; you think it’s pretty likely there is no effect. We generate our $u$ value and, lo and behold, $u = 0.023$; hurrah, we can reject the null hypothesis! Do you feel good about your experiment? Do you believe that you are one step closer to your Nobel prize? 

No... but, nonetheless, something interesting happened; our $u$-value controls type I error: we know it rarely rejects the null hypothesis when its true. With a $u$-value less than $0.05$, we can actually reject the null hypothesis, and yet not feel any better about ourselves, nor our experiment. Why?

It is because we realize rejecting the null hypothesis was a random fluke — in this particular case we are pretty certain we are committing a type 1 error[1]. This highlights a general truth about p-values that is easy to forget: it is the p-value that is the random, not the truthiness of your hypothesis (and if you do not like this: congratulations, you are a Bayesian 🎉). 

> **Once the p-value is generated, there are no more probabilistic statements to make.**
> 

Our useless p-value illustrates clearly that any statements such as 

- “the p-value is the probability the null hypothesis is true”, or
- “the p-value is less than $\alpha$ and thus the probability that the null hypothesis is true is less than $\alpha$

make absolutely no sense from a frequentist perspective (and again, we’re doing frequentist statistics here). And this naturally brings us to the next concept to explore.

## Underpowered studies

Even the best statistical methods are beholden to the quality of the data: one issue that plagues both academic studies as well as experiments in industry are underpowered studies: this refers to the setting where we have not gathered enough data to be able to confidently detect meaningful changes, even with state-of-the-art statistics: there is more noise than signal. 

At first sight, the issue is not apparent: the p-value controls type-1 error no matter what, so what is the big deal?

When there is not enough data to detect a signal in the noise, our p-value is basically a uniform random variable *whether or not the null hypothesis is true or false*. That is, any p-value becomes a $u$-value. And in the previous section, we discussed the issue with that arises with this: a rejection of the null hypothesis by a $u$-value is meaningless. It does not help us build confidence in our alternative hypothesis at all. 

> **For underpowered studies, every p-value looks like a $u$-value.**
> 

The problem with underpowered studies is that they are “dead on arrival” — whether or not we end up rejecting the null hypothesis makes no difference, in either scenario we have not gained any insight. Of course, when we do end up rejecting the null hypothesis, it may not be so apparent that we are at high risk of making a type 1 error as with our $u$-value. [2]

## Multiple testing

What is the probability a coin comes up heads? A simple answer would obviously be 50%; if you are a little more paranoid you might ask: “is the coin unbiased?” before answering (let’s assume it is). Still, this misses a salient aspect of the problem: how many coins are being flipped? Indeed, for a single coin it is 50%, but if I were to flip 2 coins, it becomes 75%. 

The same is true when we are running multiple hypothesis tests (e.g. when looking at multiple metrics in an experiment, or look at single metric for multiple segments). When we generate 20 $u$-values, with probability ~64% at least one of them will be less than 5%. We have a tendency to look at the smallest p-value, as that one corresponds to the “most significant” effect, which lands us in hot post-selection-inference waters. But this is now exacerbated by the fact that even some of our useless $u$-values will be small.

![Density of p-values when doing multiple comparisons](assets/img/useless-pvalue/density.svg)

The natural way to correct for this is to demand even smaller p-values when you run multiple tests — rather than being significantly smaller than a draw from a uniform distribution, you now want your p-value to be significantly smaller than *the smallest out of many uniform distributions*. Put differently: the more p-values we generate, the more strict we should be before rejecting null hypotheses ([which can be made more precise](https://multithreaded.stitchfix.com/blog/2015/10/15/multiple-hypothesis-testing/)). This increases the burden to find a sufficiently strong signal within the noise and hence makes it more likely we are running an underpowered experiment. [3]

> When running multiple tests, it is important to both be more strict in rejecting null hypotheses, as well as worry more about underpowered experiments.
> 

## Peeking

Finally, let’s cover the peeking problem. Now that we have covered multiple testing, this should be easy. We return to our $u$-value, but change the scenario slightly. Suppose, rather than gathering the data and then generating a single $u$-value (by now you might be sufficiently comfortable to skip the gathering data part), instead we generate a new $u$-value every day during the course of a month of “data gathering”. r/whatcouldgowrong? Clearly, the peeking problem is a multiple testing problem; again, with a probability of about 64% we will see at least one $u$-value less than 0.05 during the first 20 days of our experiment. Thus, if we reject the null hypothesis the first time we see a “statistically significant” $u$-value, our false positive rate will be much larger than 5%. Of course, this problem goes away when we discard all but the last $u$-value (that is, do not peek). The other option is to be more conservative in rejecting null hypotheses, but unfortunately that also means that we have to worry about Mr. Underpowered — the field of sequential analysis makes this more rigorous.

# Conclusion

`u = random.random()` is a valid p-value. While you probably want to avoid its use in practice, I hope this post has convinced you that it is a very useful pedagogical instrument that helps understand frequentist hypothesis testing. In particular, we now understand its more subtle and dangerous aspects: post selection inference, underpowered studies, multiple testing, and peeking. Most importantly, hopefully now it is now easier to avoid misrepresenting p-values with any kind of probabilistic meaning.

Finally, note that we can trivially adapt this article to be about the Useful Useless Confidence Interval instead. This is left as an exercise to the reader.

# Footnotes

[1] We have to put on our Bayesian hat to make this more precise, something I’m trying to avoid in this particular post.

[2] For a deeper dive and a Bayesian perspective, I recommend [Beyond Power Calculations: Assessing Type S (Sign) and Type M (Magnitude) Errors](https://journals.sagepub.com/doi/10.1177/1745691614551642) by Andrew Gelman and John Carlin

[3] There’s an interesting tangent here: suppose I generate 100 (valid) p-values simultaneously and I tell you that 20 p-values less than $0.05$. Then you know something interesting must be happening even without knowing anything about my statistical procedure. On the other hand, if I tell you that that there are only about 5 rejections, you are none the wiser: maybe there are no effects, or maybe I’ve been generating $u$-values and a more powerful methodology for generating p-values would have found more interesting results.
