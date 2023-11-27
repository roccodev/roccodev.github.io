+++
title = "Xenoblade 3: Understanding how the AI uses arts"
date = 2023-01-18

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

This is an extensive guide on how ally and enemy AI works in Xenoblade Chronicles 3.  
<!-- more -->
This is still under research and this is the result of reverse-engineering and experimentation, so data is accurate to the best of my ability.

While reading this, it is best to have the 
<a href="https://xenobladedata.github.io" title="BDAT table index maintained by Alexsey#9211">BDAT tables</a> at hand. 
Alternatively, I have a 
[collection](https://docs.google.com/spreadsheets/d/1_lK2KEO3HZlgqwDzdQxl4PWw_9W8pRdAbaDswb-gZZU) of 
[spreadsheets](https://docs.google.com/spreadsheets/d/1u7Upi2NsUzNI3gWxD81RhLuoI-XXUnWJt7zJBgupSGY) that are easier to understand.

Special credits go to:

* Lexicon (lexicon1)
* Hamidu

from the [Xenoblade Chronicles Discord server](https://discord.gg/xenoblade) for helping with research.

## How often does the AI use arts?
The AI will try to perform an action on {% tooltip(label="set intervals") %}`AiCycle` in `BTL_Talent` (class), `BTL_Enemy` (enemy), `CHR_UroBody` (interlink){% end %} 
defined by the class or the enemy type. 

This interval is different every time, being within 70-130% of the regular interval (decided randomly). Additionally, when an art is used, 
an {% tooltip(label="additional cooldown") %}`IntervalArts` in `BTL_Arts_PC` (allies) / `BTL_Arts_En` (enemies){% end %} is added,
though these two timers tick simultaneously.

## How does the AI decide what arts to use?
Every art in the game has one or two {% tooltip(label="conditions") %}`AiCond1/2` with `AiParam1/2` in `BTL_Arts_PC` and `BTL_Arts_En`{% end %} 
associated to it, that determine whether the art can be used.  
Additionally, each condition has a {% tooltip(label="priority") %}`Priority` in `BTL_Arts_Cond`, row ID is the condition ID{% end %} (1-6) 
and a {% tooltip(label="chance") %}`AiRate1/2` in `BTL_Arts_PC` and `BTL_Arts_En`{% end %} (%) associated to it.

The AI will go through every charged art and check its first condition. If the condition and the random chance
are satisfied, the AI will make sure the art is in range, and a "fusion check" will also be run (see below).
If all succeeds, it will add the art to a priority queue with a priority of `Priority * 10` for regular arts, and
`Priority * 10 + 1` for talent arts.  
If no arts could be found during the first check, the AI runs the checks again with the second conditions.

The game will then try to use the art with the highest priority in the queue. (If there is a tie, it's decided randomly.)  
At this point, any check failures will cause no art to be used during this cycle.

If the art has a positional effect and the character is not in the right position, the art will be put "on hold". The AI will try
to move to the required position, and it will use no art this cycle.

At last, the AI will use the art. If the art is linked to another art (no matter if master or class art) that's fully charged,
it will automatically fuse. However, some situations, like being out of range with one of the two arts, will cause one of the two
arts to not be used. Still, it is possible for one of the two arts to be used even if it didn't meet conditions or have high
enough priority, as long as the other art can be used instead.

## Fusion condition check
The AI features two mechanics to make the most out of fusion arts. These are mainly used to delay or cancel art usage altogether, so
they're disabled for most conditions associated with important arts, like combo and healing arts. This check is performed before adding the art
to the priority queue.

I'll distinguish the following scenarios:
- Class art is being checked and master art is charged: always add the art to the queue.
- Master art is being checked and class art is charged: Section (A)
- Either art is being checked and the other one is on cooldown: Section (B)

### Section (A): using master art with charged class art
If this check is {% tooltip(label="disabled") %}`760B765B` = 1 in `BTL_Arts_Cond`{% end %}, add the master art to the queue.  
If the fusion has been active (i.e. both arts were charged) for less than 5 seconds (10 seconds if Fusion First), add the master art to the queue.

### Section (B): using art with the other one on cooldown
If this check is {% tooltip(label="disabled") %}`B65743C7` = 1 in `BTL_Arts_Cond`{% end %}, add the art to the queue.  
Otherwise, calculate the time variable for both arts (formula below), then if `(Time(art on cooldown) * 30 / Time(charged art))` is greater than or equal to 30% (60% if Fusion First), add the art to the queue.  
Due to what seems to be a bug, the time for the art on cooldown is multiplied by 30 for both Keves and Agnus arts, which means that Agnus arts will always be used no matter the other art's recharge.  
(Essentially, this mechanic is used to determine whether using the art now would outbenefit waiting for a fusion.)

#### Time formula
For Keves arts, this is just the cooldown time left (in frames). (If the art is fully charged this is the max cooldown, not 0)  
For Agnus arts, this is `(recharge left) * (Art.LastEligibleFrame/30 + 2/3 + IntervalAT*(1 - AutoAtkAdditives%))`, where:
- `Art.LastEligibleFrame` is the last hit frame with a `DmgRt` < 255 (or the first hit frame, if no hits meet that criteria).
- `IntervalAT` is always 0 for arts.
- `AutoAtkAdditives%` is stuff like the auto-attack interval gem, Martial Artist effects, etc.
- `(recharge left)` is the {% tooltip(label="max cooldown") %}`RecastX` in `BTL_Arts_PC`, where `X` is the art's level{% end %} 
if fully charged, otherwise it's the recharge "time" left, usually the number of auto-attacks left. Note that this isn't always an integer because e.g. multi-hit auto-attacks only decrease it by a fraction.

## Other notable conditions
Combo art conditions (namely #8, #35, #43-45) will make it so the art is only used as part of the right combo. The AI is pretty smart about this, it will even make sure that by the time the art inflicts the reaction (using the exact frame for the calculation), the combo will not have expired. 
Additionally, if the battle tactic is set to the *opposite* combo (i.e. "Burst Combo" for Launch arts, "Smash Combo" for Daze arts -- "Any Combo" or the correct choice disable this check), 
it won't use the combo art, unless the reaction would have expired by the same reaction frame in the next AI cycle. 
This is why you might still get the wrong combo near the end of the combo timer. (Of course, you might still get the wrong combo anytime as part of the fusion mechanics.)

Another combo art condition (#36, generally used as a 2nd condition) works the opposite way. If the currently alive party has no way to reach the combo stage of the art, it will make it so the art is used as normal.  
For example, if the alive party has no Break or Topple art (in general, not just currently charged), the AI will not preserve a Launch art, instead using it like any other art.

## Enemy art sub-phases
Enemies don't usually have all of their registered arts available at once. Instead, there are multiple AI profiles based on their current HP percentage, their level, whether or not they are enraged, [whether you have defeated certain enemies](https://www.youtube.com/watch?v=SgmP2zk6TJc), other enemies in the fight,
whether the enemy has used a specific art, and the battle time.

For example, Kilocorn Grandeps (the superboss, not the Challenge Battle version) has 5 profiles for each of the refights, for a total of 56 profiles (10 refights + 1 default unused profile).  
At level 95 and above 80% HP, it will only use Horn Dance and Breath.  
However, at level 150 and below 20% HP, it will instead only use Tank March, Horn Dance (a stronger version that has a charge gauge and is a Smash combo), Breath (a stronger version with a charge gauge, that removes all buffs on the target, and will trigger Poison Breath and Ice Breath to be used immediately after), Killer Dunk, Ice Breath, and Poison Breath (with the latter two only being used as part of the "Breath" combo).

If you'd like to know how to interpret the table, uncover the paragraph down below.

<details>
<summary>Click to read the instructions</summary>

The `EnemyAiHead` and `EnemyAiTail` columns in `BTL_Enemy` define the range of AI profiles to use. Those AI profiles can be found in `BTL_EnemyAi`. 

In the `BTL_EnemyAi` table, the `A5C654A0` column defines the minimum enemy level for the profile. `42074031` is a HP percentage threshold, and `9184A789` is a condition for the enemy to be enraged. `0751D751` is a condition (i.e. a row from `FLD_ConditionList`). `1716B5CD` is an enemy ID (from `FLD_EnemyData`) which is checked to be alive and nearby, `CAF0DBF3` is a maximum number of enemies in the fight, `6B28F42E` requires the art to have been used (art ID from `BTL_Arts_En`). `B461BFB1` is a minimum
battle time in seconds.

Next, the `EnemyUseArts01-16` columns are what enable each art in the enemy's art palette. The values are the art slots to be enabled. For example, if an enemy AI profile has use-arts values of `0`, `1`, `3`, and `4`, it will enable `ArtsSlot0`, `ArtsSlot1`, `ArtsSlot3`, and `ArtsSlot4`. A `0` in anything other than `EnemyUseArts01` is ignored. (In that column, a `0` means to enable the art in `ArtsSlot0`. To disable the first art, `255` is used instead.)

</details>