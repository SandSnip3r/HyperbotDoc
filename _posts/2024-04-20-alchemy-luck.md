---
layout: single
title:  "Silkroad Online Alchemy - Luck"
# classes: wide
toc: true
---

## Introduction

In this post, I will walk you through the some math which a Silkroad Online bot should use when choosing whether or not to use _Lucky Powders_ and _Magic Stones of Luck_ during alchemy. Coincidentally, you can also use these results for your own gameplay. It is not necessary, but is maybe helpful, to read my last post: [Silkroad Online Alchemy - Elixirs](https://sandsnip3r.github.io/HyperbotDoc/2024/03/17/alchemy-elixirs.html).

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

What about a generic formula for any plus? It's similar to the above equation, except we need to explicitly reference the expected value of the step before. Also, remember that our probability values change with each plus. $$p(n)$$ is defined as the probability of successfully going from plus $$n-1$$ to plus $$n$$. First I'll give the equation for the expected value, then I'll explain it:

$$
E_n = p(n) \cdot (1 + E_{n-1}) + (1-p(n)) \cdot (1 + E_{n-1} + E_n)
$$

$$E_n$$ is the expected number of elixirs it will require to reach plus $$n$$.

First, let's go over the success case, $$p(n) \cdot (1 + E_{n-1})$$. If we've reached plus $$(n-1)$$ and then we succeed, it will just take one more elixir to get to plus $$n$$ on top of the elixirs required to get to plus $$(n-1)$$.

Now, look the failure case, $$(1-p(n)) \cdot (1 + E_{n-1} + E_n)$$. It will take some elixirs to get to plus $$(n-1)$$, one elixir is consumed upon failure, and then we're right back in the situation that we're trying to solve for.

Let's solve the equation for $$E_n$$:

$$
\begin{aligned}
E_n &= p(n) \cdot (1 + E_{n-1}) + (1-p(n)) \cdot (1 + E_{n-1} + E_n)\\
E_n &= 1 + E_{n-1} + E_n - p(n) \cdot E_n\\
p(n) \cdot E_n &= 1 + E_{n-1}\\
E_n &= \frac{1 + E_{n-1}}{p(n)}\\
\end{aligned}
$$

Does this result intuitively make sense? The expected number of elixirs required to get to plus $$n$$ is 1 added to the expected number of elixirs to get to plus $$n-1$$ divided by our chance of success when going from $$n-1$$ to $$n$$. It depends on the number of expected elixirs for the previous plus, I think that makes sense. If $$p(n) = 1$$, then the expected number is one more than the previous, which makes sense. If $$p(n)$$ is very small, then it greatly increases the number of expected elixirs. I think this makes sense.

With a nice recursive formula, we can easily build a chart with some values:

[![]({{ site.baseurl }}//assets/images/alchemy-luck/expected-elixirs-nothing.png)]({{ site.baseurl }}//assets/images/alchemy-luck/expected-elixirs-nothing.png)
_Note that the y-axis is on a log scale._

## Luck

When using elixirs there are two items which are consumed with the elixir that can improve your chance of success. I've mentioned them before. They are _Lucky Powders_ and _Magic Stones of Luck_. These items have some cost, so they shouldn't be wasted. How do we decide when to use them?

Not considering these items, elixirs are the our only expense when enhancing an item. Since both of these items increase our chance of success, we should expect that they decrease the number of expected elixirs to achieve a given goal.

Let's walk through a little math with an example to see how many elixirs some luck will save us. Let's say we're not going to use any consumables and want to achieve +6. How many elixirs can we expect this to cost us? Remember from the last post: with just elixirs (no lucky powder or luck stone), our chance of succeeding from +5 to +6 is 17%. Using the above expected value equation and the values from the chart above, the expected number of elixirs to get +6, given the expected number of elixirs to get +5, is:
$$
E_6 = \frac{914.04 + 1}{0.17} = 5382.56
$$

Now, let's imagine that we have some magical lucky consumable that gives us an additional 10% chance of success when trying for +6. We can use the same math above to calculate how many elixirs we expect to need to get to +6. Our new probability of success is 27%. The math now works out to:
$$
E_6 = \frac{914.04 + 1}{0.27} = 3389.02
$$

That means that to get +6, using these magical lucky consumables at +5 saves us an expected $$1993.54$$ elixirs! That's amazing! But we didn't just use one of these items, we expect to fail a few times before we succeed, right? How many of these magical lucky consumables did we spend to save these elixirs? That can easily be calculated. The expected number of attempts from +5 to +6 is simply the inverse of the success rate:
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

Next, for each consumable, we'll calculate by how much the number of expected elixirs changes and compare that against the cost of the items used to decide what's best.

#### Lucky Powder
Is it worth using a lucky powder when going from +0 to +1? First we calculate the expected number of elixirs when using the lucky powder:

$$
E_1 = \frac{1}{1} = 1
$$

Then, the expected number of attempts for +1 is $$\frac{1}{1} = 1$$.

_Remember that the base chance of success from +0 to +1 is 50%. With a lucky powder, that chance of success is increased to 100%._

**Conclusion; +1 With Lucky Powder:** On average, one lucky powder saves us one elixir. Usually, elixirs are worth much more than lucky powders, so when going for +1, it is worth it to use a lucky powder.

#### Magic Stone of Luck
Is it worth using a luck stone when going from +0 to +1? First we calculate the expected number of elixirs when using the luck stone:

$$
E_1 = \frac{1}{0.55} = 1.818
$$

Then, the expected number of attempts for +1 is $$\frac{1}{0.55} = 1.818$$.

_Remember that luck stones add +5% chance of success, so going from +0 to +1 has a 55% chance of success._

**Conclusion; +1 With Luck Stone:** On average, one luck stone saves us 0.1 elixirs. Usually, luck stones are worth more than elixirs, so when going for +1, it is not worth it to use a luck stone.

#### Powder and Stone
Is it worth using both a lucky powder a luck stone when going from +0 to +1? First we calculate the expected number of elixirs when using the lucky powder and luck stone:

$$
E_1 = \frac{1}{1} = 1
$$

Then, the expected number of attempts for +1 is $$\frac{1}{1} = 1$$.

_Adding 50% + 5% to the base 50% maxes us out at 100% chance of success._

**Conclusion; +1 With Lucky Powder and Luck Stone:** This means that using one lucky powder and one luck stone saves us one elixir. Since luck stones are usually worth more than elixirs, this is not worth it. Also, this is the same savings that we got from a lucky powder alone since that already brought us to a 100% success rate.

### +2

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_prepare.gif)]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_prepare.gif)

