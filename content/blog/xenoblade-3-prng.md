+++
title = "Xenoblade 3: why you need thread-safe PRNGs"
date = 2024-01-23
draft = true

[extra]
enable_toc = true

[taxonomies]
tags = ["xenoblade", "re"]
+++

## "Glitched" crafted accessories

Some users reported seeing one or even multiple instances of a "glitched" accessory when crafting, specifically an accessory with all parameters, including color,
effect, and stats, set to their minimum value.

![Glitched accessory](https://i.imgur.com/2xkE0BL.png)  
*Thanks Hamidu for the image!*

Originally I thought that maybe there was some correction code for errors that would
fall back to a default accessory, but that doesn't seem to be the case. It also doesn't seem to be a block of zeroed memory, as the stat values are correct.

An accessory is generated using 10 calls to the `rand` function (note that the names I use aren't official as the executable has no debug symbols), specifically to generate the color, effect, the stat types, and the stat values. Getting a minimum value accessory, assuming a well-distributed generator, would have too small of a chance to happen once, let alone multiple times.

## How numbers are generated

All the mainline Xenoblade games (I don't have a copy of 3D) use [Mersenne Twister](https://en.wikipedia.org/wiki/Mersenne_Twister)
for random number generation.

Here is an example implementation that mimics the one from the game, leaving
out unnecessary details:

```c
// Globals
static unsigned int MT_STATE_TABLE[624];
static int MT_TWIST_COUNTER = 624;
static unsigned int* MT_CURRENT_STATE = &MT_STATE_TABLE;

unsigned int rand() {
    bool needsUpdate = MT_TWIST_COUNTER < 1;
    MT_TWIST_COUNTER = MT_TWIST_COUNTER - 1;
    if (MT_TWIST_COUNTER == 0 || needsUpdate) {
        // This function resets MT_TWIST_COUNTER to 624,
        // and sets MT_CURRENT_STATE to point to the first entry
        // in the table
        mtTwist(&MT_STATE_TABLE);
    }
    
    unsigned int* curStatePtr = MT_CURRENT_STATE;
    MT_CURRENT_STATE = curStatePtr + 1;

    return mtTemper(*curStatePtr);
}
```

Setting a breakpoint in gdb for accessory crafting would reveal many `rand` calls attempted each frame:

```gdb
Thread 40 hit Breakpoint 1, ... in ?? ()
Thread 39 hit Breakpoint 1, ... in ?? ()
Thread 40 hit Breakpoint 1, ... in ?? ()
```

*Hmm, it's being called from multiple threads, surely there must be some kind of synchronization, right? ...Right?*

As it turns out, there is none. There is no lock to guard the generator's global state, no fences on increments, and no thread-local storage.
Interestingly, the implementation is largely the same since Xenoblade on the Wii, so perhaps the lack of synchronization is due to the Wii's [CPU](https://en.wikipedia.org/wiki/Broadway_(processor)) having only one core.  
In the code above, I have expanded the increment/decrement operations, to make it clear that they are not atomic. 

This leaves the code open to *race conditions*, when multiple threads try to access and update the global variables. Let's see
how this can affect the generated numbers.

It is worth noting that the game uses an internal scheduler that runs on the main thread that also manages tasks running on other threads. So while external threads (like the one I'm using for debugging) can run parallel to the main thread, the threads the game spawns must wait for the main thread before accepting new tasks. This means that `rand` calls made by the main thread (which includes most gameplay-related ones, including accessories) are generally not interfered with. However, I can confirm there are definitely race conditions
between other threads that can corrupt the state for future `rand` calls from any thread.

## Initial test: repeating values

I set up a script to run the `rand` function 1 billion times sequentially, as well as another 1 billion times in another thread.
The secondary thread only runs the function and discards the results.  
The script would count the number of zeroes rolled in total, as well as the length of any chain of consecutive zeroes.

TODO: change this to other numbers, zero is for OOB

In a single-threaded scenario, it was very rare to get even a single zero out of 1 billion rolls (generating numbers from 0 to
2,147,483,647).

In the multi-threaded test, however, I was getting many zero rolls (~220k!), and I was also seeing chains of up to 590 consecutive zeroes, which is suspiciously close to the maximum state count of 624.

This is caused by a race condition in the increment operation for `MT_CURRENT_STATE`. If another thread makes a `rand` call and reads the current value of `MT_CURRENT_STATE` before it is incremented by the first thread, but writes it after, the final value of `MT_CURRENT_STATE` will stay the same.

## Out-of-bounds reads

If there is a race condition during the `MT_TWIST_COUNTER = MT_TWIST_COUNTER + 1` operations, then there is a chance that the
`MT_CURRENT_STATE` pointer is incremented without updating the twist counter, causing it to exceed its buffer at the end
of the chain.

In a single-threaded scenario, it was very rare to get even a single zero out of 1 billion rolls (generating numbers from 0 to
2,147,483,647).
In the multi-threaded test, however, I was getting many zero rolls (~220k!), and I was also seeing chains of up to 590 consecutive zeroes, which is suspiciously close to the maximum state count of 624. I've also observed many zero-state reads in the multi-threaded scenario.
Normally, you'd get none because a good enough seed should prevent zero-states
from appearing as they are pretty hard to recover from (TODO ref). The `mtTemper` function is a series of shifts and XORs over the same value, so a zero-state always results in a zero output, i.e. `mtTemper(0) = 0`.

The reason for the zero-state reads, is that the function tries to read past the state table. Specifically, it may try to read
`MT_TWIST_COUNTER` (which is effectively a 625th state integer), or the lower bits of `MT_CURRENT_STATE`, which may be zero
depending on where code is relocated by the OS. On Yuzu, the address is on the order of `0x80000000`, so the lower bits are
all zero.

A race condition can also occur if the `MT_CURRENT_STATE` pointer is reset by `mtTwist`, and another thread's `MT_CURRENT_STATE = MT_CURRENT_STATE + 1` wins the race and sets the pointer back to the old value. Twists are particularly frequent given how many calls are made in a single frame, and if such a race occurs, the results can be quite significant. In particular, they can explain how I was getting 590 consecutive zeroes in the multi-threaded test.


(`rand(x) = rand() mod x`)

```
[Thread #n] rand(x) => rolled number (state pointer after the call)
---
[Thread #3] rand(100) => 25 (81C4CFA0)
[Thread #4] rand(100) => 19 (81C4CFA4)
[Thread #3] rand(100) => 63 (81C4CFA8)
[Thread #4] rand(100) => 1 (81C4C5EC) <== Thread 4 resets the state pointer
[Thread #4] rand(100) => 80 (81C4C5F0)
[Thread #4] rand(100) => 48 (81C4C5F4)
[Thread #3] rand(100) => 24 (81C4CFAC) <== Thread 3 doesn't care, out of bounds
```

Let's inspect the state after the last call.

```
(gdb) print/x *((int*)0x81c4cfa8)@624
$5 = {0x26e, 0x1, 0x81c4c5f4, 0x0, 0xcff777d, ..., 0x11, 0x0 <repeats 590 times>}
      ^           ^                  ^
      622         the state pointer  the seed
```

Notice the many zero values after `0x11`. Basically, if a race condition occurs such that the twist counter is reset while the state pointer is past its boundaries, the next 624 `rand` calls will read the values displayed above (including the 590 zeroes) instead of the actual state table.

Specifically, I've found one operation in particular to be problematic, as it is run by two threads for some reason. TODO

Thankfully, the pointer value of `MT_CURRENT_STATE` is only used to read data, as rewriting the state uses known offsets.

## Minimum values outside of accessories

It is worth introducing how the game generates a random *floating-point* number:

```c
float randf_inclusive() {
    // Because rand() is uniformly distributed over [0, 2^32 - 1],
    // it can generate a uniformly distributed random floating-point
    // number over [0.0, 1.0] with rand() / (2^32 - 1).
    return (float) ((double) rand() * 2.3283064370807974e-10); // (2^32 - 1)^-1
}

float randf_exclusive() {
    // This is not exactly 1/2^32. Due to the cast to float,
    // dividing by 2^32 would round to 1.0 if the number is too high.
    return (float) ((double) rand() * 2.3283041087743603e-10); // 0x3DEFFFFDE7410BE7
}

float randf_nonzero() {
    // Same as the exclusive version, but does not output 0.0.
    // (For the bounded version, it excludes the lower bound.)
    return (float) ((double) (rand() | 1) * 2.3283041087743603e-10);
}

float randf_bounded(float max) {
    // Same for exclusive
    return randf_inclusive() * max;
}
```

In particular, for all functions but the last, if `rand()` returns `0` (which can occur naturally, but it's rare), 
then the generated floating-point number is `0.0`.

There are several instances of the game failing to properly make a random check. For example, a chanced (??? TODO) integer
random check is correctly performed like this:

```c
int chance = 60;

if (rand() < chance) {
    // 60% chance to be here, i.e. we accept
    // any number from 0 to 59.

    // If chance = 0, then this block is never
    // executed, as anything < 0 evaluates to false.
    // (including 0!)
}
```

However, when it comes to floating-point checks:

```c
float chance = 0.6;

if (randf_inclusive() < chance) {
    // Correct? Not really, if chance = 1.0, and randf_inclusive() = 1.0,
    // then (1.0 < 1.0) => false, even though it should be 100%...
}

if (randf_inclusive() <= chance) {
    // Seems correct, right? Nope! If chance = 0.0, and randf_inclusive() = 0.0,
    // then (0.0 <= 0.0) => true, even though it should be 0%...
}

if (chance > 0.0 && randf_inclusive() <= chance) {
    // This is correct. A 0% chance should be ignored anyway.
}

// These are also both correct.
if (randf_nonzero() <= chance) {}
if (randf_exclusive() < chance) {}
```

The game erroneously uses the second version sometimes, failing to check for the chance to not be zero beforehand.  
This results in, for example:

* A guaranteed reaction or debuff, even if the chance would be exactly 0% (excluding immunities).
    * Note that the chance must be exactly 0, so if resistance or other effects lower it further, it's no longer guaranteed.
* A guaranteed critical hit, even if the chance is 0%.
    * Blocking and evading are not affected solely because their rates are always higher than 0.
* Enemies dropping rare, and even 0%-rate items, e.g. first defeat-only UM drops.

Obviously, all of the above need `randf_inclusive()` to also return `0.0`, which is very rare. The glitches
described earlier in the post might make such an event more common.

More generally, here are some other scenarios in which rolling a zero (either naturally or from corrupted state) would be beneficial:

* An attack that passes the accuracy check.
* A guaranteed blocked attack.
* Guaranteed Lucky Seven Doom (if enemy is eligible).

## How can this be fixed?

## RNG manipulation

To "manipulate" the random generator (i.e., predict all future outputs), it is mandatory that the only calls to the generator are the ones for the accessory. This requires:

* Disabling TAA
* Disabling SSAO
* Disabling certain post-processing effects
* An area with no wind (e.g. Agnus Castle)

The first three currently require modding. Also, note that giving accessories exclusive access to the generator would also prevent the issues described earlier. Though, even without these variables and assuming RNG calls are only made for accessories, inspecting the generator may not be trivial.

The hard part would be recovering the state. Recall that accessories use `rand(x) = rand() mod x` to generate values in the range [0,x), and `rand(x, y) = x + rand(y - x)` for [x, y).   
Usually, it would suffice to read 624 consecutive 32-bit outputs from the generator, which in our ideal case would mean generating 63 accessories. However, accessory generation does not output the full 32 bits, instead it operates modulo the max value, which changes every time. Specifically, the generator first grabs the next 32-bit value from the state and tempers it, then returns the result modulo the value, which means that the bits not covered by the modulo are lost.

If a mod is used to print out the current state, generating future outputs would be rather trivial, as Mersenne Twister is fully deterministic. This is ideal because you can only hold 300 accessories at once, and leaving the area/resetting to make space would advance the generator, invalidating the state.

I guess it comes down to personal views on whether using mods to manipulate RNG is legitimate. One could argue that if you can access the files, you could simply [edit your save](https://rocco.dev/recordkeeper) and give yourself optimal accessories. On the other hand, in modern Pokémon games, RNG manipulation is [best](https://www.pokemonrng.com/retail-usum-wormhole) [done](https://www.pokemonrng.com/retail-swsh-get-seed-with-cfw) with [mods](https://www.pokemonrng.com/retail-usum-egg-mmsc), and RNG-manipulated Pokémon are generally considered the "clean" alternative to generated ones, because they are indistinguishable (or rather, the same) from regular Pokémon generated by the game.
