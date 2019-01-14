# unsp

## Overview
This repo is a playground for reverse-engineering efforts surrounding systems based on the SunPlus/GeneralPlus μnSP core, which sits at the heart of the SPG24x, SPG28x, PAC300, and other systems-on-a-chip.

Along with surely many as-yet unidentified systems, the μnSP is the core architecture of the following games or systems which have been dumped and are emulated at least partially in MAME:
- Junglesoft "Vii" multi-game console
- Junglesoft "Zone 60" multi-game console
- Junglesoft "Wireless 60" multi-game console
- Hamy / Kids Station Toys "Wireless Hunting Video Game System"
- VTech "V.Smile TV Learning System" console
- LeapFrog "Clickstart My First Computer" console
- JAKKS Pacific Plug & Play video games, including The Batman and Wall-E
- JAKKS Pacific Game-Key-Ready video games, including WWE, Fantastic Four, Justice League, and Dora The Explorer
- Radica Play TV (NTSC) / ConnecTV (PAL) Skateboarder, Football 2, Cricket, and Skannerz TV
- Mattel "Mattel Classic Sports" multi-game console

Known but undumped systems using a μnSP-derived core include the following JAKKS Pacific Game-Key-Ready games:
- Nicktoons (3? game keys available, same as Dora The Explorer)
- SpongeBob SquarePants: The Fry Cook Games (as above)
- Namco Ms. Pac-Man (3 keys available, Dig Dug/New Rally-X, Rally-X/Pac-Man/Bosconian, and Pac-Man/Bosconian)
- Star Wars (1 key available)
- Disney (3? keys available)
- Disney Princess (? keys available)
- Spider-Man (1? key available)

Lastly, the following systems are known to be released but had no Game-Key expansions released for them, although some were in development albeit cancelled:
- Dragon Ball Z
- Capcom 3-in-1
- Scooby-Doo
- Care Bears
- Wheel of Fortune
- Winnie the Pooh

## μnSP Architecture

The μnSP is a bespoke 16-bit architecture, originally reverse-engineered purely from binary dumps by Segher Boessenkool. Vague documentation on the architecture has since surfaced, detailing opcodes available at various revision levels (1.0, 1.1, 1.2, and 2.0) as well as opcode encodings and various capabilities at the relevant revisions. However, details on how individual opcodes function, including how certain status flags (such as C and S) are calculated, are omitted.

For brevity, this section will only lightly cover certain details about revision 1.0 of the architecture.

The μnSP addresses 16-bit words as its fundamental data type. It does not support byte granularity in any fashion, nor does it support any wider accesses. It can address up to 4MWords of data, having a 22-bit address bus. All opcodes are either one or two words in length, with the second word always being the low 16 bits of an address or a 16-bit immediate value.

The μnSP has eight general-purpose registers (GPRs):
- SP (Stack Pointer)
- R1
- R2
- R3
- R4
- BP (Base Pointer)
- SR (Status Register)
- PC (Program Counter)

The μnSP additionally has a hidden 4-bit shift value used by shift and rotate opcodes, as well as two bits used to enable or disable IRQs and fast IRQs (FIQs).

The boot vector is located at address 0xFFF7. The FIQ vector is located at address 0xFFF6. IRQ vectors 0 through 7 are located at addresses 0xFFF8 upward.

The stack pointed to by SP grows downward and is post-decremented on push operations and pre-incremented on pop operations.

The status register, SR, contains, from MSB to LSB: The uppermost 6 bits of the current Data Segment (DS), four status flags (N/Z/S/C), and the uppermost 6 bits of the current Code Segment (CS).

## SunPlus/GeneralPlus SPG24x/SPG28x/PAC300

These are systems-on-a-chip using the μnSP architecture as the core CPU, with 10 kilowords of main RAM mapped at address 0, and varying configurations of peripherals mapped immediately following main RAM.

In all known configurations except the SPG110, which has an as-yet unknown peripheral configuration, peripherals are mapped as follows:
- 2800..28FF: PPU (Picture Processing Unit) registers
- 2900..2AFF: Scroll RAM (used to scroll individual tilemap lines)
- 2B00..2BFF: Palette RAM
- 2C00..2FFF: Sprite RAM (4 words per sprite0
- 3000..37FF: SPU (Sound Processing Unit) registers
- 3D00..3EFF: I/O peripheral registers

Common I/O peripherals include:
- IRQ enable/disable/acknowledge
- Full-Duplex Serial UART
- SPI
- I2C
- SIO (SunPlus-custom 2-wire serial interface)
- 3 General-Purpose I/Os (GPIOs) of varying widths, with optional 'Special' behavior per-bit
- 2 Pseudorandom Number Generators (PRNGs) with settable seeds
- Data Segment Read/Write
- Watchdog
- Sleep-Mode
- External Memory Control
- 2 Variable-Frequency Timers
- 4 Fixed-Frequency Timers
- DMA Transfer Function
- Power Monitoring
- Analog to Digital Converter (ADC)

## μnSP-Based Consoles

This section is for various specific details about consoles and one-off games that use the μnSP architecture.

### VTech V.Smile

The V.Smile supports two controllers which communicate bidirectionally with the main console through a wired connection to the serial UART. They likely have an on-board microcontroller to manage communications, which is as yet unidentified and undumped. The controllers are accessed through GPIO port C. The console can reset the controllers and indicate that it is ready to receive data, as well as transmit data to the controllers. The controllers have one joystick, 8 buttons, and 4 LEDs, the latter of which can be set via commands sent from the console. The joystick can indicate 5 positions per direction, for a total of 11 positions per-axis.

Along with other data, the console periodically transmits one of 16 possible values as a check byte to the controller, which transmits back a transform of the most recent two check bytes. The transform is as follows, as documented by Rebecca G. Bettencourt:

Response = ((A + B + 0xF) & 0xF) ^ 0xB5

In the event that two bytes have not yet been transmitted, a value of 0 is used for A.

Notably, the console has a built-in test mode which can be invoked by having both the On and Off switches set simultaneously on power-up. The test mode will perform a checksum of two different memory ranges, show the current power level as read by the ADC, and show an outline of the controller as well as the state of the buttons and joystick.

Currently, the major stumbling block to the system being marked working in MAME is that a number of games fail to boot entirely, showing nothing but a black screen.

### LeapFrog Clickstart

Controller emulation is the current issue hindering emulation of the console in MAME.

The Clickstart has a keyboard and mouse which communicate with the console over an infrared (IR) link. The mouse is connected physically to the keyboard, which manages communication with the console on behalf of both peripherals. It is currently known that the data sent by the keyboard unit is received by the SoC's serial UART through a 6-byte packet which includes 5 bytes of data and one checksum byte. However, it is yet unknown how the console communicates back to the keyboard unit, if indeed it does so at all.

At present, the emulated console will respond to exactly one keyboard/mouse packet, then stops responding to keyboard/mouse packets for an as yet unknown reason. Like the V.Smile controller issues, the intent of this repository is to provide a public view of any progress made in reverse-engineering the code which manages communicating with the keyboard/mouse unit.
