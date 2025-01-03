---
layout: post
title: "Encouragement Design Experiments: When You Can't Run a True A/B Test"
date: 2024-02-20 00:00:00 -0700
categories: experimentation
tags: statistics 
image: encouragement/header.webp

---

We firmly believe you should measure the impact of all product changes, and running an experiment (or A/B test) is the most effective way to do so.

However, it is not always possible to run an experiment. For example, you might not want to withhold an important new feature from some users. In such cases, an encouragement design experiment might be an option. You can also use encouragement designs to understand the causal impact of current features that you don't want to remove entirely for a subset of users - like if presenting optional add-ons (e.g. travel insurance) increases or decreases the likelihood of the user completing a transaction. 

In this article, we explain the basics of an encouragement design and how to implement one, then go over the assumptions that need to be met in order for the design to be valid. Finally, we look at two examples that demonstrate when encouragement designs are and are not appropriate.


*This article was originally published on [Eppo's Outperform](https://www.geteppo.com/blog/encouragement-design-experiments).*


# What is an encouragement design?

<figure>
  <img src="{{site.url}}/assets/img/encouragement/diagram.png" alt="Diagram of experimental designs"/>
  <figcaption>
        We are interested in measuring the effect of changing a treatment, W, on an outcome of interest, Y. In observational studies, this is difficult because both W and Y are affected by unobserved confounders, U. In A/B testing, we get around this by randomly assigning the treatment W, severing the link between U and W. In an encouragement design study, we cannot randomize W, but we can randomize the encouragement Z, ensuring Z is independent of U. This does rely on the assumption that Z only affects Y through W (the absence of the red arrow).
  </figcaption>
</figure>



## Randomized control trial (or A/B tests)
We run experiments because we want to understand the causal effect of an intervention (e.g., making a product change) on a metric of interest. Outside of a randomized controlled trial, this is a complex problem because many unobserved factors (confounders) affect our metric of interest. However, we can decide how product changes are implemented, and thus, we can often randomize the treatment assignment. Because the randomization does not depend on the confounders, the treatment assignment is now independent of these confounders, and we can cleanly estimate the effect of the intervention. This is why A/B testing and randomized control trials (synonyms) are ubiquitous in tech and medical trials.

## Randomized encouragement design
![I am once again asking you to take the treatment](assets/img/encouragement/bernie.jpeg)


For various reasons, randomizing the treatment assignment is not always possible. A common one is that withholding a new feature from users would be a bad user experience, perhaps because the launch is promoted with a big marketing push. This does not stop us from wanting to know the impact of such product changes. 

A potential alternative is the "randomized encouragement design" experiment. In this case, we make the treatment variant available for all users, but randomly encourage some users to use the treatment. For example, we can send them an email about the new experience or show an in-app notification that nudges them to use the treatment.

### Always-takers, never-takers, and compliers
The encouragement is unlikely to affect everyone, and it is convenient to define the different possibilities:

- Some users don't need any encouragement, they will use the new feature regardless. These are the always-takers.
- Some users will not use the feature whether they are encouraged or not. These are the never-takers.
- Then, there are the compliers, these are the users that only use the new feature when they are encouraged.
- Technically, there is a last option: contrarians will only use the feature when they are not encouraged. We call these users defiers, but assume that there are no defiers.

### Estimation

Assuming that encouraged users are more likely to use the new feature, we can try to tease apart the benefits of the treatment using instrumental variable (IV) estimation, a common technique in econometrics. A wonderful blog post by the Spotify team dives deeper into the technical details [1].

It is important to note there are two possible analyses to run here. We can see the randomizing of encouragement as a standard A/B test to measure the effect of the encouragement itself. In this case, though, that is not the quantity of interest: we want to know the effect of the treatment itself, which relies on using the encouragement as an instrumental variable.

# The good, the bad, and the ugly

![The good, bad, and ugly](assets/img/encouragement/good-bad-ugly.png)

The analysis of encouragement design experiments is more complicated than the standard A/B test, as we need to leverage IV-estimation from econometrics. For the IV estimation to work (that is, give us accurate inference), we need to make additional assumptions, which can be problematic in practice. 

The main assumptions are:

- The encouragement increases the likelihood of using the treatment (no defiers)
- The encouragement assignment is independent of hidden confounders
- The encouragement affects the outcome metric of interest only through the treatment (also known as the exclusion restriction)
- The encouragement does not affect the outcome of always-takers and never-takers

## The good
The main benefit of encouragement designs is obviously that it allows us to run an experiment to try to measure impact even when direct randomization of the treatment is not possible. However, as we see below, we might not be able to measure exactly the impact we care about.

On a more technical note, it is generally difficult to convincingly argue that the instrument in an instrumental variables regression is indeed independent of confounders that affect the outcome. But since we can randomize the encouragement, this assumption is automatically satisfied in our case. 

## The bad

### Power

The “effective” sample size of an encouragement design experiment depends on the strength of the encouragement. In particular, if the encouragement only convinces a few compliers, then the variance of our IV-estimator blows up, leading to wide confidence intervals and low power. This is generally known as the weak-instruments problem in IV-regression. 

As a decent rule-of-thumb, the number of users we need in your experiment scales with the inverse of the encouragement effect. That is, if our encouragement only affects 10% of users, we need at least 10x the number of users in our encouragement design as compared to a regular experiment. Of course, it might be difficult to guess how effective the encouragement is ahead of time, which complicates a power analysis. 

However, even though it may severely hamper your ability to run encouragement designs, post-hoc we should be able to estimate the encouragement effectiveness and thereby get a sense of the (loss of) power of the experiment.

### Exclusion restriction

The IV-regression to estimate the effect in an encouragement design hinges on the exclusion restriction: that is, the encouragement itself only affects the outcome through the treatment. It should not affect the subject's behavior other than to make it more likely that they use the treatment, which in turn might affect subject's behavior.

To make this more concrete, suppose we send an email to the encouraged group to try a new feature. We hope that this makes the group more likely to use the feature, but the email itself may also encourage more usage of our application more generally. This violates the exclusion restriction assumption. To get around this, we might also send an email to the control group without mentioning the new feature.  

## The ugly
There is a bigger problem with encouragement designs that might not be immediately obvious. That is, the encouragement design only measures the effect of the intervention (e.g. the product launch) on the compliers: it is a local average treatment effect. In the case of a product launch, the never-takers presumably have a treatment effect of 0, so we can ignore them. However, we cannot ignore the always-takers; they use the treatment, but unfortunately, we cannot measure the effectiveness on this group.

Thus, we have to make the case that we live in one of the following two scenarios in order to use the results of our encouragement design:

1. We assume the treatment effect for compliers and always-takers is the same. This can be a reasonable assumption to make in some circumstances, as we explore below.
2. We do not care about measuring the effect on the always-takers, but are only interested in the impact on compliers. This is generally a harder pill to swallow: perhaps we are mostly interested in improving outcomes for marginal users (say we run a subscription business and want to avoid users canceling their subscription). In this case, we might consider always-takers as power users who are unlikely to churn anyway.

Furthermore, it may be difficult to understand who exactly the "compliers" are — this makes the estimate of the effect on the compliers more difficult to interpret.

## Why not measure the “intent to treat effect”?

As mentioned before, we can alternatively look at an encouragement design as a regular experiment where we estimate the effect of the encouragement on the outcome. This is also known as estimating the "intent to treat effect" — we have the intention to treat the encouraged users, but cannot force them.

However, in practice this is often not that relevant to us [2]. One of the main issues is the external validity of this experiment: the intent to treat effect might look very different for various methods of encouragements: sending an email might lead to a different estimate than showing a pop-up in our application. This makes these estimates hard to interpret.

# Examples
Armed with a more nuanced understanding of encouragement designs, we consider two example use cases for encouragement designs and discuss whether the approach is valid in these cases.

## Vitamin pills

![Vitamin pills illustration](assets/img/encouragement/vitamin-pills.png)

First, let's assume we are working for a public health department, and we want to estimate the effectiveness of vitamin pills in improving health outcomes for teenagers. Ideally (from an experiment perspective), we select a random sample of teenagers whom we force to take vitamin pills, and an equivalent random sample of teenagers whom we forbid to take vitamin pills. Clearly, this is untenable, and we like to keep our jobs, so we have to devise an alternative design.

Instead of forcing the treatment group to take vitamin pills, we can simply supply them with vitamin pills for free. This might encourage them to take them. Similarly, we cannot prohibit the control group of teenagers from taking vitamin pills; they are free to buy and take the vitamins on their own if they want to, but we do not give them the pills for free.

### Compliance
Recall that the first concern of encouragement design is the effectiveness of the encouragement. Does giving teenagers free vitamins actually induce them to take them? And, can we find out who complied? For the latter, presumably, we can ask the teenagers at the end of the survey and hope they answer correctly. 

### Local ATE
The other concern of encouragement design is that we only get to measure a local average treatment effect. That is, we measure the effectiveness of vitamin pills specifically on the complier group. In this scenario, we might not be too worried about it: it seems reasonable to assume that any benefits of vitamin pills are similar between the compliers and the always-takers (though one can object to this assumption).

## Ad-free premium product

![Ad-free premium product illustration](assets/img/encouragement/ad-free.png)

As a second example, consider a social media platform that wants to introduce an ad-free premium product where users can pay a monthly subscription fee to remove ads. Suppose that leadership wants to understand the impact of this change on the product but does not greenlight an experiment where the option to upgrade is withheld for some users.  

In this case, we can still run an encouragement design experiment where we notify a random subset of users about the new premium product and encourage them to sign up. This can take the form of an email, or a pop-up in the application.

### Compliance
Again, let us first consider compliance: to what extent is the encouragement effective in convincing users to pay for the premium product, whereas without the notification, they would not have. This is difficult to predict ahead of time but might be a relatively small subgroup. This means that we would need a large sample size for this experiment to make up for the limited number of compliers.

One advantage over the vitamin pills example is that it should be much easier to track compliance in this scenario.

### Exclusion restriction
In this case, it is also worthwhile to think carefully about the exclusion restriction. Sending an email to some of our users may itself encourage increased product usage, which violates this assumption. A pop-up in the application itself is less likely to break this assumption. If we decide to send an email, we should also send the control group an email to negate the effect of receiving a generic email itself.

### Local ATE
The most worrying issue is that we are measuring the effectiveness only on the group of compliers. Unlike in the vitamin case, here the assumption that compliers are similar to always-takers is much shakier. 

For example, it may well be that the always-takers are our power users, while the compliers are more marginal users of the product. Always-takers may therefore benefit much more from the premium product. However, we do not capture their benefits in an encouragement design experiment. 

Or, perhaps the always-takers have higher disposable income, and are therefore particularly valuable for advertisers. However, the encouragement design is not able to pick up on the loss of revenue for these always-takers and might give us a false impression on the impact of overall revenue.

# Conclusion
If it is not possible to run a randomized experiment, then an encouragement design might be an appealing option. However, it is no panacea, as the cost in statistical power can be steep, and measuring the effect on compliers may not be relevant.

The ideal encouragement design experiment is an experiment where everyone who receives the treatment complies, and no one in the control group complies at all. This, as you guessed, is exactly what a randomized experiment achieves.

# Acknowledgments
Thanks to Giorgio Martini and Ryan Lucht for their thoughtful comments and discussions, which form the basis of this article.

# References
[1] [Encouragement designs and instrumental variables for a/b testing](https://engineering.atspotify.com/2023/08/encouragement-designs-and-instrumental-variables-for-a-b-testing/)

[2] Causal Inference for Statistics, Social, and Biomedical Sciences: An Introduction, p. 522



