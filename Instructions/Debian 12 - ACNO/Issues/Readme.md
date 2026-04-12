Issues, I run into and had been able to resolve them (or at least identify a work-around).

# Nextcloud

## Administrative settings: Overview > Security & setup warnings

Log in as Administrator and navigate to menu option "Administrative settings". After a few seconds the findings under "Security & setup warnings" are shown.

**ⓧ** The PHP memory limit is below the recommended value of 512 MB.

ACTION: Increase the limit, as specific in `/etc/php/8.2/apache2/php.ini` and `/etc/php/8.2/fpm/php.ini`.

```code
memory_limit = 1G
```

And restart the related services.

```console
systemctl restart php8.2-fpm apache2
```

---

**ⓧ** This server has no working internet connection: Multiple endpoints could not be reached. This means that some of the features like mounting external storage, notifications about updates or installation of third-party apps will not work. Accessing files remotely and sending of notification emails might not work, either. Establish a connection from this server to the internet to enjoy all features.

ACTION: Check the known issues like [Issue - Internet connectivity / DNS-Server](#issue---internet-connectivity--dns-server)

---

**⚠** One or more mimetype migrations are available. Occasionally new mimetypes are added to better handle certain file types. Migrating the mimetypes take a long time on larger instances so this is not done automatically during upgrades. Use the command `occ maintenance:repair --include-expensive` to perform the migrations.

ACTION: As suggested, run the command.
```console
sudo --user=www-data php /var/www/ncdirectorync/occ maintenance:repair --include-expensive`
```

---

**⚠** Your web server is not properly set up to resolve `.well-known` URLs, failed on: `/.well-known/webfinger` For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-setup-well-known-URL)

ACTION: _none_

---

**⚠** HTTP headers

Some headers are not set correctly on your instance - The `Strict-Transport-Security` HTTP header is not set (should be at least `15552000` seconds). For enhanced security, it is recommended to enable HSTS. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-security).

ACTION: Ensure the following line in file `dddd` does not starat with a `#`.

`add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;` 

---

**⚠** Server has no maintenance window start time configured. This means resource intensive daily background jobs will also be executed during your main usage time. We recommend to set it to a time of low usage, so users are less impacted by the load caused from these heavy tasks. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-background-jobs).

ACTION: Specify the the maintenance windows starting 01:00 o'clock, by executing the following command.

`sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set maintenance_window_start --value="1"`

---

**⚠** The database is missing some indexes. Due to the fact that adding indexes on big tables could take some time they were not added automatically. By running "occ db:add-missing-indices" those missing indexes could be added manually while the instance keeps running. Once the indexes are added queries to those tables are usually much faster. ...

ACTION: Execute the following command.

`sudo --user=www-data php /var/www/ncdirectorync/occ db:add-missing-indices`

---

**⚠** PHP does not seem to be setup properly to query system environment variables. The test with getenv("PATH") only returns an empty response. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-php-fpm).

ACTION: Uncomment the followinf lines in the file `/etc/php/8.2/fpm/pool.d/www.conf`.

