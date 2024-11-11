+++
title = "Metaphor ReFantazio: Sword Surfer Achievement"
date = 2024-11-11

[extra]
math = true

[taxonomies]
tags = ["metaphor", "datamine"]
+++

In this post I uncover how the sword surfing mechanic calculates distance, and how to unlock the "Sword Surfer" achievement.

## The mechanics

* Distance is checked every frame, and it is calculated as the 3D Euclidean distance to the last frame position. ($\sqrt{\Delta x^2 + \Delta y^2 + \Delta z^2}$)
   * Going up/down the vertical axis does not seem to affect the speed at which you travel horizontally (or the difference is negligible). I was able to consistently get more distance by going up/down a slanted surface.
   * The distance you get per frame understandably depends on frame rate, meaning a lower frame rate gives you a higher distance every frame. Because the rate of updates is lower, however, this does not make it any more efficient.
* The initial wind-up after you press the button does not count for distance calculation, it only counts by the time you get the "fire" effect, that is when you can press the button again.
* 5,000,000m (5,000km) of distance travelled are required to unlock the achievement.
   * With a constant rate of 27m/frame at 60 FPS (what I observed holding each stick towards each other on a flat surface), it would take approximately 3,000 seconds (51 minutes) to reach the goal starting from scratch.

## Technical discussion

I was close to wrapping up my first playthrough of the game, so I wanted to catch up on some miscellaneous achievements I had missed. One of them was the infamous "Sword Surfer", that requires you to "travel a significant distance by blade-riding."

I had tried getting some progress in the day before, close to 20 minutes of surfing, but it wasn't enough. I had a few options, keep going for however long it would take? Use an elastic band to automate the process? Dig into the game's code to figure out how it worked? You can probably guess what my choice was.

So, I loaded the game's massive (500MB) executable into Ghidra and let it load for a while. In the meantime, I started
looking for clues in the save file. I conducted this research on the Steam version of the game, on patch 1.09.

I saved my game, backed up the save file, then surfed around for a brief while. After saving again, I compared the two save files:

```diff
6c6
< 00000050: 0000 0000 f467 0200 3f02 0000 4900 0000  .....g..?...I...
---
> 00000050: 0000 0000 0968 0200 3f02 0000 4900 0000  .....h..?...I...
177c177
< 00000b00: 0000 0000 0000 0000 0100 0000 0000 0000  ................
---
> 00000b00: 0900 0000 0000 0000 0100 0000 0000 0000  ................
9549c9549
< 000254c0: 0000 0e00 0100 0400 0000 b52e 4800 0f00  ............H...
---
> 000254c0: 0000 0e00 0100 0400 0000 1631 4800 0f00  ...........1H...
82294,82296c82294,82296
< 00141750: 0104 4d61 d446 0001 0001 0c01 0400 806d  ..Ma.F.........m
< 00141760: 4500 0100 010d 0104 77dd 6647 0001 0001  E.......w.fG....
< 00141770: 0e01 04b4 413f c000 0100 010f 0000 0001  ....A?..........
---
> 00141750: 0104 bd73 d446 0001 0001 0c01 0400 806d  ...s.F.........m
> 00141760: 4500 0100 010d 0104 f840 6747 0001 0001  E........@gG....
> 00141770: 0e01 0454 ffaa bf00 0100 010f 0000 0001  ...T............
82389c82389
< 00141d40: 0001 0001 0101 0385 b744 0001 0001 0b01  .........D......
---
> 00141d40: 0001 0001 0101 033f f844 0001 0001 0b01  .......?.D......
176191c176191
< 002b03e0: 0000 00f0 0100 0000 15ff ffff ff         .............
---
> 002b03e0: 0000 00f0 0100 0000 c2ff ffff ff         .............
```

Not a big diff, so this was a good start. The value at `0x55` seemed to match the play time in seconds, so I ignored it. The value at `0x254ca` instantly caught my attention. Indeed, the value seemed to increase the longer I tried using blade surfing.

Unfortunately I made a basic mistake that cost me some time, I didn't check whether it'd also update without blade surfing,
which I later found out it did. It did not help that the value seemed legitimate enough (around 4.7 million). I tried setting this value to 4,999,999 and let it increase naturally (I was just guessing that the threshold was 5 million), but it did not unlock the achievement. However, after this attempt I was confident that the value for the achievement was the one at `0x141d47`.

