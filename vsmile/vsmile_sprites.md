# V.Smile Sprite Issues

This document contains observations and information about the issues affecting sprite updates with the V.Smile driver in MAME.

## Overview

The primary issue preventing the V.Smile driver from being marked as working in MAME is that sprite updates do not occur at the correct rate.

This document will focus on the UK version of Kung Fu Panda (kungfupuk in MAME's V.Smile softlist) due to the availability on YouTube of reference video from real hardware.

The video being used as reference can be found at the following link, as of 20 January 2019: https://www.youtube.com/watch?v=Sh0HZDOmESQ

While the timing of the animations themselves are correct between MAME and real hardware, they appear to be updating at a rate of approximately once every 8 frames in MAME, as opposed to each frame on real hardware. This is particularly noticeable with the large shadow of Po on the first intro screen: It slides smoothly to the right on hardware, but is only updated at X values of 0xFFBE, 0xFFC2, 0xFFCA, and 0xFFD1 when run in MAME.

## Details

The game in question builds its sprite list at address 0x12DA in main RAM, and periodically transfers it via a 0x400-word-long sprite DMA transfer into sprite RAM, which is located at 0x2C00. In particular, the upper-left portion of the large shadow sprite used during the entry is sprite index 45 (decimal, 0-indexed). Its updates can thus be tracked in MAME's debugger by setting a watchpoint on addresses 0x138F and 0x2CB5 using the following commands:
```
wpset 138f,1,w
wpset 2cb5,1,w
```

The sprite list appears to be built by a fairly long routine between addresses 0x03D7B4 and 0x03D9A5 in ROM. The specific write which stores the X value for the sprite in question is located at address 0x03D96A in ROM. The routine is in turn called at address 0x058740 by an outer routine located between addresses 0x058701 and 0x058745 in ROM. Neither routine has been examined in any great detail, beyond determining the addresses mentioned.

The routine which transfers the sprite list via sprite DMA is located from 0x064AED to 0x064AFD in ROM and is perfectly straightforward:
```
064AED: D288      push r1, r1 to [sp]
064AEE: 9309 12DA r1 = 12da
064AF0: D319 2870 [2870] = r1
064AF2: 9240      r1 = 00
064AF3: D319 2871 [2871] = r1
064AF5: 9309 0400 r1 = 0400
064AF7: D319 2872 [2872] = r1
064AF9: 9311 2872 r1 = [2872]
064AFB: 4E43      jne 64af9
064AFC: 9088      pop r1, r1 from [sp]
064AFD: 9A90      retf
```

This corresponds to a pseudo-C routine of:
```
#define SPDMA_SRC *(unsigned short)0x2870
#define SPDMA_DST *(unsigned short)0x2871
#define SPDMA_LEN *(unsigned short)0x2872
void do_sprite_dma()
{
    SPDMA_SRC = 0x12DA;
    SPDMA_DST = 0;
    SPDMA_LEN = 0x0400;
    while (SPDMA_LEN != 0);
}
```

This routine is called at address 0x024E4B in ROM, in a routine located between addresses 0x024E26 and 0x024E92 in ROM.

In principle, were the sprite DMA to be performed with each update to the sprite list in main RAM, it should be observed that the routine at 0x024E26 is called after each update to the sprite list. However, by using the MAME debugger to place a breakpoint at that address with the following command while having the previously-mentioned data watchpoints set, it is clear that that is not the case:
```
bpset 24e26
```

Indeed, one can observe multiple updates to the sprite's X position in the main-RAM sprite list with no corresponding call to the routine which invokes the sprite DMA transfer:
```
>g
Stopped at watchpoint 1 writing 0020 to 0000138F (PC=6FCA9)
>g
Stopped at watchpoint 1 writing 0062 to 0000138F (PC=3D916)
>g
Stopped at watchpoint 1 writing FFC2 to 0000138F (PC=3D96B)
>g
Stopped at watchpoint 1 writing 0020 to 0000138F (PC=6FCA9)
>g
Stopped at watchpoint 1 writing 0063 to 0000138F (PC=3D916)
>g
Stopped at watchpoint 1 writing FFC3 to 0000138F (PC=3D96B)
>g
Stopped at watchpoint 1 writing 0020 to 0000138F (PC=6FCA9)
>g
Stopped at watchpoint 1 writing 0064 to 0000138F (PC=3D916)
>g
Stopped at watchpoint 1 writing FFC4 to 0000138F (PC=3D96B)
```

The overall structure of the code which results in the sprite DMA being called is as follows. Indents indicate calls.
```
0x060000
    0x06FCF7
    0x06FD03
    ...
    0x01EF86
        0x024FBE
        0x024FC5
        ...
        0x023051
            Too many to list for now
        0x023B5A
        ...
        0x00A844->0x06FC74
        0x06FC7B
        ...
        0x06FDFE
        0x06FE0E
        ...
        0x050AEF
        0x050AFE
        ...
        0x06FE0F
        0x06FE1D
            0x06FDDE
            0x06FDE7/9/B/D/F
        0x06FE28
    0x01EFA6
    ...
    0x03E8E2
        0x06FD04
        0x06FD11
        ...
        0x06FC98
        0x06FCAC
        ...
        0x03D653
        0x03D6C5
        ...
        0x06FC98
        0x06FCAC
        ...
        0x06FC8A
        0x06FC97
        ...
        0x06FCF7
        0x06FD03
        ...
        0x00A85C->0x06FEC9
        0x06FED8
        ...
        0x03E7C0
            0x06FD04
            0x06FD11
            ...
            0x06FD04
            0x06FD11
            ...
            0x06FD04
            0x06FD11
        0x03E882
        ...
        0x03E499
            0x00A85A->0x06FEBB
            0x06FEC8
            ...
            0x00A85C->0x06FEC9
            0x06FED8
            ...
        0x03E520
    0x03EA92
    ...
    0x03EB17
        0x06FD04
        0x06FD11
        ...
        0x06FD04
        0x06FD11
        ...
        0x00F9BD
            0x00A854->0x0090C5
            0x0090C8
            ...
            0x00B7E0
            0x00B7F4
            ...
            0x06FDFE
            0x06FE0E
            ...
            0x06FE0F
            0x06FE1D
                0x06FDDE
                0x06FDE7/9/B/D/F
            0x06FE28
        0x00FA87
    0x03EBE3
0x0600B1
```

It is as yet unknown where the routine at address 0x060000 is called.
