# Concerning Sound 16

https://github.com/historicalsource/lurkinghorror/blob/51d806017f4cc630ede28ca4d92b9c7e9b9f0361/frob.zil#L1444-L1471

```zil
		       %<IFSOUND <SOUNDS ,S-SPARKY>>
		       <TELL
"You shove the exposed conductors into the socket, producing a shower of
sparks!">
		       <COND (<NOT <FSET? ,GLOVES ,WEARBIT>>
			      <MOVE ,HIGH-VOLTAGE ,HERE>
			      <QUEUE I-LINE-IN-WATER 2>
			      <TELL
" The sparks burn your hands! You jerk back, dropping the line!" CR>)
			     (<NOT <FSET? ,BOOTS ,WEARBIT>>
			      <JIGS-UP
" You are standing in electrified water. You are even
grounded. Do you know what this means? Can you say \"dead?\"">)
			     (<FSET? ,OUTPUT-CABLE ,RMUNGBIT>
			      <MOVE ,HIGH-VOLTAGE ,INPUT-SOCKET>
			      <TELL
" The stump of the tentacle still connected to the connector
shrivels and fries." CR>)
			     (<NOT <QUEUED? I-FROB-APPEARS>>
			      <MOVE ,HIGH-VOLTAGE ,INPUT-SOCKET>
			      <DEQUEUE I-HACKER-RETURNS>
			      <QUEUE I-FROB-APPEARS -1>
			      <TELL
" The tentacle connected to the other socket begins to jerk and twitch
spasmodically. The mass it's connected to quivers, and a horrible
noise, almost like a huge machine running without oil, issues from
the thing.">
			      %<IFSOUND <SOUNDS ,S-CRETIN>>
```

---

To begin with, Sound 11 plays.

If we are wearing boots and gloves and have unplugged the coaxial cable, then I-FROB-APPEARS is queued (with -1 for the argument), and sound_effect is called to play Sound 16.

On a slow Amiga (which was the most common at the time), Sound 11 would finish playing before the Amiga had time to load Sound 16. This was an exploit to play synchronous sounds. (It only happens one other time in this game.)

Also, if the conditions are not met (unplugged coaxial cable already, and are wearing boots and gloves), then Sound 16 does not play. All we hear is Sound 11 (the sound of electricity).

So, using synchronous sounds here sort of makes sense, I guess.

...but I digress. The true reason this note exists is Sound 16 has its 'repeats' set to 0, for an infinite loop. However, the sound seems to be scripted to be stopped at the end of the same turn (by I-FROB-APPEARS).

https://github.com/historicalsource/lurkinghorror/blob/51d806017f4cc630ede28ca4d92b9c7e9b9f0361/frob.zil#L1609

```zil
<ROUTINE I-FROB-APPEARS ()
	 <SETG END-CNT <+ ,END-CNT 1>>
	 <COND (<EQUAL? ,END-CNT 1>
		<REMOVE ,HACKER>
		<REMOVE ,MASS>
		%<IFSOUND <SOUNDS ,S-CRETIN ,S-STOP>>
```

---

Some of the more popular modern interpreters will not even play this sound audibly, even when they have special code to handle this game. Their special code allows two synchronous sounds (only when the sound ID is either 9 or 16), but it does not handle the stopping action.

When playing the original Infocom disk on Amiga (using an Amiga 500 emu in Amiga Forever), this sound only plays once.

I see no need for the loop.

Here is an excerpt from a transcript where I'm playing a build of this game with sound debugging messages in the game's text:

```
>PUT LINE IN SOCKET

[HIGH-VOLTAGE-F: <SOUNDS ,S-SPARKY>]
[<SOUNDS 11 2 8>]
[SOUND-FLAG[0]: 1]
[SOUND-FLAG[1]: 0]
[SOUND 11 2 8]
You shove the exposed conductors into the socket, producing a shower of sparks!
The tentacle connected to the other socket begins to jerk and twitch
spasmodically. The mass it's connected to quivers, and a horrible noise, almost
like a huge machine running without oil, issues from the thing.
[HIGH-VOLTAGE-F: <SOUNDS ,S-CRETIN>]
[<SOUNDS 16 2 8>]
[SOUND 16 LOOPS]
[SOUND-FLAG[1]: 264]
[SOUND-FLAG[0]: 264]
[SOUND-FLAG[1]: 264]
[SOUND 16 2 8]


The mass begins to change shape, compacting, darkening. You can briefly see
human outlines within the grey, gelatinous mass. They surround something larger,
of a shape not human, not animal, like nothing you've seen before.

[I-FROB-APPEARS: <SOUNDS ,S-CRETIN ,S-STOP>]
[<SOUNDS 16 3>]
[SOUND-FLAG[1]: 264]
[STOPPING SOUND 16]
[SOUND-FLAG[0]: 1]
[SOUND-FLAG[1]: 0]
[SOUND 16 3]

The gelatinous mass solidifies and compacts, leaving behind a litter of smoking
debris. In the debris squats a being. Huge, misshapen, it stares at you with
baleful yellow eyes. Its scaly wings beat slowly, driving a fetid stench through
the stale air of the cavern. A barbed tongue slides across its broken,
daggerlike fangs.

The smooth stone vibrates. It starts to feel warm.

>
```