```code
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

---

**⚠** The PHP OPcache module is not properly configured. The OPcache interned strings buffer is nearly full. To assure that repeating strings can be effectively cached, it is recommended to apply "opcache.interned_strings_buffer" to your PHP configuration with a value higher than "8".. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-php-opcache).

ACTION: Edit file `/etc/php/8.2/apache2/php.ini` and `/etc/php/8.2/fpm/php.ini` and change the parameter to `32`.

```code
opcache.interned_strings_buffer=32
```

And restart the related services.

```console
systemctl restart php8.2-fpm apache2
```

---

**ⓘ** Background blur

Could not check for WASM loading support. Please check manually if your web server serves `.wasm` files. To allow this check to run you have to make sure that your Web server can connect to itself. Therefore it must be able to resolve and connect to at least one of its `trusted_domains` or the `overwrite.cli.url`. This failure may be the result of a server-side DNS mismatch or outbound firewall rule.

ACTION: Verify that the existing `.wasm` can be accessed through the Web-Server.

```console
# ls /var/www/ncdirectorync/apps/spreed/js/*.wasm
/var/www/ncdirectorync/apps/spreed/js/olm.wasm			 /var/www/ncdirectorync/apps/spreed/js/vision_wasm_nosimd_internal.wasm
/var/www/ncdirectorync/apps/spreed/js/vision_wasm_internal.wasm
# curl --head --silent http://aohostao/apps/spreed/js/olm.wasm | grep --regexp="^HTTP" --regexp="^Content-Type"
HTTP/1.1 200 OK
Content-Type: application/wasm
```

To verify use the following code to create a file, used by the health check. In the browser open once more "Security & setup warnings". Issue should be gone.

```console
cd /var/www/ncdirectorync/apps/spreed/js/
# Create a minimal valid WASM module (8 bytes: magic number + version)
printf '\x00\x61\x73\x6d\x01\x00\x00\x00' > tflite.wasm
chown www-data:www-data tflite.wasm
chmod 640 tflite.wasm
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:repair
```

Remove `tflite.wasm` using command: `rm /var/www/ncdirectorync/apps/spreed/js/tflite.wasm`

Wait for the fix to be released.

Source: [Peter Bakker: Nextcloud 32 Upgrade Issue: Fixing the Talk App WASM Warning](https://pieterbakker.com/nextcloud-talk-wasm-loading-support-fix/)

---

**ⓘ** Code integrity

Integrity checker has been disabled. Integrity cannot be verified.

ACTION: _none_

Based on the instruction, you had downloaded Nextcloud directly from GitHub. This kind of installation is not covered by the integrity check.

---

**ⓘ** High-performance backend

No High-performance backend configured - Running Nextcloud Talk without the High-performance backend only scales for very small calls (max. 2-3 participants). Please set up the High-performance backend to ensure calls with multiple participants work seamlessly.

ACTION: 

---

**ⓘ** The database is used for transactional file locking. To enhance performance, please configure memcache, if available. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-transactional-locking).

ACTION: _none_

---

**ⓘ** No memory cache has been configured. To enhance performance, please configure a memcache, if available. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-performance).

ACTION: Change the configuration of Nextcloud, with

1. local memory caching enabled (via APCu).
2. Redis accessed via UNIX socket
3. file locking enabled (via Redis)
4. distributed memory caching enabled (via Redis)

All changes can be done from command line.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.local --value "\OC\Memcache\APCu"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set filelocking.enabled --value "true"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.distributed --value "\\OC\\Memcache\\Redis"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.locking --value "\\OC\\Memcache\\Redis"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis host --value=/run/redis/redis-server.sock
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis port --value "0"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis dbindex --value "3"
```

---

**ⓘ** Your installation has no default phone region set. This is required to validate phone numbers in the profile settings without a country code. To allow numbers without a country code, please add "default_phone_region" with the respective of the region to your config file. For more details see the [documentation ↗](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements).

