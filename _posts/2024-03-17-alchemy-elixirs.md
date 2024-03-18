---
layout: single
title:  "Silkroad Online Alchemy - Elixirs"
# classes: wide
---

## Introduction

In this post, I am going to talk about alchemy in Silkroad Online and why it’s relevant when developing a bot which automatically plays Silkroad Online.

### Alchemy in Silkroad Online

Alchemy in Silkroad Online refers to the process of enhancing equipment using items and luck. There are a few different focus areas of alchemy in Silkroad; I am only going to talk about the _plus_ or _+_ of an item, also known as _enhancement level_ or _opt level_. There are a few different classes of items in Silkroad which are weapon, shield, protector, and accessory. The plus of a piece of equipment is the most general way to improve the quality. For weapons, a higher plus results in higher magical and/or physical damage. For protector, shield, and accessories, a higher plus results in higher magical and/or physical defense.

[![optlevel]({{ site.baseurl }}/assets/images/optlevel.png)]({{ site.baseurl }}/assets/images/optlevel.png)

_This image shows a weapon which is +7. The physical attack power and magical attack power are the only stats which benefit from a higher plus._

In order to increase the plus of an item, there are a few alchemy materials that we need. The primary material is called an _Intensifying Elixir_. There are separate elixirs for each class of item, weapon elixirs, protector elixirs, accessory elixirs, and shield elixirs. When using an elixir to enhance an item, there is a chance of success or failure. If you are successful, the equipment’s enhancement level will increase by 1. If you are unsuccessful, the equipment’s enhancement level will drop to 0. Whether you succeed or fail, the elixir will be consumed.

[![alchemy]({{ site.baseurl }}/assets/images/alchemy.gif)]({{ site.baseurl }}/assets/images/alchemy.gif)

_This gif shows an attempt to turn a +5 blade into a +6 blade using one weapon elixir. The attempt fails and the blade is reset back to +0._

Along with elixirs, there are a few other items which improve your chances of a successful enhancement. The most common item used when enhancing items is a _Lucky Powder_. Whether you succeed or fail, the lucky powder will be consumed. A more rare item which helps your chances of success is a _Magic Stone of Luck_. A Magic Stone of Luck has different application mechanics than a lucky powder. Magic Stones are applied to equipment items in a separate alchemy step and can add or increase a “magic option” on the equipment. In the case of the Magic Stone of Luck, it has a 100% chance of success when being applied to the weapon and it grants the item “1 Lucky”. The next elixir that is used on a piece of equipment with a Lucky magic option will have a higher chance of success and consume the Lucky magic option.

[![luck stone]({{ site.baseurl }}/assets/images/magic_stone_of_luck.png)]({{ site.baseurl }}/assets/images/magic_stone_of_luck.png)
[![lucky]({{ site.baseurl }}/assets/images/lucky_magic_option.png)]({{ site.baseurl }}/assets/images/lucky_magic_option.png)

_The left image shows a Magic Stone of Luck and the right image shows a blade which has the Lucky magic option applied to it._

### Automating Alchemy
Now that we understand how this portion of alchemy works, how do we automate it? This post is not going to talk about the details required within the bot to do the mechanics of alchemy. Instead, I am going to talk about the decision making a bot needs to have to automate alchemy as one of the many things a Silkroad Online bot needs to automate.

A Silkroad Online bot will spend a significant amount of its time in the fields, killing monsters and collecting loot. This is typically called _grinding_. Many different kinds of alchemy materials are collected by the bot while grinding. At some point, it would be smart for the bot to take a break from grinding and use all of its collected alchemy materials to enhance its own equipment. After all, if the bot can improve its weapon, it should be able to kill monsters more quickly resulting in an even faster collection of alchemy materials.

The big question though, is how should the bot make that decision? All existing Silkroad bots with alchemy capabilities push this problem up to the user. The user needs manually stop the bot from grinding in the field, bring the character to town, collect all of their alchemy materials into their inventory, configure some stop conditions, and start the alchemy portion of the bot. I think a bot should do all of this automatically, and make good decisions about when and what to do.

