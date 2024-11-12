Issues, I run into and had been able to resolve them (or at least identify a work-around).

# Nextcloud

## Administrative settings: Overview > Security & setup warnings

Log in as Administrator and navigate to menu option "Administrative settings". After a few seconds the findings under "Security & setup warnings" are shown.

ERROR: The PHP memory limit is below the recommended value of 512 MB.
ACTION: Increase the limit, as specific in `/etc/php/8.2/apache2/php.ini` and `/etc/php/8.2/fpm/php.ini`.
```code
memory_limit = 1G
```
And restart the related services.
```console
systemctl restart php8.2-fpm apache2
```

WARNING: One or more mimetype migrations are available. Occasionally new mimetypes are added to better handle certain file types. Migrating the mimetypes take a long time on larger instances so this is not done automatically during upgrades. Use the command `occ maintenance:repair --include-expensive` to perform the migrations.
ACTION: As suggested, run the command.
```console
sudo --user=www-data php /var/www/ncdirectorync/occ maintenance:repair --include-expensive`
```

WARNING: Your web server is not properly set up to resolve `.well-known` URLs, failed on: `/.well-known/webfinger` For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-setup-well-known-URL)
ACTION: _none_

WARNING: Some headers are not set correctly on your instance - The `Strict-Transport-Security` HTTP header is not set (should be at least `15552000` seconds). For enhanced security, it is recommended to enable HSTS. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-security).
ACTION: Ensure the following line in file `dddd` does not starat with a `#`.
`add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;` 

WARNING: Server has no maintenance window start time configured. This means resource intensive daily background jobs will also be executed during your main usage time. We recommend to set it to a time of low usage, so users are less impacted by the load caused from these heavy tasks. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-background-jobs).
ACTION: Specify the the maintenance windows starting 01:00 o'clock, by executing the following command.
`sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set maintenance_window_start --value="1"`

WARNING: The database is missing some indexes. Due to the fact that adding indexes on big tables could take some time they were not added automatically. By running "occ db:add-missing-indices" those missing indexes could be added manually while the instance keeps running. Once the indexes are added queries to those tables are usually much faster. ...
ACTION: Execute the following command.
`sudo --user=www-data php /var/www/ncdirectorync/occ db:add-missing-indices`

WARNING: PHP does not seem to be setup properly to query system environment variables. The test with getenv("PATH") only returns an empty response. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-php-fpm).
ACTION: Uncomment the followinf lines in the file `/etc/php/8.2/fpm/pool.d/www.conf`.
```code
env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

WARNING: The PHP OPcache module is not properly configured. The OPcache interned strings buffer is nearly full. To assure that repeating strings can be effectively cached, it is recommended to apply "opcache.interned_strings_buffer" to your PHP configuration with a value higher than "8".. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-php-opcache).
ACTION: Edit file `/etc/php/8.2/apache2/php.ini` and `/etc/php/8.2/fpm/php.ini` and change the parameter to `32`.
```code
opcache.interned_strings_buffer=32
```
And restart the related services.
```console
systemctl restart php8.2-fpm apache2
```

INFO: Integrity checker has been disabled. Integrity cannot be verified.
ACTION: _none_
Based on the instruction, you had downloaded Nextcloud directly from GitHub. This kind of installation is not covered by the integrity check.

INFO: The database is used for transactional file locking. To enhance performance, please configure memcache, if available. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-transactional-locking).
ACTION: _none_

INFO: No memory cache has been configured. To enhance performance, please configure a memcache, if available. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-performance).
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

INFO: Your installation has no default phone region set. This is required to validate phone numbers in the profile settings without a country code. To allow numbers without a country code, please add "default_phone_region" with the respective of the region to your config file. For more details see the [documentation ↗](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements).
ACTION: Execute the folowing command.
```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set default_phone_region --value="aophoneregionao"
```

INFO: You have not set or verified your email server configuration, yet. Please head over to the "Basic settings" in order to set them. Afterwards, use the "Send email" button below the form to verify your settings. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-email).
ACTION: Follow the instruction and setup the connect to the email server.

INFO: The PHP module "imagick" in this instance has no SVG support. For better compatibility it is recommended to install it. For more details see the [documentation ↗](https://docs.nextcloud.com/server/29/go.php?to=admin-php-modules).
ACTION: Install the missing software, using the following command.
```console
apt install imagemagick
```

# ONLYOFFICE

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
