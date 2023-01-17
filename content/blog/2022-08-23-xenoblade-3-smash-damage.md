+++
title = "Xenoblade 3: Smash damage breakdown"
date = 2022-08-23

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

Note: this is the result of reverse-engineering. Data should be correct to the best of my ability.

### Credits
* Special thanks to [vaxherd](https://www.twitch.tv/vaxherd) for lending me tools to work on BDAT tables, which made looking for code references easier.

## How combos work

First, I'll list the base values for each reaction in the combo:

|  Reaction  |  Damage multiplier (while active) |  Base duration  |
| ---------- | --------------------------------- |  -------------  |
| Break      | 100%                | 450 frames = 15 seconds |
| Topple     | 100%                | 150 frames = 5 seconds |
| Launch     | 125%                | 150 frames = 5 seconds |
| Smash      | 150%                | 0 frames, damage multiplier is on hit |
| Daze       | 100%                | 150 frames = 5 seconds |
| Burst      | 150%                | 0 frames, damage on hit |

When you inflict a reaction, the game gives it two special values (among others), the **damage multiplier** and its **duration**.

The damage multiplier lasts for the entirety of the reaction, and increases the damage you deal to the enemy that suffers it. The duration value is a counter that ticks down as the reaction gauge depletes, and starts at the reaction's duration in frames, with all modifiers applied.

For example, when inflicting Break, the reaction is given a 100% damage multiplier (i.e., you deal normal damage), and a duration value of 450. This value then ticks down every frame (except in chain attacks) until you inflict Topple or the reaction expires.

For Break and Topple, the duration value is overwritten when you continue the combo (so 450 -> 150 -> 150 -> 0), the other reactions have a special behavior.

**Launch**:  
When you launch an enemy, the damage multiplier is set to 125% (i.e. with no extra buffs you'll deal +25% damage), and the duration value **carries over from Topple** with an extra 150 (the base duration for Launch, can be increased with accessories), which means that earlier Launches will last longer.  
For example, if you topple an enemy and launch it 30 frames later, the Launch duration value will be `120 (Topple - 30 frames) + 150` = 270 frames (9 seconds)

**Smash**:  
When you smash an enemy, you will benefit from a base damage multiplier of 150%, plus a value of **Launch duration value * 3.0**. (Note: the hit that inflicted Smash gets the Launch bonus instead.) 
With the above example, if you smash the enemy with a 0-frame penalty (e.g. in chain attacks) the final damage multiplier is `150 + (270 * 3.0)` = **960%**.  
Lastly, the duration value is set to 1800 (60 seconds). This is to allow the enemy to collide with the ground, after which the duration is reset.  
When the enemy collides with the ground after being inflicted by the reaction, it will receive fixed damage equal to the **total damage of the Art** up to that point (including Master Art damage in case of Fusion Arts), multiplied again by the Smash multiplier; nearby enemies will receive **1/4** of that impact damage. Only in this case, equipped accessories that boost Smash damage will be added to the Smash multiplier.   
For Arts that inflict Smash on multiple enemies, such as Final Lucky Seven, the impact damage will be the same for all enemies, but only damage inflicted to the targeted enemy will be considered for damage calculation.

**Daze**:  
When you daze an enemy, the effect's duration **carries over from Topple**, with an extra 150 (base duration for Daze, can be increased with accessories).  Same behavior as Launch. 

**Burst**:  
When bursting an enemy, the damage from the hit is multiplied by 150%. Then, the duration value is set to 30 (1 second).  
Unlike Launch/Smash, there is no damage bonus tied to the duration.

These bonuses also apply when enemies inflict reactions on you, provided that they follow the right combo (excluding Break). For example, if an enemy topples and dazes you, Daze's duration will be extended based on the duration left on Topple. If an enemy forces Daze on you (without Topple), the duration is unchanged.

Keep in mind that the damage multiplier does not include boosts from accessories etc., those are somewhere else in the formula. The only way to increase the Smash damage multiplier (i.e., from the Smash formula) is to increase the duration value.  
This can be accomplished by **launching and smashing as soon as possible**, because the duration value carries over as soon as Topple starts. Another way is to use **duration-extending accessories**, because the duration value (and in turn, damage) is based on the *absolute* duration of the reaction.  
Additionally, gradually filling the gauge (e.g. with the Soulhacker skill or Mio's Ouroboros form) also works.

### Tests
Here are the results of some tests I've done. Please note that I've only recorded chain attack rounds, because I couldn't keep the framerate consistent on the emulator, so the calculations would be hard to follow.

The values in the tables are calculated as soon as the reaction timer would start ticking.

#### Chain attack, no boosting accessories ([video](https://www.youtube.com/watch?v=mt9CLBgAzn0))
During chain attacks, the duration value never decreases.

|  Combo step  |  Duration (frames)  |  Damage multiplier  |
| -----------  |  ----------  |  -----------------  |
| *No Reaction* | |
| Break | 450 | 100%
| Topple | 150 | 100%
| Launch | 300 (150 from Topple + base 150 from Launch) | 125% |
| Smash | 300 (from Launch) | 150% (base) + (300 * 3.0) = **1050%**

#### Chain attack, Topple extend, Launch extend ([video](https://www.youtube.com/watch?v=jShvqj5tX04))
The character inflicting Topple has an accessory that extends Topple by 27%, and the character launching extends Launch by 27%.

|  Combo step  |  Duration (frames)  |  Damage multiplier  |
| -----------  |  ----------  |  -----------------  |
| *No Reaction* | |
| Break | 450 | 100% |
| Topple | 190.5 | 100% |
| Launch | 381 (190.5 from Topple, 190.5 from Launch) | 125% |
| Smash | 381 (from Launch) | 150% (base) + (381 * 3) = **1293%** |

#### Outside chain attacks (hypothetical)
Here's an hypothetical breakdown on how time could affect damage.

|  Combo step  |  Duration (frames)  |  Damage multiplier  |  Time since last step |
| -----------  |  ----------  |  -----------------  | ----------------- |
| Break | 450 | 100% | |
| Topple | 300 (effectively 150*) | 100% | 5 seconds (150 frames) |
| Launch | 120 (150 - 30) | 125% | 1 second (30 frames) |
| Smash | 60 (120 - 60) | 150% (base) + (60 * 3) = **330%** | 2 seconds (60 frames) |

\* Remember that for Break and Topple the old value is overwritten. It is shown here for the sake of completeness.

## Finding the Smash code

> Note: this is a section for advanced users, and it describes the game's internals in great depth.

To locate the function in charge of inflicting reactions, I decided to find usages of BDAT functions.  
As it turns out, aside from the new hash indexing, the library is mostly a carbon copy of Definitive Edition's. Therefore, I'll be using function names from that game, as Xenoblade 3 does not export symbols as of 1.1.0.

Some useful definitions:
* `game::BdatAccessor::BdatAccessor`: .text+0024f6e0
* `game::DataBdat::setupFp`: .text+00250168
* `game::DataBdat::getAccessorImpl`: .text+00257770
* `game::DataUtil::getDataBdat`: .text+00279838
* `game::BdatID::getName`: .text+0025acd4
* `Bdat::isExistFP`: .text+00a7dd98

By inspecting the decompiler output for `setupFp` and `BdatID::getName`, I was able to find the array containing (I'm assuming) most hash IDs for BDAT tables. You can mark `.text+011c26d8` as an array of 539 ints.

The table for combos (`BTL_Combo`, ID `murmur32('BTL_Combo') == 0xD34EA852`) is at index 65 (0x41), meaning I could search for usages of `getAccessorImpl(this, 0x41)`.

That led me to the function responsible for calculating reactions: `.text+001e70ac`. The parameter in `w1` is the reaction ID (0, Break, Topple, Launch, Smash, Daze, Burst), the rest are unknown.
