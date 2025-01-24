# uploaders

`loaders-update` is a tool designed to keep the FreeBSD bootcodes and loaders up-to-date.

## Usage and modes

`loaders-update mode [-gr] [-m efi_mount_dir] [-s loaders_source_dir]`

`mode` can be one of:

* `show-me` – show but do not run commands (change nothing)
* `shoot-me` – run interactively (with the option of changing nothing)

It has the following options:

* option `-f` won't check the freebsd-boot partition content for BIOS loaders update.
* option `-g` force to use 'gpart show' for disk detection.
* option `-m` to specify the mount point of the ESP
  * default: `/mnt`
* option `-r` won't check the root file system for BIOS loaders update.
* option `-s` to specify the path to loader-related files
  * default: `/boot`

## What are we talking about?

The loaders (or bootcodes) are special pieces of software designed to start the OS when you reboot or power on the machine.
They aim to load some files in the root file system and execute them (which finally lead to run the kernel).
They change at each FreeBSD upgrade. The new loader files are then put in /boot. However, the currently used loaders are in some special locations:

- For UEFI booting, they are in an efi partition (msdosfs) in the shape of one or several files.
- For legacy BIOS booting, they are, for one part, in the first sector of a disk (pmbr) and in a freebsd-boot type partition, for the other part (gptboot or gptzfsboot).

The loaders in those special locations aren't updated during a FreeBSD upgrade.This is the goal of loaders-update to do that.

You may also leave the loaders as is, but in case of zfs pool upgrading, the OS won't boot anymore. There are some others problems that may arise if you left your loaders unchanged for too long. This is why a good practice is to systematically update them.

## Use cases and capabilities

- AMD64 only (because I don't have other hardware for testing)
- [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table) (GPT) only
- BIOS boot
- [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) boot
- check all disks
- mount the ESP, if not already mounted by [fstab(5)](https://man.freebsd.org/cgi/man.cgi?query=fstab&sektion=5&manpath=freebsd-release)
- if loader-related files are present in `efi/`, list the files
- attempt to identify whether a loader-related file is FreeBSD-specific
  - if not specific, ignore the file
- if the FreeBSD loader on an ESP has a nonstandard filename (maybe changed by a system administrator), preserve the name
- for a freebsd-boot partition, compare its bootcode with the root file system
  - if not coherent, don't change the content of this freebsd-boot partition 
- if a detected loader is already up-to-date, neither suggest nor attempt an update
  
### Out of scope

- ESPs that have insufficient space
  - some installations that originated with FreeBSD 12, or earlier, may have this limitation
- ESPs with no file system
  - [FreeBSD bug 258987](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=258987)
  - the script attempts to identify this limitation
- disks with two or more ESPs, disks with two or more freebsd-boot partitions
  - if more than one exists on any single disk, the script will work with the first one alone

