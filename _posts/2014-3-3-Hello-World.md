---
layout: post
title: An extremely unwinnable Slay the Spire seed
published: true
---

## Overview
In this post, I will describe how [gamerpuppy](https://github.com/gamerpuppy) and I found an unwinnable _Slay the Spire_ seed and, jointly with [ForgottenArbiter](https://forgottenarbiter.github.io/), find a simple simple proof of unwinnability.  This work is based on the several attempts that have been made to prove unwinnable seeds exist in _Spire_ and would not have been possible without this work and much collaboration.  

## Previous progress

In a blog post on [unwinnable seeds in _Slay the Spire_](https://forgottenarbiter.github.io/Is-Every-Seed-Winnable/), ForgottenArbiter outlined a detailed approach to the problem of finding an unwinnable seed.  We take _unwinnable_ to describe any run of _Slay the Spire_ that cannot be won by any sequence of decisions allowed by the base game (ignoring glitches).  Since the random number generation of _Spire_ is determined uniquely by the run seed, the player can replay a seed, alter their card play order (and thus the outcome of later deck shuffles) slightly different, and often unintuitive orders, to maximize health and other resources throughout the run.  

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

As an integer, this seed is $$3,431,382,150,268,629$$ or $$\sim 3.4$$ quadrillion, but we make no claim that this is the first unwinnable seed.  This seed has the following properties: 

1. Neow only offers $$1$$ removal, $$100$$ gold, $$250$$ gold for some max HP, or a swap into Cursed Key.  
2. The map forces a burning elite fight (max HP Lagavulin) on floor $$6$$ with no shops or rest sites beforehand.  
3. The player encounters either $$2$$ or $$3$$ combats beforehand and the events only give $$1$$ removal.  
4. The cards offered do not directly add any damage and are draw-neutral.  

For now, we will outline the computational effort that led to its discovery.  Scroll down further to see the proof of unwinnability.  

### BCE (Before CUDA Era)
I first got involved in this search in early January 2022 not long after finding and routing an [Ascension 20 Heart snipe seed](https://youtu.be/8jHTNGrreTw).  I had accomplished (with glitches and some very careful RNG manipulation) the first turn-1 Heart kill since the boot bug exploited [here](https://youtu.be/4knfPJyKLYY) by ForgottenArbiter was patched.  For it, I used [gamerpuppy's sts_seed_search](https://github.com/gamerpuppy/sts_seed_search), which emulates essential seed searching functions at a low level in C++.  With this library, I am able to specify which RNG calculations I want to make, and specifically how seeds are filtered.  For speedrun-related searches in the past, I have predominantly used ForgottenArbiter's [SeedSearch](https://github.com/ForgottenArbiter/SeedSearch) mod, which runs through the backend of the base game and comes with an easy to use list of seed criteria.  

Like Arbiter, I prioritized Neow bonuses, card rewards, and potions first since they are relatively fast to calculate and filter seeds efficiently.  I filtered based on the floor $$0$$ rewards from Neow, rewards of $$5$$ (or sometimes $$4$$) combats, and only filtered for maps with a forced floor $$6$$ elite and no shops or rest sites beforehand.  By this point, it was clear that the order of these filters mattered.  Each filter can be evaluated in terms of its speed (how quickly it passes or rejects seeds), and its strength (how fequently it passes or rejects a seeds).  

Suppose we have two filters $$\mathcal{F},\mathcal{G}$$ which independent in usual probabilistic sense.  If $$s,t$$ are good estimates for the times spend testing a seed against $$\mathcal{F},\mathcal{G}$$, and $$p,q$$ are the probabilities of a seed passing the filters, respectively, then the expected time to test a seed against $$\mathcal{F}$$, and then $$\mathcal{G}$$ is 

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

Reversing the filter order exchanges $$s,t$$ and $$p,q$$ in this expression.  In order to minimize the average time spent testing each seed, the optimal filter order can be found by the following calculation: 

{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/filter-efficiency.png)
{: refdef}

These fractional expressions can be thought of as a measure of the _efficiency_ of the filter $$\mathcal{F}$$, under a minor concentration hypothesis.  When $$p$$ is small, the denominator term vanishes (as is the case with map filtering, only the time spent on the filter matters.  Following some experiments, we arrive at the following order on the filters: 

1. $$\mathcal{N}:$$ the floor $$0$$ rewards from Neow are "bad".  
2. $$\mathcal{C}(5):$$ the first $$5$$ card rewards do not adequately augment Silent's damage output.  
3. $$\mathcal{P}(5):$$ the potions offered from the first $$5$$ combats also do not augment damage output.  
4. $$\mathcal{E}:$$ Lagavulin is the first elite fought.  
5. $$\mathcal{M}:$$ no shops or rest sites appear before floor $$6$$, and floor $$6$$ only has elite fights.  

For other considerations such as ?-nodes outcomes and boss relic swaps, we print out some information on each seed that passes filters $$1$$ through $$5$$, and test everything else manually.  Adjusting my search parameters many times, the hardest seed I found with this approach was 37UKXMQJQ which meets the above properties.  More precisely: 

1. $$\mathcal{N}$$: $$1$$ removal, $$100$$g, and $$250$$g for some max HP, or swap into Black Star.  
2. $$\mathcal{C}(5)$$: {Deflect, Prepared, Outmaneuver}, {Outmaneuver, Dodge and Roll, Blur}, {Backflip, Concentrate, Dodge and Roll}, {Concentrate, Burst, Backflip}, {Piercing Wail, Slice, Blade Dance}
3. $$\mathcal{P}(5)$$: $$1$$ skill potion from the $$5$$-th combat (only gives Calculated Gamble, Tactician, or Setup). 
4. $$\mathcal{E}$$ and $$\mathcal{M}$$ are met, and one of the floor $$6$$ elites is the "burning" elite.  
4. ?-nodes: Winged Statue ($$-7$$ HP for 1 removal), Scrap Ooze ($$5$$ or $$6$$ hits for Tea Set), Wheel Gremlin (always gives $$1$$ removal), and combat. 

While I am optimistic that this seed is unwinnable, proving it would require fully simulating several combats.  

It became clear to me that in order to find a suitable seed, one which does not require brute force calculations and much casework, I would need to also force the player to fight a buffed Lagavulin on floor 6.  Originally, the map constraint $$\mathcal{M}$$ allows only $$1$$ in $$\sim 15,000$$ seeds through.  For a forced burning elite with no shops or rest sites beforehand (call this filter $$\mathcal{B}$$), only 1 in every $$~225,000$$ seeds passes through.  

After finishing a search through the first $$10$$ trillions seeds, making many adjustments along the way, it was time for a new approach.  

### the GPU seed farm

After several helpful conversations with gamerpuppy, they sent me a version of the [CUDA](https://en.wikipedia.org/wiki/CUDA) code used for finding incredible [Pandora's Box boss swaps](https://docs.google.com/spreadsheets/d/1A3oW0tgInXa3h5azNoES4PQsTy-VdLFvDmn_CIUrbJE).  In short, CUDA gives the programmer direct access to the GPU's virtual instruction set and parallel computational capability, allowing for simultaneous computation with tens of thousands of [threads](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)), if ran on a state-of-the-art GPU.  To optimize performance, we feed in the most efficient constraints into CUDA: $$\mathcal{N}$$ and $$\mathcal{C}(5)$$.  Any seeds that pass these filters get written to a text file which can later be fed through the C++ program for further analysis.  

To understand the strength of the card reward filter, consider the following set of "unhelpful" Silent cards: 

{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/bad-silent-cards.png){:height="528px" width="567px"}
{: refdef}

With the exception of Distraction, which may give a poison or shiv-generating card, these cards cannot be used to deal direct damage to Lagavulin or to gain positive draw.  For simplicity, we will approximate the probability that a card reward lies in this set as $$1/100$$.  With 5 card rewards potentially possible before floor 6, roughly 1 in every 10 billion seeds will have 5 consecutive bad card rewards.  On a modern GPU, hundreds of seeds pass both filters each second.  Even then, not enough seeds were passing through the remaining filters.  

### A final tradeoff

How likely is the player to encounter $$5$$ combats in a $$\mathcal{B}$$ map?  Constraining $$5$$ combat rewards was proving to be an issue.  What if we only had $$4$$, or even $$3$$ combats in total before fighting a buffed Lagavulin?  Checking the first $$100$$ billion seeds, I discovered that as many as $$~2\%$$ of all $$\mathcal{B}$$ maps had a maximum of $$3$$ combats before floor $$6$$.  Altogether, this new constraint, call it $$\mathcal{B}(3)$$, is satisfied by $$1$$ in every $$~9.17$$ million seeds, according to the sample.  

By relaxing the card constraints from $$5$$ rewards to $$3$$ (that is, replace $$\mathcal{C}(5)$$ with $$\mathcal{C}(3)$$), the CUDA code returned a batch of about $$234$$ million seeds between $$1$$ quadrillion and $$6.6$$ quadrillion seeds, after running for roughly $$16$$ hours.  Four seeds MC81APL0TK, UL8LMGQV64, 1778H44QQQP, 18ISL35FYK4 emerged after being tested against properties $$\mathcal{N}, \mathcal{C}(3), \mathcal{P}(3), \mathcal{E}, $$ and $$\mathcal{B}(3)$$,  with only the last one having a suitably unhelpul boss relic swap and event pool.  

Finally, we are prepared to prove this seed is unwinnable.  

## Proof of unwinnability

Let $$\mathcal{BS} := 3,431,382,150,268,629$$ (also known as 18ISL35FYK4}.  We list some useful facts about this seed.  

**Fact A**. As Silent on Ascension 18 or higher with, a normal run with seed $$\mathcal{BS}$$ manually entered has the following properties.  
1.  Neow offers $$1$$ card removal, gold, or a boss swap into Cursed Key.  
2.  Before floor 6, the only ?-node outcomes are Scrap Ooze, Golden Idol, and The Cleric, in this order.  
3.  The first 3 combats rewards before Lagavulin are: 
   - $$\leq 1$$ card from {Prepared, Dodge and Roll, Escape Plan}, gold, and no potion.  
   - $$\leq 1$$ card from {Escape Plan, Outmaneuver, Prepared}, gold, and no potion.  
   - $$\leq 1$$ card from {Prepared, Dodge and Roll, Footwork}, gold, and no potion.  
4.  Each path to floor $$6$$ encounters either $$2$$ combats and $$3$$ ?-nodes, or 3 combats and $$2$$ ?-nodes.  
5.  The only map node on floor $$6$$ is an elite combat aganist Lagavulin with $$145$$ HP.  

The following argument was suggested to me by Arbiter.  To prove that $$\mathcal{BS}$$ is unwinnable, we divide the fight according to the times at which a deck shuffle occurs.  In between shuffles, the player can play at most $$5$$ Strikes and at most $$1$$ Neutralize, with damage optimized by playing the Strikes as early as possible.  In this argument, we ignore the player's health, the energy cost of the player's cards, and the metallicize buff on Lagavulin.  

For convenience, let $$k$$ be the combined number of copies of Escape Plan and Prepared in the player's deck.  We state and prove the following claims about combat on floor $$6$$.  


**Claim B.0**.  The earliest deck shuffle occurs at the start of turn $$1$$ and the next deck shuffle occurs, at the earliest, at the start of turn $$2$$.  

Since Silent's deck begins with 12 basic cards and Ascender's Bane, and by floor $$6$$, only $$2$$ card removals are possible, the player's deck has at least $$11+k$$ cards.  Furthermore, noting the relics available, at most $$7+k$$ cards are drawn on turn $$1$$.  So no other shuffle occurs during turn $$1$$, completing the proof.  


**Claim B.1**.  If the player shuffles the deck at the start of turn $$t$$ where $$t \geq 2$$, then the next deck shuffle occurs, at the earliest, at the start of turn $$t+2$$.  

Noting that by floor $$6$$, at most $$2$$ removals are possible, and since Ascender's Bane can be exhausted, it follows that combined number of cards in the player's hand, discard pile, and draw pile is always least $$10 + k$$.  Since $$t \geq 2$$, turn $$t$$ begins with the player drawing 5 cards.  Moreover, due to the limited card pool and relics, and the lack of potions, no cards can be played to increase the number of cards in-hand.  It follows that the player has at most $$5$$ cards in-hand just before the shuffle at time $$t$$.  

Since the player has at most $$5$$ cards in-hand at the start of turn $$t$$, there are at least $$5+k$$ cards in the draw pile.  On turns $$t$$ and $$t+1$$, at most $$k$$ draw cards (Escape Plan and Prepared) can be played.  It follows that the player cannot shuffle the deck again during turns $$t$$ and $$t+1$$.  


**Claim B.2**.  If the player shuffles the deck during turn $$t$$ where $$t \geq 2$$ by playing cards, then the next deck shuffle occurs, at the earliest, during turn $$t+2$$.  

As before, note that the combined number of cards in the player's hand, discard pile, and draw pile is always least $$10 + k$$.  Note also that the player has at most $$5$$ cards in-hand just before this shuffle, and at least one of these cards is a draw card.  It follows that turn $$t$$ ends with one draw card and at most $$5$$ non-draw cards in the discard pile.  This implies that the draw pile consists of at least $$10-5 = 5$$ non-draw cards at the end of turn $$t$$.  So no other shuffle occurs during turns $$t$$ and $$t+1$$, as desired.  


**Claim C**.  Suppose the player shuffles the deck on turn $$s$$.  In the time after this shuffle, but before the next shuffle, the player's damage output is bounded above by the damage dealt with the following sequence of card plays: 

- play $$5$$ Strikes and $$1$$ Neutralize on turn $$s$$ if $$s = 1$$, and 
- play $$5$$ Strikes on turn $$s$$ and $$1$$ Neutralize on turn $$s+1$$.  

Indeed, during this time, each damage card (at most $$5$$ Strikes and $$1$$ Neutralize) enters the player's hand, and thus can be played, at most once.  Starting from turn $$2$$ or higher, at most $$5$$ of these cards can be played on a given turn, and playing the Strikes on the earliest possible turn deals more damage.  


To complete the proof, consider the sequence $$1 = t(1)\leq t(2)\leq \cdots$$ of turns on which a shuffle occurs.  By Claims B.0, B.1, and B.2, it follows that $$t(2)\geq 2$$ and $$t(i) \geq t(i-1) + 2$$ for all $$i \geq 3$$ such that $$t(i)$$ is defined.  By Claim C, the maximum damage dealt to Lagavulin before the 3rd debuff is bounded above by the damage dealt by (1) playing $$5$$ Strikes and $$1$$ Neutralize on the "wake-up" turn, and (2) alternatingly on future turns playing $$5$$ Strikes on even turns and $$1$$ Neutralize on odd turns.  So the damage dealt to Lagavulin before the 3rd debuff is applied is 
{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/damage-between-shuffles.png).  
{: refdef}
Since Lagavulin's total HP equals 144, $$\mathcal{BS}$$ is unwinnable as Silent on Ascension 18 or higher.  

## Future work

With more powerful seedsearching tools at our disposal, we return to some questions considered by Arbiter in his [blog on unwinnable seeds](https://forgottenarbiter.github.io/Is-Every-Seed-Winnable/), and some other problems.  If you are interested in joining the search, take a look at the [CUDA code](https://github.com/OohBleh/cuda-the-spire) as well as [sts_seed_search](https://github.com/gamerpuppy/sts_seed_search).  

### What about an unwinnable Silent seed where the player gets Neow's Lament?  

As a running assumption during this project, we assumed that the unwinnable seed had to be manually entered.  A manually entered seed always offers the player 4 options from Neow on floor $$0$$.  If instead the player were encountered our unwinnable seed 18ISL35FYK4 by chance without having reached the Act I boss on their previous run, they would be offered Neow's Lament (or some extra max HP).  Our seed 18ISL35FYK4 would surely be winnable because the burning elite could be killing using the 3rd stack of Neow's Lament.  

To work around this, we could run the search for a longer time to find many seeds similar to 18ISL35FYK4, and select one which cannot avoid $$3$$ combats before the fight with Lagavulin (whether this is due to the seeded RNG of ?-node outcomes or the map layout.  For example, we could find such a seed for which the ?-node outcomes are  event, event, combat, combat.  Likely this search would not take more than a week on a state-of-the-art Nvidia GPU (for compatibility with CUDA).  


### What about other characters?  

As Arbiter argued, it is likely that unwinnable seeds for Ironclad and Defect exist.  However, it is likely that a proof of unwinnability would require full combat simulation to test all possible ways to manipulate shuffle RNG and card play.  Other characters' starter decks can deal damage to Lagavulin after the 3rd turn of debuffing, and exhaustively simulating more than $$10$$ turns is likely impractical.  
I believe that the following search criteria are likely to yield an unwinnable Ironclad seed: 

1.  $$\mathcal{N}$$: transform 1 (receive an unhelpful card), 100g, 250g for any downside, and a harmful or neutral boss relic swap (Black Star, Sacred Bark, etc)
2.  $$\mathcal{C}(3)$$: only offered draw-neutral skill cards and unhelpful powers
3.  $$\mathcal{P}(3)$$: no potions
4.  $$\mathcal{E}$$: first elite is **Gremlin Nob**
5.  $$\mathcal{B}(3)$$: no shops or rest sites before a floor $$6$$ burning elite

Compared to Lagavulin, Gremlin Nob... 
- deals more damage than Lagavulin in a $$3$$-turn cycle, 
- begins dealing damage on turn $$2$$ regardless of the player's decisions, 
- penalizes playing skill cards and severely limits deck manipulation and blocking, and 
- still has at least $$106$$ HP if given the max HP buff.  

If the player draws Bash too late during the first deck shuffle and has an unfavorable reshuffle with their Strike cards, they are likely unable to survive Gremlin Nob's attack for too many turns, even if they are able to enter the fight with full HP ($$80$$).  Unmitigated, Gremlin Nob deals a minimum of $$88$$ damage in the first $$6$$ turns.  