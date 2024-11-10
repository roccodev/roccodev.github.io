+++
title = "Xenoblade 3: TP storage glitch breakdown"
date = 2024-02-13

[taxonomies]
tags = ["xenoblade", "glitch"]
+++

# Background

For those unfamiliar, runners in the [Chain Attack damage category](https://www.speedrun.com/xc3?h=Chain_Attack_Damage) try to achieve the maximum total damage in Chain Attacks.

On Feb 12, 2024 (CET), the Xenoblade 3 runner [UnKNOWn_Cross](https://twitter.com/UnKNOWn_Cross) revealed their new
[Xenoblade 3 Chain Attack damage record](https://twitter.com/UnKNOWn_Cross/status/1756865314917433754).

However, UnKNOWn_Cross decided to put the record on hold while waiting for approval on a "glitch" that boosted the total damage by delaying TP gain. TP (*Tactical Points*) are one of the key elements of Chain Attacks. For an in-depth breakdown, you can check the [Chain Attack page](https://www.xenoserieswiki.org/wiki/Chain_Attack_(XC3)) on Xeno Series Wiki.

As you can see in the posts showcasing it, the glitch causes TP gained during certain Chain Attack rounds to magically disappear, only to appear again all at once in a different round.

This glitch has apparently been known since at least late 2022, with one [tweet](https://twitter.com/genkai_bk/status/1565646632603254785) from [genkai_bk](https://twitter.com/genkai_bk) showcasing it in September, followed by another [tweet](https://twitter.com/RabbitTigerCrab/status/1576121220181135361) from [RabbitTigerCrab](https://twitter.com/RabbitTigerCrab) in October.

The Chain Attack speedrun scene was interested in the details of this glitch, so I went on to research it. In particular, I'd like to thank yintse99 and Krark from the [Xenoblade Speedrunning Discord server](https://www.speedrun.com/xc3) for their help.

# TP storage

Here is a quick rundown if you are only interested in how to trigger the glitch and its effects. 

TP storage involves storing *undisplayed* TP to freeze them until a subsequent round. TP stored with the glitch persist across:

* Chain attack rounds
* Chain attacks (i.e. starting a new Chain Attack carries over the undisplayed TP from the previous one)
* Battles (the new Chain Attack can be from a different battle)
* Warps and inter-world warps
* Save files (the new Chain Attack can be from the same or different save file, even after reloading)
* Future Redeemed: performing the glitch in the base game can carry its effects over to Future Redeemed, until the game is closed. ([Video proof](https://www.youtube.com/watch?v=NmkYCFOG4Ds))

The only way to reset the TP storage is either by *displaying* the TP, or by restarting the game.

To trigger the TP storage glitch, you need to **trigger more than 50 TP instances while the game is displaying 5 bubbles**. For example, you can get the **Damage TP** instance on **more than 50 enemies at once**. For an example of this, there is a practical demonstration at the end of the post.

**Update 1**: If you can stack enough TP sources, it is possible to get it with fewer enemies. Here are examples of me getting it with only [45](https://www.youtube.com/watch?v=SCaBOxvvkYQ) or [38](https://www.youtube.com/watch?v=fq8I8vjdBjU) enemies.

**Update 2**: Character TP (i.e. TP a character has after being reactivated) does not actually take the carried-over TP into account, with notable exceptions. See the [last paragraph](#regarding-character-tp) for details.

# Gaining and displaying TP

When you are awarded TP by performing certain actions, the TP type and amount are packaged together into an *instance*, and added to a queue, which is described in the next paragraph.

However, TP isn't actually displayed or added instantly. Instead, a separate entity which I'll call the *display handler* removes instances from the queue at specific intervals. When this happens, the TP bubble appears on screen and is added to the total.

# The TP queue

TP instances (or "bubbles") are added to a queue that can hold 50 instances maximum. For example, let's visualize a queue with a capacity of 6 that is half-full:

```
[ TP | TP | TP | -- | -- | -- ]
  o              ^
  |              next slot to be replaced
  next slot to be displayed
```

The `^` denotes the next slot to be replaced. When you gain TP, the `^` slot (or *head*) is replaced with the new TP data, and the `^` pointer is advanced.  
When a bubble is displayed and added to the total TP, its slot (`o`, also called the *tail*) is made empty (`--`), and the `o` pointer is advanced.

The next time you get TP, the queue will look like this:
```
[ TP | TP | TP | TP | -- | -- ]
  o                   ^
```

Let's say you stop at 4 TP bubbles and the game tries to display them. In truth, the display handler does
not know how many slots are occupied. Instead, **it keeps going until it finds an empty slot**. 
Starting from the first occupied slot, the queue becomes:
```
[ -- | TP | TP | TP | -- | -- ]
       o              ^
```
The first bubble is displayed and its slot is emptied. At the end, the queue looks like this:
```
[ -- | -- | -- | -- | -- | -- ]
                      o^
```

Now, let's add 3 more TP bubbles:
```
[ TP | -- | -- | -- | TP | TP ]
       ^              o
```
Because we've reached the end of the queue, we add new entries at the start, wrapping around.  
Let's try to display the new entries. The display handler performs the same wrap-around operation, until
it finds an empty slot:
```
[ -- | -- | -- | -- | -- | -- ]
       o^
```

Now, let's examine the problematic scenario. What happens if we add, for example, 8 TP bubbles? We know that 2 will be discarded because there isn't enough space. But let's examine it in detail, before and after the wrap-around:
```
[ -- | TP | TP | TP | TP | TP ] (still need to add 3 bubbles)
  ^    o
```
```
[ TP | TP | TP | TP | TP | TP ]
       o         ^
```
So far so good, it's all as expected. But now let's see what the display handler tries to do.
```
[ -- | -- | -- | -- | -- | -- ]
       o         ^
```
The display handler went on to display and clear occupied slots until the first empty slot, but none of the slots were empty! That means it completed one full cycle, but notice that the **head and tail are no longer in sync**! Because there were no empty slots, the display handler could not know where the queue truly ended.

What happens now when we add and display bubbles? Let's add a bubble:
```
[ -- | -- | -- | TP | -- | -- ]
       o              ^
```
and let's try to display it:
```
[ -- | -- | -- | TP | -- | -- ]
       o              ^
```
Nothing happened! That's right, the tail is currently pointing to an empty slot, so it can't do anything and won't advance.

We need to wait for the head to wrap around and reach the tail again, by adding more bubbles.

# Pratical demo

Let's analyze a real scenario. I've described the TP queue with 6 slots as an example. The game uses a queue with a capacity of **50 slots**.

I've uploaded video footage of this test [here](https://www.youtube.com/watch?v=vkDtbNCtJKA), I recommend using it to follow along. To make it easier for me, I have changed the enemy count and made it easier to kill all the enemies at once. Most Chain Attack runners have legitimate ways to replicate this setup, however.

Initially, all slots are empty. I get 4 "regular" TP which could be displayed in time, so now the first 4 slots are empty. When I spam damage TP, I basically occupy 55 slots, meaning the whole queue is now Damage TP. I also got some more TP along the way (Critical Hit, etc.), so at the end, the next slot that will be replaced is #18. Let's visualize this:
```
[ TP | TP | TP | TP | TP | TP x 12 | TP | TP x 32 ]
                      o              ^
```

As I've explained, because the queue is completely full, when we try to display the bubbles, we basically end up emptying the entire queue while **going back to the same slot we started on**. This causes the desync: now the tail is still at slot 5, while the head is at slot 18.
```
[ -- | -- | -- | -- | -- | -- x 12 | -- | -- x 32 ]
                      o              ^
```

So I enter the "empty" phase during which none of my TP is displayed. I need 33 + 5 = 38 bubbles to reach the tail again, after which the queue looks like this:
```
[ TP | TP | TP | TP | TP | -- x 12 | TP | TP x 32 ]
                      o^
```
Notice the 12 empty slots in the middle? Let's see what the display handler does next. First, it sees that the 5th slot has TP, so it displays it and clears the slot. The 6th slot is empty, so it stops there. (In the video, I got a "lone" bubble when they started appearing again.)

The next 12 slots are uneventful, bubbles appear and slots are cleared normally as if the glitch was inactive. (Basically, the head and tail walk hand-in-hand, or they at least catch up to each other.)  
The queue now looks like this:
```
[ TP | TP | TP | TP | -- | -- x 12 | TP | TP x 32 ]
                                     o^
```
When the display handler runs, it sees that the 18th slot is occupied, and displays it. But then it also notices that the next slot has TP, and so do the next 35! It then displays and clears them all at once. In total, it has displayed the 50 bubbles I missed.

The queue is now empty, but the head and tail are also desynced again!
```
[ -- | -- | -- | -- | -- | -- x 12 | -- | -- x 32 ]
                      o              ^
```

In fact, the glitch seems to keep the desync period. It is also worth noting that the queue, head, and tail persist across **battles**, **warps**, and even **save loads**! This [video](https://www.youtube.com/watch?v=ex4WFqA-jFQ) showcases the latter.

# Regarding character TP

When the game awards TP, that TP is instantly added to the character's total, that is, it doesn't wait for the TP bubble to appear to increase the character's score.

In the context of the glitch, this means that TP gained while bubbles aren't displayed is still saved to each character's score, not to the character that receives the bubbles at the end.

However, some heroic chain effects (notably, First Blood) completely replace the character's TP with the result of an operation applied to the **displayed TP** (in the case of First Blood, displayed TP multiplied by 125%). For example, in my first clip showcasing the glitch, Noah's TP went from 30 to 38, and no bubbles appeared. Technically, he still got the invisible TP added to his score, but it got replaced because of First Blood.

In particular, this means that under regular conditions, whoever receives the big drop of 50 TP instances does not actually get that TP added to their score. However, if that character has a heroic chain effect that affects the final TP result (e.g. First Blood again), then the score that is actually saved is based on the displayed TP, so the character actually gets the TP from the drop.
