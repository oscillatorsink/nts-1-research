# Patch info sysex format for the NTS-1
## Caveat
This is a work in progress and almost certainly contains unknowns and errors. This research was conducted on an NTS-1 running firmware 1.10 and v1.0.0 of the Librarian software.

## Hello message
Upon connecting the librarian sends the following to check that the NTS-1 is present:

### Request hello
`F0 42 50 00 02 F7`

| Offset | Length | Data   | Meaning
| ------ | ------ | ------ | -------
| 0      | 1      | 0xF0   | Start Sysex
| 1      | 1      | 0x42   | Korg
| 2      | 1      | 0x50   | ??? - might identify the message as being related to a device check?
| 3      | 1      | 0x00   | Because of the response detailed below I'm guessing this indicates that this is a request rather than response
| 4      | 1      | 0x02   | ???
| 5      | 1      | 0xF7   | End Sysex
