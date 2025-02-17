
# Basics

After the installation some activities are handy to have documented.

## Maintenance

### Maintenance-Mode ON

```console
sudo -u www-data php ncwebdirectorync/occ maintenance:mode --on
```

### Maintenance-Mode OFF

```console
sudo -u www-data php ncwebdirectorync/occ maintenance:mode --off
```

## Start

Manually starting all instances of a Nextcloud installation.

```console
systemctl start postgresql redis
systemctl start apache2 coturn ds-converter ds-docservice ds-metrics nginx php8.2-fpm
```

## Stop

Manually stopping all instances of a Nextcloud installation.

```console
systemctl stop apache2 coturn ds-converter ds-docservice ds-metrics nginx php8.2-fpm
systemctl stop postgresql redis
```

## OpenVPN

Access the Management Interface

```console
telnet localhost 7505
```

# `scripts` Directory

The directory contains a few helpful files to assist you in setting up and maintaining the system.

At one place, you specify the values for the Parameters in the `prepare_machine.*.sh` file(s). Allowing the information to be used during the setup (translate `Readme.md`) and the days after when everything is running (e.g. `variables.sh`, `backup.sh`, ...).

Using the `apply.sh` script, your values will be applied to any file. Making any document easier to read and the commands - in the same - tailored to your settings, just like for any shell script.

The `backup.sh` script is for the cron job to every night and create a backup on an external USB HDD. With every backup a `restore.sh` is generated to simplify and speed-up the restore operation.

## The Directory Itself

Download the content of directory [Debian 12 - ACNO > Usage > scripts](./scripts) onto your machine `aohostao`. Put it into the root home directory `/root/scripts/acno`.

Directory `~/scripts/acno`

| File | Description |
| :--- | :--- |
| `apply.sh` | applies the values for Parameters in any kind of file |
| `backup.sh` | automated script for being called from a cron job |
| `prepare_machine.ORG.sh` | prepares a newly setup machine |
| `variables.sh` | this file has to be generated |

Directory `~/scripts/acno/data`

| File | Description |
| :--- | :--- |
| `parameter_value.Default.map` | default values for the Parameters |
| `parameter_value.MIN.map` | absolute minimum of what you have to specify |
| `parameter_value.PROD.map` | minimum of what you have to specify for a production system |
| `variables.ORG.sh` | kind of translation from Parameters to environment variables and the basis of `variables.sh` |

Make sure the shell scripts have the "x" attribute set and all files can only read/(over-)written by root.

```console
chmod 700 ~/scripts/acno/*.sh
chmod 600 ~/scripts/acno/data/*
```

## The Parameter-Value Mapping

In the `data` directory you will find those `parameter_value.*.map` files, which are used to specify the values for the various Parameters. The `apply.sh` script uses these files for the translation.

The `parameter_value.*.map` files can be used in two flavours. Were as Variant B.) can be used as well by those who want to maintain only the bare minimunm.

A.) For a single machine with not backup machine.

You need only file `parameter_value.Default.map`.
Translation can be done using command e.g. `apply.sh file`. Option `-i TYPE`/`--instance=TYPE` is not used.

B.) For a single machine with backup machine available.

You maintain in file `parameter_value.Default.map` everything which all machines have in common. In another file e.g. `parameter_value.PROD.map` you would maintain what specific to the PROD machine.
Translation can be done using command e.g. `apply.sh --instance=PROD file`

For those in a hurry, would maintain their specifici values in file `parameter_value.MIN.map`
Translation can be done using command e.g. `apply.sh --instance=MIN file`

Note: The option `-i TYPE`/`--instance=TYPE` is doing nothing else than using `parameter_value.TYPE.map` in addition to `parameter_value.Default.map`.

Note: It is on you, to maintain the Parameter-Value-Mapping in these files, as it is environment specific and mission critical! Especially in regards to the backup script.

The format of these `.map` files is self-explanatory. Take for example the backup device mount point aka `aomountdirdeviceao`.

