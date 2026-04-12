# [Debian 13 - Desktop Laptop](../Readme.md) > Disk Encryption

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

## Backup

As soon as Key Files are in use, those should be stored as a backup at a safe place.

Even Passphrases should be written down and stored at a safe place.

```console
dd if=/dev/sdb of=/dev/sdc bs=4M status=progress
```

```console
dd if=/dev/sdb1 of=/dev/sdc1 bs=4M status=progress
dd if=/dev/sdb2 of=/dev/sdc2 bs=4M status=progress
```

## System Root Partition (`/`)

The goal is to setup Debian with an encrypted primary partition (`\`) on the primary disk drive and UEFI (`/boot/efi`) & EXT2 (`/boot`) partition on an USB-Stick. In order to boot the system, the USB-Stick¹ has to be plugged in.

There are plenty of other instructions out there. This one is specific to prevents the system being started without a special USB-Stick.

- encrypted System Root Partition (`/`)
- boot using USB-Stick
- unlock LUKS2 either using Passphrase (Default) or Key File (Automatic).

After the setup of the system, the partition has to be unlocked - the default way - by entering the Passphrase.

The relevant data in this case is.

- 128GB Primary Disk: mmcblk1 - 125.1 GB MMC DA4128
- 64GB USB-Stick: sdb - 64.2 GB Samsung Type-C

Note: To reduce wear of the flash disks, use `ext2` for the Linux partitions with mount options `noatime`,`nodiratime`,`relatime` enabled.

¹ Keep in mind, before the kernel will be updated ... the USB-Stick has to be plugged in and mount at `/boot` and `/boot/efi` respectively.

### Debian installation

Start the installation from a Debian 13 install image. Continue as normal until the disk has to be partitioned.

The partitioning starts with dialog "**Partitioning disks**".

- ✏️✓ select `Yes` on question "*Force UEFI installation?*"
- ✓ press ENTER
- ✏️✓ select `Manual` on "*Partitioning method*"
- ✓ press ENTER

The USB-Stick has to be formated in the following way.

- ✏️✓ create a ~100MB EFI System Partition "**ESP**" (with boot flag)
- ✏️✓ create a 3.5GB ext2 Linux partition "**Linux Boot**"
- leave the rest of the disk unused (at least for the moment)

In the dialog present the partitions of the USB-Stick in the following way.

```code
:
⛛ ScS12 (0,0,0) (sdb) - 64.2 GB Vendor Product
    >                           1.0 MB        FREE SPACE
    >       #1                 82.8 MB        ESP
    >       #2                  3.5 GB        ext2                 /boot
    >                          60.6 GB        FREE SPACE
:
```

The primary disk should have no partitions. Delete existing ones.

Before setting up the encrypted disk, it will look like the following way.

```code
:
⛛ MMC/SD card #1 (mmcblk0) - 125.1 MMC DA4128
    >                       125.1 GB        FREE SPACE
:
```

- select `Configure encrypted volumes`
- ⬆ press ENTER
    - ✏️✓ select `Yes` on question "*Write the changes to disk and configure encrypted volumes?*"
    - ✓ press ENTER
    - ✏️✓ select `Create encrypted volumes` on "*Encryption configuration action*"
    - press ENTER
    - ✏️✓ enable the primary disk (e.g. `/dev/mmcblk0`) on "*Device to encrypt*"
    - press TAB followed by ENTER
    - ✏️✓ select `Done setting up the partition` on "*Partition Settings*" (Encryption)
    - press ENTER
    - ✏️✓ select `Yes` on question "*Write the changes to disk and configure encrypted volumes?*"
    - ✓ press ENTER
    - ✏️✓ select `Finish` on "*Encryption configuration action*"
    - ✓press ENTER
    - ✏️✓ enter the passphrase for the encrypted partition ("*Encryption passphrase*")
    - ✓ press ENTER
- adjust the partition of `mmcblk0p0_crypt`
    - change partition type from `ext4` to `ext2`
    - specify mount point `/ - the root file system`
    -

In the dialog present the partitions of the primary disk in the following way.

```code
:
⛛ Encrypted volume (mmcblk0p0_crypt) - 125.1 GB Linux device-mapper (crypt)
    >       #1                125.1 GB   f  ext2
:
⛛ MMC/SD card #1 (mmcblk0) - 125.1 MMC DA4128
    >                         1.0 MB        FREE SPACE
    >       #1              125.1 GB     K  crypto                     (mmcblk0p0_crypt)
    >                         1.0 MB        FREE SPACE
:
```

From here on continue as usual.

### Post-Installation Steps

Once the setup is finished, some additional steps can be done to make life easier/safer.

#### Umount USB-Stick

Especially with a portable device, we want to remove USB-Stick after the boot phase¹.

¹ The boot phase is finished once the user is greeted by a login dialog.

In `fstab` add `noauto` to the mount parameters.

=== `/etc/fstab` ===

```code
:
# /boot was on /dev/sdc2 during installation
UUID=bc891c3e-7e8a-11f0-89ce-5f87147a4d2e /boot           ext2    noauto,noatime,nodiratime,relatime 0       2
# /boot/efi was on /dev/sdc1 during installation
UUID=7E8A-11F0  /boot/efi       vfat    noauto,umask=0077      0       1
:
```

#### Automatic Unlock

To avoid entering the Passphrase every time the system is booted, the USB-Stick can be prepared in such a way it unlocks with a key file. A key file, stored on the same USB-Stick used for booting the system.

This instruction is using `mmcblk0p1` as the partition for the encrypted System Root Partition(`/`).
Verify the chosen device is the right one. Looks a the output of `lsblk` and focus on the lines with the `/` at the end.

```console
# lsblk
:
mmcblk0             259:0    0   3.6T  0 disk  
└─mmcblk0p1         259:1    0   3.6T  0 part  
  └─mmcblk0p1_crypt 254:0    0   3.6T  0 crypt /
:
```

First backup the files, which are going to be modified.

```console
cp /etc/cryptsetup-initramfs/conf-hook /etc/cryptsetup-initramfs/conf-hook.ORG
cp /etc/crypttab /etc/crypttab.ORG
cp /etc/initramfs-tools/initramfs.conf /etc/initramfs-tools/initramfs.conf.ORG
```

Second .. make sure the partitions of the USB-Stick are mounted.

```console
mount /boot
mount /boot/efi
```

Create the Key File `primary.key` (for the primary disk), which is going to be used as an alternative to the Passphrase.

```console
mkdir --mode=700 /boot/keys
dd if=/dev/urandom of=/boot/keys/primary.key bs=512 count=16
chmod 700 /boot/keys/primary.key
```

Define the Key File `primary.key` as additional key for the System Root Partition(`/`).

```console
cryptsetup luksAddKey /dev/mmcblk0p1 /boot/keys/primary.key
```

Modify `crypttab`, by specify the Key File for the partition.

```console
echo "About to change 'crypttab', replacing 'none' by '/boot/keys/primary.key'."
if [ 1 -eq `wc -l /etc/crypttab | wc -l` ]; then
  sed --in-place --expression="s/ none luks,/ \/boot\/keys\/primary.key luks,/" /etc/crypttab
  echo "DONE"
else
  echo "There is more than one line in 'crypttab'. The change has to be done by hand."
  echo "The given example of the 'original' and 'modified' version will help you."
fi
```

Some configuration are required to ensure the initrd gets always created with the proper adjustments to unlock the encrypted partition.

Note: *The script will take over only the line of `crypttab`, which is related to the System Root Partition (`/`). This will ensure a.) no other encrypted partition will be decrypted during boot and b.) no other Key File will be stored on the USB-Stick.*

```console
sed --in-place --expression='s/^#KEYFILE_PATTERN=$/KEYFILE_PATTERN="\/boot\/keys\/*.key"/' /etc/cryptsetup-initramfs/conf-hook
cat << EOF >> /etc/initramfs-tools/initramfs.conf

UMASK=0077
EOF
cat << EOF > /etc/initramfs-tools/hooks/publish_crypttab.sh
#!/bin/sh
grep --regexp="^mmcblk0p1_crypt UUID=" /etc/crypttab > "${DESTDIR}/cryptroot/crypttab"
exit 0
EOF
```

Finally it is time to update the initrd of the existing boot images.

```
update-initramfs -u -k all
```

Note: *Watch out for error/warning!*

##### Change Passphrase

Assuming that

- the first key slot is used for the initial passphrase
- the second key slot is used for the key file

```console
# cryptsetup --verbose open --test-passphrase /dev/sda1
No usable token is available.
Enter passphrase for /dev/sda1: 
Key slot 0 unlocked.
Command successful.
# cryptsetup luksChangeKey /dev/sda1 -S 0
```

##### Backup LUKS Header

Just for the worst case we do create a backup of the LUKS header.

```console
cryptsetup luksHeaderBackup /dev/mmcblk0p1 --header-backup-file /boot/$HOSTNAME-mmcblk0p1-luks-header-$(date +'%y%m%d').img
```

##### Check initrd

The following command wil extract the initrd, shows the content of `crypttab` and the file `primary.key`.

During the creation of initrd, the process had adjusted the Key File path and refers now to directory `/cryptroot/keyfiles/`.

```console
cd /tmp
mkdir initrd
cd initrd
unmkinitramfs /boot/initrd.img-6.12.38+deb13-amd64 .
cat main/cryptroot/crypttab
ls main/cryptroot/keyfiles/primary.key
```

=== `/etc/crypttab` (original) ===

```code
mmcblk0p1_crypt UUID=3d51eafa-7f6f-11f0-8afe-2f1d5398f29d none luks,discard,x-initrd.attach
```

=== `/etc/crypttab` (modified) ===

```code
mmcblk0p1_crypt UUID=3d51eafa-7f6f-11f0-8afe-2f1d5398f29d /boot/keys/primary.key luks,discard,x-initrd.attach
```

=== `/cryptroot/crypttab` (of initrd) ===

```code
mmcblk0p1_crypt UUID=3d51eafa-7f6f-11f0-8afe-2f1d5398f29d /cryptroot/keyfiles/primary.key luks,discard,x-initrd.attach
```

=== `/etc/cryptsetup-initramfs/conf-hook` (original) ===

```code
:

# If KEYFILE_PATTERN if null or unset (default) then no key file is
# copied to the initramfs image.
#
# Note that the glob(7) is not expanded for crypttab(5) entries with a
# 'keyscript=' option.  In that case, the field is not treated as a file
# name but given as argument to the keyscript.
#
# WARNING:
# * If the initramfs image is to include private key material, you'll
#   want to create it with a restrictive umask in order to keep
#   non-privileged users at bay.  For instance, set UMASK=0077 in
#   /etc/initramfs-tools/initramfs.conf
# * If you use cryptsetup-suspend, private key material inside the
#   initramfs will be in memory during suspend period, defeating the
#   purpose of cryptsetup-suspend.
#

#KEYFILE_PATTERN=

:
```

=== `/etc/cryptsetup-initramfs/conf-hook` (modified) ===

```code
:

# If KEYFILE_PATTERN if null or unset (default) then no key file is
# copied to the initramfs image.
#
# Note that the glob(7) is not expanded for crypttab(5) entries with a
# 'keyscript=' option.  In that case, the field is not treated as a file
# name but given as argument to the keyscript.
#
# WARNING:
# * If the initramfs image is to include private key material, you'll
#   want to create it with a restrictive umask in order to keep
#   non-privileged users at bay.  For instance, set UMASK=0077 in
#   /etc/initramfs-tools/initramfs.conf
# * If you use cryptsetup-suspend, private key material inside the
#   initramfs will be in memory during suspend period, defeating the
#   purpose of cryptsetup-suspend.
#

KEYFILE_PATTERN="/boot/keys/*.key"

:
```

=== `/etc/initramfs-tools/hooks/publish_crypttab.sh` ===

```sh
#!/bin/sh
cp /etc/crypttab "${DESTDIR}/cryptroot/crypttab"
exit 0
```

=== `/etc/initramfs-tools/initramfs.conf` (original) ===

```code
:

#
# FSTYPE: ...
#
# The filesystem type(s) to support, or "auto" to use the current root
# filesystem type
#

FSTYPE=auto
```

=== `/etc/initramfs-tools/initramfs.conf` (modified) ===

```code
:

#
# FSTYPE: ...
#
# The filesystem type(s) to support, or "auto" to use the current root
# filesystem type
#

FSTYPE=auto

UMASK=0077
```

## USB-Disk (Partition)

Ensure the boot partitions are not mounted, before starting `fdisk`  to create an additional partition on the USB-Stick.

```console
umount /boot/efi 
umount /boot
fdisk /dev/sda
```

Identify the UUID of the newly created partition.

Note: *For the following UUID `cb7f829e-8035-11f0-91b3-03890b84d459` will be taken.*

```console
ls -la /dev/disk/by-uuid | grep --regexp="\/sda3$"
```

Prepare create the Key File.

```
mkdir /boot/keys
chmod 700 /boot/keys
dd if=/dev/urandom of=/boot/keys/cb7f829e-8035-11f0-91b3-03890b84d459.key bs=512 count=16
chmod 700 /boot/keys/cb7f829e-8035-11f0-91b3-03890b84d459.key
```



```console
cryptsetup luksFormat --verify-passphrase --type luks2 /dev/disk/by-uuid/cb7f829e-8035-11f0-91b3-03890b84d459
cryptsetup luksOpen /dev/sda3 sda3_crypt
dd if=/dev/urandom of=/dev/mapper/sda3_crypt
mkfs.ext2 /dev/mapper/sda3_crypt
cryptsetup luksClose sdb3_crypt
```



#!/bin/bash

/usr/sbin/cryptsetup luksOpen /dev/disk/by-uuid/include-your-uuid-from-blkid-here name-of-mapper --key-file=/path/to/keyfile

/usr/bin/mount /dev/mapper/name-of-mapper /your/chosen/mount/point


show details 
cryptsetup luksDump /dev/sda3

remove slot
cryptsetup luksKillSlot /dev/sda3 1

remove steup
cryptsetup luksErase /dev/sda3

       Example 3: Create LUKS header backup and save it to file.
           sudo cryptsetup luksHeaderBackup /dev/sdX --header-backup-file /var/tmp/NameOfBackupFile

       Example 6: Restore LUKS header from backup file.
           sudo cryptsetup luksHeaderRestore /dev/sdX --header-backup-file /var/tmp/NameOfBackupFile




# Based on

- [Cryptsetup for Debian: Full disk encryption, including /boot: Unlocking LUKS devices from GRUB](https://cryptsetup-team.pages.debian.net/cryptsetup/encrypted-boot.html)
- [Debian Manpages: / trixie / initramfs-tools-bin / unmkinitramfs(8)](https://manpages.debian.org/trixie/initramfs-tools-bin/unmkinitramfs.8.en.html)
- [Neil's blog: Mounting LUKS-encrypted disks by UUID](https://neilzone.co.uk/2023/11/mounting-luks-encrypted-disks-by-uuid/)
- [Stack Overflow: Destroying luks header on dm-crypt linux](https://stackoverflow.com/questions/60289422/destroying-luks-header-on-dm-crypt-linux)
- [Unix & Linux Stack Exchange: /etc/crypttab not updating in initramfs](https://unix.stackexchange.com/questions/708445/etc-crypttab-not-updating-in-initramfs)
- [Unix & Linux Stack Exchange: Unlock LUKS encrypted Debian root with key file on boot partition](https://unix.stackexchange.com/questions/164403/unlock-luks-encrypted-debian-root-with-key-file-on-boot-partition)
