---
layout: single
title:  "Silkroad Online Alchemy - Luck"
# classes: wide
toc: true
---

## Introduction

In this post, I will walk you through the some math which a Silkroad Online bot should use when choosing whether or not to use _Lucky Powders_ and _Magic Stones of Luck_ during alchemy. Coincidentally, you can also use these results for your own gameplay.

## Expected number of elixirs

[![]({{ site.baseurl }}//assets/images/alchemy-luck/archemy_reinforce_recipe_weapon_b.png)]({{ site.baseurl }}//assets/images/alchemy-luck/archemy_reinforce_recipe_weapon_b.png)

With the probabilities of success in hand, we can easily calculate the expected number of elixirs required to reach any plus. I hope that I don't need to explain that starting from +0, the expected number of elixirs to reach +0, is 0. I am going to give a formula for calculating the expected number of elixirs for +1:

$$
\begin{aligned}
E_1 &= 0.5 \cdot 1 + 0.5 \cdot (1 + E_1)\\
&= 0.5 + 0.5 + 0.5 \cdot E_1\\
&= \frac{1}{0.5}\\
&= 2
\end{aligned}
$$

What is this formula saying? Remember that when trying for +1 without a lucky powder, our chance of success is 50%. There is a 50% chance our elixir succeeds. In this case, it takes us 1 elixir to get +1. That is the $$0.5*1$$ bit. Then, there's a 50% chance our elixir fails. In this case, we consume 1 elixir and end up right back in the case we're trying to solve. That is the $$0.5 \cdot (1 + E_1)$$ part of the equation. Notice that both the left and right side of the equation have $$E_1$$. A little simple algebra lets us solve for $$E_1$$ to get $$2$$.

What about a generic formula for any plus? It's similar to the above equation, except we need to explicitly reference the expected value of the step before. Also, remember that our probability values change with each plus. Similar to my last post, we'll use $$p(n)$$ which gives the probability of successfully going from plus $$n$$ to plus $$n+1$$. First I'll give the equation for the expected value, then I'll explain it:

$$
E_n = p(n) \cdot (1 + E_{n-1}) + (1-p(n)) \cdot (1 + E_{n-1} + E_n)
$$

$$E_n$$ is the expected number of elixirs it will require to reach plus $$n$$.

First, let's go over the success case, $$p(n) \cdot (1 + E_{n-1})$$. If we've reached plus $$(n-1)$$ and then we succeed, it will just take one more elixir to get to plus $$n$$ on top of the elixirs required to get to plus $$(n-1)$$.

Now, look the failue case, $$(1-p(n)) \cdot (1 + E_{n-1} + E_n)$$. It will take some elixirs to get to plus $$(n-1)$$, one elixir is consumed upon failure, and then we're right back in the situation that we're trying to solve for.

Let's solve the equation for $$E_n$$:

$$
\begin{aligned}
E_n &= p(n) \cdot (1 + E_{n-1}) + (1-p(n)) \cdot (1 + E_{n-1} + E_n)\\
E_n &= 1 + E_{n-1} + E_n - p(n) \cdot E_n\\
p(n) \cdot E_n &= 1 + E_{n-1}\\
E_n &= \frac{1 + E_{n-1}}{p(n)}\\
\end{aligned}
$$

Does this result intuitively make sense? The expected number of elixirs required to get to plus $$n$$ is 1 added to the expected number of elixirs to get to plus $$n-1$$ divided by our chance of success when going from $$n-1$$ to $$n$$. It depends on the number of expected elixirs for the previous plus, I think that makes sense. If $$p(n) = 1$$, then the expected number is the previous plus one, which makes sense. If $$p(n)$$ is very small, then it greatly increases the number of expected elixirs. I think this makes sense.

With a nice recursive formula, we can easily build a chart with some values:

[![]({{ site.baseurl }}//assets/images/alchemy-luck/expected-elixirs-nothing.png)]({{ site.baseurl }}//assets/images/alchemy-luck/expected-elixirs-nothing.png)
_Note that the y-axis is on a log scale._

## Luck

When using elixirs there are two items which are consumed with the elixir that can improve your chance of success. I've mentioned them before. They are _Lucky Powders_ and _Magic Stones of Luck_. These items are not free and infinite, so they shouldn't be wasted. How do we decide when to use them?

Not considering these items, elixirs are the our only expense when enhancing an item. Since both of these items increase our chance of success, we should expect that they decrease the number of expected elixirs to achieve a given goal.

Let's walk through a little math with an example to see how many elixirs some luck will save us. Imagine we have a +5 item and want to try for +6. How many elixirs can we expect an attempt to cost us? Remember from the last post: with just elixirs (no lucky powder or luck stone), our chance of succeeding to +6 is 17%. Using the above expected value equation and the values from the chart above, the expected number of elixirs to get +6 is:
$$
E_6 = \frac{914.04 + 1}{0.17} = 5382.56
$$

Now, let's imagine that we have some magical lucky consumable that gives us an additional 10% chance of success when trying for +6. We can use the same math above to calculate how many elixirs we expect to need to get to +6. Our new probability of success is 27%. The math now works out to:
$$
E_6 = \frac{914.04 + 1}{0.27} = 3389.02
$$

That means that when trying for +6, using these magical lucky consumables saves us an expected $$1993.54$$ elixirs! That's amazing! But we didn't just use one of these items, we expect to fail a few times before we succeed, right? How many of these magical lucky consumables did we spend to save these elixirs? That can easily be calculated. The expected number of attempts from +5 to +6 is simply the inverse of the success rate:
$$
\frac{1}{0.27} = 3.704
$$

If we divide the number of saved elixirs ($$1993.54$$) by the number of used consumables ($$3.704$$), we see that each of these magical lucky consumables saves us an expected $$538.256$$ elixirs. Still, that's really good! That means that if elixirs are worth about 100,000 gold each, it would be worth it to buy this lucky item for at most 53.8 million gold.

It is important to notice that when using lucky items at multiple levels, they reduce their own expected usage numbers. This leads to exponential savings the higher you go.

## Lucky Powder and Magic Stone of Luck

[![]({{ site.baseurl }}//assets/images/alchemy-luck/archemy_reinforce_prob_up_a.png)]({{ site.baseurl }}//assets/images/alchemy-luck/archemy_reinforce_prob_up_a.png)
[![]({{ site.baseurl }}//assets/images/alchemy-luck/archemy_magicstone_luck.png)]({{ site.baseurl }}//assets/images/alchemy-luck/archemy_magicstone_luck.png)

We're going to walk through the above math using the luck effects of _Lucky Powders_ and _Magic Stones of Luck_ and determine when is best to use them. Since making a decision depends on what choice we made at lower plusses, we'll start at +0 and work our way up. Since we have two luck items we can use, we need to check if it's worth using one, the other, both, or neither of them.

### +1

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_prepare.gif)]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_prepare.gif)

First, we'll calculate the number of expected elixirs we'll use to get to +1 without any consumables.
$$
E_1 = \frac{1}{0.5} = 2
$$

Next, for each consumable, we'll calculate by how much the number of expected elixirs improves and compare that against the cost of the items used to decide what's best.

#### Lucky Powder
Is it worth using a lucky powder when going from +0 to +1? First we calculate the expected number of elixirs when using the lucky powder:

$$
E_1 = \frac{1}{1} = 1
$$

Then, the expected number of attempts for +1 is $$\frac{1}{1} = 1$$.

_Remember that the base chance of success from +0 to +1 is 50%. With a lucky powder, that chance of success is increased to 100%._

**Conclusion:** On average, one lucky powder saves us one elixir. Usually, elixirs are worth much more than lucky powders, so when going for +1, it is worth it to use a lucky powder.

#### Magic Stone of Luck
Is it worth using a luck stone when going from +0 to +1? First we calculate the expected number of elixirs when using the luck stone:

$$
E_1 = \frac{1}{0.55} = 1.818
$$

Then, the expected number of attempts for +1 is $$\frac{1}{0.55} = 1.818$$.

_Remember that luck stones add +5% chance of success, so going from +0 to +1 has a 55% chance of success._

**Conclusion:** On average, one luck stone saves us 0.1 elixirs. Usually, luck stones are worth more than elixirs, so when going for +1, it is not worth it to use a luck stone.

#### Powder and Stone
Is it worth using both a lucky powder a luck stone when going from +0 to +1? First we calculate the expected number of elixirs when using the lucky powder and luck stone:

$$
E_1 = \frac{1}{1} = 1
$$

Then, the expected number of attempts for +1 is $$\frac{1}{1} = 1$$.

_Adding 50% + 5% to the base 50% maxes us out at 100% chance of success._

**Conclusion:** This means that using one lucky powder and one luck stone saves us one elixir. Since luck stones are usually worth more than elixirs, this is not worth it. Also, this is the same savings that we got from a lucky powder alone since that already brought us to a 100% success rate.

### +2

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_prepare.gif)]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_prepare.gif)

Now, we'll walk through the same math again, now conditioned on the fact that we're deciding to only use lucky powders when going for +1. As calculated above, our expected number of elixirs to get to +1 is $$1$$, but also our expected number of lucky powders is $$1$$. Since we use them hand-in-hand, I won't calculate them separately from now on.

First, we calculate the expected number of used elixirs and lucky powders with the base success rate.

$$
\begin{aligned}
E_2 &= \frac{1+E_1}{0.4}\\
E_2 &= \frac{1+1}{0.4}\\
E_2 &= \frac{2}{0.4}\\
E_2 &= 5
\end{aligned}
$$

_Recall that the success rate for +2 is 40% without powders or stones._

Now, we calculate the expected number of elixirs and powders when using a lucky powder:

$$
E_2 = \frac{2}{0.7} = 2.857
$$

With this success rate, we expect to make $$\frac{1}{0.7}=1.429$$ attempts. Which means that we save an average 1.5 elixirs and 1.5 lucky powders with each lucky powder. Even ignoring the elixirs, it is obvious that this is worth it. We spend one lucky powder to save an average of 1.5.

Next, we calculate the expected number of elixirs and powders when using a luck stone:

$$
E_2 = \frac{2}{0.45} = 4.444
$$

With this success rate, we expect to make $$\frac{1}{0.45}=2.222$$ attempts. Which means that we save an average 0.25 elixirs and 0.25 lucky powders with each luck stone. Since I told you that luck stones are worth more than elixirs, it seems like it is not worth it to use luck stones when going for +2.

Finally, we calculate the expected number of elixirs and powders when using both a lucky powder and luck stone:

$$
E_2 = \frac{2}{0.75} = 2.667
$$

With this success rate, we expect to make $$\frac{1}{0.75}=1.333$$ attempts. Which means that we save an average 1.75 elixirs and 1.75 lucky powders with each lucky powder and luck stone. It's starting to get a little fuzzy when it comes to making a decision about what to do. I am going to ask you to believe me when I say that using one luck stone to save 1.75 elixirs and 1.75 lucky powders is not worth it. Let's do one more plus and then we'll talk about a concrete way to make a decision. For +2, we'll say that we've decided to also only use lucky powders, no luck stones.

### +3

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_success.gif)]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_success.gif)

