---
layout: post
title: ThothBackup - Part 3
date: 2018-12-09 17:00:00 -06:00
license: cc0
published: True
categories:
- ThothBackup
tags:
- BorgBackup
- Git
- Git-Annex
- Wasabi
- SyncThing
---
So, another week has gone, and it is time to update this blog with what I have
learned. Unfortunately, experiments were not able to be run this week in the
realm of data transfer. I decided to revisit the base system to focus on
encrypting backup data while it is at rest on the system. This was one of the
remaining security vulnerabilities with this process. While end-users still have
to trust _me_, they can at least be assured the data is encrypted at rest.

Essentially, if the system was ever stolen, or our apartment door was broken
down, we would just have to cut power and the data would be good. With that
previous statement, please keep in mind that this week's post only refers to the
root drive. I didn't make much progress because of things happening at work, but
this is a nice, strong, foundation to build upon.

Many of the steps in this post were cobbled together from various sources across
the internet. At the bottom of this post you can find a works cited that will
show the posts that I used to gather the appropriate information.

## End Goal ##
The end goal is to ensure that the operating system's root drive is encrypted at
rest. Full Disk Encryption is _not_ an active security measure, it is a passive
one. It is primarily there to ensure that should the system ever be stolen, it
would not be readable. The root partition will not host any user data, so the
encryption should be transparent and seamless.

In short, we will utilize a USB key to provide a Keyfile which will then be
combined with LUKS encryption to unlock the LVM array to allow the initramfs to
hand over control to the operating system.

## Notes ## {: #thoth-3-notes }
Because we are using a solid state drive, and we will be filling the drive with
data, it was important for me to over-provision the drive. The SSD we're using
comes with 240GB of space. We can assume that there is some form of manufacturer
over-provisioning in play to get that number, if I had to guess I would assume
there is actually 256GB of NAND memory on the drive, but only 240GB are made
available to the user. This is a fairly reasonable level of over-provisioning.

However, with us planning to fill the drive with pseudorandom data in order to
obfuscate the amount of data actually in use, this 16GB could potentially be
used quite quickly. SSDs cannot actually rewrite sectors on the fly, they have
to run a READ/ERASE/WRITE cycle. This is typically done by writing the new block
to an over-provisioned area and then pointing the drive's firmware at that
block. In this way we avoid the ERASE penalty, which can be on the order of 0.5
seconds per block.

Essentially then, every single write to the drive will require a
READ/ERASE/WRITE cycle, so padding the over-provisioning is a very good idea. It
will help with wear leveling and prevent severe write amplification, while also
making the drive "feel" faster.

## Prior Work ##
Before we get into the new installation, we need to prepare the drive for its
new role. Unless the flash cells are at their default state, the firmware will
regard them as holding data and will not utilize them for wear leveling, thus
rendering the over-provisioning useless.

To begin, boot the system via a Debian Live-CD and open up a root prompt using
`sudo`.

If you, like me, prefer to work remotely, you will then need to run a sequence
of commands to prep the system for SSH access. We need to add a password to the
liveCD user, then install openSSH, and finally start the service. Once all of
this is complete, you can log in from a more comfortable system.

```
# apt-get update
# apt-get install openssh-server
# passwd user
# systemctl start sshd
```

We will need to install one last software package, `hdparm`. Run
`apt-get install hdparm` to grab it. Once you have done so, run
`hdparm -I /dev/sda`. Under "Security" you are looking for the words "__not__
frozen". If it says frozen, and you are working remotely, you will need to
access the physical console to suspend/resume the machine. This should
unfreeze access to the ATA security system.

The first thing we need to do is to run an ATA Enhanced Erase. After this is
done, I still like to run `blkdiscard` just to make sure every sector has been
marked as empty. Finally, we will use `hdparm` to mark a host-protected-area,
which the drive firmware will be able to use as an over-provisioning space.
To calculate the HPA size, figure out what size you want to be available to
_you_. Convert that into bytes, and divide by 512, which is the sector size.
This will give you the number to pass to `hdparm`.

```
# hdparm --user-master u --security-set-pass Eins /dev/sda
# hdparm --user-master u --security-erase-enhanced Eins /dev/sda
# blkdiscard /dev/sda
# hdparm -Np390625000 --yes-i-know-what-i-am-doing /dev/sda
# reboot
```

