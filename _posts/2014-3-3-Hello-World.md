---
layout: post
title: An extremely unwinnable Slay the Spire seed
published: true
---
## Overview
In this post, I will describe how [gamerpuppy](https://github.com/gamerpuppy) and I found an unwinnable Slay the Spire seed along with a simple proof of unwinnability.  This project is motivated by ForgottenArbiter's success at finding a (probably) unwinnable seed.  Our goal was to find a seed for which it can be proven unwinnable without simulating combat, and without checking too many cases.  

## Previous progress

In ForgottenArbiter's blog post on [unwinnable seeds in _Slay the Spire_](https://forgottenarbiter.github.io/Is-Every-Seed-Winnable/), the author outlined a detailed approach to the problem of finding an unwinnable seed.  They take _unwinnable_ to describe any run of _Slay the Spire_ that cannot be won by any sequence of decisions allowed by the base game (ignoring glitches).  Since the random number generation of _Spire_ is determined uniquely by the run seed, this allows the player to replay a seed repeatedly, in search of winning plays, even if the individual decisions made are unintuitive. 

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
picture of the map
As an integer, this seed is 3,431,382,150,268,629 or ~3.4 quadrillion, but we make no claim that this is the first unwinnable seed.  This seed has the following properties: 

1. Neow only offers 1 removal, 100 gold, 250 gold for some max HP, or a swap into Cursed Key.  
2. The map forces a burning elite fight (max HP Lagavulin) on floor 6 with no shops or rest sites beforehand.  
3. The player encounters either 2 or 3 combats beforehand and the events only give 1 removal.  
4. The cards offered do not directly add any damage and are draw-neutral.  

I first got involved in this search in early January not long after finding and routing an [Ascension 20 Heart snipe seed](https://youtu.be/8jHTNGrreTw).  I had accomplished (with glitches and some very careful RNG manipulation) the first turn-1 Heart kill since the boot bug exploited [here](https://youtu.be/4knfPJyKLYY) by ForgottenArbiter was patched.  For it, I used [gamerpuppy's sts_seed_search](https://github.com/gamerpuppy/sts_seed_search), which emulates essential seed searching functions at a low level.  For speedrun-related searches in the past, I have predominantly used ForgottenArbiter's [SeedSearch](https://github.com/ForgottenArbiter/SeedSearch) mod, which runs through the game and comes with an easy to use list of seed criteria.  

But after two weeks of limited success 

In fact, laziness is built into the code that found this seed.  Enhancing the approach by 