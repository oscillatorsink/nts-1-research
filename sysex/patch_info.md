# Patch info sysex format for the NTS-1
## Caveat
This is a work in progress and almost certainly contains unknowns and errors. This research was conducted on an NTS-1 running firmware 1.10 and v1.0.0 of the Librarian software.

## Hello message

### Request hello
Upon connecting the librarian sends the following to check that the NTS-1 is present:
`F0 42 50 00 02 F7`

| Offset | Length | Data   | Meaning
| ------ | ------ | ------ | -------
| 0      | 1      | 0xF0   | Start Sysex
| 1      | 1      | 0x42   | Korg
| 2      | 1      | 0x50   | ??? - I think this indicates that this is a message _related to_ getting device IDs...
| 3      | 1      | 0x00   | ??? - this is a requst for identification
| 4      | 1      | 0x02   | ???
| 5      | 1      | 0xF7   | End Sysex

### Respond hello
The NTS-1 responds thus:

`F0 42 50 01 nn 02 57 01 00 00 MM MM mm mm F7`


| Offset | Length | Data   | Meaning
| ------ | ------ | ------ | -------
| 0      | 1      | 0xF0   | Start Sysex
| 1      | 1      | 0x42   | Korg
| 2      | 1      | 0x50   | ??? - I think this indicates that this is a message _related to_ getting device IDs...
| 3      | 1      | 0x01   | ??? - this is an identification response
| 4      | 1      | 0xnn   | Currently configured MIDI channel (0x00-0x0F; channel 1 = 0x00)
| 5      | 1      | 0x02   | ???
| 6      | 1      | 0x57   | This appears to be a device ID of sorts as it turns up in most other messages subsequently. Might indicate the NTS-1 specifically.
| 7      | 3      | 0x01 00 00 | ???
| 10     | 2      | 0xMM   | Major firmware version (little endian I think, otherwise I'm out by one byte; so for me on 1.10 this was `0x01 00` for the '1')
| 12     | 2      | 0xmm   | Minor firmware version (little endian I think, otherwise I'm out by one byte; so for me on 1.10 this was `0x0A 00` for the '10')