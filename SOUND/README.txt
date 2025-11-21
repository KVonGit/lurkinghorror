The files LURKIN03.SND through LURKIN18.SND are for Frotz on DOS.


Converted all sounds from the Amiga r221 DAT files
- HxD to remove first 10 bytes (header)
- Audacity:
  - import RAW 8-bit signed at MID determined frequency (actual freq_to_play)
  - export RAW 8-bit unsigned at same frequency (actual freq_to_play)
- HxD to create new 10 byte header using actual frequency values
  - this 17 is imported at its orig freq and exported using the same freq 32910,
    which I *think* is what ends up being used on Amiga on all releases...
    ...because the freq_to_play ends up being too high after converting the
       value using the MID's note (90) with the DAT's base_note (50): 82928


Sounds 11 and 16 have been combined into 20, although 11 might play on its own 
in some cases (I didn't use ID 16 because terps with code to handle TLH loop 16)

The repeats value of 16 has been changed to 1, just in case it gets called.

The routine that calls 11 then 16 also stops 16 at the same time, which skips
both sounds in some terps.


Also, sounds 8 and 9 have been combined into 21, as they only play synchronously


The ZIL has also been slightly modified. S-CRETIN is no longer a loop (and no 
longer called). It calls 20 rather than 11 & 16 in that bit of code now, and it
no longer tries to stop 16. It also calls 21 rather than 8 & 9 in *that*
particular bit of code.

(There is a ZBLORB with modified sounds in the OGG format, as well.)

With these changes, most terps seem to handle this version of the game without
identifying it as The Lurking Horror, as this is a new serial and release number
