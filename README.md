# uploaders

This sh script is a proof of concept that an automatic update of FreeBSD loaders is possible, but it's also a tool that can help to update these loaders.

Update of the loaders is linked to the evolution of zfs features, but not only.
Even if no new features are added and the zpool(s) not upgraded, from time to time, some weird problems arise if you don't update.
Not to speak about the correction of the loaders bugs, which concerns also the systems with ufs on root.
Some people told me that an automatic update of the loaders isn't possible or, at least, not desirable.
Too complex is that thing, they said... So, I wrote this script to demonstrate the opposite.

I admit, I put some serious limitations in this code, for I wanted to stay in the fields I know well.

What it does handle:
- Checks all the disks reported by the system (case of mirror disks).
- EFI and BIOS loaders.
- EFI partitions mounted or not mounted.
- Checks for the presence of EFI loaders both in efi/boot and efi/freebsd.
- Recognizes (or try) to identify FreeBSD EFI loaders and would update only them.
- Doesn't change the name of EFI FreeBSD loaders (case where the admin would have changed the default ones).
- In case of freebsd-boot partition, checks the coherence between its content and the root file system.
- Check if each detected loader may be updated or not (case where they are already up to date).

What it doesn't handle:
- Other architecture than amd64.
- Disks that have other scheme than GPT (concerns mainly MBR scheme).
- Not enough room in the efi partition to copy the loader (can arise with installed version 12 or before and never updated the loader).
- Not formatted efi partition (see https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=258987).
- More than one efi or freebsd-boot partition a disk. It examines only the first efi and freebsd-boot partition.

---------------------------------------------------------------------------------------------------------------------------------------

**There is a new feature!** I changed the name of the script: now, it's loaders-update.
Ok, it's not a feature, I was just kidding.

Changes:
- It's less verbose and sharper in the appreciation of the results, as asked by some people in the FreeBSD forum. That's not a feature as well.
- There is no more a prompt to start the script; no feature again.
- While by default, it updates nothing as before, just writing to the console the commands to type, there is a new mode: the **shoot-me** mode. If you write "shoot-me" as the first argument, it can execute these very commands but ask for confirmation before each one. This is THE feature.

I did this because I don't want to copy-paste commands. That's more cool to let the script do its job. Tried on several machines, VM and bare metal ones, without any problem.