Once this is done __reboot immediately__. There is a lot that can go wrong if
you fail to reboot. At this point, I swapped out my disk for the Debian
installer. If you are doing this on your own 2006-2008 MacMini, you may want
to use the AMD64-mac ISO that the Debian project provides.

From here, we just have to confirm that the drive shows up how we want in the
installer (200GB in size, in my case), and we can proceed with the installation.

## Installation ##
Most of the Debian installation process is self explanatory. The only point
where I will interject is partitioning. Because of the way the MacMini2,1
boots, it is important that we use an MBR based grub installation. You _can_ do
a 32bit EFI installation, but it is very fragile, and I'm not a fan of fragile
things. That being said, I still wanted the ability to use GPT partitions. I
like being able to label everything from the partition up to the individual
filesystems.

Accomplishing this is actually fairly easy anymore. You just need to create a
1MB `grub_bios` partition as part of your scheme and you're good to go. To get
the level of control we need, we will select manual partitioning when prompted
to set up our partitions in the installer.

Create a new partition table (This will default to GPT), and then lay out your
initial partition layout. It will look something like this:

```
<PART #>  <SIZE>  <NAME>          <FILESYSTEM>  <FLAGS>
#1        1MB     BIOS_PARTITION  none          grub_bios
#2        1GB     BOOT_PARTITION  ext4          bootable
#3        199GB   ROOT_PARTITION  crypto        crypto
```

When you select "Physical Volume For Encryption" it will prompt you to configure
some features. You can customize the partition there, but I actually wanted more
options than the GUI provided, so I accepted the defaults and planned to
re-encrypt later. Please make sure to allow the installer to write encrypted
data to the partition. Since we have already set up a customized HPA, a
potential attacker already knows the maximum amount of cipher text that can
be present, and if the HPA is disabled they would likely be able to gain
access to more. Therefore, it is important that we take every possible
precaution.

Once this is done, you should scroll to the top where it will say "Configure
Encryption" or something similar. Select this option, then select the physical
volume we just set up, and it should drop you back to the partitioning menu.
This time, however, you will be able to see the newly unlocked crypto partition
as something that we can further customize.

Select that volume and partition it like so:

```
<PART #>  <SIZE>  <NAME>          <FILESYSTEM>  <FLAGS>
#1        199GB                   none          lvm
```

The LVM option will show up in the menu as "Physical Volume for LVM." From here,
we go back up to the top of our menu and select "Configure Logical Volume
Manager." You will then be taken to a new screen where it should show that you
have one `PV` available for use. Create a new volume group that fills the entire
`PV` and name it as you would like. For this project, I named it `djehuti-root`
and completed setup.

Next we need to create a Logical Volume for each partition that you would like
to have. For me, this looked like the following:

```
<Logical Volume>  <Size>  <Name>
#1                30GB    root-root
#2                25GB    root-home
#3                10GB    root-opt
#4                05GB    root-swap
#5                05GB    root-tmp
#6                10GB    root-usr-local
#7                10GB    root-var
#8                05GB    root-var-audit
#9                05GB    root-var-log
#10               05GB    root-var-tmp
```

Your layout may be similar. Once this is done, you can exit out and you will
see that all of your logical volumes are now available for formatting. Since I
wanted to stick with something stable, and most importantly resizable (more on
why later), I picked `ext4` for all of my partitioning. We will tweak mount
options later. For now, the end product looked like the following:

```
<PARTITION>                       <FS>    <MOUNT POINT> <MOUNT OPTIONS>
/dev/sda2                         ext4    /boot         defaults
/dev/djehuti-root/root-root       ext4    /             defaults
/dev/djehuti-root/root-home       ext4    /home         defaults
/dev/djehuti-root/root-opt        ext4    /opt          defaults
/dev/djehuti-root/root-swap       swapfs  none          defaults
/dev/djehuti-root/root-tmp        ext4    /tmp          defaults
/dev/djehuti-root/root-usr-local  ext4    /usr/local    defaults
/dev/djehuti-root/root-var        ext4    /var          defaults
/dev/djehuti-root/root-var-audit  ext4    /var/audit    defaults
/dev/djehuti-root/root-var-log    ext4    /var/log      defaults
/dev/djehuti-root/root-var-tmp    ext4    /var/tmp      defaults
```

