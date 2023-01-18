+++
title = "Xenoblade 3: Chain attack damage ratio analysis"

[extra]
math = true

[taxonomies]
tags = ["xenoblade", "datamine"]
+++

The damage multiplier in chain attacks can be somewhat of a mystery at times, and little to nothing can be found in the official documentation.
<!-- more -->

When you start a chain attack, an initial damage ratio is set:

$$\text{Initial Ratio} = 100 + (Base \times Difficulty\text{%})$$

$Base$ is $50$ for normal rounds and $200$ for Ouroboros rounds.  
$Difficulty\text{%}$ is $100\text{%}$ on Normal/Easy, and $50\text{%}$ on Hard/Very Hard.

At the end of every round, if you complete the order (by reaching at least 100 TP),
the game first takes note of which characters used an art during the round, and checks them against the order's character.

Making a "special pair" will give you a higher chance of getting the highest ratio gain for the round. Depending on the order's character, for a round to have a "special pair" you need to use:

* If the order's character is a Hero: the class heir, you can find it in the Classes menu because their aptitude is S Rank.
    * For example, if you're building towards Ashera's order, you can use Eunie to get the "special pair" bonus.
    * Note: this is fixed for every Hero, using accessories to boost the rank from A to S does nothing. 
* If the order's character is not a Hero: either their Ouroboros partner, or one of the Heroes they inherit a class from.
    * This is also the case for Ouroboros orders: you will have to remember who has which form to pick the right character.

Here is a table with all the possible pairs. For the sake of readability, Heroes are only listed once.

<details>
<summary>
  ➜ (Click) Pair list with Heroes (warning: post-game spoilers)
</summary>
   
<br />

<table>
<thead>
<tr>
<th>Character</th>
<th><strong>Noah</strong></th>
<th><strong>Mio</strong></th>
<th><strong>Eunie</strong></th>
<th><strong>Taion</strong></th>
<th><strong>Lanz</strong></th>
<th><strong>Sena</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Noah</strong></td>
<td></td>
<td>✓</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td><strong>Mio</strong></td>
<td>✓</td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td><strong>Eunie</strong></td>
<td></td>
<td></td>
<td></td>
<td>✓</td>
<td></td>
<td></td>
</tr>
<tr>
<td><strong>Taion</strong></td>
<td></td>
<td></td>
<td>✓</td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td><strong>Lanz</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>✓</td>
</tr>
<tr>
<td><strong>Sena</strong></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>✓</td>
<td></td>
</tr>
<tr>
<td><strong>Heroes</strong></td>
<td>Ethel, Zeon, Juniper, Melia</td>
<td>Teach, Miyabi, Nia</td>
<td>Gray, Ashera, Monica</td>
<td>Isurd, Alexandria, Cammuravi</td>
<td>Valdi, Fiona, Triton</td>
<td>Riku/Manana, Ghondor, Segiri</td>
</tr>
</tbody>
</table>


</details>

Now we can get to how damage ratio gains are actually calculated.  
At the end of each round, the game runs the following formula:

$$\text{Ratio Gain} = (Base + RoundBonus + ResultBonus) \times HeroicChain \times Difficulty\text{%}$$

$Difficulty\text{%}$ is the same as before, while $Base$ is $100$ for normal rounds, and $200$ for Ouroboros rounds. (This is what the order description means by "Greatly increases damage ratio.")  
$HeroicChain$ only applies if you use Gray during the round, in which case it will have a value of $1.5$.

The formula introduces two bonuses that are added to the damage ratio.  
Here is how $RoundBonus$ is calculated:

$random(a, b)$ is a function that returns a randomly generated number between $a$ and $b$, inclusive.

| Round | Normal Pair ratio gain | Special Pair ratio gain | Max gain |
| ----- | ---------------------- | ----------------------- | -------- |
| 1, or any Ouroboros round | 0 | 0 | 0 |
| 2+ | `random(25, 50) * (round - 1)` | `random(35, 70) * (round - 1)` | 50 |

As you can see, the amount of round bonus you can gain in a single round is capped at 50, so advancing further or making special pairs only gives you a higher chance at max gains. (More specifically, the maximum becomes guaranteed starting from the third round.)  
Unfortunately, there is no round bonus for Ouroboros rounds or the first round of any chain attack.

$ResultBonus$ is a little different, as it depends on the round's result. It doesn't increase as rounds go on, but it has a higher cap for "Amazing" rounds.


| Result | Normal Pair ratio gain | Special Pair ratio gain | Max gain |
| ------ | ---------------------- | ----------------------- | ----- |
| Cool (100 TP) | 0 | 0 | 0 |
| Bravo (150 TP) | $random(25, 50)$ | $random(35, 70)$ | 50 |
| Amazing (200 TP) | $random(75, 150)$ | $random(100, 200)$ | 150 |

Again, special pairs don't necessarily increase the final ratio gain, but they give a higher chance to get the max value.

Also, please note that the exact number of TP obtained doesn't matter, you just have to meet the three thresholds.

### Additional gains

Some Heroes increase the damage ratio by a net amount when their order is completed. This occurs after the regular ratio gain described above.  
 The hero's special gain is still affected by the difficulty multiplier, however:

$\text{Ratio Gain} = Amount \times Difficulty\text{%}$

### Example scenarios

Normal Mode, no special pairs, first round, "Cool" (< 150 TP)

| Bonus | Value |
| ----- | ----- |
| Base | 100 |
| Difficulty% | 100% |
| Round Bonus | 0 |
| Result Bonus | 0 |
| Heroic Chain | 0 |
| **Total** | **+100** |

---

Hard Mode, with special pair, round #3, "Amazing" (200+ TP)

| Bonus | Value |
| ----- | ----- |
| Base | 100 |
| Difficulty% | 50% |
| Round Bonus | 50 (guaranteed max at round #3) |
| Result Bonus | 100-150 (random) |
| Heroic Chain | 0 |
| **Total** | **+125-150** |

---

Easy Mode, Ouroboros order, no special pairs, round #2, with Gray, "Bravo" (150-199 TP)

| Bonus | Value |
| ----- | ----- |
| Base | 200 |
| Difficulty% | 100% |
| Round Bonus | 25-50 (random) |
| Result Bonus | 25-50 (random) |
| Heroic Chain | 150% |
| **Total** | **+375-450** |
