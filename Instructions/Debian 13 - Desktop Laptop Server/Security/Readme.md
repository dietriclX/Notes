# [Debian 13 - Desktop Laptop](../Readme.md) > Security

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

- The Server
	- Host name: `dlhostdl`
- The Workstation
  - Software
  	- OS: Debian 13 "trixie"
  - OS Users
    - Regular User
      - Account: `dlsysuserdl`

## Authentication

Simplifies the authentication on the Workstation.

### Enable FIDO2 authentication

Follow instruction on [FIDO2 Key](#fido2-key). Once the regular user `dlsysuserdl` can login using a FIDO2 key, define a strong password for that user.

#### sudo thanks to FIDO2

With the regular user being secured it can become a sudoer.

```console
usermod --groups sudo --append dlsysuserdl
```

## Certificate

### Install CA certificate

Install the CA Certificate `own-ca.crt` in the following way.

```console
mkdir --mode=755 /usr/local/share/ca-certificates/my-custom-ca
cp own-ca.crt /usr/local/share/ca-certificates/my-custom-ca
update-ca-certificates
```

## FIDO2 Key

Software Package `fido2-tools` has to be installed

```console
apt install --yes fido2-tools libpam-u2f
```

Before adjusting the pam configuration, create the file `u2f.keys` file for storing the public FIDO2 keys.

```console
mkdir --mode=755 /etc/fido2/
touch /etc/fido2/u2f.keys
chmod 644 /etc/fido2/u2f.keys
```

Insert the `auth sufficient pam_u2f.so authfile=...` line into `common-auth`.

Starting with the first FIDO key. The generated line `dlsysuserdl:...,es256,+presence` has to be inserted into`u2f.keys`.

```console
# pamu2fcfg --username=dlsysuserdl
Enter PIN for /dev/hidraw3: 
dlsysuserdl:PK1BY...ruw==,d71ti...ixWw==,es256,+presence
```

As in `u2f.keys` there is only one line per user, any further generated output of `pamu2fcfg` (of additional keys) have to be added to the users line without the user id.

=== `/etc/pam.d/common-auth` ===

```code
:
# local modules either before or after the default block, and use
# pam-auth-update to manage selection of other modules.  See
# pam-auth-update(8) for details.

auth sufficient pam_u2f.so authfile=/etc/fido2/u2f.keys cue [cue_prompt=Touch Me]

# here are the per-package modules (the "Primary" block)
auth	[success=1 default=ignore]	pam_unix.so nullok
:
```

===`/etc/fido2/u2f.keys` (example for two FIDO keys) ===

```code
dlsysuserdl:PK1BY...ruw==,d71ti...ixWw==,es256,+presence:ODP4S...sht==,d8w9r...gfEw==,es256,+presence
```

## Keyring / Password-Store

Most commonly used is the GNOME Keyring with `seahorse` as it managing tool. The password for this certificate/key/login storage is set at its creation time and is not in sync with the systems password.

### Remove Password

From the security aspect not really recommended, but is making the life much easier. Nevertheless avoid using the Password Manager of the Browser and use instead a Password Manager like KeePass, as it makes someone independent of the Browser Software.

Follow instruction on [Update Password](#update-password), but leave the fields for the new password empty.

### Update Password

- ⬆ select program "**Password and Keys**" (by searching for `seahorse`)
    - navigate to back by a click on `<`
    - click on "**Passwords > Login**"
    - ⬆ click on "**Unlock**"
        - 🔙 enter the first password used for the account
    - navigate to back by a click on `<`
    - ⬆ select "**Change Password**" from context of "**Passwords > Login**"
        - enter the password
        - ✓🔙 click "**Continue**"
        - ✏️✓ enter in the fields the new password
        - ✓🔙 click "**Continue**"

## Passwort Manager

### KeePass

Installation

```console
apt install --yes keepass2 xdotool mono-complete
```

Note: *The packages `xdotool` and `mono-complete` are required to use the "**Auto-Type**" feature of KeePass.*

#### QR-Code Generator `QrCodeGenerator`

1. open [Plugins - KeePass](https://keepass.info/plugins.html#qrcodegen)
2. click on the hyperlink next to "Download plugin:".
3. extract the downloaded archive
4. copy the content (`.plgx` file) of the archive to ` /usr/lib/keepass2/Plugins/`
 into directory 

```console
# ls -la /usr/lib/keepass2/Plugins/
:
-rw-r--r-- 1 root root   66560 Jan  3  2021 KeePassHttp.dll
-rw-r--r-- 1 root root 1194702 Nov 16 15:19 QrCodeGenerator.plgx
```

## SSH

- dlsysuserdl: the regular user on workstation and remote machine
- dlhostdl: remote machine

### Basic SSH Setup

Keeps the ability to use either password or public key for authentication.

Copy the ssh public key onto the remote machine.

```console
$ ssh-copy-id dlsysuserdl@dlhostdl
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/dlsysuserdl/.ssh/id_rsa.pub"
The authenticity of host 'dlhostdl (<IP-Address>)' can't be established.
ED25519 key fingerprint is SHA256:td4yIx6Ap/66c5rweUl/HxGC0W1gLww3rwdoHeR9Ang.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
dlsysuserdl@dlhostdl's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'dlsysuserdl@dlhostdl'"
and check to make sure that only the key(s) you wanted were added.
```

```code
KbdInteractiveAuthentication no
```

### Expert SSH Setup

Restricts the authentication to public key only and limit the authorization to the administrator only.

The administrator maintains the users public keys in the individual files `/etc/ssh/authorized_keys.d/<user-id>`.

Assuming user `dlsysuserdl` already used command `ssh-copy-id dlsysuserdl@dlhostdl` to get the public key onto the server.

```console
mkdir --mode=755 /etc/ssh/authorized_keys.d
mv /home/dlsysuserdl/.ssh/authorized_keys /etc/ssh/authorized_keys.d/dlsysuserdl
chown root:root /etc/ssh/authorized_keys.d/dlsysuserdl
chmod 644 /etc/ssh/authorized_keys.d/dlsysuserdl
cat << EOF > /etc/ssh/sshd_config.d/90-authorized_keys.conf
# Public keys of users maintained in files /etc/ssh/authorized_keys.d/<user-id> .
# The file name is identical to the User-ID.
AuthorizedKeysFile /etc/ssh/authorized_keys.d/%u
EOF
systemctl restart ssh
```

=== /etc/ssh/sshd_config.d/90-authorized_keys.conf ===

```config
# Public keys of users maintained in files /etc/ssh/authorized_keys.d/<user-id> .
# The file name is identical to the User-ID.
AuthorizedKeysFile /etc/ssh/authorized_keys.d/%u
```
