# [Debian 13 - Desktop Laptop](../../Readme.md) > [Desktop Environment](../Readme.md) > XFCE

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

- The Workstation
  - Software
  	- OS: Debian 13 "trixie"
  - OS Users
    - Regular User
      - Account: `dlsysuserdl`

## macOS like

The XFCE does not look like a MacOS and has not elements/configurations to come close to a Mac UI.

In order to get XFCE to look a MacOS the following steps have to be done.

1. install some additional software packages
2. download and "install" some UI elements
3. backup the additional UI elements
4. configure the XFCE UI

Note: *As the preparation by step 1.-3. is done system wide, other users have to do only the last step.*

Note on Step 3.: *In order to install the same UI Elements to another machine, simply extract the archive onto it.*

### Additional Software Packages

| Software | package | Desktop | Laptop | Tablet |
| ---: | :--- | :---: | :---: | :---:|
| GtkMenuShell D-Bus exporter (GTK+3.0) | appmenu-gtk3-module | ✓ | ✓ | ✓ |
| Appmenu DBusMenu registrar | appmenu-registrar | ✓ | ✓ | ✓ |
| Metapackage for cairo-dock | cairo-dock | ✓ | ✓ | ✓ |
| Xfce integration plug-in for Cairo-dock | cairo-dock-xfce-integration-plug-in | ✓ | ✓ | ✓ |
| Application Menu plugin for xfce4-panel | xfce4-appmenu-plugin                | ✓ | ✓ | ✓ |
| plugin to display information from applications in the Xfce4 panel | xfce4-indicator-plugin | ✓ | ✓ | ✓ |
| wavelan status plugin for the Xfce4 panel | xfce4-wavelan-plugin | ✓ | ✓ | ✓ |
| default | | | | |
| Alternate menu plugin for the Xfce desktop environment | xfce4-whiskermenu-plugin | ✓ | ✓ | ✓ |
| "WhiteSur-*" Style-Theme `install.sh` script installs the following | | | | |
| C/C++ port of the Sass CSS precompiler | libsass1 | ✓ | ✓ | ✓ |
| C/C++ port of the Sass CSS precompiler - command-line tool | sassc | ✓ | ✓ | ✓ |
| Development utilities for the GLib library | libglib2.0-dev-bin | ✓ | ✓ | ✓ |
| GNOME XML library - utilities | libxml2-utils | ✓ | ✓ | ✓ |

Install the above listed software, by executing the `apt install` command as `root`.

```console
apt install --yes appmenu-gtk3-module appmenu-registrar cairo-dock cairo-dock-xfce-integration-plug-in xfce4-appmenu-plugin xfce4-indicator-plugin 
```

```console
apt install --yes libsass1 sassc libglib2.0-dev-bin libxml2-utils
```


Note: *Already installed packages: xfce4-wavelan-plugin, xfce4-whiskermenu-plugin*

### XFCE-Theme Elements

There are many themes out there to a achieve a maxOS like UI.

Example of an XFCE configuration.

- Style: McOS-XFCE-Edition
- Icons: Cupertino-Ventura
- Fonts
    - Default Font: San Francisco Display Regular
    - Default Monospace Font: Meslo LG L Regular
- Mouse Theme: macOS-Monterey-White-Cursors
- Cairo-Dock
    - Theme: mcOS BS-White
    - Dustbin Theme: BigSur-black
- Whisker Icon: Apple grey

#### "BigSur*" Icon-Theme ([GitHub: yeyushengfan258 / BigSur-icon-theme](https://github.com/yeyushengfan258/BigSur-icon-theme))

Execute the following as `root`.

```console
git clone --depth 1 --shallow-submodules --recurse-submodules --branch master https://github.com/yeyushengfan258/BigSur-icon-theme.git
cd BigSur-icon-theme
./install.sh
```

##### Cairo-Dock Dustbin Theme

In order to have the dustbin in the Cairo-Dock available, execute the following as `root`.

```console
mkdir /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur
mkdir /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur-dark
ln -s /usr/share/icons/BigSur/places/scalable/user-trash.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur/trashcan_empty.svg
ln -s /usr/share/icons/BigSur/places/scalable/user-trash-full.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur/trashcan_full.svg
ln -s /usr/share/icons/BigSur/places/scalable/user-trash-full.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur/preview
ln -s /usr/share/icons/BigSur-dark/places/scalable/user-trash.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur-dark/trashcan_empty.svg
ln -s /usr/share/icons/BigSur-dark/places/scalable/user-trash-full.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur-dark/trashcan_full.svg
ln -s /usr/share/icons/BigSur-dark/places/scalable/user-trash-full.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur-dark/preview
```

