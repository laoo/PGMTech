# IGS PGM Tech Scroll

* [Overview](#overview)
* [68000 Memory map](#68000-memory-map)
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
Top board contains:
 * 16 bit [`P` ROM](#program-rom) with main CPU program with 23 bit address space (max 16 MB),
 * 16 bit `T` ROM with tile graphics with 23 bit address space (max 16 MB),
 * optionally ASIC used for protection/encryption.

Bottom board contains:
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
| 0x000000-0x01ffff | 0x0e0000 | [internal BIOS](#internal-bios)
| 0x100000-0x7fffff | - | [`P` program ROM](#program-rom)
| 0x700006-0x700007 | - | [W/O irq4 ack](#irq4-ack)
| 0x800000-0x81ffff | 0x0e0000 | [main work RAM](#main-work-ram)
| 0x900000-0x907fff | 0x0f8000 | [video RAM](#video-ram)
| 0xa00000-0xa01fff | 0x0fe000 | [palette RAM](#palette-ram)
| 0xb00000-0xb0ffff | 0x0f0000 | [video registers](#video-registers)
| 0xc00000-0xc0000f | 0x0e7ff0 | [Z80 interface and RTC regs](#z80-interface-and-rtc-regs)
| 0xc08000-0xc08007 | 0x0e7ff8 | [I/O regs](#io-regs)
| 0xc10000-0xc1ffff | 0x0e0000 | [Z80 RAM](#z80-ram)
| 0xd00000-0xffffff |- | [`P` program ROM](#program-rom) ?

### Internal BIOS

### Program ROM

### IRQ4 Ack

### Main work RAM

### Video RAM

### Palette RAM

### Video Registers

### Z80 interface and RTC regs

### I/O regs

### Z80 RAM

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
