# Debian 13 - Desktop/Laptop/Server

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

## Boot Manager

- [Boot Manager](./Boot%20Manager/README.md)
  - [Scenario: Hide GRUB Menu](./Boot%20Manager/README.md#scenario-hide-grub-menu)

## Desktop Environment

- [Desktop Environment](./Desktop%20Environment/Readme.md)
  - [XFCE](./Desktop%20Environment/XFCE/Readme.md)

## Devices

- [Device](./Device/Readme.md)
  - [Lenovo Ideapad Duet 3 10IGL5](./Device/Lenovo%20Ideapad%20Duet%203%2010IGL5/Readme.md)

## Keyboard

### Onscreen Keyboard

#### Onboard

##### As root

Install the software package using command

```console
apt install --yes onboard
```

Enable the Onscreen-Keyboard for the login process (LightDM).

```console
cp /etc/lightdm/lightdm-gtk-greeter.conf /etc/lightdm/lightdm-gtk-greeter.conf.ORG
cat << EOF >> /etc/lightdm/lightdm-gtk-greeter.conf

indicators=~language;~a11y;~session;~power
keyboard=/usr/bin/onboard
EOF
```

##### As Regular User

- ⬆ select program "**Settings > Session and Startup**"
    - navigate to tab "**Application Autostart**"
    - in the list, scroll to entry "**Onboard (Flexible onscreen keyboard)**"
    - ✏️ enable "**Onboard (Flexible onscreen keyboard)**"
    - 🔙 click on "**❌ Close**"

Make some adjustments to the Onscreen-Keyboard.

- ⬆ select program "**Settings > Onboard Settings**"
    - navigate to "**General**" in the left panel
    - ✏️ enable "**Auto show when editing text**"
    - navigate to "**Layout**" in the left panel
    - ✏️ select "**Core layouts > Full Keyboard**"
    - navigate to "**Theme**" in the left panel
    - ✏️ select "**Model M**"
    - 🔙 click on "**❌ Close**"

## Language

In order to support various language on the same machine, the adjustments are required

- add additional using `dpkg-reconfigure locales`
- adjust system-wide configuration file `xinitrc`
- regular user maintain their language in file `.i18n`

As `root` add the if statement for the `.i18n` file to the system wide configuration file `xinitrc`.

```console
cp /etc/xdg/xfce4/xinitrc /etc/xdg/xfce4/xinitrc.ORG
vi /etc/xdg/xfce4/xinitrc
```

=== `/etc/xdg/xfce4/xinitrc` ===

```sh
#!/bin/sh

if [ -f "$HOME/.i18n" ]; then
  . "$HOME/.i18n"
fi

:
```

### Example: German

As regular user execute the following to enable the German language. After `.i18n` got created, logout and login.

```console
cat << EOF > ~/.i18n
export LANGUAGE=de_DE.utf8
export LANG=de_DE.utf8
export LC_ALL=de_DE.utf8
EOF
```

## RAM-Disk

Create some additional OS folders as RAM-Disk. Useful for devices with more than 4GB RAM and Flash Memory as storage media.

```code
cat << EOF >> /etc/fstab
# --- START OF Machine SPECIFIC DEFINITIONS --- #

# Temporary files onto tmpfs
none /tmp tmpfs defaults,noatime,mode=1777 0 0
none /var/tmp tmpfs defaults,noatime 0 0
none /var/log tmpfs defaults,noatime 0 0
none /var/cache tmpfs defaults,noatime 0 0
EOF
```

## Repositories

## Security

- [Security](./Security/Readme.md)
  - [Disk Encryption](./Security/Disk%20Encryption/Readme.md)

## Software

### Browser

- [Browser](./Browser/Readme.md)
  - [Chrome and Variants](./Browser/Chrome%20and%20Variants/Readme.md)
  - [Firefox and Variants](./Browser/Firefox%20and%20Variants/Readme.md)
