# IGS PGM Tech Scroll

* [Overview](#overview)
* [68000 memory map](#68000-memory-map)
* [Z80 memory map](#z80-memory-map)
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
 * Different custom ASICs for copy protection purposes (to do)

[Bottom board](#bottom-char-board) contains:
 * 8 bit `M` ROM with audio samples data with 24 bit address space (max 16 MB)
 * 16 bit [`B` ROM](#sprite-bitmask-b-rom) with sprite pixel masks and pixel color offsets with 23 bit address space (max 16 MB)
 * 15 bit [`A` ROM](#sprite-color-a-rom) with sprite pixel color data with 25 bit address space (max 64 MB)

### Logical components layout

The main CPU is memory-mapped into its address-space: BIOS, work RAM, video / palette RAM, Z80 interface, Z80 work RAM, I/O registers and external `P` ROM.

The secondary CPU has access to its work RAM, main CPU interface and sound chip interface.

[The video chip](#video-chip-operation) has access to [video](#video-ram) / [palette](#palette-ram) RAM, internal 2 MB of [tile data ROM](#text--tiles-t-rom), and external [`T`](#text--tiles-t-rom), [`B`](#sprite-bitmask-b-rom) and [`A`](#sprite-color-a-rom) ROMs 

The sound chip has access to internal 2 MB audio samples ROM and external `M` ROM

## 68000 Memory map

| address rage | mirroring | description |
| :-- | :--: | :-- |
| `$000000-$01ffff` | `$0e0000` | [internal BIOS](#internal-bios)
| `$100000-$7fffff` | - | [`P` program ROM](#program-rom)
| `$700006-$700007` | - | [W/O irq4 ack](#irq4-ack)
| `$800000-$81ffff` | `$0e0000` | [main work RAM](#main-work-ram)
| `$900000-$907fff` | `$0f8000` | [video RAM](#video-ram)
| `$a00000-$a01fff` | `$0fe000` | [palette RAM](#palette-ram)
| `$b00000-$b0ffff` | `$0f0000` | [video registers](#video-registers)
| `$c00000-$c0000f` | `$0e7ff0` | [Z80 interface and RTC regs](#z80-interface-and-rtc-regs)
| `$c08000-$c08007` | `$0e7ff8` | [I/O regs](#io-regs)
| `$c10000-$c1ffff` | `$0e0000` | [Z80 RAM](#z80-ram)
| `$d00000-$ffffff` |- | [`P` program ROM](#program-rom) ?

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
| `$900000-$903fff` | definition of 64*64 [background layer](#background-tiles-layer), 4 bytes each tile |
| `$904000-$905fff` | definition of 64*32 [text layer](#foreground-text-layer), 4 bytes each character |
| `$907000-$9077ff` | row scroll RAM |

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
| `$b01000-$b0103f` | zoom table, 16 entries * 4 bytes each, W/O |
| `$b02000-$b02001` | [background](#background-tiles-layer) scroll up |
| `$b03000-$b03001` | [background](#background-tiles-layer) scroll left |
| `$b04000-$b04001` | zoom flags ? |
| `$b05000-$b05001` | [text](#foreground-text-layer) scroll up |
| `$b06000-$b05001` | [text](#foreground-text-layer) scroll left |
| `$b07000-$b07001` | [screen](#video-chip-operation) scanline, R/O |
| `$b06000-$b06001` | [control flags](#b06000-control-flags) |

#### `$b06000` Control flags

```
..dcb.98765432.0
  │││ ││││││││ └─ $0001: sprite dma enable - pulse 0->1 to trigger
  │││ │││││││└─── $0004: irq4 ack - pulse 0->1 to trigger
  │││ ││││││└──── $0008: iqr6 ack - pulse 0->1 to trigger
  │││ │││││└───── $0010: ? all games set this
  │││ │││└┴────── $0060: ? all games except CAVE set this, but seems to serve no purpose
  │││ ││└──────── $0080: ? causes system to lose video synch
  │││ │└───────── $0100: ? shows garbage on screen for all except background
  │││ └────────── $0200: ? disable everything except background layer
  ││└──────────── $0800: disable text layer
  │└───────────── $1000: disable background layer
  └────────────── $2000: disable high priority sprites
```
### Z80 interface and RTC regs

| address range | | description |
| :-- | :--: | :-- | 
| `$c00002-$c00003`| R/W | sound latch 1 |
| `$c00004-$c00005`| R/W | sound latch 2 |
| `$c00006-$c00007`| R/W | calendar |
| `$c00008-$c00009`| W/O | Z80 reset |
| `$c0000a-$c0000b`| W/O | Z80 control |
| `$c0000c-$c0000d`| R/W | sound latch 3 |

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

### Z80 RAM

The range `$c10000-$c1ffff` maps to the Z80 RAM. The Z80 executes code from this 64 kB area, which can be uploaded by the 68000 as needed (eg. load additional sequence data)

## Z80 memory map

The whole Z80 address space is occupied by RAM, that is populated by main CPU.

### Z80 I/O map

| address range | description |
| :-- | :-- |
| `$8000-$8003`| ICS 2115 Interface |
| `$8100-$81ff`| sound latch 3 |
| `$8200-$82ff`| sound latch 1 |
| `$8400-$84ff`| sound latch 2 |

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
 ││││││││││└┴┴┴┴─ $001f: blue color component
 │││││└┴┴┴┴────── $03e0: green color component
 └┴┴┴┴─────────── $7c00: red color component
```
#### Sprites layer palette

Sprites layer is defined with 5-bit palette index for each sprite with 5-bit color index into each palette for each pixel which gives 32 palettes of 32 16-bit colors entries mapped for the main CPU into the address range `$a00000-$a007ff`.

#### Background layer palette

Background layer is defined with 5-bit palette index for each tile with 5-bit color index into each palette for each pixel which gives 32 palettes of 32 16-bit colors entries mapped for the main CPU into the address range `$a00800-$a00fff`. Last 31th color entry in each palette denotes transparent pixel, as a result each tile pixel can have 31 different colors.

#### Text layer palette

Text layer is defined with 5-bit palette index for each character with 4-bit color index into each palette for each pixel which gives 32 palettes of 16 16-bit colors entries mapped for the main CPU into the address range `$a01000-$a011ff`. Last 15th color entry in each palette denotes transparent pixel, as a result each character pixel can have 15 different colors.

### Background tiles layer

Background tiles layer is displayed unless it is disabled with [control flags register](#b06000-control-flags).

#### Tile format

Each tiles is a 32x32 square of 5-bit pixels defined in 32 rows of 32 pixels where each row is packed into 20 bytes. Each tile occupies 32*20 = 640 ($280) bytes. Tiles data shares space with text data within [text/tiles `T` ROM](#text--tiles-t-rom).

#### Tile map

Tile map is located in the video memory mapped into the main CPU address space range `$900000-$903fff`. It is 16 kB arranged in 64 rows of 64 tiles each, where each [tile](#tile-map-entry-format) occupies 32-bits defined as:

#### Tile map entry format
```
........yxppppp. nnnnnnnnnnnnnnnn
        │││││││  └┴┴┴┴┴┴┴┴┴┴┴┴┴┴┴─ $0000ffff: tile number
        ││└┴┴┴┴─────────────────── $003e0000: palette number
        │└──────────────────────── $00400000: x-flip
        └───────────────────────── $00800000: y-flip
```
where:

- `tile number` multipied by tile size of $280 bytes gives an offset into the [`T` ROM](#text--tiles-t-rom),
- `palette number` multipied by palette size of $40 bytes gives an offset into the [background palette](#background-layer-palette),
- `x-flip` flips the tile horizontally,
- `y-flip` fpips the tile vertivally.

### Sprites layer

### Foreground text layer

### Text / tiles `T` ROM

### Sprite bitmask `B` ROM

### Sprite color `A` ROM

## Audio chip operation

## Cartridge pinout

### Top PROG board

#### Left

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 | +5V | +5V | 32 |
| 30 | PA14_OUT | TA13 | 33 |
| 29 | U12 | TA12 | 34 |
| 28 | U11 | TA11 | 35 |
| 27 | PA_13 | TA10 | 36 |
| 26 | U10 | TA9 | 37 |
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
| 14 | INT_T_ROM_OE | TA20 | 49 |
| 13 | TD6 | TA19 | 50 |
| 12 | TD7 | TA18 | 51 |
| 11 | TD8 | TA17 | 52 |
| 10 | TD9 | TA16 | 53 |
| 9 | CART_CS | TA15 | 54 |
| 8 | TD15 | TA14 | 55 |
| 7 | TD14 | GND | 56 |
| 6 | TD13 | GND | 57 |
| 5 | TD12 | GND | 58 |
| 4 | TD11 | GND | 59 |
| 3 | TD10 | GND | 60 |
| 2 | U8 | CLK | 61 |
| 1 | GND | GND | 62 |

#### Right

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 | +5V | +5V | 32 |
| 30 | AS | PA23 | 33 |
| 29 | U7 | PA22 | 34 |
| 28 | U6 | PA21 | 35 |
| 27 | INT_PROM_OE | PA20 | 36 |
| 26 | U5 | PA19 | 37 |
| 25 | U4 | PA18 | 38 |
| 24 | U3 | PA17 | 39 |
| 23 | R/W | PA16 | 40 |
| 22 | U2 | PA15 | 41 |
| 21 | PULL_UP | PA14 | 42 |
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
| 4 | U1 | +5V | 59 |
| 3 | PA2_OUT | +5V | 60 |
| 2 | PA1_OUT | +5V | 61 |
| 1 | GND | +5V | 62 |

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
* https://www.arcade-projects.com/threads/pgm-cartridge-pinout.13847/
* https://www.arcade-projects.com/threads/pgm-mvs-homebrew.24335/