Now, we'll walk through the same math again, now conditioned on the fact that we're deciding to only use lucky powders when going for +1. As calculated above, our expected number of elixirs to get to +1 is $$1$$, but also our expected number of lucky powders is $$1$$.

First, we'll calculate the number of expected elixirs we'll use to get to +2 without any consumables.

$$
\begin{aligned}
E_{2_{elixir}} &= \frac{1+E_{1_{elixir}}}{0.4}\\
E_{2_{elixir}} &= \frac{1+1}{0.4}\\
E_{2_{elixir}} &= \frac{2}{0.4}\\
E_{2_{elixir}} &= 5
\end{aligned}
$$

_Recall that the success rate for +2 is 40% without powders or stones._

We'll also calculate the number of expected lucky powders we'll use to get +2 while not using any consumables.

_Remember that we've already committed to using lucky powders at +1._

$$
\begin{aligned}
E_{2_{powder}} &= \frac{E_{1_{powder}}}{0.4}\\
E_{2_{powder}} &= \frac{1}{0.4}\\
E_{2_{powder}} &= 2.5
\end{aligned}
$$

When not using lucky powders or luck stones when trying for +2, we expect to use $$5$$ elixirs and $$2.5$$ lucky powders.

Next, for each combination of consumables we can use, we'll calculate by how much the number of expected elixirs & lucky powders changes.

#### Lucky Powder

Now, we calculate the expected number of elixirs and powders when using a lucky powder. Since we're using elixirs and powders together at +1 and +2, the calculation for elixirs and powders is the same.

$$
\begin{aligned}
E_2 &= \frac{1+E_1}{0.7}\\
E_2 &= \frac{1+1}{0.7}\\
E_2 &= \frac{2}{0.7}\\
E_2 &= 2.857
\end{aligned}
$$

_Recall that the success rate for +2 is 70% with a lucky powder._

**Conclusion; +2 With Lucky Powder:**

When not using lucky powders, we expect to use $$5$$ elixirs and $$2.5$$ lucky powders on average to reach +2. When using lucky powders, we expect to use $$2.857$$ elixirs and $$2.857$$ lucky powders. Our net cost is $$2.857-5 = -2.143$$ elixirs and $$2.857-2.5 = 0.357$$ lucky powders. We save $$2.143$$ elixirs at the cost of $$0.357$$ lucky powders, this is great!

