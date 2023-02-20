# Buildroot overlay for BL808 based boards

## Gadgets Playground

This fork provides a playground to develop USB gadgets as we're
ironing out all the issues. Installing this fork of buildroot_bouffalo
will add a service that starts a usb gadget on boot that will be
visible to your host system when connecting to the usb-c connector.

By default this gadget will be the adb gadget, which is reliable and
allows you to use adb to connect, copy files, restart, etc:

```
adb shell
adb push test_file /root/
adb pull /etc/good.conf .
```

To change the default gadget, edit the `MODE` listed in the file
`/etc/init.d/S70gadgets` on your Ox64 and reboot. It's as simple as
that!

## How can you help?

* Install and try various gadgets. See if they work for you and report
    any errors.
* Try building and implementing other gadgets. What other useful things
    can we do?
* Improve existing gadgets. It would be awesome if the serial gadget
    fired up a console. Can the ethernet gadget be made simpler?

## Available Gadgets

### adbd

The target-side adb tools suitable for connection from your adb host.

To see if it's working:

```
grant@grant-ubuntu ~ % adb devices
List of devices attached
OBFL Ox64	device

```

Then `adb shell` etc...

### ether (basic implementation)

This creates a usb based ethernet adapter and applies the IP address
192.168.64.1 on the Ox64.

To ssh in to the machine you must add a password to the root account
with `passwd` before enabling the ethernet gadget.

Currently the routing must be configured manually on the host machine.

On ubuntu:

```bash
ip addr # get the name of the interface such as usb0
sudo ip addr add 192.168.64.2/24 dev usb0
```

Then you should be able to `ssh root@192.168.64.1` and gain access,
use `scp` etc.

You should also be able to set up routes so that you can access the
internet at large from the Ox64 over the interface. Instructions needed.


### mass_storage (basic implementation)

This will show a dummy read only filesystem with what in the future
would be instructions and information about the project.

### serial (needs features)

This creates a `/dev/ttyACM0` device on a host machine, but my
attempts to get it to show output or have a console have failed.

## And back to the old README...

## Usage

```
mkdir buildroot_bouffalo && cd buildroot_bouffalo
git clone https://github.com/buildroot/buildroot
git clone https://github.com/openbouffalo/buildroot_bouffalo
export BR_BOUFFALO_OVERLAY_PATH=$(pwd)/buildroot_bouffalo
cd buildroot
make BR2_EXTERNAL=$BR_BOUFFALO_OVERLAY_PATH pine64_ox64_defconfig
make
```

## Prebuilt images

Prebuilt images are available on the releases page (for tested images) or development images are available via the github actions page

Two images are currently build - A minimal image - `sdcard-pine64_0x64_defconfig` and a more complete image - `sdcard-pine64_0x64_full_defconfig`

The SD card images are configured with a 1Gb Swap Partition, and will resize the rootfs partition on first boot to the full size of the SD card.

Inside the downloads you will find the following files:
* m0_lowload_bl808_m0.bin - This firmware runs on M0 and forwards interupts to the D0 for several peripherals
* d0_lowload_bl808_d0.bin - This is a very basic bootloader that loads opensbi, the kernel and dts files into ram
* pine64-ox64-firmware.bin/sispeed-m1s-firmware.bin - A image containing OpenSBI, the Kernel and DTS files for Pine64 OX64 and SiSpeed M1S Dock
* sdcard-*.tar.xz - A tarball containing the rootfs for the image to be flashed to the SD card

### Development images
Latest Development Build Result:
[![Build](https://github.com/openbouffalo/buildroot_bouffalo/actions/workflows/buildroot.yml/badge.svg)](https://github.com/openbouffalo/buildroot_bouffalo/actions/workflows/buildroot.yml)

### Released images

Released Images are [Here](https://github.com/openbouffalo/buildroot_bouffalo/releases/latest)

## Flashing Instructions

Download your prefered image above and extract the files.

- Get the latest version of DevCube from http://dev.bouffalolab.com/download
- Connect BL808 board via serial port to your PC
- Set BL808 board to programming mode
    + Press BOOT button when reseting or applying power
    + Release BOOT button
- Run DevCube, select [BL808], and switch to [MCU] page
- Select the uart port and set baudrate with 2000000
    + UART TX is physical pin 1/GPIO 14.
    + UART RX is physical pin 2/GPIO 15.
- M0 Group[Group0] Image Addr [0x58000000] [PATH to m0_low_load_bl808_m0.bin]
- D0 Group[Group1] Image Addr [0x58000000] [PATH to d0_low_load_bl808_d0.bin]
- Click 'Create & Download' and wait until it's done
- Switch to [IOT] page
- Enable 'Single Download', set Address with 0xD2000, choose [PATH to pine64-ox64-firmware.bin/sispeed-m1s-firmware.bin] (depending on your board)
- Click 'Create & Download' again and wait until it's done
- flash the sdcard-pine64-*.img.xz to your SD card (you can use dd (after uncompressing) or https://github.com/balena-io/etcher)
- Serial Console access:
    + UART TX is physical pin 32/GPIO 16.
    + UART RX is physical pin 31/GPIO 17.
    + Baud 2000000.
- Enjoy!

## Compiling Applications for BL808 based boards

Buildroot provides a "SDK" for the baords. This is a tarball containing the cross compiler and sysroot for the target board. This can be used to compile applications for the board. Please refer to https://github.com/openbouffalo/buildroot_bouffalo/wiki/Building-Programs-outside-of-buildroot for basic instructions (or consult the [buildroot documentation](https://buildroot.org/downloads/manual/using-buildroot-toolchain.txt))

## Current Status of Linux

Please refer to the projects tab for the status of drivers in development.

## Disucssions

Please use the github discussions for any questions or issues.