+++
title = "Xenoblade 3: Random factors in damage calculation"

[extra]
math = true

[taxonomies]
tags = ["xenoblade", "datamine"]
+++
This analysis focuses on two hidden factors that are randomly determined when
calculating damage. I've made sure that with these disabled damage output is deterministic, barring critical hits and blocked attacks.

Special thanks to beta382#3088 from the [Xenoblade Chronicles Discord](https://discord.gg/xenoblade) for helping with research.

## Weapon stability
Your "Attack" statistic is not exactly the initial damage value for damage calculation. Instead, the weapon you use determines a random value that is added on top.

As for all weapon-related mechanics, this only depends on the weapon and *not on the class*, meaning Master Arts will use their weapon's Stability value.

The effective initial damage value will be a random value in the range $[Attack, Attack + \text{Weapon Damage} \cdot Stability]$, where $Attack$ is the "Attack" stat displayed in the Characters menu.

$\text{Weapon Damage}$ depends on your character's level, so I've left a list of data tables you can check.

| Weapon class | Stability | Stats |
| ----- | --------- | ----- |
| Ouroboros (All) | 0 | Not relevant |
| Swordfighter | 8% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam01.html)* |
| Zephyr | 5% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam02.html)* |
| Medic Gunner | 3% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam03.html)* |
| Tactician | 2% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam04.html)* |
| Heavy Guard | 12% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam05.html)* |
| Ogre | 16% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam06.html)* |
| Flash Fencer | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam07.html) |
| Yumsmith | 10% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam08.html) |
| Strategos | 8% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam09.html) |
| Troubadour | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam10.html) |
| Seraph | 16% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam11.html) |
| Lost Vanguard | 8% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam12.html) |
| Lifesage | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam14.html) |
| Royal Summoner | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam15.html) |
| War Medic | 2% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam16.html) |
| Guardian Commander | 6% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam17.html) |
| Thaumaturge | 8% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam18.html) |
| Incursor | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam19.html) |
| Stalker | 2% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam20.html) |
| Lone Exile | 8% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam21.html) |
| Soulhacker | 16% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam22.html) |
| Signifer | 6% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam23.html) |
| Machine Assassin | 10% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam24.html) |
| Martial Artist | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam25.html) |
| Full Metal Jaguar | 4% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParam26.html) |
| Swordfighter (Lucky Seven) | 0 | Not relevant |
| Sharpshooter | 5% | [Link](https://xenobladedata.github.io/xb3/BTL_WpnParamS2.html) |
| Noponic Champion | 12% | [Link](https://xenobladedata.github.io/xb3/6B1CFB92.html)

*: If the character has the upgraded version of the weapon (Ch. 7), use the `Damage2` column instead.

As an example, consider this scenario. My Lv. 66 Noah is using the 
Flash Fencer weapon. On the Characters screen I see an "Attack" value of 672. 
My weapon's Stability is $\text{Damage}_{FF}(\text{Lv66}) \cdot 4\%$. By looking at the tables, I can see that the Dual Rapiers's `Damage` at level 66 is 313.
Therefore, Stability is $313 \cdot 4/100 \approx 12.52$ and my initial damage
value will fluctuate between 672 and 684 (round down).

For Master Arts, you first need to get your Attack stat without your main weapon's modifier.
This is tricky because equipment that boosts your Attack stat is applied after the weapon modifier. For the sake of simplicity, let's consider a scenario with no Attack-boosting accessories. For instance, let's say my Flash Fencer Noah from the previous example uses Wide Slash, a Master Art from Zephyr. I already know the weapon damage modifier for Lv. 66 Flash Fencer is 313, so I subtract that from 672 to get 359. This means my Attack with the Zephyr weapon equipped (my class is still Flash Fencer) is $359 + \text{Damage}_{Zephyr}(\text{Lv66}) = 359 + 308 = 667$. Zephyr's Stability is $308 \cdot 5/100 \approx 15.4$, so my Wide Slash base damage value will be within $[667, 682]$.

## Random variance
There is another random element in damage calculation, which is a multiplicative factor near the end of the formula. Unlike stability, this also affects Interlink and Lucky Seven.

As a reminder, attacks are considered *preemptive* when they hit an enemy that hasn't noticed you (this works both ways). You can get at most one preemptive attack per enemy, and you don't get preemptive chances on bosses/forced fights.

The factor for preemptive attacks is a random number between 1.0 and 1.1. This means damage will vary between 100% and 110%. For all other attacks, the factor ranges between 0.9 and 1.1, meaning you will have higher fluctuation,
from 90% to 110%.