Alright, let's quickly zip through this one. First, the expected number of elixirs and powders:

$$
\begin{aligned}
E_3 &= \frac{1+E_2}{0.3}\\
E_3 &= \frac{1+2.857}{0.3}\\
E_3 &= \frac{3.857}{0.3}\\
E_3 &= 12.857
\end{aligned}
$$

At this point, to get to +3, without any luck consumable, it's starting to get a bit expensive. In one go, I am going to calculate the number of expected elixirs and powders when using lucky powder, luck stone, and both:

$$
E_{3_{powder}} = \frac{3.857}{0.5} = 7.714285714
$$

$$
E_{3_{stone}} = \frac{3.857}{0.35} = 11.02040816
$$

$$
E_{3_{both}} = \frac{3.857}{0.55} = 7.012987013
$$

Lucky powders and lucky stones are helping much more than they did at +1 and +2. Each powder saves us 2.571 elixirs & powders, each stone saves us 0.643, and using both saves us 3.214. It's clear that the savings keep growing. At some point, the balance will tip and a lucky stone will save a huge amount of elixirs. After that, in the same way that lucky powders save lucky powders, luck stones will save luck stones.

## Making a Decision

Alright, we know how to calculate what tradeoff we're making when deciding what to use, but how do we take action based on that? How do we make the best choice?

