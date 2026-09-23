# Prototype Z1RCC, A RC2014-Compatible, RomWBW-Capable Z180 SBC
## Introduction
The motivation for Z1RCC came from a new RomWBW feature that accommodates ROM-less Z80/Z180/Z280 computer with 512K of RAM. Z180 in DIP64 package does not have Address 19 bonded out so it can only have 512K memory, so the new RomWBW feature can work with DIP64 Z180, assuming there is a mechanism to preload the 512K with RomWBW image. This concept is prototyped with RIZ180 rev1 pc board which already had Z180, RAM, and CF wired. All it needed is a CPLD with appropriate logic to decode RAM, CF and small ROM to load RomWBW program from CF disk to RAM. This link to discussion about Z1RCC.

![protoTop](Z1RCC_prototype_topview.jpg)
## Features
- 9.22MHz Z80180 DIP64
- 512K RAM
- 44-pin IDE interface
- EPM7064SLC44 CPLD
- RTC with DS1302
- I2C bus
- 18.432 MHz oscillator
- 2“x4” pc board
- RomWBW capable
- Compatible with RC2014 bus

## Theory of Operation
The prototype Z1RCC is not completely ROMless. It does have a small 64-byte ROM in CPLD which is about as big as the 64-macrocell CPLD can support. At powerup, the 64-byte ROM resides in memory 0x0-0x3F so Z180 immediately executes the ROM program which continuously polls the compact flash BUSY signal as well as the serial port receive ready flag. If serial receive ready flag is set, it jumps into the serial bootstrap routine that loads 256 serial data into memory starting from 0xA000 and jump into 0xA000 after the 256th serial data is received.

If CF BUSY is negated, it jumps into the CF bootstrap routine that reads the 256 words of data from CF's Master Boot Record (track 0, sector 1) into memory starting from 0xA000 and jump into 0xA000 after the 256th words are read.

Because it takes a moment (0.5-1 second) for the CF disk to be ready after reset, the user can select CF bootstrap by waiting a second for the CF disk to be ready and boot or serial bootstrap by sending a character to serial port immediately after the reset button is released. The serial bootstrap is primarily for loading CF initialization software to set up a new CF disk.

## Design Information
- Schematic
- CPLD design file
  - PDF of CPLD top level schematic
  - Bootstrap ROM inside CPLD

### Prototype modifications
The prototype starts with the RIZ180 rev1 pc board because most of the wiring between Z180, RAM, CF disk, and logic are already in place. It is fairly easy to wire CPLD's signals to the vacated ROM and glue logic.

Z1RCC prototype construction aid, layout of CPLD

## Software
- Z1RCC Monitor, rev 0.2a

- Z1RCC serial loader, first load this file as binary in serial bootstrapping mode. This is a Intel Hex loader.

- Z1RCC CPLD bootstrap ROM, 64-byte ROM resided in CPLD executed by Z180 immediately after reset

- Interim RomWBW CF image, use Win32DiskImager to copy this image file to a new CF disk. This image has patch to run serial port at 57.6K because the auto-baud function failed due to excessive drift of RTC.
