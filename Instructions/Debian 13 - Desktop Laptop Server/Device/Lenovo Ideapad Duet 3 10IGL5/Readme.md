# [Debian 13 - Desktop Laptop](../../Readme.md) > [Device](../Readme.md) > Lenovo Ideapad Duet 3 10IGL5

- The Workstation
  - Software
  	- OS: Debian 13 "trixie"
  - OS Users
    - Regular User
      - Account: `dlsysuserdl`

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

## Installation

Use the Graphics Installer to install the base system with XFCE.

1. Install Debian with Encrypted [System Root Partition (`/`)](../../Security/Disk%20Encryption/Readme.md#system-root-partition-) and [Automatic Unlock](../../Security/Disk%20Encryption/Readme.md#automatic-unlock)

## Adjustments

Adjustments to be made in the following order

2. Configure [Expert SSH Setup](../../Security/Readme.md#expert-ssh-setup)
3. Correct [Display Orientation](#rotation)
4. Adjust [GRUB](../../Readme.md#boot-manager)
5. Create [RAM-Disk](../../Readme.md#ram-disk)
6. Adjust [Repositories (`contrib`, `non-free`, `Brave Browser`, `Librewolf`, `Mullvad Browser`, `Signal`)](../../Readme.md#repositories)
7. Install [Software](#software-installation)
8. Adjust [language](../../Readme.md#language) (user)
9. If you feel save enough, [remove the Password of the GNOME Keyring](../../Security/Readme.md#remove-password)
10. Adjust Touchpad [click](#enable-click-via-tap-touchpad) and [scroll](#reverse-scroll-direction-touchpad) behaviour
11. Adjust Thunar File Manager [Script Execution](#thunar-file-manager)
12. Adjust Browser to [utilize RAM-Disk](../../Software/Browser/Readme.md) and [Enable for Touch Screen / Tablet](../../Software/Browser/Firefox%20and%20Variants/Readme.md#enable-for-touch-screen--tablet)
13. Configure [On-Screen Keyboard](../../Readme.md#onboard)
14. Install [own CA Certificates](../../Security/Readme.md#install-ca-certificate)
15. Enable [FIDO2 authentication](../../Security/Readme.md#enable-fido2-authentication)
16. Restore [macOS like Elements](../../Desktop%20Environment/XFCE/Readme.md#restore-macos-like-elements) & [Adjust XFCE](../../Desktop%20Environment/XFCE/Readme.md#adjust-xfce)

### Backup

1. login as `root`
2. create file `machine.dir`
3. execute the following command

```console
tar --create --gzip --file=machine.tar.gz --directory=/ --files-from=machine.dir
```

=== `machine.dir` ===

```code
etc/default/bluetooth
etc/default/grub
etc/default/grub.ORG
etc/environment
etc/fido2/
etc/lightdm/lightdm.conf
etc/lightdm/lightdm.conf.ORG
etc/lightdm/lightdm-gtk-greeter.conf
etc/lightdm/lightdm-gtk-greeter.conf.ORG
etc/pam.d/common-auth
etc/pam.d/common-auth.ORG
etc/ssh/authorized_keys.d
etc/ssh/sshd_config.d/using_authorized_keys.d.conf
etc/systemd/logind.conf
etc/systemd/logind.conf.ORG
etc/xdg/xfce4/xinitrc
etc/xdg/xfce4/xinitrc.ORG
home/dlsysuserdl/.config/xfce4
home/dlsysuserdl/.config/cairo-dock
home/dlsysuserdl/.gnupg
home/dlsysuserdl/.i18n
home/dlsysuserdl/.librewolf
home/dlsysuserdl/.local/share/xfce4
home/dlsysuserdl/.mullvad-browser
home/dlsysuserdl/.ssh
home/dlsysuserdl/Application
home/dlsysuserdl/Helpful
usr/bin/screen_Standard.sh
usr/local/share/ca-certificates/my-custom-ca
usr/share/X11/xorg.conf.d/90-touchpad.conf
```

### Restore

After the system got setup, two account exists `root` and `dlsysuserdl`.

1. adjust Repositories
2. install Software
3. mount media with the two archives
4. execute the following commands

```console
tar --extract --gunzip --file=machine.tar.gz --directory=/
tar --extract --gunzip --file=XFCEmacOSlike.tar.gz --directory=/
update-grub
systemctl restart ssh
update-ca-certificates
usermod --groups sudo --append dlsysuserdl
```

### Screen (Build-In)

#### Rotation

With the keyboard the default orientation is landscape. In terms of the display manager the content should be rotated to the left. With the adjustments has to be made for the login manager (LightDM)

```console
cat << EOF >> /usr/bin/screen_Standard.sh
#!/bin/bash
xrandr --query                                                                                        >> /tmp/screen_Standard.out
xrandr --output DSI-1 --auto --rotate right                                                           >> /tmp/screen_Standard.out
xinput list                                                                                           >> /tmp/screen_Standard.out
xinput set-prop pointer:'ELAN901C:00 04F3:2BD6' 'Coordinate Transformation Matrix' 0 1 0 -1 0 1 0 0 1 >> /tmp/screen_Standard.out
chmod 755 /usr/bin/screen_Standard.sh
cp /etc/lightdm/lightdm.conf /etc/lightdm/lightdm.conf.ORG
cat << EOF >> /etc/lightdm/lightdm.conf

[SeatDefaults]
display-setup-script=/usr/bin/screen_Standard.sh
EOF
```

### Software Installation

```console
apt install --yes apostrophe blueman brave-browser dmidecode fido2-tools fonts-ocr-a fonts-ocr-b fonts-opendyslexic fwupd galculator libimobiledevice-utils libpam-u2f librewolf lshw mullvad-browser ntfs-3g pamu2fcfg qtqr onboard samba seahorse signal-desktop thunderbird thunderbird-l10n-de vlc xinput xournalpp
```

Additional software for [XFCE maxOS like](./XFCE%20macOS%20like/Readme.md) setup.

```console
apt install --yes appmenu-gtk3-module appmenu-registrar cairo-dock cairo-dock-xfce-integration-plug-in libglib2.0-dev-bin libsass1 libxml2-utils sassc xfce4-appmenu-plugin xfce4-indicator-plugin 
```

### Thunar File Manager

#### Allow Shell-Script execution

To be executed as the regular user ...

```console
xfconf-query --channel thunar --property /misc-exec-shell-scripts-by-default --create --type bool --set true
```

### Button/Power Behaviour

#### Login Manager (LightDM)

Change the default behaviour of the Power Button to suspend the system, instead of shutting it down.

```console
cp /etc/systemd/logind.conf /etc/systemd/logind.conf.ORG
sed --in-place \
    --expression="s/^#HandlePowerKey=poweroff$/HandlePowerKey=suspend/" \
    /etc/systemd/logind.conf
systemctl restart systemd-logind
```

=== `/etc/systemd/logind.conf` ===

```console
:
HandlePowerKey=suspend
:
```

#### Session

- ⬆ select program "**Settings > Power Manager**"
    - navigate to tab "**General**"
     - ✏️ select "`Suspend`" for option "**When power button is pressed**"
    - navigate to tab "**System**"
      - click on "**On Battery**"
        - ✏️ select "`Suspend`" for option "**System sleep mode**"
        - ✏️ move slider to `15` minutes for option "**When inactive for**"
        - ✏️ select "`Suspend`" for option "**When laptop lid is closed**"
      - click on "**Plugged in**"
        - ✏️ select "`Suspend`" for option "**System sleep mode**"
        - ✏️ move slider to `30` minutes for option "**When inactive for**"
        - ✏️ select "`Suspend`" for option "**When laptop lid is closed**"
    - 🔙 click on "**Close**"

### Touchpad

#### System Wide configuration (incl. LightDM)

```console
cat << EOF > /usr/share/X11/xorg.conf.d/90-touchpad.conf
Section "InputClass"
  Identifier "HAILUCK CO.,LTD Duet 3 USB Composite Device Touchpad"
  Driver "libinput"
  Option "Tapping" "on"
  Option "ClickMethod" "clickfinger"
  Option "NaturalScrolling" "true"
  Option "VertTwoFingerScroll" "on"
  Option "HorizTwoFingerScroll" "on"
EndSection
EOF
```

Note: *The exact name of the device (`Identifier`) was taken from the output of `xinput --list`.*

Based on [archlinux: libinput](https://wiki.archlinux.org/title/Libinput)

#### Enable click via tap (Touchpad)

- ⬆ select program "**Settings > Mouse and Touchpad**"
    - navigate to tab "**Devices**"
    - select `HAILUCK CO.,LTD Duet 3 USB Composite Device Touchpad` as "**Device**"
    - navigate to tab "**Devices > Touchpad**"
     - ✏️ enable "**Tap Touchpad to click**"
     - ✏️ select `Click with 1, 2 or 3 fingers as a left, right or middle click` as "**Click method**"
    - select `HAILUCK CO.,LTD Duet 3 USB Composite Device Touchpad` as "**Device**"
    - 🔙 click on "**Close**"