Once everything is setup appropriately, follow through the installation until
you get to the `task-sel` portion. You really only want to install an ssh server
and the standard system utilities pack. Once the installation completes, reboot
into your server and make sure everything boots appropriately. We're going to be
doing some offline tweaking after this point, so ensuring that everything is
functioning as is will save you a lot of headache.

Once you are satisfied the initial installation is functioning and booting
correctly, it is time to move on to re-encrypting the partition with our own
heavily customized parameters.

## Re-Encryption ##
This process isn't so much difficult as it is simply time consuming. Go ahead
and reboot your system to the boot media selection screen. You will want to
swap out your Debian Installation CD for the Debian LiveCD that we used earlier.
Once the disks have been swapped, boot into the live environment and then
bring up a shell. We will first need to install the tools that we will use, and
then run the actual command. The command is actually fairly self explanatory,
so I won't explain that, but I will explain the reasoning behind the parameters
below.

```
# apt-get update
# apt-get install cryptsetup
# cryptsetup-reencrypt /dev/sda3 --verbose --use-random --cipher serpent-xts-plain64 --key-size 512 --hash whirlpool --iter-time <higher number>
```

So, onto the parameters:

  * _cipher_    - I picked Serpent because it is [widely acknowledged][1] to be
                  a more "secure" cipher. Appropriate text from the above link
                  is as follows: "The official NIST report on AES competition
                  classified Serpent as having __a high security margin__ along
                  with MARS and Twofish, __in contrast to the adequate security
                  margin of RC6 and Rijndael (currently AES)__." The speed
                  trade-off was negligible for me, as the true bottleneck in the
                  system will be network speed, not disk speed.
  * _key-size_  - The XTS algorithm requires double the number of bits to
                  achieve the same [level of security][2]. Therefore, 512 bits
                  are required to achieve an AES-256 level of security.
  * _hash_      - In general, I prefer hashes that have actually had extensive
                  cryptanalysis performed to very high round counts. The best
                  example of an attack on whirlpool, with a worst case situation
                  where the attacker controls almost all aspects of the hash,
                  the time complexity is still 2^128th on 9.5 of 10 rounds. This
                  establishes a _known_ time to break of over 100 years.
  * _iter-time_ - The higher your iteration time, the longer it takes to unlock,
                  but it also makes it harder to break the hash function. So if
                  we combine what we know above with a large iteration time, we
                  gain fairly strong security at the expense of a long unlock
                  time when using a passphrase.

Once these specifications have been entered, you simply need to press enter and
sit back and relax as the system handles the rest. Once this process is
complete, you should once again reset and boot into the system to verify that
everything is still working as intended. If it is, you are ready for the next
step, which is automating the unlock process.

## Auto-Decryption ##
There are a few ways to handle USB key based auto-decryption. The end goal is
to actually use a hardware security module to do this, and I don't anticipate
the FBI busting down my door any time soon for hosting the data of my friends
and family, so I opted for one that is easily extendable.

Essentially, the key will live on an `ext4` filesystem. It will be a simple
hidden file, so nothing extremely complex to find. This shouldn't be considered
secure at _this point_, but it is paving the way to a slightly more secure
future.

The first thing that I did, though it isn't strictly necessary, is write random
data to the entire USB stick. In my case, the USB drive could be found at
`/dev/sdb`.

```
# dd if=/dev/urandom of=/dev/sdb status=progress bs=1M
```

Once this is done, we've effectively destroyed the partition table. We will
recreate a GPT table, and then create a partition that fills the usable space
of the drive.

```
# apt update
# apt install parted
# parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart KEYS ext4 0% 100%
(parted) quit
```

Now we just create the filesystem, a mount point for the filesystem, and make
our new LUKS keyfile. Once the file has been created, we just add it to the
existing LUKS header.