I didn't want to try other values, so I tried a different approach, finding how the game would invoke Steam's "grant achievement" function.
I figured it'd be easier to start from the Steam side, rather than the game. Because the game is available on multiple platforms, each having
a different achievement API, it likely had an achievement abstraction at the game level that would delegate Steam API calls to a module that's only
present in the Steam version.

I managed to find the call to Steam's `ISteamUserStats::SetAchievement` function, now it was time to walk back the stack to find the function that calculated
and checked the distance. As I had expected, the achievement module is an interface/virtual class, so the easiest way for me to find the caller was to attach a debugger.

So I set up `winedbg`, changed the value to 4,999,999 (from 4,519,999, so I was actually pretty close!) in the save file, started surfing and sure enough...
```
Stopped on breakpoint 1 at 0x0000014103d990 metaphor+0x103d990
```
This confirmed the requirement was 5,000,000m. I made sure to grab a backtrace to find the functions in the executable:
```
Wine-dbg>bt
Backtrace:
=>0 0x0000014103d990 in metaphor (+0x103d990) (0x000000038afc19)
  1 0x0000014103dae6 in metaphor (+0x103dae6) (0x000000038afc19)
  2 0x0000014103ddd1 in metaphor (+0x103ddd1) (0x000000038afc19)
  3 0x000001407cea97 in metaphor (+0x7cea97) (0x000000038afc19)
  4 0x000001406d4c4f in metaphor (+0x6d4c4f) (0x000000038afc19)
  5 0x000001405e8f0a in metaphor (+0x5e8f0a) (0x000000038afc19)
  6 0x0000014096dc71 in metaphor (+0x96dc71) (0x000000038afdd9)
  7 0x0000014051b2da in metaphor (+0x51b2da) (0x000000038afdd9)
  8 0x000001410643fd in metaphor (+0x10643fd) (0x000000038afdd9)
  9 0x000001410ecf4e in metaphor (+0x10ecf4e) (0x000000038afdd9)
  10 0x000001410e1123 in metaphor (+0x10e1123) (0x000000038afef0)
  11 0x006fffffc98b54 in kernelbase (+0x88b54) (0x000000038afef0)
```
After giving control back to the game, sure enough the achievement unlocked!  
Here is a brief look at what the distance function looks like. Note that all names are my own.

```cpp
#include <math.h>

struct Pos {
    float x;
    float y;
    float z;
};

int check_sword_surf_distance(Pos& current_pos, Pos& last_frame_pos) {
    float dx = current_pos.x - last_frame_pos.x;
    float dy = current_pos.y - last_frame_pos.y;
    float dz = current_pos.z - last_frame_pos.z;

    float sq_dist = dx * dx + dy * dy + dz * dz;
    float sqrt = metaphor_sqrt(sq_dist);
 
    int dist = get_distance_from_save();
    // Rounded down if < 0.5, up otherwise. The generated code seems to also
    // handle the case for sqrt < 0, but that is omitted for clarity.
    dist += (int) floor(sqrt + 0.5);

    if (dist >= 5000000) {
        // ACHIEVEMENT_40 is the internal name for Sword Surfer
        award_achievement(40);
    }

    set_distance_in_save(min(dist, 10000000));
}

```

The game uses a function that approximates the square root using its reciprocal. The formula it uses
is slower but a bit more accurate than `1/sqrt(x) * x` for single-precision floating point numbers.

```c
#include <immintrin.h>

inline float metaphor_sqrt(float sq) {
    __m128 tmp = _mm_set_ss(sq);
    // Get the reciprocal of the square root using x86 intrinsics
    tmp = _mm_rsqrt_ss(square);
    // No-op with -O2 on both GCC and clang
    float r_sqrt = _mm_cvtss_f32(tmp);
    // With x = sq, we have sqrt(x) = x/2 * rsqrt(x) * (3 - x * rsqrt(x)^2)
    return sq * 0.5 * r_sqrt * (3.0 - sq * r_sqrt * r_sqrt);
}
```

In retrospect, I could have tried searching for the constant `5000000` in Ghidra if I suspected that to be the requirement.
Unlike ARM, x86 instructions can load integer immediates greater than 16 bits with a single instruction.

I also spent some time trying to figure out what the function was doing with `3.0` and `0.5` because I had missed the `r` in `vrsqrtps`, so I thought it was trying to do some post-processing, instead it was just approximating it using the reciprocal. At least I got to brush up on vectorized operations.