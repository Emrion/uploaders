# uploaders

This sh script is a proof of concept that an automatic update of FreeBSD loaders is possible.

Update of the loaders is linked to the evolution of zfs features, but not only.
Even if no new features are added and the zpool(s) not upgraded, from time to time, some weird problems arise if you don't update.
Not to speak about the correction of the loaders bugs, which concerns also the systems with ufs on root.
Some people told me that an automatic update of the loaders isn't possible or, at least, not desirable.
Too complex is that thing, they said... So, I wrote this script to demonstrate the opposite.

I admit, I put some serious limitations in this code, for I wanted to stay in the fields I know well.

**It updates nothing but tells what it would do.**

What it does handle:
- Checks all the disks reported by the system (case of mirror disks).
- EFI and BIOS loaders.
- EFI partitions mounted or not mounted.
- Checks for the presence of EFI loaders both in efi/boot and efi/freebsd.
- Recognizes (or try) to identify FreeBSD EFI loaders and would update only them.
- Doesn't change the name of EFI FreeBSD loaders (case where the admin would have changed the default ones).
- In case of freebsd-boot partition, checks the coherence beetween its content and the root file system.

What it doesn't handle:
- Other architecture than amd64.
- Disks that have other scheme than GPT (concerns mainly MBR scheme).
- Not enough room in the efi partition to copy the loader (can arise with installed version 12 or before and never updated the loader).
- More than one efi or freebsd-boot partition a disk. It examines only the first efi and freebsd-boot partition.

Please, try out this sh script on your machines and report back the encountered problems along with your detailed configuration.