=== `data/parameter_value.Default.map` ===

```code
:

# Backup: Directory aka Mount Point
aomountdirdeviceao   /mnt/backup

:
```

## The Shell-Scripts

### `apply.sh`

The `apply.sh` is the shell script to apply the Parameter-Value-Mapping to the scripts or other files e.g. a `Readme.md` file.

Before you start any other shell script of the `scripts` directory, the `.map` file(s) to `variables.ORG.sh`.

With only `parameter_value.Default.map` maintained.

```console
cd ~/scripts/acno
./apply.sh variables.ORG.sh variables.sh
```

With both files `parameter_value.Default.map` and `parameter_value.MIN.map` maintained.

```console
cd ~/scripts/acno
./apply.sh --instance=MIN variables.ORG.sh variables.sh
```

Note: If `apply.sh` gets started without any Parameters or with incorrect Parameter(s), the script prints out the usage information. In case you are getting the information `usage: applygnupgdefaults` ... you haven't started the shell script `apply.sh`  ... at least that happened to me :) ... well, well, that happens if the current directory is not part of `PATH` ... keep it that way! And do not add `~/scripts/acno` to `PATH` ... as soon as you have the `scripts` directory on your backup device, you could get wiered results! Trust me ...

### `backup.sh` 

The `backup.sh` is the shell script you want to call in your daily/nightly cron job. The script relies on an up-to-date version of `variables.sh`.

### `download.sh`

???

### `prepare_machine.sh`

The `prepare_machine.sh` script can be used to prepare a fresh machine¹, to apply afterwards the `restore.sh` shell script (generated by the `backupo.sh` script).

¹ With regards to [Setup > Readme.md](../Setup/Readme.md), only the Debian system has been installed (as described) and no "**Post activities**" were started.

Note: The shell script `prepare_machine.sh` would be started on a fresh machine using the backup device. That might require to adjust some of the Parameters values, results in updating the Parameter-Value-Mapping file(s) (`.map`) and at the end "updating" the shel script `variables.sh`.

### `restore.sh`

Does not exist ... yet. This script gets created by the `backup.sh`.

Note: The shell script `restore.sh` would be started on a well prepared machine using the backup device. That might require to adjust some of the Parameters values, results in updating the Parameter-Value-Mapping file(s) (`.map`) and at the end "updating" the shel script `variables.sh`.

### `upload.sh`

The `upload.sh` shell script gets called by the `backup.sh` script, in case the feature Backup to WebDAV is enabled.

Note: I had used this feature several times. The upload to another Nextcloud instance (from a Telcom) was never successfull. Even checking after upload and re-upload was not reliably working.

### `variables.ORG.sh`

The shell script `variables.ORG.sh` is nothing more than a list of various environment variables mapped to the corresponding Parameter.

Take for example the backup device mount point aka `aomountdirdeviceao` and its equivalent environment variable `MNT_DIR_DEVICE`.

=== `data/variables.ORG.sh` ===

```console
:
MNT_DIR_DEVICE=aomountdirdeviceao
:
```

## Backup

### Prepare `aomountdirdeviceao` Directory

Make sure you have always a disk attached to the server `aohostao`. In this example it is `/dev/sda`, whose first partition will be mounted on `aomountdirdeviceao`.

```console
# mkdir --mode=770 aomountdirdeviceao
# chown root:backup aomountdirdeviceao
# mount /dev/sdb1 aomountdirdeviceao
# chown root:backup aomountdirdeviceao
# ls aomountdirdeviceao
lost+found
```

### Prepare `fstab`

Identify the UUID of your backup device, by attaching the device and executing the command.

Best practise is, to first execute `lsblk` without the backup device attached and on the second execution you see which line was added in the output. In my example the `/dev/sdb` had been the backup device.