ACTION: Execute the folowing command.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set default_phone_region --value="aophoneregionao"
```

---

**ⓘ** You have not set or verified your email server configuration, yet. Please head over to the "Basic settings" in order to set them. Afterwards, use the "Send email" button below the form to verify your settings. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-email).

ACTION: Follow the instruction and setup the connect to the email server.

---

**ⓘ** The PHP module "imagick" in this instance has no SVG support. For better compatibility it is recommended to install it. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-php-modules).

ACTION: Install the missing software, using the following command.

```console
apt install imagemagick
```

# LibreSign

## Issue - Invalid hash or binary files. jsignpdf

Example in regards to `jsignpdf-2.2.2`.

### Symptom

As Administrator

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Administration settings**" from the menu
				- ⬆ select "**LibreSign**" in the left panel
					- focus on section "**Configuration check **"

| Status | Message | Resource | Tip |
| :--- | :--- | :--- | :--- |
| error | Invalid hash or binary files. | jsignpdf | Check your nextcloud.log file and run occ libresign:install --all |

=== `/var/log/nextcloud/nextcloud.log` ===
```code
:
{"reqId":"<ID>","level":3,"time":"<Date/Time>","remoteAddr":"<IP-Address>","user":"<User-ID>","app":"libresign","method":"GET","url":"/ocs/v2.php/apps/libresign/api/v1/admin/configure-check","scriptName":"/ocs/v2.php","message":"Invalid hash of binaries files","userAgent":"<Browser User-Agent>","version":"<Nextcloud Version>","data":{"app":"libresign","result":"{\"EXTRA_FILE\":{\"jsignpdf-2.2.2/docs/JSignPdf.pdf\":{\"expected\":\"\",\"current\"\"<Hash>\"},\"jsignpdf-2.2.2/JSignPdf.jar\":{\"expected\":\"\",\"current\":\"<Hash2>\"},\"jsignpdf-2.2.2/InstallCert.jar\":{\"expected\":\"\",\"current\"\"<Hash3>\"}}}"},"id":"<ID2>"}
:
```

### Notes

Most likely this folder is a remaining of an older version of LibreSign.

### Solution

1. identify folder of JSignPdf.jar (e.g. `jsignpdf-2.2.2/docs/JSignPdf.pdf`)
2. remove parent folder (e.g. `ncdatadirectorync/appdata_ocmu53th6kbo/libresign/x86_64/jsignpdf/jsignpdf-2.2.2`)

```console
find ncdatadirectorync -name JSignPdf.jar
rm -Rf ncdatadirectorync/appdata_ocmu53th6kbo/libresign/x86_64/jsignpdf/jsignpdf-2.2.2
```

# ONLYOFFICE

## Issue - Error when trying to connect: Administrator > "Save" settings

### Symptom

As Administrator:

In menu "Adiministration settings > ONLYOFFICE", unable to "Save" changes. Getting error message

```code
Error when trying to connect (Error occurred in the document service: Error while downloading the document file to be converted.) (version 8.2.2.22)
```

### Notes

Happens, when "**ONLYOFFICE Docs address**" is referring to "https://ooserveroo.aohostao". The key point is "aohostao".
The server name cannot be resolved in the local network. The host IP-Address of "aohostao", but not for "ooserveroo.aohostao".

### Solution

n/a

## Issue - nginx 403: User > open document / Administrator > "Save" settings

### Symptom

As User:

In "Files" app, ONLYOFFICE in the browser is unable to open a document and says: download failed, press ok to to return to document list.

As Administrator:

In menu "Adiministration settings > ONLYOFFICE", unable to "Save" changes. Getting error message

```code
Error when trying to connect (Client error: `GET http://localhost:8080/cache/files/data/conv_check_29021126_65/output.docx/check_29021126.docx?md5=...&expires=...&shardkey=...&filename=check_....docx` resulted in a `403 Forbidden` response:
403 Forbidden
403 Forbidden
nginx
```

### Notes

This happened to me after an upgrade from 8.1.3-4 to 8.2.1-38 . A lot happened between the initial setup and the ONLYOFFICE upgrade, therefore I do not really know what was causing this issue.

### Solution

First, identify the `<Storage Secret key>` from the `local.json` file (see below).

In the `ds.conf` file update the value for `$secure_link_secret` with the `<Storage Secret key>`.
Restart the nginx service, using command `systemctl restart nginx`.

=== /etc/onlyoffice/documentserver/nginx/ds.conf ===

```code
include /etc/nginx/includes/http-common.conf;
server {
  listen 0.0.0.0:80;
  listen [::]:80 default_server;
  server_tokens off;

  set $secure_link_secret <Storage Secret key>;
  include /etc/nginx/includes/ds-*.conf;
}
```


=== `/etc/onlyoffice/documentserver/local.json` ===

```code
:
  "storage": {
    "fs": {
      "secretString": "<Storage Secret key>"
    }
  }
}
```

## Issue - installed onlyoffice-documentserver package post-installation script subprocess returned error exit status 1

### Symptom

As Administrator:

During the update of the package `onlyoffice-documentserver` an error occurs, at the post-installation script. A look into the journal (`journalctl`) reveals that - again - nginx wants to listen on port `80`.

```code
:
Stopping documentserver services...
Unpacking onlyoffice-documentserver (8.2.2-22) over (8.2.1-38) ...
Setting up onlyoffice-documentserver (8.2.2-22) ...
Installing new version of config file /etc/onlyoffice/documentserver/nginx/includes/ds-docservice.conf ...
Generating AllFonts.js, please wait...Done
Generating presentation themes, please wait...Done
Generating js caches, please wait...Done
Installing plugins, please wait...Done
dpkg: error processing package onlyoffice-documentserver (--configure):
 installed onlyoffice-documentserver package post-installation script subprocess returned error exit status 1
