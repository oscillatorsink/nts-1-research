# Patch info sysex format for the NTS-1
## Caveat
This is a work in progress and almost certainly contains unknowns and errors. This research was conducted on an NTS-1 running firmware 1.10 and v1.0.0 of the Librarian software.

## General observations
### One 0x00 in 15
It looks like, after the first byte of (all?) messages (so that's byte 3 onwards), every 15 bytes a single nul (0x00) byte is inserted into the message. It might be for checking that the message isn't corrupted maybe? It's a bit weird. You can see it here:

```
Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

00000000  F0 42 30 00 01 57 49 01 00 00 00 01 03 00 01 01  ðB0..WI.........
00000010  00 44 00 75 6B 65 00 00 00 00 00 00 06 00 00 43  .D.uke.........C
00000020  72 75 00 73 68 65 72 00 00 00 00 00 00 00 00 00  ru.sher.........
00000030  00 00 00 00 F7                                   ....÷
```

You can see it at offsets `0x12 0x22 0x32`; it's especially clear where it's breaking up the name of the developer and effect.

## Hello message

### Request hello
Upon connecting the librarian sends the following to check that the NTS-1 is present: 

6 bytes: `F0 42 50 00 02 F7`

| Offset | Length | Data     | Meaning
| ------ | ------ | -------- | -------
| 0      | 1      | `0xF0`   | Start Sysex
| 1      | 1      | `0x42`   | Korg
| 2      | 1      | `0x50`   | ??? - I think this indicates that this is a message _related to_ getting device IDs*
| 3      | 1      | `0x00`   | ??? - this is a requst for identification
| 4      | 1      | `0x02`   | ???
| 5      | 1      | `0xF7`   | End Sysex

\* actually by convention this is probably in the bitwise form `0b xttt ccc` where `ttt` is the type and `cccc` is the channel?..

### Respond hello
The NTS-1 responds thus:

15 bytes: `F0 42 50 01 nn 02 57 01 00 00 MM MM mm mm F7`

| Offset | Length | Data     | Meaning
| ------ | ------ | -------- | -------
| 0      | 1      | `0xF0`   | Start Sysex
| 1      | 1      | `0x42`   | Korg
| 2      | 1      | `0x50`   | ??? - I think this indicates that this is a message _related to_ getting device IDs...
| 3      | 1      | `0x01`   | ??? - this is an identification response
| 4      | 1      | `0xnn`   | Currently configured MIDI channel (0x00-0x0F; channel 1 = 0x00)
| 5      | 1      | `0x02`   | ???
| 6      | 1      | `0x57`   | This appears to be a device ID of sorts as it turns up in most other messages subsequently. Might indicate the NTS-1 specifically.
| 7      | 3      | `0x01 00 00` | ???
| 10     | 2      | `0xMM`   | Major firmware version (little endian I think, otherwise I'm out by one byte; so for me on 1.10 this was `0x 01 00` for the '1')
| 12     | 2      | `0xmm`   | Minor firmware version (little endian I think, otherwise I'm out by one byte; so for me on 1.10 this was `0x 0A 00` for the '10')
| 14     | 1      | `0xF7`   | End Sysex

## User program info
To request the details of a user program (oscillator or fx) the librarian sends the following:

10 bytes: `F0 42 3n 00 01 57 19 tt ss F7`

| Offset | Length | Data   | Meaning
| ------ | ------ | ------ | -------
| 0      | 1      | `0xF0`   | Start Sysex
| 1      | 1      | `0x42`   | Korg
| 2      | 1      | `0x3n`   | Message "type", where `n` is the MIDI channel identifid during the "hello" stage above
| 3      | 2      | `0x00 01` | ???
| 5      | 1      | `0x57`   | "Device ID" identified during "hello" stage (may indicate NTS-1 specifically)
| 6      | 1      | `0x19`   | Slot request
| 7      | 1      | `0xtt`   | Slot type (see below)
| 8      | 1      | `0xss`   | Slot number (0x00 = slot 1)
| 9      | 1      | `0xF7`   | End sysex

The _Slot type_ indicates what sort of user program we're referring to (osc, mod, delay, reverb). This seems to line up with the enum in `/platform/nutekt-digital/inc/userprg.h` from the 'logue SDK.

| Value | Meaning
| ----- | -------
| 1     | Mod
| 2     | Delay
| 3     | Reverb
| 4     | Oscillator

So to get the user delay in slot 3 with the NTS-1 receiving on MIDI channel 2 we'd send: 
`F0 42 31 00 01 57 19 02 02 F7`

The user oscillator in slot 11 with the NTS-1 receiving on MIDI channel 3 we'd send: 
`F0 42 32 00 01 57 19 04 0A F7`

Interestingly, despite there only being 8 slots each for delay and reverb on the NTS-1, version 1.0.0 of the Librarian still asks for slots 9-16. Nobody's perfect.