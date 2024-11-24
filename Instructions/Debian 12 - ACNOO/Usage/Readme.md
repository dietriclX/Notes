
# Basics

After the installation some activities are handy to have documented.

## Maintenance

### Maintenance-Mode ON

```console
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:mode --on
```

### Maintenance-Mode OFF

```console
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:mode --off
```

## Start

Manually starting all instances of a Nextcloud installation.

```console
systemctl start postgresql redis
systemctl start apache2 coturn ds-converter ds-docservice ds-metrics php8.2-fpm
```

## Stop

Manually stopping all instances of a Nextcloud installation.

```console
systemctl stop apache2 coturn ds-converter ds-docservice ds-metrics php8.2-fpm
systemctl stop postgresql redis
```

## Backup

### Prepare `backup` Directory

Make sure you have always a disk attached to the server `nchostnc`. In this example it is `/dev/sda`, whose first partition will be mounted on `/backup`.

```console
# mkdir /backup
# chmod 755 /backup
# mount /dev/sdb1 /backup
# ls /backup
lost+found
```

### Prepare `fstab`

Identify the UUID of your backup device, by attaching the device and executing the command.

```console
# lsblk -o PATH,TYPE,MOUNTPOINT,UUID
PATH      TYPE MOUNTPOINT UUID
/dev/sda  disk            
/dev/sda1 part /          3b54bcb4-b772-4fee-a146-297f9fbb4981
/dev/sdb  disk            
/dev/sdb1 part            bb09a4f1-fa6a-435b-b1f5-0c1db8bb8c2f
```

*Note: As I executed `lsblk` before attaching the USB disk, I knew the `/dev/sd?` letter.*

Next is to add a line in `fstab` for this particular device, for mount point `/backup` with the attribute `noauto` to prevent a mount at boot time.

<details>
<summary>File `/root/backup_system.sh`</summary>

```code
:

# Backup devices
UUID=bb09a4f1-fa6a-435b-b1f5-0c1db8bb8c2f /backup         ext4    noauto,errors=remount-ro 0       2

:
```

</details>

### Setup cron job

Before you can schedule the cron job, you first have  to create the script `backup_system.sh`.

```console
mkdir /root/scripts
touch /root/scripts/backup_system.sh
chmod 700 /root/scripts/backup_system.sh
```

Put the content below into the script file and test the script.

Once the script is stable and can be executed every night at 01:15, insert the following line into crontab for root (command `sudo crontab -e`).

```code
15 1 * * * /root/scripts/backup_system.sh
```

