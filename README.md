# Multiplex PCM Format

This document is a specification of the Multiplex PCM format, which was introduced around the year 2007. It is used for RC radio transmitters to encode channel values in a digital form (as opposed to the analog PPM format).

This specification was derived from a PCM signal as generated by a Multiplex Profi mc3030 radio.

## Overview

A full PCM frame consists of a SYNC bit, followed by 8 channel values CHx. Each channel value CHx is composed of five symbols representing 10-bits (8-bit value and 2-bit parity). It is followed by two frame type symbols (4 bits total). The last two channel values are interleaved and represent channel 7 to channel 10 in alternating frames. The last four frame type bits determine which channel is in the 7th and 8th channel value.

A typical 10-channel frame as generated by a Profi mc3030 radio has a length of 55ms, which is constant due to the symbol encoding used. The frame is followed by a GAP pulse. A frame repeats every 57.5ms.

```pre
SYNC|CH1|CH2|CH3|CH4|CH5|CH6|CH7/CH9|CH8/CH10|TYP1|TYP2|GAP
```

## PCM Signal

A PCM signal looks like this:

```pre
--+            +----------+      +-------+      +-----------+      +-- ...
  |            |          |      |       |      |           |      |
  +------------+          +------+       +------+           +------+

  |<--TLsync-->|<-THsync->|<-TL->|       |<-TL->|
                          |<----TP0----->|<-------TP1------>|<-----TP2- ...
```

The frame starts with a sync low-pulse, which is TLsync=1000µs long, followed by a high-pulse THsync=620µs. All following low-pulses are TL=375µs long, followed by a variable-length high-pulse. Similar to PPM, the bits are encoded by the width of the period TPx (i.e. the time between two falling edges), depicted as TP0, TP1, TP2, ...

## Symbols

To represent data, there are seven periods TPx with different width used, each 140µs apart, resulting in seven different symbols, here specified as S0 through S6:

 TPx [µs] | Symbol
----------|-------
  880     | S0
  1020    | S1
  1160    | S2
  1300    | S3
  1440    | S4
  1580    | S5
  1720    | S6

Each symbol represents two bits, i.e. a value between zero and three. For the four values, a set of four consecutive symbols is used. There are four different sets as shown below.

Symbol | Set A | Set B | Set C | Set D
-------|-------|-------|-------|------
  S0   |   00  |       |       |
  S1   |   01  |   00  |       |
  S2   |   10  |   01  |   00  |
  S3   |   11  |   10  |   01  |  00
  S4   |       |   11  |   10  |  01
  S5   |       |       |   11  |  10
  S6   |       |       |       |  11

The symbol set being used depends on the previous two bits transmitted (of the same channel value):

Previous | Set
---------|----
  None   |  A
  00     |  D
  01     |  C
  10     |  B
  11     |  A

> Note: As a result of the encoding scheme above, each symbol compensates the length of the previous symbol so that the length of a bit-pair becomes constant. For instance, if the first bit-pair is 00 (S0=880µs), the second 00 symbol (S3=1300µs) effectively adds an extra 3*140µs=420µs to the total frame lenght, so that the total length of a bit-pair becomes always 1300µs (except the last bit pair).

## Channel Value

Each channel has an 8-bit value and a 2-bit checksum, which requires five symbols to be transmitted for each channel value. Bits are transmitted MSB first. The channel pulse-width is calculated as `pulse-width = 1050 + 550 * ~value / 0x80`.

## Checksum

The 2-bit checksum that follows each byte is a logical NOT of the XOR of the four 2-bit chunks of the value.

## Frame type

The last four bits in the frame determine the frame type. The frame type determines the channel number of the 7th value and 8th value.

Type  | Value 7 | Value 8
------|---------|--------
11-00 | CH7     | CH8
10-01 | CH9     | CH10

## Examples

The following table shows how channel values translate to five symbols:

Hex | Binary   | Checksum | Symbols
----|----------|----------|--------
 00 | 00000000 |       11 | 03336
 01 | 00000001 |       10 | 03344
 02 | 00000010 |       01 | 03352
 03 | 00000011 |       00 | 03360
 04 | 00000100 |       10 | 03425
 10 | 00010000 |       10 | 04235
 FF | 11111111 |       11 | 33333

## Disclaimer

All product and company names are trademarks or registered trademarks of their respective holders. Use of them does not imply any affiliation with or endorsement by them. The author is not affiliated with Multiplex.

## License

[GNU GPLv3](./LICENSE)

Copyright (C) 2018 Marius Greuel
