---
layout: post
title: 'An extremely unwinnable Slay the Spire seed, and how to find more'
published: true
---
## Overview
In this post, I will describe a collaborative effort that proved that unwinnable _Slay the Spire_ seeds exist.  This project is joint work with [gamerpuppy](https://github.com/gamerpuppy) and [ForgottenArbiter](https://forgottenarbiter.github.io/) and is based on the several attempts that have been made to prove unwinnability for seeds in _Slay the Spire_.  

## Previous progress

In a blog post on [unwinnable seeds in _Slay the Spire_](https://forgottenarbiter.github.io/Is-Every-Seed-Winnable/), ForgottenArbiter outlined a detailed approach to the problem of finding an unwinnable and shared a [branch of SeedSearch](https://github.com/ForgottenArbiter/SeedSearch/tree/hard-seeds) which looks for these seeds.  We take _unwinnable_ to describe any run of _Slay the Spire_ that cannot be won by any sequence of decisions allowed by the base game (ignoring glitches).  Since the random number generation of _Spire_ is determined uniquely by the run seed, the player can replay a seed, alter their card play order (and thus the outcome of later deck shuffles), and make unintuitive decisions, to maximize health and other resources throughout the run.  

By analyzing optimal cases of card draw and deck shuffling, Arbiter proved that Silent's starting deck loses against Lagavulin on Ascension $$18+$$.  The argument completely ignores damage dealt to the player and focuses entirely on the lose condition: after $$3$$ of Lagavulin's debuff turns ($$-2$$ dexterity and $$-2$$ strength), the player can no longer deal damage.  If the player reaches a fight against Lagavulin without directly or indirectly improving the damage of their starter deck, they lose the run.  With this in mind, he formulated the following criteria when searching for unwinnable seeds.  

1. Play as Silent on Ascension $$18$$ or higher.  
2. Forced Lagavulin elite fight on floor $$6$$, with no shops or rest sites on the first five floors.  
3. No Neow bonuses, potion drops, or card rewards which help the player deal damage to Lagavulin.  

The "most unwinnable" seed that resulted from this search is 3LWVGX7BL which has the following properties: 

1. Neow Options: upgrade a card, $$100$$g, remove $$2$$ cards (take $$15$$ damage), swap for Cursed Key
2. Card and potions before floor $$6$$: 
    - {Bane, Outmaneuver, Deflect}
    - {Outmaneuver, Accuracy, Footwork} and Essence of Steel
    - {Deflect, Outmaneuver, Setup} and Fruit Juice
    - {Dodge and Roll, Accuracy, Tactician} 
3.  Lagavulin is the first elite.  
4.  Forced burning elite (with the metallicizing buff) on floor $$6$$ with no shops or rest sites beforehand.  
5.  ?-nodes are useless except for a possibly useful card transform (Adrenaline, etc).  

It is likely that this seed is unwinnable.  However, it can not be ruled out as winnable without detailed information about the shuffle RNG of the fight against Lagavulin.  In _Slay the Spire_, the random number generator which controls shuffle is instantiated at the start of each combat, and it depends only on the seed and the floor number.  By playing cards in a different order on the first cycle through the deck, the player can manipulate the order in which cards are drawn on later cycles.  

For example, it is possible to reach Lagavulin on this seed with a deck which draws as follows: 
- Strike+, Bane+, Defend, Strike, Survivor, Defend, Defend, 
- Footwork, Neutralize, Ascender's Bane, Defend, Strike, 
- Defend, Strike, Strike, Outmaneuver, _shuffle..._

By playing cards in a different order, the player controls the order of cards in their draw pile, and thus can manipulate how they are shuffled in the next deck cycle.  ForgottenArbiter has shown that with this deck and perfect shuffling on future turns, the player is able to deal exactly $$113$$ damage to Lagavulin (which starts with $$112$$ HP) before the debuffs reduce all damage to $$0$$.  Whether this shuffle can be realized by the game's RNG has not been determined.  With access to mid-turn deck shuffling from Adrenaline as well, this becomes much more difficult to analyze.  

## The hunt for an abomination: 18ISL35FYK4

The goal of my search was to find a seed so heinous that the proof of unwinnability can fit on an index card (e.g., it requires no combat simulation).  

{:refdef: style="text-align: center;"}
![bad map wow](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/heinous-map.png){:height="433px" width="547px"}
{: refdef}

As an integer, this seed is $$3,431,382,150,268,629$$ or $$\sim 3.4$$ quadrillion, but we make no claim that this is the first seed (as a positive integer) which is unwinnable.  This seed has the following properties: 

1. Neow only offers $$1$$ removal, gold, or a swap into Cursed Key.  
2. The map forces a burning elite fight (max HP Lagavulin) on floor $$6$$ with no shops or rest sites beforehand.  
3. The player encounters $$2$$ or $$3$$ combats and the ?-nodes are mostly unhelpful.  
4. The cards offered do not directly add any damage and are draw-neutral.  

For now, we will outline the computational effort that led to its discovery.  Scroll down further to see the proof of unwinnability.  

### BCE (Before CUDA Era)
I first got involved in this search in early January 2022 not long after finding and routing an [Ascension 20 Heart snipe seed](https://youtu.be/8jHTNGrreTw).  I had accomplished (with glitches and some very careful RNG manipulation) the first turn-$$1$$ Heart kill since the boot bug exploited [here](https://youtu.be/4knfPJyKLYY) by ForgottenArbiter was patched.  For it, I used [gamerpuppy's sts_seed_search](https://github.com/gamerpuppy/sts_seed_search), which emulates essential seed searching functions at a low level in C++.  With this library, I am able to specify which RNG calculations I want to make, and how seeds are filtered.  For speedrun-related searches in the past, I have predominantly used ForgottenArbiter's [SeedSearch](https://github.com/ForgottenArbiter/SeedSearch) mod, which runs through the backend of the base game and comes with an easy to use list of seed criteria.  

I filtered based on the floor $$0$$ rewards from Neow, rewards of $$5$$ (or sometimes $$4$$) combats, and only filtered for maps with a forced floor $$6$$ elite and no shops or rest sites beforehand.  The filter order has a large impact on the rate at which seeds are tested.  
Suppose we have two filters $$\mathcal{F},\mathcal{G}$$ which are independent in an appropriate probabilistic sense.  If $$s,t$$ are good estimates for the times spend testing a seed against $$\mathcal{F},\mathcal{G}$$, and $$p,q$$ are the probabilities of a seed passing the filters, respectively, then the expected time to test a seed against $$\mathcal{F}$$, and then $$\mathcal{G}$$ is 

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

Reversing the filter order exchanges $$s,t$$ and $$p,q$$ in this expression.  In order to minimize the average time spent testing each seed, the optimal filter order can be found by the following observation: 

{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/filter-efficiency.png){:height="46px" width="327px"}
{: refdef}

These fractional expressions can be thought of as measures of the _efficiency_ of the filters $$\mathcal{F}$$ and $$\mathcal{G}$$, under a minor concentration hypothesis.  When $$p$$ is small, the denominator term is arbitrarily close to $$1$$ (as is the case with map filtering), and only the time spent on the filter matters.  Following some experiments, we arrive at the following order on the filters: 

1. $$\mathcal{N}:$$ the floor $$0$$ rewards from Neow are "unhelpful".  
2. $$\mathcal{C}(5):$$ the first $$5$$ card rewards do not adequately augment Silent's damage output.  
3. $$\mathcal{P}(5):$$ the potions offered from the first $$5$$ combats also do not augment damage output.  
4. $$\mathcal{E}:$$ Lagavulin is the first elite fought.  
5. $$\mathcal{M}:$$ no shops or rest sites appear before floor $$6$$, and floor $$6$$ only has elite fights.  

For other considerations such as ?-nodes outcomes and boss relic swaps, we print out some information on each seed that passes filters $$1$$ through $$5$$, and test everything else manually, or with Arbiter's SeedSearch.  Adjusting parameters many times, the hardest seed I found with this approach was 37UKXMQJQ which meets the above properties.  More precisely: 

1. $$\mathcal{N}$$: $$1$$ removal, $$100$$g, and $$250$$g for some max HP, or swap into Black Star.  
2. $$\mathcal{C}(5)$$: 
- {Deflect, Prepared, Outmaneuver}, 
- {Outmaneuver, Dodge and Roll, Blur}, 
- {Backflip, Concentrate, Dodge and Roll}, 
- {Concentrate, Burst, Backflip}, and 
- {Piercing Wail, Slice, Blade Dance}.  
3. $$\mathcal{P}(5)$$: $$1$$ skill potion from the $$5$$-th combat (always {Calculated Gamble, Tactician, Setup}). 
4. $$\mathcal{E}$$ and $$\mathcal{M}$$ are met, and one of the floor $$6$$ elites is the "burning" elite.  
4. ?-nodes: Winged Statue ($$-7$$ HP for $$1$$ removal), Scrap Ooze ($$5$$ or $$6$$ hits for Tea Set), Wheel Gremlin (always gives $$1$$ removal), and combat. 

While I am optimistic that this seed is unwinnable, proving it would require fully simulating several combats.  

It became clear to me that in order to find a suitable seed, one which does not require brute force calculations and much casework, I would need to also force the player to fight a buffed Lagavulin on floor $$6$$, as is the case with Arbiter's seed.  Originally, the map constraint $$\mathcal{M}$$ allows only $$1$$ in $$\sim 15,000$$ seeds through.  For a forced burning elite with no shops or rest sites beforehand (call this filter $$\mathcal{B}$$), only $$1$$ in every $$\sim 225,000$$ seeds passes through.  This search ran through the first $$10$$ trillions seeds in one to two weeks of frequent breaks and changes to search parameters,.  It was time for a new approach.  

### the GPU seed farm

After several helpful conversations with gamerpuppy, they sent me a version of the [CUDA](https://en.wikipedia.org/wiki/CUDA) code used for finding incredible [Pandora's Box boss swaps](https://docs.google.com/spreadsheets/d/1A3oW0tgInXa3h5azNoES4PQsTy-VdLFvDmn_CIUrbJE).  In short, CUDA gives the programmer direct access to the GPU's virtual instruction set and parallel computational capability, allowing for simultaneous computation with tens of thousands of [threads](https://en.wikipedia.org/wiki/Multithreading_(computer_architecture)), if ran on a state-of-the-art GPU.  

To optimize performance, we feed in the most efficient constraints into CUDA: $$\mathcal{N}$$ and $$\mathcal{C}(5)$$ and take special care to minimize slowdown caused by [branching](https://en.wikipedia.org/wiki/Branch_(computer_science)).  Any seeds that pass these filters get written to a text file which can later be fed through the C++ program for further analysis.  

To understand the strength of the card reward filter, consider the following set of Silent cards: 

{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/bad-silent-cards.png)
{: refdef}

With the exception of Distraction, which may give a poison or shiv-generating card, these cards cannot be used to deal direct damage to Lagavulin or to gain positive draw.  For simplicity, we will approximate the probability that a card reward lies in this set as $$1/100$$.  With $$5$$ card rewards potentially possible before floor $$6$$, roughly $$1$$ in every $$10$$ billion seeds will have $$5$$ consecutive bad card rewards.  Even with several optimizations, the rate at which seeds passed $$\mathcal{N}$$ and $$\mathcal{C}(5)$$ was insufficient to find enough seeds passing the remaining filters $$\mathcal{P}(5)$$, $$\mathcal{E}$$, and most importantly, $$\mathcal{B}$$.  

### A final tradeoff

As a running hypothesis, I had been planning for the worst case scenario where the player has access the maximum number of combats.  Having seen hundreds of $$\mathcal{B}$$ maps, I knew that a fair number of them had only $$4$$-combat paths.  If we could relax the constraint on card rewards, many more seeds would be printed to file from the CUDA code, and maybe enough of those seeds would then pass the remaining $$3$$ filters.  

Checking the first $$100$$ billion seeds, I made a discovery.  As many as a third of all $$\mathcal{B}$$-maps failed to have $$5$$-combat paths, and strikingly, $$~2\%$$ of all $$\mathcal{B}$$ maps had only $$3$$ combats before the burning elite!  Altogether, this new constraint, call it $$\mathcal{B}(3)$$, is satisfied by $$1$$ in every $$~9.17$$ million seeds, according to the sample.  While this may not seem like a lot, changing the nature of the filters in this way effectively swaps two $$1$$-in-$$100$$ filters for a single $$1$$-in-$$50$$ filter.  It also reduces a great deal of branching in the CUDA code by cutting down the scope of the main loop in the card reward filter.  

After relaxing the card constraints from $$5$$ rewards to $$3$$ (that is, replace $$\mathcal{C}(5)$$ with $$\mathcal{C}(3)$$), the CUDA code returned a batch of about $$234$$ million seeds between $$1$$ quadrillion and $$6.6$$ quadrillion seeds, after only $$16$$ hours of runtime on a modern GPU.  Four seeds MC81APL0TK, UL8LMGQV64, 1778H44QQQP, 18ISL35FYK4 emerged after being tested against properties $$\mathcal{N}, \mathcal{C}(3), \mathcal{P}(3), \mathcal{E}, $$ and $$\mathcal{B}(3)$$,  with only the last one having a suitably unhelpul boss relic swap and event pool.  

Finally, we are prepared to prove this seed is unwinnable.  

## Proof of unwinnability

Let $$\mathcal{BS} := 3,431,382,150,268,629$$ (also known as 18ISL35FYK4}.  We list some useful facts about this seed.  

**Fact A**. As Silent on Ascension $$18$$ or higher with, a normal run with seed $$\mathcal{BS}$$ manually entered has the following properties.  
1.  Neow offers $$1$$ card removal, gold, or a boss swap into Cursed Key.  
2.  Before floor $$6$$, the only ?-node outcomes are World of Goop, Golden Idol, and The Cleric, in this order.  
3.  The first $$3$$ combats rewards before Lagavulin are: 
   - {Prepared, Dodge and Roll, Escape Plan}, gold, and no potion.  
   - {Escape Plan, Outmaneuver, Prepared}, gold, and no potion.  
   - {Prepared, Dodge and Roll, Footwork}, gold, and a Block Potion.  
4.  Each path to floor $$6$$ encounters either $$2$$ combats and $$3$$ ?-nodes, or $$3$$ combats and $$2$$ ?-nodes.  
5.  The only map node on floor $$6$$ is an elite combat aganist Lagavulin with $$144$$ HP.  

The following argument was suggested to me by Arbiter.  To prove that $$\mathcal{BS}$$ is unwinnable, we divide the fight according to the times at which a deck shuffle occurs.  In between shuffles, the player can play at most $$5$$ Strikes and at most $$1$$ Neutralize, with damage optimized by playing the Strikes as early as possible.  In this argument, we ignore the player's health, the energy cost of the player's cards, and the block on Lagavulin before it is awakened.  

For convenience, let $$k$$ be the combined number of copies of Escape Plan and Prepared in the player's deck.  We state and prove the following claims about combat on floor $$6$$.  


**Claim B.0**.  The first deck shuffle occurs at the start of turn $$1$$ and the next deck shuffle occurs, at the earliest, at the start of turn $$2$$.  

Since Silent's deck begins with $$12$$ basic cards and Ascender's Bane, and by floor $$6$$, only $$2$$ card removals are possible, the player's deck has at least $$11+k$$ cards.  Furthermore, noting the relics available, at most $$7+k$$ cards are drawn on turn $$1$$.  So no other shuffle occurs during turn $$1$$, completing the proof.  


**Claim B.1**.  If the player shuffles the deck at the start of turn $$t$$ where $$t \geq 2$$, then the next deck shuffle occurs, at the earliest, at the start of turn $$t+2$$.  

Noting that by floor $$6$$, at most $$2$$ removals are possible, and since Ascender's Bane can be exhausted, it follows that combined number of cards in the player's hand, discard pile, and draw pile is always least $$10 + k$$.  Since $$t \geq 2$$, turn $$t$$ begins with the player drawing $$5$$ cards.  Moreover, due to the limited card pool and relics, and the available potions, no cards can be played to increase the number of cards in-hand.  It follows that the player has at most $$5$$ cards in-hand just before the shuffle at time $$t$$.  

Since the player has at most $$5$$ cards in-hand at the start of turn $$t$$, there are at least $$5+k$$ cards in the draw pile.  On turns $$t$$ and $$t+1$$, at most $$k$$ draw cards (Escape Plan and Prepared) can be played.  It follows that the player cannot shuffle the deck again during turns $$t$$ and $$t+1$$.  


**Claim B.2**.  If the player shuffles the deck during turn $$t$$ where $$t \geq 2$$ by playing cards, then the next deck shuffle occurs, at the earliest, during turn $$t+2$$.  

As before, note that the combined number of cards in the player's hand, discard pile, and draw pile is always least $$10 + k$$.  Note also that the player has at most $$5$$ cards in-hand just before this shuffle, and at least one of these cards is a draw card.  It follows that turn $$t$$ ends with one draw card and at most $$5$$ non-draw cards in the discard pile.  This implies that the draw pile consists of at least $$10-5 = 5$$ non-draw cards at the end of turn $$t$$.  So no other shuffle occurs during turns $$t$$ and $$t+1$$, as desired.  


**Claim C**.  Suppose the player shuffles the deck on turn $$t$$.  In the time after this shuffle, but before the next shuffle, the player's damage output is bounded above by the damage dealt with the following sequence of card plays: 

- play $$5$$ Strikes and $$1$$ Neutralize on turn $$t$$ if $$t = 1$$, and 
- play $$5$$ Strikes on turn $$t$$ and $$1$$ Neutralize on turn $$t+1$$.  

Indeed, during this time, each damage card (at most $$5$$ Strikes and $$1$$ Neutralize) enters the player's hand, and thus can be played, at most once.  Starting from turn $$2$$ or higher, at most $$5$$ of these cards can be played on a given turn, and playing the Strikes on the earliest possible turn deals more damage.  


To complete the proof, consider the sequence $$1 = t(1)\leq t(2)\leq \cdots$$ of turns on which a shuffle occurs.  By Claims B.0, B.1, and B.2, it follows that $$t(2)\geq 2$$ and $$t(i) \geq t(i-1) + 2$$ for all $$i \geq 3$$ such that $$t(i)$$ is defined.  By Claim C, the maximum damage dealt to Lagavulin before the 3rd debuff is bounded above by the damage dealt by (1) playing $$5$$ Strikes and $$1$$ Neutralize on the "wake-up" turn, and (2) alternatingly on future turns playing $$5$$ Strikes on even turns and $$1$$ Neutralize on odd turns.  So the damage dealt to Lagavulin before the $$3$$-rd debuff is applied is 
{:refdef: style="text-align: center;"}
![please work prose](https://raw.githubusercontent.com/OohBleh/OohBleh.github.io/master/_posts/damage-between-shuffles.png).  
{: refdef}
Since Lagavulin's total HP equals $$144$$, $$\mathcal{BS}$$ is unwinnable as Silent on Ascension $$18$$ or higher.  

## Future work

With more powerful seedsearching tools at our disposal, we return to some questions considered by Arbiter in his [blog on unwinnable seeds](https://forgottenarbiter.github.io/Is-Every-Seed-Winnable/), and some other problems.  If you are interested in joining the search, take a look at the [CUDA code](https://github.com/OohBleh/cuda-the-spire) as well as [sts_seed_search](https://github.com/gamerpuppy/sts_seed_search).  

### What if the player gets Neow's Lament?  

As a running assumption during this project, we assumed that the unwinnable seed had to be manually entered.  A manually entered seed always offers the player $$4$$ options from Neow on floor $$0$$.  If instead the player were encountered our unwinnable seed 18ISL35FYK4 by chance without having reached the Act I boss on their previous run, they would be offered Neow's Lament (or some extra max HP).  Our seed 18ISL35FYK4 would surely be winnable because the burning elite could be killing using the $$3$$-rd stack of Neow's Lament.  

To work around this, we could run the search for a longer time to find many seeds similar to 18ISL35FYK4, and select one which cannot avoid $$3$$ combats before the fight with Lagavulin (whether this is due to the seeded RNG of ?-node outcomes or the map layout.  For example, we could find such a seed for which the ?-node outcomes are  event, event, combat, combat.  Likely this search would not take more than a week on a state-of-the-art Nvidia GPU (for compatibility with CUDA).  


### What about other characters?  

As Arbiter argued, it is likely that unwinnable seeds for Ironclad and Defect exist.  However, it is also likely that a proof of unwinnability would involve the shuffle RNG of fights, and perhaps a full combat simulation to test sequences of mid-combat decisions for survivability.  Other characters' starter decks can deal damage to Lagavulin after the $$3$$-rd turn of debuffing, and exhaustively simulating more than $$10$$ turns is likely impractical.  I believe that the following search criteria are likely to yield an unwinnable Ironclad seed: 

1.  $$\mathcal{N}$$: transform $$1$$ (receive an unhelpful card), $$100$$g, $$250$$g for any downside, and a harmful or neutral boss relic swap (Black Star, Sacred Bark, etc)
2.  $$\mathcal{C}(3)$$: only offered draw-neutral skill cards and unhelpful powers
3.  $$\mathcal{P}(3)$$: no potions
4.  $$\mathcal{E}$$: first elite is **Gremlin Nob**
5.  $$\mathcal{B}(3)$$: no shops or rest sites before a floor $$6$$ burning elite

Compared to Lagavulin, Gremlin Nob... 
- deals more damage in a $$3$$-turn cycle, 
- begins dealing damage on turn $$2$$ regardless of the player's decisions, 
- penalizes playing skill cards and severely limits deck manipulation and blocking, 
- still has at least $$106$$ HP if given the max HP buff.  

Additionally, fights against Gremlin Nob typically last fewer turns than both Lagavulin and Three Sentries and are thus easier to simulate.  

If the player draws Bash too late during the first deck shuffle and has an unfavorable reshuffle with their Strike cards, they are likely unable to survive Gremlin Nob's attack for too many turns, even if they are able to enter the fight with full HP ($$80$$).  Unmitigated, Gremlin Nob deals a minimum of $$88$$ damage in the first $$6$$ turns.  

For a similar reason, the Defect may also be unable to survive an unfavorable combat against a buffed Gremlin Nob.  

### What about Watcher?  

Watcher is highly regarded as the strongest of the $$4$$ characters in _Slay the Spire_.  With her starting deck alone, the player can deal $$123$$ damage during the first $$2$$ cycles through the deck.  This exceeds the maximum base HP of Gremlin Nob and Lagavulin, and sometimes can kill all $$3$$ Sentries.  

On the other hand, if the player is unable to play $$2$$ Strikes during this time, this amount is reduced to $$99$$, which is less than the minimum HP of a HP-buffed Gremlin Nob.  Remaining in wrath stance also poses a significant constraint on the player.  If the player is in wrath during a $$3$$-attack cycle, then, unmitigated, Gremlin Nob's attacks during this time total a minimum $$112$$ damage, which eclipses Watcher's starting HP of $$61$$ (Ascension $$14+$$), and the lesser of these two attacks totals $$64$$ damage.  

How many of Watcher's skill and power cards are helpful in this particular fight?  While the answer is probably "many", I do not believe it is "all".  I am optimistic that even an unwinnable Watcher seed could be found in a few days of searching, and proven, with the aid of a moderately optimized, RNG-accurate combat simulator.

### Are there unwinnable runs with glitches?  

If you watched [_Slay the Spire_ at AGDQ 2022](https://youtu.be/Q7FlPBFKX_Q) or have otherwise seen some _Slay the Spire_ speedrunning, you probably know that this game has [a lot of glitches](https://docs.google.com/spreadsheets/d/101K_6xxDjQH2caGApCMxgrybOScGw0VRwd5C-qYl0tA/).  Using a variety of different node entry manipulation glitches, the player can: 

1. (double node) enter $$2$$ or more map nodes on the same floor, 
2. (node dupe) enter the same map node twice, 
3. (node reroll) change the combat in a duplicated combat node, and 
4. (node ladder) climb the map on multiple paths at once.  

By node glitches alone, the player significantly increases their chance of accessing damage-dealing cards, deck manipulation, potions, card removals, card upgrades, and shops from ?-nodes.  While it may be possible to contrive scenarios for unwinnable glitched runs, they are necessarily more scarce by several orders of magnitude.  For example, by duplicating combats, the player can turn a $$3$$-combat path into one with $$5$$ or even $$6$$ combats before the burning elite.  By visiting additional map nodes, the player also changes the floor number during the burning elite fight, and can improve their deck manipulation in this fashion.  

I doubt such a seed exists.  If one does, it is unclear how one would find it, let alone prove it is unwinnable.  


## Final remarks

That it has taken $$3$$ years since the game's main release (and over $$4$$ years since it was released for early access) to find and prove seed unwinnability is a testament to the strength of _Slay the Spire_'s balance and design.  How much further can we go?  I do not know, but I invite any interested programmer to check out the [C++](https://github.com/gamerpuppy/sts_seed_search) and [CUDA](https://github.com/OohBleh/cuda-the-spire) code.  Don't be afraid to get in touch (see main blog page for contact info) and join the fun.
