---
layout: post
title:  "Experimentation with resource constraints"
date:   2020-07-07 21:26:24 -0700
categories: statistics
---

Based on work at Stitch Fix around experimentation with resource constraints and our introduction of the "virtual warehouse", Greg Novak, Dave Spiegel and I wrote [a post on the Multithreaded blog](https://multithreaded.stitchfix.com/blog/2020/11/18/virtual-warehouse/).

We introduce the post with the following thought experiment:

> Suppose a group of squirrels is considering two possible strategies to survive a harsh winter: either (A) gorge themselves on acorns in the fall and try to make it through the winter on whatever they can find, or (B) bury some acorns in the fall which will then be ready to be dug up in the winter.

> Having read a bit about data science, they might choose to A/B test these strategies, randomizing squirrels into two groups. Of course, if the squirrels are sharing the same part of the woods, we can immediately see which group will have a better chance of remaining well fed until springtime — the squirrels of group A with their “greedy” strategy (always optimizing instantaneous rate of calorie consumption) get to stuff themselves all autumn long without setting aside any nuts for the future and continue the eating through the winter by digging up the nuts from their thrifty buddies in group B who have saved acorns throughout the forest floor. Group B has a strategy that actually might be superior if it were rolled out to all squirrels, but if they share the same region as group A, their sacrificing some feasting in the fall won’t lead them to be any better off in the winter.

> In contrast, if the two experimental groups were placed in separate forests, they would get a better measure of what it would be like to roll out a strategy for all squirrels. Maybe strategy B — saving some acorns for later — is better for squirreldom than greedy strategy A, and maybe not; but the only way an A/B test could possibly reveal B as the winner is if the two groups are not competing for the same underlying resource. Thus, the randomized assignment of squirrels to the two strategies is not good enough; we also have to ensure resources of the two groups are independent.

For the entire post, head over the [Multithreaded blog](https://multithreaded.stitchfix.com/blog/2020/11/18/virtual-warehouse/).