## Luck
A few factors are important when deciding what to do:
- How many of each alchemy item do you have?
- What is the enhancement level of all of your equipment?
- What is the probability of success for any given enhancement?

There are other important factors which we are going to ignore, for now. For example, maybe you're out in the field killing a very rare unique monster and now isn't the best time to go back to town to enhance your equipment.

In order to make an intelligent decision, I believe we need to know the probability of success for any given enhancement. What is the chance of success when applying an elixir? Is the chance of success the same for every enhancement level? How do we go about getting this data?

### Automated Data Collection

Luckily, I have my Silkroad bot - Hyperbot - at my disposal. The first, and most fun approach I am going to take, is to automate alchemy and collect some data. The most simple thing I want to figure out is what the rate of success of an elixir is. Later, I can figure out lucky powders and luck stones.

To collect a lot of data, I need a lot of elixirs. I decided to do my automation on a GM (game master) account in Silkroad. This is a special account type in which all characters have special server controls/commands. The server command I want is `/makeitem`. Every time I need a new elixir, I'll simply execute `/makeitem ITEM_ETC_ARCHEMY_REINFORCE_RECIPE_WEAPON_B` to spawn a weapon elixir. Upon successful execution of the command, a weapon elixir spawns at my feet.

Automation of alchemy ends up looking something like:
1. Spawn an elixir and pick it up off the ground if I don't have one
2. Apply an elixir to the blade
3. Repeat

There is a slight caveat which is that, in Silkroad, if you fail when trying +5 or higher, you have a chance of destroying your item. If that happens, I simply just spawn another blade, pick it up, and keep going.

<video controls autoplay loop muted width="672" height="384" src="{{ site.baseurl }}/assets/images/first_alchemy.mp4" frameborder="0">Your browser does not support this video.</video>
<br>

### Results

Running this for a little less than a full day, I collected 30k samples:

[![Elixir Only]({{ site.baseurl }}/assets/images/elixir_only.png)]({{ site.baseurl }}/assets/images/elixir_only.png)
_This plot shows the measured success rate (green) and the sample size (transparent purple)._

The first thing we observe is that it appears that not all enhancement levels are equally likely. The lower ones are quite a bit easier. The second thing we observe is that it's very hard to get a high plus. Using these values, you would have approximately a 0.2% chance of getting +5 starting from +0 using 5 elixirs without failing.

$$ .5058 \cdot .4007 \cdot .2999 \cdot .1913 \cdot .1768 = 0.002056 $$

### Lucky Powder

In Silkroad, Lucky Powders can be purchased from the grocery store; they're also incredibly cheap. In practice, nobody ever does alchemy without lucky powders. Since I've converged on some decent data for the success rate of elixirs, I want to figure out what effect lucky powders have on alchemy. Again, I'll automate the process of data collection. Lucky powders have a stack size of 50, which means when I issue the command to spawn one, I might as well spawn 50 to save some time. Modifying Hyperbot to also use a lucky powder along with the elixir is a trivial change.

The updated automation process looks like:
1. Spawn an elixir and pick it up off the ground if I don't have one
2. Spawn 50 lucky powders and pick them up off the ground if I don't have any
3. Apply an elixir and lucky powder to the blade
4. Repeat

After about 2000 trials, this is what my data looks like:

[![Elixir And Powder]({{ site.baseurl }}/assets/images/elixir_powder.png)]({{ site.baseurl }}/assets/images/elixir_powder.png)

### Ground Truth

Meanwhile, while I let my data collection run, I reached out to the Silkroad Online development community and asked if anyone had tried to figure out alchemy rates in Silkroad Online. Lots of people have hosted their own private servers and I've seen a few which modified the alchemy rates. Immediately, Sector1337 from the SilkRust Discord server pointed out that the success rates of elixirs and also the effects of lucky powders are stored directly in the Silkroad server database. I happen to have access to that database, so let's take a look at that data.

