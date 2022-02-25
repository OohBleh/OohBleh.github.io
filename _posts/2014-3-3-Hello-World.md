---
layout: post
title: An extremely unwinnable Slay the Spire seed
published: true
---

## Overview
In this post, I will describe how [gamerpuppy](https://github.com/gamerpuppy) and I found an unwinnable Slay the Spire seed along with a simple proof of unwinnability.  This project is motivated by ForgottenArbiter's success at finding a (probably, but still requiring full combat simulation) unwinnable seed.  Our goal was to find a seed for which it can be proven unwinnable without simulating combat, and without checking too many cases.  

## Previous progress

In a blog post on [unwinnable seeds in _Slay the Spire_](https://forgottenarbiter.github.io/Is-Every-Seed-Winnable/), ForgottenArbiter outlined a detailed approach to the problem of finding an unwinnable seed.  We take _unwinnable_ to describe any run of _Slay the Spire_ that cannot be won by any sequence of decisions allowed by the base game (ignoring glitches).  Since the random number generation of _Spire_ is determined uniquely by the run seed, the player can replay a seed repeatedly, playing cards in slightly different orders, to maintain health and eventually win the run, even if the individual decisions made are unintuitive. 

By analyzing card the optimal cases of card draw and deck shuffling, Arbiter proved that Silent's starting deck loses against Lagavulin on Ascension 18+, even with perfect card play and deck shuffling.  The argument completely ignores damage dealt to the player and focuses entirely on the lose condition: after 3 of Lagavulin's debuff turns (-2 dexterity and -2 strength), the player can no longer deal damage.  If the player reaches a fight against Lagavulin without improving the damage of their starter deck, they lose the run.  With this in mind, he formulated the following criteria when searching for unwinnable seeds.  

0. Play as Silent on Ascension 18 or higher.  
1. Forced Lagavulin elite fight on floor 6, with no shops or rest sites on the first five floors.  
2. No Neow bonuses, potion drops, or card rewards which help the player deal damage to Lagavulin.  

The "most unwinnable" seed that resulted from this search is 3LWVGX7BL which has the following properties: 

1. Neow
2. cards and potions
3. burning Laga

Unfortunately, due to the added card draw, there are decks that can be drafted by floor 6 that, with perfect shuffle RNG, can survive the fight.  For example, with a deck of blah, the following sequence of cards played can survive the fight, if realizable: blah.  The seed is believed by many to be unwinnable, but proving it would likely require simulating the fight with Lagavulin and checking all sequences of card plays for optimal shuffle manipulation.  

## The hunt for an abomination: 18ISL35FYK4

The goal of my search was to find a seed so heinous that the proof of unwinnability can fit on an index card (e.g., it requires no combat simulation).  

{:refdef: style="text-align: center;"}
![bad map wow](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/heinous-map.png){:height="433px" width="547px"}
{: refdef}

As an integer, this seed is 3,431,382,150,268,629 or ~3.4 quadrillion, but we make no claim that this is the first unwinnable seed.  This seed has the following properties: 

1. Neow only offers 1 removal, 100 gold, 250 gold for some max HP, or a swap into Cursed Key.  
2. The map forces a burning elite fight (max HP Lagavulin) on floor 6 with no shops or rest sites beforehand.  
3. The player encounters either 2 or 3 combats beforehand and the events only give 1 removal.  
4. The cards offered do not directly add any damage and are draw-neutral.  

For now, we will outline the computational effort that led to its discovery.  Scroll down further to see the proof of unwinnability.  

### BCE (Before CUDA Era)
I first got involved in this search in early January not long after finding and routing an [Ascension 20 Heart snipe seed](https://youtu.be/8jHTNGrreTw).  I had accomplished (with glitches and some very careful RNG manipulation) the first turn-1 Heart kill since the boot bug exploited [here](https://youtu.be/4knfPJyKLYY) by ForgottenArbiter was patched.  For it, I used [gamerpuppy's sts_seed_search](https://github.com/gamerpuppy/sts_seed_search), which emulates essential seed searching functions at a low level in C++.  With this approach, I am able to specify which calculations I want to make and can evaluate the results quickly.  For speedrun-related searches in the past, I have predominantly used ForgottenArbiter's [SeedSearch](https://github.com/ForgottenArbiter/SeedSearch) mod, which runs through the backend of the base game and comes with an easy to use list of seed criteria.  

I began my search on sts_seed_search with a variety of different search parameters.  Like Arbiter, I prioritized Neow bonuses, card rewards, and potions first since they are relatively fast to calculate and filter seeds efficiently.  I filtered based on the floor-0 rewards from Neow rewards of 5 (or sometimes 4) combats, assuming they were possible before the fight, and only filtered for maps with a forced floor 6 elite and no shops or rest sites beforehand.  I also filtered for "bad" potions in the first 5 (or 4) combats.  By this point, it was clear that the order of these filters mattered.  For example, generating the Act I map takes dozens of calls to random number generators, including many floating point calculations, whereas checking 5 card rewards for only "bad" cards costs, on average, fewer than 2 RNG calls.  

Suppose we have two independent filters $$\mathcal{F},\mathcal{G}$$ and consider the process of passing seeds uniformly at random though each.  If $$s,t$$ are good estimates for the times spend testing a seed against the filters, and $$p,q$$ are the probabilities of a seed passing the filters, then the expected time to test a seed against $$\mathcal{F}$$, and then $$\mathcal{G}$$ is 

{:refdef: style="text-align: center;"}
$$
	\mathbb{E}[
    	\text{time spend testing a seed} | 
        \text{ testing } \mathcal{F}
        \text{ then } \mathcal{G}
    ]
    = s + pt.  
$$
{: refdef}

Reversing the filter order exchanges $$s,t$$ and $$p,q$$.  In order to minimize the average time spent testing each seed, it follows that the correct filter order satisfies the inequality: 

{:refdef: style="text-align: center;"}
$$
	~\\
    \dfrac{s}{1-p}\leq \dfrac{t}{1-q}.  
$$
{: refdef}

The left-hand side can be thought of as a measure of the _efficiency_ of the filter $$\mathcal{F}$$: how well does it use a resource (time) to reject an unsuitable seed?  With this heuristic in mind, it becomes clear that the optimal filter order is: 

1. $$\mathcal{N}:$$ the floor 0 rewards from Neow are "bad".  
2. $$\mathcal{C}(5):$$ the first 5 card rewards do not adequately augment Silent's damage output.  
3. $$\mathcal{P}(5):$$ the potions offered from the first 5 combats also do not augment damage output.  
4. $$\mathcal{E}:$$ Lagavulin is the first elite fought.  
5. $$\mathcal{M}:$$ no shops or rest sites appear before floor 6, and floor 6 only has elite fights.  

For other considerations such as ?-nodes outcomes and boss relic swaps, we print out some information on each seed that passes filters 1 through 5, and test everything else manually.  Adjusting my search parameters many times, the hardest seed I found with this approach was 37UKXMQJQ which meets the above properties.  More precisely: 

1. $$\mathcal{N}$$: 1 removal, 100g, and 250g for max HP, or swap into Black Star.  
2. $$\mathcal{C}(5)$$: {Deflect, Prepared, Outmaneuver}, {Outmaneuver, Dodge and Roll, Blur}, {Backflip, Concentrate, Dodge and Roll}, {Concentrate, Burst, Backflip}, {Piercing Wail, Slice, Blade Dance}
3. $$\mathcal{P}(5)$$: 1 skill potion from 5th combat (only gives Calculated Gamble, Tactician, or Setup). 
4. $$\mathcal{E}$$ and $$\mathcal{M}$$ are met, and one of the floor 6 elites is the "burning" elite.  
4. Events: Winged Statue (-7 HP for 1 removal), Scrap Ooze (5 or 6 hits for Tea Set), Wheel Gremlin (always gives 1 removal), and combat. 

While I am optimistic that this seed is unwinnable due to the damage that must be taken to survive the 5th combat, proving it would require fully simulating several combats.  

It became clear to me that in order to find a suitable seed, I would need to also force the player to fight a buffed Lagavulin on floor 6.  Originally, the map constraint $$\mathcal{M}$$ allows only 1 in $$~15,000$$ seeds through.  For a forced burning elite with no shops or rest sites beforehand ($$\mathcal{B}$$), only 1 in every $$~225,000$$ seeds passes through.  

After finishing a search through the first 10 trillions seeds, making adjustments along the way, it was time for a new approach.  

### the GPU seed farm

After several helpful conversations with gamerpuppy, they sent me a version of the [CUDA](https://en.wikipedia.org/wiki/CUDA) code used for finding incredible [Pandora's Box boss swaps](https://docs.google.com/spreadsheets/d/1A3oW0tgInXa3h5azNoES4PQsTy-VdLFvDmn_CIUrbJE).  In short, CUDA gives the programmer direct access to the GPU's virtual instruction set and parallel computational capability, allowing for simultaneous computation with tens of thousands of [threads](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)).  To optimize performance, we feed in the most efficient constraints into CUDA: $$\mathcal{N}$$ and $$\mathcal{C}(5)$$.  Any seeds that pass these filters get written to a text file which can later be fed through the C++ program for further analysis.  

To understand the strength of the card reward filter, consider the following set of "unhelpful" Silent cards: 

{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/bad-silent-cards.png){:height="528px" width="567px"}
{: refdef}

With the exception of Distraction, which has a _chance_ to be helpful against Lagavulin, these cards cannot be used to deal direct damage to Lagavulin or to gain positive draw.  For simplicity, we will approximate the probability that a card reward lies in this set as 1/100.  With 5 card rewards potentially possible before floor 6, roughly 1 in every 10 billion seeds will have 5 consecutive bad card rewards.  On a modern GPU, hundreds of seeds pass both filters each second.  Even then, not enough seeds were passing through the remaining filters.  

### An unimagineably unfair map

To make the final breakthrough, I pressed harder on my combat constraints.  How likely is the player to encounter $$5$$ combats in a $$\mathcal{B}$$ map?  What if we only had 4, or even 3 combats in total before fighting a buffed Lagavulin?  Checking the first 100 billion seeds, I made a startling discovery: more than $$~2\%$$ of all $$\mathcal{B}$$ maps had a maximum of 3 combats before floor 6.  Altogether, this new constraint $$\mathcal{B}(3)$$ was satisfied by roughly 1 in every 9.17 million seeds, according to the sample.  

By relaxing the card constraints from 5 rewards to 3, the CUDA code returned a batch of about 234 million seeds from among the first 6.6 quadrillion seeds, after running for roughly 16 hours.  Four seeds MC81APL0TK, UL8LMGQV64, 1778H44QQQP, 18ISL35FYK4 emerged after being tested against properties $$\mathcal{N}, \mathcal{C}(3), \mathcal{P}(3), \mathcal{E}, $$ and $$\mathcal{B}(3)$$,  with only the last one having a suitably unhelpul boss relic swap and event pool.  

Finally, we are prepared to prove this seed is unwinnable.  

## Proof of unwinnability

Let $$\mathcal{BS} := 3,431,382,150,268,629 \text{ (also, 18ISL35FYK4})$$.  We list some useful facts about this seed.  

**Fact A**. As Silent on Ascension 18 or higher with, a normal run with seed $$\mathcal{BS}$$ manually entered has the following properties.  
1.  Neow offers 1 card removal, gold, or a boss swap into Cursed Key.  
2.  Before floor 6, the only ?-node outcomes are Scrap Ooze, Golden Idol, and The Cleric, in this order.  
3.  The first 3 combats rewards before Lagavulin are: 
   - 0 or 1 card from {Prepared, Dodge and Roll, Escape Plan}, gold, and no potion.  
   - 0 or 1 card from {Escape Plan, Outmaneuver, Prepared}, gold, and no potion.  
   - 0 or 1 card from {Prepared, Dodge and Roll, Footwork}, gold, and no potion.  
4.  Each path to floor 6 encounters either 2 combats and 3 ?-nodes, or 3 combats and 2 ?-nodes.  
5.  The only map node on floor 6 is an elite combat aganist Lagavulin with 145 HP.  

### Case 1: No boss relic swap and Outmaneuver is skipped
In this case, the maximum damage done on any given turn is attained by playing 3 copies of Strike and Neutralize.  The player may deal damage on at most 4 turns with a strength of 0, 3 with a strength of -2, and 3 with a strength of -4.  Before Lagavulin applies the 3rd debuff, the player can deal at most 
{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/no-energy-unlimited-draw.png)
{: refdef}
damage to Lagavulin.  Since Lagavulin's HP exceeds 141, the run is unwinnable in this case.  

### Case 2: No boss relic swap and Outmaneuver is taken
In this case, we note the following: the first mid-combat deck shuffle occurs at the start of turn 3.  To see this, first let 
- $$s$$ be the number of Silent starter cards (Strike, Defend, Survivor, and Neutralize) in the deck, 
- let $$k$$ be the number of copies of Escape Plan and Prepared in the deck, and 
- let $$\ell$$ be the number of other cards in the deck.  

### Case 3: Swap Ring of the Snake for Cursed Key
