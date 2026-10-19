# Aleste 520EX for MiST-family FPGA boards

The **Aleste 520EX** is a computer made in Russia in the early 1990s
(Patisonic / Foton): an Amstrad CPC 6128 compatible that adds a mode of its
own with MSX-DOS, 16-colour high-resolution graphics, a 1 MB memory mapper,
a real-time clock and more. This core recreates it on MiST-family boards. It
is based on **Aleste LX** by hww (github.com/hww/aleste-lx-fpga-sv).

[Versión en español](README_es.md)

| Board | FPGA | Project | Audio |
|---|---|---|---|
| Poseidon | Cyclone IV GX EP4CGX150 | `poseidon/aleste_poseidon.qpf` | I2S |
| MiST | Cyclone III EP3C25 | `mist/aleste_mist.qpf` | sigma-delta |
| SiDi | Cyclone IV E EP4CE22 | `sidi/aleste_sidi.qpf` | sigma-delta |
| Calypso (CYC1000) | Cyclone 10 LP 10CL025 | `calypso/aleste_calypso.qpf` | I2S |

## Features

- **Two machines in one**, chosen from the OSD (*CPC boot*):
  - **AMSTRAD mode**: CPC 6128 compatible, BASIC 1.1 and AMSDOS. Tape
    loading from CDT/TZX files or from a real tape recorder through the
    board audio input (not on the MiST), AMSDOS disk images (DSK/EDSK), CPC mice
    (Kempston, SYMBiFACE II, Multiplay, AMX).
  - **Aleste mode**: the 520EX's own BOOTMENU and SETUP, and **MSX-DOS 1**
    with the 520EX BIOS: 640x200 and 320x200 16-colour modes with a 64-colour
    palette, 1 MB memory mapper, serial mouse, two floppy drives.
- **Z80** at 4 MHz, with the 520EX turbo mode (8 MHz) and 16 MHz.
- **Video**: Gate Array, 6845 CRTC (UM6845R) and the 520EX high-resolution
  modes. VGA output (scan doubler, scanlines) or 15 kHz RGB; horizontal and
  vertical centring.
- **Sound**: AY-3-8912 (mono or stereo ABC), **MagicSound** card (four
  channels, 8237 DMA and 8254 timer), tape sound.
- **uPD765 floppy controller** with two drives.
- **MC146818 real-time clock**, set from the MiST firmware clock, and
  **CMOS** (SETUP settings) saved to the SD card.
- **520EX keyboard** matrix, including the Russian keys and F1-F5, HELP, INS.

## SD card

Copy the contents of `sd/` to the root of the SD card (details in
[sd/README.md](sd/README.md)):

- `ALESTE.ROM`: system ROM, loaded automatically.
- `ALESTE.HD2`: CMOS (SETUP settings), mounted automatically.
- `dsk/ALESTDOS_TIME.dsk`: MSX-DOS boot disk with `TIME.COM` and two drives.
  It is in a subfolder so that it is not mounted automatically; mount it from
  the OSD, or copy it to the root as `ALESTE.HD0`.

## Tools

All in `tools/` (Python 3; the documentation of each one is in its header):

| Tool | Purpose |
|---|---|
| `make_msxdisk.py` | MSX-DOS disk images from a folder (with the system files of a boot disk); list, add and extract files in existing images |
| `make_amsdisk.py` | AMSDOS (AMSTRAD mode) disk images from a folder, split into several disks keeping each program together |
| `make_cdt.py` | Test tapes (CDT/TZX) |
| `make_rom.py` | `ALESTE.ROM` from the 520EX ROM (`AL512.BIN`) |
| `msxrom2aleste.py` | Converts MSX cartridges to the 520EX with the Aleste "GameBios" (see below) |
| `raw2edsk.py`, `bdos_2drives.py`, `dos_preload.py` | Disk image conversion, two-drive BDOS patch, simulation helpers |
| `gen_rtl.py`, `gen_u765.py` | Generate the patched RTL from Aleste LX and the u765 controller |
| `check_quartus.py`, `aleste_wbs.py`, `gen_font.py` | Quartus compatibility check, bus scripts, font for the diagnostic ROM |
| `diag/` | Diagnostic ROM (Z80 assembler, `pasmo`) |
| `gb2/` | Generic loader for MSX2 games with the GameBios v2 (Z80 assembler) and test benches in a Z80 emulator |

`msx/TIME.COM` (with its source) sets the real-time clock from MSX-DOS.

### MSX cartridges on the 520EX

The Aleste community converted MSX games with the **GameBios**, a
replacement MSX page 0 with a software VDP emulator that came inside each
game loader. `msxrom2aleste.py` automates conversions in the same way:
Konami's 1985 16 KB engine (King's Valley, Yie Ar Kung-Fu) with the first
GameBios, and **Metal Gear** (MSX2) with the second one and a loader of our
own. The GameBios is taken from the Aleste software that you provide
(`msx-games` disk); no game or GameBios is included in this repository. The
analysis and disassembly of the GameBios are in `doc/`.

