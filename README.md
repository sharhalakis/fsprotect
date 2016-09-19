# Summary

fsprotect is a set of scripts, customized for debian systems that protect existing filesystems.

It uses the AUFS filesystem and some initramfs magic to protect the root filesystem. It also uses a simple init script to protect other filesystems as early as possible.

fsprotect is excellent for public computers like those in libraries, labs, etc. It will ease the life of all administrators with a couple of simple steps.

The benefits of using fsprotect are:

 * Filesystems are protected and no change is ever written to the disk
 * Protected filesystems are mounted read-only. This means that they aren't damaged when the computer is turned off improperly.
 * It is very easy to use. Just add an "fsprotect" parameter to the kernel for the root filesystem and list the filesystems to be protected in /etc/default/fsprotect.
 * In some cases it makes the filesystem access faster.

The drawbacks of using fsprotect:

 * Filesystem changes cannot be more than a predefined limit (set by you) (in bytes).
 * Since tmpfs is heavily used, you need to have adequate swap space.

# How it works

fsprotect uses AUFS to combine two filesystems in one. AUFS does exactly that: It merges two existing filesystems in one and distributes changes among them.

For each protected filesystem, fsprotect combines the existing filesystem with a tmpfs, forcing all changes to be written to the tmpfs. This means that nothing is ever written to the disks and all changes are stored in the tmpfs. tmpfs is a memory based filesystem, similar to ramdisk but using VM instead of real memory, allowing its contents to be swapped out.

The whole protection procedure is achieved with the following steps (assuming you want to protect /test). They are performed by fsprotect; you don't have to do any of them:

 1. There is a directory named /fsprotect. Three other directories are created inside it: /fsprotect/fs/test/orig, /fsprotect/fs/test/tmp and /fsprotect/fs/test/aufs
 2. `mount -t tmpfs -o size=XXXX none /fsprotect/fs/test/tmp`
 3. `mount -o bind /test /fsprotect/fs/test/orig`
 4. `mount -t aufs -o dirs=/fsprotect/fs/test/tmp=rw:/fsprotect/test/orig=ro none /fsprotect/test/aufs`
 5. `umount /test`
 6. `mount -o bind /fsprotect/fs/test/aufs /test`
 7. `umount /fsprotect/fs/test/aufs`
 8. `mount -o remount,ro /fsprotect/fs/test/orig`

The actual steps are somehow more complicated to take care of filesystems inside other filesystems.

## Boot

The protection of the root filesystem is a very special case since all other filesystems are mounted beneath it. For fsprotect to succeed, the above procedure needs to be run before the root filesystem is moved to /. To achieve this, fsprotect uses an initramfs script that runs very early in the boot process, after the root filesystem is mounted but before is is moved to /. It then exchanges the existing filesystem with an aufs and lets the boot procedure continue.

The protection of non-root filesystems is somehow easier. For each filesystem, fsprotect runs the above procedure exchanging the existing filesystem with an aufs.

# Installation

fsprotect is available as a debian package. It depends on aufs, so you'll need that too.

To install fsprotect do:

```
# apt-get update
# apt-get install fsprotect
```

After that:

 * Edit /boot/grub/menu.lst or /etc/default/grub2 or /etc/lilo.conf and add "fsprotect=1G" to kernel parameters. Modify 1G as needed. Apply changes (i.e. run update-grub)
 * Edit /etc/default/fsprotect if you want to protect filesystems other than /.
 * reboot

You may also want to password protect the grub bootloader or forbid any changes to it.

# Authors

Stefanos Harhalakis <v13@v13.gr>

# License

fsprotect is ditributed under the GPLv3 license

# Build

	cd fsprotect
	lyx -e pdf doc/fsprotect.lyx
	debuild -us -uc -b
