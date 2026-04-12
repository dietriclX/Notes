# [Debian 13 - Desktop Laptop](../../Readme.md) > [Browser](../Readme.md) > Chrome and Variants

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

## UI Language

An option to change the UI language, system-wide, is to modify the Start-Script (from the The Chromium Authors) which starts the browser. For example for English UI, add the line `export LANGUAGE=EN_us` to the top of the script.

| Browser Product   | Start-Script                       |
| :---------------- | :--------------------------------- |
| Google Chrome     | /opt/google/chrome/google-chrome   |
| Brave Web browser | /opt/brave.com/brave/brave-browser |

=== Start-Script ===

```sh
#!/bin/bash
#
# Copyright 2011 The Chromium Authors
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

export LANGUAGE=EN_us

# Let the wrapped binary know that it has been run through the wrapper.
export CHROME_WRAPPER="`readlink -f "$0"`"

:
```

Based on [superuser: How to change language of google chrome in ubuntu](https://superuser.com/questions/1360872/how-to-change-language-of-google-chrome-in-ubuntu)

## Brave Browser

### Brave using RAM-Disk as Cache (`/run/user/UID/brave-cache`)

In one directory create the two scripts `Create-Brave-Browser-Menu.sh` and `Create-Brave-Browser-Dock.sh`. Make both executable.

- `Create-Brave-Browser-Menu.sh`: Creates a users own `.desktop`file for the browser, for using the RAM-Disk as cache.
- `Create-Brave-Browser-Dock.sh`: Creates a launcher in Cairo-Dock for the browser. Will internall execute the 1st script.

=== `Create-Brave-Browser-Menu.sh` ===

```sh
#!/bin/bash

# identify users runtime directory e.g. /run/user/1000
if [ -n "$XDG_RUNTIME_DIR" ]; then
  USERS_RUNTIME_DIR=$XDG_RUNTIME_DIR
elif [ -n "$UID" ]; then
  USERS_RUNTIME_DIR=/run/user/$UID
else
  USERS_RUNTIME_DIR=/run/user
fi

# convert for example /run/user/1000 into \/run\/user\/1000
SED_USERS_RUNTIME_DIR="${USERS_RUNTIME_DIR//\//\\\/}"

# create modified version "Brave Web Browser (RAMDisk)" in file  $HOME/.local/share/applications/com.brave.Browser.desktop
cp /usr/share/applications/com.brave.Browser.desktop $HOME/.local/share/applications
sed --in-place \
    --expression="s/^\(Name=Brave Web Browser\)$/\1 (RAMDisk)/" \
    --expression="s/^\(Exec=\/usr\/bin\/brave-browser-stable\)/\1 --disk-cache-dir=$SED_USERS_RUNTIME_DIR\/brave-cache --password-store=basic/" \
    $HOME/.local/share/applications/com.brave.Browser.desktop

# create modified version "Brave Web Browser (RAMDisk)" in file  $HOME/.local/share/applications/brave-browser.desktop
cp /usr/share/applications/brave-browser.desktop $HOME/.local/share/applications
sed --in-place \
    --expression="s/^\(Name=Brave Web Browser\)$/\1 (RAMDisk)/" \
    --expression="s/^\(Exec=\/usr\/bin\/brave-browser-stable\)/\1 --disk-cache-dir=$SED_USERS_RUNTIME_DIR\/brave-cache --password-store=basic/" \
    $HOME/.local/share/applications/brave-browser.desktop
```

=== `Create-Brave-Browser-Dock.sh` ===

```sh
#!/bin/bash

source $(dirname "$0")/Create-Brave-Browser-Menu.sh

cat << EOF > $HOME/.config/cairo-dock/current_theme/launchers/01brave-browser.desktop
#3.4.1

#[gtk-about]

[Desktop Entry]

#F[Icon]
frame_maininfo=

#d+ Name of the container it belongs to:
Container=_MainDock_

#v
sep_display=

#s[Default] Launcher's name:
Name=

#S+[Default] Image's name or path:
Icon=

#s[Default] Command to launch on click:
#{Example: nautilus --no-desktop, gedit, etc. You can even enter a shortkey, e.g. <Alt>F1, <Ctrl>c,  <Ctrl>v, etc}
Exec=


#X[Extra parameters]
frame_extra=

#b Don't link the launcher with its window
#{If you chose to mix launcher and applications, this option will deactivate this behaviour for this launcher only. It can be useful for instance for a launcher that launches a script in a terminal, but you don't want it to steal the terminal's icon from the taskbar.} 
prevent inhibate=false

#K[Default] Class of the program:
#{The only reason you may want to modify this parameter is if you made this launcher by hands. If you dropped it into the dock from the menu, it is nearly sure that you shouldn't touch it. It defines the class of the program, which is useful to link the application with its launcher.}
StartupWMClass=brave-browser

#b Run in a terminal?
Terminal=false

#i-[0;16] Only show in this specific viewport:
#{If '0' the launcher will be displayed on every viewport.}
ShowOnViewport=0

#f[0;100] Order you want for this launcher among the others:
Order=-0.60009765625

Icon Type=0
Type=Application
Origin=$HOME/.local/share/applications/brave-browser.desktop
EOF
pkill cairo-dock
sleep 1s
cairo-dock -o &
```

### Delete Everything

```console
rm -Rf $HOME/.config/BraveSoftware $HOME/.cache/BraveSoftware/Brave-Browser
```

### Brave Recommendation

In Brave Web Browser, tap on `≡` to open to the menu followed by  tap on "**⚙ Settings**"

In the "**⚙ Settings**" of Brave follow the [instructions](https://github.com/dietriclX/Notes/tree/main/Instructions).

- ⬆ click on "**Get started**" in the left panel
  - keep `Continue where you left off` enabled as option off "**On startup**"
  - 🖉 select `Blank page` as "**New tab page shows**"
- ⬆ click on "**Appearance**" in the left panel
  - 🖉 click on "**Use GTK**" to get "**Theme: GTK**"
	- 🖉 enable "**Use system title bar and borders**"
	- 🖉 ~~disable~~ "**Brave News button**"
	- 🖉 ~~disable~~ "**Brave Leo AI button**"
	- 🖉 ~~disable~~ "**Brave Rewards button**"
	- 🖉 ~~disable~~ "**Brave Wallet button**"
	- 🖉 ~~disable~~ "**Enable bottom toolbar**"
  - keep "**Show autocomplete suggestions in address bar**"
	- 🖉 ~~disable~~ "**Top Suggestions**"
	- 🖉 ~~disable~~ "**Quick commands**"
	- 🖉 ~~disable~~ "**Leo AI Assistant**"
- ⬆ click on "**Content**" in the left panel
	- 🖉 ~~disable~~ "**Show Wayback Machine prompt on 404 pages**"
- ⬆ click on "**Shields**" in the left panel
	- 🖉 ~~disable~~ "**Show the number of blocked items on the Shields icon**"
	- 🖉 select `Aggressive` as "**Trackers & ads blocking**"
  - keep "**Block fingerprinting**" enabled
	- 🖉 enable "**Forget me when I close this site**"
	- 🖉 ~~disable~~ "**Store contact information for future broken site reports**"
	- ⬆ click on "**Content Filtering**"
		- 🖉 click on "**Update lists**"
		- 🖉 enable every variation of ¹
			- "**AdGuard**" 
			- "**EasyList**" 
			- "**Fanboy's**"
			- "**Blocklists Anti-Porn**"
		- 🔙 navigate back
	- 🖉 ~~disable~~ "**Allow Facebook logins and embedded posts**"
	- 🖉 ~~disable~~ "**Allow X (ppreviously Twitter) embedded tweets**"
  - keep "**Allow Linkedin embedded posts**" disabled
- ⬆ click on "**Privacy and security**" in the left panel
	- ⬆ click on "**Delete browsing data**"
    - navigate to tab "**Basic**"
	  - 🖉 select `All time` as "**Time range**"
		- 🖉✓ ~~disable~~ "**Browsing history**"
    - click on "**Delete data**"
	- ⬆ click on "**Delete browsing data**"
    - navigate to tab "**On exit**"
		- 🖉✓ enable "**Cookies and other site data**"
		- 🖉✓ enable "**Leo AI**"
		- 🖉✓ enable "**Passwords and other sign-in data**"
		- 🖉✓ enable "**Autofill form data**"
		- 🖉✓ enable "**Hosted app data**"
    - ✓🔙 click on "**Save**"
	- ⬆ click on "**Security**"
    - tbd
		- 🔙 navigate back
	- ⬆ click on "**Site and Shields Settings**"
    - ⬆ click on "**Location**"
      - 🖉 select "**Don't allow sites to see your location**"
      - 🔙 navigate back
    - ⬆ click on "**Camera**"
      - 🖉 select "**Don't allow sites to use your camera**"
      - 🔙 navigate back
    - ⬆ click on "**Microphone**"
      - 🖉 select "**Don't allow sites to use your microphone**"
      - 🔙 navigate back
    - ⬆ click on "**Notifications**"
      - 🖉 select "**Don't allow sites to send notifications**"
      - 🔙 navigate back
    - click on "**Additional content settings**"
    - tbd
    - 🔙 navigate back
  - keep "**Auto-redirect AMP pages**" enabled
  - keep "**Auto-redirect tracking URLs**" enabled
  - keep "**Prevent sites from fingerprinting me based on my language preferences**" enabled
	- 🖉✓⬆ enable "**Send a 'Do not Track' request with your browsing traffic**"
    - ✓🔙 click on "**Confirm**"
  - 🖉 ~~disable~~ "**Private windows with Tor**"
  - 🖉 ~~disable~~ "**Allow privacy-preserving product analytics (P3A)**"
  - 🖉 ~~disable~~ "**Automatically send daily usage ping to Brave**"
  - 🖉 ~~disable~~ "**Automatically send diagnostic reports**"
- ⬆ click on "**Web3**" in the left panel
    - 🖉 select `Extensions (no fallback) "**Default Ethereum wallet**" 
    - 🖉 select `Extensions (no fallback) "**Default Solana wallet**" 
- ⬆ click on "**Leo**" in the left panel
  - 🖉 ~~disable~~ "**Show Leo icon in the sidebar**"
  - 🖉 ~~disable~~ "**Show Leo in the context menu on websites**"
  - 🖉 ~~disable~~ "**Tab Focus Mode**"
  - 🖉✓⬆ ~~disable~~ "**Store my conversation history**"
    - ✓🔙 click on "**OK**"
- ⬆ click on "**Search engine**" in the left panel
  - ⬆ click on "**Change**" for "**Normal Windows**"
    - 🖉✓ select "**Qwant**"
    - ✓🔙 click on "**Set as default**"
  - ⬆ click on "**Change**" for "**Private Windows**"
    - 🖉✓ select "**Qwant**"
    - ✓🔙 click on "**Set as default**"
  - 🖉 ~~disable~~ "**Improve search suggestions**"
- ⬆ click on "**Extensions**" in the left panel
  - tbd
- ⬆ click on "**Autofill and passwords**" in the left panel
  - ⬆ click on "**Password Manager**"
    - ⬆ click on "**⚙ Settings**" in the left panel (of the Password Manager)
      - 🖉 ~~disable~~ "**Offer to save passwords and passkeys**"
      - 🖉 ~~disable~~ "**Sign in automatically**"
      - 🔙🛝 close browser tab "**Password Manager**"
  - ⬆ click on "**Payment Method**"
    - 🖉 ~~disable~~ "**Save and fill payment methods**"
    - 🖉 ~~disable~~ "**Allow sites to check if you have payment methods saved**"
    - 🔙 navigate back
  - ⬆ click on "**Addresses and more**"
    - 🖉 ~~disable~~ "**Save and fill addresses**"
    - 🔙 navigate back
  - 🖉 ~~disable~~ "**Allow auto-fill in private windows**"
- ⬆ click on "**Languages**" in the left panel
  - 🖉 ~~disable~~ "**Check for spelling errors when you type text on web pages**"
  - 🖉 ~~disable~~ "**Use Brave Translate**"
- ⬆ click on "**System**" in the left panel
  - 🖉 ~~disable~~ "**Continue running background apps when Brave is closed**"
  - 🖉 ~~disable~~ "**Warn me before closing window with multiple tabs**"

¹ Here I fully rely on the trustworthy [author](https://www.kuketz-blog.de) and his [german explanation on  uBlock Origin usage](https://www.kuketz-blog.de/ublock-origin-schutz-gegen-tracker-und-werbung/). Thanks a million.
I do not know how to deal with "Filterliste »[Actually Legitimate URL Shortener Tool](https://raw.githubusercontent.com/DandelionSprout/adfilt/master/LegitimateURLShortener.txt)« über die Importfunktion hinzufügen" and therefore leaving it out for the moment. Once the wisdom has kissed me, I will incorporate this part as well.
