+++
title = "Xenoblade 3: Effects explained"

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

**Note**: This post might be incomplete. Data will be added as I make new findings.

The damage formula is a chain of multiplications. Several effects are grouped together to form a single term in the multiplication.
<!-- more -->

### Multiplier group 1

| Effect description | Notes |
| ------------------ | ----- |
| Boosts allies' damage by `...`% and reduces enemy Block Rate by `...`% (does not stack). | |
| Flare Element | +5%/element, max +15% |
| Boosts damage by `...`% for every enemy in battle (max. 200%). | |
| Boosts damage dealt when attacking broken/toppled/launched enemies by `...`%. | Also includes Universal Annihilation |
| Grants a `...` increase to damage dealt when attacking enemies during combos. | |
| Boosts damage dealt when attacking enemies targeting you by `...`%. | |
| Grants a `...` increase to damage dealt the more enemies target you (up to a maximum of 300%). | |
| Boosts damage dealt indoors/outdoors by `...`%. | |
| Boosts damage dealt by `...`% when HP is at `...`% or lower/higher. | |
| Species Expert | |
| Boosts damage dealt when attacking from behind/the front/side by `...`%. | |
| Boosts damage dealt when attacking higher-level enemies by `...`%. | |
| Boosts damage dealt while an ally is incapacitated by `...`%. | |
| `...` boosts damage dealt when an ally is down/has low HP. | |
| Boosts damage dealt by `...`% for the first 30 seconds of battle. | |
| Boosts damage dealt by `...`% per enemy defeated (to a maximum of 250%). | |
| Boosts damage dealt by `...`% each time an ally is incapacitated (to a maximum of 200%). | |
| Boosts auto-attack damage by `...`%. | |
| Boosts damage dealt by `...`% when fighting a unique or boss monster. | |
| Deal `...`% more/less damage but take `...`% more/less damage. | |
| Boosts damage against enemies performing Arts by `...`%. | |
| Deals `...`% more damage against enemies who are at 30% HP or less. | Includes the Chain Attack version |
| Deal `...`% more and take 25% less damage while within a field effect. | |
| Boosts damage dealt by Fusion Arts by `...`%. | |
| Each attack that hits boosts damage dealt by `...`% (max. 255%). Missing cancels the boost. | |
| Boosts damage dealt by `...`% when Awakened. | |
| Boosts damage dealt for each active buff. | |
| Boosts damage dealt by `...`% when afflicted with a debuff. | |
| Grants a small increase to damage dealt (max. 500%) for each buff active on all allies. | +50%/buff (+30% without the upgrade) |
| Grants a small increase to damage dealt for each debuff active on all enemies (up to a maximum of 500%). | +50%/buff (+30% without the upgrade) |
| Boosts damage dealt by physical/ether Arts by `...`%. | |
| Grants a small increase to damage dealt, based on heat gauge level. | |
| Boosts damage to Moebius by `...`%. Cannot be blocked. | |
| Boosts damage dealt by `...`% after receiving no damage for a set time. | |
| Boosts damage dealt by `...`%, but reduces max HP by half. | |
| Boosts damage dealt for each Soulhacker Art or Skill learned. | (Attack Mastery and Final Countdown) |
| After own revival, deal `...`% more damage and take `...`% less damage for a fixed time. | |
| Boosts damage dealt by `...`% when on land/in the water.| |
| Boosts damage dealt by `...`% when attacking a debuffed enemy. | |
| Evading deals `...`% of Attack damage and boosts damage dealt by `...`% (up to max. of 300%). | |
| Boosts damage dealt and Critical Rate by `...`% on critical hit. | |
| Boosts damage dealt by `...`% when landing a critical hit (up to a maximum of 150%). | |
| Boosts damage dealt by `...`% when using a Talent Art (up to a maximum of 200%). | |
| Each time you perform a cancel, boosts damage dealt by `...`% (to a maximum of 150%). | |
| Buffing an ally boosts damage dealt by `...`% (up to a maximum of 200%). | |
| Debuffing an enemy boosts damage dealt by `...`% (up to a maximum of 150%). | |
| Each elemental discharge boosts damage dealt by `...`% (up to a maximum of 400%). | |
| Own combo reactions boost damage dealt by `...`% (up to a maximum of 500%). | |
| Awakening (buff) | +75% damage dealt (base) |
| Attack Up (buff) | +25% damage dealt (base) |
| Reduce All ~ Origin Blade (debuff) | -30% damage dealt (base) |
| Interlink bonus/penalty | -20% (Lv. 0), 50% (Lv. 1), 150% (Lv. 2), 300% (Lv. 3) |
| Boosts elemental damage by `...`% and elemental buff effects by `...`%. | Acts as a damage additive to elementals |
| Boosts elemental damage by `...`%. | Acts as a damage additive to elementals |

### Defensive multiplier group
(Subtract the total from 100%)

