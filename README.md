# ACZ_CE_freecam_script

Descrition:
-----------
A Cheat Engine table file containing scripts that allows the user to enable a very rudimentary freecam mode when running Ace Combat Zero: The Belkan War in the PCSX2 emulator. This table is made of multiple scripts that enable the freecam mode not only during flight, but also in the hangar during the aircraft/SpW selection screen. 

Besides the freecam scripts the table also comes with optional scripts that can modify some graphic parameters on the fly, locate entities and print their coordinates and even enable what seems to be a dummied out "free look" option as part of a (yet to be uncovered if it exists) in-game debug tool.

The scripts within the table uses CE's Auto Assembler language to perform the operations required to disable the game's camera code and give the user control to manipulate it. If you want to modify this table but want to get acquainted with the language or with Cheat Engine in general you can head over to their official [wiki](https://wiki.cheatengine.org/index.php?title=Main_Page) and/or [forums](https://forum.cheatengine.org/).

Required files/applications:
----------------------------
- Have Cheat Engine version 7.0 or newest installed in your computer.
- The american/NTSC-U release of Ace Combat Zero: The Belkan War **(SLUS 21346)** .
- PCSX2, preferably the lastest stable version (1.6.0) although it also has been tested with version PCSX2 version 1.7.0-dev-158. **See NOTES AND KNOWN ISSUES section.**

Usage:
------
TODO

Notes and known issues:
-----------------------
- The original purpose of this table was to create a way to manipulate the game's camera and move it around an aircraft to take screenshots but moving the camera to see the environment around is just an afterthought and it can be a bit tedious to do. If this is the case please use the FREE LOOK script for this.

- **It is very likely that the freecam scripts will not work on your end.** It's a bit difficult to explain the reason but CE forum user PenguinParkour hopefully can shed a bit of light on the cause of this issue. [To quote him](https://www.cheatengine.org/forum/viewtopic.php?t=591303):

>If you did research into what pointers are and emulators are, you should realize finding a static pointer to some value in a software being emulated is completely different from finding one in an already compiled binary. The pcsx2 emulator could be categorized as a virtual machine- a piece of software that emulates a computer system. As such, that asm you're looking at isn't a part of the game, but of the emulator itself. AKA, those instructions you see won't lead you to any pointers. You'd need to reverse engineer both the emulator and the game to find a good reference to your value. Even then, you'd still need a static reference from your computer to the game in the emulator.

Basically, the scripts on this table are dependent of some code instructions (also called opcodes) that are specific to early versions of PCSX2. As the emulator is contantly updated its base/source code is also constantly changing meaning that these instructions are also changed, rendering this table useless for the newest (and future stable) builds of this emulator. To make things worse, the chances of the freecam script working are also tied to your computer specs, such as CPU architecture, OS, etc. So yeah, if you are going to try to run this table keep in mind that it is very likey that it will not work and sadly there's not much I can do about it. So this is why I ask you to use either the last stable version of PCSX2 or any of the nightly builds up to PCSX2 1.7.0-dev-158, so the chances of the script working are higher.

- When zooming into the player's aircraft some weird graphical glitches can occur (such as the aircraft model becoming distorted) just hit the panic key (SPACE key) to reset the camera position.

TODO
----
- Find a way to make the input of the freecam speed values a bit easier.
- Add proper support for cockpit and HUD views.
- Clean the code a bit.

Special thanks
--------------
- BelkanLoyalist (@ Twitter and ModDB) for testing and helping with some things.

- Frouk from the CE forums for the scripting help.
