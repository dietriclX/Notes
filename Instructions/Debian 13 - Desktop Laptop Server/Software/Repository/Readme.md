# [Debian 13 - Desktop Laptop](../../Readme.md) > [Software](../Readme.md) > Repository

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

Debian provides many different ways of getting software installed on a machine. Most commonly used is the Debian Repository with packages published by the Debian-Team or by Others.

## Debian Repository (maintained by Debian-Team)

### Add Debians own `contrib` and `non-free`

```console
cp /etc/apt/sources.list /etc/apt/sources.list.ORG
sed --in-place --expression='s/main non-free-firmware/main contrib non-free non-free-firmware/g' /etc/apt/sources.list
apt update
```

## Debian Repository (maintained by Others)

### Add `Brave Browser`

sudo apt install curl
sudo curl -fsSLo /usr/share/keyrings/brave-browser-archive-keyring.gpg https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
sudo curl -fsSLo /etc/apt/sources.list.d/brave-browser-release.sources https://brave-browser-apt-release.s3.brave.com/brave-browser.sources
sudo apt update

### Add `Mullvad Browser`

```console
curl -fsSLo /usr/share/keyrings/mullvad-keyring.asc https://repository.mullvad.net/deb/mullvad-keyring.asc
echo "deb [signed-by=/usr/share/keyrings/mullvad-keyring.asc arch=$( dpkg --print-architecture )] https://repository.mullvad.net/deb/stable $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/mullvad.list
apt update
```

### Add `Signal`

```console
wget -O- https://updates.signal.org/desktop/apt/keys.asc | gpg --dearmor > signal-desktop-keyring.gpg
cat signal-desktop-keyring.gpg | tee /usr/share/keyrings/signal-desktop-keyring.gpg > /dev/null
echo 'deb [arch=amd64 signed-by=/usr/share/keyrings/signal-desktop-keyring.gpg] https://updates.signal.org/desktop/apt xenial main' | tee /etc/apt/sources.list.d/signal-xenial.list
apt update
```

## External Debian Repository

### Add `Librewolf`

```console
apt update && apt install extrepo -y
extrepo enable librewolf
apt update
```
