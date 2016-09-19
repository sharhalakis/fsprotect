# Summary

fsprotect is a set of scripts, customized for debian systems that protect existing filesystems.

It uses the OverlayFS filesystem and some initramfs magic to protect the root filesystem. It also uses a simple init script to protect other filesystems as early as possible.

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

fsprotect uses OverlayFS to combine two filesystems in one. OverlayFS does exactly that: It merges two existing filesystems in one and distributes changes among them.

For each protected filesystem, fsprotect combines the existing filesystem with a tmpfs, forcing all changes to be written to the tmpfs. This means that nothing is ever written to the disks and all changes are stored in the tmpfs. tmpfs is a memory based filesystem, similar to ramdisk but using VM instead of real memory, allowing its contents to be swapped out.

The whole protection procedure is achieved with the following steps (assuming you want to protect /test). They are performed by fsprotect; you don't have to do any of them:

 1. There is a directory named /fsprotect. Three other directories are created inside it: /fsprotect/fs/test/orig, /fsprotect/fs/test/tmp and /fsprotect/fs/test/overlay
 2. `mount -t tmpfs -o size=XXXX none /fsprotect/fs/test/tmp`
 3. `mkdir /fsprotect/fs/test/tmp/{upper,work}`
 4. `mount -o bind /test /fsprotect/fs/test/orig`
 5. `mount -t overlay -o upperdir=/fsprotect/fs/test/tmp/upper,lowerdir=/fsprotect/fs/test/orig,workdir=/fsprotect/fs/test/tmp/work overlay /fsprotect/test/overlay`
 6. `umount /test`
 7. `mount -o bind /fsprotect/fs/test/overlay /test`
 8. `umount /fsprotect/fs/test/overlay`
 9. `mount -o remount,ro /fsprotect/fs/test/orig`

**Note**: Recursion filesystem is not support.

## Boot

The protection of the root filesystem is a very special case since all other filesystems are mounted beneath it. For fsprotect to succeed, the above procedure needs to be run before the root filesystem is moved to /. To achieve this, fsprotect uses an initramfs script that runs very early in the boot process, after the root filesystem is mounted but before is is moved to /. It then exchanges the existing filesystem with an overlayfs and lets the boot procedure continue.

The protection of non-root filesystems is somehow easier. For each filesystem, fsprotect runs the above procedure exchanging the existing filesystem with an overlayfs.

# Installation

fsprotect is available as a debian package.

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