## How it works

The core reuses Aleste LX as a git submodule: `tools/gen_rtl.py` generates
the files in `rtl/` from it, with the changes needed for these boards and
for the 520EX (each change is described in the script). The 520EX's own
hardware (CPC video, keyboard, clock, MagicSound, mice, tape) is in
`rtl/cpc/`. The memory is in the board SDRAM; on the Calypso (8 MB) the
memory map is relocated.

## Building

```
git clone --recursive https://github.com/shaeon/aleste520ex.git
```

Open the project for your board in Quartus Prime 22.1 Lite and compile; the
`.rbf` is written to `output_files/`. The Verilator test benches are in
`sim/` (`make`, `make fase2`, ...). The documentation in `doc/` is in
Spanish; [doc/00-guia-de-uso.md](doc/00-guia-de-uso.md) is the user guide.

## Credits

This core stands on the work of many people. Thank you all.

**Code used in the core**

- **Aleste LX** (the base of the core: Z80, MMU, video, memory arbiter and
  more): hww (github.com/hww/aleste-lx-fpga-sv). Apache License 2.0. Used as
  a submodule; the files generated from it keep their origin noted.
- **MiST modules** (`user_io`, `data_io`, video, OSD, scan doubler): Till
  Harbaum, Gyorgy Szombathelyi (gyurco) and the MiST community. GPL v3 or
  later.
- **u765** uPD765 floppy controller and **tzxplayer**: Gyorgy Szombathelyi
  (gyurco), from his Amstrad and Spectrum MiST cores. GPL.
- **UM6845R** CRTC and the **Kempston, SYMBiFACE II, Multiplay and AMX**
  mouse modules: Sorgelig, adapted for MiST by gyurco. GPL.
- Delta-sigma DAC (SiDi): based on Xilinx application note XAPP154.

**References**

- **hww's Aleste 520EX archive** (github.com/hww/Aleste520EX): ROMs,
  schematics, manuals, software and the GameBios loaders.
- **CPCWiki** pages on the Aleste 520EX, and the **MAME** `al520ex` driver.
- **Metal Gear** annotated disassembly by Manuel Pazos
  (github.com/GuillianSeed/MetalGear).
- fMSX disk tools `rddsk`/`wrdsk` (Arnold Metselaar) and `cpcxfs`, as a
  reference for the disk tools; the openMSX software database, to identify
  ROMs.
- Board pinouts taken from the NewBrain core (github.com/shaeon/newbrain).

**Tools**

- The GameBios test benches and analyses use the Python `z80` package by
  Ivan Kosarev (MIT license, not included). The Z80 sources assemble with
  `pasmo`.

## Thanks

To **hww**, for Aleste LX and for preserving and publishing the Aleste 520EX
material, and to the people who designed and programmed the Aleste 520EX.

To the **Retro-Wiki FPGA-dev Team**, especially **Ron, Manuel (teiram), Somhi,
Roderick, Rampa, Kyp and Benito**, for their help and patience. A special
mention to **Ron**.

To **Jaime González (alt)**, possibly the only owner of an Aleste 520EX in
Spain, for his unconditional help.

## AI assistance and resources

Parts of this core were developed with the assistance of Claude (Anthropic),
used for code, testbenches, co-simulation and documentation. Access to Claude
was provided by Advanced Computer Trading, S.L. (actsl.com), which also
authorises the publication of this work under the license below. All design
decisions, integration, testing on real hardware and final review were
carried out by the author, Shaeon (Carlos Palmero).

## License

Copyright (C) 2026 Shaeon (Carlos Palmero).

This program is free software: you can redistribute it and/or modify it
under the terms of the **GNU General Public License** as published by the
Free Software Foundation, either **version 3** of the License, or (at your
option) any later version. See [LICENSE](LICENSE).

Version 3 is required because some of the included modules (the MiST
modules) are licensed under GPL v3 or later. Third-party files keep their
own copyright notices and licenses, which are compatible with the GPL v3
(Aleste LX, Apache License 2.0).

The 520EX ROM in `sd/ALESTE.ROM` comes from the material published by hww
and is included for convenience; it belongs to its respective owners.

## Disclaimer

This project is provided **"as is", without warranty of any kind**, express
or implied, including but not limited to the warranties of merchantability
and fitness for a particular purpose. In no event shall the authors or
contributors be liable for any claim, damages or other liability arising
from the use of this software, of the hardware it runs on, or of any
connected equipment. Use it at your own risk.

Aleste, Amstrad, MSX and the names of the games mentioned are trademarks of
their respective owners. This project is not affiliated with or endorsed by
them.
