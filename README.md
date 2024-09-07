# IGS PGM Technical description

* [Overview](#overview)
* [Cartridge pinout](#cartridge-pinout)
* [Memory map](#memory-map)
* [References](#references)

## Overview

## Memory map

## Cartridge pinout

### PROG board

### Left

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

### Right

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

### CHAR board

### Left

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 |  |  | 32 |
| 30 |  |  | 33 |
| 29 |  |  | 34 |
| 28 |  |  | 35 |
| 27 |  |  | 36 |
| 26 |  |  | 37 |
| 25 |  |  | 38 |
| 24 |  |  | 39 |
| 23 |  |  | 40 |
| 22 |  |  | 41 |
| 21 |  |  | 42 |
| 20 |  |  | 43 |
| 19 |  |  | 44 |
| 18 |  |  | 45 |
| 17 |  |  | 46 |
| 16 |  |  | 47 |
| 15 |  |  | 48 |
| 14 |  |  | 49 |
| 13 |  |  | 50 |
| 12 |  |  | 51 |
| 11 |  |  | 52 |
| 10 |  |  | 53 |
| 9 |  |  | 54 |
| 8 |  |  | 55 |
| 7 |  |  | 56 |
| 6 |  |  | 57 |
| 5 |  |  | 58 |
| 4 |  |  | 59 |
| 3 |  |  | 60 |
| 2 |  |  | 61 |
| 1 |  |  | 62 |

### Right

| nr | bottom | top | nr |
| --: | :-- | --: | :-- |
| 31 |  |  | 32 |
| 30 |  |  | 33 |
| 29 |  |  | 34 |
| 28 |  |  | 35 |
| 27 |  |  | 36 |
| 26 |  |  | 37 |
| 25 |  |  | 38 |
| 24 |  |  | 39 |
| 23 |  |  | 40 |
| 22 |  |  | 41 |
| 21 |  |  | 42 |
| 20 |  |  | 43 |
| 19 |  |  | 44 |
| 18 |  |  | 45 |
| 17 |  |  | 46 |
| 16 |  |  | 47 |
| 15 |  |  | 48 |
| 14 |  |  | 49 |
| 13 |  |  | 50 |
| 12 |  |  | 51 |
| 11 |  |  | 52 |
| 10 |  |  | 53 |
| 9 |  |  | 54 |
| 8 |  |  | 55 |
| 7 |  |  | 56 |
| 6 |  |  | 57 |
| 5 |  |  | 58 |
| 4 |  |  | 59 |
| 3 |  |  | 60 |
| 2 |  |  | 61 |
| 1 |  |  | 62 |

## References
* http://www.igspgm.com/
* http://www.igspgm.com/repairs/tech.htm
* http://www.igspgm.com/iq132/data1.htm
* https://github.com/mamedev/mame/tree/master/src/mame/igs
* https://www.arcade-projects.com/threads/pgm-cartridge-pinout.13847/
