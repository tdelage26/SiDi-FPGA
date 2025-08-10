Sega Master System and Game Gear Core for SiDi128
----------------------------------------------

This core is a port of Bens Sega Master System implementation for the
Papilio. See http://fpga-hacks.blogspot.de/

The core has been adopted to the SiDi128 upload logic so cartridge images
can be uploaded via the SiDi128 OSD as usual. To run a cartridge image
press F12 to open the OSD. Select a *.SMS file for upload. The core
will automatically be restarted after upload, so the cartridge is
started immediately.

It's recommended to use a USB joystick or gamepad with more the one
fire button since many SMS games expect a second fire button.


Backup RAM support
------------------

Some game carts have a battery-backed RAM, where high scores or other states
are preserved. You can use this on SiDi128 with a bit of preparation:

- Create an empty file on the SD-Card, with .sav extension (with 8192 bytes length).
- After loading the ROM, choose "Mount *.SAV" OSD option, and select the
  files created previously. The game will reset, after the RAM content has loaded.
- To preserve the Backup RAM state, choose "Write Save RAM" from the OSD menu.
  A loaded Backup RAM is indicated by the yellow LED. If it doesn't lit, saving is not
  possible.

A list of games with Backup RAM: http://www.smspower.org/Tags/MemoryBackup

* Note: only 8K backup RAM is supported (maybe some carts have up to 32K, but
  seems all uses only 8K).

History
-------

  - Initial release