#### Magic Stone of Luck

Next, we calculate the expected number of elixirs when using a luck stone:

$$
\begin{aligned}
E_{2_{elixir}} &= \frac{1+E_{1_{elixir}}}{0.45}\\
E_{2_{elixir}} &= \frac{1+1}{0.45}\\
E_{2_{elixir}} &= \frac{2}{0.45}\\
E_{2_{elixir}} &= 4.444
\end{aligned}
$$

Also, expected powders:

$$
\begin{aligned}
E_{2_{powder}} &= \frac{E_{1_{powder}}}{0.45}\\
E_{2_{powder}} &= \frac{1}{0.45}\\
E_{2_{powder}} &= 2.222
\end{aligned}
$$

*Recall that the success rate for +2 is 45% with a luck stone. Also note that we don't add 1 in the numerator for the expected number of lucky powders because we're currently calculating for only using luck stones at +2, not lucky powders.*

**Conclusion; +2 With Luck Stone:**

Our net cost is $$4.444-5 = -0.556$$ elixirs and $$2.222-2.5 = -0.278$$ lucky powders. With this success rate, we expect to make $$\frac{1}{0.45}=2.222$$ attempts. This means that each luck stone yields a net of $$\frac{-0.556}{2.222} = -0.25$$ elixirs and $$\frac{-0.278}{2.222} = -0.125$$ lucky powders. Since I told you that luck stones are worth more than elixirs, it seems like it is not worth it to use luck stones when going for +2.

#### Powder and Stone

Finally, we calculate the expected number of elixirs and powders when using both a lucky powder and luck stone. Since we're using elixirs and powders together at +1 and +2, the calculation for elixirs and powders is the same.

$$
\begin{aligned}
E_2 &= \frac{1+E_1}{0.75}\\
E_2 &= \frac{1+1}{0.75}\\
E_2 &= \frac{2}{0.75}\\
E_2 &= 2.667
\end{aligned}
$$

*Recall that the success rate for +2 is 75% with a lucky powder and luck stone.*

**Conclusion; +2 With Lucky Powder and Luck Stone:**

Our net cost is $$2.667-5 = -2.333$$ elixirs and $$2.667-2.5 = -0.167$$ lucky powders. With this success rate, we expect to make $$\frac{1}{0.75}=1.333$$ attempts. This means that for each luck stone we use, we have a net of $$\frac{-2.333}{1.333} = -1.75$$ elixirs and $$\frac{-0.167}{1.333} = -0.125$$ lucky powders. Since I told you that luck stones are worth more than elixirs and elixirs are worth more than lucky powders, it seems like it is not worth it to use a luck stone even when using a lucky powder when going for +2.

Let's do one more plus and then we'll talk about a concrete way to make a decision. For +2, we'll say that we've decided to also only use lucky powders, no luck stones.