Processing triggers for libc-bin (2.36-9+deb12u9) ...
Errors were encountered while processing:
 onlyoffice-documentserver
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

```code
:
... systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
... nginx[114244]: 2024/11/28 18:21:32 [info] 114244#114244: Using 116KiB of shared memory for nchan in /etc/nginx/nginx.conf:61
... nginx[114244]: 2024/11/28 18:21:32 [info] 114244#114244: Using 131072KiB of shared memory for nchan in /etc/nginx/nginx.conf:61
... nginx[114249]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
... nginx[114249]: nginx: [emerg] still could not bind()
... systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
... systemd[1]: nginx.service: Failed with result 'exit-code'.
... systemd[1]: Failed to start nginx.service - A high performance web server and a reverse proxy server.
:
```

### Notes

This happened when the `onlyoffice-documentserver` package comes with an update of the nginx configuration. 

### Solution / Work-Around

Run the configuration of the package again, this time with Apache being shutdown. Once done, modify the `ds.conf` file and (re-)start both web servers.

```console
systemctl stop apache2
dpkg --configure onlyoffice-documentserver
sed -i -e "s/listen 0\.0\.0\.0\:80;/listen 0.0.0.0:8080;/" -e "s/listen \[::\]:80 /listen \[::\]:8080 /" /etc/onlyoffice/documentserver/nginx/ds.conf
systemctl restart nginx
systemctl start apache2
```

## Issue - /var/log/turn/server_*.log : `CONFIG ERROR: Empty cli-password, ...`


### Symptom

coturn log file contains the following line

```code
coturn log file
	CONFIG ERROR: Empty cli-password, and so telnet cli interface is disabled! Please set a non empty cli-password!
```

### Solution / Work-Around

Sdd the "--no-cli" parameter.

=== `/lib/systemd/system/coturn.service` ===

```code
:
ExecStart=/usr/bin/turnserver -c /etc/turnserver.conf  --no-cli ...
:
```

## Issue - /var/log/turn/server_*.log : `Bad configuration format: no-loopback-peers`

### Symptom

coturn log file contains the following line

```code
	0: : Bad configuration format: no-loopback-peers
	0: : Listener address to use: 0.0.0.0
	0: : Bad configuration format: no-loopback-peers
	0: : 0 bytes per second allowed, combined server capacity
	0: : Bad configuration format: no-loopback-peers
```

### Notes

Since version 4.5.2 the options `no-loopback-peers` and `no-multicast-peers` got replaced by `allow-loopback-peers`.

### Solution / Work-Around

Remove the following lines from the coturn configuration `/etc/turnserver.conf`.

```code
	no-loopback-peers
	no-multicast-peers
  ```

## Issue - /var/log/turn/server_*.log : `WARNING: cannot find <certificate related> file`

### Symptom

