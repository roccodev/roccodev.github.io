+++
title = "Xenoblade 3: Hidden values explained (Part 2: Land of Challenge)"
date = 2023-12-04

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

This post series illustrates some recent findings in Xenoblade 3 datamining.

Values are mostly taken from the newly researched *devxml* files. Information about those values is being collected over at this [spreadsheet](https://docs.google.com/spreadsheets/u/1/d/1_prmO3-6ra4kaCy07Kd2E7Lbcyf1X052Is7MmfREkYk/htmlview).

This is the second of a three-part coverage. ([Part 1](@/blog/xenoblade-3-devxml-1.md), [Part 3](@/blog/xenoblade-3-devxml-3.md))  
This time, I am going over the DLC challenge modes, explaining internal values, and showcasing unused content.

Thanks again to Lexicon (lexicon1) and Hamidu for helping with research.

# Time Attack

## Archsage's rewards

The Nopon Archsage rewards the player if all challenges are completed.

Starting from version 1.3.0, when all Time Attack and Gauntlet challenges (except at most "The Two Saviors") have been completed with an S rank on any difficulty, the Nopon Archsage will give the player the Archsage's Flower Fan, an accessory that buffs the user and shows cool particle effects. This event also sets 1-bit flag 3441.

If 1-bit flag 3441 is set on your save file when you receive this reward (i.e. on subsequent NG+ playthroughs), you are given the Humongous Nopon Doll instead. This is a joke item that weakens the whole party when equipped.

## Challenge bonus

When a challenge is completed, a random number between 0 and 99 is rolled. If the number is less than 25, one challenge is chosen for the 3x bonus. If the number is less than 5, two challenges are chosen instead.

The number of challenges with a 3x bonus active cannot exceed 5.

## Origin challenge movies

In "Stage of Destiny," the movies playing on the theater screen change every 20 seconds. The movies cycle in this order:

* Lanz and Sena's memories
* Eunie and Taion's memories
* Noah and Mio's memories
* The background clip used for Z's fire phase
* The background clip used for Z's chill phase
* The empty screen clip

## Enemy waves

When all enemies are defeated, a new wave immediately starts. The game takes a while to process this, and spawns new enemies with a delay of ~10ms, this is usually when the enemy target bar disappears from the HUD.
This should be enough for AOE arts that can hit new enemies, though the enemy only becomes fully visible and targettable about a second later.

The "new wave" text has a delay of one second. The "challenge complete" text appears 3 frames after the timer stops. The ~10ms delay described above also affects the end of the challenge: the timer is not stopped when the enemy is dealt the final hit, but only ~10ms later.

It should be noted that due to vertical sync, when the game can't keep 30 FPS it will reduce its tick rate to 15 Hz. Combat actions will be slowed down, but the challenge timer will still be based on real time, i.e some frames might add 3.3ms to the timer, while others might add 6.6ms. The delays I described may then vary in practice.

In "The Four Devas," new waves start when there is only one enemy in the arena.
The second wave starts either when Kilocorn Grandeps is defeated, or automatically after 60 seconds.

# Archsage's Gauntlet

## Round settings

Each round (or stage) in a challenge has different enemy placements, shop tables, and event/whimsy settings.

Rounds are defined in the following BDAT tables:

* [5541FE8B](https://xenobladedata.github.io/xb3_200_dlc4/5541FE8B.html) for the Beginner challenge,
* [440A05E7](https://xenobladedata.github.io/xb3_200_dlc4/440A05E7.html) for the Intermediate challenge,
* [D31CA255](https://xenobladedata.github.io/xb3_200_dlc4/D31CA255.html) for the Pro challenge,
* [58F078A6](https://xenobladedata.github.io/xb3_200_dlc4/58F078A6.html) for the unused infinite challenge. (see the last section)

Regarding enemy placements, each round row defines the enemy's level and additional stat multipliers. Enemies are selected randomly from [this table](https://xenobladedata.github.io/xb3_200_dlc4/BTL_ChSU_EnemyTable.html), based on the range of row IDs defined by `7EE21AA2` and `BDC9BBC2` (or by `E0975E89` and `B62754C4` if `BTL_ChSU_MapBattleLock.D93144C4` = 1).

I've collected shop, event, and whimsy data in an easier-to-read [spreadsheet](https://docs.google.com/spreadsheets/d/1eINXX6iDIjIa-yOAvvohU4b_ZZYIq3n4-7yWPh2ihw0/htmlview). The most salient points are:

* The first three stages have no events, no whimsies, and fixed shop types: the first two let you choose a free hero, the third only gives away emblems.
* Boss stages (`C63CA812` = 1) don't have any events during or whimsies after the stage.
* Boss stages always have Nopwatch refill in the shops *before* and *after* the stage. The shop after a boss stage never offers any heroes.
* Certain stages cannot get the "Lucky Monsters" or "Moebius Interference" events. This is accomplished by removing them from the random set of events that can appear, as well as setting `2DEAFB66` to 1, which makes the "Mystery Beasts" whimsy do nothing.

Note that the indication "the enemy power level has risen" simply states that enemy levels have increased.

In the Pro and infinite challenges (starting on rounds 71 and 70, respectively), more difficult challenge settings are loaded:

* The Invincible buff is not applied, like on Time Attack when the difficulty is set to Hard.
* Enemies get enhance 3926 as an extra enhance slot. This deals 10% of the enemy's Attack as fixed damage when one of their attacks hits.

## Nopwatch

The full Nopwatch gauge is worth 100 points, so percentages are equivalent.

Every second not in a menu or Chain Attack, the Nopwatch gauge is drained by:

* 0.5% on Easy (100 to 0% in 200s)
* 0.3% on Normal (333s)
* 0.25% on Hard (400s)

Perhaps counterintuitively, the Nopwatch gauge drains faster on Easy and slower on Hard. It is unclear whether this is a mistake, or a deliberate plan to balance challenge duration when enemies have lower HP.

Under the "Nopwatch Depletes Faster" stage event, the Nopwatch gauge depletes twice as fast.

When the Nopwatch gauge is fully drained, the following effects are applied to the player's team:

* 20% health recovery,
* 20% art recharge rate,
* 33% rescue speed, for both revivals and combo rescuing.

## Enemy score

The player starts a gauntlet challenge with 3,000 points added to their score.

The enemy score formula is the product of the following factors:

* The base value:
  * For score gained by defeating enemies: `Score` from `FLD_EnemyData`
  * For score dropped by enemies on hit during Launch Charge bonus time: 5
  * When inflicting Topple: 20 (max 20 times per stage)
  * When inflicting Smash: 50 (max 5 times per stage)
  * When inflicting Burst: 50 (max 5 times per stage)
  * When defeating an enemy with a chain attack: 200 (once per stage, this is separate from score gained by defeating the enemy)
* The current kill streak bonus:
  * 100%, no kill streak
  * 104%, 1-kill streak 
  * 108%, 2-kill streak
  * 112%, 3-kill streak
  * 116%, 4-kill streak
  * 120%, 5-kill streak
  * 120%, 6+-kill streak (this was probably higher at some point in development)
* During Launch Charge bonus time, score is doubled.

Exclusively for score gained by killing enemies, the following multipliers are also present:

* The effect of the "Prosperity" and "Score Up" emblems.
* The enemy value-related battle events:
  * "No Enemy Value": 0%
  * "Lower Enemy Value": 50%
  * "Higher Enemy Value": 150%
  * "Higher Enemy Value+": 200%
* The current stage's `ScoreRate`. For rounds 101+ in the infinite challenge, this is the score rate for round 100.

## Launch Charge

Launch Charge can be recharged by defeating enemies, with a full gauge being worth 300 points, dischargeable every 100 points. The amount of gauge an enemy fills is defined by `RisingCharge (4BAF120D)` in `FLD_EnemyData`, however:

* Defeating [Prosperous Ottil](https://rocco.dev/xc3-enemy-compendium/4302) or [Joyful Ottil](https://rocco.dev/xc3-enemy-compendium/4303) recharges one full bar.
* If emblem or stage effects altering Launch Charge buildup speed are active, those are multiplied to the enemy's value. This does not affect the two enemies mentioned above.  
  * In particular, the stage event "Launch Charge Unusable" also nullifies Launch Charge buildup for that stage. (except for the Ottils)
  * The "Launch Charge Fills Faster" event multiplies gain by 500%.


When Launch Charge is initiated, all enemies in battle are launched, if they aren't already.

Launch Charge still follows combo duration rules but is not associated to any character. This means that:

* Launch duration carries over from Topple, if the enemy is toppled.
* Launch duration accessories do not work on Launch Charge, and neither does arts link (Chain/Shackle Ring).
* The "Combo Duration Up" emblem can however extend Launch Charge duration.
* Launch Charge duration is affected by difficulty, i.e. 5 seconds on Easy/Normal, 3.75 seconds on Hard.

Additionally, starting Launch Charge will activate "Bonus Time". This mode lasts 15 seconds, its effects are described in the "Enemy score" section.

## Shops

Only these shop types can have Nopwatch refill:

* "Welcome, Welcome!": 40% chance
* "Friend get 20% off this time!": 40% chance
* "All items currently half-off!": 40% chance
* "Welcome, Welcome!" (after the "Nopwatch, Inc." whimsy): 100% chance
* "Kudos, friend! Have prezzie!" (after boss): 100% chance
* "Selection suddenly huge!": 40% chance
* Unused shop with random prices: 40% chance
* "Next is boss! Best to top up!": 100% chance

There is an unused shop type, as well as a whimsy to trigger it, that would show a selection of three items with prices ranging between -50% and 50% of their regular price.

The "Discount Card" emblem reduces shop prices (including Nopwatch refills and hero costs) by 4, 6, 8, 12, 16, 20%.

Purchasing a Nopwatch refill (either by paying with score or for free) increases the price of the next one by 400, maxing out at 8,000. The first two Heroes are free, then the price increases by 2,500 each time, with 10,000 as the maximum.

Defeating [Fortunate Pippito](https://rocco.dev/xc3-enemy-compendium/4299), or trading via whimsies yields one Shuffle Ticket.  
Shuffle Tickets cannot be used on shops that only allow one item to be purchased, when a purchase has already been made.

## Emblems

An unused emblem effect (16) would grant an invincibility period after completing a Chain Attack.

Three unused emblem effects (35-37) would speed up launch charge buildup if the team has no defenders, attackers, or healers respectively.

An unused mechanic would also increase the likelihood of {% tooltip(label="certain emblems") %}`BTL_ChSU_ShopItem.4534B6B3` non-zero{% end %} appearing in the shop, if none of the same category have been purchased yet.

## Events

Enemy Rush provides two additional waves of enemies. The next wave appears when either half of the previous wave's enemies have been defeated, or automatically after 60 seconds.

## Unused infinite challenge

The game supports, and has round data for, a never-ending gauntlet challenge.

The infinite challenge is perfectly functional, though it requires both editing BDATs and modifications to the executable to be loaded.

It is possible to make any other challenge infinite by setting `Floor` to 255 in `BTL_ChSU_List`. The challenge will loop after stage 100 in the manner described below.

This mode is mostly similar to Pro, but with different enemies and with the following changes:

* Bosses appear every 4 rounds, and are either Lodmoro Plambus, two Serpronds, or Consul J.
  * There is no special shop before and after boss stages, battle events and whimsies can also appear just fine.
* After round 100, enemy stats are increased (round 100 is taken as base) as defined by row 101 of the challenge's table, that is, +1 level, +100% HP, +50% strength, +250% healing, +2% agility, +2% dexterity, every round.
  * Shop, event, and whimsy data is also taken from row 101.
  * Enemies are chosen from rows 210-257 of the enemy table.
  * 200 Noponstone is added to the final reward every round.
* The "more difficult" challenge settings are effective starting from stage 70. (instead of 71)

Note that while there is a fully functional save slot for this challenge's records, attaining an S rank in the challenge will interfere with the Flower Fan/Nopon Doll's reward logic: the game expects an exact total of 21 challenges completed with S rank, excluding "The Two Saviors" from the count. If this challenge is completed, and there are no other challenges left without an S rank (bar the Future Redeemed one), the only way to trigger the Archsage's event would be to edit the save file.