*Note: The internet offers a variety of tools to help with cron job scheduling. For example [Cronitor](https://crontab.guru)*

At the end of the script, the log file will be send to the administrators email address. The two lines above (starting with `#`) show the usage of pgp encryption.

=== `/root/backup_system.sh` ===

```code
#!/bin/bash

START=$(date +'%Y%m%d-%H%M')
STARTDATE=$(date +'%Y%m%d')
STARTTIME=$(date +'%H%M')
LOGTMP=/tmp/backup_system_$START.out

# Mount Backup device
echo "$(date +'%H:%M:%S') Device: mounting ..." >> $LOGTMP
mount /backup
retVal=$?
if [ $retVal -ne 0 ]; then
    echo "$(date +'%H:%M:%S') Device: not available" >> $LOGTMP
    echo "$(date +'%H:%M:%S') Device: lsblk" >> $LOGTMP
    lsblk >> $LOGTMP
    exit 1
fi
echo "$(date +'%H:%M:%S') Device: mounted" >> $LOGTMP

# Mount was successful ... do a quick check if the device was intended for the job
if [ -f "/backup/ACNOO-Backup" ]; then
  echo "$(date +'%H:%M:%S') Device: contains the marker file ACNOO-Backup" >> $LOGTMP
else
  # keep the temporary log file in /tmp
  echo "$(date +'%H:%M:%S') Device: MISSING marker file ACNOO-Backup" >> $LOGTMP
fi

# Check for existing today directory. If found, backup the same.
if [ -d "/backup/today" ]; then
  echo "$(date +'%H:%M:%S') Subdirectory /backup/today already exists" >> $LOGTMP
  mv /backup/today /backup/today_before_$START 2>&1 >> $LOGTMP
  echo "$(date +'%H:%M:%S') renamed to today_before_$START" >> $LOGTMP
else
  echo "$(date +'%H:%M:%S') Subdirectory /backup/today does not exist" >> $LOGTMP
fi

# Create today directory with read/write for everyone
mkdir /backup/today 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') Created today directory" >> $LOGTMP
echo "$(date +'%H:%M:%S') Let everyone write into today directory" >> $LOGTMP
chmod 777 /backup/today 2>&1 >> $LOGTMP

# Disable cron jobs for www-data
echo "$(date +'%H:%M:%S') User www-data: backup and disable cron jobs" >> $LOGTMP
crontab -l -u www-data > /backup/today/www-data.cron.backup  2>> $LOGTMP
crontab -r -u www-data 2>&1 >> $LOGTMP

# Put Nextcloud into maintenance mode
echo "$(date +'%H:%M:%S') Nextcloud: turn Maintenance-Mode ON" >> $LOGTMP
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:mode --on 2>&1 >> $LOGTMP

cd /backup/today

# Backup Databases
echo "$(date +'%H:%M:%S') ONLYOFFICE: backup database ..." >> $LOGTMP
sudo --user=postgres pg_dump --username=postgres oodatabaseoo --file=oodatabaseoo.pgsql 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') ONLYOFFICE: backup done" >> $LOGTMP
echo "$(date +'%H:%M:%S') coturn: backup database ..." >> $LOGTMP
sudo --user=postgres pg_dump --username=postgres ctdatabasect --file=ctdatabasect.pgsql 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') coturn: backup done" >> $LOGTMP
echo "$(date +'%H:%M:%S') Nextcloud: backup database ..." >> $LOGTMP
sudo --user=postgres pg_dump --username=postgres ncdatabasenc --file=ncdatabasenc.pgsql 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') Nextcloud: backup done" >> $LOGTMP

# Log
echo "$(date +'%H:%M:%S') /var/log: backup files ..." >> $LOGTMP
tar -zcf var_log.tar.gz /var/log 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') /var/log: backup done" >> $LOGTMP

# OS related
echo "$(date +'%H:%M:%S') OS: backup files ..." >> $LOGTMP
tar -zcf osandco.tar.gz /etc/apt/sources.list /etc/apt/sources.list.d/onlyoffice.list /etc/default/grub /etc/fstab /etc/ssh /etc/ssl/certs/own_ca.crt /etc/ssl/crl/own_ca.crl /etc/ssl/certs/tiny.crt /etc/ssl/certs/aodomainao.crt /etc/ssl/private/tiny.key /etc/ssl/private/aodomainao.key /etc/tmpfiles.d/log-subfolder.conf /home/aosysuserao/.ssh /usr/share/keyrings/onlyoffice.gpg 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') OS: backup done" >> $LOGTMP

# Apache
echo "$(date +'%H:%M:%S') Apache: backup files ..." >> $LOGTMP
tar -zcf apache2.tar.gz /etc/apache2/conf-enabled/security.conf /etc/apache2/sites-available/any.conf /etc/apache2/sites-available/*-include.conf /var/www/html/incorrect_client_certificate.php /var/www/html/phpinfo.php 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') Apache: backup done" >> $LOGTMP

# coturn
echo "$(date +'%H:%M:%S') coturn: backup files ..." >> $LOGTMP
tar -zcf coturn.tar.gz /etc/turnserver.conf /lib/systemd/system/coturn.service 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') coturn: backup done" >> $LOGTMP

# Let's Encrypt
echo "$(date +'%H:%M:%S') Let's Encrypt: backup files ..." >> $LOGTMP
tar -zcf letsencrypt.tar.gz /etc/letsencrypt 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') Let's Encrypt: backup done" >> $LOGTMP

# Nextcloud
echo "$(date +'%H:%M:%S') Nextcloud: backup files ..." >> $LOGTMP
tar -zcf nextcloud.tar.gz /etc/systemd/system/notify_push.service /var/www/ncdirectorync /var/lib/nextcloud 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') Nextcloud: backup done" >> $LOGTMP

# nginx
echo "$(date +'%H:%M:%S') nginx: backup files ..." >> $LOGTMP
tar -zcf nginx.tar.gz /etc/nginx/nginx.conf 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') nginx: backup done" >> $LOGTMP

# ONLYOFFICE
echo "$(date +'%H:%M:%S') ONLYOFFICE: backup files ..." >> $LOGTMP
tar -zcf onlyoffice.tar.gz /etc/onlyoffice/documentserver /etc/onlyoffice/documentserver-example /var/lib/onlyoffice /var/www/onlyoffice 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') ONLYOFFICE: backup done" >> $LOGTMP

# OpenVPN
echo "$(date +'%H:%M:%S') OpenVPN: backup files ..." >> $LOGTMP
tar -zcf openvpn.tar.gz /etc/openvpn/server /etc/ssl/certs/dhparams.aodomainao.pem /etc/ssl/certs/aodomainao.crt /etc/ssl/private/aodomainao.key /etc/ssl/private/ta.aodomainao.key 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') OpenVPN: backup done" >> $LOGTMP

# PHP
echo "$(date +'%H:%M:%S') PHP: backup files ..." >> $LOGTMP
tar -zcf php.tar.gz /etc/php/8.2/apache2/php.ini /etc/php/8.2/fpm/php.ini /etc/php/8.2/cli/php.ini 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') PHP: backup done" >> $LOGTMP

# PostgreSQL
echo "$(date +'%H:%M:%S') PostgreSQL: backup files ..." >> $LOGTMP
tar -zcf postgresql.tar.gz /etc/postgresql/15/main/postgresql.conf 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') PostgreSQL: backup done" >> $LOGTMP

# Redis
echo "$(date +'%H:%M:%S') Redis: backup files ..." >> $LOGTMP
tar -zcf redis.tar.gz /etc/redis/redis.conf 2>&1 >> $LOGTMP
echo "$(date +'%H:%M:%S') Redis: backup done" >> $LOGTMP

# OS changes - begin of script
cat << EOF > /backup/today/os_changes.sh

# Add own ca to the list of OS CAs
mkdir /usr/local/share/ca-certificates/my-custom-ca
chmod 700 /usr/local/share/ca-certificates/my-custom-ca
ln -s /etc/ssl/certs/own_ca.crt /usr/local/share/ca-certificates/my-custom-ca
update-ca-certificates

# Make certificates/keys of the Web-Server accessible by coturn
cd /usr/local/etc
ln -s /etc/ssl/certs/aodomainao.crt server.crt
ln -s /etc/ssl/private/aodomainao.key server.key
ln -s /etc/ssl/certs/own_ca.crt ca.crt

# Web-Server (Apache) remove the default site and replace if by "any.conf"
cd /etc/apache2/sites-enabled/
rm 000-default.conf
ln -s ../sites-available/any.conf any.conf

# Nextcloud should get links to the two files
cd /var/www/ncdirectorync
ln -s ../html/incorrect_client_certificate.php incorrect_client_certificate.php
ln -s ../html/phpinfo.php phpinfo.php

EOF
cd /backup/today
# OS changes - end of script

# Remove write access on today directory and log file
echo "$(date +'%H:%M:%S') remove write access on today directory and log file" >> $LOGTMP
chmod 755 /backup/today 2>&1 >> $LOGTMP

# End the maintenance mode for Nextcloud
echo "$(date +'%H:%M:%S') Nextcloud: turn Maintenance-Mode OFF" >> $LOGTMP
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:mode --off 2>&1 >> $LOGTMP

# Restore cron jobs for www-data
echo "$(date +'%H:%M:%S') User www-data: restore cron jobs" >> $LOGTMP
crontab -u www-data /backup/today/www-data.cron.backup 2>&1 >> $LOGTMP

cd /root

# Change today directory name to YYMMDD
echo "$(date +'%H:%M:%S') directory today becomes $STARTDATE" >> $LOGTMP
mv /backup/today /backup/$STARTDATE 2>&1 >> $LOGTMP
cp $LOGTMP /backup/$STARTDATE/$STARTTIME-$(date +'%H%M').log

# Un-mount backup device
echo "$(date +'%H:%M:%S') Device: unmount ..." >> $LOGTMP
umount /backup
echo "$(date +'%H:%M:%S') Device: unmount done" >> $LOGTMP

# Send log file
#gpg --recipient aoadmin@addressao --sign --armor --output $LOGTMP.asc --encrypt $LOGTMP
#mail aoadmin@addressao < $LOGTMP.asc
mail aoadmin@addressao < $LOGTMP
```

## Backup TEST

Take a spare machine or Virtual Machine (VM), pretend you run into the "Second worst scenario" and have to restore the whole system.

## Restore

Even if the restore is labor intensive ... do not forget to setup on the new machine the backup!

### Available Backup

Take a look at the latest backup...

```console
# mount /backup
# cd /backup
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

### Restore OS Configuration

Assuming that the system got created based on the documentation. Focus on SSL related parts.

```console
cd /backup
mkdir restore
tar xfz ../<backup date>/osandco.tar.gz
cp etc/ssl/certs/* /etc/ssl/certs/
cp etc/ssl/private/* /etc/ssl/private/
cp etc/ssl/crl/* /etc/ssl/crl
mkdir /usr/local/share/ca-certificates/my-custom-ca
chmod 700 /usr/local/share/ca-certificates/my-custom-ca
ln -s /etc/ssl/certs/own_ca.crt /usr/local/share/ca-certificates/my-custom-ca
update-ca-certificates
```

### Restore Database(s)

Assuming that the DBO of the database still exists, or had been recreated.

#### coturn

Drop database before loading.

```console
# sudo --user=postgres psql --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# DROP DATABASE ctdatabasect;
DROP DATABASE
postgres=# CREATE DATABASE ctdatabasect OWNER ctdboct;
CREATE DATABASE
postgres=# GRANT ALL privileges ON DATABASE ctdatabasect TO ctdboct;
postgres=# \q
# sudo --user=postgres psql --host=localhost --dbname=ctdatabasect --username=ctdboct --password -f <backup date>/ctdatabasect.pgsql
could not change directory to "/root": Permission denied
Password: 
:
```

#### Nextcloud

Drop database before loading.

```console
# sudo --user=postgres psql --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# DROP DATABASE ncdatabasenc;
DROP DATABASE
postgres=# CREATE DATABASE ncdatabasenc TEMPLATE template0 ENCODING 'UNICODE' OWNER ncdbonc;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE ncdatabasenc TO ncdbonc;
GRANT
postgres=# \q
# sudo --user=postgres psql --host=localhost --dbname=ncdatabasenc --username=ncdbonc --password -f <backup date>/ncdatabasenc.pgsql
could not change directory to "/root": Permission denied
Password: 
:
```

#### ONLYOFFICE

Drop database before loading.

```console
# sudo --user=postgres psql --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# DROP DATABASE oodatabaseoo;
DROP DATABASE
postgres=# CREATE DATABASE oodatabaseoo OWNER oodbooo;
CREATE DATABASE
postgres=# GRANT ALL privileges ON DATABASE oodatabaseoo TO oodbooo;
postgres=# \q
# sudo --user=postgres psql --host=localhost --dbname=oodatabaseoo --username=oodbooo --password -f <backup date>/oodatabaseoo.pgsql
could not change directory to "/root": Permission denied
Password: 
:
```

### Restore Installation(s)

#### coturn

With a fresh installation

```console
cp /etc/turnserver.conf /etc/turnserver.conf.ORG
cp /lib/systemd/system/coturn.service /lib/systemd/system/coturn.service.ORG
cd /backup
mkdir restore
cd restore
tar xfz ../<backup date>/coturn.tar.gz
cp etc/turnserver.conf /etc/turnserver.conf
cp lib/systemd/system/coturn.service /lib/systemd/system/
systemctl daemon-reload
```

#### Nextcloud

Get same install version of Nextcloud, as of Backup.

```console
cd /var/www
git clone --depth 1 --shallow-submodules --recurse-submodules --branch v29.0.8 https://github.com/nextcloud/server.git
touch server/config/config.php
mv server ncdirectorync
chown --recursive --no-dereference www-data:www-data ncdirectorync
```

Restore `config.php` and additional APPs.

```console
cd /backup
mkdir restore
tar xfz ../<backup date>/nextcloud.tar.gz
mv var/lib/nextcloud /var/lib
cp var/www/ncdirectory/config/config.php /var/www/ncdirectory/config
cd var/www/ncdirectory/apps
ls --directory * | while read d
do
  if [ ! -d "/var/www/ncdirectorync/apps/$d" ]; then
    cp --recursive "$d" "/var/www/ncdirectorync/apps"
  fi
done
```

Restore Notify-Push service.

```console
cp etc/systemd/system/notify_push.service /etc/systemd/system
systemctl daemon-reload
systemctl enable notify_push
```

#### ONLYOFFICE

Assuming that the system got created based on the documentation. Focus on configuration related part.

```console
cd /backup
mkdir restore
tar xfz ../<backup date>/onlyoffice.tar.gz
cp etc/onlyoffice/documentserver/local.json /etc/onlyoffice/documentserver/
```

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

```console

```

### Nextcloud

```console
cd /var/www
sudo --user=www-data php /var/www/ncdirectorync/occ maintenance:mode --on
git clone  --single-branch --recurse-submodules --shallow-submodules --branch=v29.0.8 --depth 1 https://github.com/nextcloud/server.git
systemctl stop apache2 coturn ds-converter ds-docservice ds-metrics php8.2-fpm
mv ncdirectorync ncdirectorync.BAK
mv server ncdirectorync
cd ncdirectorync.BAK
cp config/config.php /var/www/ncdirectorync/config
cd apps
ls --directory * | while read d
do
  if [ ! -d "/var/www/ncdirectorync/apps/$d" ]; then
    cp --recursive "$d" "/var/www/ncdirectorync/apps"
  fi
done
cd /var/www/
chown -R www-data:www-data ncdirectorync
find ncdirectorync/ -type d -exec chmod 750 {} \;
find ncdirectorync/ -type f -exec chmod 640 {} \;
chmod 740 ncdirectorync/apps/notify_push/bin/x86_64/notify_push
systemctl start apache2 coturn ds-converter ds-docservice ds-metrics php8.2-fpm
sudo --user=www-data php /var/www/ncdirectorync/occ upgrade
sudo --user=www-data php /var/www/ncdirectorync/occ maintenance:mode --off
```

# Apackage Management

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

Example for `nchostnc.crt`

```console
# openssl x509 -text -noout -in nchostnc/nchostnc.crt
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
                DNS:localhost, DNS:nchostnc
:
```

Example for `aodomainao.crt`, highlighting the major different to the other one.

```console
# openssl x509 -text -noout -in nchostnc/nchostnc.crt
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
# openssl x509 -text -noout -in "nchostnc/client A.crt"
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
                  DNS:nchostnc
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
	- on the 'nchostnc' server
	- on each client device

# Add-On

## OpenVPN

### Identify Clients

The status log tells you, which client got connected and so on.

Remember the "server" attribute of the OpenVPN configuration? The line means, the clients will be in the IP range of 10.9.8.1 - 10.9.8.255

```code
server 10.9.8.0 255.255.255.0
```

In the following example, the client device with the name "ncclientnc", got the Tunnel IP 10.9.8.2 . Means from the `nchostnc`, you can start ssh session , using this reported IP address.

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