coturn log file contains the following line

```code
	0: : WARNING: cannot find CA file: /usr/local/etc/ca.crt (1)
	0: : WARNING: cannot start TLS and DTLS listeners because CA file is not set properly
	0: : WARNING: cannot find certificate file: /usr/local/etc/server.crt (1)
	0: : WARNING: cannot start TLS and DTLS listeners because certificate file is not set properly
	0: : WARNING: cannot find private key file: /usr/local/etc/server.key (1)
	0: : WARNING: cannot start TLS and DTLS listeners because private key file is not set properly
  ```

### Notes

The coturn server runs as "turnserver" user/group. At least the private key is accessible only by root. In addition, it seems like symbolic links are not "allowed" by ´the turnserver.

### Solution / Work-Around

```console
rm /usr/local/etc/ca.crt /usr/local/etc/server.crt /usr/local/etc/server.key
cp /usr/local/share/ca-certificates/my-custom-ca/canameca.crt /usr/local/etc/ca.crt
chown turnserver:turnserver /usr/local/etc/ca.crt
cp /etc/ssl/certs/aodomainao.crt /usr/local/etc/server.crt
chown turnserver:turnserver /usr/local/etc/server.crt
cp /etc/ssl/private/aodomainao.key /usr/local/etc/server.key
chown turnserver:turnserver /usr/local/etc/server.key
```

## Issue - nextcloud.log : `Infected file deleted. Heuristics.Phishing.Email.SpoofedDomain`

### Symptom

Unable to synchronize Thunderbird export, using the Nextcloud Client. Most likely caused by an email with a virus of the kind "Heuristics.Phishing.Email.SpoofedDomain".

"**Administration settings** > **Log reader**"

| Level | Application | Message | Time |
| :--- | :--- | :--- | :--- |
| Error | webdav | **UnsupportedMediaType** Virus Heuristics.Phishing.Email.SpoofedDomain detected in the file. Upload cannot be completed. | *timestamp* |
| Error | no app in context | **InvalidContentException** Virus Heuristics.Phishing.Email.SpoofedDomain detected in the file. Upload cannot be completed. | *timestamp* |
| Error | files_antivirus | Infected file deleted. Heuristics.Phishing.Email.SpoofedDomain File: files/...part Account: ...  | *timestamp* |
| Warning | files_antivirus | Infected file deleted. Heuristics.Phishing.Email.SpoofedDomain File: files/...part Account: ...  | *timestamp* |

=== `/var/log/nextcloud/nextcloud.log` ===

```log
:
{"reqId":"7Y2IClNVIkNtNlnHcIwK","level":2,"time":"<timestamp>","remoteAddr":"<ip address>","user":"<user>","app":"files_antivirus","method":"MOVE","url":"/remote.php/dav/uploads/<user>/<number>/.file","message":"Infected file deleted. Heuristics.Phishing.Email.SpoofedDomain Account: <user> Path: files/<folder>/<uuid>.ocTransferId255247632.part","userAgent":"Mozilla/5.0 (Linux) mirall/3.7.3git (Nextcloud, debian-6.1.0-37-amd64 ClientArchitecture: x86_64 OsArchitecture: x86_64)","version":"<version>","clientReqId":"<uuid>","data":{"app":"files_antivirus"}}
:
```

### Solution / Work-Around

The perfect solution would be to identify and delete, in Thunderbird, the email with this virus. Than export and upload the archive again.

A quick and dirty would be to disable scanning for such kind of virus.

```console
sed --in-place --expression="s/^PhishingScanURLs true/PhishingScanURLs false/" /etc/clamav/clamd.conf
systemctl restart clamav-daemon
```

## Issue - Broadcast message : `UPS updevicenameup@uphostnameup is unavailable `

### Symptom

In a terminal session the following message appears.

> 
> Broadcast message from root@aohostao (somewhere) (timestamp):
> 
> UPS updevicenameup@uphostnameup is unavailable
> 

