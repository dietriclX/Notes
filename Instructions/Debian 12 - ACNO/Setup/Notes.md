# Introduction

To get the most out of this documentation.

1. Read [Instruction > Readme.md](../../Readme.md)
2. Make use of [`apply.sh`](../Usage/scripts/apply.sh) ¹

¹ Read section [Using `scripts` Directory](../Usage/Readme.md#using-scripts-directory). But you can also use the alternative search&replace the parameters by their values, as soon as you come a long a - for you - new parameter..

I went through several iterations, of setting up, testing, evaluating. Checked out the different solutions/software/technologies, like Collabora, Let's Encrypt, nginx and so on.

My opinion so far ...

- Collabora: The License of the "Development Edition" does not fit to my planned usage. The "Try the development edition for free" is basically what license text says. No, thanks.
- Let's Encrypt: I do have to deploy the Client-Certificates, together with my own CA-Certificate, anyway ... so there is no need to have the Server-Certificate signed by an official CA.
- nginx: I didn't like the way of configuration. Sorry, I had already experiences with Apache from the past. And to be honest, nginx doesn't come with the same reputation as Apache. Also in this case, the licensing model ... not impressed, not interested at all.
- Apache `SSLVerifyClient require`: Restricting the access on the Web-Server level, works extremely well. At least from the security aspect. However you cannot use any of the Android/iOS apps. The compromise is to use the Client-Certificates for authentication and allow access of Nextcloud also for others.

At the end, I decided to go with the following "Basis" setup.

## Basis

We are using two network devices to create something similar to a DMZ. The connection to the internet is established by a DSL-Router. The only devices connected to the DSL-Router are a Wireless Router and the ACNO machine (production). The Wireless Router provides the internal network either wireless or via RJ45 cable. By using only the interfaces of the Wireless Router, we have created a kind of a DMZ between the DSL-Router and the Wireless Router.

```code
Internet <---> DSL-Router <-DMZ-> Wireless Router & Switch <-LAN-> Workstation
                             |                                |
                             |                                +--> machine `aohostao` (during setup)
                             |
                             +--> machine `aohostao` (DMZ)
```

## Used Parameters/Attributes/configuration

In order to make use of the [provided scripts](https://github.com/dietriclX/Automaton/tree/main/Debian%2012%20-%20ACNO), we have to maintain all parameter/values in the `.map`/`.smap` files of directory `data`. We take the delivered `parameter_value.Default.*` as template to create our own definitions files named e.g. `parameter_value.Own.*`.

Note: *The name format for the `.map`/`.smap` files is given by the script [`apply.sh`](https://github.com/dietriclX/Automaton/blob/main/Debian%2012%20-%20ACNO/apply.sh). The reference to our own instances of the mapping files is made by using the script parameter `-i TYPE` or `--instance=TYPE`. With placeholder `TYPE` referring to the `TYPE` in our own file names `parameter_value.TYPE.map` and `parameter_value.TYPE.smap`.*

 the `acno` directory and get the relevant files from [GitHub: dietriclX / Automaton](https://github.com/dietriclX/Automaton).

```console
cd $HOME
wget https://github.com/dietriclX/Automaton/archive/refs/heads/main.zip
unzip main.zip
find "Automaton-main/Debian 12 - ACNO" -type d -exec chmod 700 '{}' \+
find "Automaton-main/Debian 12 - ACNO" -type f -exec chmod 600 '{}' \+
find "Automaton-main/Debian 12 - ACNO" -type f -name '*.sh' -exec chmod 700 '{}' \+
mkdir --parents --mode=700 scripts
mv "Automaton-main/Debian 12 - ACNO" scripts/acno
```

Create the 

# Certificates

## Preparation

### Content Certificates parameter_value.*.map

🏎 Automation - Prepare `scripts` directory on machine, where the Certificate maintenance is done.

Before we are able to use the `apply.sh` script, we have to create the files `parameter_value.Own.map` and `parameter_value.Sensitive.map`.

- `parameter_value.Own.map` for maintaining general information on our own Certificate Authority (CA), own Intermediate Certificate Authority (ICA) and the requestor of the Certificates.
- `parameter_value.Sensitive.map` for maintaining the passphrases and password to access the Keys and Client Certificates.

Note: *We use meaningless names, to prevent other of creating a link between the web site and ourself in the real world.*

Now with all parameters given for the installation specific values, we can make use of `apply.sh`.

```console
cd $HOME/scripts/acno
./apply.sh --force --instance=Own data/variables.ORG.sh variables.sh
./ca_prepare_directory.sh
```

=== `~/scripts/acno/data/parameter_value.Own.map` ===

```code
#
# Certificate related information
#

# own Certificate Authority (CA)
#
# short name as used in file/directory names
canameca        example_ca
# Country as used in the certificate
cacountryca     US
# State/Province/County as used in the certificate
castateca       MA
# Locality as used in the certificate
cacityca        Wakefield
# Organisation as used in the certificate
caorgca         'not W3C'
# Organisational Unit as used in the certificate
caorgunitca     'Certificate Authority for Example'
# Common Name as used in the certificate
cacommonnameca  'CA for Example'
# Initial 'serial number' for signing.
# Will be set at the creation of the Certificate Authority.
# - to be specified as hexadecimal number
castartserialnrca       30
# Initial 'CRL number' for revoking.
# Will be set at the creation of the Certificate Authority.
# - to be specified as hexadecimal number
castartcrlnrca          25

# own Certificate Unit, which requests certificates and use it to run the server(s)
#
# Country as used in the certificate
cucountrycu     US
# State/Province/County as used in the certificate
custatecu       MA
# Locality as used in the certificate
cucitycu        Wakefield
# Organisation as used in the certificate
cuorgcu         'not W3C'
# Organisational Unit as used in the certificate
cuorgunitcu     'Service Provider for Example'

# own Intermediate Certificate Authority (ICA)
#
# Not required for setting up the ACNO system, but might become useful for other systems.
# For example a OPNsense/pfSense Firewall with its own OpenVPN server.
#
# short name as used in file/directory names
ciname1ci       example_ica
# Country as used in the certificate
cicountry1ci    US
# State/Province/County as used in the certificate
cistate1ci      MA
# Locality as used in the certificate
cicity1ci       Wakefield
# Organisation as used in the certificate
ciorg1ci        'not W3C'
# Organisational Unit as used in the certificate
ciorgunit1ci    'Intermediate Certificate Authority for Example'
# Common Name as used in the certificate
cicommonname1ci 'ICA for Example'
# Initial 'serial number' for signing.
# Will be set at the creation of the Intermediate Certificate Authority.
# - to be specified as hexadecimal number
cistartserialnr1ci       1030
# Initial 'CRL number' for revoking.
# Will be set at the creation of the Intermediate Certificate Authority.
# - to be specified as hexadecimal number
cistartcrlnr1ci          1025

```

=== `~/scripts/acno/data/parameter_value.Own.smap` ===

```code
######## Place, to store the most sensitive information e.g. passwords #########
### to avoid accidental copy of such information from one machine to another ###
######### to avoid account name and password in the same mapping file ##########



#
# Certificate related information
#

# own Certificate Authority (CA)
#

# temporary Passphrase for the encrypted key
#   used by script
catmpkeypassphraseca    temporary_ca_key_pass

# Passphrase for the encrypted key
cakeypassphraseca       ca_key_pass

# Users Client Certificates
#
# Password for exporting/importing .p12/.pfx Client Certificates
capasswordca            export_key_pass

# own Intermediate Certificate Authority (ICA)
#

# temporary Passphrase for the encrypted key of ICA#1
#   used by script
citmpkeypassphrase1ci   temporary_ica1_key_pass

# Passphrase for the encrypted key
cikeypassphrase1ci       ica_key_pass

# Users Client Certificates (issued by ICA#1)
#
# Password for exporting/importing .p12/.pfx Client Certificates
cipassword1ci           export_ica_key_pass

```

## Critical Files

In general all files created in this section should be handled with care. Making a backup of the to-be-created `SSL.canameca` directory is recommended.

You are about to create a CA, Under no circumstances should the CA files (`canameca.key` & `canameca.crt`) get lost, or even worse get stolen.

## Pass Phrase & Password

Make sure that you write down this information and do not loose it ... like with the CA files, the information is important/critical.

### Pass Phrase

During the first steps (becoming a CA), you will have to enter a Pass Phrase for the CA key (`canameca.key`). This is indicated by `Enter PEM pass phrase:`  and `Verifying - Enter PEM pass phrase:`. Than later and each time OpenSSL has to access the CA key, you get asked by `Enter pass phrase for CA/canameca.key:` to enter the exact same Pass Phrase as specified during the first steps (creating your CA).

To simplify the process a little bit, we are not dealing with Pass Phrases for the Client-/Server-Keys. So ... temporarly a file `*pass.key` gets created with Pass Phrase `secret` (`-passout pass:secret`) and afterwards the real `.key` gets created with the Pass Phrase removed.

### Ex-/Import password

The Client-Certificate for the initial user `ncadminnc` gets exported as PKCS#12-Certificate and has to have a password. You get prompted by `Enter Export Password:` and `Verifying - Enter Export Password:`. The so called Export-/Import-Password is later required when importing the Client-Certificate onto the client device. So again, write the password down.





# Server Installation

## OS installation

### Post activities

#### GRUB

🖴 Backup done by [backup_os_basis.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/backup_os_basis.sh) script. Restore covers an execution of `update-grub`.

#### OpenSSH for SSH-Server & -Client-Authentication

🖴 Backup done by [backup_certkey.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/backup_certkey.sh) and [backup_user_regular.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/backup_user_regular.sh) scripts.

#### SSD

🖴 Backup done by [backup_os_related.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/backup_os_related.sh) script.

#### Locales

🖴 Backup done by [backup_os_basis.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/backup_os_basis.sh) script. Restore covers an execution of `locale-gen`.

#### Tools

🖴 Get installed by [acno_prepare_machine.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/acno_prepare_machine.sh) script.

## Nextcloud Prerequisites

### OS

#### Repositories

🖴 Backup of the Debian Repositories gets done by [backup_os_basis.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/backup_os_basis.sh) script. Backup of the Online Editors gets done by [software_collabora.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/software_collabora.sh) (Collabora Online) and [software_onlyoffice.sh](../../../../Automaton/Debian%2012%20-%20ACNO/modules/software_onlyoffice.sh) (ONLYOFFICE)  scripts.

### Proxy-Server - Nginx

Unfortunately `nginx` is tightly bound with the ONLYOFFICE installation. I first tried to ignore this circumstance and had disabled the `nginx` Web-Server Proxy, but after an upgrade from version 8.1.3-4 to 8.2.0-143 ... I had been unable to open the documents (error `Cannot GET /8.2.0-88d72bc60b1e714a35888b9a72c76d20/web-apps/... `). So for the moment, we will leave `nginx` acting as Proxy but on port `oodsportoo`.

The port is defined in  file `/etc/onlyoffice/documentserver/nginx/ds.conf`, which is set during installation and can be adjusted before installation by executing the following command.

```console
echo onlyoffice-documentserver onlyoffice/ds-port select oodsportoo | sudo debconf-set-selections
```

### Configuration

### Trusted Domains

With the machine `aohostao` in the same network as the workstation `wshostws`, you have to enable the whole local network as Trusted Domain.

Example for a local noetwork with IP Address / Subnet `192.168.1.0 /23`.

```console
sudo --user=www-data php ncwebdirectorync/occ config:system:set trusted_domains 4 --value '192.168.1.*'
```

I did not get it to work within my local network. So I allowed every domain and IP range.

```console
sudo --user=www-data php ncwebdirectorync/occ config:system:set trusted_domains 4 --value '*'
```

# Add-On

## Backup (Local/WebDAV)

While keep a system running and also doing backup, we have to deal always with "limited" resources. Primarely the avaialable storage.

With regards to the running ACNO system, there is a given size of the storage capacity. To ensure that at any given time, the users data will never cause a run out of the available storage, there is the "Quota" system for users and "Team folders". With a freshly setup system it is easy to calculate the quotas. The SDD has X-Number of GB available, that's to be split up into a reserved disk space and the disk space to be used yb the user.

Before getting to the aspect of backups, let us take a look at what data is easily "eating" up the storage. In some cases it is not the size of a single file, but more the vast number of file of that category.

- Audio/Music
- Pictures
- Videos
- CD/DVD backups
- PDF documents with scanned pages
- Software installation media

Beside CD/DVD backup, these files have a low compression ratio. Meaning for example, one 1GB of music will become 1GB on the backup media.

For backup it looks a little bit different, as it all depends on the Backup-Strategy. Meaning ... What should be accomplished by doing backups. Let's look at my goals.

I might be force to restore the last backup, being not older than one to two days.
I might get a request from a user, to restore a file - deleted a few days back. As the user might not immediately identify the missing and communication takes time, I would have to access a backup older than two days.
Gurglars might steal my equipment (incl. backups), so I need a backup outside my home.

- I want to be able to restore the backup of the last day or the day before.
    - using two different USB HDD
    - running each night a backup in an alternating manner
- I want to keep backup for at least one week. 12 days even better
    - estimated required space ⇒ 2 x 25GB + 4 x 20GB + 1GB (reserved) = 151GB data every day
        - two Team Folder with a Quota of 25GB each
        - four users with a Quota of 20GB each
    - estimated available storage ⇒ 2 x 1024GB - 150GB (reserved) = 1898GB available disk space
        - each USB HDD has 1024GB of capacity
    - estimated number of backups ⇒ 1898GB / 151GB = 12.57
- I want a backup stored outside my home. It does not matter, how long it takes to restoring a system, but I have to be capable of doing so.
    - keeping the latest backup on a WebDAV drive
        - no ideal, but better than nothing
    - keeping daily backups on a Remote Backup Unit
        - the preferred method
        - basically the backup will be identical to the one at home
- I do not want to spend time on taking backup. Especially taking regular backups at night is not acceptable. Automation wanted, but with evidence on success and/or failures.
    - based on a Shell Script
    - backup nightly
        - backup started at 01:00 am
        - backup might last until 04:00 am
        - system available after 06:00 am
        - leaving two hours buffer, where also the DSL reset can take place
        - email based notifications
            - summary
            - details
            - encrypted messages using GnuPG (PGP)
- I do not want to spend time of deleting outdated backups, to free up some disk space. Automation wanted, but with evidence on success and/or failures.
    - based on a Shell Script
    - preferablke to be done by the same script which does the backup
    - delete the oldest backup
        - to ensure the new backup will have space enough
        - it might not be enough to delete one old backup
        - assume a growth of the backup size by a factor of 1.5 per day¹ (to be specified in `aobackupgrowthao`)

¹ That is especially useful at the beginning, when users get more an more familiar with the system and its features. So once rolled out to the "Users" more and more data will be stored on the ACNO system. "Oh, that allows me to backup my 6GB of pictures - from my smartphone - on the server" Excellent ...

## OpenVPN

### Server

#### Routing configuration


Note: I haven't understood this completely ... maybe if this output is missing ... check `/etc/default/netfilter-persistent` if that corresponds with the content of `/usr/share/netfilter-persistent/plugins.d` .... ? Once had I deleted - manually - the files within directory `/usr/share/netfilter-persistent/plugins.d` and aftwards `netfilter-persistent save` was not willing to re-create the files `15-ip4tables` and `25-ip6tables`. The only way I had been able to solve this was by purge&install `netfilter-persistent iptables-persistent`.

=== `/etc/default/netfilter-persistent` ===

```code
# Configuration for netfilter-persistent
# Plugins may extend this file or have their own

FLUSH_ON_STOP=0

# Set to yes to skip saving rules/sets when netfilter-persistent is called with
# the save parameter
# IPTABLES_SKIP_SAVE=yes
# IP6TABLES_SKIP_SAVE=yes
# IPSET_SKIP_SAVE=yes


# Set to yes for not flushing existing ip[6]tables rules when netfilter-persistent
# is called with the start parameter
# IPTABLES_RESTORE_NOFLUSH=yes
# IP6TABLES_RESTORE_NOFLUSH=yes


# Set to yes to test load the rules before applying them. This avoids loading failure
# from causing no rules to be loaded in the kernel
IPTABLES_TEST_RULESET=yes
IP6TABLES_TEST_RULESET=yes
```

## UnifiedPush

The Nextcloud "**UnifiedPush Provider**" app is the generic server part, when using "**NextPush**" app on an Android device. The server part is "generic", as the support for client apps does not work out-of-the-box. For example Molly-FOSS requires to have MollySocket, installed on the server, as well.

### UnifiedPush Provider

#### Molly-FOSS (Signal chat client)

On [GitHub: mollyim / mollysocket](https://github.com/mollyim/mollysocket) in section "**Overview**" is a great diagram, which illustrates the communication.

During the configuration of Molly-FOSS, for using UnifiedPush, a QR-Code is required. This is generated by "**MollySocket**" by opening URL `https://aodomainao/upmollypathup/`.

In case of server misconfiguration the page `https://aodomainao/upmollypathup/` with the QR-Code will present a message with details. Therefor it is a useful check before rolling out the service.

The QR-Code contains the following information.

```uri
mollysocket://link?vapid=<key>&url=https%3A%2F%2Faodomainao%2Fupmollypathup%2F&type=webserver
```

Note: The `<key>` is not identical to the one in file `vapid_privkey.txt`.

##### Server

The "**MollySocket**" server is the link between UnifiedPush Provider and the Signal Server.

The configuration is a little bit different, than outlined in [GitHub: mollyim / mollysocket / INSTALL.md](https://github.com/mollyim/mollysocket/blob/main/INSTALL.md). The main differences to the instruction of [GitHub: mollyim / mollysocket / INSTALL.md](https://github.com/mollyim/mollysocket/blob/main/INSTALL.md).

- leaving out `mollysocket-vapid.service` and instead do the prerequisits manually
- name the binary `mollysocket` instead of `ms`
- stored the VAPID in plain text (`/etc/mollysocket/vapid_privkey.txt`)
- database file name `database.sqlite` instead of `ms.db`
- add `https://aodomainao` to `allowed_endpoints`
