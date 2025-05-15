+++
title = "Xenoblade X DE: Online vulnerabilities fixed in v. 1.0.2"
date = 2025-05-15

[extra]
shallow_toc = true

[taxonomies]
tags = ["xenoblade", "vulnerability"]
+++

Xenoblade Chronicles X: Definitive Edition version 1.0.2 was released on Apr 23, 2025. It was the first
update released to the public since the release of the game (Mar 20), which started on version 1.0.1 for the majority
of players, with no changelog for the first patch release. Among the several bugfixes, the English changelog
for 1.0.2 reports that it "fixed confirmed vulnerability bugs." (Curiously, this line would roughly translate to
"fixed some issues to improve the gameplay experience." in the Japanese changelog)

Among them are three security issues I reported to Nintendo, with help from 
[Nenkai](https://github.com/Nenkai), as part of their bug bounty program on [HackerOne](https://hackerone.com/nintendo). 
On Apr 25, bounties were awarded to the three reports. Permission for disclosure was granted on May 15.

The reports are listed in this post in order of severity.

<blockquote>
    <ol>
        <li><a href="#1-formatting-tag-injection-and-profanity-filter-bypass">Formatting tag injection and profanity filter bypass</a></li>
        <li><a href="#2-the-back-slash-vulnerability">The "Back Slash" vulnerability</a></li>
        <li><a href="#3-remote-save-file-write-from-unrestricted-rpc-access">Remote save file write from unrestricted RPC access</a></li>
    </ol>
</blockquote>

---

# 1. Formatting tag injection and profanity filter bypass

**Reported:** Mar 23, 2025  
**CVSS v3:** 5.3 - AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:N  
**Original report:** <https://hackerone.com/reports/3052880>

When receiving the player's **profile name** and **avatar name** from other players **online**, 
the game could interpret text as formatting tags, allowing formatting changes and bypassing the profanity filter.

When displaying text in the UI, the game can use formatting tags to replace certain text or change the formatting. These tags follow a syntax close to XML: `[tag params]inner text[/tag]`, for example `[ST:col p1=red]Red text[/ST:col]`. Some tags can accept no parameters, or no closing tag. When displaying another player's **avatar name** or **Nintendo Switch profile name** (in the player list, press X to toggle between profile and avatar names), the game could extract tags from the name and parses them, allowing the player to control what is displayed to other users.

There are several formatting tags, the most interesting in my opinion are:
* `[ST:icon p1=x]` (**save edit**): shows the icon `x` (e.g. `a` for the A button icon). This can catch people's attention and could be used to bypass filters.
* `[ST:n]` (**no save modification**): supposed to be a newline, but most of the time the player name is displayed it just hides what comes after it. Can be used to create empty text, even when normally not allowed.
* `[ML:wide]`: stretches the next character. This could be used to bypass filters because it acts as a zero-width space, especially with `[ML:wide scale=0]`, but this usage requires modification of the save file (for length constraints, see below)
* `[ST:col p1=x]`  (**save edit**): changes the color for the text following it to `x` (e.g. `red`). This can catch people's attention, and could be used to create "official-looking" names (because they are different from regular ones)
* `[ST:playername]`  (**save edit**): replaced with the name of the player that is reading the text. This is particularly damaging combined to the profanity filter bypasses, as you could create "targeted insults" that would appear with the name of whoever is reading it (different every time).
* `[ST:gender p1=x p2=y]` (**save edit**): displays `x` if the avatar is male, `y` if the avatar is female. This had limited usage to bypass the profanity filter, but `[ML:wide]` was generally better because it is shorter.

All tags that are longer than 10 characters require save modification, as the prompt you get when
starting a new game is limited to 10 characters in an unmodified scenario. However, 
**simply modifying the save file is not enough to bypass the filter**, the words are still masked
(with asterisks) when they are displayed to other users online. This filter however fails if you 
intersperse the letters with something like `[ML:wide]`. For example, `ba[ML:wide scale=0]nned word`
is displayed the same as "banned word", but bypasses filters if the word "banned" would normally be blocked.

The maximum length for avatar names to be viewed in their entirety by other players is 32 bytes (UTF-8 encoded and nul-terminated),
in places like player lists and notifications. When doing live peer-to-peer missions, the limit is 64 bytes, allowing for more tags to be used.

Additionally, I tried the same on the original game on Wii U, and names with tags were interpreted literally there, not as tags. So this issue was introduced **in the Switch remaster**.

These are the solutions I suggested:

* Filter out problematic character groups like `[System:`, `[ST:`, `[ML:`, and `[KR:`, preferably on the server side
* Replace `[` characters with `\[`, but proper care must be taken when inserting characters on the client side (see the second vulnerability)
* Block `[` characters altogether on the server side, but this is likely unviable as there is no option to change character name, and there are probably players already using those characters in a legitimate way.

## Impact

This isn't really a security vulnerability, however under the right circumstances it could allow creating inappropriate content, affecting the user experience negatively.
The game lacks a report feature, which makes the effects more persistent.

Without a **modified save**, a malicious user could:
* Start a new game file with an "empty" name for their avatar, going against the minimum character limit of 1 the game enforces under normal operation. In 1.0.1, anyone could reproduce this very easily by inputting `[ST:n]` using the software keyboard. Additionally, using just `[ST:n ` (with a space but no closing bracket) would allow for hiding any text that follows the player's name when interpolated.
* Change their Nintendo Switch profile name to something like `[ST:n]` or `[ML:wide]` to show an empty name in the online player list (you need to press X to toggle between profile and in-game avatar name).

With a **modified save** (which doesn't necessarily require a modified console, as save files can be downloaded through console transfer or from Switch Online cloud backups), a malicious user could:
* Change their avatar name to include longer formatting tags, like `[ST:playername]`, `[ST:col]`, `[ST:icon]` (longer because it requires parameters, see above), etc, which catch people's attention.
* In particular, using a tag like `[ML:wide]` (preferably `[ML:wide scale=0]` but it may be too long) would allow someone to bypass Nintendo's **profanity filter** by splitting the banned words in parts that are far away from each other in text, but displayed adjacently. As I said above, editing the save to include the banned word is not enough to bypass the filter, as banned words are still masked every time they are displayed.

A malicious user exploiting this bug could get their name onto the leaderboards or another place with high exposure, catching people's attention with the formatting while having an **inappropriate name**. Every place that shows a player's name was affected by this bug.

I've also tried invalid tag formattings, parameters, etc. to get the game to crash, but it did not occur under my tests.

## The fix

Starting from version 1.0.2, the game now inserts a space after the colon in user-controlled strings
the first time it encounters the sequences `[ST:`, `[ML:`, `[KR:`, or `[System:`. For example, 
`[ML:wide]` is now converted to `[ML: wide]`, which is printed literally.

Additionally, if the string it receives is longer than 10 UTF-8 characters, it replaces anything
past those 10 with <code>&nbsp;/e..</code> (+ a NUL byte).

---

# 2. The "Back Slash" vulnerability

**Reported:** Mar 20, 2025  
**CVSS v3:** 8.2 - AV:N/AC:L/PR:N/UI:N/S:U/C:N/I:L/A:H  
**Original report:** <https://hackerone.com/reports/3048061>

The same function from the first vulnerability (which was reported after this one) performs what is
effectively a `strcpy` while attempting to escape '\\' and '%' (backslash and percent) characters 
by duplicating them. However, when duplicating them it fails to check the buffer length and ends 
up overflowing it. The vulnerable function is at offset `+0054b940` in the 1.0.1 executable, 
and the fixed version is at `+0054c010` in 1.0.2.

The function takes an input "fixed string" and an output "fixed string", the "fixed strings" are pointers to this structure:
```cpp
struct FixedString {
    char buf[64];
    uint32_t length;
};
```
where `buf` is a NUL-terminated UTF-8 string, and `length` is the number of character bytes, i.e. `strlen(buf)`.
Essentially a string with variable length but fixed capacity.

Here is a recreation of the function. The disassembly is much more verbose (with the escape_percent check moved out of the loop), this snippet should be easier to understand.

```cpp
bool strcpy_escape(FixedString &out, const FixedString &in, bool escape_percent) {
    out.buf[0] = '\0';
    out.length = 0;
    if (in.length == 0) {
        return true;
    }
    for (int i = 0; i < in.length; i++) {
        char c = in.buf[i];
        char tmp[2] = { c, '\0' };
        if ((c == '\\') || (c == '%' && escape_percent)) {
            // This is not fine! Now out is longer than in, so the
            // in.length limit is too high
            strcat(out.buf, tmp);
            out.length += strlen(tmp); // basically + 1
        }
        // This would normally be fine because both out and in's 
        // buffers have the same size (in.length < 64 by definition)
        strcat(out.buf, tmp);
        out.length += strlen(tmp); // basically + 1
    }
    return true;
}
```

The vulnerable function fails to check whether duplicating the special characters would overflow the buffer.
If `buf` contains 32 backslashes and 5 letter 'a's (a totally valid `FixedString`), for example,
before the first 'a' is written, `buf` would have already overflown into the string length.

As you can see, using backslashes to exploit the function is more consistent than with percent
characters, because not all call sites have the last parameter set to true. And regular characters
should not cause an overflow, in fact filling the buffer with e.g. 63 'a's, or any other character
other than '\\' or '%' would not result in a buffer overflow.

I decided to call this vulnerability "Back Slash" after Shulk's Art, which is also present in this game.

## Stack buffer exploit

Out of all the uses of the function, what I presented was an exploit that makes use of the 
**player's avatar name**. This is one of the few untrusted inputs the game receives (another one is Skell name). 
The function is used by several UI code paths to display text in menus, for example with some 
pre-processing through Nintendo's profanity filter (which was analyzed in the first vulnerability).
This includes the function that displays the player's avatar name.

Some callers of the function for this use case allocate the output buffer on the heap, 
while others allocate it on the stack. Both eventually lead to a crash, but in the latter case,
it could be used specifically to overwrite the return address, but this is easier said than done
(see below).

## Requirements

To reproduce this locally, an attacker needs a modified console to replace the **save files**, 
or a way to get them transferred to their console (Switch Online backups or the save transfer feature).
This is because when the game prompts the player for the avatar name in an unmodified scenario,
the name is limited to 10 characters. To use it successfully online, the attacker needs to
 **patch their game** so that it doesn't crash for them.

The save file and saving logic (at least for what is relevant in this case) is largely similar
to that from the original Xenoblade Chronicles X on the Wii U. In fact, in the save file for the
original, the avatar name is stored at the same location and in the same way (64-byte buffer).
However, trying to exploit the same vector does not result in a buffer overflow in the original
game, so this vulnerability was **introduced in the remaster**.

## Impact

The vulnerability isn't really interesting in itself because it requires modification of the save file.
However, I highly suspected that it would also have an effect when another player's avatar name is
displayed in the UI using the **multiplayer feature** (of course, after patching the attacker's game
so that it doesn't keep crashing theirs).

Here's how I went about testing it. First, I wanted to see whether other player names would make
their way to the vulnerable function. Without running a modified save yet, I did so by patching the
vulnerable function to always replace the first character with a period, that way if I saw a period
at the start of a player's name, I would know it could be exploited. This was the case for pretty
much any way a player's name is displayed.

However, there is another condition for this to be exploitable, that the name is not stripped of
the backslashes or trimmed to a size less than the required character count for overflow. For this,
I had to test it with Nenkai in a private group.

* It was **not exploitable** when viewing the group member list, or leaderboards (which is awesome,
as it would have been quite a damaging target considering anyone can view them)
* It was **exploitable** in player name tags when playing peer-to-peer in a group mission, see the
attached video. So for example, an attacker could join random missions and disconnect everyone playing.

The reason it's not exploitable in player lists is that it seems to trim it down to a 32-byte buffer
(I counted 31 backslashes in the victim's POV), so it can't result in an overflow because the
vulnerable function extends that to 64 bytes.

When the function is exploited, it leads to **denial of service** with a possible crash
(as shown in the proof of concept).

I have tried escalating from there to achieve code execution through return-oriented programming,
because even with ASLR (and barring other protections)
the 2 least significant bytes of the return address can still be controlled. 
However, the exploit requires writing at least 1 NUL byte to the return address, 
because in the `strcat` call, the second argument is nul-terminated by construction.
So the potential for remote code execution was limited.

## The fix

Part of the fix for the first vulnerability also applies here. If the string is longer than
10 UTF-8 characters (4-byte characters are not supported, so this also fits in a 32-bit buffer),
it stops being processed and the excess is replaced with <code>&nbsp;/e..</code> (+ a NUL byte).

## Exploit showcase

Here is a video (credits to Nenkai) showing that an unmodified client crashes connecting to a group with a player
whose name can trigger the exploit.

<iframe width="640" height="360" src="https://www.youtube.com/embed/bwv3E0mL3QQ"
title="[Xenoblade Chronicles X: Definitive Edition] Multiplayer Crash PoC" frameborder="0"
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Of course, the same would happen to everyone in the group if it was the attacker joining instead.

---

# 3. Remote save file write from unrestricted RPC access

**Reported:** Mar 31, 2025  
**CVSS v3:** 9.3 - AV:N/AC:L/PR:N/UI:N/S:C/C:N/I:H/A:L  
**Original report:** <https://hackerone.com/reports/3062122>

**Documentation note**: The Switch game doesn't ship with function names or debug symbols, but where applicable, I've reported the names of the most similar functions from the Wii U game, which does have function names in the executable.

The game seems to use a reimplementation of the NEX network code based on duplicated objects.
As part of its peer-to-peer protocol, the game allows peers in a mission to send **RPC commands** (remote procedure calls)
to each other.  I found a few issues with the implementation, I'll list the ones that were acknowledged by Nintendo:

* Many RPCs accept inputs that are later used to index arrays, but in many cases those bounds were not checked, allowing an attacker to write arbitrary memory remotely.
* There is an unrestricted RPC that allows editing another player's save file, I'll describe it below.

Flags are used to track progress, among other things, and are divided based on size. 
There are blocks of 1-bit, 2-bit, 4-bit, 8-bit, 16-bit, and 32-bit flags.

The game has RPCs that allow **writing flags to the in-memory save file**. 
There is no RPC for 32-bit flags, and the RPCs for setting 2, 4, 8, and 16-bit flags are unused, but still handled by the game when received. There is a single usage for the 1-bit flag RPC, which I explain below. Most importantly, the **input flag ID was not checked or restricted to a specific range**, which allowed an attacker to **set arbitrary flags** in other players's in-memory save files, which then get written to flash memory when the game autosaves, or the player manually saves.

The one place the 1-bit flag RPC is used is `Gimmick::CGimmickMapObjPop::onRequestCtrlAccepted` (`+0038d858` on 1.0.1), which gets a flag ID with `Gimmick::CGimmickMapObjPop::getGameFlagId`, and then uses `sendRpcAll(1, 7)` to synchronize it with the other peers. I suggested that instead of sending the flag ID, the gimmick ID should be sent instead, and then each peer calculates the flag ID on their own. After this change, all the flag RPCs could be safely removed.

Some solutions I suggested:

* Remove the unused RPCs for changing 2, 4, 8, 16-bit flags, and 
remove the RPC for changing 1-bit flags, which has only one usage that 
calculates the flag ID based on a gimmick ID, but this could be done by each peer as explained above.
* Restrict the range of flags that can be modified by the RPCs.

I also recommended going over all the RPCs (there aren't too many), ensuring that inputs that are later used to index arrays are properly bounds-checked.

## Impact

This vulnerability would allow an attacker that is in **the same peer-to-peer mission as the victim**
(again, just connecting online or to the same group is not sufficient) to
**write arbitrary flags in other players's save files**, potentially resetting their progress or
corrupting their save file. This would require no action from the victim, other than being in the same
mission as the attacker. Additionally, no indication would be given to the victim, who may then
inadvertently manually save.

Additionally, because it probably wasn't meant to be public-facing,
this RPC (among others) would also fail to validate that the flag ID is within bounds.
There are only 2048 16-bit flags, yet the function accepts any flag ID 0-FFFF,
allowing **arbitrary memory writes** up to 0x1F000 bytes past the area of 16-bit flags.
For higher granularity but more limited range, 8-bit flags can be used instead.
The layout of the game flags singleton is static, so the structures next to it in memory are
always the same. I was able to get the game in a state where I was still able to save,
but when trying to load the save the game would get stuck in the "Now Loading" screen, basically
**corrupting the save slot**.

The attacker would need a way to send modified packets, the easiest way being on a modified
console. In my tests, I used a mod to call the `sendRpcAll` functions manually.

The reason being in a live mission is needed is that the RPC command manager doesn't seem to parse
requests outside of a peer-to-peer mission, even if connected to an online group.

Some high-value flags include:

* The **scenario flag**, which controls general main story progress.
* The flag that controls Overdrive: unsetting this flag after Chapter 5 makes a core mechanic inaccessible with no way to restore it.
* Flags that unlock the main story quests.

## The fix

The unused RPCs for setting 2, 4, 8, and 16-bit flags were removed, and the 1-bit flag RPC had the
range of allowed flags limited to the ones the game actually uses for online missions.

Additionally, other RPCs had checks added to prevent out-of-bounds memory accesses.

## Exploit showcase

Again, credits to Nenkai for the videos.

* **Video 1** (victim POV):  Setting the scenario flag to 0. No visible effect to the victim right away,
they only notice when they leave the mission and see that most menus and features are locked.
By that point, the game might have already autosaved.

<iframe width="640" height="360" src="https://www.youtube.com/embed/h0WMCT2ASCg"
title="[Xenoblade Chronicles X: Definitive Edition] Remote save tampering (Locking features)"
frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

* **Video 2** (victim POV):  Setting the scenario flag to 1. This is interesting because it triggers
a callback that plays the intro cutscene even during online play mode, which allows a player to edit
their avatar name. Generally, an attacker with the intent to corrupt other players's save files is
better off using the 0 version as to not raise suspicion.

<iframe width="640" height="360" src="https://www.youtube.com/embed/j7WQMsFW74w"
title="[Xenoblade Chronicles X: Definitive Edition] Remote save tampering (Story progress reset)"
frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

# Closing thoughts

Thank you for reading this writeup. It was the first time I participated in a bug bounty, and
it was nice contributing to making the game better in a more direct way and one that would benefit
way more people compared to my mods and tools. The staff over at Nintendo were very friendly
and appreciative, for example I honestly didn't expect the formatting tags report to be acknowledged, and I
was also surprised by the triage speed, especially considering the game is not from an entirely
first-party studio. Again huge thanks to Nenkai forhelping me with the reports.

Also if you came here for the research but haven't played the Xenoblade games yet,
what are you waiting for?