[![Database]({{ site.baseurl }}/assets/images/database.png)]({{ site.baseurl }}/assets/images/database.png)

Here, `ITEM_ETC_ARCHEMY_REINFORCE_RECIPE_WEAPON_B` is the weapon elixir and `ITEM_ETC_ARCHEMY_REINFORCE_PROB_UP_A_10` is the lucky powder. For anyone familiar with the game, this set of `_A` elixirs is an older set of elixirs that only works with Chinese items. The `_B` set is the typical "Intensifying Elixir" that you see in the game. These work with both Chinese and European items, the two races in Silkroad.

Anyways, let's see if we can get the luck rate out of this table. What we care about here are the params. For elixirs, `Param1` and `Param5` tell you what types of items the elixir is used for. What do `Param2`, `Param3`, and `Param4` contain? This "Desc" field contains text which seems to be some kind of description. Each of these has 4 comma-separated values. Could these params contain some information about the success rate of +1 through +12? Looking at the table schema, these Param fields are each a 32-bit integer. What if we unpacked this integer into 4 8-bit integers?

```py
def unpack(v):
  print(f'{v>>24},{(v>>16)&0xFF},{(v>>8)&0xFF},{v&0xFF}')

for v in [841489939, 286331153, 286002188]:
  unpack(v)
```
For the above code, the output is
```
50,40,30,19
17,17,17,17
17,12,12,12
```

If we interpret these as percent probabilities of success, these values very closely match our experimental results, at least for the plus values we were able to achieve with the bot.

[![Compare Elixir]({{ site.baseurl }}/assets/images/compare_elixir.png)]({{ site.baseurl }}/assets/images/compare_elixir.png)

This is great! We now have ground truth data for the success rate of any elixir. According to the community, anything higher than +12 carries the same probability as +12. Now, what about lucky powders? Lets use the same technique on the database data.

```py
for v in [840832008, 134744072, 134744072]:
  unpack(v)
```
```
50,30,20,8
8,8,8,8
8,8,8,8
```

If we assume this is just an additive bonus on top of the elixir's success rate, let's see how that aligns with our experimental data.

[![Compare Elixir and Powder]({{ site.baseurl }}/assets/images/compare_elixir_and_powder.png)]({{ site.baseurl }}/assets/images/compare_elixir_and_powder.png)

This pretty closely aligns with our data, except for +6 and higher, what's going on there? Looking back at our previous chart where we show the sample size, the sample size for these is really low, just 20, 2, 1, and 2 respectively. I think it's safe to assume that these discrepancies are due to small sample size.

### Magic Stone of Luck

Great! We have ground truth data for elixirs and lucky powders. What effect does a Magic Stone of Luck have? I also asked around in the community, and it sounds like this data is not available in the database; people generally think it's hard-coded in the gameserver. This sounds like a great excuse to go back to using Hyperbot to collect data. We only need to make a small change to the bot to add lucky to the item for each attempt. Also, given the natural difficulty of getting higher plus values, our data was suffering from low sample size at higher plus values. To account for this, any time we fail, rather than starting over again, we'll just drop our blade and spawn a new blade with a non-zero plus. For example, to spawn a +5 blade, the GM command is `/makeitem ITEM_CH_BLADE_10_A 5`. When spawning a blade, we can choose any plus up to +8. I'm not sure why GM commands limit us to +8, given that the maximum plus of an item is the max of an unsigned 8-bit int (+255), but oh well. We will cycle through 1,2,3,4,5,6,7,8 every time we spawn a new blade. This should do fine for our data distribution.

The updated automation process looks like:
1. Spawn an elixir and pick it up off the ground if I don't have one
2. Spawn 50 lucky powders and pick them up off the ground if I don't have any
3. Spawn, pick, and apply a Magic Stone of Luck if the blade does not have a Lucky magic option
4. Apply elixir and lucky powder to the blade
5. If we fail, drop the blade, spawn a new one, and pick it up
6. Repeat

