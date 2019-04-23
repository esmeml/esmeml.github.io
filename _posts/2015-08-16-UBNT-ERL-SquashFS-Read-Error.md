---
layout: post
title: "Ubiquiti EdgeRouter Lite SquashFS Block Read Error: Part 1"
license: cc0
date: 2015-08-16 00:00:00 -05:00
categories:
- Tutorials
- Networking
tags:
- EdgeRouter
- Repairs
---

## Problem Description ##
Upon booting the EdgeRouter Lite, the system will not boot. Connecting to
console reveals the following log:

```
SQUASHFS error: zlib_inflate error, data probably corrupt
SQUASHFS error: squashfs_read_data failed to read block 0x1b4574
SQUASHFS error: Unable to read fragment cache entry [1b4574]
SQUASHFS error: Unable to read page, block 1b4574, size 8519
SQUASHFS error: Unable to read fragment cache entry [1b4574]
SQUASHFS error: Unable to read page, block 1b4574, size 8519
```

This can be caused by an unclean shutdown, which could potentially occur with a
power failure. Due to the failure, parts of the internal ext3 filesystem
become corrupted, which causes the inability to load the SquashFS partition
which contains EdgeOS. The end result, of course, is a non-functional unit.

## Problem Severity ##
There are four potential levels to determining the severity of this issue:

* Can the filesystem be salvaged by fscking the ext3 partition until it is
  salvageable? (**Severity:** Annoying.)
* If the answer to the above is no, can the flash drive be saved by zeroing out
  the device and writing a new filesystem to it? (**Severity:** Hardware issue -
  End user repair possible..)
* If the answer to the above is no, is the device capable of booting? If yes, it
  is possible you may be suffering
  from the [EdgeRouter Lite RAM issue][1]. (**Severity:** Hardware issue -
  RMA the device.)
* If the answer to the above is no, is the device under warranty? If yes,
  (**Severity:** Hardware issue - RMA the device.) If no, (**Severity:** Device
  Replacement.)

## Problem Resolution ##
Resolutions will be broken down into two posts, one for each problem the user
could resolve without warranty work.

### Resolution Credits ###
A special thanks to Ubiquiti for providing a community support option via
their official website. Additionally, special thanks to all those users that
utilize said forum, your information was invaluable in helping me to fix my
EdgeRouter, and write this article.

## Resolution One: Repair the File System ##
This is by far the easiest resolution. Open the EdgeRouter Lite (**Caution:**
*Your warranty is now void. Depending on how soon you need the device
back in operation, it may be acceptable for you to void the warranty on a
$100USD device if you can get it back in service by the end of the day. If
you think your device may need to be RMA’d, do not open the device and proceed
to UBNT’s Support Site.*) by removing two screws on the bottom
of the device, located right under where the interface connections are. Your
device will slide apart, revealing the internal board. On the board you
will see a soldered on female USB port with a small silver USB flash drive,
roughly 3cm long. This is your 2GB of internal memory, but the device itself
is 2.6GB.

Remove the USB drive and plug it into a USB port on your Mac OSX or Linux
powered device. Open a terminal, and run dmesg. You are looking for the
devfs name of your device.

```
[  262.645401] scsi 2:0:0:0: Direct-Access                               5.00 PQ: 0 ANSI: 2
[  262.647109] sd 2:0:0:0: Attached scsi generic sg1 type 0
[  262.648527] sd 2:0:0:0: [sdb] 4057088 512-byte logical blocks: (2.07 GB/1.93 GiB)
[  262.648795] sd 2:0:0:0: [sdb] Write Protect is off
[  262.648817] sd 2:0:0:0: [sdb] Mode Sense: 0b 00 00 08
[  262.649095] sd 2:0:0:0: [sdb] No Caching mode page found
[  262.649114] sd 2:0:0:0: [sdb] Assuming drive cache: write through
[  262.652309]  sdb: sdb1
[  262.655018] sd 2:0:0:0: [sdb] Attached SCSI removable disk
```

In this case, my device name is `sdb`. Please note that for this guide I am not
using an actual EdgeRouter flash drive. This is a stand-in device. The next
thing to do is make sure none of the partitions on the device have been
automatically mounted. Where I used `sdb` in the following example, use whatever
dmesg identified as your USB device.

