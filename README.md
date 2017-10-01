# About

This patch changes Linux's `joydev` module in order to make certain PlayStation dance pads, coupled with certain PlayStation/USB adapters, work with StepMania, fixing the infamous axis problem.  I know for sure that it works with my [dance pad](dance-pad.jpg) and my [adapter](adapter.jpg) (model Pu120T) in the Linux versions listed below.  Two dance pads are supported simultaneously.

# Download

Patch | Kernel   | File
----- | -------- | ----
1.2   | 4.13.3   | [joydev-dancepad-1.2-4.13.3.patch](joydev-dancepad-1.2-4.13.3.patch)
1.2   | 3.14.6   | [joydev-dancepad-1.2-3.14.6.patch](joydev-dancepad-1.2-3.14.6.patch)
1.2   | 3.1.6    | [joydev-dancepad-1.2-3.1.6.patch](joydev-dancepad-1.2-3.1.6.patch)
1.1   | 2.6.37   | [joydev-dancepad-1.1-2.6.37.patch](joydev-dancepad-1.1-2.6.37.patch)
1.1   | 2.6.29.6 | [joydev-dancepad-1.1-2.6.29.6.patch](joydev-dancepad-1.1-2.6.29.6.patch)
1.1   | 2.6.25.9 | [joydev-dancepad-1.1-2.6.25.9.patch](joydev-dancepad-1.1-2.6.25.9.patch)
1.1   | 2.6.22.8 | [joydev-dancepad-1.1-2.6.22.8.patch](joydev-dancepad-1.1-2.6.22.8.patch)

Download the file that best matches your kernel version, but notice that version 1.1 of the patch applies the fix to whatever device Linux recognizes as a joystick, whereas version 1.2 applies the fix selectively.

# Use

1. Apply this patch to a properly configured Linux source tree:
````
patch -Np1 -i joydev-dancepad.patch
````
2. Edit `drivers/input/joydev.c` so that the macros `DANCEPAD_VENDOR` and `DANCEPAD_VENDOR` correspond to your adapter.  You can find which values to use with, e.g., `lsusb`.
3. Compile and install `joydev.ko`.
4. When using the module, make sure StepMania is reading from `/dev/input/js*`, and not `/dev/input/event*`.  You could remove the latter in order to make sure it will read from the correct device.

If the steps above sound difficult, you may want to read the answers to [this question](http://askubuntu.com/questions/168279/how-do-i-build-a-single-in-tree-kernel-module).  Some tools that may come in handy are `dmesg` and `lsof`.

# Thanks

I would like to thank Jozef Riha for his help in testing version 1.2 of the patch.