### The Value of Luck

There are a few factors at play here. First, and hopefully most obviously, there is some ratio between elixirs and luck consumables which is important. For example, back when we were evaluating +1, we saw that 1 lucky powder saved us an average of one elixir. If it is easier to get lucky powders than elixirs, this option is profitable. The second factor is that we might have multiple viable options. As we saw at the end, with +3, it seemed beneficial to use a lucky powder on its own, but it also might have been a good idea to use both a lucky powder and luck stone. There needs to be some way to assign a value to each of the possible options.

I propose that we convert everything into elixirs, conceptually; then we will have a single value which we can maximize. We can simply calculate an expected number of saved "elixirs" and choose the option which maximizes this value.

For example, let's say that an elixir is worth 100 lucky powders and a luck stone is worth 10 elixirs. For the +3 case, we calculated that 1 lucky powder saves us 2.571 elixirs and 2.571 lucky powders. Using the values I just gave, we can calculate that this choice saves us $$2.571 + \frac{2.571}{100} - \frac{1}{100} = 2.58671$$ elixirs-worth. We also calculated that, using both, 1 luck stone and 1 lucky powder saves us 3.214 elixirs and 3.214 lucky powders. Using the values I just gave, we find that this option ends up saving us $$3.214 + \frac{3.214}{100} - 1\cdot10 = -6.75386$$ elixirs-worth. Because of how valuable luck stones are, it costs us.

