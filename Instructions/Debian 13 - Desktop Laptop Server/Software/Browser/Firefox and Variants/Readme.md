# [Debian 13 - Desktop Laptop](../../Readme.md) > [Browser](../Readme.md) > Firefox and Variants

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

- The Workstation
  - Software
  	- OS: Debian 13 "trixie"
  - OS Users
    - Regular User
      - Account: `dlsysuserdl`
      - Account ID: `dlsysuseriddl` (identified by `echo $UID` e.g. `1000`)

##  Enable for Touch Screen / Tablet

### via `about:config`

Open `about:config` and add/adjust the following parameters.

| Parameter | Value | Comment |
| ---: | :--- | :--- |
| dom.w3c_touch_events.enabled | 1 | page scrolling with gesture on touch screen |

### via `/etc/environment`

As an alternative to set property `dom.w3c_touch_events.enabled` inside the browser, setting the variable `MOZ_USE_XINPUT2=1` does the same.

```console
cat << EOF > /etc/environment
MOZ_USE_XINPUT2=1
EOF
```

Note: *Reboot is required.*

Based on [yethiel@gitlab: posts / Firefox Kinetic Scrolling | 2020-04-07](https://yethiel.gitlab.io/post/firefox-kinetic-scrolling/)

## Firefox

### Firefox using RAM-Disk as Cache (`/run/user/UID/firefox-cache`)

Open `about:config` and add/adjust the following parameters.

| Parameter | Value | Comment |
| ---: | :--- | :--- |
| browser.cache.disk.parent_directory | /run/user/dlsysuseriddl/firefox-cache | on RAM-Disk |

## LibreWolf

### LibreWolf using RAM-Disk as Cache (`/run/user/UID/librewolf-cache`)

Open `about:config` and add/adjust the following parameters.

| Parameter | Value | Comment |
| ---: | :--- | :--- |
| browser.cache.disk.parent_directory | /run/user/dlsysuseriddl/librewolf-cache | on RAM-Disk |

## Mullvad Browser

### Mullvad using RAM-Disk as Cache (`/run/user/UID/mullvad-cache`)

Open `about:config` and add/adjust the following parameters.

| Parameter | Value | Comment |
| ---: | :--- | :--- |
| browser.cache.disk.parent_directory | /run/user/dlsysuseriddl/mullvad-cache | on RAM-Disk |