```console
# lsblk -o PATH,TYPE,MOUNTPOINT,UUID
PATH      TYPE MOUNTPOINT UUID
/dev/sda  disk            
/dev/sda1 part /          3b54bcb4-b772-4fee-a146-297f9fbb4981
/dev/sdb  disk            
/dev/sdb1 part            bb09a4f1-fa6a-435b-b1f5-0c1db8bb8c2f
```

Next is to add a line in `fstab` (above `tmpfs` entries) for this particular device, for mount point `aomountdirdeviceao` with the attribute `noauto` to prevent a mount at boot time.

=== `/etc/fstab` ===

```code
:

# Backup devices
UUID=bb09a4f1-fa6a-435b-b1f5-0c1db8bb8c2f aomountdirdeviceao         ext4    noauto,errors=remount-ro 0       2

:
```

### Setup cron job

Before you enable the scron job using the `backup.sh` shell script, you have to test/execute the script with all the up-to-date Parameter-Value-Mapping `.map` file(s) (and depending `.sh`files`).

Once the `backup.sh` shell script works as expected, it can be executed every night at 01:15 . Simply insert the following lines into crontab for root (command `crontab -e`).

```code
# --- START OF ACNO SPECIFIC DEFINITIONS --- #
15 1 * * * /root/scripts/acno/backup.sh
```

*Note: The internet offers a variety of tools to help with cron job scheduling. For example [Cronitor](https://crontab.guru)*

At the end of the script, the log file can be send to the administrators email address. If preferred even pgp encrypted.

## Backup TEST

Take a spare machine or Virtual Machine (VM), pretend you run into the "Second worst scenario" and have to restore the whole system.

## Restore

Even if the restore is labor intensive ... do not forget to setup on the new machine the backup!

### Available Backup

Take a look at the latest backup...

```console
# mount aomountdirdeviceao
# cd aomountdirdeviceao
# ls <backup date>
apache2.tar.gz	coturn.tar.gz  ctdatabasect.pgsql  ncdatabasenc.pgsql  nextcloud.tar.gz  nginx.tar.gz  onlyoffice.tar.gz  oodatabaseoo.pgsql  osandco.tar.gz  os_changes.sh  php.tar.gz  postgresql.tar.gz  redis.tar.gz  var_log.tar.gz  www-data.cron.backup
```

Worst case scenario ... No, let us be optimistic ...

### Second worst scenario

You have a backup from last night and your disk got afterwards corrupt. You can now try to figure out what information on the disk can be still used, or you go straight to the actions of newly setting up the system.

1. make sure your router is no longer forwarding internet requests to your system
2. setup the system from the scratch (primarily ensure the same software is installed, as from last backup) 
3. restore all configuration and data

### Third worst scenario

You have a backup from last night and later you screwed one of the services completely. Either Nextcloud, ONLYOFFICE, or-what-so-ever is no longer working.

1. identify which parts of the system are in good shape and can be still used
2. identify which parts of the system have to be replaced by a backup
3. get the corrupt parts reinstalled to ensure they are in default configuration
4. restore the configuration and data of those parts - currently - in default configuration

### Restore Aktivities

In a backup directory you will find everything from the `scripts/acno` folder - as listed above - plus a shell script `restore.sh`. The script can be used to run from A-Z or you modify the file by removing/adjusting those parts that in your backup scenario are irrelevant or require adjustments.

## Rollback

You must have a backup in place, in case the rollback bricks your system.

### ONLYOFFICE

At first, you should identify the current version and the one before. Assuming that the "one before" was working fine.

```console
# apt list --all-versions onlyoffice-documentserver
Listing... Done
onlyoffice-documentserver/squeeze,now 8.2.0-143 amd64 [residual-config]
onlyoffice-documentserver/squeeze 8.1.3-4 amd64 [residual-config]
onlyoffice-documentserver/squeeze 8.1.1-26 amd64 [residual-config]
:
```

Once you have identify the version number, you can

1. uninstall the currently installation
2. install the version, you want to go back to

Example: The released version 8.2.0-143 did not start (`Cannot GET /8.2.0-<hec number>/web-apps/apps/.../main/index_loader.html`). So I did a rollback to version 8.1.3-4 . I marked that version to be not automatically updated ... at least not before a newer version got released.

```console
apt remove onlyoffice-documentserver
apt install onlyoffice-documentserver=8.1.3-4
apt-mark hold onlyoffice-documentserver
```

## Update

### General

Keep your OS and your applications up-to-date.

- on OS-level regular run `apt update`/`apt upgrrade`
- check for new Nextcloud updates and apply these
    - [GitHub Releases](https://github.com/nextcloud/server/releases)
    - [nextcloud.com Releases](https://download.nextcloud.com/server/releases/)
- check for new Nextcloud-Apps updates and apply these
    - In your Nextcloud instance, choose Administrator-Menu option "+ Apps" and look at "Update aoos" in the left panel
    - Execute commands `...occ app:update --showonly`/`...occ app:update --all` (or individual app)

### Nextcloud

Basically you do the following

1. download & extract the new release
2. shutdown (nearly) everything
3. change existing Nextcloud directory into a backup version `ncwebdirectorync.bak`
4. change directory of the new release to `ncwebdirectorync`
5. take over, from the backup, those apps not part of the distribution
6. correct the ownership and some attributes of the release
7. startup everything, which got shutdown in step 2.
8. initiate the `occ upgrade` to upgrade the database as well (and maybe other stuff as well)

#### Variant A

```console
cd ncwebdirectorync
cd ..
git clone  --single-branch --recurse-submodules --shallow-submodules --branch=v30.0.4 --depth 1 https://github.com/nextcloud/server.git
sudo --user=www-data php ncwebdirectorync/occ maintenance:mode --on
systemctl stop apache2 php8.2-fpm notify_push coturn nginx ds-converter ds-metrics ds-docservice
mv ncwebdirectorync ncwebdirectorync.BAK
mv server ncwebdirectorync
cd ncwebdirectorync.BAK
cp config/config.php ncwebdirectorync/config
cd apps
echo Take over Nextcloud Apps
ls --directory * | while read d
do
  if [ ! -d "ncwebdirectorync/apps/$d" ]; then
    echo $d
    cp --recursive "$d" "ncwebdirectorync/apps"
  fi
done
chown --recursive --no-dereference www-data:www-data ncwebdirectorync
find ncwebdirectorync/ -type d -exec chmod 750 {} \;
find ncwebdirectorync/ -type f -exec chmod 640 {} \;
chmod 740 ncwebdirectorync/apps/notify_push/bin/x86_64/notify_push
systemctl start apache2 php8.2-fpm notify_push coturn nginx ds-converter ds-metrics ds-docservice
sudo --user=www-data php ncwebdirectorync/occ upgrade
sudo --user=www-data php ncwebdirectorync/occ maintenance:mode --off
```

Note: Turning Nextcloud into the Maintenance-Mode and back, is not really required. As by the above commands, also the Apache Web-Server gets shutdown ... so being unable to tell the user anything. Let's see it more as praticing the right way, when making serious changes on the Nesxtcloud installation.

As the release from Github does not include all apps, at least for a Major release upgrade, I would recommend to look into the release on nextcloud.com .

#### Variant B

Similar to the way (above) to take over your additionally installed apps, use the following to take over the newly added apps.

```console
cd /tmp
wget https://download.nextcloud.com/server/releases/nextcloud-30.0.6.tar.bz2
tar -xf nextcloud-30.0.6.tar.bz2
cd nextcloud/apps
ls --directory * | while read d
do
  if [ ! -d "ncwebdirectorync/apps/$d" ]; then
    echo Found missing $d. Will take it over into the Nextcloud installation.
    cp --recursive "$d" "ncwebdirectorync/apps"
  fi
done
cd ../..
chown --recursive --no-dereference www-data:www-data nextcloud
find nextcloud/ -type d -exec chmod 750 {} \;
find nextcloud/ -type f -exec chmod 640 {} \;
chmod 740 nextcloud/apps/notify_push/bin/x86_64/notify_push
systemctl stop apache2 php8.2-fpm notify_push coturn nginx ds-converter ds-metrics ds-docservice
mv ncwebdirectorync ncwebdirectorync.BAK
mv nextcloud ncwebdirectorync
systemctl start apache2 php8.2-fpm notify_push coturn nginx ds-converter ds-metrics ds-docservice
sudo --user=www-data php ncwebdirectorync/occ upgrade
sudo --user=www-data php ncwebdirectorync/occ maintenance:mode --off
```

Output when Nextcloud had been upgrade from 29.0.10 to 30.0.4 .

```console
Found missing app_api. Will take it over into the Nextcloud installation.
Found missing twofactor_nextcloud_notification. Will take it over into the Nextcloud installation.
Found missing webhook_listeners. Will take it over into the Nextcloud installation.
```

# Package Management

## Restrict updates

I run into an issue with ONLYOFFICE and had to go back to a previous version. I wanted to prevent ONLYOFFICE to be update for the next period, at least until a new version becomes available. The keywords are "mark", "hold" and "unhold".

### Hold package

Example: ONLYOFFICE Documentserver

```console
# apt-mark hold onlyoffice-documentserver
onlyoffice-documentserver set on hold.
# 
```

### Unhold package

Example: ONLYOFFICE Documentserver

```console
# apt-mark unhold onlyoffice-documentserver
Canceled hold on onlyoffice-documentserver.
# 
```

## Tips

### Apache

Especially when the systems is going to be used by others, using `apachectl` can become handy.

This command checks the syntax, before you are going to restart/reload the web server. 

```console
# apache2ctl configtest 
Syntax OK
```

# Certificates

Beside creating additional client certificates, you usually do not need to create new certificates. Well, except for the following circumstances.

## OpenSSL for Web-Server & -Client-Authentication

### CA-Certificate

When looking at the CA-Certificate, the most import parts are the following.

- Valid timeframe 10 years
- X509v3 extensions
	- CA: Yes
	- Usage: signing certificates and revocation lists

```console
# openssl x509 -text -noout -in own_ca.crt
:
        Validity
            Not Before: Jan  1 06:19:54 2024 GMT
            Not After : Jan  1 06:19:54 2034 GMT
:
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:TRUE
:
            X509v3 Key Usage: critical
                Certificate Sign, CRL Sign
:
```

### Server-Certificate

When looking at the Server-Certificate, the most import parts are the following.

- Valid timeframe 1 year
- X509v3 extensions
	- CA: No
	- Usage: Signature, Encipher Key, Agree on Key
	- TLS Web: authenticate server
	- DNS: list of valid hostnames/FQDNs

Example for `aohostao.crt`

```console
# openssl x509 -text -noout -in aohostao/aohostao.crt
:
        Validity
            Not Before: Sep  8 07:37:31 2024 GMT
            Not After : Sep  8 07:37:31 2025 GMT
:
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier: 
                05:9A:EF:54:49:DE:16:D2:DB:9F:E5:E2:4F:42:93:E4:12:C2:C4:D4
            X509v3 Authority Key Identifier: 
                F2:18:0E:36:31:11:66:36:31:10:18:27:B8:2F:B9:26:98:31:B5:71
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Key Agreement
            X509v3 Extended Key Usage: critical
                TLS Web Server Authentication
            X509v3 Subject Alternative Name: 
                DNS:localhost, DNS:aohostao
:
```

Example for `aodomainao.crt`, highlighting the major different to the other one.

```console
# openssl x509 -text -noout -in aohostao/aohostao.crt
:
            X509v3 Subject Alternative Name: 
                DNS:aodomainao, DNS:*.aodomainao
:
```

### Client-Certificate

When looking at the Client-Certificate, the most import parts are the following.

- Valid timeframe 1 year
- X509v3 extensions
	- CA: No
	- Usage: Signature, Encipher Key, Agree on Key
	- TLS Web: authenticate client
	- DNS: list of servers hostnames/FQDNs

```console
# openssl x509 -text -noout -in "aohostao/client A.crt"
:
        Validity
            Not Before: Sep  8 08:04:57 2024 GMT
            Not After : Sep  8 08:04:57 2025 GMT
        Subject: C = DE, ST = Berlin, O = Beispiel GmbH, CN = User A
:
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier: 
                F7:D7:51:7E:9A:E1:46:84:33:96:6C:64:98:28:50:19:01:08:69:06
            X509v3 Authority Key Identifier: 
                EE:AE:E0:4B:51:CA:02:6A:5A:6F:57:AF:63:E7:B6:16:23:BA:FF:EE
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment, Key Agreement
            X509v3 Extended Key Usage: critical
                TLS Web Client Authentication
            X509v3 Name Constraints: critical
                Permitted:
                  DNS:aohostao
                  DNS:aodomainao
                  DNS:.aodomainao
            X509v3 Subject Alternative Name: 
                email:user@example.com
:
```

## Lost Device

1. revoke the Client Certificate
2. add the content of the create `.crl` file to the file `/etc/ssl/crl/own_ca.crl`
3. reload apache
4. create new Client Certificate
5. deploy new Client Certificate

## Every year

### Web-Server

First check, when the own_ca certificate expires. If it will happen within a year, renew the CA certificate first (see "Every 10 year").

For the apache web server, you have to do the following.

1. create new Server-Certificate
2. deploy new Server-Certificate
3. reload apache

### Clients

First check, when the own_ca certificate expires. If it will happen within a year, renew the CA certificate first (see "Every 10 year").

For every user, you have to do the following.

1. create new Client Certificate
2. deploy new Client Certificate

## Every 10 year

For your own_ca, you have to do the following.

1. create new CA-Certificate
2. deploy new CA-Certificate
	- on the Web-Server
	- on the 'aohostao' server
	- on each client device

# Add-On

## OpenVPN

### Identify Clients

The status log tells you, which client got connected and so on.

Remember the "server" attribute of the OpenVPN configuration? The line means, the clients will be in the IP range of 10.9.8.1 - 10.9.8.255

```code
server 10.9.8.0 255.255.255.0
```

In the following example, the client device with the name "ncclientnc", got the Tunnel IP 10.9.8.2 . Means from the `aohostao`, you can start ssh session , using this reported IP address.

```console
# cat /run/openvpn-server/status-server.log
HEADER,CLIENT_LIST,Common Name,Real Address,Virtual Address,Virtual IPv6 Address,Bytes Received,Bytes Sent,Connected Since,Connected Since (time_t),Username,Client ID,Peer ID,Data Channel Cipher
CLIENT_LIST,ncclientnc,<remote IP>:<remote port>,10.9.8.2,,11437,11186,2024-10-01 18:00:27,1727798427,UNDEF,0,0,AES-256-GCM
HEADER,ROUTING_TABLE,Virtual Address,Common Name,Real Address,Last Ref,Last Ref (time_t)
ROUTING_TABLE,10.9.8.2,ncclientnc,<remote IP>:<remote port>,2024-10-01 18:10:03,1727799003
GLOBAL_STATS,Max bcast/mcast queue length,1
GLOBAL_STATS,dco_enabled,0
END
# 
```

# This and That

## Last login

Nextcloud can tell you, when a user (<User-ID>) had been logged in the last time.

```console
# sudo -u www-data php ncwebdirectorync/occ user:lastseen <User-ID>
```

Or use the `--all` option to see the last login of every user.

```console
# sudo -u www-data php ncwebdirectorync/occ user:lastseen --all
```

## SSH: Create Host Keys

```console
rm /etc/ssh/*key*
dpkg-reconfigure openssh-server
```
