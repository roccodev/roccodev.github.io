+++
title = "Minecraft 1.8 PvP damage breakdown"
date = 2026-05-20

[extra]
math = true
enable_toc = true

[taxonomies]
tags = ["minecraft", "datamine"]
+++

Damage values are to be interpreted on a 20 scale. For example, 15 damage = 15 HP = 7.5 hearts.

Unless mentioned otherwise, this post only applies to Minecraft 1.8, mechanics in newer versions may be different.

# Melee damage

Players have a base attack damage of 1. The held item can add to this damage, for example:

* Wood and gold swords give an extra 4 base damage. (5 total)
* Stone swords give an extra 5 damage. (6 total)
* Iron swords give an extra 6 damage. (7 total)
* Diamond swords give an extra 7 damage. (8 total)

## Critical hits

When the player attacks another entity while falling, the base damage of the attack is multiplied by 1.5.

The player can't deal critical hits under any of the following conditions:

* The player is on the ground, or not falling.
* The player is in water (lava and cobwebs are fine).
* The player is touching ladders or vines.
* The player has the Blindness effect.
* The player is riding another entity.

<!-- TODO: what if fallDistance doesn't get reset and the player goes up -->

## Damage enchantments

Sharpness applies before critical hits, it increases the base damage by 1.25 per level. The monster-specific enchantments are added at the end, adding 2.5 total damage per level.

## Weapon attributes bug