```
# mkfs.ext4 -L KEYS /dev/sdb1
# mkdir /mnt/KEYS
# mount LABEL=KEYS /mnt/KEYS
# dd if=/dev/random of=/mnt/KEYS/.root_key bs=1 count=4096 status=progress
# cryptsetup luksAddKey /dev/sda3 /mnt/KEYS/.root_key
```

After this point, the setup diverges a bit depending on what guide you follow.
We will stick close to the guide posted to the Debian mailing list for now, as
that guide got me a successful boot on the first try. The others are slightly
more elegant looking, but at the expense of added complexity. As such, they may
end up being the _final_ configuration, but for this prototyping phase they are
a bit excessive.

We have to modify the `crypttab` file to enable the keyfile to be loaded off of
our freshly set up key drive.

```
sda3_crypt  UUID="..."  /dev/disk/by-label/KEYS:/.root_key:5  luks,initramfs,keyscript=/lib/cryptsetup/scripts/passdev,tries=2
```

At this point, we need to repackage our startup image, update grub, and reboot
to test the whole package.

```
# update-initramfs -tuck all
# update-grub
# reboot
```

At this point the system should boot automatically, but you will notice a weird
`systemd` based timeout that happens. This is mentioned in the guide posted to
the Debian Stretch mailing list, and is fairly easy to solve. We just need to
create an empty service file to prevent `systemd` from doing it's own thing.

```
# touch "/etc/systemd/system/systemd-cryptsetup@sda3_crypt.service"
# reboot
```

At this point, everything should boot correctly and quickly. You may notice a
few thrown errors, but it shouldn't be anything severe, more services loading
out of order.

At this point, it used to be possible to allow for the creation of a fallback
in the event that the key drive wasn't present, but that seems to have been
removed. I plan to look into it further when I have more time.

## Conclusion ## {: #thoth-3-conclusion }
This concludes the first part of the Operating System setup process. The next
step was originally planned to be thin-provisioning the partitions inside the
`djehuti-root` volume group, but there seems to be some problems in getting
the system to boot from a thin-provisioned root. I'm looking into a weird
combined system, where the root is static but all the accessory partitions are
thinly provisioned, but it will take time to tinker with this and report back.

Thin Provisioning isn't strictly required, but it is a rather neat feature and
I like the idea of being able to create more partitions than would technically
fit. I'm not sure when this would be useful, but we will see.

Once all of this is finalized, we will move on to hardening the base system,
and last but not least creating the Stage 1 Project page. Then it is back to
experiments with data synchronization. This is a fairly large step back in
progress, but I am hopeful it will result in a better end product, where
security can be dynamically updated as needed.

## Works Cited ##
The following sources were invaluable in cobbling this process together. I
sincerely thank the authors both for figuring the process out and documenting
the process online.

  * [Debian Stretch - USB Keyfile with LUKS][3]
  * [Arch Wiki - Secure Disk Wiping][4]
  * [Arch Wiki - Encrypting an Entire Disk][5]
  * [Gentoo Wiki - Sakaki's EFI Install Guide][6]
  * [Chroot Into Broken Linux Install][7]
  * [Linux Images for 32Bit EFI Macs][8]
  * [Reducing 30 Second Delay when Booting Linux][9]
  * [Over-Provisioning SSDs][10]

[1]:  https://en.wikipedia.org/wiki/Serpent_(cipher)
[2]:  https://en.wikipedia.org/wiki/Disk_encryption_theory#XTS
[3]:  https://lists.debian.org/debian-user/2017/12/msg00523.html
[4]:  https://wiki.archlinux.org/index.php/Securely_wipe_disk
[5]:  https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system
[6]:  https://wiki.gentoo.org/wiki/Sakaki%27s_EFI_Install_Guide/Preparing_the_LUKS-LVM_Filesystem_and_Boot_USB_Key
[7]:  https://aaronbonner.io/post/21103731114/chroot-into-a-broken-linux-install
[8]:  https://mattgadient.com/2016/07/11/linux-dvd-images-and-how-to-for-32-bit-efi-macs-late-2006-models/
[9]:  https://mattgadient.com/2018/02/12/reducing-the-30-second-delay-when-starting-64-bit-ubuntu-in-bios-mode-on-the-old-32-bit-efi-macs/
[10]: https://support.siliconmechanics.com/portal/kb/articles/over-provisioning-ssds
