# IGS PGM Tech Scroll

* [Overview](#overview)
* [68000 memory map](#68000-memory-map)
* [Z80 memory map](#z80-memory-map)
* [Video chip operation](#video-chip-operation)
* [Audio chip operation](#audio-chip-operation)
* [Cartridge pinout](#cartridge-pinout)
* [References](#references)

## Overview

IGS PolyGame Master is a cartridge based system.

### Main board

From programmer's perspective important components located on motherboard are:
* Main CPU Motorola MC68HC000 CPU clocked at 20 MHz
* Secondary CPU for sound handling Zilog Z80 at 8.468 MHz
* Custom video chip IGS 023
* Sound chip Wave Front ICS2115
* [128 kB of BIOS](#internal-bios)
* 2 MB tile graphics ROM
* 2 MB audio samples data ROM
* [128 kB main CPU work RAM](#main-work-ram)
* [64 kB Z80 work RAM](#z80-ram)
* [video](#video-ram) / [palette RAM](#palette-ram)

### Cartridge

 A cartridge consists of two Printed Circuit Boards.

[Top board](#top-prog-board) contains:
 * 16 bit [`P` ROM](#program-rom) with main CPU program with 23 bit address space (max 16 MB),
 * 16 bit `T` ROM with tile graphics with 23 bit address space (max 16 MB),
 * optionally ASIC used for protection/encryption.

[Bottom board](#bottom-char-board) contains:
 * 8 bit `M` ROM with audio samples data with 24 bit address space (max 16 MB)
 * 16 bit `B` ROM with sprite pixel masks and pixel color offsets with 23 bit address space (max 16 MB)
 * 15 bit `A` ROM with sprite pixel color data with 25 bit address space (max 64 MB)

### Logical components layout

The main CPU has mapped into its address-space: BIOS, work RAM, video / palette RAM, Z80 interface, Z80 work RAM, I/O registers and external `P` ROM.

Secondary CPU has access to its work RAM, main CPU interface and sound chip interface.

Video chip has access to video / palette RAM, internal 2 MB of tile data ROM, and external `T`, `B` and `A` ROMs 

Sound chip has access to internal 2 MB audio samples ROM and external `M` ROM

## 68000 Memory map

| address rage | mirroring | description |
| :-- | :--: | :-- |
| `0x000000-0x01ffff` | `0x0e0000` | [internal BIOS](#internal-bios)
| `0x100000-0x7fffff` | - | [`P` program ROM](#program-rom)
| `0x700006-0x700007` | - | [W/O irq4 ack](#irq4-ack)
| `0x800000-0x81ffff` | `0x0e0000` | [main work RAM](#main-work-ram)
| `0x900000-0x907fff` | `0x0f8000` | [video RAM](#video-ram)
| `0xa00000-0xa01fff` | `0x0fe000` | [palette RAM](#palette-ram)
| `0xb00000-0xb0ffff` | `0x0f0000` | [video registers](#video-registers)
| `0xc00000-0xc0000f` | `0x0e7ff0` | [Z80 interface and RTC regs](#z80-interface-and-rtc-regs)
| `0xc08000-0xc08007` | `0x0e7ff8` | [I/O regs](#io-regs)
| `0xc10000-0xc1ffff` | `0x0e0000` | [Z80 RAM](#z80-ram)
| `0xd00000-0xffffff` |- | [`P` program ROM](#program-rom) ?

### Internal BIOS

Machine starts up from internal BIOS. If cartridge is not inserted (or test button is pressed) it runs main menu. Without cartridge it is mirrored to first 8MB of address space. 

### Program ROM

`P` ROM must start from first 128 entries of Motorola 68000 exception vector table and some mandatory fields.

| offset | size | description |
| --: | :-- | :-- |
| `0x000` | `0x04` | initial stack pointer |
| `0x004` | `0x04` | program start address |
| `0x070` | `0x04` | [irq4](#irq4-ack), configured by cartridge, e.g. coin insertion |
| `0x078` | `0x04` | [irq6](#irq6-vbl), VBL interrupt |
| `0x200` | `0x20` | `IGS PGM PLATFORM GAMES\0\0\0\0\0\0\0\0\0\0` |
| `0x220` | `0x10` | zero padded game name |
| `0x230` | `0x0a` | version string |
| `0x23a` | `0x04` | [initialization routine address](#initialization) |
| `0x23e` | `0x0a` | date |
| `0x248` | `0x08` | time |

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
| `0x800000-0x8009ff` | buffer for 256 sprites 10 bytes each |
| `0x800a00-0x81ffff` | general usage |

### Video RAM

| address range | description |
| :-- | :-- |
| `0x900000-0x903fff` | definition of 64*64 background layer, 4 bytes each tile |
| `0x904000-0x905fff` | definition of 64*32 text layer, 4 bytes each character |
| `0x907000-0x9077ff` | row scroll RAM |

### Palette RAM

| address range | description |
| :-- | :-- |
| `0xa00000-0xa007ff` | 32 * 2 bytes x 32 sprite palettes |
| `0xa00800-0xa00fff` | 32 * 2 bytes x 32 background palettes |
| `0xa01000-0xa011ff` | 16 * 2 bytes x 32 text palettes |
| `0xa01200-0xa01fff` | unused palette RAM |

### Video Registers

| address range | description |
| :-- | :-- |
| `0xb00000-0xb00fff` | buffer for 256 sprites 16 bytes each copied by sprite DMA |
| `0xb01000-0xb0103f` | zoom table, 16 entries * 4 bytes each, W/O |
| `0xb02000-0xb02001` | background scroll up |
| `0xb03000-0xb03001` | background scroll left |
| `0xb04000-0xb04001` | zoom flags ? |
| `0xb05000-0xb05001` | text scroll up |
| `0xb06000-0xb05001` | text scroll left |
| `0xb07000-0xb07001` | screen scanline, R/O |
| `0xb06000-0xb06001` | [control flags](#control-flags) |

#### Control flags

| mask | description |
| :-- | :-- |
| `0x0001` | sprite dma enable - pulse 0->1 to trigger |
| `0x0002` | _nothing?_ |
| `0x0004` | irq4 ack - pulse 0->1 to trigger |
| `0x0008` | iqr6 ack - pulse 0->1 to trigger |
| `0x0010` | _all games set this?_ |
| `0x0060` | _all games except CAVE set this, but seems to serve no purpose when set?_ |
| `0x0080` | _causes system to lose video synch?_ |
| `0x0100` | _shows garbage on screen for all except background?_ |
| `0x0200` | _disable everything except background layer?_ |
| `0x0400` | _nothing?_ |
| `0x0800` | disable text layer |
| `0x1000` | disable background layer |
| `0x2000` | disable high priority sprites |
| `0xc000` | _nothing?_ |

### Z80 interface and RTC regs

| address range | description |
| :-- | :-- |
| `0xc00002-0xc00003`| sound latch 1 |
| `0xc00004-0xc00005`| sound latch 2 |
| `0xc00006-0xc00007`| calendar |
| `0xc00008-0xc00009`| Z80 reset W/O |
| `0xc0000a-0xc0000b`| Z80 control W/O |
| `0xc0000c-0xc0000d`| sound latch 3 |

### I/O regs
    $c08000 R/O |xxxxxxxxxxxxxxxx| Player 1 & 2 controls
                 ||||||||||||||||_ P1 START
                 |||||||||||||||__ P1 UP
                 ||||||||||||||___ P1 DOWN
                 |||||||||||||____ P1 LEFT
                 ||||||||||||_____ P1 RIGHT
                 |||||||||||______ P1 A
                 ||||||||||_______ P1 B
                 |||||||||________ P1 C
                 ||||||||_________ P2 START
                 |||||||__________ P2 UP
                 ||||||___________ P2 DOWN
                 |||||____________ P2 LEFT
                 ||||_____________ P2 RIGHT
                 |||______________ P2 A
                 ||_______________ P2 B
                 |________________ P2 C

    $c08002 R/O |xxxxxxxxxxxxxxxx| Player 3 & 4 controls
                 ||||||||||||||||_ P3 START
                 |||||||||||||||__ P3 UP
                 ||||||||||||||___ P3 DOWN
                 |||||||||||||____ P3 LEFT
                 ||||||||||||_____ P3 RIGHT
                 |||||||||||______ P3 A
                 ||||||||||_______ P3 B
                 |||||||||________ P3 C
                 ||||||||_________ P4 START
                 |||||||__________ P4 UP
                 ||||||___________ P4 DOWN
                 |||||____________ P4 LEFT
                 ||||_____________ P4 RIGHT
                 |||______________ P4 A
                 ||_______________ P4 B
                 |________________ P4 C

    $c08004 R/O |xxxxxxxxxxxxxxxx| Extra controls
                 ||||||||||||||||_ P1 COIN
                 |||||||||||||||__ P2 COIN
                 ||||||||||||||___ P3 COIN
                 |||||||||||||____ P4 COIN
                 ||||||||||||_____ P1/2 TEST
                 |||||||||||______ P1/2 SERVICE
                 ||||||||||_______ P3/4 TEST
                 |||||||||________ P3/4 SERVICE
                 ||||||||_________ P1 D
                 |||||||__________ P2 D
                 ||||||___________ P3 D
                 |||||____________ P4 D
                 ||||_____________ RESET
                 |||______________ RESERVED1
                 ||_______________ RESERVED2
                 |________________ RESERVED3

    $c08006 R/O |........x...xxxx| Dip switches
                         |   ||||_ Test mode (1-ON,0-OFF)
                         |   |||__ Music (0-ON,1-OFF)
                         |   ||___ Voice (0-ON,1-OFF)
                         |   |____ Free Play (0-ON,1-OFF)
                         |   |____ Stop Mode (0-ON,1-OFF)
                         |________ QC mode (1-ON,0-OFF)

All inputs are active low.

With the QC mode dip switch set you can hold A+B when turning on the PGM to access the QC test menu. QC menu options are selected by pressing 1P START rather than 1P A as with the normal operator test menu. You can also hold B+C down in QC mode when turning on the PGM to boot the current cartridge in QC mode. Depending on the cartridge there may or may not be additional functionality.

### Z80 RAM

Range `0xc10000-0xc1ffff` maps to Z80 RAM, that needs to be loaded by main CPU.

## Z80 memory map

Whole Z80 address space is occupied by RAM, that is populated by msin CPU.

### Z80 I/O map

| address range | description |
| :-- | :-- |
| `0x8000-0x8003`| ICS 2115 Interface |
| `0x8100-0x81ff`| sound latch 3 |
| `0x8200-0x82ff`| sound latch 1 |
| `0x8400-0x84ff`| sound latch 2 |

## Video chip operation

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