It's that simple! If you tell me the ratio of their values, we can make the optimal decision. [Here is a link to a spreadsheet](https://docs.google.com/spreadsheets/d/17-zvOfd9tQImwpyoql0aFHvUjtUn0JNiZILbqChhk2E/edit?usp=sharing) where you can play with the values yourself and see when it's best to use what. You'll need to create a copy of the sheet to modify it.

Here's an image with some common values:

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alchemy-sheet.png)]({{ site.baseurl }}//assets/images/alchemy-luck/alchemy-sheet.png)

Notice the columns in the middle of the screenshot: _Use Powder_, _Use Stone_, _Use Both_. With these values, it is not until +7 that it is finally worth it to use a luck stone.

## Ratios

I gave you a nice way to calculate when to use powders and stones, but how do you figure out the actual value ratio between these items?

Typically in Silkroad Online, elixirs can only be collected as drops from monsters. Lucky powders can always be purchased from the grocery NPC with an infinite supply. Magic Stones of Luck can either be collected as drops from monsters or aquired when disassembling equipment items. What is the best way to pick a good ratio between elixirs and stones? It seems pretty tricky. The most obvious way is to simply use the market value. If an elixir sells for 250,000 gold and a luck stone sells for 10 million gold, then it's clear that the ratio is 40:1. The grocery store sells lucky powders for around 71,000 gold, so with the above elixir price, the elixir:powder ratio is about 1:3.5.

But what if there is no market? What if you're dropped in a world and you're the first person? Or what if you're shunned from anyone selling you items? How do you establish a ratio between powders and elixirs? What about elixirs and stones? For both of these, I think you can use drop rates.

First, let's think about how to do this for lucky powders. Lucky powders can be sold at the shop for gold, so there's no "drop rate" for them. There is, however, a drop rate for gold. If you can estimate how long it will take to collect enough gold to purchase a powder, you can compare that to the amount of time required to get an elixir drop. I think this ratio is sufficient for making a decision. Now, you're not comparing elixirs saved, nor gold saved, but time saved. If you stand to save 10 elixirs by buying a lucky powder, then as long as it takes less time to collect the gold than 10 elixirs, it's worth it.

Now, what can we do for luck stones? I am going to ignore the option to disassemble equipment to get luck stones and just talk about their drop rate. You might assume that we can simply deduce the ratio between these items by comparing their drop rates, but, there is a key difference here. With lucky powders, you have a choice. You can choose to spend your gold on the powders or not. When it comes to elixirs and stones, they will drop at the rate they drop and that's the end of the story. There is no option to farm one item and not the other; elixirs and luck stones drop from the same types of monsters. In this case, rather than use all of the math we've done above, the solution is quite simple. You must use them at the rate that you collect them. If you were to use stones at a ratio less than the drop ratio, you'll acumulate extra stones with no use for them. Likewise if you used them too frequently, you'd wastefully use them. A similar calculation to what we've done above can be done to figure out what plus to use stones at to match that drop ratio.

## Conclusion & Further Work

Thanks for sticking with me through all that. Hopefully this is useful to you. There are lots of intricacies which we didn't explore, such as:

- Getting luck stones from disassembling items
  - This makes luck stones slightly more plentiful, but only slightly
- _Magic Stone of Astral_
  - A premium-currency item which causes an item to only drop to +4 on failure, not +0
- Lucky avatars & Premium PLUS
  - Since these are not consumables, they're a bit out of scope here. However, you could do similar math and, based on the volume of alchemy you're doing, you could calculate if they're worth buying for their luck