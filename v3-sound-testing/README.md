# zil-v3-sound_testing

## The Lurking Horror R221

| TERP | REQUIRES BIT 4 IN FLAGS | REPEATS FROM SOUND FILE HEADER | REPEATS FROM BLORB LOOP CHUNK | PLAYS SYNCHRONOUS SOUNDS | PLAYS FINAL SOUND(S) |
|-|-|-|-|-|-|
| Infocom AMIGAZIP | NO | YES | N/A | YES [1] | YES |
| [frotzpl3](https://www.ifarchive.org/indexes/if-archive/infocom/interpreters/frotz/old/#frotzpl3.zip) (DOS) | YES | YES | N/A | YES [2] | YES |
| [Gargoyle (Windows)](https://ccxvii.net/gargoyle/) | NO | N/A | NOT THIS GAME [3][8] | YES [3] | YES |
| [Ozmoo (MEGA65)](https://github.com/johanberntsson/ozmoo) | NO | NO [3] | N/A | YES [3] | YES |
| [Frotz 2.51 (DOS)](https://www.ifarchive.org/indexes/if-archive/infocom/interpreters/frotz/old/#frotz251.zip) [6] | YES [4] | NO | N/A | YES [3] | NO |
| [Frotz 2.55 (Linux)](https://www.ifarchive.org/indexes/if-archive/infocom/interpreters/frotz/#frotz-2.55.tar.gz) | YES [4] | N/A | NOT THIS GAME [3][8] | YES [3] | YES [7] |
| [Windows Frotz 1.27](https://github.com/DavidKinder/Windows-Frotz/releases/tag/1.27) | YES |	N/A | NO [3] | YES [3] | NO |
| [Filfre 1.1.1 (Windows)](http://maher.filfre.net/filfre/index.html) | NO | NO [3] | N/A | NO | NO |
| Parchment - [online](https://iplayif.com/api/sitegen) HTML conversion (using OGG sounds) | NO | N/A | N/A | NO | YES [9] |
| fizmo-ncursesw 0.7.14-2 ([on Linux Mint](https://packages.ubuntu.com/source/jammy/fizmo-ncursesw)) | NO | N/A | NO [10] | NO | NO |

[1] Amiga models 1000, 500, and 2000 load sounds so slowly, synchronous sounds work, but it is an exploit.

[2] This build of Frotz is unique. It has a script that **(a)** makes any currently playing sound’s repeat value 1 (so it finishes after it plays the current play-through) and **(b)** runs a `while` loop until the current sound finishes, before starting the next sound. This is the best terp for V3 sounds that I’ve found.

[3] The terp has code to handle sounds as special cases if the game is identified as The Lurking Horror (using the release and serial numbers to identify the game)

[4] Frotz (not Windows Frotz, but Frotz) checks for either bit 4 *or* bit 7 when the story version is V3, presumably because i6 sets bit 7 for V3, but the specs seem to say it should set bit 4 (although the actual AMIGAZIP terp cares not about either bit; it just plays sounds regardless, like Gargoyle and Filfre).

[6] The last release of Frotz **for DOS** with working sound was 2.51.

[7] The ncurses build of Frotz 2.55 on Linux Mint

[8] I believe this terp *does* read from a LOOP chunk, but it has the stuff for Lurking Horror hard-coded as a special case.

[9] After converting to HTML using the Parchment site generator ([online version](https://iplayif.com/api/sitegen)) with OGG sound files packaged with the story file in a ZBLORB, most sounds play correctly. When there are synchronous calls to sound, we only hear the last sound. This is the case for the last sounds in the game, too. It also fails to play S-SPARKY and S-CRETIN, probably because of the three calls to sound in that one routine, the last stopping S-CRETIN.

[10] I had to toggle $SOUND off and on at least three times while using fizmo. Sounds that should not repeat play on infinite loops. This terp seems to loop ALL sounds, no matter what, even when I use a test game of my own.

---
Also, sfrotz 2.55 on Linux (any flavor, WSL or not)  → This terp does not stop the sound when you wake up at the PC. It continues looping.

---
Also also:

https://curiousdannii.github.io/infocom-frotz/lurkinghorror.html

The first sound does not stop looping after waking up at the PC. Using Firefox on Windows 11.

I'm not at all certain, but this *seems* to be running a WASM version of sfrotz, which looks and behaves much like this.

---
## On Amiga

On Amiga (the only official *Lurking Horror* releases with sound), each sound has three files (using sound 3 as an example):
- s3.dat
- s3.mid
- s3.nam

The DAT file is the actual sound data. It is RAW, 8-bit signed, mono, and the frequency is normally around 16,000.

Here's the header from s3.dat (the rest of the file is the actual sound data, omitted here):
```
Offset(h) 00 01 02 03 04 05 06 07 08 09

00000000  C2 64 01 3C 3C 00 00 00 C2 5C
```
- Bytes 0-1: C2 64 = length of the following data is 49764 bits
- Byte 2: 01 = play once
- Bytes 3: 3C = base_note is 60
- Bytes 4-5: 3C 00 = sample frequency is 15360
- Bytes 6-7: always 00 00 and always unused
- Bytes 8-9: C2 5C = actual length of sound data (which follows) is 49756 bits

---
The MID file determines the actual frequency played.

Here is the hex from s3.mid:
```
Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

00000000  00 09 90 4C 40 FF 00 04 90 4C 00                 ...L@ÿ...L.
```

Byte 3 is the "note" value. Here, we have 76.

To find what frequency this will use, there is a formula:
```
  pow (2, ("note" - "base note of dat file") / 12) * "sample frequency of dat file" / 4
```

In this case, our values are:
```
   pow (2, (76 - 60) / 12) * 15360 / 4
```

...which is:
```
(2 ^ (16/12)) * 3840
```

...which is:
```
(2 ^ 1.33333333333) * 3840
```

..which is:
```
2.51984209978 * 3840
```

So, the frequency to play here will end up being 9676, which slows the original down a bit.

---
The NAM file's header points to the DAT file and the MID file. The NAM file's name must be `s[id].nam` (in this case **s3.nam**), and the MID and DAT files can be named whatever you please.

Here is the hex from s3.nam:
```
Offset(h) 00 01 02 03 04 05 06 07

00000000  01 00 73 33 2E 64 61 74  ..s3.dat
00000008  00 00 73 33 2E 6D 69 64  ..s3.mid
00000010  00 00 00 00 00 00 00     .......
```

---
*The Lurking Horror* doesn't take advantage of this, really, but *Sherlock* uses one sound file with three NAM and MID files (just to pick one example from *Sherlock*), playing it at three different speeds depending on the situation. This is a great way to get a lot out of a little, as was the Infocom way. (Plus, one of *those* MID files is also used by another sound's NAM file. Very tricksy!)

---
My extensive notes: https://gist.github.com/KVonGit/68509da41b48070f3a5c8e3e6829852d

---
Documentation by Stefan Jokisch: https://unbox.ifarchive.org/1wwrgqi0vl/sox-beta/infocom.info

---
### The unofficial DOS sound format

There were no official releases with sounds for DOS, but there *is* a special sound file format for Frotz for DOS.

It is basically the same as described above, but:
- there are no NAM files
- there are no MID files
- the sound data must be RAW 8-bit **unsigned** mono, with the same general frequency range

Besides that, the header we add to the sound file is identical to the sound data file for Amiga.

The naming convention is also different. We use the first seven characters of the story file's name, appended by the ID (2 digit, with leading 0). So, the file named S3.DAT in *The Lurking Horror* for Amiga will be **LURKIN03.SND** to match the DOS **LURKING.DAT** file. (Also, all sounds belong in a sub-folder named "SOUND".)

The terp I would recommend is frotzpl3. It will use the SND file header to get the "repeats" value. 0 means infinite loop. 1 through 254 will play N amount of times total.

frotzpl3 will also handle synchronous sounds within the same routine. If a sound is called while a sound is playing, frotzpl3 will:
- change the current sound's repeat value to 1 (to make it end when it finishes playing through)
- run a `while` loop until the current sound finishes
- rinse and repeat

This allows each sound to play through exactly once, the text printed during play can be delayed when TELL is called after SOUND in the routine, and the last sound's repeats value will be used. It mimics the old Amiga 1000, 500, and 2000 behavior (and handles things a little better actually, in my opinion, by actually handling synchronous sounds, rather than squeezing them in because the slow machines could be exploited that way).

---
> [!IMPORTANT]
> If using Frotz 2.32 or later, it will incorrectly figure the repeats value from the non-existent high byte of the third sound_effect argument.
> Normally in V3, this volume argument can be 1-8 (low to high), or 255 (or -1) for default volume (which is the same as 8, really).
> BUT... Frotz 2.32 or later treats this value as a V5 value, and it sets the repeats value from the high byte which does not exist in V3, which is *de facto* 0, which means an infinite loop in v3!
> So, if you know your game will be played on Frotz 2.32 or later, use 264 instead of 8 for the max volume setting (256 + ACTUAL_VOLUME). When Frotz 2.32 or later finds 256 in the high byte, it will play the sound once.

---
### Theory concerning the lack of a Mac version with sound

There was probably no Mac version of *The Lurking Horror* with sound because it seems MACZIP cannot handle looping sounds. In the Mac version of Sherlock, the infinitely looping sounds do not play at all. The sounds that *do* play cause game-play to freeze until the sound finishes. For example, when Big Ben strikes the hour, you must wait however long it takes to finish (if near Big Ben).

I believe half of the sounds in *The Lurking Horror* loop; so, that might be why it never got a Mac version with sound.

---
### The blorb LOOP chunk

This is not used in *The Lurking Horror* as far as I can tell, as all the terps that can use sounds from a blorb also have code to handle *The Lurking Horror* sounds as a special case, but there *is* a way to (mostly) use the old V3 looping in modern terps that use the LOOP chunk data from a blorb. The only two I've found so far are Gargoyle and the Unix version of Frotz.

We create a file with the header data to tell the terp which sounds to infinitely loop. (We cannot set the actual 'repeats' value to anything other than "infinite loop" or "play once", though.)

Here is the loop chunk from *The Lurking Horror*:

```
Offset(h) 00 01 02 03 04 05 06 07

00000000  00 00 00 04 00 00 00 00  ........
00000008  00 00 00 0A 00 00 00 00  ........
00000010  00 00 00 0D 00 00 00 00  ........
00000018  00 00 00 0F 00 00 00 00  ........
00000020  00 00 00 10 00 00 00 00  ........
00000028  00 00 00 11 00 00 00 00  ........
00000030  00 00 00 12 00 00 00 00  ........
```

The sounds that loop are: 4, 10, 13, 15, 16, 17, and 18. (The value of the fourth byte of every 8-byte sequence is a sound ID which loops.)

---
Possibly noteworthy: I have modified Windows Frotz to use the LOOP chunk. While doing so, I also made it read the 8th byte from each sequence to get the number of repeats. This way, the standard LOOP chunks will always be 0, which is what the terps that read the LOOP chunk normally expect anyway. But, with a modified terp, it can also correctly use our repeats value. If this is not good practice, you'll have to excuse me, as I am unlearned in the ways of the code.

---
### Sound in Ozmoo on a MEGA65

https://github.com/johanberntsson/ozmoo/blob/master/documentation/manual/manual.md#sound

---
Basically, we need a WAV, mono, 8-bit unsigned PCM, and each file is named [id].WAV with two leading zeroes. So, Amiga's S3.DAT or DOS's LURKIN03.SND would be 003.WAV for Ozmoo.

We can also use AIFF ([id].AIFF), but WAV is recommended.

---
The only way to have our own looping sounds in our own v3 games would be to use the same release and serial numbers as *The Lurking Horror*, and choose our sound IDs accordingly.

> [!NOTE]
> Alternatively, we could modify Ozmoo (by changing the repeats table in **sound.asm**) before building the game to set the repeats however we like. (Thanks to fredrikr for this tip!)

---
### Filfre 1.1.1

This terp wants AIFF files named SND[id].AIFF, and it wants them in the same folder as the story file. So, S3.DAT would be SND3.AIFF.

I need to look into the other details, but I assume it would be a normal AIFF, with no special settings when exporting the sound.

---
### Transcript

Transcript with sound debugging messages (using frotzpl3 on DOS):

<details><summary> CLICK HERE TO VIEW TRANSCRIPT </summary>

```
Start of a transcript of THE LURKING HORROR.
THE LURKING HORROR
An Interactive Horror
Copyright (c) 1987 by Infocom, Inc. All rights reserved.
THE LURKING HORROR is a trademark of Infocom, Inc.
Release 0 / Serial number 251112

>restart
Your score is 0 of a possible 100, in 0 moves. Graded on the curve, you are in
the class of Freshman.
Do you wish to restart?
(Y is affirmative): >y
Restarting.
[<SOUND 0 3>]
[<SOUND 0 4>]
You've waited until the last minute again. This time it's the end of the term,
so all the TechNet terminals in the dorm are occupied. So, off you go to the old
Comp Center. Too bad it's the worst storm of the winter (Murphy's Law, right?),
and you practically froze to death slogging over here from the dorm. Not to
mention jumping at every shadow, what with all the recent disappearances. Time
to find a free machine, get to work, and write that twenty page paper.

THE LURKING HORROR
An Interactive Horror
Copyright (c) 1987 by Infocom, Inc. All rights reserved.
THE LURKING HORROR is a trademark of Infocom, Inc.
Release 0 / Serial number 251112

Terminal Room
This is a large room crammed with computer terminals, small computers, and
printers. An exit leads south. Banners, posters, and signs festoon the walls.
Most of the tables are covered with waste paper, old pizza boxes, and empty Coke
cans. There are usually a lot of people here, but tonight it's almost deserted.

A really whiz-bang pc is right inside the door.

Nearby is one of those ugly molded plastic chairs.

Sitting at a terminal is a hacker whom you recognize.

>turn on pc
The computer powers up, goes through a remarkably fast self-check, and greets
you, requesting "LOGIN PLEASE:". The only sound you hear is a very low hum.

>login 872325412
The computer responds "PASSWORD PLEASE:"

>password uhlersoth
The computer responds "Good evening. You're here awfully late." It displays a
list of pending tasks, one of which is in blinking red letters, with large
arrows pointing to it. The task reads "Classics Paper," some particularly
ominous words next to it say "DUE TOMORROW!" and more reassuringly, a menu box
next to that reads "Edit Classics Paper."

>click edit
The menu box is replaced by the YAK text editor and menu boxes listing the
titles of your files. The one for your paper is highlighted in a rather urgent-
looking shade of red.

>click yak
You click the box for your paper, and the box grows reassuringly until it fills
most of the screen. Unfortunately, the text that fills it bears no resemblance
to your paper. The title is the same, but after that, there is something
different, very different.

>x paper
The paper appears to be a facsimile overlaid with occasional typescript. The
text is mostly in a sort of "Olde English" you've never seen before. What you
read is a combination of incomprehensible gibberish, latinate pseudowords,
debased Hebrew and Arabic scripts, and an occasional disquieting phrase in
English.

As you look at it more closely, you find it hard to focus on the screen, but
impossible to look away. Your finger strays toward the "MORE" box...

>click more
You touch the MORE box, and a new page appears.

The second page is much like the first, but around the edges, not when you look
at it straight, it's almost readable. There is something about a "summoning," or
a "visitor."

>g
You touch the MORE box, and a new page appears.

The third page is in the same script as the first, but laid out like a poem.
There are woodcut illustrations which are queasily disturbing.

There is a translation, or notes for one, typed between the lines of the poem:

"He returns, he is called back (?)
The loyal ones (acolytes?) make a sacrifice
Those who survive will meet him (be absorbed? eaten?)
They will live, yet die
Forever will be (is?) nothing to them (to him?)

"His place (lair? burrow?) must be prepared
His food (offerings?) must be prepared
Call him forth (invite him?) with great power
Only an acceptable (tasteful?) sacrifice will call him forth
He will be grateful (satiated?)"

The rest is even more fragmentary.

>g
You touch the MORE box, and a new page appears.

The fourth page is a photograph. You try to recoil from the screen, but cannot.
Fascinated and repelled at the same time, you wonder: is that a mouth, and what
is in it?

>g
You touch the MORE box, and a new page appears.

You faint, and when you awaken...


<SOUNDS ,S-DRONE ,S-START 2>

[Use $SOUND to toggle sound usage on and off.]

[SOUND 10 IS A LOOPING SOUND.]
[<SOUND 10 2 2>]
Place
This is a place. Things move about on a broken, rocky surface. Harsh sounds
split the air. Something sticky grabs at your feet. There is no color,
everything is drained of brightness, dull and lifeless. A path descends into a
shallow bowl of black basalt.

>d

<SOUNDS ,S-DRONE ,S-START 4>
[SOUND 10 IS A LOOPING SOUND.]
[<SOUND 10 2 4>]
Basalt Bowl
You are at the bottom of a deeply cut, smooth basalt bowl. Dimly seen shapes
crowd you on all sides. Ahead, in the focus of the movement, is a rock platform.

>d

<SOUNDS ,S-DRONE ,S-START 6>
[SOUND 10 IS A LOOPING SOUND.]
[<SOUND 10 2 6>]
At Platform
You stand before a low rock platform, more like an afterthought of piled rocks
or a glacial moraine than a work of artifice. You are pushed against the pile by
the crowd around you.

One small stone stands out in the pile, smooth, shiny, and glowing with a
blazing light.

>get stone
Taken.


<SOUNDS ,S-DRONE ,S-START 8>
[SOUND 10 IS A LOOPING SOUND.]
[<SOUND 10 2 8>]
Suddenly, the dimness becomes darkness, and the crowd around you explodes with
excitement. You are jostled and shoved from all sides. A low keening begins,
building into a deafening, almost mechanical chant. The darkness before you
compacts and deepens.

>x it
It's a smooth, shiny piece of what might be obsidian. Scratched on it is a
symbol.

The darkness before you, now visible, is a creature. It towers over the now-
silent crowd. The thing jerks this way and that, spraying a foul ichor. Its
palps twitch expectantly, then pound impatiently against the rock. You can feel
the smooth stone vibrating in your hand.

>throw it at creature
You can't. You go through the motions, but the stone doesn't leave your hand.

The thing now turns, sensing the presence of the stone. It quests almost blindly
for it, then those surrounding you thrust you forward. The thing stoops, its
mandibles grasping you. You are lifted towards its gaping maw. The stench and
the sounds issuing from it are overwhelming, and you fall unconscious.


<SOUNDS ,S-DRONE ,S-STOP>
[SOUND 10 IS A LOOPING SOUND.]
[<SOUND 10 3>]
You are awakened by the thump of your head hitting the terminal in front of you.
Falling asleep over term papers! It must have been a nightmare. Embarrassed, you
glance around. Yes, the hacker is looking in your direction. He must have heard
the thump.

Terminal Room, on the chair

A really whiz-bang pc is right inside the door.

Sitting at a terminal is a hacker whom you recognize.

>ask hacker about keys
"I've accumulated a few keys over the years. I'm a licensed locksmith, which
helps. I can get into any room at Tech." He pulls the keyring out on its chain,
and shows off a key you hadn't noticed before. "This is a master key," he says.

The hacker wanders over, trying to look nonchalant as he takes over your chair.
"Losing, huh?" he asks wittily. He glances at your terminal, which displays a
pattern of snow and unusual characters. He appears somewhat excited.

>ask him about master
"That's one of my best keys. It's a Tech master key. Not that it really opens
every door at Tech, but I'd say three out of five, at least. Naturally, some
labs are off-limits even to this key."

The hacker, mumbling under his breath, begins a flurry of activity. First the
screen returns to something nearly normal, then windows begin popping up like
toadstools after a rain. The screen looks a lot like the top of his terminal
table (or the bottom of a trash can).

>ask him for master
"Fat chance! This is a master key! What have you done for me lately?"

The hacker types furiously, and the screen displays what to you looks like an
explosion in a teletype factory. After a while he says. "Chomping file system.
Your directory has gone seriously west. I fixed it." He checks the screen. "It
was mixed up on the file server with some files from the Department of Alchemy."
He grunts. "People's names for their nodes are getting weird. This one is called
'Lovecraft.'" He pauses. "Your paper is gone, though. Sorry. Maybe they could
help you down there."

>s
Second Floor
This is the second floor of the Computer Center. An elevator and call buttons
are on the south side of the hallway. A large, noisy room is to the north.
Stairs also lead up and down, for the energetic. To the west a corridor leads
into a smaller room.

>w
Kitchen
This is a filthy kitchen. The exit is to the east. On the wall near a counter
are a refrigerator and a microwave.

Sitting on the kitchen counter is a package of Funny Bones.

>open fridge
Opening the refrigerator reveals a cardboard carton and a two liter bottle of
Classic Coke.

>get coke and carton
two liter bottle of Classic Coke: Taken.
cardboard carton: Taken.

>open oven
The microwave oven is now open.

>open carton
Opening the cardboard carton reveals Chinese food.

>put it in oven
Done.

>close oven
Okay, the microwave oven is now closed.

>press 4 then press 0 then press 5
The timer display now reads 0:04.

The timer display now reads 0:40.

The timer display now reads 4:05.

>press start
The microwave starts up. The timer begins counting down.

The timer display now reads 3:05.

>z
Time passes...

The timer display now reads 2:05.

>z
Time passes...

The timer display now reads 1:05.

>z
Time passes...

The timer display now reads 0:05.

>z
Time passes...

The microwave stops. The timer display now reads 0:00.

>open oven
The microwave oven is now open.

>get carton
Taken.

>e
Second Floor

>n
Terminal Room

Nearby is one of those ugly molded plastic chairs. Sitting on the chair is a
hacker.

A really whiz-bang pc is right inside the door.

>give carton to hacker
"Ah! Serious food!" He plunges into the food with all the delicacy and table
manners of a shark at a feeding frenzy. Soon a satisfied expression appears on
his face. "Now, what was it you were wanting?" he asks.

>ask him for master
"Well, I suppose I could loan you the master key for a while. Just don't get
into trouble, okay? I'll find you later, when I'm done with all this, and get it
back." He hands you the key.

>s
Second Floor

>press down
The down-arrow begins to glow.

You hear the elevator begin moving.

>z
Time passes...

The down-arrow blinks off.

>z
Time passes...

The elevator doors slide open.

>in
Elevator
This is a battered, rather dirty elevator. The fake wood walls are scratched and
marred with graffiti. The elevator doors are open. To the right of the doors is
an area with floor buttons (B and 1 through 3), an open button, a close button,
a stop switch, and an alarm button. Below these is an access panel which is
closed.

>open panel
Opening the access panel reveals a flashlight.

The elevator doors slide closed.

>get light
Taken.

>press open
The doors spring open.

>out
Second Floor

The elevator doors are open.

The elevator doors slide closed.

>d
Computer Center
This is the lobby of the Computer Center. An elevator and call buttons are to
the south. Stairs also lead up and down, for the energetic. To the north is
Smith Street.

>d
Basement
Bare concrete walls line a wide corridor leading east and west. An elevator and
call button are to the south. Stairs also lead up, for the energetic. From floor
to ceiling run wire channels and steam pipes.

>e
Temporary Basement
During the Second World War, some temporary buildings were built to house war-
related research. Naturally, these buildings, though flimsy and ugly, are still
around. This is the basement of one of them. The basement extends west, a
stairway leads up, and a large passage is to the east.

There is a crowbar and a pair of electrician's gloves here.

>get gloves and crowbar
pair of electrician's gloves: Taken.
crowbar: Taken.

>wear gloves
You put on the gloves. They're a little big, but not really such a bad fit at
all.

>u
It is pitch black.

>light
(flashlight)
The flashlight clicks on.

Temporary Lab
This is a laboratory of some sort. It takes up most of the building on this
level, all the interior walls having been knocked down. (One reason these
temporary buildings are still here is their flexibility: no one cares if they
get more or less destroyed.) A stairway leads down, and a door leads north.

There is a metal flask here.

>get flask
Taken.

>d
Temporary Basement

>turn off light
The flashlight clicks off.

>w
Basement

>w
Aero Basement
This basement level room is made of smooth, damp-seeming concrete. Fluorescent
lights cast harsh shadows. To the west is a stairway, and to the east the
basement area continues.

There is a forklift here.

>w
Stairway
A dimly lit stairway leads up and down from here. A corridor continues east.

>u
Aero Lobby
This is the lobby of the Aeronautical Engineering Building. Stairs lead down and
a corridor heads south towards the main building.

>s
Infinite Corridor
The so-called infinite corridor runs from east to west in the main campus
building. This is the west end. Side corridors lead north and south, and a set
of doors leads west into the howling blizzard.

There is a plastic container here.

There is a largish machine being operated down the hall to the east.

>get container
Taken.

>e
Infinite Corridor
The so-called infinite corridor runs from east to west in the main campus
building. The corridor extends both ways from here. Many closed and locked
offices are to the north and south.

A maintenance man is here, riding a floor waxer.

>z
Time passes...

>z
Time passes...

>z
Time passes...

>z
Time passes...

The floor waxer waxes away to the east.

>e
Infinite Corridor
The so-called infinite corridor runs from east to west in the main campus
building. The corridor extends both ways from here. A stairway leads up, and a
door leads out to the Great Court.

A maintenance man is here, riding a floor waxer.

There is a wall socket on one wall, and a heavy-duty power cord is plugged into
it. The cord leads to a large floor waxer.

>u
Great Dome
Here a walkway circles the base of a huge ornate dome. Below is the Infinite
Corridor. From stories of Tech Exploring trips, you recall that there is
supposed to be a ladder here. On the other hand, there is a shiny rope-like
thing hanging near where the ladder used to be, and leading upward. Below you,
in the corridor, you can see a floor waxer, busily waxing the floor.

>climb rope
The wet stuff on the strand sticks to the gloves, but doesn't otherwise affect
you. You have a little trouble climbing up to the catwalk, but grab the rail
just before your strength gives out. You heave yourself up onto the catwalk.

You stand up on the catwalk, catching your breath for a moment. Your eyes stray
along the strand you climbed. It trails along the catwalk, where it joins
something large and squishy squatting at the far side. A single, bright-blue eye
opens in the squishy mass, and the tentacle (for that's what it is) retracts.
The mass almost flows through the spaces in the catwalk railing and drops to the
floor fifteen feet below. Before you can react, it's gone.

Top of Dome
Inside the great dome, near the top, a metal catwalk is precariously perched.
There is no way further up, but a small metal door is set in the side of the
dome.

Where the pulpy mass was squatting, a wooden ladder lies on the catwalk.

>lower ladder
You lower the ladder to the walkway below. It's just the right length to climb
down.

>open door
You open the door, and freezing air, blowing snow, and howling wind enter and
whip around you.

>out
You enter the freezing, biting cold of the blizzard.

Roof of Great Dome
You are perched precariously on the roof of the Great Dome. A set of narrow
indentations in the dome provides a dangerous route to the very tip-top of the
dome.

>u
You scramble up icy surface of the dome, almost slipping a few times, but
finally you make it to the top.

On the Great Dome
This is the very top of the Great Dome, a favorite place for Tech fraternities
to install cows, Volkswagen Beetles, giant birthday candles, and other bizarre
objects. The top is flat, round, and about five feet in diameter. It's very
windy, which has kept the snow from accumulating here. The only way off is down.

In the exact center of the flat area is a bronze plug.

Bitter, bone-cracking cold assaults you continuously. The temperature and the
blizzard conditions are both horrible.

>remove plug
You pry the plug out of its socket, revealing a cylindrical hole about the same
diameter but somewhat deeper. There is a piece of paper in the hole.

>drop it
Dropped.

>get paper
Taken.

>d
Roof of Great Dome

You can feel the cold worming its way through your layers of clothing and biting
into your flesh.

>in
You push your way into the welcoming warmth inside.

Top of Dome

A wooden ladder leads down from here.

>d
You scramble down the ladder.

Great Dome

A wooden ladder leads up from here.

>d
Infinite Corridor

There is a wall socket on one wall, and a heavy-duty power cord is plugged into
it. The cord leads away to the east.

There is a largish machine being operated down the hall to the east.

>e
Infinite Corridor
The so-called infinite corridor runs from east to west in the main campus
building. The corridor extends both ways from here. Many closed and locked
offices are to the north and south.

There is a glass-fronted emergency cabinet here.

There is a largish machine being operated down the hall to the east.

The floor waxer, approaching from the east, is now here.

>z
Time passes...

>z
Time passes...

>z
Time passes...

The floor waxer continues waxing a section of floor nearby. The maintenance man
operating it stares at you suspiciously.

>z
Time passes...

>z
Time passes...

>z
Time passes...

The floor waxer waxes away to the east.

>break glass
Wearing the heavy gloves, you confidently smash the glass with a blow of your
hands!

>get axe
Taken.

>w
Infinite Corridor

There is a wall socket on one wall, and a heavy-duty power cord is plugged into
it. The cord leads away to the east.

There is a largish machine being operated down the hall to the east.

>hit cord with axe
The axe crashes against the floor, and the power cord severs! The whine of the
floor waxer slows, and the maintenance man jerks to alertness.

>open container
You pull off the seal and open the container, revealing a smelly, viscous
liquid.

The maintenance man, growling foul-sounding imprecations, descends from the
floor waxer and heads towards you.

>empty it
It pours out and spreads like ants at a picnic. The floor is now covered from
wall to wall with slippery floor wax.

The maintenance man lurches toward you with surprising speed. He is nearly upon
you.

>drop container and assignment
plastic container: Dropped.
assignment: Dropped.

The maintenance man lurches toward you with surprising speed. Just as he is
about to grab you he slips on the wax. His hand whips by, inches from your
throat, and he drops to the floor, screaming in frustration.

>e
You slip and slide on the wax. It's like walking on wet ice. You can barely keep
upright, but it's worse for the maintenance man. His gait is so jerky that each
time he takes a step he falls! His persistence is impressive, but you slip
(literally) by before he can grab you.

Infinite Corridor

There is a formerly glass-fronted emergency cabinet here.

There is a largish machine down the hall to the east.

The maintenance man continues slipping, falling, standing, and so on. He reminds
you of a badly made windup toy.

>e
Infinite Corridor
The so-called infinite corridor runs from east to west in the main campus
building. This is the east end. The corridor branches north and south here.

A disabled floor waxer looms nearby.

The maintenance man continues slipping, falling, standing, and so on. He reminds
you of a badly made windup toy.

>n
Fruits and Nuts
This is the central corridor of the Nutrition Building. The main building is
south, and a stairway leads down.

>d
Cluttered Passage
This cluttered passage leads southeast. It is full of apparently discarded
electronic equipment, old rusty file cabinets, and other detritus. A stairway
also leads up.

>se
Brown Basement
This is a cluttered basement below the Brown Building. Discarded equipment
nearly blocks an already narrow hallway that terminates in a stairway leading
up. The passage itself continues northwest.

There is a pair of rubber boots here.

>get boots then wear them
Taken.

Snug, but okay.

>u
Brown Building
This is the lobby of the Brown Building, an eighteen-story skyscraper which
houses the Meteorology Department and other outposts of the Earth Sciences. The
elevator is out of order, but a long stairway leads up to the roof, and another
leads down to the basement. A revolving door leads out into the night.

>u
Top Floor
This is the top of the stairway. A door leads out to the roof here, and you can
hear the wind blowing beyond. There is a sign on the door.

>unlock door with key
The door is now unlocked.

>open door
You push the door open, revealing a windswept, snow-covered roof. Frigid wind
whips snow into your face.

>out
You enter the freezing, biting cold of the blizzard.

Skyscraper Roof
A low parapet surrounds a small roof here. The air conditioning cooling tower
and the small protrusion containing the stairs are dwarfed by a semitransparent
dome which towers above you. The blowing snow obscures all detail of the city
across the river to the south.

>u
Inside Dome
You are inside a large domed area. The dome contains equipment that makes it
clear it is a weather observation station. For some reason, it also contains a
small peach tree. Wind whistles outside, and snow blasts against the
semitransparent material of the dome.

Something smashes against the glass of the dome! You turn and see a dark shape
clinging to the outside of the structure.

>d
You enter the freezing, biting cold of the blizzard.

Skyscraper Roof

A dark shape watches balefully from nearby.

The dark shape moves. Above the howl of the wind you hear a high-pitched keening
noise.

>throw stone at creature
The stone hits the dark beast, and appears to go completely through it as though
the creature was made of air. 
<SOUNDS ,S-DIE ,S-INIT>
[<SOUND 12 1>]
The smooth stone disappears over the south edge of the building
<SOUNDS ,S-DIE>
[<SOUND 12 2 8>]
, and the creature follows it, screaming frustration into the storm.

Bitter, bone-cracking cold assaults you continuously. The temperature and the
blizzard conditions are both horrible.

>u
You push your way into the welcoming warmth inside.

Inside Dome

>dig in tub
You root around in the dirt for a while, when you encounter something hard.
Further exploration reveals it to be a dried, chewed looking human hand.

>get hand
Taken.

>d
You enter the freezing, biting cold of the blizzard.

Skyscraper Roof

>in
Top Floor

>d
Brown Building

>s
You enter the freezing, biting cold of the blizzard.

Small Courtyard
This courtyard is a triumph of modern architecture. It is spare, cold, angular,
overwhelming in size, and bears a striking resemblance to a wind tunnel whenever
the breeze picks up. Right now this is true of the whole campus, though. A huge
mass lurks nearby, and an almost featureless skyscraper is to the north.

There is a smooth stone here.

>get stone
Taken.

Bitter, bone-cracking cold assaults you continuously. The temperature and the
blizzard conditions are both horrible.

>n
You push your way into the welcoming warmth inside.

Brown Building

>d
Brown Basement

>nw
Cluttered Passage

>u
Fruits and Nuts

>s
Infinite Corridor

A disabled floor waxer looms nearby.

>w
Infinite Corridor

There is a formerly glass-fronted emergency cabinet here.

There is a largish machine down the hall to the east.

>w
Infinite Corridor

The floor here is covered with slippery, messy floor wax.

There is a wall socket on one wall, and a heavy-duty power cord is plugged into
it. The cord terminates in a severed stump.

There is an assignment and a plastic container here.

There is a largish machine down the hall to the east.

>w
You slip and slide on the wax. It's like walking on wet ice. You can barely keep
upright, but you manage to lose your balance in just the right way to keep
going.

Infinite Corridor

There is a largish machine down the hall to the east.

>w
Infinite Corridor

Slouching nearby is an urchin. He's a youngish teenager wearing a ski hat,
running shoes, and a bulky, suspiciously bumpy, threadbare parka. He's jumpy,
and looks suspiciously at you.

There is a largish machine down the hall to the east.

>n
Aero Lobby

>d
Stairway

>e
Aero Basement

There is a forklift here.

>enter lift
You are now in the forklift.

>start it
The forklift sputters to life.

>e
Basement, on the forklift

>e
Temporary Basement, on the forklift

>e
It is pitch black.

>light
(flashlight)
The flashlight clicks on.

Dead Storage, on the forklift
This is a storage room. It contains an incredible assemblage of discarded junk.
Some of it is so old and mouldering that you can't be sure where one bit of junk
stops and the next begins. It's piled to the ceiling on ancient, rotting
pallets; you can't even see the east wall.

>remove junk with lift then g then g then g
You have a little trouble using the forklift, but it's not really all that hard.
You start clearing junk, moving it around and trying to create a passage.

You continue moving junk, becoming more proficient with the forklift.

You continue moving junk, becoming more proficient with the forklift.

You've built a fairly narrow (about one forklift wide) path through the junk.
You can see an opening into a further storage room beyond this one.

>e
Ancient Storage, on the forklift
What's deader than dead storage? That's what's in this room. Most of the
contents have collapsed or rusted back to the primordial ooze. There is mold
growing on some of the unidentifiable piles. Stagnant puddles of water pollute
the floor. You can now believe how old some of these foundations are said to be.

There is a closed, disused-looking manhole here.

>exit lift
You are now on your feet.

>open manhole with crowbar
You lever the manhole cover aside, and crusted dirt falls into a dark, partly
obstructed hole below.

>d
You push your way through cobwebs, damp fungus, and other obstructions.

Brick Tunnel
This is an ancient tunnel constructed of roughly mortared bricks and stones. A
slippery and almost invisible set of handholds leads up. The tunnel continues a
long way north and south from here.

>n
You make your way along the long tunnel.

Renovated Cave
You are in a huge, cave-like construction. A path leads down to a floor partly
covered with rough concrete. The walls and ceiling are high and reinforced with
beams of wood, iron, and steel. In the center of the floor you can see a large,
flat slab of granite. The only exit is behind you to the south.

>d
Before the Altar
You are at the bottom of the cave. The huge slab of granite in the center is a
sort of altar. It is carved with strange and disturbing symbols, the largest of
which looks very familiar. Some of the symbols are obscured by rusty red stains.
Nearby is an iron plate set in the concrete of the floor.

Lying to one side of the altar stone is a sharp, thin-bladed knife.

>get knife
Taken.

>u
Renovated Cave

>s
Brick Tunnel

>u
Ancient Storage

In one corner of the room a manhole cover is partly buried in the dirt and crud.

There is an open manhole here.

There is a forklift here.

>w
Dead Storage

>w
Temporary Basement

>turn off light
The flashlight clicks off.

>w
Basement

>w
Aero Basement

>w
Stairway

>u
Aero Lobby

>s
Infinite Corridor

There is a largish machine down the hall to the east.

>e
Infinite Corridor

There is a largish machine down the hall to the east.

>e
Infinite Corridor

The floor here is covered with slippery, messy floor wax.

There is a wall socket on one wall, and a heavy-duty power cord is plugged into
it. The cord terminates in a severed stump.

There is an assignment and a plastic container here.

There is a largish machine down the hall to the east.

>e
You slip and slide on the wax. It's like walking on wet ice. You can barely keep
upright, but you manage to lose your balance in just the right way to keep
going.

Infinite Corridor

There is a formerly glass-fronted emergency cabinet here.

There is a largish machine down the hall to the east.

>e
Infinite Corridor

A disabled floor waxer looms nearby.

>e
You can't go that way.

>s
Chemistry Building
This corridor is lined with closed, dark offices. At the south end of the
corridor is a door with a light shining behind it. There is something written on
the door.

>knock on door
You knock on the door. The hollow sound reverberates down the hall. You sort of
wish you had knocked more softly.

>z
Time passes...

The door opens partway, revealing a professorial man in a white lab coat. He
smiles. "Good evening! I don't get many visitors this late. You're not one of my
students, are you?" He ushers you into the room without waiting for an answer,
closing the door behind you.

Department of Alchemy
This office is clinically clean, shiny, and modern. It looks like something out
of a science fiction movie. A closed door to the north leads back into the
corridor and an archway opens to the south.

Taped to the wall to the right of the archway is a sign-up sheet.

The professor is here.

>give paper to professor
He reads it carefully. "What drivel! This just confirms my suspicions. He had
clearly gone over the edge. Drug use, drinking, insanity. It's only too bad that
I didn't realize what was happening. I might have helped him."

The professor gazes at you with a distinctly predatory air.

>s
"Ah! You'd like to see the lab?" the professor asks in a rather unctuous tone.
"Come right in!" He ushers you through the archway into the lab, following
quickly behind you and turning on the lights.

Lab
The lab is an ultramodern, fully equipped chemistry lab. Unfortunately, or
perhaps fortunately, you aren't a chemistry major, so the equipment might as
well be magical.

The professor is here.

There is a lab bench and an Alchemy Department computer here. Sitting on the lab
bench is a vat. The vat contains a tarry liquid.

The professor guides you to the center of the lab, where a strange pentagonal
symbol is chalked on the floor. He cuts one of the chalk lines with a small
knife you had not previously noticed, pushes you into the center of the chalked
symbol, and redraws the line, muttering softly and rhythmically as he does so.
"There, that's done. Don't move from there, it'll only make things worse for
you." He makes some odd gestures at the archway and then goes over to the lab
bench.

>z
Time passes...

The professor is preparing something at the lab bench. "Alchemy is my chosen
field, and I've gotten ridiculed for it. It's like chemistry, except that
chemists don't recognize that some natural laws are enforced by persons, not
physics. Some of them will grant power, or knowledge, but they must be placated,
or even bribed. They're not of this earth, not demons or devils, and they aren't
always friendly. To me it's just an unpleasant necessity on the path to power.
When I'm done, they won't laugh anymore!"

>z
Time passes...

The professor enters another pentagram, and begins a highly choreographed
ritual. "This may seem a little silly to you, but the symbology is what's
important. Certain alignments, certain aspects. In a few moments, it won't
matter anyway," he remarks. "There is very little room for error here, so be
calm." He chants, he brandishes strange instruments, moves about inside the
pentagram, and occasionally points to you. It becomes clear exactly what he
meant by the word "bribe."

>cut line with knife
You cut the outer lines of the pentagram. It no longer completely encloses you.
The professor sees what you've done out of the corner of his eye. He stares,
horrified. "Stop, don't move!" he says between verses of the chant. The chant
takes on a pleading tone.

The chant grows more complex, the professor having difficulty with the almost
unpronounceable words, with rhythms and cadences that make you want to stop your
ears. The room appears to be getting darker.

>exit
You push your way through a soft spot just over the scuff marks, and are outside
the pentagram. The air is thick and close.

<SOUNDS ,S-PSYCHO ,S-START 2
[SOUND 13 IS A LOOPING SOUND.]
[<SOUND 13 2 2>]

A thick black mist begins to form in the room. Parts are darker, and parts
lighter, and the dark parts form a disturbing shape. The professor chants and
calls more loudly now, clearly terrified of what may happen, and you realize the
calls are being answered.

>move bench
It's heavy, but it moves, revealing a hinged metal trapdoor beneath.

<SOUNDS ,S-PSYCHO ,S-START 4>
[SOUND 13 IS A LOOPING SOUND.]
[<SOUND 13 2 4>]

The room is now freezing cold, though the windows are shuttered and tightly
curtained. Low, bone-rattling vibrations shake the room in cadence with the
chant. The black mist is growing thicker. The professor is alternately looking
at you and at the mist.

>open trapdoor
It swings open easily.

<SOUNDS ,S-PSYCHO ,S-START 6>
[SOUND 13 IS A LOOPING SOUND.]
[<SOUND 13 2 6>]

The black mist swirls wildly around the room, and a deep bass voice gibbers out
of thin air. "No!" screams the professor, and jumps toward you out of his own
pentagram. He realizes what he has done, and tries to reenter, but the mist
grabs at him.

>d
Cinderblock Tunnel
This is a tunnel whose walls are cinderblock, with a concrete floor and ceiling.
A metal ladder leads up to an open metal plate in the ceiling, and the tunnel
continues north, where the cinderblock walls become brick.

<SOUNDS ,S-PSYCHO ,S-START 8>
[SOUND 13 IS A LOOPING SOUND.]
[<SOUND 13 2 8>]

From above, you hear a thunderous noise, a maniacal scream, and then the sound
of equipment smashing. The trapdoor slams shut, but around it pours a blinding
flash of light. Finally you hear an almost inaudible whimper, then nothing. The
light fades, leaving you in the dark.

<SOUNDS ,S-PSYCHO ,S-STOP>
[SOUND 13 IS A LOOPING SOUND.]
[<SOUND 13 3>]

>open trapdoor
It pushes open easily.

>light
(flashlight)
The flashlight clicks on.

Cinderblock Tunnel
This is a tunnel whose walls are cinderblock, with a concrete floor and ceiling.
A metal ladder leads up to an open metal plate in the ceiling, and the tunnel
continues north, where the cinderblock walls become brick.

>u
Lab
The lab is a shambles. It looks like something red and sticky has been spread
over the walls, ceiling, and floor. Much of the equipment, particularly that
near the center of the room, has been destroyed. There is an open metal plate in
the floor.

There is a brass hyrax, a piece of paper, a lab bench and an Alchemy Department
computer here. Sitting on the lab bench is a vat. The vat contains a tarry
liquid.

>put hand in vat
When you dip the mummified hand in the liquid, the elixir begins to bubble
furiously. You can't really see the hand, except when a finger pokes up every so
often.

The hand bobs to the surface. It's odd, but it looked like one of the fingers
moved.

>z
Time passes...

The hand splashes to the surface. The fingers are moving!

>z
Time passes...

The hand is trying to crawl out of the vat.

>get hand and ring
human hand: You grab the wiggling hand and draw it forth, newly animated, from
the vat. As it emerges, the elixir flows off. The hand scuttles up your arm and
perches quietly on your shoulder.
brass hyrax: Taken.

>put ring on hand
The ring fits the hand perfectly.

>drink coke
Delicious! Contains caffeine, one of the four basic food groups. Too bad they
make it with fructose these days, instead of sucrose. You feel much more alert
and awake now.

>n
Department of Alchemy

Taped to the wall to the right of the archway is a sign-up sheet.

>open door
Okay, the Alchemy Department door is now open.

>n
Chemistry Building

>turn off light
The flashlight clicks off.

>n
Infinite Corridor

A disabled floor waxer looms nearby.

>w
Infinite Corridor

There is a formerly glass-fronted emergency cabinet here.

There is a largish machine down the hall to the east.

>w
Infinite Corridor

The floor here is covered with slippery, messy floor wax.

There is a wall socket on one wall, and a heavy-duty power cord is plugged into
it. The cord terminates in a severed stump.

There is an assignment and a plastic container here.

There is a largish machine down the hall to the east.

>w
You slip and slide on the wax. It's like walking on wet ice. You can barely keep
upright, but you manage to lose your balance in just the right way to keep
going.

Infinite Corridor

There is a largish machine down the hall to the east.

>w
Infinite Corridor

There is a largish machine down the hall to the east.

>n
Aero Lobby

>d
Stairway

>e
Aero Basement

>e
Basement

>e
Temporary Basement

>e
It is pitch black.

>light
(flashlight)
The flashlight clicks on.

Dead Storage
This is a storage room. It contains an incredible assemblage of discarded junk.
Some of it is so old and mouldering that you can't be sure where one bit of junk
stops and the next begins. It's piled to the ceiling on ancient, rotting
pallets. A narrow path winds eastward through the junk.

>e
Ancient Storage

In one corner of the room a manhole cover is partly buried in the dirt and crud.

There is an open manhole here.

There is a forklift here.

>w
Dead Storage

>w
Temporary Basement

>turn off light
The flashlight clicks off.

>w
Basement

>w
Aero Basement

>w
Stairway

>d
Subbasement
This is the subbasement of the Aeronautical Engineering Building. A stairway
leads up. A narrow crack in the northwest corner of the room opens into a larger
space.

>u
Stairway

>u
Aero Lobby

>s
Infinite Corridor

There is a largish machine down the hall to the east.

>w
You enter the freezing, biting cold of the blizzard.

Mass. Ave.
This is the main entrance to the campus buildings. Blinding snow obscures the
stately Grecian columns and rounded dome to the east. You can barely make out
the inscription on the pediment (which reads "George Vnderwood Edwards, Fovnder;
P. David Lebling, Architect; Russell Lieblich, Sovnd Engineer"). West across
Massachusetts Avenue are other buildings, but you can't see them.

>e
Infinite Corridor

There is a largish machine down the hall to the east.

>e
Infinite Corridor

Slouching nearby is an urchin.

There is a largish machine down the hall to the east.

>show hand to urchin
The urchin sees the hand twitching. "I heard what happens around here! I'm not
gettin' fed to a monster!" He lets loose a scream of fear and zooms away.
Something drops from beneath his parka as he departs. He slows for a moment,
decides not to retrieve it, and disappears into the distance.

>get cutter
Taken.

>w
Infinite Corridor

There is a largish machine down the hall to the east.

>w
You enter the freezing, biting cold of the blizzard.

Mass. Ave.

>e
Infinite Corridor

There is a largish machine down the hall to the east.

>n
Aero Lobby

>d
Stairway

>d
Subbasement

>drop cutter, flask, and axe
bolt cutter: Dropped.
metal flask: Dropped.
fire axe: Dropped.

>nw
Tomb
This is a tiny, narrow, ill-fitting room. It appears to have been a left over
space from the joining of two preexisting buildings. It is roughly coffin
shaped. The walls are covered by decades of overlaid graffiti, but there is one
which is painted in huge fluorescent letters that were apparently impossible for
later artists to completely deface. On the floor is a rusty access hatch locked
with a huge padlock.

>unlock lock with key
The lock, though rusty and unwilling, opens, releasing the hatch.

>get lock
Taken.

>open hatch

<SOUNDS ,S-HATCH>
[<SOUND 6 2 8>]
The hatch is heavy, and its hinges are rusty, but you pull and strain and it
opens with a scream of metal. Revealed below is a rusty ladder leading down.
Warm, fetid air coils up out of the hole. There is a burned out (no, smashed)
utility light set in the wall a few feet down.

>light
(flashlight)
The flashlight clicks on.

>d
Steam Tunnel
This dank and grimy tunnel is largely filled with an imperfectly insulated steam
pipe. The tunnel is uncomfortably hot and damp. You have gone from the arctic to
the tropics. The concrete tunnel has odd molds and fungi growing on its walls
and ceiling, and the floor is squishy. Torn clots of insulation litter the
floor. Along the ceiling runs a thick tangle of coaxial cable. The tunnel heads
east and west. A rusty metal ladder leads up.

<SOUNDS ,S-ATTACK ,S-START 2>
[SOUND 4 IS A LOOPING SOUND.]
[<SOUND 4 2 2>]

You can hear, in the distance, a chittering, scratching sound.

>e
Steam Tunnel
This dank and grimy tunnel is largely filled with an imperfectly insulated steam
pipe. The tunnel is uncomfortably hot and damp. A thick bundle of coaxial cable
runs east to west along the ceiling. There is a pressure release valve on the
steam pipe here.

<SOUNDS ,S-ATTACK ,S-START 3>
[SOUND 4 IS A LOOPING SOUND.]
[<SOUND 4 2 3>]

The sound is louder. It sounds like small animals. Is it rats?

>turn valve with crowbar
The valve, with a horrible scream of tortured metal, gives a little, and a small
trickle of steam issues forth.

<SOUNDS ,S-ATTACK ,S-START 4>
[SOUND 4 IS A LOOPING SOUND.]
[<SOUND 4 2 4>]

The sound continues. It's almost certainly rats.

>z
Time passes...

<SOUNDS ,S-ATTACK ,S-START 6
[SOUND 4 IS A LOOPING SOUND.]
[<SOUND 4 2 6>]

The rat sounds are growing louder, but you still can't see any rats.

>z
Time passes...

<SOUNDS ,S-ATTACK ,S-START 6
[SOUND 4 IS A LOOPING SOUND.]
[<SOUND 4 2 6>]

The rat sounds are growing louder, but you still can't see any rats.

>z
Time passes...

A troop of rats appears out of the darkness. The rats are momentarily startled
by your presence, but soon the bolder ones begin to approach. There are more
rats here than you have ever seen.

>open valve
The valve screeches open. A jet spray of live steam issues from it, filling the
tunnel in front of you.
<SOUNDS ,S-ATTACK ,S-STOP>
[SOUND 4 IS A LOOPING SOUND.]
[<SOUND 4 3>]
 The rats are caught in the full force of the blast. Horrible squeals can be
heard from the midst of the steam cloud, and scalded rats charge past you, all
interest in anything but flight forgotten. One of their number remains, dead.

>close it
The valve closes, more easily than it opened.

>e
Steam Tunnel
The steam tunnel is narrow here, and its construction is more archaic. It's now
mostly brick, although the floor is concrete. The steam pipe and coaxial cable
continue along their appointed paths. The tunnel is damp and even a little
muddy.

>e
Steam Tunnel
The steam pipe and coaxial cable turn upwards and disappear into the ceiling
here. The tunnel itself comes to an end in a grimy, damp, and dripping triad of
crumbling brick walls. The south wall looks particularly decrepit.

>remove bricks with crowbar then g
The wall grudgingly yields to your efforts. A brick, less well mortared than its
fellows, pulls out of the wall.

The wall grudgingly yields to your efforts. A brick, less well mortared than its
fellows, drops to the floor on the other side, making a hole through the wall.
You can see a rusty steel reinforcing rod in the hole.

>w
Steam Tunnel

>w
Steam Tunnel

There is a dead rat here.

>w
Steam Tunnel

>u
Tomb

>turn off light
The flashlight clicks off.

>drop knife and crowbar
knife: Dropped.
crowbar: Dropped.

>se
Subbasement

There is a fire axe, a metal flask and a bolt cutter here.

>get all
fire axe: Taken.
metal flask: Taken.
bolt cutter: Taken.

>u
Stairway

>e
Aero Basement

>e
Basement

>u
Computer Center

>press down
The down-arrow begins to glow.

>d
Basement

>z
Time passes...

You hear the elevator begin moving.

>z
Time passes...

You no longer hear the elevator moving.

>wedge doors with axe
You force the doors apart with the fire axe, revealing a dark elevator shaft.
The fire axe wedges nicely between the doors, holding them securely open. In the
elevator shaft a tangle of machinery is visible. One bit of the tangle is very
much like a hook.

>d
You drop to the floor below, which isn't all that far down.

Concrete Box
This small room is pretty bare and featureless. The north wall is brick and the
other three walls are concrete. The east and west walls are adorned with rails
not unlike railroad rails. Above you and out of reach, the shaft is blocked by
something.

The brick wall has a hole in it. There is a vertical reinforcing rod visible in
the middle of the hole.

There is a greasy length of chain coiled on the floor here.

There is a new brick here.

>get chain
Taken.

>tie chain to rod
You wrap the chain around the rod, but find it too thick to actually tie it.

>lock chain with lock
You lock the chain with the padlock. It now forms a secure loop around the rod.

>u
You climb up the gritty wall and out into the basement.

Basement

The elevator doors are wedged open with a fire axe, revealing a dim elevator
shaft beyond.

In the elevator shaft a tangle of machinery is visible. One bit of the tangle is
very much like a hook.

>put chain on hook
You hook the chain to the hook, where it looks quite secure.

>get axe
You take the fire axe away, and the doors spring shut.

>u
Computer Center

>u
Second Floor

>press down
The down-arrow begins to glow.

You hear the elevator begin moving.

>d
Computer Center

<SOUNDS ,S-ELCRSH>
[<SOUND 7 2 8>]

From below, you hear a tearing, rending sound, then a rumbling crash.

You no longer hear the elevator moving.

>d
Basement

>wedge doors with axe
You force the doors apart with the fire axe, revealing a dark elevator shaft.
The fire axe wedges nicely between the doors, holding them securely open. The
dark shaft opens like a waiting mouth.

>d
You drop to the floor below, which isn't all that far down.

Concrete Box
This small room is pretty bare and featureless. The north wall is brick and the
other three walls are concrete. The east and west walls are adorned with rails
not unlike railroad rails. Above you is an empty shaft. On the north side of the
shaft is an elevator door wedged open by a fire axe.

The brick wall has an enormous hole ripped in it.

There is a new brick here.

>get axe
You take the fire axe away, and the doors spring shut, leaving you in the dark.

>light
(flashlight)
The flashlight clicks on.

Concrete Box
This small room is pretty bare and featureless. The north wall is brick and the
other three walls are concrete. The east and west walls are adorned with rails
not unlike railroad rails. Above you is an empty shaft. On the north side of the
shaft is an elevator door.

The brick wall has an enormous hole ripped in it.

There is a new brick here.

>n
Steam Tunnel

The southern brick wall has an enormous hole ripped in it.

There is a broken brick here.

>w
Steam Tunnel

>w
Steam Tunnel

There is a dead rat here.

>w
Steam Tunnel

>w
Steam Tunnel
This dank and grimy tunnel is largely filled with an imperfectly insulated steam
pipe. The tunnel is uncomfortably hot and damp. A bundle of coaxial cable runs
along the ceiling, festooned with damp mold and cobwebs. The tunnel continues
west.

>w
Tunnel Entrance
The tunnel continues west from here, becoming narrow, muddy, and forbidding. The
walls no longer seem to be as finished as they were. The steam pipe and coaxial
cable disappear into the ceiling at this point. The temperature has dropped
considerably, as well.

>d
Muddy Tunnel
The tunnel you came through continues down, barely large enough to enter. It is
made of sticky gelatinous mud that's been pushed by something into a semblance
of a passage.

>d

<SOUNDS ,S-VOICE ,S-START 8>
[SOUND 15 IS A LOOPING SOUND.]
[<SOUND 15 2 8>]
Large Chamber
This is a wide spot in the tunnel, just as wet and muddy as elsewhere. The walls
are slimy as well. Numerous slots or indentations about two feet wide and a foot
high open here and there. Thin, wire or ropelike growths emerge from a hole
further down and enter each of the slots. There is background noise here, almost
loud enough to hear clearly.

A small, furtive motion attracts your attention to the slots.

>cut wires with cutter
You strain and push the two handles of the bolt cutter together with all your
strength. At first it looks like nothing will happen, but then, with a loud
click, the jaws cut the wire!

The wire, as though under tension, rapidly begins to curl up, disappearing down
the tunnel and away. Urchins burst forth from the slots. 
<SOUNDS ,S-VOICE ,S-STOP>
[SOUND 15 IS A LOOPING SOUND.]
[<SOUND 15 3>]
The effect on the urchins is electric (perhaps literally). They twitch, jerk
spasmodically, and fall to the ground almost in unison. They have lost all
interest in you.

>d
A few of the urchins grab feebly at you as you pass, but none is a serious
barrier.


<SOUNDS ,S-VOICE ,S-STOP>
[SOUND 15 IS A LOOPING SOUND.]
[<SOUND 15 3>]
Wet Tunnel
You are lost in narrow, wet tunnels burrowed through the mud. Muddy, oily water
covers the floor.

The hand points its mutilated ring finger north and grips your shoulder tightly.

>n
Wet Tunnel
You are lost in narrow, wet tunnels burrowed through the mud. Muddy, oily water
covers the floor.

The hand points its mutilated ring finger down and grips your shoulder tightly.

>d
Wet Tunnel
You are lost in narrow, wet tunnels burrowed through the mud. Muddy, oily water
covers the floor.

The hand points its mutilated ring finger south and grips your shoulder tightly.

>s
Wet Tunnel
You are lost in narrow, wet tunnels burrowed through the mud. Muddy, oily water
covers the floor.

The hand points its mutilated ring finger south and grips your shoulder tightly.

>s
Wet Tunnel
You are lost in narrow, wet tunnels burrowed through the mud. Muddy, oily water
covers the floor.

The hand points its mutilated ring finger down and grips your shoulder tightly.

>d
Wet Tunnel
You are lost in narrow, wet tunnels burrowed through the mud. Muddy, oily water
covers the floor. A curtain of moldy slime covers the south wall.

The hand points its mutilated ring finger south and grips your shoulder tightly.

>open flask
You open the flask, and a cold, white mist boils out.

>pour liquid on slime
The liquid splashes onto the curtain, and a cold mist fills the room. The slime
begins to freeze. Nearly the entire curtain solidifies, shatters, and drops to
the ground, revealing an ancient wooden door.

>unlock door with key
The door is now unlocked.

The mist dissipates.

>open door
Okay, the ancient door is now open.

>s

<SOUNDS ,S-ZOMBIE
[SOUND 17 IS A LOOPING SOUND.]
[<SOUND 17 2 8>]
As you enter, the door closes and locks behind you.

Inner Lair
The floor here is a stagnant, slime infested pool of water. It feels to be about
six inches deep. Ropes or wires tumble down the slope, where they enter a large
whitish mass which takes up much of the chamber. The noise is loud here, and
comes from the mass, which undulates in synchrony with the noise. Wan,
sourceless light illuminates the chamber.

Set on the wall, incongruous in its surroundings, is a metal box. On one side a
coaxial cable enters the box. On the other, a cablelike appendage leads from the
box to the mass. There is a metal cover on the box.

Suddenly, the hand leaps from your shoulder into the slime-encrusted puddle. It
dives beneath the water.

>reach into pool
You root around blindly in the gooey, slimy water. You feel something thick and
slippery! A tentacle? No, it's cold and dead. It seems to be a line of some
kind, just below the surface.

You hear noises outside the door.

>get line
You pull a length of the line out of the water. It's like holding a large, heavy
snake.

You hear a stumbling noise behind you, turn and see the hacker staggering into
the cavern.

The hacker stares at you, shocked. "It's you! When I gave you my key, I never
suspected you'd get this far!"

>cut it with axe
You strike the line with the axe, making a deep gash in the insulation.

The hacker stares at the thing in the cave. "I got very suspicious about your
problems with the net. I began to trace some coax, found some repeaters and
bridges that weren't on the layout charts, and started following them. Anyway,
here I am. That thing there, whatever it is, and those wires, are interfaced to
the whole campus net. And that means it's tied into all the nets, commercial,
government, even military, potentially."

>g
Your blow cuts through more insulation and into the conductors.

"I guess I better do something. It could be a serious compromise of system
integrity if this thing isn't dealt with." He peers at the mass, as if
evaluating it. He then reaches into a pocket and pulls out a small pair of wire
strippers.

>g
The line parts! The two ends begin to sink towards the water as they straighten
out.

The hacker advances on the mass, apparently planning to cut some of the wires
leading into it. As he approaches it, the sound stops completely, and the wires
begin a frantic, looping, twining dance. The mass begins to flow towards the
hacker almost as quickly as he walks toward it. They reach each other and begin
to merge together. He screams; a long, ululating cry that echoes through the
cavern. Then he is engulfed.

>get line
You carefully grab the line just as it's about to drop into the water.

The mass is bulging, vibrating, and rippling.

>open cover
You remove the cover, revealing a plethora of electronic innards. Most prominent
are a socket into which the coaxial cable is plugged, and a connector into which
the glistening cablelike appendage disappears.

The mass is bulging, vibrating, and rippling.

>unplug coaxial
You unscrew the cable. It now hangs unconnected inside the box.

The mass is bulging, vibrating, and rippling. A huge tear is forming near where
the hacker was absorbed.

>put line in socket

<SOUNDS ,S-SPARKY>
[<SOUND 11 2 8>]
You shove the exposed conductors into the socket, producing a shower of sparks!
The tentacle connected to the other socket begins to jerk and twitch
spasmodically. The mass it's connected to quivers, and a horrible noise, almost
like a huge machine running without oil, issues from the thing.
<SOUNDS ,S-CRETIN>
[SOUND 16 IS A LOOPING SOUND.]
[<SOUND 16 2 8>]


The mass begins to change shape, compacting, darkening. You can briefly see
human outlines within the grey, gelatinous mass. They surround something larger,
of a shape not human, not animal, like nothing you've seen before.

<SOUNDS ,S-CRETIN ,S-STOP>
[SOUND 16 IS A LOOPING SOUND.]
[<SOUND 16 3>]

The gelatinous mass solidifies and compacts, leaving behind a litter of smoking
debris. In the debris squats a being. Huge, misshapen, it stares at you with
baleful yellow eyes. Its scaly wings beat slowly, driving a fetid stench through
the stale air of the cavern. A barbed tongue slides across its broken,
daggerlike fangs.

The smooth stone vibrates. It starts to feel warm.

>z
Time passes...

The thing tenses, preparing to leap. Its mouth opens, revealing not the
glistening interior, but a dead-black outline like a hole into nothingness.

The smooth stone is now glowing with a bright-red heat that nevertheless fails
to burn you.

>throw stone at creature
The stone smashes into the creature, sticking to its ichorous hide. The thing
thrashes about, trying to bite at the stone, which is glowing brighter and
brighter. Small hands issue from beneath its scales to tug in vain at the
irritant. The creature begins to show gaping holes of dark, light-devouring
nothingness around the stone. Its wings spread painfully, as though it were
trying to fly away, and then fold. It widens its jaw in an almost human scream
of agony. The black hole of its maw overwhelms it, and indeed the creature
appears to be swallowing itself. At last, a grey cloud of greasy smoke surrounds
the glowing stone, still suspended in midair. Then even that vanishes, and the
stone drops to the ground, no longer glowing. The thing is gone.

>get stone
[<SOUND 8 2 8>]

<SOUNDS ,S-CRACK>
You pick up the stone. It has a long jagged crack that almost breaks it in half.
As you pick it up, you feel it bump to one side. Then, as you are holding it in
your hand, something pushes its way out through the crack, breaking the stone
into two pieces. Something small, pale, and damp blinks its watery eyes at you.
It hisses, gaining strength, and spreads membranous wings. [<SOUND 9 2 8>]

<SOUNDS ,S-GHIDRA>
It takes to the air, at first clumsily, then with increased assurance, and
disappears into the gloom. One eerie cry drifts back to where you stand.

Something rises out of the mud, slowly straightening. The hacker, mud-covered
and weak, staggers to his feet. "Can I have my key back?" he asks.
[<SOUND 0 3>]
[<SOUND 0 4>]

Your score is 100 of a possible 100, in 315 moves. Graded on the curve, you are
in the class of President of the Institute.

Would you like to restart the game from the beginning, restore a saved game
position, or end this session of the game?
(Type RESTART, RESTORE, or QUIT):
>quit
```

</details>

---
### Steps I use during testing

<details><summary> CLICK HERE TO VIEW THE STEPS </summary>

```
turn on pc
login 872325412
password uhlersoth
click edit
click yak
x paper
* last command in next line plays sound S-DRONE (10) [looping]
click more then g then g then g
d
d
get stone
x it
* next command stops sound S-DRONE (10)
throw it at creature
ask hacker about keys
ask him about master
ask him for master
z
s
w
open fridge
get coke and carton
open carton then open oven
put carton in oven then close oven
press 4 then press 0 then press 5
press start
z then z then z then z
open oven
get carton
e
n
give carton to hacker
ask him for master
s
press down
z
z
in
open panel
get light
press open
out
d
d
e
get gloves and crowbar
wear gloves
u
light
get flask
d
turn off light
w
w
w
u
s
get container
e
z
z
z
z
e
u
climb rope
lower ladder
open door
out
u
remove plug
drop plug
get paper
d
in
d
d
e
z
z
z
z
* z (only if the man has not yet gone east, and repeat until he does!)
break glass
get axe
w
throw axe at cord
open container
empty it
drop container and assignment
e
e
n
d
se
get boots
wear them
u
u
unlock door with key
open door
out
u
d
* next move plays sound S-DIE (12) [no looping]
throw stone at creature
u
dig in tub
get hand
d
in
d
s
get stone
n
d
nw
u
s
w
w
w
w
n
d
e
enter lift
start it
e
e
e
light
remove junk with lift then g then g then g
e
exit lift
open manhole with crowbar
d
n
d
get knife
u
s
u
w
w
turn off light
w
w
w
u
s
e
e
e
e
s
knock on door
z
give paper to professor
s
z
z
cut line with knife
* next move plays sound S-PSYCHO (13) [looping]
exit
move bench
open trapdoor
* next move stops sound S-PSYCHO (13)
d
open trapdoor
light
u
put hand in liquid
z
z
get hand and ring
put ring on hand
drink coke
n
open door
n
turn off light
n
* [start looking for the urchin along the way at this point]
w
w
w
w
n
* [TIME TO FIND THE URCHIN, IF WE HAVE NOT ALREADY]
show hand to urchin
get cutter
* [GO TO BACK TO THE STAIRWAY and DOWN TO THE SUB-BASEMENT]
drop cutter, flask, and axe
nw
unlock lock with key
get lock
* next move plays sound S-HATCH (6) [no looping]
open hatch
light
* next move plays sound S-ATTACK (4) [looping]
d
e
turn valve with crowbar
z
z
z
* next move stops sound S-ATTACK (4)
open valve
close valve
e
e
remove bricks with crowbar then g
w
w
w
u
turn off light
drop knife and crowbar
se
get all
u
e
e
u
press down
d
z
z
wedge doors with axe
d
get chain
tie chain to rod
lock chain with lock
u
put chain on hook
get axe
u
u
press down
d
* next move plays sound S-ELCRSH (7) [no looping]
d
z
wedge doors with axe
d
get axe
light
n
w
w
w
w
w
d
* next move plays sound S-VOICE (15) [looping]
d
* next move stops sound S-VOICE (15)
cut wires with cutter
d
n
d
s
s
d
open flask
pour liquid on slime
unlock door with key
open door
* next move plays sound S-ZOMBIE (17) [looping]
s
reach into pool
get line
cut line with axe
g
g
get line
open cover
unplug coax
* next move plays two sounds then stops the second one at the end of the same routine
* 1) it plays sound S-SPARKY (11) [no looping]
* 2) then it plays sound S-CRETIN (16) [looping]
* 3) then it stops sound S-CRETIN (16)
put line in socket
z
throw stone at creature
* next move plays two sounds
* 1) it plays sound S-CRACK (8) [no looping]
* 2) then it plays sound S-GHIDRA (9) [no looping]
get stone
```

At this point, the game ends, and `KILL-SOUNDS` is called, which in turn calls `sound_effect 0 3;` then `sound_effect 0 4;`, which does not stop these last two synchronous sounds in frotzpl3, because frotzpl3 runs a `while` loop throughout each sound's duration when there are synchronous `sound_effect` calls.

Also noteworthy: the only times `KILL-SOUNDS` is called during this is when the game ends or if we enter $SOUND to disable sounds during play.

The game only uses ID `0` in the `KILL-SOUNDS` script. Real IDs are used otherwise.

</details>

---
### Screen Recordings

https://drive.google.com/drive/folders/1Lj4qFd-GUPjm_dknMsc2czqisZDYPBTs?usp=sharing

---
## Conclusion (sort of)

So... This game's sounds need to be created quite a few different ways if we want it to work in all the terps that can play sounds.

For the Infocom AMIGAZIP terp, we need three files (NAM, MID, and DAT), and the data needs to be RAW, mono, 8-bit signed (with a custom header).

For frotzpl3 for DOS, we need one SND file, which is RAW, mono, 8-bit unsigned (with a custom header).

For Filfre, we need AIFF.

For a blorb for Windows Frotz, we can use OGG, AIFF, or MOD.

For a blorb for Parchment, we need OGG.

For a blorb for Gargoyle, I need to double-check the rest of the formats, but AIFF definitely works, and it reads the LOOP chunk from the blorb for V3 (although it doesn't use it for *The Lurking Horror*).

For a blorb for Frotz on Linux, just like Gargoyle, I need to double-check the rest of the formats, but AIFF definitely works, and it reads the LOOP chunk from the blorb for V3 (although it doesn't use it for *The Lurking Horror*).

For Ozmoo, we can use WAV with custom settings, or AIFF.
