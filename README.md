# uploaders

`loaders-update` is a tool designed to keep the FreeBSD bootcodes and loaders up-to-date.

## Usage and modes

`loaders-update mode [-befgnoqry] [-m efi_mount_dir] [-s loaders_source_dir]`

`mode` can be one of:

* `show-me` – show but do not run commands (change nothing)
* `shoot-me` – run interactively (with the option of changing nothing)
* `make-efi-failsafe` – not really a mode, but a command. Implant the EFI Fail Safe feature on the machine. Once done, the backup of the EFI loader will be automatic (see *About the EFI Fail Safe feature* below).

It has the following options:
* option `-b` scan only BIOS loaders (exclude EFI loaders).
* option `-e` scan only EFI loaders (exclude BIOS loaders).
* option `-f` won't check the freebsd-boot partition content for BIOS loaders update.
* option `-g` force to use 'gpart show' for disk detection.
* option `-o`  specify the directory where to save the BIOS loaders.
  * default: `/root`
* option `-n` don't backup the BIOS loaders.
* option `-q` quiet mode. No output to the console.
* option `-m` to specify the mount point of the ESP.
  * default: `/mnt`
* option `-r` won't check the root file system for BIOS loaders update.
* option `-s` to specify the path to loader-related files.
  * default: `/boot`
* option `-y` answer yes for shoot-me mode. Use with caution!

## Return codes
0: no error.  
1: an error occured.  
2: at least one loader isn't up-to-date.  
3: = 1 + 2.  

## What are we talking about?

The loaders (or bootcodes) are special pieces of software designed to start the OS when you reboot or power on the machine.
They aim to load some files from the root file system and execute them (which finally lead to run the kernel).
They change at each FreeBSD upgrade. The new loader files are then put in /boot. However, the currently used loaders are in some special locations:

- For UEFI booting, they are in an efi partition (msdosfs) in the shape of one or several files.
- For legacy BIOS booting, they are, for one part, in the first sector of a disk (pmbr) and in a freebsd-boot type partition, for the other part (gptboot or gptzfsboot).

The loaders in those special locations aren't updated during a FreeBSD upgrade.This is the goal of loaders-update to do that.

You may also leave the loaders as is, but in case of zfs pool upgrading, the OS won't boot anymore. There are some others problems that may arise if you left your loaders unchanged for too long. This is why a good practice is to systematically update them.

## Use cases and capabilities

- Architectures AMD64 and ARM64 only
- [GUID Partition Table](https://en.wikipedia.org/wiki/GUID_Partition_Table) (GPT) only
- BIOS boot
- [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) boot
- check all disks
- mount the ESP, if not already mounted by [fstab(5)](https://man.freebsd.org/cgi/man.cgi?query=fstab&sektion=5&manpath=freebsd-release)
- if loader-related files are present in `efi/`, list the files
- attempt to identify whether a loader-related file is FreeBSD-specific
  - if not specific, ignore the file
- for a freebsd-boot partition, compare its bootcode with the root file system
  - if not coherent, don't change the content of this freebsd-boot partition 
- if a detected loader is already up-to-date, neither suggest nor attempt an update
- Systematically save the current BIOS loaders before to update them (unless option -n is selected)
- If the EFI FailSafe feature is implanted, save the previous EFI loader before to update.
  
### Out of scope

- ESPs that have insufficient space
  - some installations that originated with FreeBSD 12, or earlier, may have this limitation
- ESPs with no file system
  - [FreeBSD bug 258987](https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=258987)
  - the script attempts to identify this limitation
- disks with two or more ESPs, disks with two or more freebsd-boot partitions
  - if more than one exists on any single disk, the script will work with the first one alone

### About the EFI Fail Safe feature

Allows to implant the EFI Fail Safe feature in the machine. It creates an EFI boot var (labeled `LU-FailSafe`) that points to `LU-old-loader.efi` in the main ESP.  At each EFI loaders update with the `shoot-me` mode, the previous loader is saved to `LU-old-loader.efi`.  In case of failure of the new EFI loader for starting, you can call the boot menu and choose `LU-FailSafe` entry.  

**Be aware that the manipulation of EFI vars can sometimes lead to some damages with bogus EFI firmware (typically on old machines).**

In case you want to remove this feature:  
           `# efibootmgr -v | grep LU-FailSafe`  

You will see a line beginning by `Bootxxxx`, note the `xxxx`, then:  
           `# efibootmgr -b xxxx -B`  
        
At this point, the feature is only partially removed. The EFI loader `LU-old-loader.efi` is still present in the ESP. When you will invoke the `shoot-me` mode, the previous loader will still be saved to `LU-old-loader.efi`. If you want to prevent this backup, mount your ESP (if not already mounted) and simply delete `LU-old-loader.efi`.

### About the BIOS loaders backup
By default, you will find them in `/root`, but you can change that with the `-o` option. Only the loaders that have been actually updated are saved. Their names are: `pmbr.geom` and `[gptzfsboot|gptboot].part_name`  

Where `geom` is the name of the disk (e.g. ada0) for pmbr and `part_name`, the partition name for gptzfsboot/gptboot (e.g. ada0p2).  
  
If you need to downgrade a given geom:  
`# gpart bootcode -b pmbr.geom geom`  
 and/or  
 `# gpart bootcode -p [gptzfsboot|gptboot].part_name -i index diskname`  
  (In the example, `diskname` is ada0, `index` is 2)




