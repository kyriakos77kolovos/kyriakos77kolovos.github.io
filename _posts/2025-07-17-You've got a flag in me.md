---
title: You've got a flag in me
date: 2025-07-17 12:00:00 +0300
categories: [CTFs, CCSC2025]
tags: [misc, ctf, python]     # TAG names should always be lowercase
---
# You've got a flag in me
![img-description](/assets/img/chall.png)


_The Challenge_

For CCSC’s CTF, I encountered a fun “misc” challenge involving a MIDI file. The description teased:
`“This pretty midi file looks like it sings something. The acoustic grand piano is so soothing!”`
The flag format was ECSC{...}, and I needed to find the hidden message encoded in the MIDI.

My first instinct was to poke around with `file` and `strings`. I spotted a suspicious-looking “KEY” array in the file:

```shell
kr7pt0@kr7pt0~$ file flag.midi
flag.midi: Standard MIDI data (format 0) using 1 track at 1/240
```

```shell
kr7pt0@kr7pt0~$ strings flag.midi
MThd
MTrk
KEY=[7,58,391,58,129,80,537,80,389,33,80,107,522,391,389,148,386,522,389,58,240,240,107,1]
```
Clearly, this was meant for something! I then opened the MIDI with Python’s `mido` to see what instruments and notes were present.


By examining the MIDI data, I learned that only channels without a `program_change` are set to the acoustic grand piano (program 0) by default. In this file, channel 2 fit the bill, and, crucially, it also contained note data.

The phrase “so soothing” from the description got me thinking, maybe only notes played “gently” were important? In MIDI, a `note_on` event with velocity zero actually signals the end of a note (essentially a `note_off`). Could this be the key?
I wrote a script to extract all notes from channel 2 where the velocity was exactly 0.


At this point, my approach seemed logical, but I still had the mysterious KEY array and wasn’t sure how it fit in. After about 15 minutes of staring at my ceiling, inspiration struck: the first four numbers in the key `7, 58, 391, 58` lined up interestingly with the expected flag prefix `ECSC`. In particular, the fact that both the third and fourth flag letters are “C” and the key repeats “58” caught my attention—maybe the numbers were indexes?
I scrolled through my list of extracted notes with velocity 0 and checked the note at index 58. When I converted it to ASCII, the letter “C” appeared—bingo! From there, I wrote a simple Python script to automate extracting all the key-indexed notes and decoding the flag.


```python
from mido import MidiFile

# The KEY as given in the challenge
KEY = [7,58,391,58,129,80,537,80,389,33,80,107,522,391,389,148,386,522,389,58,240,240,107,1]

mid = MidiFile('flag.midi')
zero_velocity_notes = []

# 1. Collect note numbers from note_on (channel 2, velocity 0)
for msg in mid:
    if msg.type == 'note_on' and msg.channel == 2 and msg.velocity == 0:
        zero_velocity_notes.append(msg.note)

# 2. Use KEY as index to extract the flag notes
flag_notes = []
for idx in KEY:
    if idx < len(zero_velocity_notes):
        flag_notes.append(zero_velocity_notes[idx])
    else:
        flag_notes.append(None)

# 3. Print result as both note numbers and ASCII
print("Extracted note numbers:")
print(flag_notes)
print("\nAs ASCII characters (where printable):")
flag_string = ""
for n in flag_notes:
    if n is not None and 32 <= n <= 126:
        flag_string += chr(n)
    else:
        flag_string += '?'
print(flag_string)
```


```shell
kr7pt0@kr7pt0~$ python3 sol.py
ECSC{M1D1_F1L3S_4R3_C00L!}
```




