# uploaders

`loaders-update` is a proof of concept, a [sh(1)](https://man.freebsd.org/cgi/man.cgi?query=sh&sektion=1&manpath=freebsd-release) script for attention to: 

- the copy of the FreeBSD boot loader on an [EFI system partition](https://en.wikipedia.org/wiki/EFI_system_partition) (ESP)
- FreeBSD bootcode, where an ESP is either not present or unused.

The script can help to update these things.

Some people feel that automation is not possible, or not desirable.
Too complex, they say … so, I wrote this script.

### FreeBSD Project changes to the loader

Sometimes, these changes relate to changes to ZFS.

Somtimes not. An outdated loader might be problematic _without_ changes to ZFS; _without_ an upgrade to a ZFS pool; or with _UFS_ on root. 

## Usage and modes

`loaders-update mode [-m efi_mount_dir] [-s loaders_source_dir]`

`mode` can be one of:

* `show-me` – show but do not run commands (change nothing)
* `shoot-me` – run interactively (with the option of changing nothing).

### New

* option `-m` to specify the mount point of the ESP
  * default: `/mnt`
* option `-s `to specify the path to loader-related files
  * default: `/boot`.
   
## Use cases and capabilities

- AMD64 only
- [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table) (GPT) only
- BIOS boot
- [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) boot
- check all disks
  - RAID
  - non-RAID mirroring
- mount the ESP, if not already mounted by [fstab(5)](https://man.freebsd.org/cgi/man.cgi?query=fstab&sektion=5&manpath=freebsd-release)
- if loader-related files are present in `efi/`, list the files
- attempt to identify whether a loader-related file is FreeBSD-specific
  - if not specific, ignore the file
- if the FreeBSD loader on an ESP has a nonstandard filename (maybe changed by a system administrator), preserve the name
- for a freebsd-boot partition, compare its bootcode with bootcode in the OS partition
- if a detected loader is already up-to-date, neither suggest nor attempt an update.
  
### Out of scope

- ESPs that have insufficient space
  - some installations that originated with FreeBSD 12, or earlier, may have this limitation
- ESPs with no file system
  - [FreeBSD bug 258987](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=258987)
  - the script attempts to identify this limitation
- disks with two or more ESPs, disks with two or more freebsd-boot partitions
  - if more than one exists on any single disk, the script will work with the first one alone.

