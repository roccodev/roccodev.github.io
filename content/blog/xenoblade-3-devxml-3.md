+++
title = "Xenoblade 3: Hidden values explained (Part 3: Future Redeemed)"
date = 2023-12-09

[extra]
math = true

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

<noscript>
<b>Warning</b>: This post contains some math formulas that are best displayed with JavaScript enabled.
</noscript>

This post series illustrates some recent findings in Xenoblade 3 datamining.

Values are mostly taken from the newly researched *devxml* files. Information about those values is being collected over at this [spreadsheet](https://docs.google.com/spreadsheets/u/1/d/1_prmO3-6ra4kaCy07Kd2E7Lbcyf1X052Is7MmfREkYk/htmlview).

This is the last part of my coverage, focusing on the game's DLC expansion: Future Redeemed. ([Part 1](@/blog/xenoblade-3-devxml-1.md), [Part 2](@/blog/xenoblade-3-devxml-2.md))  

Thanks again to Lexicon (lexicon1) and Hamidu for helping with research.

## Fusion arts

Master and fusion arts work a bit differently in Future Redeemed.

In Future Redeemed, there are basically no differences when using a class art compared to a master art. Both arts use the character's stats, equipment, and weapon parameters.

The fusion system is different from the base game, however. Instead of spawning a bullet and granting a specific bonus to both arts, when fusing in Future Redeemed the main (*right-side*) art will be used as normal, and a separate entity will perform a new special art.

This special art varies based on which arts were chosen, I have a list of the possible options and effects [here](https://docs.google.com/spreadsheets/d/1YRbu3-A5QOZo_w5LhoTsQGtDVmv_ycd14dgB1f5b9Ek/htmlview). The original master art is not used at all, and it is possible
for either of the two arts to miss. Being separate arts also allows getting duplicate TP instances in chain attacks.

The fused art is performed by an "Ouroboros" form, whose Attack stat (in the base game and in guides, this is most commonly known as the "displayed Attack") is calculated as follows:

$$\text{Effects}(\text{Ouro Strength} + \text{Weapon Damage})$$

* $\text{Ouro Strength}$ is $\left( \text{StrLv1} + \frac{(\text{StrLv99} - \text{StrLv1}) \cdot (\text{Level} - 1)}{98} \right)$.
  * $\text{StrLv1}$ and $\text{StrLv99}$ are from [`CHR_UroBody`](https://xenobladedata.github.io/xb3_200_dlc4/CHR_UroBody.html), the row ID starts at 7 with Matthew.
  * $\text{Level}$ is the user's level.
* $\text{Weapon Damage}$ is from [`BTL_WpnParamU1`](https://xenobladedata.github.io/xb3_200_dlc4/BTL_WpnParamU1.html) (`Damage` column), for the row equal to the user's level.
* $\text{Effects}: \mathbb{R} \to \mathbb{R}$ is a function that modifies the final stat. This includes effects from the player's equipment that boost the Attack stat directly, either as flat additives or as multipliers.

Other stats are calculated similarly, but without the weapon parameters.

This stat formula is the only place the user's equipment is taken into account. When in combat, the Ouroboros form
does not inherit the user's effects, meaning:

* It is not affected by equipped accessories, skills, or gem effects (excluding stat ones).
* It is not affected by Matthew's Shackle Ring effect, it doesn't advance or reset it, nor gain bonuses from it.
* It is not affected by the user's buffs, including Power Charge.

In other words, when using fusion arts, the user's equipment will only affect the *right-side* art.

## Vision

Vision makes the party automatically evade attacks, similar to the effect of the Decoy buff. Unlike Decoy and manual evasion, however, 
Vision does not protect characters suffering from Sleep, Moebius Shackles (absent in FR), or any variation of Bind.

Vision uses different parameters depending on art level:

* Level 1: 1 use, lasts 15s
* Level 2: 1 use, lasts 20s
* Level 3: 2 uses, lasts 15s

Vision also acts as a debuff barrier, for example for enemies that would inflict a debuff upon getting hit.

During Vision, A gets access to a new talent art, Vanishing End. This art can be recharged with one role action, it pierces enemy defence, and has a 14x damage multiplier at max level over nine hits. It also has a wide spherical AOE range, much like Element Genesis in the base game.

## Combos

Smash was severely nerfed in Future Redeemed.

First, the base damage was reduced to 50% (from 150%). In the base game, Smash would also get a damage boost equal to 3x the frames left on Launch duration. In Future Redeemed, the coefficient was changed to 0.25x. Overall, this amounts to a total damage difference of ~88% on a typical chain attack scenario.

Furthermore, the only source of Smash in Future Redeemed are Union Combo finishers. Not only do they have the lowest base damage of the pair finishers, but due to the way they are implemented, the Smash impact damage is also only based on the damage of the finisher hit, not the total damage of the Union Special. As described in the respective section, this is only determined by a fraction of the total damage.

## Enemy weakness

Future Redeemed introduces a new damage multiplier: enemy weakness. Enemies can be either weak to attacks from one side (front, back, or left/right - the latter simultaneously), 
which is displayed in the HUD with an arrow icon, or they can be weak when in a combo state, which is represented as an icon of a falling man. In both cases, damage dealt
to weakened enemies is increased by 1.5x.

Enemy weakness works in chain attacks: combo weakness always works if the enemy is in a combo, though side weakness only works when the character is by the actual side; unlike positionals, it is not guaranteed.

## Union Specials

The Union Special gauge is very similar to the Interlink gauge. The full gauge is worth 60 points, and can fluctuate as described here:

* +3 (0.5%) points on each art used by one of the two characters, 
* +7 (1.17%) on each fusion, 
* -30 (50%) when one of the characters in the pair dies. 

Positive gains are affected by difficulty (75% on Hard), and also how many Specials have been used by the pair during the battle:

* If no Specials have been used yet, no correction is applied.
* When one Special has been used, gain is reduced by 10%.
* When the count reaches two, it is reduced by 20%.
* When it reaches three, gain is reduced by 35%.
* When four or more Specials have been used, points gained are halved.

When a Special is started while an enemy is launched, the Special will end with a combo finisher. These finishers deal "true" damage (like Smash impact damage) that isn’t directly affected by equipment, can’t crit, can’t be blocked, doesn’t benefit from combo damage boost, weakness, etc. The calculation works like this: 

$$\text{Solo Arts Damage} \cdot \text{Base} \cdot \left( 100\\% + \sum_{c \in \\{0, 1\\}} \text{Union Combo Up}_c \right)$$

* $\text{Solo Arts Damage}$ is the total damage dealt by characters when they strike alone (usually the first part of the special). For an exact amount, check the frame data for arts 848-859 (only the arts with `S_pair_attack`, not `S_pair_finisher`, and filter by character ID in `UseChr`, ignore the art names) to see which hits actually count.
* $\text{Base}$ is `Damage` from [`BTL_HyperCombo`](https://xenobladedata.github.io/xb3_200_dlc4/BTL_HyperCombo.html).
* $\text{Union Combo Up}_c$ is the value of the "Union Combo Up" effect for character $c$. This stacks additively for both characters in the pair.

Additionally, when using Power Charge, it will only apply to the aforementioned "solo" attack, due to the joint combo being a different pair of arts. As described in the
calculations, it will still indirectly boost the final damage output.

## Chain attacks

The chain attack gauge now takes longer to fully charge:

* Gauge gained by inflicting Break or Topple was reduced by 33%.
* Gauge gained by inflicting other combo stages or canceling was reduced by 25%.

The base damage ratio for regular chain orders is now 50% (instead of 100%), 100% for Unity orders. (Ouroboros orders had 200%.)

Performing a Unity Combo adds +25% to the next chain attack's damage ratio. This bonus
is affected by difficulty, i.e. it is halved on Hard. The effect itself also has no maximum, though the damage ratio cannot go above 9,999%.

The Gray effect heroic chains (1.5x damage ratio gain) stack multiplicatively (i.e. 2.25x). This was not introduced in Future Redeemed, but it is the first
time that more than one instance of the same effect can be used in the same round.

Some key pairs (for damage ratio gain) aren't *symmetric*, unlike in the base game. That is, if character $a$ makes a key pair when used in character $b$'s order, it is not necessarily true that character $b$ makes a pair on $a$'s order. Here is a table:

| Order character | Eligible characters |
| --------------- | ------------------- |
| Matthew | A, Na’el |
| A | Matthew, Shulk |
| Nikol | Glimmer, Shulk |
| Glimmer | Nikol, Rex |
| Shulk | Rex, Nikol |
| Rex | Shulk, Glimmer |
| Na'el | Matthew |
| Unity orders | the two characters in the finisher |

Notice that while Shulk makes a key pair when used during A's order, the reverse is not true. Additionally, Unity orders *exclusively* allow
the two characters that appear in the finisher. In the base game, key pairs were also *transitive* for Ouroboros orders: for example, since Ashera is eligible for a key pair
during Eunie's order, she would also be eligible for Eunie's Ouroboros order.


## Enemy stats

Enemy base stats are different in Future Redeemed. Compared to the base game, enemies have lower HP, strength, agility and dexterity, though they have
significantly higher healing power. Base stats are defined in [this table](https://xenobladedata.github.io/xb3_200_dlc4/BTL_EnParamTable.html#201) for each level, which
also contains the base stats for the base game. (In the table, row 201 is for Future Redeemed's level 1 enemies.)

As mentioned in the first post of this series, the boosts given to elite enemies are different in Future Redeemed:

* HP formula: $\text{HpMaxBase} \cdot (\frac{\text{StRevHp}}{100} \cdot 1.25 + 0.25)$
  * `HpMaxBase` is from `BTL_EnParamTable`.
  * `StRevHp` is from `BTL_Enemy`.
* Strength multiplier: 125%
* Healing multiplier: 150%
* Dexterity multiplier: 125%
* Agility multiplier: 125%
* EXP multiplier: 250%
* CP multiplier: 300%
* Gold multiplier: 350%

The elite boosts follow the trend of Future Redeemed enemies having increased healing power.

## Food effects

Future Redeemed introduces a new food effect called Combat Streak EXP+. For each enemy killed without leaving combat (i.e. deaggroing every
enemy), the next enemies killed will yield gradually increasing EXP.

Combat Streak EXP+ stacks up to 10 times, which results in a maximum EXP boost of 200% for "Tender Boiled Lobster," 300% for "Chunky Veggie Soup," and 500% for "Legendary Doodlenoodles." 

Collectible Boost was also nerfed in Future Redeemed, as it no longer works on accessories, only material/collectible drops.

## Enemy drops

In the base game, food bonuses and overkill rate boost the amount of accessory/materials you get from enemies, but can't make it exceed the enemy's maximum. For example, if you have no food and a 200% overkill, and the enemy would drop between 1 and 3 accessories, you'd get between 2 and 3 accessories instead. (This is rounded down so a 199% bonus will still give you 1-3.)

In Future Redeemed this works slightly differently. There is no quantity boost for accessories, and material drops have a $(\text{Bonus} - 100\\%)$ chance to drop one extra collectible, and they can even go past the maximum. This is guaranteed for overkill (as 200%-100% = 100%); as another example, a +75% enemy drop boost food grants a 75% chance to get an extra collectible. 

Note that this is not the same as the Collectible Boost food effect, this only boosts the number of different items you get from an enemy, not the number of copies of each item.  

## Multi-enemy battles

The mechanic described by the game as "Enemy Strength in Numbers" works as follows:

* Totems/rifts/nests/etc. deal 50% more damage for each enemy in the fight, excluding self, max +150%.
* Those enemies also take 30% less damage for each enemy in the fight, excluding self, max -90%.

Some enemies and bosses in Future Redeemed are subject to the same reduced recharge rate system for multi-enemy battles as the base game, for which I've recently added an explanation in the [first post](@/blog/xenoblade-3-devxml-1.md).
Notably, the Shulk and Rex boss battle can be made easier by issuing the focus command, as this reduces the other's recharge rate to 50%.

## Moebius W fight

The [boss](https://rocco.dev/xc3-enemy-compendium/4440) has two notable arts, among the rest:

* Motes of Revival: revives the heads. The boss has three copies of this art, two used when each of the heads are dead, and one when both heads are down.
The art starts with a full recharge, and takes 50 seconds to recharge (on Normal difficulty) when used.
* Limitless Lightning: the boss enters a long animation and becomes vulnerable to attacks. It also casts a damage-over-time field (24 damage every 3 seconds, for 30 minutes).

While the three "Motes of Revival" arts have separate cooldowns, the boss will always wait at least 90 seconds between consecutive revivals.

The boss has a passive effect reducing damage taken by 99%. When the boss uses "Limitless Lightning", the effect is nullified. (Technically, the art gives him a +99% damage taken effect.)  
The art is only used once both heads are dead and the boss has used "Motes of Revival" at least once. Once this is satisfied, the boss will get 100% recharge on Limitless Lightning;
while it’s not satisfied, its charge is forcibly set to 0%.

## Alpha fight

In the first battle, Alpha will immediately use Erasure Strike when the battle starts.

The "Nullify Most Attacks" buff can be bypassed using fusion arts, in both phases. 

The "Vision Mode" (second phase) art animation lasts 45 seconds. If you deal 110,000 damage (using fusion arts) before the time is up, the boss will instantly use "Flawed Monado Buster." If instead you fail to make it in time, the boss will use "Perfected Monado Buster," which has a 15x damage multiplier, is unblockable (though it can be evaded), and will likely kill your team. 

A similar mechanic is also present in both games since the first version of the game: if the user of an art with `ArtsSpEffect` 6 is invincible, while it is in the 
art's animation the damage it would receive is stored and released as additional damage the next time an art with `ArtsSpEffect` 7 is used (though it still can't exceed 9,999,999).  
The mechanic goes unused as there are no player or enemy arts with those effects.

Alpha will only use "Monado Purge" when Vision is active. Additionally, when the art hits it will cancel Vision.