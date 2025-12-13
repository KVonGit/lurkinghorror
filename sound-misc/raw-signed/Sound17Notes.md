# Concerning Sound 17

We can read the hex data from the sound files on the Amiga disk.

The header shows that the original sample rate of the data was 32910, and the base note was 50.

The SND files on if-archive do not use the Amiga DAT files' sample rate values. They instead bypass the need for MID files and use the result of the formula which determines the frequency used to actually play the sound:

https://www.ifarchive.org/if-archive/infocom/info/sound_format.txt

```
  pow (2, ("note" - "base note") / 12) * "sample frequency" / 4
```


Here is the hex dump of **s17.mid**:

```
Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

00000000  00 09 90 5A 40 FF 00 07 90 5A 00                 
```

The note value is $5a (which is 90 in decimal).

Let's plug our values into that formula.

```
  pow(2, (90 - 50) / 12) * 32910 / 4
```

That ends up being (roughly) 82928. The Big Endian word value cannot be higher than 65535; so, it seems that is why this is the value used in the DOS version of Sound 17 from if-archive:

```
FILE: LURKIN17.SND
NOTE: 50 (32 HEX)
SAMPLE_FREQ: 65535 (FF FF)
SOUND_DATA_LEN: 39248 (99 50)
```

That plays the sound at more than 2X speed. When playing the original game on Amiga Forever, that is not the case. On Amiga, the sound (to my ear) is played at its original 32910. (My guess is it defaults to the original sample frequency value because it cannot handle playing a sound at 65,535?)

Also, Sound 17 in the blorb on if-archive uses the original 32910, where the majority of those sounds use the same frequency as the DOS sounds.

I'm still leaving the value in the recreated Amiga files at $5a for two reasons:
- I don't fully understand all this sound stuff, especially on DOS and Amiga. =)
- It sounds fine this way on Amiga. ;)
