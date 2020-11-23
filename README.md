# About

This patch changes Linux's `joydev` module in order to make certain PlayStation dance pads, coupled with certain PlayStation/USB adapters, work with StepMania, fixing the infamous axis problem.  I know for sure that it works with my [dance pad](dance-pad.jpg) and my [adapter](adapter.jpg) (model Pu120T) in the Linux versions listed below.  Two dance pads are supported simultaneously.

# Download

Patch | Kernel   | File
----- | -------- | ----
1.2   | 4.4.92   | [joydev-dancepad-1.2-4.13.3.patch](joydev-dancepad-1.2-4.13.3.patch)
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
2. Edit `drivers/input/joydev.c` so that the macros `DANCEPAD_VENDOR` and `DANCEPAD_PRODUCT` correspond to your adapter.  You can find which values to use with, e.g., `lsusb`.
3. Compile and install `joydev.ko`.
4. When using the module, make sure StepMania is reading from `/dev/input/js*`, and not `/dev/input/event*`.  You could remove the latter in order to make sure it will read from the correct device.

If the steps above sound difficult, you may want to read the answers to [this question](http://askubuntu.com/questions/168279/how-do-i-build-a-single-in-tree-kernel-module).  Some tools that may come in handy are `dmesg` and `lsof`.

# Applying this patch to your kernel for geeks / non-kernel developers.

Here is the workings of how to apply this patch to your kernel.  Separate instructions are provided for Mageia 5 and Arch Linux. If you're using other Linux distribution you may need to adjust them slightly.

## Prep
You will need at least:  patch, wget, gcc and make installed, along with enough space for ther kernel source that matches your distribution.
This does expect you to be comfortable with the command line.

## Backup existing module
```bash
cd /lib/modules/`uname -r`/kernel/drivers/input/
cp joydev.ko.xz joydev.ko.xz.orig
```

## Mageia 5

### Unpack and CD into to the kernel source tree
Get your running kernel version, by running `uname -r`

Install matching source, in my case 4.4.92-1.mga5
```bash
 urpmi kernel-source-4.4.92-1.mga5 patch gcc wget make
 cd /usr/src/kernel-4.4.92-1.mga5
 ls -l drivers/input/joydev.c && echo "Yes, this is the correct place and the source file is present."
```

### Get the patch
```bash
 wget https://raw.githubusercontent.com/adiel-mittmann/dancepad/master/joydev-dancepad-1.2-4.13.3.patch
```

### Apply patch
```bash
patch -Np1 -i joydev-dancepad-1.2-4.13.3.patch
```

Output should be like:
```
 patching file drivers/input/joydev.c
 Hunk #1 succeeded at 38 (offset 1 line).
 Hunk #2 succeeded at 61 (offset 1 line).
 Hunk #3 succeeded at 121 (offset 1 line).
 Hunk #4 succeeded at 1008 (offset -5 lines).
```

### Prepare the source tree to compile modules for your existing running kernel
```bash
make mrproper
cp /usr/lib/modules/$(uname -r)/build/.config ./
cp /usr/lib/modules/$(uname -r)/build/Module.symvers ./
make oldconfig
make prepare && make scripts
```

### Build the input modules
```bash
make SUBDIRS=drivers/input modules
```

### Check output from the build
```bash
ls drivers/input/joydev.ko -l
xz drivers/input/joydev.ko
```

### Install module
unplug the dance pad, if you have it plugged in, then:
 ```bash
 rmmod joydev
 cp drivers/input/joydev.ko.xz /lib/modules/`uname -r`/kernel/drivers/input/
 depmod
 modprobe joykey
 dmesg
 lsmod | grep joydev
 rmmod joydev
```

## Arch Linux

All commands listed below are run under your regular non-root user. `Sudo` is used for commands that need root access.

### Get and unpack kernel source tree into ~/build

```bash
sudo pacman -S asp

cd ~
mkdir build
cd build

asp update linux
asp export linux

cd linux
makepkg -od

cd src/archlinux-linux
```

### Get the patch
```bash
wget https://raw.githubusercontent.com/adiel-mittmann/dancepad/master/joydev-dancepad-1.2-4.13.3.patch
```

### Apply patch
```bash
patch -Np1 -i joydev-dancepad-1.2-4.13.3.patch
```

Edit `drivers/input/joydev.c`: adjust `DANCEPAD_VENDOR` and `DANCEPAD_PRODUCT` based on `lsusb` output while the dancepad is connected.

### Build the module

```bash
make mrproper
zcat /proc/config.gz > .config
cp /usr/lib/modules/$(uname -r)/build/Module.symvers .
make oldconfig
make modules_prepare

# this will speed up process of compilation by only compiling joydev module
perl -i.bk -pe 's/^/#/ if /^obj-\$\(CONFIG_(INPUT_|RMI4)/ and !/joydev.o/' drivers/input/Makefile

make M=drivers/input
xz drivers/input/joydev.ko
```

### Install the module

First unplug the dance pad.

```bash
sudo modprobe -r joydev
sudo cp drivers/input/joydev.ko.xz /lib/modules/`uname -r`/kernel/drivers/input/
sudo depmod
sudo modprobe joydev
```

## Connect the dance pad
Check the kernel logs by running: `dmesg`

The output should show something like the example below.  The "Dancepad detected: activating workaround" entries are important.
```
[14293.162094] usb 4-1.1: new low-speed USB device number 3 using ehci-pci
[14293.253453] usb 4-1.1: New USB device found, idVendor=0810, idProduct=0001
[14293.253459] usb 4-1.1: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[14293.253469] usb 4-1.1: Product: Twin USB Joystick
[14293.269226] input: Twin USB Joystick as /devices/pci0000:00/0000:00:1d.0/usb4/4-1/4-1.1/4-1.1:1.0/0003:0810:0001.000B/input/input24
[14293.269305] input: Twin USB Joystick as /devices/pci0000:00/0000:00:1d.0/usb4/4-1/4-1.1/4-1.1:1.0/0003:0810:0001.000B/input/input25
[14293.269369] pantherlord 0003:0810:0001.000B: input,hidraw10: USB HID v1.10 Joystick [Twin USB Joystick] on usb-0000:00:1d.0-1.1/input0
[14293.269376] pantherlord 0003:0810:0001.000B: Force feedback for PantherLord/GreenAsia devices by Anssi Hannula <anssi.hannula@gmail.com>
[14293.276830] Dancepad detected: activating workaround.
[14293.320238] Dancepad detected: activating workaround.
```

## Remove /dev/input/eventXY before running stepmania

You'll need to do this everytime you reconnect the dance pad and/or reboot your computer.
```bash
sudo rm $(readlink -f $(evdev-joystick --l))
```

If you're looking for a permanent solution you could try to add this `udev` rule into `/usr/lib/udev/rules.d/99-remove_event_device_for_patched_joydev_device.rules`:

```
# remove /dev/input/event device corresponding to an axis-issue affected dance pad
# adjust idVendor and idProduct variables based on the output of lsusb command (while dance pad is connected)
KERNEL=="event*", NAME="input/%k", ATTRS{idVendor}=="0810", ATTRS{idProduct}=="0001", ACTION=="add", RUN+="/usr/bin/rm /dev/input/%k"
```

# DKMS configuration

Experimental DKMS setup is located in `dkms/`. In order to use it copy `joydev_dance-1.0` directory into `/usr/src` (edit VENDOR and PRODUCT in `joydev_dance.c` if appropriate) and run `dkms install joydev_dance/1.0`. This should compile and install `joydev_dance` module which can live next to the official `joydev` module. Also DKMS framework should take care of recompiling this module everytime you change (e. g. upgrade) your kernel which means you have one less thing to worry about.

Beware that you *have to* blacklist `joydev` module so that only the patched one is loaded (otherwise you'll end up with two gamepads detected).

```
echo "blacklist joydev" > /etc/modprobe.d/joydev.conf 
```

# Thanks

I would like to thank Jozef Riha for his help in testing version 1.2 of the patch.
