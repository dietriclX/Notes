# [Debian 13 - Desktop Laptop](../../Readme.md) > Boot Manager

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

- The Workstation
  - Software
  	- OS: Debian 13 "trixie"
  - OS Users
    - Regular User
      - Account: `dlsysuserdl`

## GRUB

### Customization

Note: *Don't use the `grub-customizer` ... it will make so many changes, that the GRUB configuration is no longer Debian "compatible".*

### Background Image

Default image is `/usr/share/images/desktop-base/desktop-grub.png`.

Remove it, by adding the following lines in `/etc/default/grub` 

```code
# Specify Background image (or "" for none)
GRUB_BACKGROUND=""
```

### Repair Boot Manager (e.g. after Windows installation)

Use a Debian Live CD to get a system running, for the execution of the following commands.

_Assuming, the UEFI boot manager got installed on `/dev/nvme0n1p1` and the debian system on `/dev/nvme0n1p2`._

```sh
mount /dev/nvme0n1p2 /mnt/
mount /dev/nvme0n1p1 /mnt/boot/efi
for i in /dev /dev/pts /proc /sys /sys/firmware/efi/efivars /run; do sudo mount -B $i /mnt$i; done
chroot /mnt
grub-install /dev/nvme0n1
update-grub
```

_Assuming, the UEFI boot manager got installed on `/dev/sda1` and the debian system on `/dev/sda5`._

```sh
mount /dev/sda5 /mnt/
mount /dev/sda1 /mnt/boot/efi
for i in /dev /dev/pts /proc /sys /sys/firmware/efi/efivars /run; do sudo mount -B $i /mnt$i; done
chroot /mnt
grub-install /dev/sda
update-grub
```

Based on [debian / wiki / GrubEFIReinstall](https://wiki.debian.org/GrubEFIReinstall)

### Waiting-Time

Default image is "5" seconds.

Can be changed in `/etc/default/grub` in the line ...

```code
GRUB_TIMEOUT=4
```

### Scenario: Hide GRUB Menu

Hide the GRUB menu during boot time and use no background.

Note: *Keep the SHIFT key press right after start, in order to access the boot menu.*

```console
cp /etc/default/grub /etc/default/grub.ORG
sed --in-place --expression="s/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0\nGRUB_TIMEOUT_STYLE=hidden/" /etc/default/grub
cat << EOF >> /etc/default/grub

# Specify Background image (or "" for none)
GRUB_BACKGROUND=""
EOF
update-grub
```

=== `/etc/default/grub` (modified) ===

```code
:
GRUB_TIMEOUT=0
GRUB_TIMEOUT_STYLE=hidden
:

# Specify Background image (or "" for none)
GRUB_BACKGROUND=""
```

### Scenario: "[OldBIOS](https://www.gnome-look.org/p/2072033)" Theme

Download archive `OldBIOS.zip` and extract it into the Download folder.

```console
mkdir /boot/grub/themes
mv /home/dlsysuserdl/Downloads/OldBIOS /boot/grub/themes
chown -R root:root /boot/grub/themes/OldBIOS
```

Edit the file `/etc/default/grub` and set the following

```code
# GRUB THEME "BIOS boot device" (https://github.com/Blaysht/grub_bios_theme)
GRUB_THEME="/boot/grub/themes/OldBIOS/theme.txt"
```

Update the GRUB runtime, using command: `update-grub`

#### Font

In addition of having the GRUB menu looking like a BIOS, you can make it even more BIOS-look-a-like ...
by using the font "[InsydeH2O Setup Utility](https://fontstruct.com/fontstructions/show/2186015)" in a 1024x768 screen resolution.

1. download the archive "insydeh2o-setup-utility.zip" of the font "[InsydeH2O Setup Utility](https://fontstruct.com/fontstructions/show/2186015)" and extract the archive in the Download folder.
2. convert the font into an PFF2 formated font file, using command: `grub-mkfont --size=16 --output=insydeh2o_regular_16.pf2 "/home/dlsysuserdl/Downloads/insydeh2o-setup-utility/insydeh2o-setup-utility.ttf"`
3. copy the newly created file "insydeh2o_regular_16.pf2" into the directory `/boot/grub/themes/OldBIOS/f`
4. in file `OldBIOS/theme.txt` replace `Mbytepc230 Regular 16` and `MBytePC230 Regular 16` by `InsydeH2O Setup Utility Regular 16`
5. add the following two lines to `/etc/default/grub`

```code
# In combination with font "InsydeH2O Setup Utility" https://fontstruct.com/fontstructions/show/2186015
GRUB_GFXMODE=1024x768
```
6. update the GRUB runtime