Due to a [bug](https://bugs.mojang.com/browse/MC/issues/MC-28289) that's still not fixed in 26.1, when switching between items with different damage attributes (e.g. fist and sword, or two different swords) and attacking immediately after, the correct item will be used but with the wrong damage.

For example, when switching between fists and a wooden sword enchanted with Fire Aspect, the sword will lose durability and the enemy will catch on fire, but they will only receive 1 damage.

This was [fixed](https://github.com/ProjectKig/KigPaper/commit/e25a1dd8847e5c8c8b170439c1eba89a8e0492c9) on [KIG Network](https://playkig.com) in November 2025.

# Arrow damage

The base damage for arrows is 2. The Power enchantment further boost this by $0.5 \cdot \text{level} + 0.5$.

When an arrow hits an entity, the base damage is multiplied by the arrow's current speed (i.e. the length of its motion vector) and rounded up.

The arrow's initial velocity is determined by the player's normalized direction vector and the bow charge time.
The initial speed ranges between 0.3 and 3 m/t (meters or blocks per tick), and is calculated as $c^2 + 2c$ where $c$ is the charge time in seconds. This shows that full charge is 1 second, and the minimum charge is ~0.14 seconds.  
An approximately normally distributed vector with values in $(-0.0075, 0.0075)$ is added to the velocity before it is multiplied by the initial speed.

Each tick during which the arrow doesn't collide with an object, the arrow's velocity is multiplied by **drag** (0.99 in air and 0.6 in water), and **gravity** (0.05) is subtracted from the vertical component. From this we infer that shooting an arrow downwards (assuming it hits the target) reduces both travel time (thanks to gravity further increasing the downwards vertical velocity) and the effect drag has on the arrow (because the horizontal components are smaller). This allows maintaining most of the initial speed on impact, as well as potentially increasing it further the more the arrow travels downwards.   
This [tool](https://minecraft.wiki/w/Calculators/Projectile_motion) can be used to calculate the impact velocity of an arrow. It is calibrated for 26.1, but the movement logic is the same in 1.8 (although the initial velocity is calculated differently).

If the arrow hits an entity which causes it to bounce off (e.g. invincible mobs), the arrow's velocity is multiplied by -0.1 (i.e. 10% magnitude and opposite direction). This may cause a fully charged shot to deal less than the minimum damage.

If the bow was fully charged, the arrow becomes **critical** and gains a trail of crit particles.
When a critical arrow hits an entity, it will deal up to $\lfloor \frac{d}{2} \rfloor + 1$ extra damage in **random** increments of 1 HP, where $d$ is the original damage of the arrow.

For example, if the original damage is 6, a critical arrow would deal damage equal to a uniformly distributed integer between 6 and 10.  
If the original damage is 7, it would instead be between 7 and 11, and so on.

Here is a table with damage values for common scenarios at full charge, the highlighted speeds are the most common ones:

| Power Lv. (Base Damage) | Impact Speed | Min Damage | Max Damage (crit) | **Average Damage (crit)** |
| --------  | ---------- | ----------- | ---------- | ----------------- |
| None (2) | **2.5-3 m/t** | 6 | 10 | **8** |
| None (2) | 3-3.5 m/t | 7 | 11 | **9** |
| Power 1 (3) | **2.67-3 m/t** | 9 | 14 | **11.5** |
| Power 1 (3) | 3-3.33 m/t | 10 | 16 | **13** |
| Power 1 (3) | 3.33-3.67 m/t | 11 | 17 | **14** |

In the table above, the speed ranges are start-exclusive. Higher speed values can be achieved by shooting downwards, as explained above.

# Fall damage

The base fall damage is the entity's fall distance on the previous tick, rounded up. Then:

* Damage is reduced by 3, meaning entities have to fall at least 3 blocks before they take fall damage, and the resulting
  damage is calculated over the excess distance.
* Each level of Jump Boost reduces damage by 1.

# Explosion damage

Explosion damage is calculated as
$$\text{damage} = 1 + 8 \cdot \text{power} \cdot \frac{\text{impact}^2 + \text{impact}}{2}$$
where
$$\text{impact} = \left( 1 - \frac{\text{distance}}{2 \cdot {\text{power}}} \right) \cdot \text{exposure}$$

Explosion power is 3 for creepers, 4 for TNT, and 6 for charged creepers.  

Distance is measured relative to the player's feet position and the explosion center.
For TNTs this is the entity's position, but shifted to the vertical midpoint of the entity. For creepers the center is the creeper's feet position. A patch on KIG Network gives them a quirk similar to TNTs, which fixes creepers dealing very little damage while standing on carpets, snow layers, etc.  
Because those use the entity's top-left corner, explosions aren't perfectly centered and deal slightly more damage to entities closer
to that corner.

Exposure is calculated by casting rays from the explosion center to a set of points on the player's bounding box, to determine how much of the explosion
is obstructed by blocks. The result is the ratio between the number of unobstructed rays and the total rays cast. What's notable is that a ray counts as obstructed if it hits **any** block that isn't water, lava, or fire. This even includes passable blocks like tall grass and flowers. As explained above, because the explosion center is usually at or near the bottom of the source, a TNT that for example is inside a flower block will deal the minimum damage (1) to any entity it affects.

<iframe
    src="https://www.youtube.com/embed/87p-I0jfXsk"
    webkitallowfullscreen
    mozallowfullscreen
    allowfullscreen
    width="640" height="360"
    referrerpolicy="strict-origin"
>
</iframe>

Explosion damage is affected by difficulty:
* On Peaceful, players take no damage from explosions.
* On Easy, damage is halved then increased by 1.
  * In 1.8, if the base damage is 1, this causes it to deal 1.5 damage instead. This is fixed in the latest version. 
* On Hard, damage is multiplied by 3/2.

Minigames on KIG Network run on Easy difficulty.

# Reducing damage

After the final damage of an attack is calculated, it is then reduced in accordance with the target's defensive effects.

* For anvil and falling block damage, wearing any helmet reduces the damage by 25%.
* Blocking reduces damage by $\frac{d - 1}{2}$ (i.e. by 50%, but leaving an extra 0.5 damage). This doesn't affect damage that ignores armor, i.e. burning, drowning/suffocation, starvation, fall, void, thrown potions, and poison/wither. The list notably does not include:
  * Fire and lava contact damage, you can indeed reduce the damage received for standing in fire or lava by blocking (or wearing armor), just not the ticking burning damage.
  * Cactus damage, though because cacti only inflict 1 damage, the extra 0.5 damage from the blocking formula nullifies its effect, so armor is needed here.
* For damage causes that don't ignore armor (see above), damage is reduced by 4% per half-shield of armor resistance (as seen in the HUD).
* The protective armor enchantments are applied.
* The Resistance potion effect reduces non-starvation damage by 20% per level.
* Absorption hearts are consumed and used to reduce the damage further.

Protective armor enchantments stack together with different weights (all weights are rounded down):
* Protection (for all causes except starvation and void) has a weight of $1.5 + \frac{1}{4} \cdot \text{level}^2$.
* Fire Protection (for fire, lava, and fireballs) has a weight of $2.5 + \frac{5}{12} \cdot \text{level}^2$.
* Blast and Projectile Protection have a weight of $3 + \frac{1}{2} \cdot \text{level}^2$.
* Feather Falling has a weight of $5 + \frac{5}{6} \cdot \text{level}^2$.

The weights are added together from each armor piece (for the enchantments matching the damage cause), capped at 25, then a random number between the result and half of it is taken, which is then capped at 20. The resulting damage reduction is $4\\% \cdot \text{random total weight}$ (so the maximum reduction is 80%).

# Health regeneration

Players have several ways to regain health.

* Every tick, if the player's food level is 18 or higher (90%) and the player is missing health, the natural regeneration timer is incremented, otherwise it is reset. When it reaches 80 (4 seconds worth), the player regains 1 HP.
* On Peaceful difficulty, players regen an additional 1 HP every 20 ticks (1 second).
* The Regeneration effect gives 1 HP every $\lfloor \frac{50}{\text{level}} \rfloor$ ticks. The timer starts when the effect is obtained.

A fun takeaway is that because the Regeneration and saturation timers can be desynced (e.g. by delaying the potion or taking damage), it is possible to regen 2 HP within a regular 10-tick damage cooldown. Although very rare, this gives a way to e.g. survive two critical hits from an iron sword.

# Damage cooldown

When an entity takes damage, they are invulnerable for 10 ticks (0.5 s). 

During this period, the last damage taken is stored, and if the entity receives new damage that is strictly greater than the last damage, the difference in damage is dealt bypassing the invulnerability. The last damage is also updated with the new damage, but the invulnerability timer is not reset, and the entity doesn't flash red again. 

This, for example, makes it so:
* Fall damage deaths can't be avoided by taking invulnerability from a weaker source.
* It is possible to hit with a weapon, switch to a stronger one, then hit again quickly to deal the stronger weapon's damage.
* It is possible to hit on the ground, jump, then hit again (fast enough) while falling to turn the hit into a critical hit.
* It is possible to take multiple explosions on the same tick, only the highest damage will apply (but the knockback will stack, see bottom).

When dealing the damage difference, it's not technically the same as if the original damage came from the new source, as the difference is considered a new damage instance, so it can be reduced independently from the original damage. For example, consider this scenario:

* Player deals a critical hit with an iron sword, for $7 \cdot 1.5 = 10.5$ damage.
* Damage cooldown expires.
* Player deals a regular hit with an iron sword, for 7 damage.
* The opponent starts blocking.
* Player deals a critical hit with the iron sword, during invulnerability, so $10.5 - 7 = 3.5$ damage should be dealt. However, because the opponent is blocking, the actual damage dealt is $3.5 - \frac{3.5 - 1}{2} = 2.25$.
* Damage cooldown expires.

In the above scenario, the opponent has taken $10.5 + 7 + 2.25 = 19.75$ total damage. Had the second hit been a crit to begin with, the opponent blocking wouldn't have mattered and they would have taken $10.5 + 10.5 = 21$ damage instead.

During invulnerability, the entity can't take additional knockback except from explosions. They can, however, take the extra knockback from sprinting and the Knockback enchantment, for as many times as the damage is changed. In multiplayer, however, this only works on mobs, not players.
