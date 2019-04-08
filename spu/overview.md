Please feel free to make any changes to this document!

# Overview
The SPG2xx Sound Processing Unit (SPU) contains 16 channels (or voices).

Each channel has the following features:
* Support for 8-bit PCM, 16-bit PCM, and ADPCM formats.
* Supports one-shot, manual loop, and software-controlled PCM modes.
* Hardware envelope macro feature. Has a configurable address, length, and loopstart.
* Envelope clock rate control.
* Stereo panning feature. Range from 0 (left) to 127 (right).
* 19-bit frequency and phase accumulator registers. The supported sampling rate range is ~0.536 Hz to 281.25 KHz.
* Hardware equalizer (4 bands) for all channels.

# Memory Map
For the 3000..33FF range of the memory map, the next channel's address is formed by adding 10h to the current channel's address.

The SPU has the following per-channel registers, named accordingly from a [MAME source file](https://github.com/mamedev/mame/blob/master/src/devices/machine/spg2xx_audio.h):

| Address | 16-bit Value Name  | Bit    | Description                                     |
| ------- | ------------------ | ------ | ----------------------------------------------- |
| 3000    | WAVE_ADDR          | 0..15  | The lower 16 bits of the start block's address. |
| 3001    | MODE               | 0..5   | The upper 6 bits of the start block's address.  |
| 3001    | MODE               | 6..11  | The upper 6 bits of the loop block's address.   |
| 3001    | MODE               | 12..13 | 0: Software PCM mode.                           |
| 3001    | MODE               | 12..13 | 1: One-shot PCM mode.                           |
| 3001    | MODE               | 12..13 | 2: Manual-loop PCM mode.                        |
| 3001    | MODE               | 12..13 | 3: ???                                          |
| 3001    | MODE               | 14..15 | 0: 8-bit PCM mode.                              |
| 3001    | MODE               | 14..15 | 1: 16-bit PCM mode.                             |
| 3001    | MODE               | 14..15 | 2: ADPCM mode.                                  |
| 3001    | MODE               | 14..15 | 3: ???                                          |
| 3002    | LOOP_ADDR          | 0..15  | The lower 16 bits of the loop block's address.  |
| 3003    | PAN_VOL            | 0..6   | The main channel volume, scaled linearly.       |
| 3003    | PAN_VOL            | 8..14  | The panning value. 64 = center, 0 = far left?   |
| 3004    | ENVELOPE0          | 0..6   | Envelope table index increase value per tick??? |
| 3004    | ENVELOPE0          | 7      | Sign to bits 0..6.                              |
| 3004    | ENVELOPE0          | 8..14  | Loopstart value???                              |
| 3004    | ENVELOPE0          | 15     | Enables envelope looping???                     |
| 3005    | ENVELOPE_DATA      | 0..6   | Envelope Direct Data                            |
| 3005    | ENVELOPE_DATA      | 8..15  | Data count. Number of envelope steps???         |
| 3006    | ENVELOPE1          | 0..7   | Load value. What's that???                      |
| 3006    | ENVELOPE1          | 8      | Enables envelope repeat.                        |
| 3006    | ENVELOPE1          | 9..15  | Repeat count. 0 = infinite???                   |
| 3007    | ENVELOPE_ADDR_HIGH | 0..5   | The upper 6 bits of the envelope's address.     |
| 3007    | ENVELOPE_ADDR_HIGH | 6      | Enables an IRQ.                                 |
| 3007    | ENVELOPE_ADDR_HIGH | 7..15  | IRQ address. What's the formula???              |
| 3008    | ENVELOPE_ADDR      | 0..15  | The lower 16 bits of the envelope's address.    |
| 3009    | WAVE_DATA_PREV     | 0..15  | The previous wavetable sample that was played.  |
| 300A    | ENVELOPE_LOOP_CTRL | 0..8   | Envelope address offset.                        |
| 300A    | ENVELOPE_LOOP_CTRL | 9..15  | Rampdown offset.                                |
| 300B    | WAVE_DATA          | 0..15  | The current wavetable sample that's played.     |
| 300C    | ADPCM_SEL          | 9..14  | Point number. What's that???                    |
| 300C    | ADPCM_SEL          | 15     | 0: 32 Kbps codec, 1: 36 Kbps codec              |
| 3200    | PHASE_HIGH         | 0..2   | The upper 3 bits of the frequency register.     |
| 3201    | PHASE_ACCUM_HIGH   | 0..2   | The upper 3 bits of the phase accumulator.      |
| 3202    | TARGET_PHASE_HIGH  | 0..2   | The upper 3 bits of the target phase.           |
| 3203    | RAMP_DOWN_CLOCK    | 0..2   | *Hz* = max(4×2^(*value*×2),8192)×13 ???         |
| 3204    | PHASE              | 0..15  | The lower 16 bits of the frequency register.    |
| 3205    | PHASE_ACCUM        | 0..15  | The lower 16 bits of the phase accumulator.     |
| 3206    | TARGET_PHASE       | 0..15  | The lower 16 bits of the target phase.          |
| 3207    | PHASE_CTRL         | 0..11  | The phase offset.                               |
| 3207    | PHASE_SIGN         | 12     | The sign to bits 0..11???                       |
| 3207    | PHASE_TIME_STEP    | 13..15 | The phase time step.                            |