### Solution / Work-Around

- verify that the nut service on uphostnameup is up and running
- verify that the name or IP-Address of uphostnameup is still correct

## Issue - E: Package 'coolwsd' has no installation candidate

### Symptom

The attempt of installing Collabora Online/CODE results in an error.

```console
# apt install --yes coolwsd code-brand fonts-crosextra-carlito fonts-liberation fonts-opensymbol fonts-takao-gothic supervisor
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package coolwsd is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'coolwsd' has no installation candidate
```

### Solution / Work-Around

Check the output of command `uname -a`. If it is ending with `i686 GNU/Linux`, the system was setup usign the incorrect bit format. In such a case it is required to setup the system from scratch using the "**adm64**" image.

## Issue - E: Unable to locate package onlyoffice-documentserver

### Symptom

The attempt of installing Collabora Online/CODE results in an error.

```console
# apt install --yes nginx-extras onlyoffice-documentserver rabbitmq-server ttf-mscorefonts-installer
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package onlyoffice-documentserver
```

### Solution / Work-Around

## Issue - apt update / Missing key D8915E456E7C440E, which is needed to verify signature.

### Symptom

The execution of `apt update` reports the missing public key for the in ONLYOFFICE Docs package(s).

```console
root@aohostao:~# apt update
:
Hit:4 https://download.onlyoffice.com/repo/debian squeeze InRelease                                  
Get:5 https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb ./ InRelease [1,728 B]
Err:5 https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb ./ InRelease
  Sub-process /usr/bin/sqv returned an error code (1), error message is: Missing key D8915E456E7C440E, which is needed to verify signature.
Warning: OpenPGP signature verification failed: https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb ./ InRelease: Sub-process /usr/bin/sqv returned an error code (1), error message is: Missing key D8915E456E7C440E, which is needed to verify signature.
Error: The repository 'https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb ./ InRelease' is not signed.
Notice: Updating from such a repository can't be done securely, and is therefore disabled by default.
Notice: See apt-secure(8) manpage for repository creation and user configuration details.
root@aohostao:~# 
```

### Solution / Work-Around

