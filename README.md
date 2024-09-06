# uploaders

This sh script is a proof of concept that an automatic update of FreeBSD loaders is possible, but it's also a tool that can help to update these loaders.

Update of the loaders is linked to the evolution of zfs features, but not only.
Even if no new features are added and the zpool(s) not upgraded, from time to time, some weird problems arise if you don't update.
Not to speak about the correction of the loaders bugs, which concerns also the systems with ufs on root.
Some people told me that an automatic update of the loaders isn't possible or, at least, not desirable.
Too complex is that thing, they said... So, I wrote this script to demonstrate the opposite.

It has two operating modes:  

Usage: ./up mode [-m efi_mount_dir] [-s loaders_source_dir]  
mode can be:  
* **show-me**: just show the commands to type, change nothing.  
* **shoot-me**: may update the loader(s), but ask for confirmation before each one.

**NEW!**
* It's now possible now to specify the directory where the efi partitions are mounted with the -m option (default = /mnt).
* It's also possible to set the directory where the loaders files reside with the -s option (default = /boot).

<br />
    
What it does handle:
- Checks all the disks reported by the system (case of mirror disks).
- EFI and BIOS loaders.
- EFI partitions mounted or not mounted.
- Checks for the presence of EFI loaders in efi/ and lists them all.
- Recognizes (or try) to identify FreeBSD EFI loaders and would update only them.
- Doesn't change the name of EFI FreeBSD loaders (case where the admin would have changed the default ones).
- In case of freebsd-boot partition, checks the coherence between its content and the root file system.
- Checks if each detected loader is already up to date and doesn't try or propose to update in this case.
  
What it doesn't handle:
- Other architecture than amd64.
- Disks that have other scheme than GPT (concerns mainly MBR scheme).
- Not enough room in the efi partition to copy the loader (can arise with installed version 12 or before and never updated the loader).
- Not formatted efi partition (see https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=258987). However, it tries to identify this case.
- More than one efi or freebsd-boot partition a disk. It examines only the first efi and freebsd-boot partition.