#### "Cupertino-BigSur" Icon-Theme ([GitHub: USBA / Cupertino-Sonoma-iCons](https://github.com/USBA/Cupertino-Sonoma-iCons))

Execute the following as `root`.

```console
git clone --depth 1 --shallow-submodules --recurse-submodules --branch master https://github.com/USBA/Cupertino-Sonoma-iCons.git
rm --force --recursive Cupertino-BigSur-iCons/.git Cupertino-BigSur-iCons/.gitattributes
mv Cupertino-BigSur-iCons /usr/share/icons/Cupertino-BigSur
find /usr/share/icons/Cupertino-BigSur -type d -exec chmod +x '{}' \+
find /usr/share/icons/Cupertino-BigSur -name '\.*' -exec rm "{}" \+
mv '/usr/share/icons/Cupertino-BigSur/apps/128/Deepin camera.png' '/usr/share/icons/Cupertino-BigSur/apps/128/deepin-camera.png'
mv '/usr/share/icons/Cupertino-BigSur/apps/128/gnome-control-center datetime.png' '/usr/share/icons/Cupertino-BigSur/apps/128/gnome-control-center-datetime.png'
mv '/usr/share/icons/Cupertino-BigSur/apps/128/gnome-control-center sound.png' '/usr/share/icons/Cupertino-BigSur/apps/128/gnome-control-center-sound.png' 
gtk-update-icon-cache /usr/share/icons/Cupertino-BigSur
```

##### Cairo-Dock Dustbin Theme

In order to have the dustbin in the Cairo-Dock available, execute the following as `root`.

```console
mkdir /usr/share/cairo-dock/plug-ins/dustbin/themes/Cupertino-BigSur
ln -s /usr/share/icons/Cupertino-BigSur/places/128/user-trash.png /usr/share/cairo-dock/plug-ins/dustbin/themes/Cupertino-BigSur/trashcan_empty.png
ln -s /usr/share/icons/Cupertino-BigSur/places/128/user-trash-full.png /usr/share/cairo-dock/plug-ins/dustbin/themes/Cupertino-BigSur/trashcan_full.png
```

#### "macOS-Monterey*" Cursor-Theme

As regular user (dlsysuserdl)