### +3

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_success.gif)]({{ site.baseurl }}//assets/images/alchemy-luck/alcm_effect_success.gif)

First, we'll calculate the number of expected elixirs we'll use to get to +3 without using any consumables at +3.

$$
\begin{aligned}
E_{3_{elixir}} &= \frac{1+E_{2_{elixir}}}{0.3}\\
E_{3_{elixir}} &= \frac{1+2.857}{0.3}\\
E_{3_{elixir}} &= \frac{3.857}{0.3}\\
E_{3_{elixir}} &= 12.857
\end{aligned}
$$

_Recall that the success rate for +3 is 30% without powders or stones._

We'll also calculate the number of expected lucky powders we'll use to get +3 while not using any consumables at +3.

_Remember that we've already committed to using lucky powders at +1 and +2._

$$
\begin{aligned}
E_{3_{powder}} &= \frac{E_{2_{powder}}}{0.3}\\
E_{3_{powder}} &= \frac{2.857}{0.3}\\
E_{3_{powder}} &= 9.524
\end{aligned}
$$

When not using lucky powders or luck stones when trying for +3, we expect to use $$12.857$$ elixirs and $$9.524$$ lucky powders.

Next, for each combination of consumables we can use, we'll calculate by how much the number of expected elixirs & lucky powders changes.

#### Lucky Powder

Now, we calculate the expected number of elixirs and powders when using a lucky powder. Since we're using elixirs and powders together at +1, +2, and +3, the calculation for elixirs and powders is the same.

$$
\begin{aligned}
E_3 &= \frac{1+E_2}{0.5}\\
E_3 &= \frac{1+2.857}{0.5}\\
E_3 &= \frac{3.857}{0.5}\\
E_3 &= 7.714
\end{aligned}
$$

_Recall that the success rate for +3 is 50% with a lucky powder._

**Conclusion; +3 With Lucky Powder:**

When not using lucky powders, we expect to use $$12.857$$ elixirs and $$9.524$$ lucky powders on average to reach +3. When using lucky powders, we expect to use $$7.714$$ elixirs and $$7.714$$ lucky powders. Our net cost is $$7.714-12.857 = -5.142$$ elixirs and $$7.714-9.524 = -1.810$$ lucky powders. We expect to save $$5.142$$ elixirs **and** $$1.810$$ lucky powders. Since we expect this to take $$\frac{1}{0.5} = 2$$ attempts, each lucky powder saves us $$2.571$$ elixirs and $$0.905$$ lucky powders. This is amazing! We're starting to see the compounding savings that I mentioned earlier.

#### Magic Stone of Luck

Next, we calculate the expected number of elixirs when using a luck stone:

$$
\begin{aligned}
E_{3_{elixir}} &= \frac{1+E_{2_{elixir}}}{0.35}\\
E_{3_{elixir}} &= \frac{1+2.857}{0.35}\\
E_{3_{elixir}} &= \frac{3.857}{0.35}\\
E_{3_{elixir}} &= 11.020
\end{aligned}
$$

Also, expected powders:

$$
\begin{aligned}
E_{3_{powder}} &= \frac{E_{2_{powder}}}{0.35}\\
E_{3_{powder}} &= \frac{2.857}{0.35}\\
E_{3_{powder}} &= 8.163
\end{aligned}
$$

*Recall that the success rate for +3 is 35% with a luck stone. Also note that we don't add 1 in the numerator for the expected number of lucky powders because we're currently calculating for only using luck stones at +3, not lucky powders.*

**Conclusion; +3 With Luck Stone:**

Our net cost is $$11.020-12.857 = -1.837$$ elixirs and $$8.163-9.524 = -1.361$$ lucky powders. With this success rate, we expect to make $$\frac{1}{0.35}=2.857$$ attempts. This means that each luck stone yields a net of $$\frac{-1.837}{2.857} = -0.643$$ elixirs and $$\frac{-1.361}{2.857} = -0.476$$ lucky powders. Again, it's not quite worth it to use the

#### Powder and Stone

Finally, we calculate the expected number of elixirs and powders when using both a lucky powder and luck stone. Since we're using elixirs and powders together at +1, +2, and +3, the calculation for elixirs and powders is the same.

$$
\begin{aligned}
E_3 &= \frac{1+E_2}{0.55}\\
E_3 &= \frac{1+2.857}{0.55}\\
E_3 &= \frac{2}{0.55}\\
E_3 &= 7.013
\end{aligned}
$$

*Recall that the success rate for +3 is 55% with a lucky powder and luck stone.*

**Conclusion; +3 With Lucky Powder and Luck Stone:**

Our net cost is $$7.013-12.857 = -5.844$$ elixirs and $$7.013-9.524 = -2.511$$ lucky powders. With this success rate, we expect to make $$\frac{1}{0.55}=1.818$$ attempts. This means that for each luck stone we use, we have a net of $$\frac{-5.844}{1.818} = -3.214$$ elixirs and $$\frac{-2.511}{1.818} = -1.381$$ lucky powders. Lucky powders and lucky stones are helping much more than they did at +1 and +2. It's clear that the savings keep growing. At some point, the balance will tip and a lucky stone will save a huge amount of elixirs. After that, in the same way that lucky powders save lucky powders, luck stones will save luck stones.

## Making a Decision

Alright, we know how to calculate what tradeoff we're making when deciding what to use, but how do we take action based on that? How do we make the best choice?

### The Value of Luck

There are a few factors at play here. First, and hopefully most obviously, there is some ratio between elixirs and luck consumables which is important. For example, back when we were evaluating +1, we saw that 1 lucky powder saved us an average of one elixir. If it is easier to get lucky powders than elixirs, this option is profitable. The second factor is that we might have multiple viable options. As we saw at the end, with +3, it seemed beneficial to use a lucky powder on its own, but it also might have been a good idea to use both a lucky powder and luck stone. There needs to be some way to assign a value to each of the possible options.

I propose that we convert everything into elixirs, conceptually; then we will have a single value which we can maximize. We can simply calculate an expected number of saved "elixirs" and choose the option which maximizes this value.

For example, let's say that an elixir is worth 100 lucky powders and a luck stone is worth 10 elixirs. For the +3 case, we calculated that 1 lucky powder saves us $$2.571$$ elixirs and $$0.905$$ lucky powders. Using the values I just gave, we can calculate that this choice saves us $$2.571 + \frac{0.905}{100} - \frac{1}{100} = 2.570$$ elixirs-worth. We also calculated that, using both, 1 luck stone and 1 lucky powder saves us $$3.214$$ elixirs and $$1.381$$ lucky powders. Using the values I just gave, we find that this option ends up saving us $$3.214 + \frac{1.381}{100} - 1\cdot10 = -6.6772$$ elixirs-worth. Because of how valuable luck stones are, it costs us more than it saves us.

It's that simple! If you tell me the ratio of their values, we can make the optimal decision. [Here is a link to a spreadsheet](https://docs.google.com/spreadsheets/d/17-zvOfd9tQImwpyoql0aFHvUjtUn0JNiZILbqChhk2E/edit?usp=sharing) where you can play with the values yourself and see when it's best to use what. You'll need to create a copy of the sheet to modify it.

Here's an image with some common values:

[![]({{ site.baseurl }}//assets/images/alchemy-luck/alchemy-sheet.png)]({{ site.baseurl }}//assets/images/alchemy-luck/alchemy-sheet.png)

Notice the columns in the middle of the screenshot: _Use Powder_, _Use Stone_, _Use Both_. With these values, it is not until +7 that it is finally worth it to use a luck stone.

## Ratios

I gave you a nice way to calculate when to use powders and stones, but how do you figure out the actual value ratio between these items?

Typically in Silkroad Online, elixirs can only be collected as drops from monsters. Lucky powders can always be purchased from the grocery NPC with an infinite supply. Magic Stones of Luck can either be collected as drops from monsters or acquired when disassembling equipment items. What is the best way to pick a good ratio between elixirs and stones? It seems pretty tricky. The most obvious way is to simply use the market value. If an elixir sells for 250,000 gold and a luck stone sells for 10 million gold, then it's clear that the ratio is 40:1. The grocery store sells lucky powders for around 71,000 gold, so with the above elixir price, the elixir:powder ratio is about 1:3.5.

But what if there is no market? What if you're dropped in a world and you're the first person? Or what if you're shunned from anyone selling you items? How do you establish a ratio between powders and elixirs? What about elixirs and stones? For both of these, I think you can use drop rates.

First, let's think about how to do this for lucky powders. Lucky powders can be sold at the shop for gold, so there's no "drop rate" for them. There is, however, a drop rate for gold. If you can estimate how long it will take to collect enough gold to purchase a powder, you can compare that to the amount of time required to get an elixir drop. I think this ratio is sufficient for making a decision. Now, you're not comparing elixirs saved, nor gold saved, but time saved. If you stand to save 10 elixirs by buying a lucky powder, then as long as it takes less time to collect the gold than 10 elixirs, it's worth it.

Now, what can we do for luck stones? I am going to ignore the option to disassemble equipment to get luck stones and just talk about their drop rate. You might assume that we can simply deduce the ratio between these items by comparing their drop rates, but, there is a key difference here. With lucky powders, you have a choice. You can choose to spend your gold on the powders or not. When it comes to elixirs and stones, they will drop at the rate they drop and that's the end of the story. There is no option to farm one item and not the other; elixirs and luck stones drop from the same types of monsters. In this case, rather than use all of the math we've done above, the solution is quite simple. You must use them at the rate that you collect them. If you were to use stones at a ratio less than the drop ratio, you'll accumulate extra stones with no use for them. Likewise if you used them too frequently, you'd wastefully use them. A similar calculation to what we've done above can be done to figure out what plus to use stones at to match that drop ratio.

## Conclusion & Further Work

Thanks for sticking with me through all that. Hopefully this is useful to you. There are lots of intricacies which we didn't explore, such as:

- Getting luck stones from disassembling items
  - This makes luck stones slightly more plentiful, but only slightly
- _Magic Stone of Astral_
  - A premium-currency item which causes an item to only drop to +4 on failure, not +0
- Lucky avatars & Premium PLUS
  - Since these are not consumables, they're a bit out of scope here. However, you could do similar math and, based on the volume of alchemy you're doing, you could calculate if they're worth buying for their luck