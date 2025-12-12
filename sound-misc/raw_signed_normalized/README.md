These files were created using these files:
https://unbox.ifarchive.org/?url=/if-archive/infocom/media/sound/lhsound.zip

For each file:
- made notes concerning the header data
- used HxD to remove the header

These are the files in this directory.

To create the actual sound files (in the "Sound" directory), I then did this with each file:
- imported RAW 8-bit unsigned data to Audacity, using the sample rate from the header data
- normalized sounds to -2 if they red-lined
- exported that as RAW 8-bit signed data
- used HxD to create the header, saving as **.dat**

---
ALSO SEE Sound17Notes.md