Download the missing public key from the Ubuntu [OpenPGP Keyserver](https://keyserver.ubuntu.com/) and import the public key into the existing keyring `onlyoffice.gpg`.

1. open URL `https://keyserver.ubuntu.com/`
2. enter `D8915E456E7C440E` in field "*Search for an OpenPGP Public Key, ie 0x.*`
3. click on "**🔍 Search Key**" will open new page with the title "**Search results for '0xD8915E456E7C440E'**"
4. click on link in line `**pub**` will download the public key
5. rename the downloaded file to `D8915E456E7C440E.asc`
6. transfer the `D8915E456E7C440E.asc` file onto `aohostao`
7. import the public key into `onlyoffice.gpg`

```console
root@aohostao:~# cat D8915E456E7C440E.asc | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/onlyoffice.gpg --import
gpg: key D8915E456E7C440E: public key "Collabora Productivity (Release Package Signing Key) <hello@collaboraoffice.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

# UnifiedPush

## Issue - Failed at step CREDENTIALS spawning mollysocket: Protocol error

### Symptom

The MollySocket process does not start. Only `journalctl` reveals the reason.

=== Output of `systemctl status mollysocket` ===

```code
× mollysocket.service - MollySocket
     Loaded: loaded (/etc/systemd/system/mollysocket.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since ...
   Duration: 30ms
 Invocation: 654ea5589c8a4b86b730f1699ba29889
    Process: 366612 ExecStart=mollysocket server (code=exited, status=243/CREDENTIALS)
   Main PID: 366612 (code=exited, status=243/CREDENTIALS)

... systemd[1]: mollysocket.service: Scheduled restart job, restart counter is at 5.
... systemd[1]: mollysocket.service: Start request repeated too quickly.
... systemd[1]: mollysocket.service: Failed with result 'exit-code'.
... systemd[1]: Failed to start mollysocket.service - MollySocket.
```

=== Output of `journalctl` ===

```code
... systemd[1]: Started mollysocket.service - MollySocket.
... (sd-mkdcre[366597]: Embedded credential name 'ms_vapid' does not match filename 'vapid.key', refusing.
... (lysocket)[366595]: mollysocket.service: Failed to set up credentials: Protocol error
... (lysocket)[366595]: mollysocket.service: Failed at step CREDENTIALS spawning mollysocket: Protocol error
... systemd[1]: mollysocket.service: Main process exited, code=exited, status=243/CREDENTIALS
... systemd[1]: mollysocket.service: Failed with result 'exit-code'.
```

### Notes

An existing example with the used paramenter `--name=ms_vapid` mislead me to used the same in my instruction.

=== `https://github.com/mollyim/mollysocket` ===

> - How to backup VAPID key?
> 
> MollySocket is designed for self-hoster, and the idea is to renew the VAPID key if you have to reinstall MollySocket somewhere else. If you are asking for this, you are probably trying to use systemd-creds, else you'd have the VAPID private key in plain text.
> 
> If you haven't generated the VAPID key yet, just pipe the command to a temporary file: `mollysocket vapid gen | tee key.tmp | systemd-creds encrypt --name=ms_vapid -p - -`, key.tmp will contain the key, you can store it in a safe and remove the file.

### Solution / Work-Around

Instruction got updated.

# Server in General

## Issue - Internet connectivity / DNS-Server

### Symptom

On the server do a names lookup for a known Web-Server FQDN. The request is send to DNS Server at `<ip-address>`.

```console
root@aohostao:~# nslookup www.google.com
;; communications error to <ip-address>#53: timed out
;; communications error to <ip-address>#53: timed out
;; communications error to <ip-address>#53: timed out
```

Executed the same command on the Workstation.

```console
root@wshostws:~# nslookup www.google.com
:
Name:	www.google.com
Address: 172.217.18.4
Name:	www.google.com
Address: 2a00:1450:4001:82a::2004
```

On the server download a page from the known Web-Server ... using the IP-Address identified on the Workstation.

```console
root@aohostao:~# curl 172.217.18.4
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```

```console
root@aohostao:~# cat /etc/resolv.conf
# Generated by dhcpcd from eno1.dhcp, eno1.dhcp6, eno1.ra
# /etc/resolv.conf.head can replace this line
domain <domain-local/regular-network>
nameserver <ip-address>
nameserver <ip6-address>
# /etc/resolv.conf.tail can replace this line
```

### Notes

Issue related to DNS only.

Server was setup in the local/regular network, not the DMZ. After moving the server into the DMZ, connections to the internet are no longer possible.

When the server was setup, the IP-Address was given by the DHCP server of the local/regular network.

### Solution / Work-Around

Adjust the `nameserver` entries, with the IP-Address(es) of the DMZ, in `/etc/resolv.conf`.

## Issue - Redis: WARNING Memory overcommit must be enabled!

### Symptom

A line in the Redis log file reports about this possible issue and its work-around.

=== `/var/log/redis/redis-server.log` ===

```code
... # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
```

### Solution / Work-Around

```console
echo vm.overcommit_memory = 1 >> /etc/sysctl.conf
sysctl --load
```

Alternative is to add the rule to the new file `99-vm_overcommit_memory.conf`.

```console
echo vm.overcommit_memory = 1 > /etc/sysctl.d/99-vm_overcommit_memory.conf
sysctl --load
```

### Notes

Once the work-around got implemented, the correctness can be verified in the following way. With the output of the last command being **`0`**.

```console
systemctl stop redis-server
rm --force /var/log/redis/redis-server.log
systemctl start redis-server
sleep 3s
grep "WARNING Memory overcommit must be enabled!" /var/log/redis/redis-server.log | wc --lines
```
