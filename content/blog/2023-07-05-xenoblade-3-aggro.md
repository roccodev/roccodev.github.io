+++
title = "Xenoblade 3: Aggro mechanics explained"

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

This post describes how aggro generation works in Xenoblade 3.

Overall, I don't think the aggro mechanics in this game are well designed. With high-damage scenarios, and
especially with how aggro interacts with chain attacks, it's almost impossible to draw aggro off of the
main damage dealers.

## Aggro slots

Each enemy has four slots for each character in the field. The meaning of those slots is unknown, but slot
\#1 is generally used for "active" aggro, slot \#2 for "passive" aggro, and slot \#3 for Taunt aggro. I haven't
observed slot \#4 ever being used. (It is possible that it's used for Eclipse Soul fights, but I haven't tested it.)
In reality, when choosing the attack target only the total is considered, and multiplications act on all four slots, so it doesn't really matter.

## Aggro types

What actually matters is how aggro is generated.  
When attacking, there are eight main ways to generate aggro:

1. Damage dealt
    + Aggro: `damage * 0.5 * fusion bonus * additives`
    + *Fusion bonus*: 1.5 for applicable arts
2. Damage taken
    + Aggro: `-1.5 * damage`
3. HP healed
    + Aggro: `1.25 * HP difference * additives`
    + *HP difference*: total HP actually healed
    + Aggro is gained by the healer
4. Arts and auto-attacks
    + Aggro: `2000 + 120 * (chr. level - 1) * additives` (same formula for 5, 6, 7)
5. Healing arts
    + Includes buff arts that generate aggro
7. Field Arts
8. "Extra" effects, like +aggro on evasion, etc.
9. Taunts
    + Aggro: `2000 + 120 * (chr. level - 1) * additives * shackle ring`
    + *Shackle ring*: max 1.8 at an art chain of 6

### Additives

Additives include all effects that "boost aggro generated when ...". The multiplier usually starts at 1.0, and effects that hinder aggro generation are subtracted instead. In most cases, it is not possible for aggro that should be "gained" to go negative as the result of that effect; instead, no aggro is gained. This allows for silly builds (e.g. Stalker Night Hunt) that generate little to no aggro, even with millions of damage.

Additionally:
* "Boosts aggro generated from dealing damage" only affects type 1 aggro.
    + This is different from "Reduces aggro generated from attacks", which works on both 1 and 4.
* Aggro types 4, 5, and 6 have a base multiplier of `Hate / 100`, where `Hate` is from `BTL_Arts_PC`.
* For type 7, the base multiplier is defined by the effect itself. For example, the highest obtainable value for the evasion effect is `6.5% = 0.065`. (from one of Masha's accessories)
 
## Choosing targets

The aggro manager checks regularly whether it should change an enemy's target. The character with the highest total aggro becomes the new target.

In case of a tie, the character that is higher in the list takes precedence (e.g. Noah over Mio). Ouroboros forms come after the 7th member.

Characters that are suffering from Eclipse Soul are excluded from the calculation. Additionally, Eclipse Soul doesn't follow the normal targeting rules. Instead, it tries to hit a random character with the most populated role, choosing Attacker > Defender > Healer if tied.  

## Reactions

Generally, reactions only affect aggro indirectly, e.g. by providing damage bonuses. The Daze line is an exception,
however:

* Out of the eight aggro generation types listed above, only Taunts actually generate aggro during Daze.
* When the enemy is inflicted with Burst, all Defenders in the party gain +20% aggro, while everyone else gets their aggro reduced by 20%.

## Reviving

Reviving generates aggro equal to `1000 + (dead chr. level - 1) * 60`. This aggro is then divided between the
characters that participated. This includes Memory Locket users, but there is no additional penalty for the item,
unlike what the caption implies.

For Liberty Wing, aggro is split in half between Eunie and Taion.

Note that when a character dies, their aggro is reset (unless they have effects preventing it).

## Interlinking

Ouroboros forms are considered different characters and have separate aggro slots.

When a pair of characters interlinks, their current aggro is frozen, and the newly spawned Ouroboros form gains
the sum of the two characters's aggro.

Additionally, if the user who initiated the interlink (which of the two characters is important) is a Defender[^1], the Ouroboros's starting aggro is multiplied by 2 (Interlink Lv. 1), 3.5 (Lv. 2), or 6 (Lv. 3). 

Switching between forms doesn't affect this bonus or aggro at all, as aggro is shared between the two forms of each pair.

When the interlink is canceled, the two characters get their old aggro back, and the Ouroboros form's aggro is deleted.

## Over-time aggro generation/reduction

Every second, the aggro manager runs some checks to give or take aggro from characters.  
"Additives" here are strictly the ones affecting aggro gain/reduction over time.

If any character is at least 12.5 units away from the enemy, they lose `(160 + 60 * (chr. level - 1)) * additives / 30` aggro per second from that enemy.

Regardless of distance, every character loses `75 * additives` (or `50 * additives` if Defender) per second from every enemy.

Some accessories and skills grant characters aggro generation over time. For example, Defender Lucky Seven generates `30 * additives` aggro every second.

## Chain Attacks

For some odd reason, in Chain Attacks you can only *gain* aggro, with the only exceptions being chain orders that explicitly reduce it.

Most of the aggro manager's functions (all prior to 1.3.0, see below) don't run at all, so there is no aggro generation or reduction over time, as well as no visible target switching from the enemy.

The eight main ways for gaining aggro still work (meaning you will get a *ton* of aggro, especially with how crazy damage in chain attack gets), with only a few ways to reduce it.

Lanz's chain order multiplies every Defender's aggro by `1.0 + effect` (max. 1.5 at Lv. 81), while Mio's multiplies every non-Defender's aggro by `1.0 - effect` (min. 0 at Lv. 81).

Lanz's effect uses a function in the chain attack logic to multiply aggro. Mio's effect, however, uses a function in the aggro manager, despite being pretty much the same effect. As noted earlier, before 1.3.0 no function from the aggro manager would be run during Chain Attacks, so Mio's chain order would have no effect.  
Curiously, the effect was actually implemented, and a check for Chain Attacks was also present for the effect,
but logic would be halted earlier when inside a Chain Attack.

In 1.3.0, the logic for Mio's effect is the only function run during chains from the aggro manager. As mentioned, getting aggro to change during them does not immediately result in a target switch. Instead, the enemy will switch targets once the chain attack ends.

---

To test things for this article I've mainly used a debugger, but I've also designed a very barebones aggro visualizer that uses debug functions to draw text. To continue research I plan to submit that to [xenomods](https://github.com/BlockBuilder57/xenomods) at some point, so that people who want it can start optimizing builds.

---

[^1] <a id="1"></a> **Did you know?** When interlinking at lv. 1 or higher, there's a special effect based on the character's role. Defenders gets extra aggro on the interlink, Healers give AOE heal and regeneration, while Attackers inflict fixed damage that scales with Attack.

