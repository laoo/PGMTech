# IGS PGM Tech Scroll

* [Overview](#overview)
* [68000 memory map](#68000-memory-map)
* [Z80 memory map](#z80-memory-map)
* [Main CPU ↔ Z80 communication](#main-cpu--z80-communication)
* [Video chip operation](#video-chip-operation)
* [Audio chip operation](#audio-chip-operation)
* [Cartridge pinout](#cartridge-pinout)
* [References](#references)

## Overview

IGS PolyGame Master is a cartridge based system designed for use in an arcade cabinet, with a typical JAMMA pinout.

### Main board

From a programmer's perspective, the important components located on the motherboard are:
* Main CPU; Motorola MC68HC000 CPU clocked at 20 MHz
* Secondary CPU for sound; Zilog Z80 at 8.468 MHz
* Custom video chip; IGS023
* Sound chip; WaveFront ICS2115
* [128 kB of BIOS program ROM (68000)](#internal-bios)
* [2 MB tile graphics ROM](#text--tiles-t-rom)
* 2 MB audio samples data ROM
* [128 kB main CPU work RAM](#main-work-ram)
* [64 kB Z80 work RAM](#z80-ram)
* [video](#video-ram) / [palette RAM](#palette-ram)

### Cartridge

 A cartridge consists of two Printed Circuit Boards.

[Top board](#top-prog-board) contains:
 * 16 bit [`P` ROM](#program-rom) with main CPU program with 23 bit address space (max 16 MB),
 * 16 bit [`T` ROM](#text--tiles-t-rom) with tile graphics with 23 bit address space (max 16 MB),
 * Cartridge-dependent add-ons mapped to main CPU address space,
 * Different custom ASICs for copy protection purposes (to do)

[Bottom board](#bottom-char-board) contains:
 * 8 bit `M` ROM with audio samples data with 24 bit address space (max 16 MB)
 * 16 bit [`B` ROM](#sprite-bitmask-b-rom) with sprite pixel masks and pixel color offsets with 23 bit address space (max 16 MB)
 * 15 bit [`A` ROM](#sprite-color-a-rom) with sprite pixel color data with 25 bit address space (max 64 MB)

### Logical components layout

The main CPU is memory-mapped into its address-space: BIOS, work RAM, video / palette RAM, Z80 interface, Z80 work RAM, I/O registers, external `P` ROM and cartridge-dependent add-ons.

The secondary CPU has access to its work RAM, main CPU interface and sound chip interface.

[The video chip](#video-chip-operation) has access to [video](#video-ram) / [palette](#palette-ram) RAM, internal 2 MB of [tile data ROM](#text--tiles-t-rom), and external [`T`](#text--tiles-t-rom), [`B`](#sprite-bitmask-b-rom) and [`A`](#sprite-color-a-rom) ROMs 

The sound chip has access to internal 2 MB audio samples ROM and external `M` ROM

## 68000 Memory map

| address rage | mirroring | description |
| :-- | :--: | :-- |
| `$000000-$01ffff` | `$0e0000` | [internal BIOS](#internal-bios)
| `$100000-$7fffff` | - | [`P` program ROM](#program-rom) and cartridge-dependent add-ons
| `$700006-$700007` | - | [W/O irq4 ack](#irq4-ack)
| `$800000-$81ffff` | `$0e0000` | [main work RAM](#main-work-ram)
| `$900000-$907fff` | `$0f8000` | [video RAM](#video-ram)
| `$a00000-$a01fff` | `$0fe000` | [palette RAM](#palette-ram)
| `$b00000-$b0ffff` | `$0f0000` | [video registers](#video-registers)
| `$c00000-$c0000f` | `$0e7ff0` | [Z80 interface and RTC regs](#z80-interface-and-rtc-regs)
| `$c08000-$c08007` | `$0e7ff8` | [I/O regs](#io-regs)
| `$c10000-$c1ffff` | `$0e0000` | [Z80 RAM](#z80-ram)
| `$d00000-$ffffff` |- | Cartridge-dependent add-ons

### Internal BIOS

The machine starts up from its internal BIOS. If no cartridge is inserted (or the test button is pressed), the test menu is launched. Without a cartridge it is mirrored to the first 8MB of address space. 

### Program ROM

`P` ROM must start from first 128 entries of Motorola 68000 exception vector table and some mandatory fields.

| offset | size | description |
| --: | :-- | :-- |
| `$000` | `$04` | initial stack pointer |
| `$004` | `$04` | program start address |
| `$070` | `$04` | [irq4](#irq4-ack), configured by cartridge, e.g. coin insertion |
| `$078` | `$04` | [irq6](#irq6-vbl), VBL interrupt |
| `$200` | `$20` | `IGS PGM PLATFORM GAMES\0\0\0\0\0\0\0\0\0\0` |
| `$220` | `$10` | zero padded game name |
| `$230` | `$0a` | version string |
| `$23a` | `$04` | [initialization routine address](#initialization) |
| `$23e` | `$0a` | date |
| `$248` | `$08` | time |

Each word of image file must have its bytes swapped.

### initialization

According to [snake](https://www.arcade-projects.com/threads/pgm-mvs-homebrew.24335/#post-378297) source code:
```
; initialization routine
; very similar to the USER routine in Neo-Geo
fn_initialize_fn
	; same w/py2k2, photoy2k, kov2, dmnfront, probably others
	; link    A6, #$0
	; move.l  D2, -(A7)

	; move.w ($8,A6), D2	; select what this function does...
	; if (D2 == 0) initialize (set up) NV RAM
	; if (D2 == 1) operator setting menu (soft dips)
	; if (D2 == 2) seems to do nothing? (bra.s	lb_skip_initialize)
	; if (D2 == 3) seems to do nothing?	(bra.s	lb_skip_initialize)
	; if (D2 >= 4) skip initialize ?
lb_skip_initialize
	; move.l  (A7)+, D2
	; unlk    A6
	rts
```

### IRQ4 Ack

### IRQ6 VBL

### Main work RAM

Work RAM usage:

| address range | description |
| :-- | :-- |
| `$800000-$8009ff` | buffer for 256 sprites 10 bytes each |
| `$800a00-$81ffff` | general usage |

### Video RAM

| address range | description |
| :-- | :-- |
| `$900000-$900fff` | definition of 64*16 [background layer](#background-tiles-layer), 2 words each tile |
| `$904000-$905fff` | definition of 64*32 [text layer](#foreground-text-layer), 2 words each character |
| `$907000-$9077ff` | [row scroll RAM](#background-tilemap-scrolling) |

### Palette RAM

| address range | description |
| :-- | :-- |
| `$a00000-$a007ff` | 32 * 2 bytes x 32 [sprite](#sprites-layer) [palettes](#palettes) |
| `$a00800-$a00fff` | 32 * 2 bytes x 32 [background](#background-tile-layer) [palettes](#palettes) |
| `$a01000-$a011ff` | 16 * 2 bytes x 32 [text](#foreground-text-layer) [palettes](#palettes) |
| `$a01200-$a01fff` | unused palette RAM |

### Video Registers

| address range | description |
| :-- | :-- |
| `$b00000-$b00fff` | buffer for 256 [sprites](#sprites-layer) 16 bytes each copied by sprite DMA |
| `$b01000-$b0103f` | [zoom table](#scaling), 16 entries * 4 bytes each, W/O. |
| `$b02000-$b02001` | [background](#background-tiles-layer) [scroll up](#background-tilemap-scrolling) |
| `$b03000-$b03001` | [background](#background-tiles-layer) [scroll left](#background-tilemap-scrolling) |
| `$b04000-$b04001` | [BG layer scaling](#b04000-bg-layer-scaling) |
| `$b05000-$b05001` | [text](#foreground-text-layer) scroll up |
| `$b06000-$b06001` | [text](#foreground-text-layer) scroll left |
| `$b07000-$b07001` | [screen](#video-chip-operation) scanline, R/O |
| `$b0e000-$b0e001` | [control flags](#`$b0e000`-control-flags) |

#### `$b0e000` Control flags

```
..dcb.98765432.0
  │││ │││└┤│││ └─ $0001: sprite dma enable - pulse 0->1 to trigger
  │││ │││ │││└─── $0004: irq4 clear to ack, set to enable. Triggered every 62 scanlines (3.968 ms), not synced to VBL
  │││ │││ ││└──── $0008: irq6 clear to ack, set to enable. Triggered each VBL
  │││ │││ │└───── $0010: ? all games set this
  │││ │││ └────── $0060: ? all games except CAVE set this, but seems to serve no purpose
  │││ ││└──────── $0080: ? causes system to lose video synch
  │││ │└───────── $0100: ? shows garbage on screen for all except background
  │││ └────────── $0200: ? disable everything except background layer
  ││└──────────── $0800: disable text layer
  │└───────────── $1000: disable background layer
  └────────────── $2000: disable high priority sprites
```

#### `$b04000` BG layer scaling

```
.....a9876543210
     │└───┤└───┴─ $001f: horizontal scaling from 50% to 200%, 0x10 being 100%
     │    └────── $03e0: vertical scaling from 50% to 200%, 0x200 being 100%
     └─────────── $0400: unknown, set by some games
```

### Z80 interface and RTC regs

| address range | | description |
| :-- | :--: | :-- | 
| `$c00002-$c00003`| R/W | [sound latch 1](#sound-latches); main-CPU write asserts Z80 `/NMI` |
| `$c00004-$c00005`| R/W | [sound latch 2](#sound-latches) |
| `$c00006-$c00007`| R/W | calendar |
| `$c00008-$c00009`| R/W | [Z80 reset](#bus-arbitration): `$a659` halts, `$5050` runs |
| `$c0000a-$c0000b`| R/W | [Z80 bus control](#bus-arbitration): `$45d3` grants the Z80 RAM bus to the main CPU |
| `$c0000c-$c0000d`| R/W | [sound latch 3](#sound-latches) |

### I/O regs

| address range | | description |
| :-- | :--: | :-- | 
| `$c08000-$c00001`| R/O | [Player 1 & 2 controls](#c08000-player-1--2-controls) |
| `$c08002-$c00003`| R/O | [Player 3 & 4 controls](#c08002-player-3--4-controls) |
| `$c08004-$c00005`| R/O | [Extra controls](#c08004-extra-controls) |
| `$c08006-$c00007`| R/O | [Dip switches](#c08006-dip-switches) |

All inputs are active low.

#### `$c08000` Player 1 & 2 controls

```
fedcba9876543210
│││││││││││││││└─ $0001: P1 START
││││││││││││││└── $0002: P1 UP
│││││││││││││└─── $0004: P1 DOWN
││││││││││││└──── $0008: P1 LEFT
│││││││││││└───── $0010: P1 RIGHT
││││││││││└────── $0020: P1 A
│││││││││└─────── $0040: P1 B
││││││││└──────── $0080: P1 C
│││││││└───────── $0100: P2 START
││││││└────────── $0200: P2 UP
│││││└─────────── $0400: P2 DOWN
││││└──────────── $0800: P2 LEFT
│││└───────────── $1000: P2 RIGHT
││└────────────── $2000: P2 A
│└─────────────── $4000: P2 B
└──────────────── $8000: P2 C
```

#### `$c08002` Player 3 & 4 controls

```
fedcba9876543210
│││││││││││││││└─ $0001: P3 START
││││││││││││││└── $0002: P3 UP
│││││││││││││└─── $0004: P3 DOWN
││││││││││││└──── $0008: P3 LEFT
│││││││││││└───── $0010: P3 RIGHT
││││││││││└────── $0020: P3 A
│││││││││└─────── $0040: P3 B
││││││││└──────── $0080: P3 C
│││││││└───────── $0100: P4 START
││││││└────────── $0200: P4 UP
│││││└─────────── $0400: P4 DOWN
││││└──────────── $0800: P4 LEFT
│││└───────────── $1000: P4 RIGHT
││└────────────── $2000: P4 A
│└─────────────── $4000: P4 B
└──────────────── $8000: P4 C
```

#### `$c08004` Extra controls

```
...cba9876543210
   ││││││││││││└─ $0001: P1 COIN
   │││││││││││└── $0002: P2 COIN
   ││││││││││└─── $0004: P3 COIN
   │││││││││└──── $0008: P4 COIN
   ││││││││└───── $0010: P1/2 TEST
   │││││││└────── $0020: P1/2 SERVICE
   ││││││└─────── $0040: P3/4 TEST
   │││││└──────── $0080: P3/4 SERVICE
   ││││└───────── $0100: P1 D
   │││└────────── $0200: P2 D
   ││└─────────── $0400: P3 D
   │└──────────── $0800: P4 D
   └───────────── $1000: RESET
```

#### `$c08006` Dip switches

```
........7..43210
        │  ││││└─ $0001: Test mode (1-ON,0-OFF)
        │  │││└── $0002: Music (0-ON,1-OFF)
        │  ││└─── $0004: Voice (0-ON,1-OFF)
        │  │└──── $0008: Free Play (0-ON,1-OFF)
        │  └───── $0010: Stop Mode (0-ON,1-OFF)
        └──────── $0080: QC mode (1-ON,0-OFF)
```

With the QC mode dip switch set, you can hold A+B when turning on the PGM to access the QC test menu. QC menu options are selected by pressing 1P START rather than 1P A as with the normal operator test menu. You can also hold B+C down in QC mode when turning on the PGM to access cartridge-dependent additional test elements (eg. protection ASIC test, tile ROM test).

## Z80 memory map

The whole Z80 address space is occupied by RAM, that is populated by main CPU.

### Z80 I/O map

| address range | description |
| :-- | :-- |
| `$8000-$8003`| ICS 2115 interface (4 registers) |
| `$8100-$81ff`| [sound latch 3](#sound-latches) |
| `$8200-$82ff`| [sound latch 1](#sound-latches); Z80 read clears `/NMI` |
| `$8400-$84ff`| [sound latch 2](#sound-latches) |

## Main CPU ↔ Z80 communication

The Z80 is a sound coprocessor with no ROM of its own: its entire address space is
RAM that the main CPU populates. Three hardware primitives connect the two CPUs:

* the [shared Z80 RAM window](#shared-z80-ram-window) (plus [bus arbitration](#bus-arbitration)) for uploading code and data,
* three [sound latches](#sound-latches) for short messages,
* the Z80 [NMI and INT](#interrupts) lines for signalling.

Everything above these — command formats, mailboxes, handshakes — is software
convention layered on top of these primitives.

### Shared Z80 RAM window

The main CPU sees the 64 kB of Z80 RAM through the `$c10000-$c1ffff` window (see
[Z80 RAM](#z80-ram)). The window is 16-bit while the Z80 is byte addressed, so a
byte stream written by the main CPU is laid out as:

* **even** Z80 address → **high** byte (`[15:8]`) of the main-CPU word,
* **odd** Z80 address → **low** byte (`[7:0]`).

So when uploading Z80 code or data, byte 0 goes in the high half of the first
word, byte 1 in the low half, byte 2 in the high half of the second word, and so
on. The main CPU can only access this RAM while it owns the bus (see below).

### Bus arbitration

The Z80 RAM is shared, so the main CPU must take the bus from the Z80 before
touching the window and give it back afterwards. Two registers drive this:

| register | value | effect |
| :-- | :-- | :-- |
| `$c00008` Z80 reset | `$a659` | assert Z80 reset (halt) |
| | `$5050` | release Z80 reset (run); also pulses the ICS2115 reset |
| `$c0000a` Z80 bus control | `$45d3` | request the Z80 RAM bus for the main CPU (asserts `BUSRQ`) |
| | other (eg. `$0a0a`) | leave the Z80 running |

Writing `$45d3` to `$c0000a` asserts the Z80 `BUSRQ`. The Z80 finishes its current
machine cycle, tri-states its address / data / control buses and asserts `BUSAK`;
only then does the main CPU own the `$c10000` window. Writing any other value (eg.
`$0a0a`) releases `BUSRQ`; the Z80 reclaims its buses and resumes from the exact
instruction it was paused on — taking the bus this way is non-destructive.

There are two acquisition paths:

* **Z80 held in reset** (`$c00008 = $a659`): the bus is granted immediately,
  because a reset Z80 is not running and cannot acknowledge. Used while uploading
  the driver.
* **Z80 running**: the main CPU must wait for `BUSAK` before the window is valid.
  As there is no ready flag to poll, drivers insert a short fixed delay after
  writing `$45d3`. Used to poke RAM while the sound driver runs.

A typical upload is therefore: assert reset and request the bus → fill / write Z80
RAM → release the bus, then release reset to start the Z80 from `$0000`.

### Sound latches

Three 16-bit latch registers are shared between the main CPU and the Z80. Each is a
single register, readable **and** writable from both sides; the hardware enforces
no direction, so producer / consumer roles are purely software convention. The Z80
sees only the low byte.

| latch | main CPU | Z80 I/O | hardware side effect |
| :-- | :-- | :-- | :-- |
| 1 | `$c00002` | `$8200` | main-CPU **write** asserts Z80 `/NMI`; Z80 **read** clears it |
| 2 | `$c00004` | `$8400` | none |
| 3 | `$c0000c` | `$8100` | none |

Only latch 1 has a side effect, which makes it the natural "doorbell": the main
CPU writes a command (or a token) to `$c00002`, the resulting `/NMI` wakes the Z80,
and the Z80 reads `$8200` to both consume the value and clear the interrupt.
Latches 2 and 3 are plain shared bytes with no signalling.

### Interrupts

The Z80 has two interrupt sources, conventionally used under interrupt mode 1:

| line | vector | source |
| :-- | :-- | :-- |
| `/NMI` | `$0066` | main-CPU write to sound latch 1 (`$c00002`); cleared by Z80 read of `$8200` |
| `/INT` | `$0038` | ICS2115 `IRQ` line |

So the maskable interrupt belongs to the sound chip, while the non-maskable
interrupt belongs to the host. A driver typically handles incoming main-CPU
commands in the `/NMI` handler and ICS2115 events in the `/INT` handler.

## Video chip operation

Video generation is handled by a custom video chip IGS023. It generates 448 x 224 resolution (10 MHz pixel clock) display with standard 4:3 display aspect ratio.

Display is composed of three graphics layers:

1. [Background tiles layer](#background-tiles-layer)
2. [Sprites layer](#sprites-layer)
3. [Foreground text layer](#foreground-text-layer)

Text layer is drawn always on top. Each sprite has a priority setting enabling it to be drawn below or above tiles layer.

### Palettes

Graphics of each layer is palettized with each own individual palette. Each [tile](#background-layer-palette), [text character](#text-layer-palette) or [sprite](#sprites-layer-palette) has 5-bit index to one of 32 palettes. [Background layer](#background-layer-palette) and [sprites layer](#sprites-layer-palette) has 5-bit pixels determining a color of each pixel as a one of 32 colors each, whereas [text layer](#foreground-text-layer) has 4-bit pixels which translates to 16 colors.

 All palette tables has common 15-bit [palette format](#palette-format):

#### Palette format
```
.rrrrrgggggbbbbb
 └───┤└───┤└───┴─ $001f: blue color componentenlarges
     │    └────── $03e0: green color component
     └─────────── $7c00: red color component
```
#### Sprites layer palette

Sprites layer is defined with 5-bit palette index for each sprite with 5-bit color index into each palette for each pixel which gives 32 palettes of 32 16-bit colors entries mapped for the main CPU into the address range `$a00000-$a007ff`.

#### Background layer palette

Background layer is defined with 5-bit palette index for each tile with 5-bit color index into each palette for each pixel which gives 32 palettes of 32 16-bit colors entries mapped for the main CPU into the address range `$a00800-$a00fff`. Last 31th color entry in each palette denotes transparent pixel, as a result each tile pixel can have 31 different colors.

#### Text layer palette

Text layer is defined with 5-bit palette index for each character with 4-bit color index into each palette for each pixel which gives 32 palettes of 16 16-bit colors entries mapped for the main CPU into the address range `$a01000-$a011ff`. Last 15th color entry in each palette denotes transparent pixel, as a result each character pixel can have 15 different colors.

### Background tiles layer

Background tiles layer is displayed unless it is disabled with [control flags register](#`$b0e000`-control-flags).

#### Background tile format

Each tile is a 32x32 square of 5-bit pixels defined in 32 rows of 32 pixels where each row is packed into 20 bytes. Each tile occupies 32*20 = 640 ($280) bytes. Tiles data shares space with text data within [text/tiles `T` ROM](#text--tiles-t-rom).

#### Background tilemap

Tile map is located in the video memory mapped into the main CPU address space range `$900000-$903fff`. It is 16 kB arranged in 64 rows of 64 tiles each, where each tile occupies two words defined as:

```
$0 nnnnnnnnnnnnnnnn 
   └──────────────┴─ $ffff: tile number

$2 ........yxppppp. 
           ││└───┴── $003e: palette number
           │└─────── $0040: x-flip
           └──────── $0080: y-flip
```
where:

- `tile number` multipied by tile size of $280 bytes gives an offset into the [`T` ROM](#text--tiles-t-rom),
- `palette number` multipied by palette size of $40 bytes gives an offset into the [background palette](#background-layer-palette),
- `x-flip` flips the tile horizontally,
- `y-flip` flips the tile vertically.

#### Background tilemap scrolling

To make the background tilemap scroll vertically, edit `$b020000` (word) - increment it to make the layer scroll down, and decrement it to make it scroll up.
To make the background tilemap scroll horizontally, edit `$b03000` (word) - increment it to make the layer scroll to the right, and decrement it to make it scroll to the left.

Rowscroll can be used on the background tilemap: each line can be drawn with a horizontal offset. This is used in games such as Martial Masters (True Lotus Master stage), Espgaluda (Kakusei toggle), or the BIOS introduction itself (see below).
In the range `$907000-$9077ff`, each word controls the offset of one line, for a total of 512 lines from top to bottom. Increment an offset to shift it to the right, and decrement it to shift it to the left.

NOTE: The BIOS introduction screen writes $0010 in the `$9070c0-$9070ff` range in order to offset the "PolyGame Master" text 16 pixels to the right. As such, make sure to write $0000 to this range in order to avoid unexpected background tilemap rendering.

### Sprites layer

There are a maximum of 256 sprites on the PGM and they are copied via DMA from the first 2560 bytes of work RAM (10 bytes per sprite) to the internal sprite registers every frame. The sprite definition consists of 5 words of packed bits per sprite stored sequentially in memory:

```
$0 mttttxxxxxxxxxxx 
   |└──┤└─────────┴─ $07FF: X position (11 bit signed)
   |   └──────────── $7800: Horizontal Zoom/Shrink table select
   └──────────────── $8000: Horizontal Zoom/Shrink mode select

$2 mtttt.yyyyyyyyyy 
   |└──┤ └────────┴─ $07FF: Y position (10 bit signed)
   |   └──────────── $7800: Vertical Zoom/Shrink table select
   └──────────────── $8000: Vertical Zoom/Shrink mode select

$4 .vhpppppmxxxxxxx
    ||└───┤|└─────┴─ $007F: Sprite mask B ROM address MSB
    ||    |└──────── $0080: Priority mode (Over(0) or Under(1) background)
    ||    └───────── $1f00: Palette number
    |└────────────── $2000: Horizontal flip
    └─────────────── $4000: Vertical flip

$6 xxxxxxxxxxxxxxxx
   └──────────────┴─ $ffff: Sprite mask B ROM address LSB

$8 .wwwwwwhhhhhhhhh*
    └────┤└───────┴─ $01ff: Sprite height
         └────────── $7e00: Sprite width (in 16 pixel units)
```

- Last sprite entry marker is `$8 = .000000000000000`.

#### Scaling

`t`-bits select an entry in an internal zoom table while `m` selects the operation mode as _grow_ when set and _shrink_ otherwise.

The zoom tables which are used for scaling the sprites have been determined by analysing bus access on the mask ROM bus:
```
   Normal   Flipped
 0 AAAAAAAA AAAAAAAA
 1 A8AAAAAA AAAAAA2A
 2 A8AAA8AA AA2AAA2A
 3 A8A8A8AA AA2A2A2A
 4 A8A8A8A8 2A2A2A2A
 5 88A8A8A8 2A2A2A22
 6 88A888A8 2A222A22
 7 888888A8 2A222222
 8 88888888 22222222
 9 80888888 22222202
10 80888088 22022202
11 80808088 22020202
12 80808080 02020202
13 80008080 02020002
14 80008000 00020002
15 00008000 00020000
16 00000000 00000000
17 00010000 00010000
18 00010001 00010001
19 01010001 00010101
20 01010101 01010101
21 01110101 01011101
22 01110111 11011101
23 11110111 11011111
24 11111111 11111111
25 11511111 11111511
26 11511151 15111511
27 51511151 15111515
28 51515151 15151515
29 51555151 15155515
30 51555155 55155515
31 55555155 55155555
```
The 5 bit number made from `m` as the high bit and `t` as the low bits is the index into the above table.

For unflipped sprites sample each bit in the table entry from 31 down to 0 circularly for each line rendered. When in _shrink_ mode the sprite line after the current line will be skipped if the bit is set and in _grow_ mode the current line will be duplicated if the bit is set. If not set the next line is rendered as normal.

Vertically flipped sprites start sampling the zoom table entry at the bit number given by the low 5 bits of the sprite height. If the height is 32, for example, zoom table sampling will start at bit 0 and continue as for normal sprites above.

Vertically flipped sprites terminate one line early in _shrink_ mode if the zoom bit is set on the last line. This only happens for vertically flipped sprites, unflipped sprites render the last line as expected.

### Foreground text layer

The foreground text layer is logically the same as the background layer but the tile (or character) size is 8x8 pixels and limited to 16 colours per tile. It can be too disabled with [control flags register](#`$b0e000`-control-flags).

#### Character tilemap

Character tile map is located in the video memory mapped into the main CPU address space range `$904000-$905fff`. It is 8 kB arranged in 64 rows of 32 tiles each.

#### Character tile format

Each tile is a 8x8 pixels in size and comprises of 8 rows of 8 pixels packed into 4 bytes each row stored contiguously in memory. Each nibble represents a palette index into the palette specified by the text layer tile entry, the lowest order nibble is displayed first on screen (the leftmost pixel).

```
bbbbaaaa 
└──┤└──┴─ $0f: leftmost pixel
   └───── $f0: rightmost pixel
```

Each tile occupies 8*4 = 32 bytes. Text layer tile data shares space with background tile data within [text/tiles `T` ROM](#text--tiles-t-rom). The address of the tile within the T ROM is calculated as the tile number (0-$ffff) multiplied by 32. This gives an effective address space of 2MB (0-$1FFFFF) for text layer tiles.

### Text / tiles `T` ROM

The T ROM on the top (PROG) PCB can hold upto 16MB of data which is accessed by the PGM as 8Mx16. Both background and text tile data are stored in the T ROM.

### Sprite Data ROMs

Sprites are split accross two different ROMs on the bottom (CHAR) PCB. The B ROM holds the address of the colour data and the transparency information for the sprite while the A ROM holds the palette index colour 
information. Information about the sprites size, location, palette and the address of the sprite data in the B ROM are part of the sprite definition. A combination of this information and the data in the A and B ROMs are used to render the sprite.

#### Bitmask `B` ROM

The first long-word read by the sprite engine (given by the sprite entry) from the B ROM is the address of the colour data from the A ROM. Following this is a bitmask defining the visibility (1) or transparency (0) of each pixel in the sprite. The sprite engine reads the B ROM pixel visibility sequentially, one word at a time (sprites are a multiple of 16 pixels wide), and processes the bits from LSB to MSB. Therefore the LSB of each word of visibility information is the leftmost pixel processed and the MSB is the rightmost.

The opacity bitmap is followed by the long-word with the address of the end of the colour data of given sprite. It enables rendering the sprite flipped vertically - the sprite engine processes the entry in B ROM backwards from the end (the end can be found knowing the dimensions of the sprite) drawing the sprite from right to left.

#### Sprite color `A` ROM

Sprite colour information is stored as 5 bits per pixel and packed into 3 pixels per word. Colour information is read sequentially when drawing sprites and is also processed from LSB to MSB.

```
.cccccbbbbbaaaaa 
 └───┤└───┤└───┴─ $001f: palette index (leftmost pixel)
     |    └────── $03e0: palette index
     └─────────── $7c00: palette index (rightmost pixel)
```

Bit 15 of the A ROM is physically unconnected on the CHAR PCB.

## Audio chip operation

## Cartridge pinout

### Top PROG board

#### Left

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 | +5V | +5V | 32 |
| 30 | PA14_OUT | TA13 | 33 |
| 29 | ? | TA12 | 34 |
| 28 | ? | TA11 | 35 |
| 27 | PA_13 | TA10 | 36 |
| 26 | ? | TA9 | 37 |
| 25 | PA12_OUT | TA8 | 38 |
| 24 | PA11_OUT | TA7 | 39 |
| 23 | PA10_OUT | TA6 | 40 |
| 22 | PA9_OUT | TA5 | 41 |
| 21 | PA8_OUT | TA4 | 42 |
| 20 | TD0 | TA3 | 43 |
| 19 | TD1 | TA2 | 44 |
| 18 | TD2 | TA1 | 45 |
| 17 | TD3 | TA0 | 46 |
| 16 | TD4 | TA22 | 47 |
| 15 | TD5 | TA21 | 48 |
| 14 | PGM_TILE_EN# | TA20 | 49 |
| 13 | TD6 | TA19 | 50 |
| 12 | TD7 | TA18 | 51 |
| 11 | TD8 | TA17 | 52 |
| 10 | TD9 | TA16 | 53 |
| 9 | TILE_CS# | TA15 | 54 |
| 8 | TD15 | TA14 | 55 |
| 7 | TD14 | GND | 56 |
| 6 | TD13 | GND | 57 |
| 5 | TD12 | GND | 58 |
| 4 | TD11 | GND | 59 |
| 3 | TD10 | GND | 60 |
| 2 | ? | CLK20 | 61 |
| 1 | GND | GND | 62 |

| signal | notes |
| :-- | :-- |
| PGM_TILE_EN# | Driven low to enable tile output from the PGM onboard tile ROM (default), driven high when cart is outputting tile data (TD15-0) |
| TILE_CS# | Enables tile output on KOVSH cart, pulled down and not used on other games |
| CLK20 | 20Mhz system clock |

#### Right

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 | +5V | +5V | 32 |
| 30 | AS# | PA23 | 33 |
| 29 | ? | PA22 | 34 |
| 28 | BLANK# | PA21 | 35 |
| 27 | PGM_PRG_EN# | PA20 | 36 |
| 26 | WR# | PA19 | 37 |
| 25 | ? | PA18 | 38 |
| 24 | RESET# | PA17 | 39 |
| 23 | RD# | PA16 | 40 |
| 22 | ? | PA15 | 41 |
| 21 | ? | PA14 | 42 |
| 20 | PD15 | PA13 | 43 |
| 19 | PD14 | PA12 | 44 |
| 18 | PD0 | PA11 | 45 |
| 17 | PD1 | PA10 | 46 |
| 16 | PD2 | PA9 | 47 |
| 15 | PD3 | PA8 | 48 |
| 14 | PD4 | PA7 | 49 |
| 13 | PD5 | PA6 | 50 |
| 12 | PD6 | PA5 | 51 |
| 11 | PD7 | PA4 | 52 |
| 10 | PD8 | PA1 | 53 |
| 9 | PD13 | PA2 | 54 |
| 8 | PD12 | PA3 | 55 |
| 7 | PD11 | U8 | 56 |
| 6 | PD10 | +5V | 57 |
| 5 | PD9 | +5V | 58 |
| 4 | ? | +5V | 59 |
| 3 | PA2_OUT | +5V | 60 |
| 2 | PA1_OUT | +5V | 61 |
| 1 | GND | +5V | 62 |

| signal | notes |
| :-- | :-- |
| AS# | 68000 address strobe |
| PGM_PRG_EN# | Driven low to enable program data output from the PGM onboard BIOS ROM (default), driven high when cart is outputting program data (PD15-0) |
| WR# | Driven low when 68000 is writing to the bus |
| RESET# | Driven low when system is in reset, connected to reset switch on PGM motherboard |
| RD# | Driven low when 68000 is reading from the bus |
| BLANK# | Driven low at the beginning of each renderered line (?) and during vblank. 224 short low bursts and 1 long per frame. Sprite DMA happens at line 221.  |

### Bottom CHAR board

#### Left

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 | +5V | +5V | 32 |
| 30 | BA17 | BA14 | 33 |
| 29 | BA20 | BA16 | 34 |
| 28 | BA19 | BA15 | 35 |
| 27 | BA22 | BA18 | 36 |
| 26 | BA21 | +5V | 37 |
| 25 | AA8 | BD15 | 38 |
| 24 | AA22 | BD14 | 39 |
| 23 | AA9 | AA7 | 40 |
| 22 | AA23 | AA6 | 41 |
| 21 | AA10 | AA5 | 42 |
| 20 | AA24 | AA4 | 43 |
| 19 | AA14 | AA3 | 44 |
| 18 | AA15 | AA2 | 45 |
| 17 | BD0 | AA1 | 46 |
| 16 | BD1 | AA0 | 47 |
| 15 | BD2 | +5V | 48 |
| 14 | BD3 | BA0 | 49 |
| 13 | BD4 | BA1 | 50 |
| 12 | BD5 | BA2 | 51 |
| 11 | BD6 | BA3 | 52 |
| 10 | BD7 | BA4 | 53 |
| 9 | BD8 | BA5 | 54 |
| 8 | BD13 | BA7 | 55 |
| 7 | GND | BA8 | 56 |
| 6 | BD12 | BA9 | 57 |
| 5 | BD11 | BA10 | 58 |
| 4 | BD10 | BA11 | 59 |
| 3 | BD9 | BA12 | 60 |
| 2 | BA6 | BA13 | 61 |
| 1 | GND | GND | 62 |

#### Right

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 | +5V | +5V | 32 |
| 30 | AA17 | MA9 | 33 |
| 29 | AA16 | MA10 | 34 |
| 28 | AA19 | MA11 | 35 |
| 27 | AA18 | GND | 36 |
| 26 | AA21 | MD7 | 37 |
| 25 | AA20 | MD6 | 38 |
| 24 | MD5 | MA12 | 39 |
| 23 | MD4 | MA13 | 40 |
| 22 | MD3 | MA14 | 41 |
| 21 | MD2 | MA15 | 42 |
| 20 | MD1 | MA16 | 43 |
| 19 | MD0 | MA17 | 44 |
| 18 | MA0 | MA18 | 45 |
| 17 | MA1 | MA19 | 46 |
| 16 | MA2 | MA20 | 47 |
| 15 | MA3 | MA23 | 48 |
| 14 | MA4 | MA22 | 49 |
| 13 | MA5 | MA21 | 50 |
| 12 | AD7 | MA6 | 51 |
| 11 | AD6 | MA8 | 52 |
| 10 | AD5 | MA7 | 53 |
| 9 | AD4 | AD14 | 54 |
| 8 | AD0 | AD13 | 55 |
| 7 | AD1 | AD12 | 56 |
| 6 | AD2 | AD11 | 57 |
| 5 | AD3 | AD10 | 58 |
| 4 | AA11 | AD9 | 59 |
| 3 | AA12 | AD8 | 60 |
| 2 | AA13 | INT_M_ROM_OE | 61 |
| 1 | GND | GND | 62 |

## References
* http://www.igspgm.com/
* http://www.igspgm.com/repairs/tech.htm
* http://www.igspgm.com/iq132/data1.htm
* https://github.com/mamedev/mame/tree/master/src/mame/igs
* https://github.com/finalburnneo/FBNeo/tree/master/src/burn/drv/pgm
* https://github.com/wickerwaka/Arcade-IGSPGM_MiSTer
* https://www.arcade-projects.com/threads/pgm-cartridge-pinout.13847/
* https://www.arcade-projects.com/threads/pgm-mvs-homebrew.24335/


