Matra-Hachette Alice MC-10 for SiDi FPGA
==========================================

This is the port of the Alice 4K / Tandy MC-10 to SiDi.

Cassettes
---------

To facilitate the use of the cassette player, an option to display the data stream on the screen is available from the OSD.

The core is compatible with .c10 tape files.

How to load a tape :

After booting the core, just type cload and hit enter. Then press F12 to display the menu and load a tape file.
Once done you're advised to set "show tape stream" and "Enable tape audio" options to ON.
Then select "Rewind tape" (just to make sure the file will start at the beginning) and next select "Play/pause tape" to start the reading of the tape file (you should be able to hear through your speakers and visualize the stream on the overlay at the bottom part of the screen).
When done, leave the menu pressing F12 and once the game will be finished loading you should get an OK prompt on screen.
When it appears, just type RUN to launch the program you just loaded.

Joystick
--------

While the Alice 4k was sold with a DB9 adapter cartridge, the MC10 has no official support for joysticks. The two joystick interfaces have been implemented in the core.