Download the two archives and extract them in the Download folder.
[macOS Monterey](https://www.xfce-look.org/p/1648124/) > macOS-Monterey.tar.gz
[macOS Monterey White](https://www.xfce-look.org/p/1648129/) > macOS-Monterey-White.tar.gz

Both are based on [GitHub: ful1e5 / apple_cursor](https://github.com/ful1e5/apple_cursor)

Execute the following as `root`.

```shell
mv /home/regular_user/Downloads/macOS-Monterey*  /usr/share/icons/
chown -R root:root /usr/share/icons/macOS-Monterey*
chmod -R go-w /usr/share/icons/macOS-Monterey*
find /usr/share/icons/macOS-Monterey* -type d -exec chmod +x '{}' \+
gtk-update-icon-cache /usr/share/icons/macOS-Monterey
gtk-update-icon-cache /usr/share/icons/macOS-Monterey-White
```

#### "mcOS BS-*" Cairo-Dock-Theme

As a regular user (dlsysuserdl) download from [mcOS BS for Cairo Dock](https://www.pling.com/p/1401527) the following files and extract them in the Download folder.

- `mcOS-BS-Dark.tar.gz`
- `mcOS-BS-Lightblue.tar.gz`
- `mcOS-BS-Purple-Pink.tar.gz`
- `mcOS-BS-Redish.tar.gz`
- `mcOS-BS-White.tar.gz`

Execute the following as `root`.

```console
mv /home/dlsysuserdl/Downloads/mcOS-BS-* /usr/share/cairo-dock/themes
chown -R root:root /usr/share/cairo-dock/themes/mcOS-BS-*
chmod -R go-xw /usr/share/cairo-dock/themes/mcOS-BS-*
find /usr/share/cairo-dock/themes/mcOS-BS-* -type d -exec chmod +x '{}' \+
```

#### "McOS-themes" Style-Theme ([xfce-look.org: McOS-themes](https://www.xfce-look.org/p/1241688/))

Download "McOS-CTLina-XFCE-Dark.tar.xz" & "McOS-CTLina-XFCE.tar.xz" and extract them in Download folder.

Execute the following as `root`.

```console
mv /home/dlsysuserdl/Downloads/McOS-CTLina-XFCE /usr/share/themes
mv /home/dlsysuserdl/Downloads/McOS-CTLina-XFCE-Dark /usr/share/themes
chown -R root:root /usr/share/themes/McOS-CTLina-XFCE*
chmod -R go-w /usr/share/themes/McOS-CTLina-XFCE*
find /usr/share/themes/McOS-CTLina-XFCE* -type d -exec chmod +x '{}' \+
```

#### "McOS-XFCE-Edition" Style-Theme ([xfce-look.org: McOS-XFCE-Edition](https://www.xfce-look.org/p/1210386))

Download "McOS-XFCE-Edition-II-1.tar.xz" and extract it in Download folder.

Execute the following as `root`.

```console
mv /home/dlsysuserdl/Downloads/McOS-XFCE-Edition-II-1 /usr/share/themes/McOS-XFCE-Edition
chown -R root:root /usr/share/themes/McOS-XFCE-Edition
chmod -R go-w /usr/share/themes/McOS-XFCE-Edition
find /usr/share/themes/McOS-XFCE-Edition -type d -exec chmod +x '{}' \+
```

#### "OS Catalina" Icon-Theme ([GitHub: zayronxio / Os-Catalina-icons](https://github.com/zayronxio/Os-Catalina-icons))

Execute the following as `root`.

```console
git clone --depth 1 --shallow-submodules --recurse-submodules --branch master https://github.com/zayronxio/Os-Catalina-icons.git
rm --force --recursive Os-Catalina-icons/.git
mv Os-Catalina-icons /usr/share/icons/Os-Catalina
chmod -R go-w /usr/share/icons/Os-Catalina
find /usr/share/icons/Os-Catalina -type d -exec chmod +x '{}' \+
gtk-update-icon-cache /usr/share/icons/Os-Catalina
```

##### Cairo-Dock Dustbin Theme

In order to have the dustbin in the Cairo-Dock available, execute the following as `root`.

```console
mkdir /usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina
ln -s /usr/share/icons/Os-Catalina/128x128/places/user-trash.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina/trashcan_empty.svg
ln -s /usr/share/icons/Os-Catalina/128x128/places/user-trash-full.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina/trashcan_full.svg
ln -s /usr/share/icons/Os-Catalina-Night/128x128/places/user-trash.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina-Night/trashcan_empty.svg
ln -s /usr/share/icons/Os-Catalina-Night/128x128/places/user-trash-full.svg /usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina-Night/trashcan_full.svg
```

#### "osXFCE Theme" Style-Theme ([GitHub: imccausl / osXFCE](https://github.com/imccausl/osXFCE))

Execute the following as `root`.

```console
git clone https://github.com/imccausl/osXFCE.git 
mv osXFCE /usr/share/themes
```

Additional command, if case of using Plank instead of Cairo-Dock.

```console
mv /usr/share/themes/osXFCE/plank/flatabOSX-Theme /usr/share/plank/themes/osXFCE
```

Based on [osXFCE ist ein MacOS-inspiriertes Thema für XFCE, das auf arc-flatabulous basiert](https://blog.desdelinux.net/de/osxfce-tema-inspirado-macos-xfce-basado-arc-flatabulous/)

#### "WhiteSur-*" Style-Theme ([GitHub: vinceliuice / WhiteSur-gtk-theme](https://github.com/vinceliuice/WhiteSur-gtk-theme))

Execute the following as `root`.

```console
git clone --depth 1 --shallow-submodules --recurse-submodules --branch master https://github.com/vinceliuice/WhiteSur-gtk-theme.git
cd WhiteSur-gtk-theme
./install.sh
```

#### "WhiteSur*" Icon-Theme ([GitHub: vinceliuice / WhiteSur-icon-theme](https://github.com/vinceliuice/WhiteSur-icon-theme))

Execute the following as `root`.

```console
git clone --depth 1 --shallow-submodules --recurse-submodules --branch master https://github.com/vinceliuice/WhiteSur-icon-theme.git
cd WhiteSur-icon-theme
./install.sh
```

#### "WhiteSur Cursors" Cursor-Theme ([GitHub: vinceliuice / WhiteSur-cursors](https://github.com/vinceliuice/WhiteSur-cursors))

Execute the following as `root`.

```console
git clone --depth 1 --shallow-submodules --recurse-submodules --branch master https://github.com/vinceliuice/WhiteSur-cursors.git
cd WhiteSur-cursors
./install.sh
```

#### "Apple Menu Icon" Icon Sub-Sets ([pling.com: Apple Menu Icon](https://www.pling.com/p/1107444))

As regular user (dlsysuserdl)

Download the "55995-apple_logo.tar.tar" file. Extract the archives in "~/Download" folder.

Execute the following as `root`.

```console
mv /home/dlsysuserdl/Downloads/55995-apple_logo/*.svg /usr/share/icons/Os-Catalina/128x128/apps/
chown root:root /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*
chmod go-Wx /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*
cp /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*  /usr/share/icons/Os-Catalina-Night/128x128/apps/
cp /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*  /usr/share/icons/Cupertino-BigSur
cp /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*  /usr/share/icons/Cupertino-Ventura
gtk-update-icon-cache /usr/share/icons/Os-Catalina
gtk-update-icon-cache /usr/share/icons/Cupertino-BigSur
gtk-update-icon-cache /usr/share/icons/Cupertino-Ventura
```

#### "San Francisco" Font

As regular user (dlsysuserdl)

Download the file "XO.for.Dash.to.DOCK-GNOME.3.36.tar.xz" from [Gnome Shell Themes "XONE Catalina"](https://www.xfce-look.org/p/1213208). Extract the archive in "\~/Download" folder.

Execute the following as `root`.

```console
mkdir /usr/share/fonts/macOS
mv /home/dlsysuserdl/Downloads/XO.for.Dash.to.DOCK-GNOME.3.36/FONT/San* /usr/share/fonts/macOS
chown root:root /usr/share/fonts/macOS/San*
chmod go-x /usr/share/fonts/macOS/San*
find /usr/share/fonts/macOS/San* -type d -exec chmod +x '{}' \+
```

#### "Meslo" Font

As regular user (dlsysuserdl)

Download the file "Meslo LG v1.2.1.zip" and "Meslo LG v1.2.1.zip" from http://github.com/andreberg/Meslo-Font . Extract the archives in "\~/Download" folder.

Execute the following as `root`.

```console
mkdir /usr/share/fonts/macOS
mv /home/dlsysuserdl/Downloads/Meslo\ LG\ DZ\ v1.2.1/*.ttf /usr/share/fonts/macOS
mv /home/dlsysuserdl/Downloads/Meslo\ LG\ v1.2.1/*.ttf /usr/share/fonts/macOS
chown root:root /usr/share/fonts/macOS/*
chmod go-x /usr/share/fonts/macOS/*
find /usr/share/fonts/macOS -type d -exec chmod +x '{}' \+
```

#### Install icon for application

For some application, an icon might be missing ... in result the Cairo-Dock will use instead a placeholder. To fix this for example for "Xournal++", the following has to be done.

```console
cd ~
wget https://upload.wikimedia.org/wikipedia/commons/8/8a/Xournal%2B%2B_logo.svg
mv ~/Xournal++_logo.svg ~/xournalpp.svg
convert -background none ~/xournalpp.svg ~/xournalpp.png
cp ~/xournalpp.svg /usr/share/icons/BigSur/apps/scalable
cp ~/xournalpp.svg /usr/share/icons/BigSur-black/apps/scalable
cp ~/xournalpp.png /usr/share/icons/Cupertino-BigSur/apps/128
cp ~/xournalpp.png /usr/share/icons/Cupertino-Ventura/apps/128
cp ~/xournalpp.svg /usr/share/icons/WhiteSur/apps/scalable
gtk-update-icon-cache /usr/share/icons/BigSur
gtk-update-icon-cache /usr/share/icons/BigSur-black
gtk-update-icon-cache /usr/share/icons/Cupertino-BigSur
gtk-update-icon-cache /usr/share/icons/Cupertino-Ventura
gtk-update-icon-cache /usr/share/icons/WhiteSur
```

### Archive

#### Backup macOS like Elements

Create a file `XFCEmacOSlike.dir` with all the directories to be included in the backup.

Execute the following as `root`.

```console
tar --create --gzip --file=XFCEmacOSlike.tar.gz --directory=/ --files-from=XFCEmacOSlike.dir
```

=== `XFCEmacOSlike.dir` ===

```code
usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur
usr/share/cairo-dock/plug-ins/dustbin/themes/BigSur-dark
usr/share/cairo-dock/plug-ins/dustbin/themes/Cupertino-BigSur
usr/share/cairo-dock/plug-ins/dustbin/themes/Cupertino-Ventura
usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina
usr/share/cairo-dock/plug-ins/dustbin/themes/Os-Catalina-Night
usr/share/cairo-dock/themes/mcOS-BS-Dark
usr/share/cairo-dock/themes/mcOS-BS-Lightblue
usr/share/cairo-dock/themes/mcOS-BS-Purple-Pink
usr/share/cairo-dock/themes/mcOS-BS-Redish
usr/share/cairo-dock/themes/mcOS-BS-White
usr/share/fonts/macOS
usr/share/icons/BigSur
usr/share/icons/BigSur-black
usr/share/icons/BigSur-black-dark
usr/share/icons/BigSur-dark
usr/share/icons/Cupertino-BigSur
usr/share/icons/Cupertino-Ventura
usr/share/icons/macOS-Monterey
usr/share/icons/macOS-Monterey-White
usr/share/icons/Os-Catalina
usr/share/icons/Os-Catalina-Night
usr/share/icons/WhiteSur
usr/share/icons/WhiteSur-cursors
usr/share/icons/WhiteSur-dark
usr/share/icons/WhiteSur-light
usr/share/slim/themes/Catalina-Day
usr/share/slim/themes/Catalina-Night
usr/share/themes/McOS-CTLina-XFCE
usr/share/themes/McOS-CTLina-XFCE-Dark
usr/share/themes/McOS-XFCE-Edition
usr/share/themes/osXFCE
usr/share/themes/WhiteSur-Dark
usr/share/themes/WhiteSur-Dark-hdpi
usr/share/themes/WhiteSur-Dark-solid
usr/share/themes/WhiteSur-Dark-solid-hdpi
usr/share/themes/WhiteSur-Dark-solid-xhdpi
usr/share/themes/WhiteSur-Dark-xhdpi
usr/share/themes/WhiteSur-Light
usr/share/themes/WhiteSur-Light-hdpi
usr/share/themes/WhiteSur-Light-solid
usr/share/themes/WhiteSur-Light-solid-hdpi
usr/share/themes/WhiteSur-Light-solid-xhdpi
usr/share/themes/WhiteSur-Light-xhdpi
```

#### Restore macOS like Elements

In case the archive `XFCEmacOSlike.tar.gz` already exists, the new Elements can be easily installed on another PC.

Execute the following as `root`.

```console
tar --extract --gunzip --file=XFCEmacOSlike.tar.gz --directory=/
```

### Adjust XFCE

#### Desktop

Arrange the default icons:

- Home
- File System
- Trash

- ⬆ select "**Desktop Settings...**" from context menu of the desktop
    - navigate to tab "**Desktop Icons**"
    - keep `File/launcher icons` as "**Icon type**"
    - keep `48` as "**Icon size**"
    - keep "**Show icons tooltips**"" with "**Size**" `64`
    - keep `Top Left Vertical` as "**Placement direction**" 
    - ✏️ enable "**Show icons only on primary display**"
    - ✏️ enable "**Single click to activate items**"
    - 🔙 click on "**❌ Close**"

#### Appearance

- ⬆ select program "**Settings > Appearance**"
    - navigate to tab "**Style**"
    - ✏️ select `McOS-XFCE-Edition`
    - navigate to tab "**Icons**"
    - ✏️ select `Os-Catalina`
    - navigate to tab "**Fonts**"
    - ⬆ tab on "**Default Font**' entry ("Sans Regular    10")
        - type `san fran` into the search field
        - ✏️✓ select `San Francisco Display Regular`
        - keep `10` as "**Size**"
        - ✓🔙 click on "**Select**"
    - ⬆ tab on "**Default Monospace Font**' entry ("Monospace Regular    10")
        - type `meslo` into the search field
        - ✏️✓ select `Meslo LG L Regular`
        - keep `10` as "**Size**"
        - ✓🔙 click on "**Select**"
    - 🔙 click on "**❌ Close**"

#### Window Manager

- ⬆ select program "**Settings > Window Manager**"
    - navigate to tab "**Style**"
    - ✏️ select `McOS-XFCE-Edition` as "**Theme**"
    - ⬆ tab on "**Title Font**' entry ("Sans Bold    9")
        - type `san fran` into the search field
        - ✏️✓ select `San Francisco Display Bold`
        - keep `9` as "**Size**"
        - ✓🔙 click on "**Select**"
    - ✏️ arrange the "**Button layout**": _ 🗖 x Titel
    - 🔙 click on "**❌ Close**"

#### Windows Manager Tweaks

- ⬆ select program "**Settings > Windows Manager Tweaks**"
    - navigate to tab "**Compositor**"
    - keep "**Show shadows under popup windows**" ~~disabled~~
    - ✏️ ~~disable~~ "**Show shadows under dock windows**"
    - ✏️ ~~disable~~ "**Show shadow under regular windows**"
    - 🔙 click on "**❌ Close**"

Note: *Those changes are needed to prevent a ghost line under the top bar.*

#### Workspaces

- ⬆ select program "**Settings > Workspaces**"
    - navigate to tab "**General**"
    - ✏️ type `1` into "**Number of workspaces**"
    - 🔙 click on "**❌ Close**"

#### Mouse and Touchpad

- ⬆ select program "**Settings > Mouse and Touchpad**"
    - navigate to tab "**Theme**"
    - ✏️ select `macOS-Monterey`
    - type `32` into "**Cursor size**"
    - 🔙 click on "**❌ Close**"

#### Xfce Terminal

- ⬆ select program "**Settings > Xfce Terminal Settings**"
    - navigate to tab "**Appearance**"
    - keep "**Use system font**" ~~disabled~~
    - ⬆ tab on "**Font**' entry ("Monospace Regular    12")
        - type `meslo` into the search field
        - ✏️✓ select `Meslo LG L Regular`
        - keep `12` as "**Size**"
        - ✓🔙 click on "**Select**"
    - 🔙 click on "**❌ Close**"

#### Session and Startup

- ⬆ select program "**Settings > Session and Startup**"
    - navigate to tab "**Application Autostart**"
    - ⬆ click on "**➕ Add**"
        - ✏️✓ type `Glx-Dock / Cairo-Dock` in "**Name**"
        - ✏️✓ type `An OS-X-Dock like starter/task bar` in "**Description**"
        - ✏️✓ type `cairo-dock -o &` in Command**"
        - keep `on login` as "**Trigger**"
        - ✓🔙 click on "**OK**"
    - 🔙 click on "**❌ Close**"

Logout and Login again to verify that the Cairo-Dock is visible (got started)!

Note *"**Welcome to Cairo-Dock ! ...**" informs about the successful start of the Cairo-Dock.

#### Panel

- ⬆ select program "**Settings > Panel**"
    - from the top listbox select the bottom panel¹
    - ✏️✓ click on "—" (next to panel name)
    - ✓ confirm the deletion by click on "**Remove**"
    - continue with the top panel selected
    - navigate to tab "**Display**"²
    - navigate to tab "**Appearance**"²
    - ✏️ ~~disable~~ "**Dark mode**"
    - navigate to tab "**Items**"
    - ✏️ adjust the elements in the list by adding/removing one or the other³
    - ⬆ double click on 1. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - keep "**Expand**" ~~disabled~~
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 2. item "**Whisker-Menu**"
        - navigate to tab "**General**"
        - ✏️ select `Show as icons`
        - navigate to tab "**Appearance**"
        - focus on "**Panel Button**"
        - ⬆ click on the symbol next to "**Icon**"
            - type `apple` in "**Search icon**"
            - ✏️✓ select `apple_logo_grey`
            - ✓🔙 click on "**OK**"
        - navigate to tab "**Behaviour**"
        - focus on "**Default Category**"
        - ✏️ select option "**Recently Used**"
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 4. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - ✏️ enable "**Expand**"
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 5. item "**Indicator Plugin**"
        - navigate to tab "**Appearance**"
        - focus on "**Appearance**"
        - ✏️ enable "**Arrange indicators in a single row**"
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 6. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - keep "**Expand**" ~~disabled~~
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 8. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - keep "**Expand**" ~~disabled~~
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 11. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - keep "**Expand**" ~~disabled~~
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 12. item "**Clock**"
        - focus on "**Clock Options**"
        - ✏️ select `Date Only` as "**Layout**"
        - ⬆ tab on "**Font**' entry ("Sans Regular    8")
            - type `san fran` into the search field
            - ✏️✓ select `San Francisco Display Bold`
            - keep `10` as "**Size**"
            - ✓🔙 click on "**Select**"
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 13. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - keep "**Expand**" ~~disabled~~
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 14. item "**Clock**"
        - focus on "**Clock Options**"
        - ✏️ select `Time Only` as "**Layout**"
        - ⬆ tab on "**Font**' entry ("Sans Regular    8")
            - type `san fran` into the search field
            - ✏️✓ select `San Francisco Display Bold`
            - keep `10` as "**Size**"
            - ✓🔙 click on "**Select**"
        - 🔙 click on "**❌ Close**"
    - ⬆ double click on 15. item "**Separator**"
        - ✏️ select `Transparent` as "**Style**"
        - keep "**Expand**" ~~disabled~~
        - 🔙 click on "**❌ Close**"
    - 🔙 click on "**❌ Close**"

¹ To be identified by looking at tab "Items". Or looking at the options on tab "Display" ... "Automatically hide the panel" is set to "Intelligently" and the slide for "Length (pixel)" is far to the left.

² With Debian 12, two additional changes are needed
- ✏️ select `Primary` as "**Output**"
- ✏️ ~~disable~~ "**Keep panel above windows**"

³ The following elements should be in the top panel/bar (all Separator being transparent)
 1. Separator
 2. Whisker-Menu
 3. AppMenu-Plugin
 4. Separator
 5. Indicator Plugin
 6. Separator
 7. Status-Tray Plugin
 8. Separator
 9. PulseAudio Plugin
10. Notification Plugin
11. Separator
12. Clock
13. Separator
14. Clock
15. Separator

#### Cairo-Dock

Anywhere on the Cairo-Dock click on get its Context-Menu.

- ⬆ select "**Cairo-Dock > Configure**" from context menu of the Cairo-Dock
    - navigate to tab "**Themes**"
        - ✏️✓ select `mcOS-BS-White`
    - navigate to tab "**Add-ons**"
    - ✏️ adjust the elements in the list by enabling/~~disabling~~ one or the other¹
    - double click on "**Bin**"
    - navigate to tab "**Current Items > Bin > Configuration**"
    - ✏️ select `BigSur` as "**Choose one of the available themes**"
    - navigate to tab "**Configuration > Behaviour**"
    - focus on "**Taskbar**"
    - ✏️✓ select `At the end of the dock` as "**Place new icons**"
    - focus on "**Icon's animation and effects**"
    - keep `Pulse` as "**On mouse hover**"
    - keep `Bounce` as "**On click**"
    - ✏️✓ select `Evaporate` as "**On appearance/disappearance**"
    - navigate to tab "**Configuration > Appearance**"
    - focus on "**Style**"
    - ⬆ click on the black color box
        - ✏️✓ select a light grey
        - ✓🔙 click on "**Select**"
    - ✓🔙 click on "**Apply**"
- ✏️ adjust the elements in the Cairo-Dock²

¹ Only the following elements should be enabled
- Bin
- Applications Menu

² Take for example the following list of application. Simply search in the Whisker menu for the programm and drag&drop the icon onto the Cairo-Dock.
 1. Applications-Menu
 2. Mail Reader
 3. LibreWolf
 4. Thunar File Manager
 5. Xfce-Terminal
 6. Separator
 7. Shortcuts
 8. Bin
 9. Separator

### Adjust Applications

#### Chrome and Variants

In the web browser, tap on `≡` to open to the menu followed by  tap on "**⚙ Settings**"

In the "**⚙ Settings**" do the following changes, to change the buttons close/maximize/minimize in the macOS like form. 

- ⬆ click on "**Appearance**" in the left panel
    - click on **`Use GTK`** as "**Theme**"
    - 🖉 enable "**Use system title bar and borders**"

#### Firefox and Variants

By default the window title bar is hidden and in result the buttons close/maximize/minimize are not shown in the macOS like form. To change the appearance, the title bar has to be enabled. 

- press ALT key to make the menu appear
- select menu option "**View** > **Toolsbars** > **Customize Toolbar...**"
    - 🖉 enable checkbox "**Title Bar**"
    - ✓🔙 click on "**Done**"