Unfortunately, applying a Magic Stone of Luck takes as much time as applying a regular elixir, so my rate of data collection halves. To combat this, I will run multiple Hyperbots concurrently; 11 characters all doing alchemy at the same time.

<iframe width="560" height="315" src="https://www.youtube.com/embed/MgMQJmgcF2g?si=Revji4v2QiGsLHfG" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

After running this for a couple days, I collected about 50,000 samples. Let's see what effect we see with a Magic Stone of Luck compared to the ground truth of just an elixir and lucky powder.

[![Elixir Powder and Stone]({{ site.baseurl }}/assets/images/elixir_powder_and_stone.png)]({{ site.baseurl }}/assets/images/elixir_powder_and_stone.png)

Apart from the highest plus values, where we have comparatively low sample size, it looks like the luck stone applies a flat additive 5% chance of success. How do we verify this? We can keep sampling, but we'll never know for sure. Is there some way that we can mathematically state some confidence about a result?

#### Wilson Score Interval

In statistics, a _binomial proportion confidence interval_ is a confidence interval for the probability of success calculated from the outcome of a series of success–failure experiments. The [Wilson score interval](https://en.wikipedia.org/wiki/Binomial_proportion_confidence_interval#Wilson_score_interval) is an improvement over the normal approximation interval. Using this concept, we can plug in a desired confidence, say 95%, and be able to calculate a lower bound and an upper bound for the likely actual value, based on the volume of data that we've collected. When we have little data, this confidence interval will be very wide. As we collect more data, this confidence interval will narrow down. As we approach infinite samples, the confidence interval should converge on the actual value. I've created a nice visualization using python to show how this confidence interval converges as more data is collected.

[![Confidence Interval]({{ site.baseurl }}/assets/images/ci_plot.gif)]({{ site.baseurl }}/assets/images/ci_plot.gif)

- The blue data points show the ground truth success rate for elixirs.
- The green data points show the ground truth success rate for elixirs when combined with lucky powders.
- The black points show my hypothesis, which is a flat 5% added to the chance of success.
- The red points show the observed success rate.
- The red shaded region shows the range of possible success rates that we have a 95% confidence of. As you can see, the observed success rate and confidence interval very closely match the 5% additive bonus hypothesis.

### Other Sources of Luck

In Silkroad, there are two other sources of luck. The first is a premium subscription ("Premium PLUS") which adds "5% increase in success rate".

[![Premium]({{ site.baseurl }}/assets/images/premium.png)]({{ site.baseurl }}/assets/images/premium.png)

Quickly collecting 70k samples, it does seem to be the case that Premium does what it says.

[![Premium Chart]({{ site.baseurl }}/assets/images/premium_chart.png)]({{ site.baseurl }}/assets/images/premium_chart.png)

The second additional source of luck is an avatar with the Lucky magic option. Avatar dresses in Silkroad can have up to 4 magic attributes granted on them.

[![Empty Avatar]({{ site.baseurl }}/assets/images/empty_avatar.png)]({{ site.baseurl }}/assets/images/empty_avatar.png)[![Lucky Avatar]({{ site.baseurl }}/assets/images/avatar_with_lucky.png)]({{ site.baseurl }}/assets/images/avatar_with_lucky.png)

In order to exaggerate the effects when collecting data, I created an avatar with 40% added luck.

[![40% Avatar]({{ site.baseurl }}/assets/images/avatar_with_40_luck.png)]({{ site.baseurl }}/assets/images/avatar_with_40_luck.png)

Again, with just 70k samples, the data does seem to reflect a flat additive increase in success rate.

[![40% Avatar Chart]({{ site.baseurl }}/assets/images/40_avatar_chart.png)]({{ site.baseurl }}/assets/images/40_avatar_chart.png)

## Intelligence

We had to uncover a lot of details to finally be able to make an attempt to answer our initial question. Our initial question is:

> While Hyperbot is grinding and collecting loot in Silkroad, at what point should it make an attempt to improve its own gear?

If our weapon is +0, then it seems obvious that we should immediately apply an elixir to our weapon, because we can only go up from +0. What if our weapon is +6? We probably shouldn't attempt +7 if we only have 1 elixir because there's a very high chance that we'll fail and end up at +0. We should instead wait until we've collected a bunch of elixirs, but how many?

### Mathematical Analysis

I am going to explain a mathematical model that we can use to make the above decision. To keep the calculations simple, I am going to use the success rates from the database of an elixir combined with a lucky powder, since this is the most common case. Luck stones can be rare, and avatars and premium subscriptions cost real money.

[![Elixir and Powder Rates]({{ site.baseurl }}/assets/images/elixir_and_powder_rates.png)]({{ site.baseurl }}/assets/images/elixir_and_powder_rates.png)
_Here are our rates again, to easily reference._

First, it will be easiest to work with an example. If we have a +1 weapon and have 3 elixirs, what is the probability that we can make it to +2?

- If we succeed on our first attempt, then we're at +2 and we're done. There's a 70% chance of this.
- If we fail, then we're back at +0 with 2 elixirs. Now we have to get to +2 without failing, which is $$100\% \cdot 70\% = 70\%$$.

Our overall chance of success is
$$ 70\% + 30\% \cdot 70\% = 91\% $$

I'm not sure if you caught the pattern there, but let's define this generically. We want to calculate our chance of getting plus $$x$$ with $$N$$ elixirs, starting from plus $$y$$. The calculation can be done recursively using the following function $$f$$. We'll use the function $$p(z)$$, which is the probability of achieving plus $$z$$ from the above table. There are two possibilities:
- We succeed and we're at one higher plus than we were previously
  - In this case, we recurse with our current plus being one higher $$y+1$$, having one fewer elixir $$N-1$$, and having the same goal $$x$$
- We fail and we're back to 0
  - In this case, we recurse with our current plus at $$0$$, having one fewer elixir $$N-1$$, and having the same goal $$x$$

$$ f(y, N, x) = \\
\begin{align}
\begin{aligned}
&p(y+1) \cdot f(y+1, N-1, x) +\\
(1-&p(y+1)) \cdot f(0, N-1, x)
\end{aligned}
\end{align}
$$

Using the example above with this equation, we get:

$$ f(1, 3, 2) = \\
\begin{align}
\begin{aligned}
0.70 \cdot &f(2, 2, 2) +\\
0.30 \cdot &f(0, 2, 2)
\end{aligned}
\end{align}
$$

The first evaluation of `f`, `f(2,2,2)`, evaluates to 100%, because the target plus is equal to the goal plus.
The second evaluation of `f`, `f(0,2,2)`, recurses as follows:

$$ f(0, 2, 2) = \\
\begin{align}
\begin{aligned}
1.00 \cdot &f(1, 1, 2) +\\
0.00 \cdot &f(0, 1, 2)
\end{aligned}
\end{align}
$$

The first evaluation of `f`, `f(1,1,2)`, recurses as follows:

$$ f(1, 1, 2) = \\
\begin{align}
\begin{aligned}
0.70 \cdot &f(2, 0, 2) +\\
0.30 \cdot &f(0, 0, 2)
\end{aligned}
\end{align}
$$

Finally, the first evaluation of `f`, `f(2,0,2)`, evaluates to 100%, because the target plus is equal to the goal plus.
The second evaluation of `f`, `f(0,0,2)`, evaluates to 0%, because there are no elixirs left. Fully expanded out, the equation is

$$ f(1, 3, 2) = \\
\begin{align}
\begin{aligned}
&0.70 \cdot 1.00 +\\
&0.30 \cdot 1.00 \cdot
\left(
\begin{aligned}
&0.70 \cdot 1.00 +\\
&0.30 \cdot 0.00
\end{aligned}
\right)
\end{aligned}
\end{align} \\
= 0.91
$$

Now that we have the general formula, lets plot the data for a few different goals and see how many elixirs it will take to achieve each goal.

[![Elixirs Required]({{ site.baseurl }}/assets/images/elixirs_required.png)]({{ site.baseurl }}/assets/images/elixirs_required.png)

Each of these labeled points shows how many elixirs are required to have at least a 90% probability of achieving the goal.
- To have at least a 90% chance of achieving +2, starting from +0, we need at least 10 elixirs
- To have at least a 90% chance of achieving +3, starting from +0, we need at least 47 elixirs
- To have at least a 90% chance of achieving +4, starting from +0, we need at least 205 elixirs
- To have at least a 90% chance of achieving +5, starting from +0, we need at least 840 elixirs
- To have at least a 90% chance of achieving +6, starting from +0, we need at least 3384 elixirs

If we plot this requirement for a few more goals, we see that it requires exponentially more elixirs to reach a higher plus. This is unsurpising.

[![Elixirs Required 2]({{ site.baseurl }}/assets/images/elixirs_required_2.png)]({{ site.baseurl }}/assets/images/elixirs_required_2.png)

### Strategy

The above calculations are based on a specific strategy for achieving a goal. When calculating the probability of getting to a specified goal, we assume that we're going to spend all of our elixirs to get that goal, even if improbable. When actually playing Silkroad this isn't the best strategy. This strategy is likely to leave the player with a low plus weapon.

Instead, the player should stop once they achieve as high of a plus as they think they can with the elixirs they have. This will usually mean that the player stops early and has some leftover elixirs. The player then collects elixirs until they have enough to start trying again.

### The Final Decision

With the probabilities of success figured out, how do we use this to make a decision with Hyperbot? Thanks to a great insight from a friend and coworker, [Reed](https://github.com/reedwm), we can simply compare the upside against the downside. Specifically, we will compare the probability of our plus being greater than or equal to our current plus against the probability of it ending at a lower plus. We'll call this ratio $$R$$.

We'll use the same $$p(z)$$ as above which gives the probability of achieving plus $$z$$. First, we define a function $$b(x, N)$$ as follows

$$
b(x,N) = 
\begin{cases} 
True &
\begin{aligned}
&p(x+1) + \\
(1-&p(x+1)) \cdot g(0, N-1, x)
\end{aligned} \geq R \\
False & otherwise
\end{cases}
$$

This returns $$True$$ if the sum of the following is at least our specified ratio $$R$$:
- The probability of success when trying for $$x+1$$
- The probability of returning to $$x$$ after failing $$x+1$$

As you see referenced in $$b$$, we also define a new function $$g(y, N, x)$$. This function $$g$$ returns the probability of getting plus $$x$$ with $$N$$ elixirs, starting from plus $$y$$, but now while following our strategy. The function $$g$$ is defined as follows

$$
g(y, N, x) = 
\begin{cases}
1 & y == x \\
0 & y + N < x \\
0 & b(y, N) == False \\
\begin{aligned}
&p(y+1) \cdot g(y+1, N-1, x) +\\
(1-&p(y+1)) \cdot g(0, N-1, x)
\end{aligned}
& otherwise
\end{cases}
$$

_Note that $$g$$ depends on $$b$$, which means the probability of achieving the given goal will be 0 if the strategy decides not to take action._

The cases of the above function are:
- $$y==x$$, which means we've reached our goal
- $$y+N < x$$, which means we do not have enough elixirs to reach our goal
- $$b(y,N) == False$$, which means that the bot would not take action
- The last case sums the probability of succeeding and continuing towards our goal with the probability of failing and continuing towards our goal.

#### Simple Example

Let's walk through this with a simple example. We will define our ratio, $$R$$, to be 0.8. This means that we will only take action if we're 4x (0.8 vs 0.2) as likely to achieve a plus greater than or equal to our current plus than we are to settle with a lower plus. Let's say that our current weapon is +1 and we have 2 elixirs. Should we try for +2? As stated above, we're using the luck rates with lucky powders, which means the probability of going from +0 to +1 is 100% and the probability of going from +1 to +2 is 70%.

Evaluating $$b(1,2)$$ we get the following:
$$
0.7 + 0.3 \cdot g(0,1,1)
$$

This call to $$g$$ is asking "if we're at +0 and have 1 elixir, what is the probability that we get to +1?" Or, equivalently, "what is the probability of getting back to where we were before we failed +2?"

Since the probability of going from +0 to +1 is 100%, $$g(0,1,1)$$ returns 1.0. Our evaluation of $$g$$ within $$b(1,2)$$ results in $$1.0$$ which is greater than $$0.8$$, so we will attempt a higher plus.

If we think about this for a second, what did we expect we should do? The probability of success is $$0.7$$, which is lower than our $$R$$ of $$0.8$$. But we're not just comparing our chances of success, we're also accounting for our chance to get back where we are. We have a $$70\%$$ chance of success and a $$30\%$$ chance of failure. If we do fail, we have one elixir left and a 100% chance of getting to +1. In this case, of course we should try.

#### Practical Example

With these equations, we can write a program to find the minimum number of elixirs required to try for the next plus. Let's say we're at +5 and we want a 4x chance of not ending at a lower plus. How many elixirs do we need before trying?

```py
probabilities = [ 1.0, 0.7, 0.5, 0.27, 0.25, 0.25 ]
R = 0.8
current = 5
goal = 6

@functools.lru_cache(maxsize=None)
def takeAction(current, numElixirs):
  return probabilities[current] + (1-probabilities[current])*probability(0,numElixirs-1,current) >= R

@functools.lru_cache(maxsize=None)
def probability(current, numElixirs, goal):
  if current == goal:
    return 1.0
  if current + numElixirs < goal:
    return 0.0
  if not takeAction(current, numElixirs):
    return 0.0
  return probabilities[current]  * probability(current+1, numElixirs-1, goal) + \
    (1 - probabilities[current]) * probability(0,         numElixirs-1, goal)

count = 0
while True:
  result = probability(current, count, goal)
  if result > 0.0:
    print(f'{count} minimum elixirs required.')
    print(f'Probability of achieving goal is {result*100}%.')
    print(f'Probability of getting back to +{current} is {probability(0, count-1, current)*100}%.')
    break
  count += 1
```

The output of this program is
```
230 minimum elixirs required.
Probability of achieving goal is 25.0%.
Probability of getting back to +5 is 73.46588637693769%.
```

Summing up these probabilities, we get $$0.25 + 0.75 \cdot 0.7346588637693769 = 0.800994147827032675$$. Which means that we have an $$80.099\%$$ chance of ending at +5 or +6 with our starting 230 elixirs.

## Conclusion

With this, we have one single configurable value which determines whether we take action or keep collecting. We just need to pick a good value for the ratio $$R$$. Your chosen value of $$R$$ depends on your preference for risk. $$R = 0$$ means that you will always try for the next plus as soon as you get an elixir. A $$R$$ close to $$1$$ means that you will stockpile many elixirs before any attempt at approving your weapon.

Traditionally, Silkroad bots, if they even have some form of auto alchemy, expose tons of knobs and configurations for what kind of alchemy the bot should do. With this, we narrow down the amount of configuration to at most one value. We also enable Hyperbot to choose when to do alchemy, rather than relying on the user decide when to stop training and start alchemy.

### Further Work

This post skipped some important open questions, such as:
- Do different items have different chances of success?
  - _I think the answer is no. People who have done reverse engineering of the gameserver have not found such a field in the_ `CGItem` _structure._
- Can we use reverse engineering to figure out the effect of Magic Stone of Luck?
- If you have just a few luck stones, when should you use them?
  - How does that affect the calculation?
- What if your character is about to level up and get a higher level weapon anyways?
  - How do you not waste resources enhancing an item which is soon to be obsolete?
  - Maybe you got a new weapon which you're not high enough level to use yet. Maybe this weapon is instead a better one to invest your resources in?
- What if you have multiple weapons? Which one do you enhance? Always the weakest one? What if one is your primary and the other is an only occasionally-used secondary?