| Effect description | Notes |
| ------------------ | ----- |
| Reduces damage taken by allies in a fixed radius by `...`%. | |
| Reduces damage to allies by `...`%, and boosts HP recovery by `...`% (does not stack). | |
| Reduces damage taken by all allies while Art is active by `...`%. Also counters attacks directed at self. | |
| Earth Element | -5% damage taken/orb, max -15% |
| After own revival, deal `...`% more damage and take `...`% less damage for a fixed time. | |
| Reduces damage taken by `...`% when recharge for all Arts is depleted. | |
| Defender Mastery | |
| Stance Effect: Shadow Parade | | 
| Reduces damage taken by `...`% while performing Art. | |
| Stance Effect: Determination | |
| Deal `...`% less/more damage but take `...`% less/more damage. | |
| *Slightly boosts damage dealt when an ally is down/has low HP.* (see note) | "Low HP": <= 30%. Some variations of this effect (e.g. Healer Lucky Seven's skill) also reduce damage taken |
| Reduces damage taken by `...`% when HP is at `...`% or lower. | |
| Reduces damage taken by `...`% per enemy defeated (max. 20%). | |
| Awakening (buff) | -25% damage taken (base) |
| Defense Up (buff) | -15% damage taken (base) |
| Reduce All ~ Origin Blade (debuff) | +50% damage taken (base) |
| Interlink bonus/penalty | +40% damage taken (Lv. 0), +0% (Lv. 1), -30% (Lv. 2), -60% (Lv. 3). Damage during Interlink affects heat buildup speed. | 


### Multiplier group 2
(Add 100% to the total)

| Name | Value |
| ---- | ----- |
| Power Charge (buff) | 50% (base) |
| Accelerating Attacks | 50% |
| Element Genesis | up to 300% | 
| Raging Force | up to 613% |

### Multiplier group 3
(Add 100% to the total)

| Name | Value |
| ---- | ----- |
| Cancel bonus | *varies* |
| Supercharged | up to 90% |
| Piercing Laser | up to 120% |

### Independent multipliers

| Name | Multiplier |
| ---- | ---------- |
| User Attack | *varies* (as seen on the Characters menu) + Stability ([read more](https://blog.rocco.dev/2023/01/01/xenoblade-3-random-factors.html#weapon-stability)) |
| Art/Auto-attack/Elemental damage multiplier | *varies*. Flare/Earth: 200%, Aqua: 150% |
| Critical hit | 125% (base) |
| Launch/Smash/Burst damage bonus | 125% (launch), 150% (burst), 150% (smash, base) |
| Chain/Shackle Ring | up to 180% | 
| Enemy defenses | 100% (bypassed), or 100% - defense
| Attack blocked | 50% (base) / 25% (base, when Eye of the Storm is fused) |
| Random variance | 90%-110% (random, [read more](https://blog.rocco.dev/2023/01/01/xenoblade-3-random-factors.html#random-variance)) |
| Chain attack damage ratio | *varies* |
| Fusion damage bonus, if supported by the art that generated the hit | 150% |
| Attacking an enemy from behind, if it hasn't noticed the player | 150% |
| Multi-hit damage correction | 1/(number of damage hits), can be 2/n or more, depending on the art/hit. As of 1.1.0, does not apply to *fused* Master Arts that do not spawn bullets normally (and that have multiple fusion hits) -- only eligible art is Quickdraw |
| AOE penalty | 100% (main target), 75% (other enemies) -- also works for enemy attacks |

### Useful buffs/debuffs

| Buff | Effect | Duration | Max duration | Base chance (Debuff type) |
| ---- | ------ | -------- | ------------ | ----------- |
| Attack Up | +25% damage (see Multiplier group 1) | 20s | 40s | -- |
| Awakening | +75% damage dealt (see Multiplier group 1), -25% damage taken (see Defensive multiplier group), +50% recharge speed | 20s | 40s | -- |
| Critical Rate Up | Critical hit rate: +25% | 20s | 40s | -- |
| Critical Hit Plus | Critical damage bonus: +50 percentage points | 20s | 40s | -- | 
| Phys./Eth. Def. Down | Does nothing if defense is 0. If > 0, multiplies it by 75% (base). If < 0, multiplies it by 125%. | 15s | 30s |  75% (Phys./Eth.) |
| Reduce All (Origin Blade) | -30% damage dealt (stacks with Multiplier group 1), +50% damage taken (see Defensive multiplier group), -30% accuracy/evasion | 60s | 60s | *guaranteed* |
| Resistance Down | Break and debuff resist: -25 percentage points | 15s | 30s | 75% (Ether) |
| Bleed | 4% of art damage every 2 seconds | 46s | 92s | 100% (Phys.) |
| Blaze | 8% of art damage every second | 10s | 20s | 100% (Ether) |
| Freeze | 6% of art damage every 1.5 seconds | 21.5s | 43s | *unavailable* |
| Toxin | 12% of art damage every 4 seconds | 24s | 48s | 100% (Ether) |

### Other

| Effect description | Notes |
| ---- | ----- |
| Boosts Smash damage by +`...`% | Net additive. Unlike the Smash damage bonus, which applies to both the art's and Smash's damage, this only boosts the special damage dealt when the enemy touches the ground after being smashed. | 