```
zyradyl@captor ~ $ mount | grep sdb
/dev/sdb1 on /media/zyradyl/b7bcf200-26a1-41ed-9122-625558dbc907 type ext4 (rw,nosuid,nodev,relatime,data=ordered,uhelper=udisks2)
```

This would indicate that at least one of the partitions on the disk has been
mounted. To correct this we need to use the `umount` command. Note that in
some operating systems an eject button exists in the taskbar. In my experience,
not only do these buttons unmount the mounted partitions, it also removed the
device entry from the kernel. So I prefer to use manual `umount` commands. In
the following example, do one line per mounted partition, substituting the
device names of your partitions where I specified `/dev/sdb1`.

```
zyradyl@captor ~ $ sudo umount /dev/sdb1
```

Once you have unmounted the devices, you can use fdisk to get a good view
of the partition layout on your USB device. I like to do this even if I know
what I should expect, just as a form of a sanity test to make sure I have the
right device. Note that, as previously stated, this device is a stand in. I have
tried to recreate the partition table to the best of my memory.

```
zyradyl@captor ~ $ sudo fdisk -l /dev/sdb

Disk /dev/sdb: 2 GiB, 2077229056 bytes, 4057088 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd9dd6356

Device     Boot  Start     End Sectors  Size Id Type
/dev/sdb1         2048  264191  262144  128M  b W95 FAT32
/dev/sdb2       264192 4057087 3792896  1.8G 83 Linux
```

So we can see from the above that we have two partitions on the USB device. In
your case, when using an actual EdgeRouter flash drive, one of the partitions
will be a vfat partition type, and the second one will be a linux
partition type. Now that we know where our partitions are, and that they are
safely unmounted, we can use fsck. Remember to replace my instance of `sdb2`
with the correct partition for your device. Note that depending on the damage to
the filesystem, this has the potential to be a **destructive operation**.

```
zyradyl@captor ~ $ fsck.ext3 -fvy /dev/sdb2
```

Allow the program to run till completion. If you notice the command has become
looped, you will need to clear the journal out on your device. Note that
this is a **potentially destructive operation** to data that had not been
written from the journal to the disk. The first thing that we need to do is
clear out the recovery indicator using `debugfs` Remember to replace `sdb2`
with the proper device name of your ext3 partition.

```
zyradyl@captor ~ $ debugfs -w -R “feature ^needs_recovery” /dev/sdb2
```

After clearing out this flag, it is now possible to use `tune2fs` to force
removal of the journal.

```
zyradyl@captor ~ $ tune2fs -f -O ^has_journal /dev/sdb2
```

Now you can go back up and run the `fsck` command. Once that is done, you will
need to re-enable the journal on your filesystem.

```
zyradyl@captor ~ $ tune2fs -f -O has_journal -j size=128M /dev/sdb2
```

Once you have a clean fsck output, and you have your journal enabled again, plug
the USB device back into the EdgeRouter and boot. If you are lucky, everything
will come back as it should. As there may be data loss, make sure to restore
from a recent configuration backup (you **do** have configuration backups,
right?) and reboot, before logging into the device through ssh to make sure
everything is where it should be.

If you are still having problems, stick around for part two of this guide, where
I will be walking you through using a separate host computer to recreate the
filesystem.

__EDIT (2018/12/09):__ _Part 2 was never written, but this is actually a fairly
common issue with EdgeRouter Lites. A simple Google search should turn up the
EdgeRouter Emergency Recovery Kit GitHub, and that plus some posts on the UBNT
forums should help you solve the problem. Sorry about my inability to deliver
on what I mention in my posts._

## External Links & Resources ##
[How to Recover an EXT3 Volume with an Unreadable Journal][2]

[1]:  https://community.ubnt.com/t5/EdgeMAX/EdgeRouter-LITE-OS-and-hardware-problems/td-p/667557
[15]: https://trick.vanstaveren.us/wp/2009/06/19/how-to-recover-an-ext3-volume-with-an-unreadable-journal/
