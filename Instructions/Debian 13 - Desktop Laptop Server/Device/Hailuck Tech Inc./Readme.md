# [Debian 13 - Desktop Laptop](../../Readme.md) > [Device](../Readme.md) > Hailuck Tech Inc.

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

## Touchpad

### HAILUCK CO.,LTD Duet 3 USB Composite Device Touchpad

To be identified via users terminal session (X11).

```console
$ xinput --list
:
⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
:
⎜   ↳ HAILUCK CO.,LTD Duet 3 USB Composite Device Touchpad	id=13	[slave  pointer  (2)]
:
```

Note: *The name before `id=13` has to be used as value for `Identifier`.*

#### Click by Tap with 1,2,3 fingers

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
