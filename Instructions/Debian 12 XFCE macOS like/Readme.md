_Note: Replace "USERID" by your user-name._

# Dock: [Glx-Dock / Cairo-Dock](https://glx-dock.org/)


## Theme: MaxOSX

In the "Cairo-Dock-Einstellungen > Themen", select "MacOSX".

In the "Einstellungen > Leiste" remove "Leiste 1" (the dock) but keep "Leiste 0" (the top menu). Maybe the numbers are different ... just take a look at tab "Objekte" to identify the right one.

Make it general available for all users 

```console
mv /home/USERID/.config/cairo-dock/themes/MacOSX /usr/share/cairo-dock/themes
chown -R root:root /usr/share/cairo-dock/themes/MacOSX
chmod -R go-xw /usr/share/cairo-dock/themes/MacOSX	
find /usr/share/cairo-dock/themes/MacOSX -type d | while read x
do
chmod +x $x
done
```


## Theme: mcOS BS for Cairo Dock

Download from [mcOS BS for Cairo Dock](https://www.pling.com/p/1401527) the following files and extract them into `/usr/share/cairo-dock/theme`.

-mcOS-BS-Dark.tar.gz
-mcOS-BS-Lightblue.tar.gz
-mcOS-BS-Purple-Pink.tar.gz
-mcOS-BS-Redish.tar.gz
-mcOS-BS-White.tar.gz

Make sure the access rights are set properly

```console
chown -R root:root /usr/share/cairo-dock/themes/mcOS-BS-*
chmod -R go-xw /usr/share/cairo-dock/themes/mcOS-BS-*
find /usr/share/cairo-dock/themes/mcOS-BS-* -type d | while read x
do
chmod +x $x
done
```

# XFCE-Theme

Theme: McOS-XFCE-Edition
Icons: Cupertino-Ventura
Fonts San Francisco Display Regular
Mouse Theme:  macOS-Monterey-White-Cursors


1. Install the theme
2 In "Einstellungen" select the theme "osXFCE" for
    - "Erscheinungsbild" on tab "Oberfläche"
    - "Fensterverwaltung" on tab "Stil"

###### Theme: [osXFCE Theme](https://github.com/imccausl/osXFCE)

```console
git clone https://github.com/imccausl/osXFCE.git 
cp -r ~/osXFCE /usr/share/themes/osXFCE 
```

Only if using Plank

```console
cp -r ~/osXFCE/plank/flatabOSX-Theme /usr/share/plank/themes/osXFCE
```

Based on [osXFCE ist ein MacOS-inspiriertes Thema für XFCE, das auf arc-flatabulous basiert](https://blog.desdelinux.net/de/osxfce-tema-inspirado-macos-xfce-basado-arc-flatabulous/)

[McOS-XFCE-Edition](https://www.xfce-look.org/p/1210386)

Download "McOS-XFCE-Edition-II-1.tar.xz" and extract it in Download folder.

```console
mv /home/USERID/Downloads/McOS-XFCE-Edition-II-1 /usr/share/themes/McOS-XFCE-Edition
chown -R root:root /usr/share/themes/McOS-XFCE-Edition
chmod -R go-w /usr/share/themes/McOS-XFCE-Edition
find /usr/share/themes/McOS-XFCE-Edition -type d | while read x
do
chmod +x $x
done
```


[McOS-themes](https://www.xfce-look.org/p/1241688/)

Download "McOS-CTLina-XFCE-Dark.tar.xz" & "McOS-CTLina-XFCE.tar.xz" and extract them in Download folder.

```console
mv /home/USERID/Downloads/McOS-CTLina-XFCE /usr/share/themes
mv /home/USERID/Downloads/McOS-CTLina-XFCE-Dark /usr/share/themes
chown -R root:root /usr/share/themes/McOS-CTLina-XFCE*
chmod -R go-w /usr/share/themes/McOS-CTLina-XFCE*
find /usr/share/themes/McOS-CTLina-XFCE* -type d -exec chmod +x '{}' \+
```



# Fonts

1. Install the fonts
2. In "Einstellungen > Erscheinungsbuild" on tab "Schriften" select
    - Vorgabeschrift: San Francisco Display Regular
    - Vorgebene nicht proportionale Schrift: Meslo LG L Regular

## Font "San Francisco"

As user

Download the file "XO.for.Dash.to.DOCK-GNOME.3.36.tar.xz" from [Gnome Shell Themes "XONE Catalina"](https://www.xfce-look.org/p/1213208). Extract the archive in "~/Download" folder.

As root

```console
mkdir /usr/share/fonts/macOS
mv /home/USERID/Downloads/XO.for.Dash.to.DOCK-GNOME.3.36/FONT/San* /usr/share/fonts/macOS
chown root:root /usr/share/fonts/macOS/San*
chmod go-x /usr/share/fonts/macOS/San*
find /usr/share/fonts/macOS/San* -type d | while read x
do
chmod +x $x
done
```

## Font "Meslo"

As user

Download the file "Meslo LG v1.2.1.zip" and "Meslo LG v1.2.1.zip" from http://github.com/andreberg/Meslo-Font . Extract the archives in "~/Download" folder.

As root

```console
mkdir /usr/share/fonts/macOS
mv /home/USERID/Downloads/Meslo\ LG\ DZ\ v1.2.1/*.ttf /usr/share/fonts/macOS
mv /home/USERID/Downloads/Meslo\ LG\ v1.2.1/*.ttf /usr/share/fonts/macOS
chown root:root /usr/share/fonts/macOS/*
chmod go-x /usr/share/fonts/macOS/*
find /usr/share/fonts/macOS -type d | while read x
do
chmod +x $x
done
```

# Icons

1. Install the icons
2. In "Einstellungen > Erscheinungsbild" on tab "Symbole" select "Os-Catalina" or Os-Catalina-Night".

###### [Full Icon Themes "OS Catalina"](https://www.gnome-look.org/p/1309810)

As user

Download the "Os-Catalina-icons.tar.xz" file.

As root

```console
cd /usr/share/icons/
tar xf /home/USERID/Downloads/Os-Catalina-icons.tar.xz
mv /usr/share/icons/Os-Catalina-icons /usr/share/icons/Os-Catalina
chown -R root:root /usr/share/icons/Os-Catalina*
chmod -R go-w /usr/share/icons/Os-Catalina*
find /usr/share/icons/Os-Catalina* -type d | while read x
do
chmod +x $x
done
gtk-update-icon-cache /usr/share/icons/Os-Catalina
gtk-update-icon-cache /usr/share/icons/Os-Catalina-Night
```


###### [Cupertino iCons](https://www.xfce-look.org/p/1102582/)

As user

Download and extract the archive "Cupertino-Ventura.tar.xz" file.

As root

```console
cd /usr/share/icons/
mv /home/USERID/Downloads/Cupertino-Ventura/Cupertino-Ventura .
chown -R root:root /usr/share/icons/Cupertino-Ventura
chmod -R go-w /usr/share/icons/Cupertino-Ventura
find /usr/share/icons/Cupertino-Ventura -type d | while read x
do
chmod +x $x
done
find Cupertino-Ventura -name '\.*' | while read x
do
rm "$x"
done
mv 'Cupertino-Ventura/apps/128/Deepin camera.png' 'Cupertino-Ventura/apps/128/deepin-camera.png'
mv 'Cupertino-Ventura/apps/128/gnome-control-center datetime.png' 'Cupertino-Ventura/apps/128/gnome-control-center-datetime.png'
mv 'Cupertino-Ventura/apps/128/gnome-control-center sound.png' 'Cupertino-Ventura/apps/128/gnome-control-center-sound.png' 
gtk-update-icon-cache /usr/share/icons/Cupertino-Ventura
```

###### [Icon Sub-Sets "Apple Menu Icon"](https://www.pling.com/p/1107444)

As user

Download the "55995-apple_logo.tar.tar" file. Extract the archives in "~/Download" folder.

As root

```console
mv /home/USERID/Downloads/55995-apple_logo/*.svg /usr/share/icons/Os-Catalina/128x128/apps/
chown root:root /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*
chmod go-Wx /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*
cp /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*  /usr/share/icons/Os-Catalina-Night/128x128/apps/
cp /usr/share/icons/Os-Catalina/128x128/apps/apple_logo_*  /usr/share/icons/Cupertino-Ventura
gtk-update-icon-cache /usr/share/icons/Os-Catalina
gtk-update-icon-cache /usr/share/icons/Cupertino-Ventura
```

## Mouse Cursor

Download the two archives and extract them in the Download folder.
[macOS Monterey](https://www.xfce-look.org/p/1648124/) \> macOS-Monterey.tar.gz
[macOS Monterey White](https://www.xfce-look.org/p/1648129/) \> macOS-Monterey-White.tar.gz

Both are based on [github: ful1e5 / apple_cursor](https://github.com/ful1e5/apple_cursor)

```console
mv /home/USERID/Downloads/macOS-Monterey*  /usr/share/icons/
chown -R root:root /usr/share/icons/macOS-Monterey*
chmod -R go-w /usr/share/icons/macOS-Monterey*
find /usr/share/icons/macOS-Monterey* -type d | while read x
do
chmod +x $x
done
gtk-update-icon-cache /usr/share/icons/macOS-Monterey
gtk-update-icon-cache /usr/share/icons/macOS-Monterey-White
```

# Archive: Fonts, Icons, Themes, Cairo-Themes

The directories with the relevant content.

```console
/usr/share/cairo-dock/themes/MacOSX
/usr/share/cairo-dock/themes/mcOS-BS-*
/usr/share/fonts/macOS
/usr/share/icons/Cupertino-Ventura
/usr/share/icons/Os-Catalina*
/usr/share/icons/macOS-Monterey 
/usr/share/themes/McOS-XFCE-Edition
/usr/share/themes/osXFCE/
```

Create an archive, using command:

```console
tar -zcvf XFCEmacOSlike.tar.gz -C ~ /usr/share/cairo-dock/themes/MacOSX /usr/share/cairo-dock/themes/mcOS-BS-* /usr/share/fonts/macOS /usr/share/icons/Cupertino-Ventura /usr/share/icons/Os-Catalina* /usr/share/icons/macOS-Monterey /usr/share/themes/osXFCE /usr/share/themes/McOS-XFCE-Edition


Extract on target system, using command:

```console
tar -xzvf /home/USERID/XFCEmacOSlike.tar.gz -C /
```


# Activitation

Install some packages.

```console
apt install -y xfce4-appmenu-plugin cairo-dock
```

- 🪜 open XFCE menu "**Settings** > **Appearance**"
  - navigate to tab "**Style**"
  - 🖉 select theme "**McOS-XFCE-Edition**"
  - navigate to tab "**Icons**"
  - 🖉 select theme "**Cupertino-Ventura**"
  - navigate to tab "**Fonts**"
  - 🪜 click on button below "**Default Font**"
    - ✓🖉 select font "**San Francisco Display Regular**"
    - ✓🛝 click on "**Select**"
  - 🪜 click on button below "**Default Monospace Font**"
    - ✓🖉 select font "**Meslo LG L Regular**"
    - ✓🛝 click on "**Select**"
  - 🛝 click on "**Close**"
- 🪜 open XFCE menu "**Settings** > **Windows Manager**"
  - navigate to tab "**Style**"
  - 🖉 select theme "**McOS-XFCE-Edition**"
- 🪜 click on button below "**Title font**"
    - ✓🖉 select font "**San Francisco Display Regular**"
    - ✓🛝 click on "**Select**"
  - 🖉 drag&drop "**▼**" from group "**Active**" to "**Hidden**"
  - 🖉 drag&drop "**🠅**" from group "**Active**" to "**Hidden**"
  - 🖉 rearrange button from group "**Active**" in this order "**_ 🗖 x Titel**"
  - 🛝 click on "**Close**"
- 🪜 open XFCE menu "**Settings** > **Windows Manager Tweaks**"
  - navigate to tab "**Compositor**"
  - 🖉 ~~disable~~ option "**Show shadows under regular windows**"
  - 🖉 ~~disable~~ option "**Show shadows under popup windows**"
  - 🖉 ~~disable~~ option "**Show shadows under dock windows**" ¹
  - 🛝 click on "**Close**"
- 🪜 open XFCE menu "**Settings** > **Mouse and Touchpad**"
  - navigate to tab "**Theme**"
  - 🖉 select theme "**macOS-Monterey-White**"
  - 🛝 click on "**Close**"
- 🪜 open XFCE menu "**Settings** > **Xfce Termin Preferences**"
  - navigate to tab "**Appearance**"
  - 🖉 enable option "**Use system font**"
  - 🛝 click on "**Close**"
- 🪜 open XFCE menu "**Settings** > **Session and Startup**"
  - navigate to tab "**Application Autostart**"
  - 🪜 click on "**➕ Add**" button
    - ✓🖉 specify "Glx-Dock / Cairo-Dock" as "**Name**"
    - ✓🖉 specify "An OS-X-Dock like starter/task bar" as "**Description**"
    - ✓🖉 specify "cairo-dock &" as "**Command**"
    - ✓🖉 select "on login" as "**Trigger**"
    - ✓🛝 click on "**OK**"
  - 🛝 click on "**Close**"
- logout of XFCE session

- login to XFCS session
- 👁 ensure the Cairo-Dock is visible!
- 🪜 open XFCE menu "**Settings** > **Panel**"
  - navigate to tab "**Items**"
  - from top listbox select a panel, which items do matches those from the **bottom** panel ("Panel 1")
  - 🪜✓🖉 click on the "**🗑**" button to delete the panel
      - ✓🛝 click on "**Remove**" to confirm the deletion
  - modify the remaining top panel ("Panel 0")
  - navigate to tab "**Display**"
  - 🖉 enable option "**Keep panel above windows**"
  - navigate to tab "**Appearance**"
  - 🖉 ~~disable~~ option "**Dark mode**"
  - navigate to tab "**Items**"
  - 🖉 add/remove/reorder the items to get at the end
    - Separator
      - Style: Transparent
      - Expand: _
    - Whisker Menu
      - Symbol: /usr/share/icons/Cupertino-Ventura/apple_logo_grey.svg
    - AppMenu Plugin
    - Separator
      - Style: Transparent
      - Expand: x
    - Indicator Plugin
    - Separator
      - Style: Transparent
      - Expand: _
    - Status Tray Plugin
    - Separator
      - Style: Transparent
      - Expand: _
    - PulseAudio Plugin
    - Notification Plugin
    - Engerieverwaltungserweiterung
    - Separator
      - Style: Transparent
      - Expand: _
    - Clock (Properties > Einstellungen für die Uhr > Anordnung: Ausschließlich Datum)
    - Separator
      - Style: Transparent
      - Expand: _
    - Clock (Properties > Einstellungen für die Uhr > Anordnung: Ausschließlich Uhrzeit)
    - Separator
      - Style: Transparent
      - Expand: _
  - navigate to tab "**Items**"
- in the Cairo Dock open context menu of one of the symbols XFCE menu
- 🪜 choose menu "**Cairo-Dock** > **Configure**"
  - navigate to tab "**Configuration** > **Behaviour**"
  - ✓🖉 select "End of Dock" for "**Place new icons**" in section "**Taskbar**"
  - ✓🖉 select "Bounce" for "**On click**" in section "**Icon's animation and effects**'
  - navigate to tab "**Themes**"
  - ✓🖉 select theme "**mcOS-BS-White**"
  - ✓ click on "**Apply**"

¹ From [Horizontal Line Across the Screen on Xfce](https://nochkawtf.wordpress.com/2015/05/03/horizontal-line-across-screen-on-xfce/)


Install icon for "Xournal++"
```console
cd ~
wget https://upload.wikimedia.org/wikipedia/commons/8/8a/Xournal%2B%2B_logo.svg
convert -background none ~/Xournal++_logo.svg ~/xournalpp.png
mv ~/xournalpp.png /usr/share/icons/Cupertino-Ventura/apps/128
gtk-update-icon-cache /usr/share/icons/Cupertino-Ventura
```


gray - Default (Light) = #8E8E92

Based on [Color | Apple Developer Documentation > macOS system colors](https://developer.apple.com/design/human-interface-guidelines/color#macOS-system-colors)

	

```console
root@iThink:/usr/share/themes/McOS-XFCE-Edition/gtk-3.0# diff gtk.css gtk.css.ORG 
13131,13133c13131,13137
<   background-color: #FFFFFF;
<   color: #1c1c1c;
<   -gtk-icon-shadow:none;
---
>   background-color: shade(#686868, 0.35);
>   color: #fcfcfc;
>   text-shadow:  0 -1px alpha(#ffffff, 0.04),
> 				 -1px  0px alpha(#202020, 0.05),
> 				  1px  0px alpha(#202020, 0.05),
> 				  0px  1px alpha(#202020, 0.5),
> 				  0px  2px alpha(#202020, 0.05);
13139c13143
<   background-color: #FFFFFF;
---
>   background-color: #37383d;
13144,13147c13148,13154
<   color: #1c1c1c;
<   text-shadow: none;
<   -gtk-icon-shadow:none;
< }
---
>   color: #dde2e7;
>   text-shadow:  0 -1px alpha(#ffffff, 0.04),
> 				 -1px  0px alpha(#202020, 0.05),
> 				  1px  0px alpha(#202020, 0.05),
> 				  0px  1px alpha(#202020, 0.5),
> 				  0px  2px alpha(#202020, 0.05);
>   /*text-shadow: 0px 1px rgba(0, 0, 0, 0.1);*/ }
13153c13160
<   background-color: #1c1c1c;
---
>   background-color: #53555c;
13155c13162,13167
<   color: #ffffff;
---
>   color: #dde2e7;
>   text-shadow:  0 -1px alpha(#ffffff, 0.04),
> 				 -1px  0px alpha(#202020, 0.05),
> 				  1px  0px alpha(#202020, 0.05),
> 				  0px  1px alpha(#202020, 0.5),
> 				  0px  2px alpha(#202020, 0.05);
13162c13174
<   background-color: #1c1c1c;
---
>   background-color: #53555c;
13164c13176,13181
<   color: #ffffff;
---
>   color: #dde2e7;
>   text-shadow:  0 -1px alpha(#ffffff, 0.04),
> 				 -1px  0px alpha(#202020, 0.05),
> 				  1px  0px alpha(#202020, 0.05),
> 				  0px  1px alpha(#202020, 0.5),
> 				  0px  2px alpha(#202020, 0.05);
13174c13191
<  
---
> 
```



xfconf-query -c xsettings -p /Net/ThemeName -s "McOS-XFCE-Edition"


Based on
[command line - Change default XFCE backdrop for all users via cli? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/204586/change-default-xfce-backdrop-for-all-users-via-cli)
[XFCE: start » howto » install_new_themes](https://wiki.xfce.org/howto/install_new_themes)
[XFCE: start » xfce » xfconf - Configuration Storage System](https://docs.xfce.org/xfce/xfconf/start)
[XFCE: start » xfce » xfconf » xfconf-query](https://docs.xfce.org/xfce/xfconf/xfconf-query)


McOS-XFCE-Edition
/usr/share/themes/McOS-XFCE-Edition/gtk-3.0/gtk.css


Effects the Panel itself and Objects with Graphics
```
.xfce4-panel.background {

  background-color: shade(#686868, 0.35);
  color: #fcfcfc;
  text-shadow:  0 -1px alpha(#ffffff, 0.04),
				 -1px  0px alpha(#202020, 0.05),
				  1px  0px alpha(#202020, 0.05),
				  0px  1px alpha(#202020, 0.5),
				  0px  2px alpha(#202020, 0.05);
  font-weight: normal; }
```

change into

```
.xfce4-panel.background {

  -gtk-icon-shadow:none;
  background-color: #ffffff;
  color: #1c1c1c;
  font-weight: normal; }
```


Effects on the Panel the Objects with Text or Icon
```
.xfce4-panel.background button {

  background-image: none;
  background-color: #37383d;
  border-radius: 0;
  border: none;
  box-shadow: none;
  padding: 0 1px;
  color: #dde2e7;
  text-shadow:  0 -1px alpha(#ffffff, 0.04),
				 -1px  0px alpha(#202020, 0.05),
				  1px  0px alpha(#202020, 0.05),
				  0px  1px alpha(#202020, 0.5),
				  0px  2px alpha(#202020, 0.05);
  /*text-shadow: 0px 1px rgba(0, 0, 0, 0.1);*/ }
```

change into

```
.xfce4-panel.background button {

  -gtk-icon-shadow:none;
  background-image: none;
  background-color: #ffffff;
  border-radius: 0;
  border: none;
  box-shadow: none;
  color: #1c1c1c;
  text-shadow: none;
  padding: 0 1px; }
```

```
.xfce4-panel.background button:hover,
.xfce4-panel.background button:active:hover,
.xfce4-panel.background button:checked:hover {

  background-color: #53555c;
  background-image: none;
  color: #dde2e7;
  text-shadow:  0 -1px alpha(#ffffff, 0.04),
				 -1px  0px alpha(#202020, 0.05),
				  1px  0px alpha(#202020, 0.05),
				  0px  1px alpha(#202020, 0.5),
				  0px  2px alpha(#202020, 0.05);
  /*box-shadow: inset 0 -1px alpha(white,0), inset 1px 0 alpha(white,0.15), inset -1px 0 alpha(white,0.15), inset 0 1px alpha(white,0.15);*/
  transition: none; }
```

```
.xfce4-panel.background button:hover,
.xfce4-panel.background button:active:hover,
.xfce4-panel.background button:checked:hover {

  background-color: #1c1c1c;
  background-image: none;
  color: #ffffff;
  text-shadow:  0 -1px alpha(#ffffff, 0.04),
				 -1px  0px alpha(#202020, 0.05),
				  1px  0px alpha(#202020, 0.05),
				  0px  1px alpha(#202020, 0.5),
				  0px  2px alpha(#202020, 0.05);
  /*box-shadow: inset 0 -1px alpha(white,0), inset 1px 0 alpha(white,0.15), inset -1px 0 alpha(white,0.15), inset 0 1px alpha(white,0.15);*/
  transition: none; }
```


McOS-themes > McOS-CTLina-XFCE
/usr/share/themes/McOS-CTLina-XFCE/gtk-3.0/gtk.css

from line 12223 onwards

Effects the Panel itself and Objects with Graphics

```
.xfce4-panel.background {

  background-color: #2c2b2c;
  color: #dddddd;
  text-shadow: text-shadow: 0px 1px alpha(#000000, 0.40),0 -1px alpha(#ffffff, 0.04),-1px  0px alpha(#ffffff, 0.04);
  font-weight: normal; }
```

```
.xfce4-panel.background {
  -gtk-icon-shadow:none;
  background-color: #ffffff;
  color: #1c1c1c;
  text-shadow: text-shadow: 0px 1px alpha(#000000, 0.40),0 -1px alpha(#ffffff, 0.04),-1px  0px alpha(#ffffff, 0.04);
```

Effects on the Panel the Objects with Text or Icon

```
.xfce4-panel.background button {
  color: rgba(255,255,255,0.6);
  text-shadow: text-shadow: 0px 1px alpha(#000000, 0.40),0 -1px alpha(#ffffff, 0.04),-1px  0px alpha(#ffffff, 0.04);
  background-image: none;
  background-color: #2c2b2c;
  border-radius: 0;
  border: none;
  box-shadow: none;
  padding: 0 1px;}
```

```
.xfce4-panel.background button {
  -gtk-icon-shadow:none;
  color: #1c1c1c;
  text-shadow: ;
  background-image: none;
  background-color: #ffffff;
  border-radius: 0;
  border: none;
  box-shadow: none;
  padding: 0 1px;}
```


Background: #ffffff
Forground: #1c1c1c

