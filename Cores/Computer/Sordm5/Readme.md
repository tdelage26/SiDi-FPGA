Sord M5 for SiDi FPGA
===============================

This is the port of the Sord M5 for SiDi FPGA.
The SORDM5.ROM bios file needs to be placed in the same folder as the rbf for the core to boot.

## What is implemented
### Basic HW
* CPU
* CTC
* VDP TMS9918
* Sound SN76489
* Keyboard
* Joypad

### Expansion modules
* EM-5 32 KB RAM expansion (memory)
* EM-64 64 KB RAM expansion
* 64KBF 64 KB RAM expansion with Basic-F
* 64KRX 64 KB RAM expansion with WINDOWS TOOL, BASIC-I, BASIC-G, BASIC-F, MSX emulation
* Brno mod 64 KB RAM, 256 KB RAMDISK, WINDOWS TOOL, BASIC-I, CP/M loader

### Other

* ROM load.
* There is no CAS file tape support (the only way to load tape is using the basic TAPE command and connect a real or virtual tape player on the Audio_In jack).
