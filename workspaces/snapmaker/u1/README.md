# Dredge workspace for Snapmaker U1

## Handy Links

Product Page: https://www.snapmaker.com/snapmaker-u1  
Support Page: https://support.snapmaker.com/hc/en-us/categories/36087874981527  
Firmware releases: https://wiki.snapmaker.com/en/snapmaker\_u1/firmware/release\_notes

## System Description

Uses an android-ish A/B boot process, and a buildroot-based filesystem. Android-esque /oem, /vendor, /userdata, and similar are present.  
The root is rendered writable by overlying with /oem/overlay, but unless the "debug" flag is set (/oem/.debug), this overlay is cleared on boot.

The UI is drawn by a compiled binary named `gui` in the app user's home, which contains LVGL, and writes to the framebuffer directly, as well as does houskeeping tasks.

## Updates
Updates are a matroska:

1. Outermost is a format unknown to the project so far. The unpack implementation, and other details are from the extended firmware (see WHENCE entry for upfile)
2. Rockchip firmware bundle (incl loader)
3. Rockchip android package

1 is unpacked by upfileUnpack.
```
Usage: upfileUnpack [-c] [-o <output directory>] [-i <input file>]
parameter:
    -i    Input file, use default upgrade.bin if not specified.
    -o    Output directory, use default output/ if not specified.
    -c    Only check and display file information.
    -h    Display this help information.

example:
    upfileUnpack -o output -i upgrade.bin
```

It implements a check function, which gives the following output when run on 0.0.9
```
% /bin/upfileUnpack -c -i 0.0.9-upgrade.bin
Version         : 0.9.0.121
Build time      : 20251106132913
Sub files count : 4

Sub file 0:
Type : 0
Size : 273242698
MD5  : 76ff81e23f3588a61f3b6a8c1b14e880

Sub file 1:
Type : 1
Size : 46988
MD5  : c1266f484ec4f7f0290d8b0c7c2a776e

Sub file 2:
Type : 2
Size : 45452
MD5  : c5d0663a1d26a9273208293dd8981b04

Sub file 3:
Type : 3
Size : 26
MD5  : af7331c1f7e37118c7b856cfb99205f3
```

2 is then unpacked with rockchip's updateEngine, which also handles the remaining layers

```
LOG_INFO: *** update_engine: V1.0.1-g<unknown> ***.
LOG_INFO: --misc=now             Linux A/B mode: Setting the current partition to bootable.
LOG_INFO: --misc=other           Linux A/B mode: Setting another partition to bootable.
LOG_INFO: --misc=update          Recovery mode: Setting the partition to be upgraded.
LOG_INFO: --misc=display         Display misc info.
LOG_INFO: --misc=wipe_userdata   Format data partition.
LOG_INFO: --misc_custom= < op >  Operation on misc for custom cmdlineLOG_INFO:         op:     read   Read custom cmdline to /tmp/custom_cmdlineLOG_INFO:                 write  Write /tmp/custom_cmdline to custom areaLOG_INFO:                 clean  clean custom areaLOG_INFO: --update               Upgrade mode.
LOG_INFO: --partition=0x3FFC00   Set the partition to be upgraded.(NOTICE: OTA not support upgrade loader and parameter)
LOG_INFO:                        0x3FFC00: 0011 1111 1111 1100 0000 0000.
LOG_INFO:                                  uboot trust boot recovery rootfs oem
LOG_INFO:                                  uboot_a uboot_b boot_a boot_b system_a system_b.
LOG_INFO:                        100000000000000000000000: Upgrade loader
LOG_INFO:                        010000000000000000000000: Upgrade parameter
LOG_INFO:                        001000000000000000000000: Upgrade uboot
LOG_INFO:                        000100000000000000000000: Upgrade trust
LOG_INFO:                        000010000000000000000000: Upgrade boot
LOG_INFO:                        000001000000000000000000: Upgrade recovery
LOG_INFO:                        000000100000000000000000: Upgrade rootfs
LOG_INFO:                        000000010000000000000000: Upgrade oem
LOG_INFO:                        000000001000000000000000: Upgrade uboot_a
LOG_INFO:                        000000000100000000000000: Upgrade uboot_b
LOG_INFO:                        000000000010000000000000: Upgrade boot_a
LOG_INFO:                        000000000001000000000000: Upgrade boot_b
LOG_INFO:                        000000000000100000000000: Upgrade system_a
LOG_INFO:                        000000000000010000000000: Upgrade system_b
LOG_INFO:                        000000000000001000000000: Upgrade misc
LOG_INFO:                        000000000000000100000000: Upgrade userdata
LOG_INFO: --reboot               Restart the machine at the end of the program.
LOG_INFO: --version_url=url      The path to the file of version.
LOG_INFO: --image_url=url        Path to upgrade firmware.
LOG_INFO: --savepath=url         save the update.img to url.
LOG_INFO: --version              the version of updateEngine
LOG_INFO: --rkdebug              Log output to serial port
LOG_INFO: --ui_rotation          UI rotation,has 4 angles(0-3).
```
