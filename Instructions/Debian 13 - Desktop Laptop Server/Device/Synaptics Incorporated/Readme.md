# [Debian 13 - Desktop Laptop](../../Readme.md) > [Device](../Readme.md) > Synaptics Incorporated

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

## Touchpad

### SynPS/2 Synaptics TouchPad

Install the X11 driver.

```
apt install --yes xserver-xorg-input-synaptics
```

To be identified via users terminal session (X11).

```console
$ xinput --list
⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
:
⎜   ↳ SynPS/2 Synaptics TouchPad              	id=21	[slave  pointer  (2)]
:
```

#### Click by Tap with 1,2,3 fingers

And allow scrolling with two finger, were as the vertical scrolling is reverse.

```console
cat << EOF > /etc/X11/xorg.conf.d/90-elantech.conf
Section "InputClass"
    Identifier "SynPS/2 Synaptics TouchPad"
    Driver "synaptics"
    MatchIsTouchpad "on"
		# Which mouse button is reported on a non-corner X-finger tap.
        Option "TapButton1" "1"
        Option "TapButton2" "3"
        Option "TapButton3" "2"
	# Enable vertical scrolling when dragging with two fingers anywhere on the touchpad.
	Option "VertTwoFingerScroll" "on"
	Option "VertScrollDelta" "-111"
	Option "HorizTwoFingerScroll" "om"
EndSection
EOF
```

Based on

- [debian Wiki: SynapticsTouchpad](https://wiki.debian.org/SynapticsTouchpad)
- [archlinux: Touchpad Synaptics](https://wiki.archlinux.org/title/Touchpad_Synaptics)
