Apache coturn Nextcloud Office / Acronym "ACNO" or just "ao".

From [GitHub: dietriclX/Notes](https://github.com/dietriclX/Notes) based on [License](https://github.com/dietriclX/Notes/blob/main/LICENSE).

# Introduction

You do have some basic knowledge as a Linux user? With "basic knowledge" referring to the usage of a character based terminal session.

You would like to run your own Nextcloud server?

Based on this guide we are going to set up a machine with Nextcloud at home.

All written with the best intention, but limited to the writers knowledge/skills.

The goal of the instruction is to setup everything in a secure manner, so the services can be made available to the World-Wild-Web (WWW). This leads to the oddity of spending at first, time with keys and certificates which are the basis of the later secure communication with Nextcloud & Co. .

☝ [Notes: Introduction](./Notes.md#introduction)

## Basis

- Connectivity
	- DDNS Service Provider: *your choice*
	- DSL
		- Download: 50 MBit/s max.
		- Upload: 20 MBit/s max.
	- DSL-Router
		- Support DDNS: ✔️
		- Support Port Forwarding: ✔️
    - LAN Port #1: Wireless Router
    - LAN Port #2: Machine name `aohostao` (production)
	- Wireless Router
		- OS: OpenWrt
    - LAN Port #1: Workstation `wshostws`
    - LAN Port #2: Machine `aohostao` (during setup)
  - Hypertext Transfer Protocol Secure (https)
    - own Certificate Authority (CA) or Let's Encrypt
  - Simple Mail Transfer Protocol (smtp)
    - GNU Privacy Guard (gpg)
- The Machine #1 (becomes the server for ACNO)
	- Host name: `aohostao` (generic `aohostgenericao`)
  - Hardware
	  - CPU: Intel Core i5
	  - RAM: 8GB DDR3
	  - SSD: 128GB SATA
  - Software
  	- OS: Debian 12 "bookworm"
    - DBMS: PostgreSQL 15 (via Debian)
    - In-Memory Storage System: Redis/7.0.15 (via Debian)
    - Other Servers:
      - STAN/TURN Server: coturn/4.6.1 (via Debian)
      - Web-Server: Apache/2.4.62 (via Debian)
      - Web-Server: nginx/1.22.1 (via Debian)
    - Office Application:
      - Nextcloud/29.0.10 (via GitHub - Nextcloud)
      - ONLYOFFICE/8.2.2-22 (via ONLYOFFICE Repository)
- The Machine #2 (backup system for ACNO)
  - Host name: `aohost2ao` (generic `aohostgenericao`)
  - Hardware
    - identical as Machine #1
    - plus HDD: same size or bigger than SSD (used for testing restore only)
- Backup Hardware
  - Local for Machine #1
    - 2x USB HDD with 1TB minimum
  - Remote Backup Unit (rbu)
    - SBC or Small PC
      - HDD 1TB minimum
- The Workstation (used for setup and administration of the ACNO machine)
  - Host name: `wshostws`
  - Software
  	- OS: Debian 12 "bookworm"
  - OS Users
    - Regular User
      - Account: `wssysuserws`

## Used Parameters/Attributes/configuration

As values for the different Parameters, the usage of characters is limited to "@", "-", "_", " ", "A"-"Z" and "a"-"z". This will avoid any kind of weird issues.¹

There are scripts available to handle the Parameters and their Values. See section [Using `scripts` Directory](../Usage/Readme.md#using-scripts-directory)

¹ Only the following Parameters are allowed to have a " " (space) as part of their values: "`castateca`", "`cacityca`", "`caorgca`", "`caorgunitca`", "`cacommonnameca`", "`custatecu`", "`cucitycu`", "`cuorgcu`", "`cuorgunitcu`"
  In addition ... `ncadminsubjectprio1nc`, `ncadminsubjectprio2nc`, `ncadminsubjectprio3nc`, `ncadminsubjectinfonc`

☝ [Notes: Introduction > Used Parameters/Attributes/configuration](Notes.md#used-parametersattributesconfiguration)

### General

Independent of the instance.

- Installation Name: `aonameao` (a name that describes the best the whole installation e.g. "ACNO")
- Installations Owner Email-Address: `aoowner@addressao` ()
- own Certificate Authority (CA)
  - Country: `cacountryca` (as used in the certificate)
  - State/Province/County: `castateca` (as used in the certificate)
  - Locality: `cacityca` (as used in the certificate)
  - Organisation: `caorgca` (as used in the certificate)
  - Organisational Unit: `caorgunitca` (as used in the certificate)
  - Common Name: `cacommonnameca` (as used in the certificate)
  - short name: `canameca` (as used in file names)
- Certificate requesting Unit
  - Country: `cucountrycu` (as used in the certificate)
  - State/Province/County: `custatecu` (as used in the certificate)
  - Locality: `cucitycu` (as used in the certificate)
  - Organisation: `cuorgcu` (as used in the certificate)
  - Organisational Unit: `cuorgunitcu` (as used in the certificate)
  - Common Name: `cucommonnamecu` (as used in the certificate)
  - short name: `cunamecu` (as used in file names)
- OS Users
  - Administrator:
    - Account: `root`
  	- Email: `aosystem@addressao` (e.g. "user@example.com")
  - Regular User
    - Account: `aosysuserao`
- System
  - LAN interface: `aohostlanao` (e.g. "eth0"; connection to the intranet)
  - Host IP address: `aoipv4addressao` & `aoipv6addressao`
  - DMZ Domain name: `aodmzdomainoa`
  - Phone Region: `aophoneregionao` (e.g. "DE")
  - Time Zone: `aotimezoneao` (e.g. "Europe/Berlin")
- Router
	-  Local IP address: `aorouteripv4addressao` (e.g. 10.0.0.1)
- Web-Server
	- Domain Name: `aodomainao` (e.g. "example.com")
	- SSL via own CA
- PostgreSQL
	- host name: `pshostps` (e.g. "localhost", which allows the use of unix sockets)
	- network port: `psportps` (as alternative to unix sockets)
  - Unix socket: `pssocketps` (the default "/var/run/postgresql")
  - administrator user: `psadmps` (the default "postgres")
  - administrator user password: `psadmpasswordps`
  - locals: `aodblocalesao` (e.g. "de_DE.UTF-8")

### Production Instance

The PROD Instance of the Office Solution.

- Nextcloud
	- host name: `nchostnc` (e.g. "localhost")
	- web directory: `/var/www/ncdirectorync` (e.g. "/var/www/something")
	- data directory: `ncdatadirectorync` (e.g. "/var/lib/nextcloud/something")
  - database host name: `ncdbhostnc` (usually ≙ `pshostps`)
  - database server port: `ncdbportnc` (usually ≙ `psportps`)
	- database name: `ncdatabasenc`
	- database owner: `ncdbonc`
	- database owners password: `ncdbopasswordnc`
	- Redis DB Index: `ncredisdbindexnc`
	- administrator: `ncadminnc`
	- administrator password: `ncadminpasswordnc`
  - administrator email address: `ncadmin@addressnc`
	- URL: `https://aodomainao` (e.g. "https://example.com")
- ONLYOFFICE
	- host name: `oohostoo`
	- local directory: `oowebdirectoryoo` (e.g. "/var/www/onlyoffice")
	- lib directory: `oolibdirectoryoo` (e.g. "/var/lib/onlyoffice")
  - secret: `oosecretkeyoo` (specified in `local.json`)
  - database host name: `oodbhostoo` (usually ≙ `pshostps`)
  - database server port: `oodbportoo` (usually ≙ `psportps`)
	- database name: `oodatabaseoo`
	- database owner: `oodbooo`
	- database owners password: `oodbopasswordoo`
	- public server name: `ooserveroo`
	- URL: `https://ooserveroo.aodomainao` (e.g. "https://ooserveroo.example.com")
- coTURN
	- host name: `cthostct`
  - database host name: `ctdbhostct` (usually ≙ `pshostps`)
  - database server port: `ctdbportct` (usually ≙ `psportps`)
	- database name: `ctdatabasect`
	- database owner: `ctdboct`
	- database owners password: `ctdbopasswordct`
  - secret: `ctsecretct` (as for `static-auth-secret` in coturn configuration)
	- public server name: `ctserverct`
	- URL: `https://ctserverct.aodomainao` (e.g. "https://ctserverct.example.com")

### Backup Instance

The BACKUP Instance of ACNO system.

*... this aspect is not covered yet ...*

# DDNS Configuration

First thing, do register your DSL-Router under a URL of your choice. There are several service provider for this kind of DNS.

1. Register yourself at a service provider for DDNS.
2. At the DDNS Service Provider,
	1. reserve the `aodomainao` domain.¹
	2. add two CNAME entries to your domain.
		- for Collabora Online/CODE `coserverco`²
		- for ONLYOFFICE Docs `ooserveroo`²
		- for coturn (STUN/TURN) `ctserverct`³
3. Configure your DSL-Router, to daily announce its public Internet IP address to the DDNS Service Provider.

¹ Usually you can only register a sub-domain of given domains (DDNS provider specific).

² Reserve it already for the Online Office Editors.

³ Reserve it already for the STUN/TURN server used by Nextcloud Talk.

## DSL-Router

### Configuration: DDNS

The location of the configuration depends on the Router model. For mine, I found the configuration at "Internet > Dynamic DNS".

- ⬆ navigate to the DDNS settings
- ✏️✔️ tick the checkbox for "**Use dynamic DNS**"
- ✏️✔️ for "**Provider**" select "**Other provider**"
- ✏️✔️ enter the following information
	- Domain name: `aodomainao`
	- User name: `<account login for DDNS-Provider>`
	- Password: `<account password for DDNS-Provider>`
	- Updateserver-Address: `<check documentation of DDNS-Provider>`
	- Protocol: `HTTPS`
	- Port: `443`
- ✔️ click on "**Save**"
- 🔙 reboot the router

Note: *Rebooting the router will ensure that A.) the change got stored permanently and B.) the `aodomainao` DNS entry gets updated.*

# Certificates

**Prerequisite:** We do have the `aodomainao` available for our setup and the domain gets automatically updated with the internet address (provided by ISP e.g. Telekom). See the two sections above [DDNS Configuration](#ddns-configuration) and [DSL-Router](#dsl-router). At the end, if we would execute `nslookup aodomainao` we would get the (external) IP address of the DSL-Router.

An essential part of this server setup is the usage of certificates. Therefore the creation of the same, should be done right at the beginning. 

The instructions are meant to be done/executed on your workstation `wshostws` or even better a dedicated machine for dealing with such sensitive data.

Action to be done.

1. become Certificate Authority (CA)
2. create Web-Server Certificates
3. create Client-Certificate for `ncadminnc`
4. revoke and re-Create the Client-Certificate
5. create archive, for the machine `aonameao`

In order to learn the process of revoking a Client-Certificate, in case of a stolen device, do all the steps up to the end of this section.

Note: If you haven't worked with OpenSSL before, please read at least [Notes > Certificates](./Notes.md#certificates).

## Preparation

Create the file `parameter_value.Own.map` and set the correct values for the listed parameters.

☝ [Notes: Certificates > QQQ](Notes.md#content-certificates-parameter_valueownmap)

```console
cd $HOME/scripts/acno
```


A small directory structure needs to be created, helping to distinguish between the CA, the server and the client(s).

| directory  | description |
| :--- | :--- |
| `ssl` | contain everything of your CA |
| `ssl/ca.canameca` | contain its own certificate/key and all configuration files required for the `openssl` operations|
| `ssl/server` | all files related to the services running on machine `aohostao` |
| `ssl/client` | all files to be deployed onto the clients/users PC or Smartphone |

Prepare the directories and create the files required for using OpenSSL properly.

```console
cd $HOME
mkdir --mode=700 --parents scripts/ssl
cd scripts/ssl
mkdir --mode=700 --parents ca.canameca/certs \
                           ca.canameca/crl \
                           ca.canamecaA/newcerts \
                           aohostao \
                           aodomainao
touch ca.canameca/index.txt \
      ca.canameca/crl.pem
echo 70 > ca.canameca/serial
echo 30 > ca.canameca/crlnumber
```

At the end you will have the following files created for your operations.

| file  | description |
| :--- | :--- |
| `ca/data/crl/canameca.pem` | empty Certificate Revokation List |
| `ca/data/crlnumber` | counter for Certificate revokation (starts with 30) |
| `ca/data/index.txt` | logfile on create/revoke of Certificates |
| `ca/data/serial` | counter for Certificate creation (starts with 70) |

The configuration files have to be created manually (content see below).

| file  | used for creating |
| :--- | :--- |
| `ca/data/itself.cnf` | for creating CA |
| `ca/data/env.cnf` | CA environment |
| `ca/data/aonameao.cnf` | anything for aonameao |
| `ca/data/aonameao4client.xcnf` | Client-Certificate X.509 extensions |
| `ca/data/aonameao4intranet.xcnf` | local Server-Certificate X.509 extensions |
| `ca/data/aonameao4internet.xcnf` | public Server-Certificate X.509 extensions |

=== `CA/env.cnf` ===

```code
dir           = CA                    # Where everything is kept
certificate   = $dir/canameca.crt     # CA Certificate
private_key   = $dir/canameca.key     # CA Key
certs         = $dir/certs            # Where the issued certs are kept
crl_dir       = $dir/crl              # Where the issued crl are kept
database      = $dir/index.txt        # database index file.
new_certs_dir = $dir/newcerts         # default place for new certs.
serial        = $dir/serial.txt       # The current serial number
crlnumber     = $dir/crlnumber.txt    # the current crl number
crl           = $dir/canameca.pem     # The current CRL
```

=== `CA/ca.cnf` ===

```code
[ req ]
prompt = no

default_bits       = 4096
distinguished_name = req_distinguished_name
attributes         = req_attributes
string_mask        = utf8only
policy             = policy_anything

[ req_distinguished_name ]
countryName                    = cacountryca
stateOrProvinceName            = castateca
localityName                   = cacityca
0.organizationName             = caorgca
organizationalUnitName         = caorgunitca
commonName                     = cacommonnameca  

[ req_attributes ]

[ policy_anything ]
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ v3_ca ] ### to sign itself ###
basicConstraints       = critical,CA:true
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
keyUsage               = critical, keyCertSign, cRLSign

[ v3_ica ] ### to sign Intermediate Certificate Authority ###
basicConstraints       = critical,CA:true,pathlen:0
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
keyUsage               = critical, keyCertSign, cRLSign
```

=== `CA/aonameao.cnf` ===

```code
[ ca ]
default_ca = CA_default        # The default ca section

[ CA_default ]
.include CA/env.cnf
prompt = no

default_days     = 365                # how long to certify for
default_crl_days = 3                  # how long before next CRL

default_md       = sha256

policy           = policy_match       # match of country state organization
email_in_dn      = no                 # Email was required for the request, but no longer needed

name_opt         = ca_default         # Subject Name options
cert_opt         = ca_default         # Certificate field options
preserve         = no                 # keep passed DN ordering
x509_extensions  = usr_cert           # The extensions to add to the cert

[ policy_match ]
countryName            = match
stateOrProvinceName    = match
organizationName       = match
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional
```

===`CA/aonameao4client.xcnf` ===

```code
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, clientAuth
nameConstraints        = critical, permitted;DNS:aodomainao,permitted;DNS:.aodomainao,permitted;DNS:aohostao,permitted;DNS:.aohostao
```

=== `CA/aonameao4intranet.xcnf` ===

```code
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, serverAuth
subjectAltName         = DNS:aohostao,DNS:*.aohostao
```

=== `CA/aonameao4internet.xcnf`¹ ===

```code
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, serverAuth
subjectAltName         = DNS:aodomainao,DNS:*.aodomainao
```

¹ 

```code
subjectAltName         = DNS:aodomainao,DNS:*.aodomainao,DNS:aodomain2ao,DNS:*.aodomain2ao
```

## CA-Certificate

To become a technical CA, the first steps are to create the key and certificate of your own CA. The certificate will be valid for the next 10 years (3650 days).

As a final step, create a symlink in folder `aonameao` to the newly create CA-Certificate.

```console
# cd ~/scripts/ssl
# openssl genrsa -utf8 -aes256 -out ca.canameca/canameca.key 4096
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
# openssl req -config CA/itself.cnf -days 3650 -extensions v3_ca -new -x509 -key CA/canameca.key -out CA/canameca.crt
Enter pass phrase for canameca.key:
```

Note: With the newly created files `canameca.key` & `canameca.crt` and (of course) the Pass Phrase you can create Certificates on any other machine.

## Server-Certificate

We are going to create two kinds of certificates. One for the Intranet and the other one for the Internet. That's required to not expose your machine names to the world, as host/domain name become part of the Server-Certificate.

- Intranet: The URL will refer to the server/domain `localhost`, `aohostao` or `aohostao.aodmzdomainoa`.
- Internet: The URL will refer to the server/domain `aodomainao`.

```console
cd ~/SSL.canameca
openssl genrsa -utf8 -aes256 -passout pass:secret -out aonameao/server-pass.key 4096
openssl rsa -passin pass:secret -in aonameao/server-pass.key -out aonameao/etc/ssl/private/aohostao.key
rm aonameao/server-pass.key
openssl req -subj "/C=cucountrycu/ST=custatecu/L=cucitycu/O=cuorgcu/OU=cuorgunitcu/CN=aohostao" -new -key aonameao/etc/ssl/private/aohostao.key -out aonameao/aohostao.csr
openssl ca -batch -config CA/aonameao.cnf -extfile CA/aonameao4intranet.xcnf -days 365 -out aonameao/etc/ssl/certs/aohostao.crt -infiles aonameao/aohostao.csr
rm aonameao/aohostao.csr
```

Server-Certificate for access the Web-Server from the Internet. Addressing the server by the names `aodomainao` or `<something>.aodomainao`.

```console
cd ~/SSL.canameca
openssl genrsa -utf8 -aes256 -passout pass:secret -out aonameao/server-pass.key 4096
openssl rsa -passin pass:secret -in aonameao/server-pass.key -out aonameao/etc/ssl/private/aodomainao.key
rm aonameao/server-pass.key
openssl req -subj "/C=cucountrycu/ST=custatecu/L=cucitycu/O=cuorgcu/OU=cuorgunitcu/CN=aodomainao" -new -key aonameao/etc/ssl/private/aodomainao.key -out aonameao/aodomainao.csr
openssl ca -batch -config CA/aonameao.cnf -extfile CA/aonameao4internet.xcnf -days 365 -out aonameao/etc/ssl/certs/aodomainao.crt -infiles aonameao/aodomainao.csr
rm aonameao/aodomainao.csr
```

## Client-Certificate

As we are dealing with the two scenarios of usage in Intranet and via Internet, the Client-Certificate has to cover the server/domain names from both.
The last two command are going to export the Client-Certificate in the PKCS#12-Certificate format. The `.pfx` will be in the "new" format of OpenSSL and the `.p12` will be in the "old" format (option `-legacy`). The `.p12` file will be needed for MacOS/iOS and old Android versions.

```console
cd ~/SSL.canameca
openssl genrsa -utf8 -aes256 -passout pass:secret -out Clients/client-pass.key 4096
openssl rsa -passin pass:secret -in Clients/client-pass.key -out Clients/aonameao_ncadminnc.key
rm Clients/client-pass.key
openssl req -subj "/C=cucountrycu/ST=custatecu/L=cucitycu/O=cuorgcu/OU=cuorgunitcu/CN=ncadminnc" -new -key Clients/aonameao_ncadminnc.key -out Clients/aonameao_ncadminnc.csr
openssl ca -batch -config CA/aonameao.cnf -extfile CA/aonameao4client.xcnf -days 365 -out Clients/aonameao_ncadminnc.crt -infiles Clients/aonameao_ncadminnc.csr
rm Clients/aonameao_ncadminnc.csr
openssl pkcs12 -export -out Clients/aonameao_ncadminnc.pfx -inkey Clients/aonameao_ncadminnc.key -in Clients/aonameao_ncadminnc.crt -certfile CA/canameca.crt
openssl pkcs12 -export -legacy -out Clients/aonameao_ncadminnc.p12 -inkey Clients/aonameao_ncadminnc.key -in Clients/aonameao_ncadminnc.crt -certfile CA/canameca.crt
```

## Revoke Client-Certificate

We prepare as well a test on the revoke of Client-Certificate. The crl information gets added to the directory `aonameao/crl` for later use by the Apache Web-Server..

```console
cd ~/SSL.canameca
openssl ca -config CA/aonameao.cnf -revoke Clients/aonameao_ncadminnc.crt
openssl ca -config CA/aonameao.cnf -gencrl -out CA/crl/aonameao_ncadminnc_$(date +'%y%m%d').crl
cd aonameao/etc/ssl/crl
ln --symbolic ../../../../CA/crl/aonameao_ncadminnc_$(date +'%y%m%d').crl `openssl crl -hash -noout -in ../../../../CA/crl/aonameao_ncadminnc_$(date +'%y%m%d').crl`.r0
cd -
```

## Renew Client-Certificate

Keep the original files, that will be used for one of our tests.

```console
cd ~/SSL.canameca
mkdir Clients/revoked
mv Clients/aonameao_ncadminnc.* Clients/revoked
```

Follow again the instruction on [Client-Certificate](#client-certificate).

## Archive `aohostao` etc

Create an archive of the `etc` directory under `aonameao`.

```console
cd ~/scripts/SSL
mkdir --mode=755 --parents etc/ssl
mkdir --mode=755           etc/ssl/certs
mkdir --mode=755           etc/ssl/crl
mkdir --mode=710           etc/ssl/private
chgrp ssl-cert etc/ssl/crl
chgrp ssl-cert etc/ssl/private
cp ca.canameca/canameca.crt etc/ssl/certs
chmod 644 etc/ssl/certs/canameca.crt
cp aodomainao/aodomainao.crt etc/ssl/certs
chmod 644 etc/ssl/certs/aodomainao.crt
cp aohostao/aohostao.crt etc/ssl/certs
chmod 644 etc/ssl/certs/aohostnao.crt
cp canameca/crl.pem etc/ssl/crl/canameca.crl
chmod 644 etc/ssl/crl/canameca.crl
cp aodomainao/aodomainao.key etc/ssl/private
chmod 640 etc/ssl/private/aodomainao.key
chgrp ssl-cert etc/ssl/private/aodomainao.key
cp aohostao/aohostao.key etc/ssl/private
chmod 640 etc/ssl/private/aohostao.key
chgrp ssl-cert etc/ssl/private/aohostao.key
tar --create --gzip --directory=. --file=etc_$(date +'%y%m%d').tar.gz etc
chown wssysuserws:wssysuserws etc_$(date +'%y%m%d').tar.gz
mv etc_$(date +'%y%m%d').tar.gz /home/wssysuserws
```

## Files

At the end you will have all files required for the deployment/usage.

| file name | purpose |
| :--- | :--- |
| `CA/canameca.crt` | Certificate of canameca |
| `aonameao/etc/ssl/certs/canameca.crt` | symlink to Certificate of canameca |
| `CA/canameca.key` | Key of canameca |
| `aonameao/etc/ssl/certs/aohostao.crt` | for the Web-Server to authenticate itself as Intranet Web-Server |
| `aonameao/etc/ssl/private/aohostao.key` | / |
| `aonameao/etc/ssl/certs/aodomainao.crt` | for the Web-Server to authenticate itself as Internet Web-Server |
| `aonameao/etc/ssl/private/aodomainao.key` | / |
| `aonameao/etc/ssl/crl/<hash>.r0` | Certificate Revocation of `ncadminnc` 1. certificate |
| `Clients/aonameao_ncadminnc.crt` | certificate and key for user `ncadminnc` |
| `Clients/aonameao_ncadminnc.key` | / which certificate gets renewed |
| `Clients/aonameao_ncadminnc.pfx` | PKCS#12 format of certificate for user `ncadminnc` |
| `Clients/aonameao_ncadminnc.p12` | "old" PKCS#12 format of certificate for user `ncadminnc` |
| `Clients/revoked/aonameao_ncadminnc.*` | files of `ncadminnc` 1. certificate |

---

# Server Installation

## OS installation

Have the image for [Debian 12](https://www.debian.org/releases/bookworm/debian-installer/) ready on an USB Stick.

1. insert USB Stick
2. boot machine (ensure the machine will boot from the USB Stick)
3. select "**Installation**" from the GRUB boot menu
4. dialog "Select a language"
	- ✏️ select `English`
5. dialog "Select your location
	- ✏️ select `Germany`
6. dialog "Configure locales
	- ✏️ select `United States`
7. dialog "Configure the keyboard"
	- ✏️ select your country
8. dialog "Configure the network"
	- ✏️ type `aohostao` into "**Hostname**"
	- ✏️ set "**Domain name**" to be empty
9. dialog "Set up users and passwords"
	- ✏️ specify root password (note it down)
10. dialog "Set up users and passwords"
	- ✏️ specify "Full name for the new user"
	- ✏️ type `aosysuserao` into "Username for your account"
	- ✏️ specify "Choose a password for the new user"
11. dialog "Partition disks"
	- ⬆ select "**Manual**"
    - focus on the specific drive
	  - ⬆ select "**FREE SPACE**" entry
	    - ✏️✓ select "**Create a new partition**"
	    - ✏️✓ specify "**New partition size**"
	    - ✏️✓ select "**Primary**"
	    - ✏️✓ specify "**Use as**" to be `Ext2 file system`
      - keep "**Mount point**" as `/`
    	- ✏️✓ specify "**Mount options" to be `noatime,relatime`
    	- ✏️✓ specify "**Label" to be `ACNO`
    	- ✏️✓ specify "**Bootable flag**" to be `on`
      - ✓ select "**Done setting up the partition**"
	  - select "**Finish partitioning and write changes to disk**"
	  - ✏️✓ select "**No**" on the question about "swap space"
	  - ✓🔙 select "**Yes**" to get the changes written onto the disk
12. dialog "Configure the package manager"
	- ✏️ select your country as "**Debian archive mirror country**"
	- ✏️ select your mirror as "**Debian archive mirror**"
	- ✏️ specify no "**HTTP proxy information (blank for none)**"
13. dialog "Configuring popularity-contest"
	- ✏️ select "**Yes**"
14. dialog "Software selection"
	- ✏️✓ ~~disable~~ "**Debian desktop environment**"
	- ✏️✓ ~~disable~~ "**GNOME**"
  - ✏️✓ enable "**web server**"
	- ✏️✓ enable "**SSH server**"
  - keep "**standard system utilities**" enabled
	- ✓ select "Continue"
15. dialog "grub-pc"
	- ✏️ select "**Yes**" to install GRUB boot loader
	- ✏️ select drive (used to previously create partition)
16. dialog "Finish the installation"
	- remove USB stick
	- select "**Continue**"¹
17. reboot take place, ending with login prompt²

¹ We will use the reboot to go into the BIOS and remove the USB devices as boot option.

² The 1st reboot of the system should have been successful.

Note: *From now on, the communication with the server will be done using ssh and http(s).*

### Post activities

Note: *In case a backup exist, which is going to be restored on the machine ... the following steps are not required.*

#### GRUB

Hide the GRUB menu during boot time. If need at a later point in time, keep the SHIFT key press right after start.

☝ [Notes: Server Installation > OS installation > Post activities > GRUB](Notes.md#grub)

```console
cp /etc/default/grub /etc/default/grub.ORG
sed --in-place --expression="s/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0\nGRUB_TIMEOUT_STYLE=hidden/" /etc/default/grub
cat << EOF >> /etc/default/grub

# Specify Background image (or "" for none)
GRUB_BACKGROUND=""
EOF
update-grub
```

=== `/etc/default/grub` ===

```code
:
GRUB_TIMEOUT=0
GRUB_TIMEOUT_STYLE=hidden
:

# Specify Background image (or "" for none)
GRUB_BACKGROUND=""
```

#### OpenSSH for SSH-Server & -Client-Authentication

☝ [Notes: Server Installation > OS installation > Post activities > OpenSSH for SSH-Server & -Client-Authentication](Notes.md#openssh-for-ssh-server---client-authentication)

##### Client-Keys

Like for the CA files ... do not loose the Client-Keys. Otherwise you do have to get physical access to the machine `aohostao`, in order to login in again.

The following steps have to be done on your workstation, used to remotely login to the machine `nchost`.

###### As regular user `wssysuserws`

In case you have no ssh-key pair yet ... create it using the following command (and press ENTER when asked for passphrase)

```console
$ ls ~/.ssh
known_hosts  known_hosts.old
```

If files `id_rsa` and `id_rsa.pub` are listed as well, you have already prepared your workstation and therefor have to skip the following commands.

$ ssh-keygen -t rsa -b 4096 -C "wssysuserws wshostws" -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/wssysuserws/.ssh/id_rsa
Your public key has been saved in /home/wssysuserws/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:z5s8Vby75poeVE4YfRK3jnAZ/RFaHjMPIn5fLRj2xZI wssysuserws wshostws
The key's randomart image is:
+---[RSA 4096]----+
|          o      |
|         . = o . |
|          E *.**o|
|         . + +B*B|
|        S . o+o==|
|         o o. ++=|
|          oo o ++|
|         ..oo *.o|
|          +. ++* |
+----[SHA256]-----+
$ ls ~/.ssh
id_rsa	id_rsa.pub  known_hosts  known_hosts.old
```

Copy the key to the `aohostao` machine. During the `ssh-copy-id` execution, you get asked for the users password ... that's the password of machine `aohostao` and not your workstation.

```console
$ ssh-copy-id aosysuserao@aohostao
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/aosysuserao/.ssh/id_rsa.pub"
The authenticity of host 'aohostao (10.0.0.20)' can't be established.
ED25519 key fingerprint is SHA256:td4yIx6Ap/66c5rweUl/HxGC0W1gLww3rwdoHeR9Ang.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
aosysuserao@aohostao's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'aosysuserao@aohostao'"
and check to make sure that only the key(s) you wanted were added.
```

Check out if the login works, using the key.

```console
$ ssh aosysuserao@aohostao
Linux aohostao 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
aosysuserao@aohostao:~$ 
```

##### Disable Password usage

To be done on the machine `aohostao`.

To prevent password authentication completely, even when using `ssh-copy-id` should be made impossible, the "authentication_keys" have to be stored in a file with user access rights reduced to Readonly.
This means only the `root` account will be able to authorize a new public key. The public keys - per user - will be stored in a file with the name identicaly to the user name (e.g. `aosysuserao`). To simplify the administration, these files will be maintained in directory `/etc/ssh/authorized_keys.d`.

Note: Do not logout, until you have verified via another session, that you are able to access `aohostao`!

1. create the central directory for the "authentication_keys" files
2. move the existing `authorized_keys` file of `aosysuserao` into the newly create directory 
3. rename "authentication_keys" file and reduce its the access rights
4. create an additional config file reflecting the above changes
5. create an additional config file to hide OS information (e.g. `ssh-keyscan`)
6. restart the ssh service, using command `systemctl restart ssh`

```console
mkdir --mode=755 /etc/ssh/authorized_keys.d
mv /home/aosysuserao/.ssh/authorized_keys /etc/ssh/authorized_keys.d/aosysuserao
chown root:root /etc/ssh/authorized_keys.d/aosysuserao
chmod 644 /etc/ssh/authorized_keys.d/aosysuserao
cat << EOF > /etc/ssh/sshd_config.d/using_authorized_keys.d.conf
# Specifies whether public key authentication is allowed. The default is yes.
PubkeyAuthentication yes

# Authorized public keys stored at a single place.
# User has only Read-Access.
AuthorizedKeysFile /etc/ssh/authorized_keys.d/%u

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no
EOF
cat << EOF > /etc/ssh/sshd_config.d/hide_os.d.conf
# Specifies whether the distribution-specified extra version suffix is
# included during initial protocol handshake. The default is yes.
DebianBanner no
EOF
systemctl restart ssh
```

===`/etc/ssh/sshd_config.d/using_authorized_keys.d.conf` ===

```code
# Specifies whether public key authentication is allowed. The default is yes.
PubkeyAuthentication yes

# Authorized public keys stored at a single place.
# User has only Read-Access.
AuthorizedKeysFile /etc/ssh/authorized_keys.d/%u

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no
```

==== `/etc/ssh/sshd_config.d/hide_os.d.conf` ===

```code
# Specifies whether the distribution-specified extra version suffix is
# included during initial protocol handshake. The default is yes.
DebianBanner no
```

##### Test as root

Try out the remote login as `aosysuserao`.

1. Attempt: using the publikc key of root -> fails
2. Attempt: using the publikc key of aosysuserao -> succeeds

```console
# ssh aosysuserao@aohostao
The authenticity of host 'aohostao (ip-address)' can't be established.
ED25519 key fingerprint is SHA256:td4yIx6Ap/66c5rweUl/HxGC0W1gLww3rwdoHeR9Ang.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'aohostao' (ED25519) to the list of known hosts.
aosysuserao@aohostao: Permission denied (publickey).
# sudo --user=aosysuserao ssh aohostao
Linux aohostao 6.1...
:
```

#### SSD

To reduce wear of the SSD, the disk already gets mounted with specific parameter that reduce write operations. In addition, some information are temporary and might not be required to be stored on disk and therefore RAM-Disk directories will be prepared.

☝ [Notes: Server Installation > OS installation > Post activities > SSD](Notes.md#ssd)

##### RAM-Disk

Create some OS folders as RAM-Disk

```code
cp /etc/fstab /etc/fstab.ORG
cat << EOF >> /etc/fstab
# --- START OF ACNO SPECIFIC DEFINITIONS --- #

# Temporary files onto tmpfs
none /tmp tmpfs defaults,noatime,mode=1777 0 0
none /var/tmp tmpfs defaults,noatime 0 0
none /var/log tmpfs defaults,noatime 0 0
none /var/cache tmpfs defaults,noatime 0 0
EOF
```

=== `/etc/fstab` ===

```code
:
# --- START OF ACNO SPECIFIC DEFINITIONS --- #

# Temporary files onto tmpfs
none /tmp tmpfs defaults,noatime,mode=1777 0 0
none /var/tmp tmpfs defaults,noatime 0 0
none /var/log tmpfs defaults,noatime 0 0
none /var/cache tmpfs defaults,noatime 0 0
```

Note: The directory `/run` is a tmpfs and available inDebian 10 and higher.

###### `/var/log` folders

The installed software expects to have always the needed log-directories accessible for its account. This can be achieved by making use of  `/etc/tmpfiles.d/log-subfolder.conf`. Once done, reboot the machine and the folders get created.

```code
cat << EOF > /etc/tmpfiles.d/log-file.conf
f       /var/log/clamav/clamav.log                    0664 clamav clamav - -
f       /var/log/clamav/freshclam.log                 0664 clamav clamav - -
EOF
cat << EOF > /etc/tmpfiles.d/log-subfolder.conf
d       /var/log/apache2                              0775 www-data www-data - -
d       /var/log/clamav                               0775 clamav clamav - -
d       /var/log/lighttpd                             0775 www-data www-data - -
d       /var/log/msmtp                                0775 msmtp msmtp - -
d       /var/log/nextcloud                            0775 www-data www-data - -
d       /var/log/nginx                                0775 www-data www-data - -
d       /var/log/onlyoffice                           0775 ds ds - -
d       /var/log/onlyoffice/documentserver            0775 ds ds - -
d       /var/log/onlyoffice/documentserver/metrics    0775 ds ds - -
d       /var/log/onlyoffice/documentserver/converter  0775 ds ds - -
d       /var/log/onlyoffice/documentserver/docservice 0775 ds ds - -
d       /var/log/onlyoffice/documentserver-example    0775 ds ds - -
d       /var/log/openvpn                              0775 root root - -
d       /var/log/rabbitmq                             0775 rabbitmq rabbitmq - -
d       /var/log/redis                                0775 redis redis - -
d       /var/log/turn                                 0775 turnserver turnserver - -
EOF
```

=== `/etc/tmpfiles.d/log-subfolder.conf` ===

```code
d       /var/log/apache2                              0775 www-data www-data - -
d       /var/log/lighttpd                             0775 www-data www-data - -
d       /var/log/msmtp                                0775 msmtp msmtp - -
d       /var/log/nextcloud                            0775 www-data www-data - -
d       /var/log/nginx                                0775 www-data www-data - -
d       /var/log/onlyoffice                           0775 ds ds - -
d       /var/log/onlyoffice/documentserver            0775 ds ds - -
d       /var/log/onlyoffice/documentserver/metrics    0775 ds ds - -
d       /var/log/onlyoffice/documentserver/converter  0775 ds ds - -
d       /var/log/onlyoffice/documentserver/docservice 0775 ds ds - -
d       /var/log/onlyoffice/documentserver-example    0775 ds ds - -
d       /var/log/openvpn                              0775 root root - -
d       /var/log/rabbitmq                             0775 rabbitmq rabbitmq - -
d       /var/log/redis                                0775 redis redis - -
d       /var/log/turn                                 0775 turnserver turnserver - -
```

=== `/etc/tmpfiles.d/log-file.conf` ===

```code
f       /var/log/clamav/clamav.log                    0664 clamav clamav - -
f       /var/log/clamav/freshclam.log                 0664 clamav clamav - -
```

#### Locales

To be done, especially when using different database locales `aodblocalesao`, as defined by Step 6. of "OS installation".

☝ [Notes: Server Installation > OS installation > Post activities > Locales](Notes.md#locales)

1. start `dpkg-reconfigure locales`
2. identify line with `aodblocalesao`
3. ensure the line is enabled
4. press TAB to select "OK"
5. press ENTER to update/install the additional locales
6. answer with "OK" on the question `Default locale for the system environment` and `en_US.UTF-8` highlighted

#### Tools

Required, for the setup of additional Software Package Resources, are some basic tools.

☝ [Notes: Server Installation > OS installation > Post activities > Tools](Notes.md#tools)

```console
apt install --yes curl git gpg sudo unzip
```

#### Finish OS Setup

In order to find your machine IP address based on the MAC address, checkout command `ip a`.

- shutdown the machine
- detach monitor and keyboard
- start the machine
- ensure the machine can be still reached via ssh

## Nextcloud Prerequisites

### OS

#### Repositories

A must-have is to be able to install software from the categories `contrib` and `non-free`.

Optional is either the external repository for Collabora Online/CODE or ONLYOFFICE Docs.

☝ [Notes: Server Installation > Nextcloud Prerequisites > OS > Repositories](Notes.md#repositories)

##### Repository Category contrib and non-free

Add `contrib` and `non-free` area to the Repository.

```console
cp /etc/apt/sources.list /etc/apt/sources.list.ORG
sed --in-place --expression='s/main non-free-firmware/main contrib non-free non-free-firmware/g' /etc/apt/sources.list
```

=== `/etc/apt/sources.list` ===

```code
deb http://ftp.gwdg.de/debian/ bookworm main contrib non-free-firmware
deb-src http://ftp.gwdg.de/debian/ bookworm main contrib non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free-firmware

# bookworm-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://ftp.gwdg.de/debian/ bookworm-updates main contrib non-free-firmware
deb-src http://ftp.gwdg.de/debian/ bookworm-updates main contrib non-free-firmware
```

##### Repository for Collabora Online/CODE

The packages for Collabora Online/CODE are not part of the usual Debian Packages, therefor the repository from Collabora Limited has to be included.

```console
mkdir /etc/apt/keyrings
wget --output-document=/etc/apt/keyrings/collaboraonline-release-keyring.gpg https://collaboraoffice.com/downloads/gpg/collaboraonline-release-keyring.gpg
chmod 644 /etc/apt/keyrings/collaboraonline-release-keyring.gpg
cat << EOF > /etc/apt/sources.list.d/collaboraonline.sources
Types: deb
URIs: https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb
Suites: ./
Signed-By: /etc/apt/keyrings/collaboraonline-release-keyring.gpg
EOF
chmod 644 /etc/apt/sources.list.d/collaboraonline.sources
apt update
```

=== `/etc/apt/sources.list.d/collaboraonline.sources` ===

```code
Types: deb
URIs: https://www.collaboraoffice.com/repos/CollaboraOnline/CODE-deb
Suites: ./
Signed-By: /etc/apt/keyrings/collaboraonline-release-keyring.gpg
```

##### Repository for ONLYOFFICE Docs

The packages for ONLYOFFICE Docs are not part of the regular Debian Packages, therefor the repository from Ascensio System SIA has to be included.

```console
mkdir /etc/apt/keyrings
curl --fail \
     --silent \
     --show-error \
     --location \
     https://download.onlyoffice.com/GPG-KEY-ONLYOFFICE | \
  gpg --no-default-keyring \
      --keyring gnupg-ring:/etc/apt/keyrings/onlyoffice.gpg \
      --import
chmod 644 /etc/apt/keyrings/onlyoffice.gpg
cat << EOF > /etc/apt/sources.list.d/onlyoffice.sources
Types: deb
URIs: https://download.onlyoffice.com/repo/debian
Suites: squeeze
Components: main
Signed-By: /etc/apt/keyrings/onlyoffice.gpg
EOF
chmod 644 /etc/apt/sources.list.d/onlyoffice.sources
apt update
```

=== `/etc/apt/sources.list.d/onlyoffice.sources` ===

```code
Types: deb 
URIs: https://download.onlyoffice.com/repo/debian
Suites: squeeze
Components: main
Signed-By: /etc/apt/keyrings/onlyoffice.gpg
```

#### Software

##### Software for Add-On

You can also install Let's Encrypt and OpenVPN from section [Add-On](#add-on). Just in case you do decide later on to choosing these options. Ignore the question about backup, as it is a new installation.

```console
apt install --yes certbot iptables iptables-persistent netfilter-persistent openvpn python3-certbot-apache
```

##### Basis Software

Install more of the required software. 

Note: *During the install process a dialog pop-up, asking for information, in regards to the ONLYOFFICE database ... press ENTER to continue. As a result the installation ends with an error, but that's fine ... this will be handled later.*

Before installing the remaining software, stop the Apache Web-Server, as the system will get the nginx Web-Server as part of ONLYOFFICE installed. That will collides with the ports, if Apache would be still running ... nevertheless we will deal with that later as well.

```console
apt install --yes coturn ffmpeg imagemagick libapache2-mod-php ntfs-3g php php-apcu php-bcmath php-bz2 php-cli php-common php-curl php-dev php-dompdf php-fpm php-gd php-gmp php-imagick php-intl php-json php-mbstring php-pear php-pgsql php-redis php-soap php-xml php-zip postgresql redis
```

##### Software for Collabora Online/CODE

```console
apt install --yes coolwsd code-brand fonts-crosextra-carlito fonts-liberation fonts-opensymbol fonts-takao-gothic supervisor
```

##### Software for ONLYOFFICE Docs

```console
echo onlyoffice-documentserver onlyoffice/ds-port select oodsportoo | debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-host string localhost | debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-user string oodbooo | debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-pwd password oodbopasswordoo | debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-name string oodatabaseoo | debconf-set-selections
echo onlyoffice-documentserver onlyoffice/jwt-secret password oosecretkeyoo | debconf-set-selections
apt install --yes nginx-extras onlyoffice-documentserver rabbitmq-server ttf-mscorefonts-installer
```

#### Certificates & Co.

Copy the created files from section [Certificates](#certificates) from workstation `wshostws` onto machine `aohostao`.

##### On Workstation `wshostws`

```console
scp /home/wssysuserws/etc_YYMMDD.tar.gz aosysuserao@aohostao:/home/aosysuserao
```

##### On Machine `aohostao`

```console
tar --extract --gzip --directory=/ --file=/home/aosysuserao/etc_YYMMDD.tar.gz
```

Add your own ca to the list of Certificate Authorities of the system (incl. Mono environment)

```console
mkdir --mode=755 /usr/local/share/ca-certificates/my-custom-ca
ln --symbolic /etc/ssl/certs/canameca.crt /usr/local/share/ca-certificates/my-custom-ca
update-ca-certificates
```

### Script-Runtime - PHP

#### `php.ini` modifications

Check the existance of the `php.ini` files.

```console
# find / -name php.ini
/etc/php/8.2/apache2/php.ini
/etc/php/8.2/fpm/php.ini
/etc/php/8.2/cli/php.ini
# 
```

Make the modification either automatically or manually. However at the end the changes have to look like the following.

```console
# grep -e "^memory_limit" -e "^output_buffering" -e "^max_execution_time" -e "^post_max_size" -e "^upload_max_filesize" -e "^date.timezone" -e "^opcache.enable" -e "^opcache.memory_consumption" -e "^opcache.interned_strings_buffer" -e "^opcache.max_accelerated_files" -e "^opcache.revalidate_freq" -e "^opcache.save_comments" -e "^\[APCu\]" -e "^apc.enable_cli" /etc/php/8.2/cli/php.ini
output_buffering = Off
max_execution_time = 300
memory_limit = -1
post_max_size = 600M
upload_max_filesize = 500M
date.timezone = Europe/Amsterdam
opcache.enable=1
opcache.memory_consumption=1024
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=10000
opcache.revalidate_freq=1
opcache.save_comments=1
[APCu]
apc.enable_cli=1
# grep -e "^memory_limit" /etc/php/8.2/apache2/php.ini /etc/php/8.2/fpm/php.ini
/etc/php/8.2/apache2/php.ini:memory_limit = 512M
/etc/php/8.2/fpm/php.ini:memory_limit = 512M
# 
```

##### Automated

The easiest way is to patch all three `php.ini` files using `sed`.

```console
cp /etc/php/8.2/apache2/php.ini /etc/php/8.2/apache2/php.ini.ORG
cp /etc/php/8.2/fpm/php.ini /etc/php/8.2/fpm/php.ini.ORG
cp /etc/php/8.2/cli/php.ini /etc/php/8.2/cli/php.ini.ORG
sed --in-place \
    --regexp-extended \
    --expression="s/^output_buffering *= *[0-9]+$/output_buffering = Off/" \
    --expression="s/^max_execution_time *= *[0-9]+$/max_execution_time = 300/" \
    --expression="s/^post_max_size *= *[0-9]+[GgKkMm]?$/post_max_size = 600M/" \
    --expression="s/^upload_max_filesize *= *[0-9]+[GgKkMm]?$/upload_max_filesize = 500M/" \
    --expression="s/^;date.timezone *=.*$/date.timezone = Europe\/Amsterdam/" \
    --expression="s/^;opcache.enable *=.*$/opcache.enable=1/" \
    --expression="s/^;opcache.memory_consumption *= *[0-9]+$/opcache.memory_consumption=1024/" \
    --expression="s/^;opcache.interned_strings_buffer *= *[0-9]+$/opcache.interned_strings_buffer=16/" \
    --expression="s/^;opcache.max_accelerated_files *= *[0-9]+$/opcache.max_accelerated_files=10000/" \
    --expression="s/^;opcache.revalidate_freq *= *[0-9]+$/opcache.revalidate_freq=1/" \
    --expression="s/^;opcache.save_comments *= *[0-9]+$/opcache.save_comments=1/" \
    --expression="$ a \\\n[APCu]\napc.enable_cli=1" \
    /etc/php/8.2/cli/php.ini /etc/php/8.2/apache2/php.ini /etc/php/8.2/fpm/php.ini
sed --in-place \
    --regexp-extended \
    --expression="s/^memory_limit *= *[0-9]+M$/memory_limit = 512M/" \
    /etc/php/8.2/apache2/php.ini /etc/php/8.2/fpm/php.ini
```

##### Manually

Adjust the PHP configuration.

| Property | original | modified |
| ---: | :---: | :---: |
| display_errors | n/a | Off² |
| output_buffering | n/a | Off |
| max_execution_time | 30 | 300 |
| memory_limit | 128M³ | 512M³ |
| post_max_size | 8M | 600M |
| upload_max_filesize | 2M | 500M |
| zend_extension | n/a | opcache¹ |
| date.timezone | n/a | Europe/Amsterdam |
| opcache.enable | n/a | 1 |
| opcache.memory_consumption | n/a | 1024 |
| opcache.interned_strings_buffer | n/a | 16 |
| opcache.max_accelerated_files | n/a | 10000 |
| opcache.revalidate_freq | n/a | 1 |
| opcache.save_comments | n/a | 1 |

¹ *Change not required for Debian.*

² *It is the default value.*

³ *For cli, leave the value unchanged as `-1`*

*Note: "n/a" means, the attribute was not set (commented out)*

Add add the following lines.

```code

[APCu]
apc.enable_cli=1
```

For Debian the step of enabling extensions is not required, as they are already build-in. So the following is just to cover this aspect as well.

#### Check

Note: That will be the first check to verify

1. the Web-Server is up and running.¹
2. the Web-Server is configured for PHP execution.
3. the PHP module has all required modules.

¹ *It might be required to restart Apache, using command `systemctl restart apache2`.*

Create in the `html` directory a php file, which does nothing else than calling `phpinfo()`.

```console
sudo --user=www-data echo '<?php phpinfo(); ?>' > /var/www/html/info.php
```

The Check: Open the URL "http://aohostao/info.php". Verify that the most relevant parts are available. See [Installation and server configuration / PHP Modules & Configuration](https://docs.nextcloud.com/server/latest/admin_manual/installation/php_configuration.html)

Note: Do not forget to delete the file, once your installation is running.

### Database - PostgreSQL 15

1. add user `postgres`to group `backup`
2. modify the `postgresql.conf` file
	1. by removing leading "#" of line `listen_addresses = 'localhost'` definition
	2. by changing the socket permission from `0777` to `0770`
3. modify the `pg_hba.conf` file
  1. change the authentication via Unix-Sockets from "peer" to "md5"
4. restart the PostgreSQL service, using command `systemctl restart postgresql`
5. set password for `postgres`user

```console
usermod --append --groups backup postgres
cp /etc/postgresql/15/main/postgresql.conf /etc/postgresql/15/main/postgresql.conf.ORG
sed -i -e "s/#listen_addresses = 'localhost'/listen_addresses = 'localhost'/" -e "s/#unix_socket_permissions = 0777/unix_socket_permissions = 0770/" /etc/postgresql/15/main/postgresql.conf
cp /etc/postgresql/15/main/pg_hba.conf /etc/postgresql/15/main/pg_hba.conf.ORG
sed --in-place --expression="s/\(^local[[:space:]]*all[[:space:]]*all[[:space:]]*\)peer/\1md5/" /etc/postgresql/15/main/pg_hba.conf
systemctl restart postgresql
sudo --user=postgres psql --command="ALTER USER postgres WITH PASSWORD 'psadmpasswordps';"
```

=== `/etc/postgresql/15/main/postgresql.conf` ===

```code
:

# - Connection Settings -

listen_addresses = 'localhost'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
port = 5432                             # (change requires restart)
max_connections = 100                   # (change requires restart)
#superuser_reserved_connections = 3     # (change requires restart) 
unix_socket_directories = 'pssocketps'  # comma-separated list of directories
                                        # (change requires restart)
#unix_socket_group = ''                 # (change requires restart)
unix_socket_permissions = 0770          # begin with 0 to use octal notation
                                        # (change requires restart)
#bonjour = off                          # advertise server via Bonjour
                                        # (change requires restart)
#bonjour_name = ''                      # defaults to the computer name
                                        # (change requires restart)

:
```

=== `/etc/postgresql/15/main/pg_hba.conf.ORG` ===

```code
# "local" is for Unix domain socket connections only
local   all             all                                     peer
```

=== `/etc/postgresql/15/main/pg_hba.conf` ===

```code
:
# "local" is for Unix domain socket connections only
local   all             all                                     md5
:
```

### Database - Redis

1. modify the `redis.conf` file
	1. by removing leading "#" of line `unixsocket ...` definition
	2. by changing the socket permission from `0700` to `0770`
2.restart the Redis service, using command `systemctl restart redis`

```console
cp /etc/redis/redis.conf /etc/redis/redis.conf.ORG
sed -i -e "s/^# unixsocket \/\(.*\)/unixsocket \/\1/" -e "s/^# unixsocketperm 700/unixsocketperm 770/" /etc/redis/redis.conf
systemctl restart redis
```

Enable Unix socket and ensure also member of os group `redis` will be able to access it.

=== `/etc/redis/redis.conf` ===

```code
:

# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
unixsocket /run/redis/redis-server.sock
unixsocketperm 770

:
```

### coturn

#### Create database

```console
sudo --user=postgres psql --host=ctdbhostct --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# CREATE USER ctdboct WITH PASSWORD 'ctdbopasswordct';
CREATE ROLE
postgres=# CREATE DATABASE ctdatabasect TEMPLATE template0 OWNER ctdboct LC_COLLATE 'aodblocalesao' LC_CTYPE 'aodblocalesao';
CREATE DATABASE
postgres=# GRANT ALL privileges ON DATABASE ctdatabasect TO ctdboct;
postgres=# \q
```

#### Prepare database

```console
# sudo --user=postgres psql --host=ctdbhostct --dbname=ctdatabasect --username=ctdboct --password -f /usr/share/coturn/schema.sql
could not change directory to "/root": Permission denied
Password: 
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
CREATE TABLE
# 
```

#### Configuration

Several changes are required

1. but first make a backup of the original configuration and service definition
2. adjust the service, by adding the argument `--psql-userdb`
3. configuration starts from the scratch
4. in `/usr/local/etc` create references to the certificates and key (I know ... not very secure)
5. restart coturn

```console
cp /etc/ssl/certs/canameca.crt /usr/local/etc
chown turnserver:turnserver /usr/local/etc/canameca.crt
cp /etc/ssl/certs/aodomainao.crt /usr/local/etc
chown turnserver:turnserver /usr/local/etc/aodomainao.crt
cp /etc/ssl/private/aodomainao.key /usr/local/etc
chown turnserver:turnserver /usr/local/etc/aodomainao.key
cp /etc/turnserver.conf /etc/turnserver.conf.ORG
cp /lib/systemd/system/coturn.service /lib/systemd/system/coturn.service.ORG
sed -i -e 's/turnserver.conf --pidfile=/turnserver.conf --psql-userdb="host=ctdbhostct dbname=ctdatabasect user=ctdboct password=ctdbopasswordct connect_timeout=30" --pidfile=/' /lib/systemd/system/coturn.service
cat << EOF > /etc/turnserver.conf
log-file=/var/log/turn/server.log

realm=aodomainao
server-name=turn.aodomainao
listening-ip=0.0.0.0
listening-port=3478
tls-listening-port=5349
min-port=10000
max-port=20000

CA-file=/usr/local/etc/canameca.crt
cert=/usr/local/etc/aodomainao.crt
pkey=/usr/local/etc/aodomainao.key
cipher-list="ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384"
no-tlsv1
no-tlsv1_1
no-stdout-log

fingerprint
use-auth-secret
static-auth-secret=ctsecretct
total-quota=100
bps-capacity=0
stale-nonce=600
EOF
systemctl daemon-reload
systemctl restart coturn
```

=== `/etc/turnserver.conf` ===

```code
log-file=/var/log/turn/server.log

realm=aodomainao
server-name=ctserverct.aodomainao
listening-ip=0.0.0.0
listening-port=3478
tls-listening-port=5349
min-port=10000
max-port=20000

CA-file=/usr/local/etc/canameca.crt
cert=/usr/local/etc/aodomainao.crt
pkey=/usr/local/etc/aodomainao.key
cipher-list="ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384"
no-tlsv1
no-tlsv1_1
no-stdout-log

fingerprint
use-auth-secret
static-auth-secret=ctsecretct
total-quota=100
bps-capacity=0
stale-nonce=600
```

=== `/lib/systemd/system/coturn.service` ===

```code
:
ExecStart=/usr/bin/turnserver -c /etc/turnserver.conf --psql-userdb="host=ctdbhostct dbname=ctdatabasect user=ctdboct password=ctdbopasswordct connect_timeout=30" --pidfile=
:
```

### Collabora DocumentServer/CODE

For Nextcloud there are two Apps for Collabora listed. Where as the first one is a must, as it is the integration part for Collabora Online/CODE. The second app must not be installed, as this document leads through the installation of the Collabora Online/CODE instance.

- **[Nextcloud Office](https://apps.nextcloud.com/apps/richdocuments)**
- ~~[Collabora Online - Built-in CODE Server](https://apps.nextcloud.com/apps/richdocumentscode)~~

Software should have been installed already. See [Software for Collabora Online/CODE](#software-for-collabora-onlinecode)

The Apache Web-Server will be acting as proxy for the Collabora Online/CODE server and therefore encrypted communication by the Collabora server is not required.

```console
cp /etc/coolwsd/coolwsd.xml /etc/coolwsd/coolwsd.xml.ORG
sed --in-place\
    --expression="s/\(Connection via proxy .* default=.false.>\)false\(<\/termination>\)/\1true\2/" \
    --expression="s/\(Controls whether SSL encryption between coolwsd .* default=.true.>\)true\(<\/enable>\)/\1false\2/" \
    --expression="s/\(Controls whether SSL encryption between coolwsd .* default=.true.>\)true\(<\/enable>\)/\1false\2/" \
    --expression="s/\(JSON file that lists fonts .* default=..>\)\(<\/url>\)/\1https:\/\/coserverco.aodomainao\/index.php\/apps\/richdocuments\/settings\/fonts.json\2/" \
    /etc/coolwsd/coolwsd.xml
systemctl restart coolwsd
```

### ONLYOFFICE Docs

For Nextcloud there are two Apps for ONLYOFFICE listed. Were as this guide is for the use of "ONLYOFFICE" (onlyoffice) App intended.

- **[ONLYOFFICE](https://apps.nextcloud.com/apps/onlyoffice)**
- [Community Document Server](https://apps.nextcloud.com/apps/documentserver_community)

Before we start, we want to make sure that ONLYOFFICE will be allowed of using Unix Domain Socket (aka socket) for communication with the PostgreSQL and Redis. That is achieved by "ds" becoming member of groups "postgres" and "redis".

```console
usermod --append --groups postgres,redis ds
```

Software should have been installed already. See [Software for ONLYOFFICE Docs](#software-for-onlyoffice-docs)

#### Create robots.txt in ONLYOFFICE Docs space

```console
cat << EOF > /var/www/onlyoffice/robots.txt
User-agent: *
Disallow: /
EOF
```

#### Disable Welcome Page

```console
sed --in-place '--expression=s/^\(location = \/ { return 302\)/#\1/' /etc/onlyoffice/documentserver/nginx/includes/ds-docservice.conf
```

#### Create database

```console
sudo --user=postgres psql --host=oodbhostoo --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# CREATE USER oodbooo WITH PASSWORD 'oodbopasswordoo';
CREATE ROLE
postgres=# CREATE DATABASE oodatabaseoo TEMPLATE template0 OWNER oodbooo LC_COLLATE 'aodblocalesao' LC_CTYPE 'aodblocalesao';
CREATE DATABASE
postgres=# GRANT ALL privileges ON DATABASE oodatabaseoo TO oodbooo;
postgres=# \q
```

#### Prepare database

```console
# sudo --user=postgres psql --host=oodbhostoo --dbname=oodatabaseoo --username=oodbooo --password -f /var/www/onlyoffice/documentserver/server/schema/postgresql/createdb.sql
could not change directory to "/root": Permission denied
Password: 
CREATE TABLE
CREATE TABLE
CREATE FUNCTION
```

### Proxy-Server - Nginx

☝ [Notes: Server Installation > Nextcloud Prerequisites > Proxy-Server - Nginx](./Notes.md#proxy-server---nginx)

### Web-Server - Apache2

#### Hide Server Information

Adjust the settings for `ServerTokens` and `ServerSignature` to hide information about service Web-Server and underlying OS.

```console
cp /etc/apache2/conf-available/security.conf /etc/apache2/conf-available/security.conf.ORG
sed --in-place \
    --expression="s/^\(#ServerTokens Minimal.*\)/ServerTokens Prod\n\1/" \
    --expression="s/^\(ServerTokens OS.*\)/#\1/" \
    --expression="s/^#\(ServerSignature Off\)/\1/" \
    --expression="s/^\(ServerSignature On\)/#\1/" \
    /etc/apache2/conf-available/security.conf
```

The change takes place after a reload/restart of the Web-Server. Take the chance to observer the behaviour before and after.


```console
root@aohostao:~# curl --head --no-progress-meter 127.0.0.1 | grep ^Server
Server: Apache/2.4.65 (Debian) OpenSSL/3.5.1
root@aohostao:~# systemctl restart apache2
root@aohostao:~# curl --head --no-progress-meter 127.0.0.1 | grep ^Server
Server: Apache
```

=== `/etc/apache2/conf-available/security.conf.ORG` ===

```code
:

#
# ServerTokens
# This directive configures what you return as the Server HTTP response
# Header. The default is 'Full' which sends information about the OS-Type
# and compiled in modules.
# Set to one of:  Full | OS | Minimal | Minor | Major | Prod
# where Full conveys the most information, and Prod the least.
ServerTokens Prod
#ServerTokens Minimal
#ServerTokens OS
#ServerTokens Full

#
# Optionally add a line containing the server version and virtual host
# name to server-generated pages (internal error documents, FTP directory
# listings, mod_status and mod_info output etc., but not CGI generated
# documents or custom error documents).
# Set to "EMail" to also include a mailto: link to the ServerAdmin.
# Set to one of:  On | Off | EMail
ServerSignature Off
#ServerSignature On

:
=== `/etc/apache2/conf-available/security.conf` ===

```code
:

#
# ServerTokens
# This directive configures what you return as the Server HTTP response
# Header. The default is 'Full' which sends information about the OS-Type
# and compiled in modules.
# Set to one of:  Full | OS | Minimal | Minor | Major | Prod
# where Full conveys the most information, and Prod the least.
ServerTokens Prod
#ServerTokens Minimal
#ServerTokens OS
#ServerTokens Full

#
# Optionally add a line containing the server version and virtual host
# name to server-generated pages (internal error documents, FTP directory
# listings, mod_status and mod_info output etc., but not CGI generated
# documents or custom error documents).
# Set to "EMail" to also include a mailto: link to the ServerAdmin.
# Set to one of:  On | Off | EMail
ServerSignature Off
#ServerSignature On

:
```

#### Web-Site configuration

Before we forget it ... for the RSA 4096 bit key, create a corresponding new Diffie-Hellman group.

```console
openssl dhparam -out /etc/ssl/certs/dhparams.aodomainao.pem 4096
```

Now we continue with the configuration...
Start by doing the following steps

1. stop the Web-Server
2. replace the `index.*` files from apache and nginx by an empty `index.html`
3. create the directory `/var/www/ncdirectorync`, so Apache does not complain on its first start
4. enable the required modules
5. disable the original Web-Site
6. create a stub Web-Site configuration and enable this
8. keep the Web-Server stopped until Nextcloud is installed

```console
systemctl stop apache2
rm /var/www/html/info.php /var/www/html/index.* 
mkdir /var/www/ncdirectorync
touch /var/www/html/index.html
cat << EOF > /var/www/html/robots.txt
User-agent: *
Disallow: /
EOF
a2enmod headers proxy proxy_http proxy_wstunnel rewrite ssl
a2dissite 000-default
mv /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf.ORG
touch /etc/apache2/sites-available/any.conf
a2ensite any
systemctl stop apache2
cd /etc/apache2/sites-available
```

Create all files for the Web-Sites served by the Apache Web-Server.

Note: In file `any.conf` the directive "<IfDefine !HTTPonly>" is been used. That might become handy under some circumstances, predominantly when using directly ["Let's Encrypt"](#lets-encrypt). That allows to start the Web-Server, regardsless of certificate existance, with http only. See Section "Add-On > Let's Encrypt" below for further details.

=== `/etc/apache2/sites-available/any.conf` ===

```code
ServerName aohostao

#LogFormat      "%V %{%b %d %H:%M:%S}t %a:%{remote}p \"%{User-agent}i\" %{SSL_CLIENT_VERIFY}x %u \"%r\" %>s %b"
#ErrorLogFormat "%V %{%b %d %H:%M:%S}t %a \"%{User-agent}i\" %F: %E: %M"

# Access Web-Server using IP-Address as server name.
#
<VirtualHost 127.0.0.1:80 [::1]:80>
  DocumentRoot "/var/www/ncdirectorync"

  TransferLog /var/log/apache2/localhost-access.log
  ErrorLog    /var/log/apache2/localhost-error.log
  LogLevel warn

  Include /etc/apache2/sites-available/nextcloud-include.conf
</VirtualHost> # http://127.0.0.1

# Access Web-Server using a none defined server name.
#
<VirtualHost *:80 [::]:80>
  ServerName dummy.aolocaldomainao
  ServerAlias dummy
  Header set Access-Control-Allow-Origin "*"
  DocumentRoot "/var/www/html/"
  TransferLog /var/log/apache2/dummy-access.log
  ErrorLog    /var/log/apache2/dummy-error.log
   LogLevel warn
</VirtualHost>

# Local access of Nextcloud Web-Server using internal server/domain names as server name.
#
<VirtualHost *:80 [::]:80>
  ServerName aohostgenericao.aolocaldomainao
  ServerAlias aohostgenericao aohostao.aolocaldomainao aohostao aohost2ao.aolocaldomainao aohost2ao
  DocumentRoot "/var/www/html"
  RewriteEngine On
  RewriteCond %{HTTPS} off
  RewriteRule (.*) https://%{SERVER_NAME}/$1 [R,L] 
</VirtualHost> # http://aohostgenericao*

# Local access of Nextcloud Web-Server using internal server/domain names as server name.
#
<IfDefine !HTTPonly>
  <VirtualHost *:443 [::]:443>
    ServerName aohostgenericao.aolocaldomainao
    ServerAlias aohostgenericao aohostao.aolocaldomainao aohostao aohost2ao.aolocaldomainao aohost2ao
    DocumentRoot "/var/www/ncdirectorync"
  
    TransferLog /var/log/apache2/aohostgenericao-access.log
    ErrorLog    /var/log/apache2/aohostgenericao-error.log
    LogLevel warn
  
    SSLCertificateFile    /etc/ssl/certs/aohostgenericao.aolocaldomainao.crt
    SSLCertificateKeyFile /etc/ssl/private/aohostgenericao.aolocaldomainao.key
    SSLVerifyClient optional
    #SSLVerifyClient require
    Include /etc/apache2/sites-available/ssl-include.conf
  
    Include /etc/apache2/sites-available/nextcloud-include.conf
  </VirtualHost> # https://aohostgenericao*
</IfDefine>

# Access Web-Server using a none defined server name.
#   via HTTP -> redirect to HTTPS
#
<VirtualHost *:80 [::]:80>
  ServerName aodomainao
  ServerAlias www.aodomainao coserverco.aodomainao ooserveroo.aodomainao
  DocumentRoot "/var/www/html"
  RewriteEngine On
  RewriteCond %{HTTPS} off
  RewriteRule (.*) https://%{SERVER_NAME}/$1 [R,L] 
</VirtualHost>

# Access Nextcloud Web-Server using domain `aodomainao` as server name.
#
<IfDefine !HTTPonly>
  <VirtualHost *:443 [::]:443>
    ServerName aodomainao
    DocumentRoot "/var/www/ncdirectorync"
      
    TransferLog /var/log/apache2/www-access.log
    ErrorLog /var/log/apache2/www-error.log
    LogLevel warn

    SSLCertificateFile /etc/ssl/certs/aodomainao.crt
    SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
    SSLVerifyClient optional
    #SSLVerifyClient require
    Include /etc/apache2/sites-available/ssl-include.conf

    Include /etc/apache2/sites-available/nextcloud-include.conf
  </VirtualHost> # https://aodomainao
</IfDefine>

# Access Nextcloud Web-Server using `www` and domain `aodomainao` as server name.
#
<IfDefine !HTTPonly>
  <VirtualHost *:443 [::]:443>
    ServerName www.aodomainao
    DocumentRoot "/var/www/ncdirectorync"
  
    TransferLog /var/log/apache2/www-access.log
    ErrorLog    /var/log/apache2/www-error.log
    LogLevel warn

    SSLCertificateFile /etc/ssl/certs/aodomainao.crt
    SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
    SSLVerifyClient optional
    #SSLVerifyClient require
    Include /etc/apache2/sites-available/ssl-include.conf

    Include /etc/apache2/sites-available/nextcloud-include.conf
  </VirtualHost> # https://aodomainao
</IfDefine>

# Access Collabora DocumentServer/CODE server/service using `coserverco` and domain `aodomainao` as server name.
#
<IfDefine !HTTPonly>
  <VirtualHost *:443 [::]:443>
    ServerName coserverco.aodomainao
    DocumentRoot "/var/www/html"

    TransferLog /var/log/apache2/coserverco-access.log
    ErrorLog    /var/log/apache2/coserverco-error.log
    LogLevel warn

    SSLCertificateFile /etc/ssl/certs/aodomainao.crt
    SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
    #SSLVerifyClient optional
    SSLVerifyClient require
    Include    /etc/apache2/sites-available/ssl-include.conf

    Include /etc/apache2/sites-available/collabora-include.conf
  </VirtualHost> # https://coserverco.aodomainao
</IfDefine>

# Access ONLYOFFICE Docs server/service using `ooserveroo` and domain `aodomainao` as server name.
#
<IfDefine !HTTPonly>
  <VirtualHost *:443 [::]:443>
    ServerName ooserveroo.aodomainao
    DocumentRoot "/var/www/html"

    TransferLog /var/log/apache2/ooserveroo-access.log
    ErrorLog    /var/log/apache2/ooserveroo-error.log
    LogLevel warn

    SSLCertificateFile /etc/ssl/certs/aodomainao.crt
    SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
    #SSLVerifyClient optional
    SSLVerifyClient require
    Include /etc/apache2/sites-available/ssl-include.conf

    Include /etc/apache2/sites-available/onlyoffice-include.conf
  </VirtualHost> # https://ooserveroo.aodomainao
</IfDefine>

# Access cotun server/service using `ctserverct` and domain `aodomainao` as server name.
#
<IfDefine !HTTPonly>
  <VirtualHost *:443 [::]:443>
    ServerName ctserverct.aodomainao
    DocumentRoot "/var/www/html"
  
    TransferLog /var/log/apache2/ctserverct-access.log
    ErrorLog    /var/log/apache2/ctserverct-error.log
    LogLevel warn

    SSLCertificateFile /etc/ssl/certs/aodomainao.crt
    SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
    #SSLVerifyClient optional
    SSLVerifyClient require
    Include /etc/apache2/sites-available/ssl-include.conf

    Include /etc/apache2/sites-available/coturn-include.conf
  </VirtualHost> # https://ctserverct.aodomainao
</IfDefine>
```

=== `/etc/apache2/sites-available/collabora-include.conf` ===

```code
  AllowEncodedSlashes NoDecode
  ProxyPreserveHost On

  # static html, js, images, etc. served from coolwsd
  # browser is the client part of Collabora Online
  ProxyPass           /browser http://127.0.0.1:9980/browser retry=0
  ProxyPassReverse    /browser http://127.0.0.1:9980/browser

  # WOPI discovery URL
  ProxyPass           /hosting/discovery http://127.0.0.1:9980/hosting/discovery retry=0
  ProxyPassReverse    /hosting/discovery http://127.0.0.1:9980/hosting/discovery

  # Capabilities
  ProxyPass           /hosting/capabilities http://127.0.0.1:9980/hosting/capabilities retry=0
  ProxyPassReverse    /hosting/capabilities http://127.0.0.1:9980/hosting/capabilities

  # Main websocket
  ProxyPassMatch      "/cool/(.*)/ws$"      ws://127.0.0.1:9980/cool/$1/ws nocanon

  # Admin Console websocket
  ProxyPass           /cool/adminws wss://127.0.0.1:9980/cool/adminws

  # Download as, Fullscreen presentation and Image upload operations
  ProxyPass           /cool http://127.0.0.1:9980/cool
  ProxyPassReverse    /cool http://127.0.0.1:9980/cool
  # Compatibility with integrations that use the /lool/convert-to endpoint
  ProxyPass           /lool http://127.0.0.1:9980/cool
  ProxyPassReverse    /lool http://127.0.0.1:9980/cool
```

=== `/etc/apache2/sites-available/coturn-include.conf` ===

```code
  ProxyPreserveHost On
  ProxyPass / http://127.0.0.1:3478/
  ProxyPassReverse / http://127.0.0.1:3478/
```

=== `/etc/apache2/sites-available/nextcloud-include.conf` ===

```code
  <IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteRule ^/\.well-known/carddav            /remote.php/dav [R=301,L]
    RewriteRule ^/\.well-known/caldav             /remote.php/dav [R=301,L]
    RewriteRule ^/\.well-known/webfinger          /index.php/.well-known/webfinger [R=301,L]
    RewriteRule ^/\.well-known/nodeinfo           /index.php/.well-known/nodeinfo [R=301,L]
    RewriteRule ^/\.well-known/host-meta          /public.php?service=host-meta [QSA,L]
    RewriteRule ^/\.well-known/host-meta\.json    /public.php?service=host-meta-json [QSA,L]
  </IfModule>

  <IfModule mod_reqtimeout.c>
    RequestReadTimeout body=0
  </IfModule>

  TraceEnable off

  <Files ".ht*">
    Require all denied
  </Files>
  <Directory /var/www/ncdirectorync/>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
  <Directory /var/www/ncdirectorync/data>
    Require all denied
  </Directory>
  <Directory /var/lib/nextcloud/ncdatadirectorync/>
    Require all denied
  </Directory>

  ProxyPass        /push/ws    ws://localhost:7867/ws
  ProxyPass        /push/      http://localhost:7867/
  ProxyPassReverse /push/      http://localhost:7867/
  ProxyPass        /whiteboard http://localhost:3002/ upgrade=websocket

  RewriteEngine On
  RewriteCond %{REQUEST_METHOD} ^TRACK
  RewriteRule .* - [R=405,L]
```

=== `/etc/apache2/sites-available/onlyoffice-include.conf` ===

```code
  <IfModule mod_reqtimeout.c>
    RequestReadTimeout body=0
  </IfModule>

  SetEnvIf Host "^(.*)$" THE_HOST=$1
#  RequestHeader setifempty X-Forwarded-Proto http
  RequestHeader setifempty X-Forwarded-Proto https
  RequestHeader setifempty X-Forwarded-Host %{THE_HOST}e
  ProxyAddHeaders Off

  RewriteEngine on
  RewriteCond %{HTTP:Upgrade} websocket [NC]
  RewriteCond %{HTTP:Connection} upgrade [NC]
  RewriteRule ^/?(.*) "ws://localhost:8080/$1" [P,L]
  ProxyPass / "http://localhost:8080/"
  ProxyPassReverse / "http://localhost:8080/"
```

=== `/etc/apache2/sites-available/ssl-include.conf` ===

```code
    SSLOpenSSLConfCmd DHParameters /etc/ssl/certs/dhparams.aodomainao.pem
  SSLCACertificateFile /etc/ssl/certs/canameca.crt
  SSLCARevocationFile /etc/ssl/crl/canameca.crl
  SSLCARevocationCheck chain

  Protocols h2 h2c http/1.1
  Header always set Strict-Transport-Security "max-age=15552000"
  ServerAdmin mail@example.com

  # SSLVerifyClient to be set in any.conf
  #SSLVerifyClient optional
  #SSLVerifyClient require
  SSLVerifyDepth 3

  SSLEngine on
  SSLCompression off
  SSLOptions +StrictRequire
  SSLProtocol -all +TLSv1.3 +TLSv1.2
  SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
  SSLHonorCipherOrder off
  SSLSessionTickets off
  SSLOpenSSLConfCmd Curves X448:secp521r1:secp384r1:prime256v1
  SSLOpenSSLConfCmd ECDHParameters secp384r1

  SSLUserName SSL_CLIENT_S_DN_CN
```

## Nextcloud installation

Before we start, we want to make sure that Nextcloud will be allowed of using Unix Domain Socket (aka socket) for communication with the PostgreSQL and Redis. That is achieved by "www-data" becoming member of groups "postgres" and "redis".

```console
usermod --append --groups postgres,redis www-data
systemctl restart apache2
```

### Install directory

Get the version 29.0.10 from [nextcloud.com: Index of /server/releases](https://download.nextcloud.com/server/releases/).

```console
cd /tmp
wget https://download.nextcloud.com/server/releases/nextcloud-29.0.10.zip
unzip nextcloud-29.0.10.zip
mv nextcloud/* /var/www/ncdirectorync
rm -Rf nextcloud*
cat << EOF > /var/www/ncdirectorync/robots.txt
User-agent: *
Disallow: /
EOF
chown --recursive --no-dereference www-data:www-data /var/www/ncdirectorync
find /var/www/ncdirectorync/ -type d -exec chmod 750 {} \;
find /var/www/ncdirectorync/ -type f -exec chmod 640 {} \;
```

As an alternative, you can get the Nextcloud Server Version 29.0.10 from GitHub.However, you will not get all apps!

```console
cd /var/www
git clone --depth 1 --shallow-submodules --recurse-submodules --branch v29.0.10 https://github.com/nextcloud/server.git
touch server/config/config.php
mv server/* /var/www/ncdirectorync
chown --recursive --no-dereference www-data:www-data /var/www/ncdirectorync
find /var/www/ncdirectorync/ -type d -exec chmod 750 {} \;
find /var/www/ncdirectorync/ -type f -exec chmod 640 {} \;
```

### Default Apps

There are differences between versions and downloads options. From nextcloud.com (Hub 8 & Hub 9) you get the full set of apps, where as from github.com (Github 29.0.10 & Github 30.0.4) the number if apps is limited.

| apps/... | listed | Name | Github 29.0.10 | Hub 8 | Github 30.0.4 | Hub 9 |
| ---: | :---: | :--- | :---: | :---: | :---: | :---: |
| `activity` | ✔️ | Activity |  | ✔️ |  | ✔️ |
| `admin_audit` | ✔️ | Auditing/Logging | ✔️ | ✔️ | ✔️ | ✔️ |
| `app_api` | ✔️ | AppAPI |  |  |  | ✔️ |
| `bruteforcesettings` | ✔️ | Brute-force settings |  | ✔️ |  | ✔️ |
| `circles` | ✔️ | Teams  |  | ✔️ |  | ✔️ |
| `cloud_federation_api` |  | Cloud Federation API | ✔️ | ✔️ | ✔️ | ✔️ |
| `comments` | ✔️ | Comments | ✔️ | ✔️ | ✔️ | ✔️ |
| `contactsinteraction` | ✔️ | Contacts Interaction | ✔️ | ✔️ | ✔️ | ✔️ |
| `dashboard` | ✔️ | Dashboard | ✔️ | ✔️ | ✔️ | ✔️ |
| `dav` |  | WebDAV | ✔️ | ✔️ | ✔️ | ✔️ |
| `encryption` | ✔️ | Default encryption module | ✔️ | ✔️ | ✔️ | ✔️ |
| `federatedfilesharing` |   | Federated file sharing | ✔️ | ✔️ | ✔️ | ✔️ |
| `federation` | ✔️ | Federation | ✔️ | ✔️ | ✔️ | ✔️ |
| `files` |  | Files | ✔️ | ✔️ | ✔️ | ✔️ |
| `files_downloadlimit` | ✔️ | Files download limit |  | ✔️ |  | ✔️ |
| `files_external` | ✔️ | External storage support | ✔️ | ✔️ | ✔️ | ✔️ |
| `files_pdfviewer` | ✔️ | PDF viewer |  | ✔️ |  | ✔️ |
| `files_reminders` | ✔️ | Files reminders | ✔️ | ✔️ | ✔️ | ✔️ |
| `files_sharing` | ✔️ | Diles sharing | ✔️ | ✔️ | ✔️ | ✔️ |
| `files_trashbin` | ✔️ | Deleted files | ✔️ | ✔️ | ✔️ | ✔️ |
| `files_versions` | ✔️ | Versions | ✔️ | ✔️ | ✔️ | ✔️ |
| `firstrunwizard` | ✔️ | First run wizard |  | ✔️ |  | ✔️ |
| `logreader` | ✔️ | Log Reader |  | ✔️ |  | ✔️ |
| `lookup_server_connector` |  | Lookup Server Connector | ✔️ | ✔️ | ✔️ | ✔️ |
| `nextcloud_announcements` | ✔️ | Nextcloud announcements |  | ✔️ |  | ✔️ |
| `notifications` | ✔️ | Notifications |  | ✔️ |  | ✔️ |
| `oauth2` |  | OAuth 2.0 | ✔️ | ✔️ | ✔️ | ✔️ |
| `password_policy` | ✔️ | Password policy |  | ✔️ |  | ✔️ |
| `photos` | ✔️ | Photos |  | ✔️ |  | ✔️ |
| `privacy` | ✔️ | Privacy |  | ✔️ |  | ✔️ |
| `provisioning_api` |  | Provisioning API | ✔️ | ✔️ | ✔️ | ✔️ |
| `recommendations` | ✔️ | Recommendations |  | ✔️ |  | ✔️ |
| `related_resources` | ✔️ | Related Resources |  | ✔️ |  | ✔️ |
| `serverinfo` | ✔️ | Monitoring |  | ✔️ |  | ✔️ |
| `settings` |  | Nextcloud settings | ✔️ | ✔️ | ✔️ | ✔️ |
| `sharebymail` | ✔️ | Share by mail | ✔️ | ✔️ | ✔️ | ✔️ |
| `support` | ✔️ | Support |  | ✔️ |  | ✔️ |
| `survey_client` | ✔️ | Usage survey |  | ✔️ |  | ✔️ |
| `suspicious_login` | ✔️ | Suspicious Login |  | ✔️ |  | ✔️ |
| `systemtags` | ✔️ | Collaborative tags | ✔️ | ✔️ | ✔️ | ✔️ |
| `testing` | ✔️ | QA testing |  | ✔️ | ✔️ |  |
| `text` | ✔️ | Text |  | ✔️ |  | ✔️ |
| `theming` |  | Theming | ✔️ | ✔️ | ✔️ | ✔️ |
| `twofactor_backupcodes` |  | Two factor backup codes | ✔️ | ✔️ | ✔️ | ✔️ |
| `twofactor_nextcloud_notification` | ✔️ | Two-Factor Authentication via Nextcloud notification |  |  |  | ✔️ |
| `twofactor_totp` | ✔️ | Two-Factor TOTP Provider |  | ✔️ |  | ✔️ |
| `updatenotification` | ✔️ | Update notification | ✔️ | ✔️ | ✔️ | ✔️ |
| `user_ldap` | ✔️ | LDAP user and groupd backend | ✔️ | ✔️ | ✔️ | ✔️ |
| `user_status` | ✔️ | User status | ✔️ | ✔️ | ✔️ | ✔️ |
| `viewer` |  | Simple file viewer with slideshow for media |  | ✔️ |  | ✔️ |
| `weather_status` | ✔️ | Weather status | ✔️ | ✔️ | ✔️ | ✔️ |
| `webhook_listeners` | ✔️ | Nextcloud webhook support |  |  | ✔️ | ✔️ |
| `workflowengine` |  | Nextcloud workflow engine | ✔️ | ✔️ | ✔️ | ✔️ |

### Database

Create the database, by executing the following SQL Statements.

```sql
CREATE USER ncdbonc WITH PASSWORD 'ncdbopasswordnc';
CREATE DATABASE ncdatabasenc OWNER ncdbonc TEMPLATE template0 ENCODING 'UNICODE' LC_COLLATE 'aodblocalesao' LC_CTYPE 'aodblocalesao';
GRANT ALL PRIVILEGES ON DATABASE ncdatabasenc TO ncdbonc;
```

To be done in the following way.

```console
sudo --user=postgres psql --host=ncdbhostnc --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# CREATE USER ncdbonc WITH PASSWORD 'ncdbopasswordnc';
CREATE ROLE
postgres=# CREATE DATABASE ncdatabasenc OWNER ncdbonc TEMPLATE template0 ENCODING 'UNICODE' LC_COLLATE 'aodblocalesao' LC_CTYPE 'aodblocalesao';
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE ncdatabasenc TO ncdbonc;
GRANT
postgres=# \q
```

### Data directory

Create Data-Directory

```console
# mkdir --parents ncdatadirectorync
# chown --recursive www-data:www-data ncdatadirectorync
```

### Configuration

#### Basis

Specify base base configuration.

```console
# sudo -u www-data php /var/www/ncdirectorync/occ maintenance:install --database "pgsql" --database-host "ncdbhostnc" --database-name "ncdatabasenc" --database-user "ncdbonc" --database-pass "ncdbopasswordnc" --admin-user "ncadminnc" --admin-pass "ncadminpasswordnc" --data-dir "ncdatadirectorync"
Nextcloud was successfully installed
# 
```

#### Redis

Enable Redis usage, with Nextcloud specific `dbindex`.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.distributed --value "\\OC\\Memcache\\Redis"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.locking --value "\\OC\\Memcache\\Redis"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis dbindex --value "ncredisdbindexnc"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis host --value "/run/redis/redis-server.sock"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis port --value "0"
```

#### Trusted domains

Specify the trusted_domains.

☝ [Notes: Server Installation > Nextcloud Installation > Configuration > Trusted Domains](./Notes.md#trusted-domains)

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 0 --value "localhost"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 1 --value "127.0.0.1"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 2 --value "::1"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 3 --value "aodomainao"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 4 --value "aorouteripv4addressao"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 4 --value '*'
```

The trusten domains can be check at any time.

```console
# sudo --user=www-data php /var/www/ncdirectorync/occ  config:system:get trusted_domains
localhost
127.0.0.1
::1
aodomainao
aorouteripv4addressao
*
# 
```

#### Others aspects

Logging

```console
sudo --user=www-data php /var/www/ncdirectorync/occ log:manage --timezone 'Europe/Amsterdam'
sudo --user=www-data php /var/www/ncdirectorync/occ log:file --file "/var/log/nextcloud/nextcloud.log"
```

Further adjustments

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.local --value "\OC\Memcache\APCu"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set filelocking.enabled --type=boolean --value true
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set maintenance_window_start --value "1"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set default_phone_region --value "aophoneregionao"
sudo --user=www-data php /var/www/ncdirectorync/occ config:app:set workflowengine logfile --value=/var/log/nextcloud/flow.log
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set simpleSignUpLink.shown --type=boolean --value=false
```

#### The `config.php` file

At the end we will have a `config.php` looking like the following.

=== `/var/www/ncdirectorync/config/config.php` ===

```code
<?php
$CONFIG = array (
  'passwordsalt' => '<password salt>',
  'secret' => '<its secret>',
  'trusted_domains' => 
  array (
    0 => 'localhost',
    1 => '127.0.0.1',
    2 => '::1',
    3 => 'aodomainao',
    4 => 'aorouteripv4addressao',
    5 => '192.168.1.*',
    6 => '*',
  ),
  'datadirectory' => 'ncdatadirectorync',
  'dbtype' => 'pgsql',
  'version' => '29.0.10.1',
  'overwrite.cli.url' => 'http://localhost',
  'dbname' => 'ncdatabasenc',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'dbuser' => 'ncdbonc',
  'dbpassword' => 'ncdbopasswordnc',
  'installed' => true,
  'instanceid' => '<Instance ID>>',
  'memcache.distributed' => '\\OC\\Memcache\\Redis',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'redis' => 
  array (
    'dbindex' => 'ncredisdbindexnc',
    'host' => '/run/redis/redis-server.sock',
    'port' => '0',
  ),
  'logtimezone' => 'aotimezoneao',
  'logfile' => '/var/log/nextcloud/nextcloud.log',
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'filelocking.enabled' => true,
  'maintenance_window_start' => '1',
  'default_phone_region' => 'aophoneregionao',
  'simpleSignUpLink.shown' => false,
);
```

#### Establish Background Job

Add the "Background jobs" to the cron jobs of user `wwww-data`.

```console
crontab -u www-data -e
```

Add the two lines

```code
# --- START OF ACNO SPECIFIC DEFINITIONS --- #
*/5  *  *  *  * php -f /var/www/ncdirectorync/cron.php --define apc.enable_cli=1
```

Get rid of unwanted Nextcloud-Apps.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:remove testing
sudo --user=www-data php /var/www/ncdirectorync/occ app:disable weather_status
```

#### Install Nextcloud-Apps

Install the usual Nextcloud-Apps.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install calendar
sudo --user=www-data php /var/www/ncdirectorync/occ app:install camerarawpreviews
sudo --user=www-data php /var/www/ncdirectorync/occ app:install contacts
sudo --user=www-data php /var/www/ncdirectorync/occ app:install groupfolders
sudo --user=www-data php /var/www/ncdirectorync/occ app:install memories
sudo --user=www-data php /var/www/ncdirectorync/occ app:install notes
sudo --user=www-data php /var/www/ncdirectorync/occ app:install notify_push
sudo --user=www-data php /var/www/ncdirectorync/occ app:install previewgenerator
sudo --user=www-data php /var/www/ncdirectorync/occ app:install spreed
sudo --user=www-data php /var/www/ncdirectorync/occ app:install tasks
sudo --user=www-data php /var/www/ncdirectorync/occ app:install user_saml
sudo --user=www-data php /var/www/ncdirectorync/occ memories:places-setup
```

At any point in time, you are able to update the Nextcloud-Apps from command line, using command `sudo --user=www-data php /var/www/ncdirectorync/occ app:update --all`. Right after the installation you should get the message `All apps are up-to-date or no updates could be found`.

##### Apps for Collabora Online/CODE

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install richdocuments
```

```console
# sudo --user=www-data php /var/www/ncdirectorync/occ richdocuments:setup --callback-url "http://localhost"
✓ Set callback url to http://localhost
Checking configuration
🛈 Configured WOPI URL: https://coserverco.aohostao.aodmzdomainao
🛈 Configured public WOPI URL: https://coserverco.aohostao.aodmzdomainao
🛈 Configured callback URL: http://localhost

✓ Fetched /hosting/discovery endpoint
✓ Valid mimetype response
✓ Valid capabilities entry
✓ Fetched /hosting/capabilities endpoint
✓ Detected WOPI server: Collabora Online Development Edition 25.04.5.2

Collabora URL (used for Nextcloud to contact the Collabora server):
  https://coserverco.aohostao.aodmzdomainao
Collabora public URL (used in the browser to open Collabora):
  https://coserverco.aohostao.aodmzdomainao
Callback URL (used by Collabora to connect back to Nextcloud):
  http://localhost
```

```
  #SSLVerifyClient optional
  SSLVerifyClient require
  SSLUserName SSL_CLIENT_S_DN_CN
```



##### Apps for ONLYOFFICE Docs

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install onlyoffice
```

#### App-Install Banner

Are you annoyed as well, when each web-site ask you to install their app. Even worst if it is your own. For iOS this banner can be de-activated using SQL Statement `INSERT INTO oc_appconfig(appid,configkey,configvalue) VALUES ('theming', 'iTunesAppId', '');`.

```console
# sudo --user=postgres psql --host=localhost --dbname=ncdatabasenc --username=ncdbonc --password
Password: 
psql (15.8 (Debian 15.8-0+deb12u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

ncdatabasenc=> INSERT INTO oc_appconfig(appid,configkey,configvalue) VALUES ('theming', 'iTunesAppId', '');
INSERT 0 1
ncdatabasenc=> SELECT * FROM oc_appconfig WHERE appid = 'theming';
  appid  |     configkey     | configvalue | type | lazy 
---------+-------------------+-------------+------+------
 theming | installed_version | 2.4.0       |    2 |    0
 theming | types             | logging     |    2 |    0
 theming | enabled           | yes         |    2 |    0
 theming | iTunesAppId       |             |    2 |    0
(4 rows)
```

#### Email-Accounts

As `ncadminnc` login and specify the parameters with regards to Email-Accounts.

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `http://aohostao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Personal settings**" from the menu
				- ✏️ enter `ncadmin@addressnc` in field "**Email**"
			- ⬆ select "Basic settings" in the left panel
				- ✏️✔️ fill out the "**Email server**" settings for `aosystem@addressao`
				- ✔️ click on "**Save**"
        - click on "**Send email**"

#### Client-Certificate authentication

Instead of typing in user/password, the authentication can be established based on the Client-Certificate. Which got deployed anyway and is secure enough to authenticate a user and its device.

In file `ssl-include.conf` the following line must exist, otherwise the environment variable `REMOTE_USER` cannot be used for identifying the right user account. The reference to `SSL_CLIENT_S_DN_CN` means, the "Common Name" of the Client-Certificate, is the User-ID for Nextcloud.

```code
  SSLUserName SSL_CLIENT_S_DN_CN
```

As Administrator, using Browser A (e.g. Google Chrome).

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `http://aohostao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Adiministration settings**" from the menu
				- ⬆ select "SSO & SAML authentication" in the left panel
          - ✏️⬆ click on "**Use environment variable**"
            - ✏️🔙 enter the password of `ncadminnc` and click on "**Confirm**" (Dialog "Confirm your password")
          - ✏️ enter "REMOTE_USER" in field under "**General** > **Attribute to map the UID to.**"
					- ✏️ ~~disable~~ "**Only allow authentication if an account exists on some other backend (e.g. LDAP).**"

Stay logged in as Administrator, until the test is finished and everything setup properly.

Open another Browser B (e.g. Mozilla Firefox).
Prepare the Browser for using Client-Certificate Authentication, by instzalling the Client-Certificate of `ncadminnc`.

- ⬆ open browser menu `≡` and select "**Settings**"
  - enter "ertif" in the search field
  - ⬆ click on "***View Certificates..:**"
    - navigate to tab "**Your Certificates**"
    - ⬆ click on "**Import...**"
      - navigate to directory `home/wssysuserws/SSL.caname/Clients` (Dialog "Certificate File import")
      - ✏️select file `aonameao_ncadminnc.pfx` and click on "**Open**"
      - ✏️🔙 enter the Import Password as specified in [Client Certificate](#client-certificate) (Dialog "Password Required - Firefox")
    - navigate to tab "**Authorities**"
    - select your own CA-Certificate "caorgca > cacommonnameca"
    - ⬆ click on "**Edit Trust...**"
      - ✏️✔️ enable option "**This certificate can identify websites.**" (Dialog "Edit CA certificate trust settings.)
      - ✔️🔙 click on "OK"
    - 🔙 click on "OK"
    - 🔙 close "**Settings**" tab
- ⬆ open another "**New private window**", in the web browser
	- ⬆ open URL `https://aohostao`
		- ✏️ select the Client-Certificate of the **user**

You got automatically logged in.

The login process was working as expected? Then you are save to close the session in Web Browser A.

##### Revert change

At anytime you can disable the App and everyone has to login like before. However, those users created automatically via SSO are gone.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:disable user_saml
```

### Place in DMZ

- shutdown the machine
- put it at its place ... in the DMZ
- start the machine

## Bring aonameao into the Internet

### DSL-Router Adjustments

Once your have tested the Web-Server in your local network, it is time to make it available for the clients on the Internet.

#### Reboot

As part of your setup/test, you should reboot the Router to ensure that the configuration got stored properly.

The location of the configuration depends on the Router model. For mine, I found the configuration at "Settings > Problem handling > Restart".

click on "Restart"

#### Configuration - Intranet Communication

##### Email SMTP-Server

Some router do block SMTP communication in general and allow communication only to those maintain in a list of E-Mail servers.

The location of the configuration depends on the Router model. For mine, I found the configuration at "Internet > E-mail abuse detection".

Add those SMTP-Server, you are certain about being used by the users, but not in the defined list of servers.

##### Port Forwarding

The location of the configuration depends on the Router model. For mine, I found the configuration at "Internet > Port activiation > Port redirections and port forwardings".

Three rules have to be created. One for the Web-Server and the other two for the TURN-Server.

Name of the redirection: Web-Server 443 only
Applies to the following device: aohostao
Use template: Web-Server
Ports to redirect:
    TCP: 443 🞂 443

Name of the redirection: TURN-Server
Applies to the following device: aohostao
Use template: No template
Ports to redirect:
    TCP: 5349 🞂 5349
    UCP: 5349 🞂 5349

Name of the redirection: TURN-Server ports
Applies to the following device: aohostao
Use template: No template
Ports to redirect:
    TCP: 10000-20000 🞂 10000-20000
    UCP: 10000-20000 🞂 10000-20000

## Internet Nextcloud

As Administrator, using Browser B (e.g. Firefox), which has the Certificates already installed.

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ✏️ select the Client-Certificate of the **user**

Hurray, your Nextcloud instance is accessible via Internet and you are able to login as Nextcloud Administrator.

Now we can continue with those configurations, which were not possible with machine `aohostao` accessible only within the local network.

### Configuration - Internet Communication

#### Client Push (notify_push)

With Client Push installed as Nextcloud App, the Client Push service is neither installed, enabled nor configured. To get the service setup properly the following steps are required.

1. Ensure the `notify_push` binary is executable
2. Create service file
3. Enable service
4. Configure service

```console
chmod 770 /var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push
cat << EOF > /etc/systemd/system/notify_push.service
[Unit]
Description = Push daemon for Nextcloud clients

[Service]
Environment=PORT=7867
Environment=NEXTCLOUD_URL=https://aodomainao
ExecStart=/var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push /var/www/ncdirectorync/config/config.php
Type=notify
User=www-data

[Install]
WantedBy = multi-user.target
EOF
systemctl --now enable notify_push
systemctl start notify_push
```

=== `/etc/systemd/system/notify_push.service` ===

```code
[Unit]
Description = Push daemon for Nextcloud clients

[Service]
Environment=PORT=7867
Environment=NEXTCLOUD_URL=https://aodomainao
ExecStart=/var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push /var/www/ncdirectorync/config/config.php
Type=notify
User=www-data

[Install]
WantedBy = multi-user.target
```

Setup the configuration.

```console
# sudo --user=www-data php /var/www/ncdirectorync/occ notify_push:setup https://aodomainao/push
✔️ redis is configured
✔️ push server is receiving redis messages
✔️ push server can load mount info from database
✔️ push server can connect to the Nextcloud server
✔️ push server is a trusted proxy
✔️ push server is running the same version as the app
  configuration saved
# 
```

#### Configuration for Collabora Online/CODE (richdocuments)

As Administrator

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Administration settings**" from the menu
				- ⬆ select "**Office**" in the left panel
					- expand the "**Adavanced server settings**"
					- ✏️✔️ enter the configuration for the "Nextcloud Office" app
          - ✏️✔️ select "**Use your own server**"
          - ✏️✔️ type `https://coserverco.aodomainao`
					- ✔️ click on "**Save**"

#### Configuration for ONLYOFFICE Docs (onlyoffice)

First, identify the `<Secret key>` from the `local.json` file (see below). It is identical for all three nodes

- services > CoAuthoring > secret > inbox > string
- services > CoAuthoring > secret > outbox > string
- services > CoAuthoring > secret > session > string

As Administrator

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Administration settings**" from the menu
				- ⬆ select "**ONYLOFFICE**" in the left panel
					- expand the "**Adavanced server settings**"
					- ✏️✔️ enter the configuration for the "ONLYOFFICE" app
					- ✔️ click on "**Save**"

##### Configuration of "ONLYOFFICE" app:

  ONLYOFFICE Docs address
	  https://ooserveroo.aodomainao

  Secret key (leave blank to disable):
	  <Secret key>

  ONLYOFFICE Docs address for internal requests from the server
  	http://localhost:8080/

  Server address for internal requests from ONLYOFFICE Docs
  	http://localhost/

=== `/etc/onlyoffice/documentserver/local.json` ===

```code
{
  "services": {
    "CoAuthoring": {
      "sql": {
        "type": "postgres",
        "dbHost": "localhost",
        "dbPort": "5432",
        "dbName": "oodatabaseoo",
        "dbUser": "oodbooo",
        "dbPass": "ooodbopasswordoo"
      },
      "token": {
        "enable": {
          "request": {
            "inbox": true,
            "outbox": true
          },
          "browser": true
        },
        "inbox": {
          "header": "Authorization"
        },
        "outbox": {
          "header": "Authorization"
        }
      },
      "secret": {
        "inbox": {
          "string": "<Secret key>"
        },
        "outbox": {
          "string": "<Secret key>"
        },
        "session": {
          "string": "<Secret key>"
        }
      }
:
```

#### Talk (spreed)

As Administrator

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Administration settings**" from the menu
				- ⬆ select "**Talk**" in the left panel
          - scroll down to "**STUN servers**" section
					- ✏️ enter `ctserverct.aodomainao:5249` in field "**stun**"
          - scroll down to "**TURN servers**" section
					- ✏️ select "turns: only"
          - ✏️ enter `ctserverct.aodomainao:5249` in field "**TURN server URL**"
          - ✏️ enter `ctsecretct` in field "**TURN server secret**"
					- ✏️ select "UDP and TCP"
          - click on "**Add a new TURN server**"
					- ✏️ select "turns: only"
          - ✏️ enter `ctserverct.aodomainao:443` in field "**TURN server URL**"
          - ✏️ enter `ctsecretct` in field "**TURN server secret**"
					- ✏️ select "TCP only"

# Add-On

## Administrator (Backup)

Highly recommended is to have a second administrator, one which to be used in the worst case of loosing access to the primary administrator. The information of the backup administrator should be stored at a save place.

The relevant data to login as the Administrator (Backup):

- Account Name
- Account Password
- Client Certificate

Steps to create the backup administrator.

- ⬆ open URL `https://aodomainao`
	- ⬆ login with the administrator account `ncadminnc`
		- ⬆ select "**Accounts**" from the menu
			- ⬆ click on "**➕ New account**" in the left panel
        - ✏️✓ enter `ncadmin2nc` in field "**Account name (required)**"
        - ✏️✓ enter `Administrator (Backup)` in field "**Display name**"
        - ✏️✓ enter `ncadmin2passwordnc` in fields "**Password (required)**"
        - ✏️✓ enter `ncadmin2@addressnc` in fields "**Email (required)**"
        - ✏️✓ select `admin` as "**Email Member of the following groups**"
        - keep "**Admin of the following groups**" as it is
        - keep "**Quota**" as it is
        - keep "**Manager**" as it is
			- ⬆ click on "**Add new account**"

Once done, verify that the newly create backup administrator is working.

1. create the Client Certificate for this account
2. deploy the Client Certificate on Workstation B
3. on Workstation B, login as "Administrator (Backup)"
4. disable the regular "Administrator" account
5. on `wshostws`, try to login as the regular "Administrator" (this has to be impossible)
6. reactivate/enable the regular "Administrator" account
7. on `wshostws`, login as the regular "Administrator" (this has to work)
8. uninstall/remove the Client Certificate from Workstation B
9. make a backup of the relevant data, to access the "Administrator (Backup)"

## Anti-Virus

The countermeasures, on destructive and/or manipulative program-code inside files, is curcial to prevent this kind of code to be spread via your ACNO instance. The Nextcloud App "Antivirus for files" is cabable of utilizing state-of-the-art antivirus engines.

The installation is basically 

1. install the antivirus engines
2. configure the engine
3. test the antivirus engine
3. install the Nextcloud App
4. configure the App

### ClamAV

#### Installation

- install the components
- manually update the database (while update service is stopped)

- restart ClamAV
- manually scan complete ACNO machine

Install ClamAV and directly manually update its database.

```console
apt install -y clamav clamav-freshclam clamav-daemon
systemctl stop clamav-freshclam
freshclam
```

#### Configuration

Create a backup of the default ClamAV configuration file and adjust the following parameters.

  - scan up to 50 MB file size
  - crawl into up to 25 sub directories

```console
cp /etc/clamav/clamd.conf /etc/clamav/clamd.conf.ORG
sed --in-place --expression="s/MaxFileSize.*/MaxFileSize 50M/" --expression="s/MaxDirectoryRecursion.*/MaxDirectoryRecursion 25/" --expression="s/PCREMaxFileSize.*/PCREMaxFileSize 50M/"  --expression="s/StreamMaxLength.*/StreamMaxLength 50M/" /etc/clamav/clamd.conf
```

Restart the ClamAV services and start manually the virus scan on the whole ACNO machine.

```console
systemctl restart clamav-daemon clamav-freshclam
clamscan --recursive /
```

We can check the version of the database, by looking at the date/time the database got build/signed ...

```console
# sigtool --info /var/lib/clamav/daily.cld 
File: /var/lib/clamav/daily.cld
Build time: <timestamp>>
:
Builder: raynman
Verification OK.
```

#### Test

No usage of an antivirus software, without having tested it.

1. download a know virus into the temporary directory
2. scan the temporary directory

There are lot of databases out there, providing samples of any kind of malware. For example [EICAR](https://www.eicar.org/) and its [Anti Malware Testfile](https://www.eicar.org/download-anti-malware-testfile/). One test file has the URL: https://www.eicar.org/download/eicar_com-zip/?wpdmdl=8847&refresh=67d51a23391221742019107

```console
cd /tmp
wget --output-document=aoantivirusfilesampleao aoantivirusurlsampleao
clamscan --no-summary --suppress-ok-results --recursive --infected aoantivirusfilesampleao
/tmp/aoantivirusfilesampleao: <details¹> FOUND
```

¹ The `<details>` represents the name of the detected maleware.

#### Nextcloud App

Install "Antivirus for files" App

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install files_antivirus
```

Configure "Antivirus for files" App

As Administrator

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Administration settings**" from the menu
				- ⬆ select "**Security**" in the left panel
          - scroll down to section "**Antivirus for Files**"
					- ✏️✔️ select "ClamAV Daemon (socket) as "**Mode**"
          - ✏️✔️ enter "52428800" as "**Stream length**" in bytes
          - ✏️✔️ select "Delete file" as action on "**background scan**" identifying an "**infected file**"
					- ✔️ click on "**Save**"

=== `/etc/clamav/clamd.conf` ===

```code
:
MaxDirectoryRecursion 25
:
MaxFileSize 50M
:
PCREMaxFileSize 50M
:
StreamMaxLength 50M
:
```

## Backup (DRAFT - work in progress)

The basis for the backup are multiple aspects of different kinds.

1. the ACNO Scripts
2. information on attributes and values
3. a resource for the backup
4. cronjob in place

### Backup - Local

### Backup - Remote

For the ssh connection the user `aounitbsysuserao` will be used, which must exist on the ACNO server and the Remote Backup Unit.

```console
usermod --append --groups backup aosysuserao
cat << EOF >> /etc/rsyncd.conf
use chroot = yes
max connections = 4
syslog facility = local5

[Backup]
  path = aomountdirbackupao/remote
  comment = rsync Backup
  uid = root
  gid = backup
  read only = true
EOF
systemctl --now enable rsync
```

=== `/etc/rsyncd.conf` ===

```code
use chroot = yes
max connections = 4
syslog facility = local5

[Backup]
  path = aomountdirbackupao/remote
  comment = rsync Backup
  uid = root
  gid = backup
  read only = true
```

### Backup - WebDAV

### Scripts

Section [Scripts (Automation)](#scripts-automation) describes how to get the scripts (and its data) installed.

### USB Drive

```console
touch aomountdirbackupao/ACNO-Backup
chown root:backup aomountdirbackupao/ACNO-Backup
```

### Backup devices
UUID=625665d9-3464-413c-b055-52610338725f /mnt/backup         ext4    noauto,user 0       2



### Backup (Remote)

From a higher level, the prerequisits are similar to those for [Backup (Local/WebDAV)](#backup-remote).

#### Scripts

Section [Scripts (Automation)](#scripts-automation) describes how to get the scripts (and its data) installed.

#### Variables
variables.remote.sh (variables.remote.ORG.sh)

## Certificates

### Let's Encrypt

For the initial creation of the Certificates, the Apache Web-Server process has to be started with the additional parameter `-DHTTPonly`. The parameter can be set in file `/etc/apache2/envvars`. Based on the definition in `any.conf` this prevents the HTTPS part to be started.

```console
sed -i -e "s/^#export APACHE_ARGUMENTS=''/export APACHE_ARGUMENTS='-DHTTPonly'/" /etc/apache2/envvars
systemctl reload apache2
```

=== `/etc/apache2/envvars` ===

```code
:

## If you would like to pass arguments to the Web-Server, add them below
## to the APACHE_ARGUMENTS environment.
export APACHE_ARGUMENTS='-DWithHTTPS'

:
```

Create the Certificates/Keys for the three Web-Site.

Note: Later I figured out that the key type and length does not match the original configuration. That's why I added the following step to change to RSA 4098 bits. Maybe at the creation of the certificates, using `--key-type rsa --rsa-key-size 4096` the keys get directly created in the right format.

```console
certbot --agree-tos --email aoowner@addressao --domain aodomainao
certbot --agree-tos --email aoowner@addressao --domain www.aodomainao
certbot --agree-tos --email aoowner@addressao --domain coserverco.aodomainao
certbot --agree-tos --email aoowner@addressao --domain ooserveroo.aodomainao
certbot --agree-tos --email aoowner@addressao --domain ctserverct.aodomainao
```

Change from ECDSA to RSA 4096 bit

Once created, you can change to the 4096 bit length with RSA.
During the process, you have to answer the question "(U)pdate key type/(K)eep existing key type:" with "U".

```console
certbot --agree-tos --key-type rsa --rsa-key-size 4096 --email aoowner@addressao --domain aodomainao
certbot --agree-tos --key-type rsa --rsa-key-size 4096 --email aoowner@addressao --domain wwww.aodomainao
certbot --agree-to --key-type rsa --rsa-key-size 4096 --email aoowner@addressao --domain coserverco.aodomainao
certbot --agree-to --key-type rsa --rsa-key-size 4096 --email aoowner@addressao --domain ooserveroo.aodomainao
certbot --agree-tos --key-type rsa --rsa-key-size 4096 --email aoowner@addressao --domain ctserverct.aodomainao
```

Remove the start parameter `-DWithHTTPS`.

```console
sed -i -e "s/^export APACHE_ARGUMENTS='-DHTTPonly'/#export APACHE_ARGUMENTS=''/" /etc/apache2/envvars
systemctl reload apache2
```

=== `/etc/apache2/envvars` ===

```code
:

## If you would like to pass arguments to the Web-Server, add them below
## to the APACHE_ARGUMENTS environment.
#export APACHE_ARGUMENTS=''

:
```

Next open the `any.conf` file in an editor and adjust the references, of the `aodomainao`, `ooserveroo.aodomainao` and `ctserverct.aodomainao` virtual servers definitions, to the Let's Encrypt certificate and key files. Use the output from the creation above to identify the correct path/file combinations.

```code
  SSLCertificateFile /etc/ssl/certs/...crt
  SSLCertificateKeyFile /etc/ssl/private/...pem
```

```code
  SSLCertificateFile    /etc/letsencrypt/live/.../fullchain.pem
  SSLCertificateKeyFile /etc/letsencrypt/live/.../privkey.pem
```

### Intermediate Certificate Authority

In case of using router with pfSense/OPNsense, it is useful to have CA certificate/key installed on it. That will make it much easier to manage Certificates on the router for its Web-Console and OpenVPN Clients.

The router will become an Intermediate CA, to manage only those certificates/keys used in the context of the router.

Based the existing `SSL.canameca` directory, the certificate/key for the Intermediate CA can be created in the following way.

```console
cd SSL.canameca
mkdir CA.intermediate
openssl genrsa -utf8 -aes256 -out ./CA.intermediate/SSL.canameca.intermediate.key 4096
openssl req -utf8 -config config/ica.cnf -new  -passin "pass:caicapassphraseca" -key ./CA.intermediate/SSL.canameca.intermediate.key -out $1/tmp/client.csr
openssl ca -batch -utf8 -config config/ca.cnf -extensions v3_intermediate_ca -days 1095 -passin "pass:caicapassphraseca" -out ./CA.intermediate/SSL.canameca.intermediate.crt -infiles $1/tmp/client.csr
openssl rsa -passin "pass:caicapassphraseca" -in ./CA.intermediate/SSL.canameca.intermediate.key -out ./CA.intermediate/SSL.canameca.intermediate.unencrypted.key
```

Note: The last command creates an addition `.key` file with the key beeing not encrypted.

## Disk (SDD/HDD)

### Encryption

#TODO

### HDD - idle => power off

In case of a real HDD, you might to want the backup device to go into sleep mode ... once the backup is done. Or does it go into power off, once un-mounted or unused for several minutes?

```console
apt install --yes hdparm
```

### LUKS - Disk Encryption

#### Automatic Unlock

Prerequisite: During the systems setup the `/` (root) partion must have been installed in an encrypted volume/partition.

To avoid entering the Passphrase every time the system is booted, the USB-Stick can be prepared in such a way it unlocks with a key file. A key file, stored on the same USB-Stick used for booting the system.

This instruction is using `sda1` as the partition for the encrypted System Root Partition(`/`).
Verify the chosen device is the right one. Looks a the output of `lsblk` and focus on the lines with the `/` at the end.

```console
# lsblk
NAME           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda              8:0    0 465.8G  0 disk  
└─sda1           8:1    0 465.8G  0 part  
  └─sda1_crypt 254:0    0 465.7G  0 crypt /
:
```

First backup the files, which are going to be modified.

```console
cp /etc/cryptsetup-initramfs/conf-hook /etc/cryptsetup-initramfs/conf-hook.ORG
cp /etc/crypttab /etc/crypttab.ORG
cp /etc/initramfs-tools/initramfs.conf /etc/initramfs-tools/initramfs.conf.ORG
```

Second .. make sure the `/boot` partition is mounted.

```console
mount /boot
```

Create the Key File `primary.key` (for the primary disk), which is going to be used as an alternative to the Passphrase.

```console
mkdir --mode=700 /boot/keys
dd if=/dev/urandom of=/boot/keys/primary.key bs=512 count=16
chmod 700 /boot/keys/primary.key
```

Define the Key File `primary.key` as additional key for the System Root Partition(`/`).

```console
cryptsetup luksAddKey /dev/sda1 /boot/keys/primary.key
```

Modify `crypttab`, by specify the Key File for the partition.

```console
echo "About to change 'crypttab', replacing 'none' by '/boot/keys/primary.key'."
if [ 1 -eq `wc -l /etc/crypttab | wc -l` ]; then
  sed --in-place --expression="s/ none luks,/ \/boot\/keys\/primary.key luks,/" /etc/crypttab
  echo "DONE"
else
  echo "There is more than one line in 'crypttab'. The change has to be done by hand."
  echo "The given example of the 'original' and 'modified' version will help you."
fi
```

Some configuration are required to ensure the initrd gets always created with the proper adjustments to unlock the encrypted partition.

Note: *The script will take over only the line of `crypttab`, which is related to the System Root Partition (`/`). This will ensure a.) no other encrypted partition will be decrypted during boot and b.) no other Key File will be stored on the USB-Stick.*

```console
sed --in-place --expression='s/^#KEYFILE_PATTERN=$/KEYFILE_PATTERN="\/boot\/keys\/*.key"/' /etc/cryptsetup-initramfs/conf-hook
cat << EOF >> /etc/initramfs-tools/initramfs.conf

UMASK=0077
EOF
cat << EOF > /etc/initramfs-tools/hooks/publish_crypttab.sh
#!/bin/sh
grep --regexp="^sda1_crypt UUID=" /etc/crypttab > "${DESTDIR}/cryptroot/crypttab"
exit 0
EOF
```

Finally it is time to update the initrd of the existing boot images.

```
update-initramfs -u -k all
```

Note: *Watch out for error/warning!*

```console
cryptsetup luksHeaderBackup /dev/sda1 --header-backup-file /boot/$HOSTNAME-sda1-luks-header-$(date +'%y%m%d').img
```



### S.M.A.R.T. / Error Detection

Usually every disk/flash drive support this feature. The tool set for this feature can be installed in the following way.

```console
apt install --yes smartmontools
```

On one hand you have a command line interface.

```console
# smartctl -i /dev/sda
SMART support is: Disabled
# smartctl -s on /dev/sda
# smartctl -a /dev/sda | grep Temp | cut -d" " -f 2,37
```

On the other hand, you have an additional `smartd` monitoring the devices. A service capable of send email notificaitons.

#### Email notification

Very handy feature, you can make use of. The prerequisits are

- Add-On: Email for root
- Add-On: Email Encryption (recommended)
- Add-On: Scripts

First make a backup of the existing script.

```console
cp /etc/smartd.conf /etc/smartd.conf.ORG
mkdir --mode=755 /etc/smartmontools/run.d.ORG
mv /etc/smartmontools/run.d/10mail /etc/smartmontools/run.d.ORG
```

Now create the new version of `10mail`.

=== `/etc/smartmontools/run.d/10mail` ===

```sh
#!/bin/bash

# directory of scripts
#
DIR=/root/scripts/acno

# set the variables / script in same directory
#
source $DIR/variables.sh

LOGTMP=$(mktemp /tmp/10mail.XXXXXX)


echo "$SMARTD_TFIRST - $SMARTD_FAILTYPE"    >>$LOGTMP
echo ""                                     >>$LOGTMP
echo "$3"                                   >>$LOGTMP
echo ""                                     >>$LOGTMP
cat "$1"                                    >>$LOGTMP

# something with the disks ... high Prio
#
source $DIR/modules/send_email.sh $LOGTMP "$SUBJECT_ADMIN_PRIO1"

## Clean up files
rm --force $LOGTMP $LOGTMP.asc
```

##### Test - Send Test-Alert

The tool comes with a test feature, with gets enabled/triggered by `-M test` as the first parameter for `DEVICESCAN`.

The following runs the test.

1. enable Test-Feature
2. restart service
3. check your Inbox
4. disable Test-Feature
5. restart server

```console
sed --in-place --expression="s/^DEVICESCAN -\([^M]\)/DEVICESCAN -M test -\1/" /etc/smartd.conf
systemctl restart smartd
# email notification should have been send out ...
sed --in-place --expression="s/^DEVICESCAN -M test /DEVICESCAN /" /etc/smartd.conf
systemctl restart smartd
```

## Email for root

It might be usefully, for later usage, to send emails as `root` user. Like sending a status report to administrator.

### Prepare - Sending

First install the software packages, required to send out emails. During the process, answer the question `Enable AppArmor support?` with `Yes`.
To configure the sending of emails, you have to define the `msmtprc` and `mail.rc` files.
Make sure that the configuration of the STMP server and email account fits to your setup.

```console
apt install --yes msmtp msmtp-mta mailutils
touch /etc/msmtprc
chmod 600 /etc/msmtprc
```

Later, the command `mail ncadmin@addressnc < AnyFile` can be used to send any file to the administrator or other users.

=== `/etc/msmtprc` ===

```code
# Default values for all accounts.
defaults
auth on
tls on
account root
host <SMTP server address of email provider>
port 587
from aosystem@addressao
user aosystem@addressao
password <password for email account aosystem@addressao>
# Set a default account
account default : root
```

=== `/etc/mail.rc` ===

```code
set sendmail="/usr/bin/msmtp -t"
```

### Test - Sending

```console
echo "From Russian with Love" | mail --subject="How are you?" ncadmin@addressnc
```

## Email Encryption

In case you have enabled the Add-On "Email for root", might consider also to encrypt the email message.

### Prerequisite

On another machine, which is neither `aohostao` nor `wshostws`, you have to create the keys and exported them.

1. Private Key `PRIVATE_SUBKEYS_aosystem@addressao.asc` for Email-Address `aosystem@addressao` - used by the `root`account on machine `aohostao` for sending encrypted log information to the Nextcloud Administrator.
2. Public Key `PUBLIC_ncadmin@addressnc.asc ` for Email-Address `ncadmin@addressnc` - used by the Nextcloud Administrator account `ncadminnc` to read the encryption log information.

Both keys have to be transfered - securely - onto machine `aohostao`.

If GnuPG is not already installed ...

```console
apt install --yes gpg
```

### Import Keys

1. prepare `~/.gnupg` directory
2. import the Private Key for email account `aosystem@addressao`
2. import the Public Key for email account `ncadmin@addressnc`
3. start edit the Public Key of `ncadmin@addressnc`
3. ultimately trust the imported Public Key of `ncadmin@addressnc`

```console
# mkdir --mode 700 ~/.gnupg
# gpg --import PRIVATE_SUBKEYS_aosystem@addressao.asc
:
# gpg --import PUBLIC_ncadmin@addressnc.asc 
:
# gpg --edit-key 'ncadmin@addressnc'
gpg > trust

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5
```

### Remove Passphrase

Remove the passphrase for the `aosystem@addressao` private key.

1. identify the id of the secret key
2. start edit the id
3. change the passphrase to "" (literaly nothing), using gpg-command `passwd`

```console
 # gpg --list-secret-keys --keyid-format=long
 /root/.gnupg/pubring.kbx
------------------------
              
sec#  ed25519/8E084C9895463551 2024-12-25 [C] [expires: 2029-12-24]
      92E501428E084C98954635514DEAD4D992E50142
:
# gpg --edit-key 8E084C9895463551
gpg (GnuPG) 2.2.40; Copyright (C) 2022 g10 Code GmbH
:

gpg> passwd
```

### Test

```console
echo "From Russian with Love" > qqq.txt
gpg --recipient ncadmin@addressnc --sign --armor --output qqq.txt.asc --encrypt qqq.txt
mail --subject="How are you?" ncadmin@addressnc
```

## GeoBlocker

### MaxMind Database

```console
apt install --yes geoipupdate
cd /var/lib/GeoIP
wget https://github.com/maxmind/GeoIP2-php/releases/download/v3.1.0/geoip2.phar
```

Open the configuration file `/etc/GeoIP.conf` and modify the following parametera.

- `AccountID`
- `LicenseKey`
- `EditionIDs` set to ` GeoLite2-Country` (other databases are not required)

```console
sed -i -e "s/^# \(AccountID\) YOUR_ACCOUNT_ID_HERE/\1 aomaxmindaccountao/" -e "s/^# \(LicenseKey\) YOUR_LICENSE_KEY_HERE/\1 aomaxmindlicenseao/" -e "s/^\(EditionIDs GeoLite2-Country\) GeoLite2-City/\1/" /etc/GeoIP.conf
```

Restart the GeoIP Update service.

```console
systemctl restart geoipupdate
```

Install the Nextcloud App "**GeoBlocker**".

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install geoblocker
```

As Administrator

- ⬆ open a "**New private window**", in the web browser
	- ⬆ open URL `https://aodomainao`
		- ⬆ login with the administrator account `ncadminnc`
			- ⬆ select "**Administration settings**" from the menu
				- ⬆ select "**GeoBlocker**" in the left panel
					- ✏️ select "MaxMindGeoLite2 (local)" as source for the service

=== `/etc/GeoIP.conf` ===

```code
# Please see https://dev.maxmind.com/geoip/updating-databases?lang=en for
# instructions on setting up geoipupdate, including information on how to
# download a pre-filled GeoIP.conf file.

# Replace YOUR_ACCOUNT_ID_HERE and YOUR_LICENSE_KEY_HERE with an active account
# ID and license key combination associated with your MaxMind account. These
# are available from https://www.maxmind.com/en/my_license_key.
AccountID YOUR_ACCOUNT_ID_HERE
LicenseKey YOUR_LICENSE_KEY_HERE

# Enter the edition IDs of the databases you would like to update.
# Multiple edition IDs are separated by spaces.
#
# Include one or more of the following edition IDs:
# * GeoLite2-ASN - GeoLite 2 ASN
# * GeoLite2-City - GeoLite 2 City
# * GeoLite2-Country - GeoLite2 Country
EditionIDs GeoLite2-Country
```

## Intrusion Detection/Prevention (DRAFT - work in progress)

### Fail2ban

Before we start, it is to be said


fail2ban-client status --all

fail2ban-client banned

fail2ban-client set apache-onlyoffice unbanip 192.168.22.101

#### Setup

Install the required software

```console
apt install --yes fail2ban
```

Configure and enable the rules for

- nextcloud
- apache-badbots, apache-botsearch, apache-fakegooglebot (using existing "`*.conf`")
- apache-badurls

Note: *Even if the **`bantime`** is specified mostly as "**`48h`**", use "**`172800`**" instead with some variations of the value (e.g. ±3600 for hours; ±600 for 10 minutes). Or ajust the values in free-style, like the following demonstrates*

- **`86400`** -> **`86413`**
- **`48h`** -> **`172800`** -> **`173254`**
- **`48h`** -> **`172800`** -> **`172385`**
- **`48h`** -> **`172800`** -> **`171903`**

```console
# nextcloud
cat << EOF >> /etc/tmpfiles.d/log-file.conf
f       /var/log/nextcloud/nextcloud.log              0640 www-data www-data - -
EOF
cat << EOF > /etc/fail2ban/filter.d/nextcloud.conf
[Definition]
_groupsre = (?:(?:,?\s*"\w+":(?:"[^"]+"|\w+))*)
failregex = ^\{%(_groupsre)s,?\s*"remoteAddr":"<HOST>"%(_groupsre)s,?\s*"message":"Login failed:
            ^\{%(_groupsre)s,?\s*"remoteAddr":"<HOST>"%(_groupsre)s,?\s*"message":"Trusted domain error.
datepattern = ,?\s*"time"\s*:\s*"%%Y-%%m-%%d[T ]%%H:%%M:%%S(%%z)?"
EOF
cat << EOF > /etc/fail2ban/jail.d/nextcloud.local
[nextcloud]
backend = auto
enabled = true
port = 80,443
protocol = tcp
filter = nextcloud
maxretry = 3
bantime = 86413
findtime = 43200
logpath = /var/log/nextcloud/nextcloud.log
EOF
# apache-badbots.local
cat << EOF >> /etc/fail2ban/jail.d/apache-badbots.local
[apache-badbots]
port     = http,https
filter   = apache-badbots
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
EOF
# apache-botsearch.local
cat << EOF >> /etc/fail2ban/jail.d/apache-botsearch.local
[apache-botsearch]
port     = http,https
filter   = apache-botsearch
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
EOF
# apache-fakegooglebot.local
cat << EOF >> /etc/fail2ban/jail.d/apache-fakegooglebot.local
[apache-fakegooglebot]
port     = http,https
filter   = apache-fakegooglebot
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
EOF
# apache-badurls
cat << EOF > /etc/fail2ban/filter.d/apache-badurls.conf
# Fail2Ban configuration file
#
# Regexp to catch known spambots and software alike. Please verify
# that it is your intent to block IPs which were driven by
# above mentioned bots.

[Definition]

failregex = ^<HOST> -.*"(GET|POST|HEAD) \/shell.php.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/shells.php.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/console.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/.env HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/.env\..* HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/env HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/env\..* HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/docker-.*.yml HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/config.js HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/Autodiscover.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/vendor.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/solr.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/admin HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/wp\/.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/wp-login HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/blog HTTP.*

ignoreregex =.*(robots.txt|favicon.ico|jpg|jpeg|png|gif|css)

datepattern = ^[^\[]*\[({DATE})
              {^LN-BEG}
EOF
cat << EOF > /etc/fail2ban/jail.d/apache-badurls.local
[apache-badurls]

port     = http,https
filter   = apache-badurls
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
EOF
systemctl restart fail2ban
```

Test status for "nextcloud" configuration/rule.

```console
# fail2ban-client status nextcloud
Status for the jail: nextcloud
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	0
|  `- File list:	/var/log/nextcloud/nextcloud.log
`- Actions
   |- Currently banned:	0
   |- Total banned:	0
   `- Banned IP list:	
```

Test status for "apache-badurls" configuration/rule.

```console
# fail2ban-client status apache-badurls
Status for the jail: apache-badurls
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	0
|  `- File list:	...
`- Actions
   |- Currently banned:	0
   |- Total banned:	0
   `- Banned IP list:	
```

=== `/etc/fail2ban/filter.d/nextcloud.conf` ===

```code
[Definition]
_groupsre = (?:(?:,?\s*"\w+":(?:"[^"]+"|\w+))*)
failregex = ^\{%(_groupsre)s,?\s*"remoteAddr":"<HOST>"%(_groupsre)s,?\s*"message":"Login failed:
            ^\{%(_groupsre)s,?\s*"remoteAddr":"<HOST>"%(_groupsre)s,?\s*"message":"Trusted domain error.
datepattern = ,?\s*"time"\s*:\s*"%%Y-%%m-%%d[T ]%%H:%%M:%%S(%%z)?"
```

=== `/etc/fail2ban/jail.d/nextcloud.local` ===

```code
[nextcloud]
backend = auto
enabled = true
port = 80,443
protocol = tcp
filter = nextcloud
maxretry = 3
bantime = 86400
findtime = 43200
logpath = /var/log/nextcloud/nextcloud.log
```

=== `/etc/fail2ban/jail.d/apache-badbots.local` ===

```config
[apache-badbots]
port     = http,https
filter   = apache-badbots
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
```

=== `/etc/fail2ban/jail.d/apache-botsearch.local` ===

```config
[apache-botsearch]
port     = http,https
filter   = apache-botsearch
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
```

=== `/etc/fail2ban/jail.d/apache-fakegooglebot.local` ===

```config
[apache-fakegooglebot]
port     = http,https
filter   = apache-fakegooglebot
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
```

=== `/etc/fail2ban/filter.d/apache-badurls.conf` ===

```code
# Fail2Ban configuration file
#
# Regexp to catch known spambots and software alike. Please verify
# that it is your intent to block IPs which were driven by
# above mentioned bots.

[Definition]

failregex = ^<HOST> -.*"(GET|POST|HEAD) \/shell.php.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/shells.php.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/console.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/.env HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/.env\..* HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/env HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/env\..* HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/docker-.*.yml HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) .*\/config.js HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/Autodiscover.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/vendor.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/solr.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/admin HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/wp\/.*HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/wp-login HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/blog HTTP.*

ignoreregex =.*(robots.txt|favicon.ico|jpg|jpeg|png|gif|css)

datepattern = ^[^\[]*\[({DATE})
              {^LN-BEG}
```

=== `/etc/fail2ban/jail.d/apache-badurls.local` ===

```code
[apache-badurls]

port     = http,https
filter   = apache-badurls
logpath  = /var/log/apache2/dummy-access.log
           /var/log/apache2/aohostgenericao-access.log
           /var/log/apache2/www-access.log
           /var/log/apache2/other_vhosts_access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
```

=== `/etc/fail2ban/filter.d/apache-onlyoffice.conf` ===

```code
[Definition]

failregex = ^<HOST> -.*"(GET|POST|HEAD) \/ HTTP.*
            ^<HOST> -.*"(GET|POST|HEAD) \/welcome.*HTTP.*

ignoreregex =.*(robots.txt|favicon.ico|jpg|jpeg|png|gif|css)

datepattern = ^[^\[]*\[({DATE})
              {^LN-BEG}
```

=== `/etc/fail2ban/jail.d/apache-onlyoffice.local` ===

```code
[apache-onlyoffice]

port     = http,https
filter   = apache-onlyoffice
logpath  = /var/log/apache2/ooserveroo-access.log
maxretry = 1
findtime = 30m
bantime  = 48h
enabled  = true
```

## IPsec VPN & Co.

### OpenVPN

#### Prerequisites

This server process as well, needs some certificates, most of which can be "-reused" from the existing Apache configuration.

- /etc/ssl/certs/canameca.crt
- /etc/ssl/crl/canameca.crl
- /etc/ssl/certs/aodomainao.crt
- /etc/ssl/private/aodomainao.key
- /etc/ssl/certs/dhparams.aohostao.pem

##### TLS authentication (TA)

The OpenVPN connection, in this configuration, is intended to be used by the administrator only. Also for simplicity, to avoid using TLS Crypt v2.

```console
openvpn --genkey secret /etc/ssl/private/ta.aodomainao.key
```

#### Server

##### Installation

1. Beside OpenVPN, iptables is required to allow communication from the VPN-Tunnel to the Internet. And to store the iptables configuration, the two `*-persitent` packages are required as well.
2. Create the VPN-Server configuration file `aoservervpnprimaryao.conf`.
3. Enable and start the OpenVPN Server.

Note: During the installation of `iptables-persistent`, you will be asked some questions. Ignore these (select "<No>"), we will deal with this aspect later

```console
apt install --yes openvpn iptables netfilter-persistent iptables-persistent
vi /etc/openvpn/server/aoservervpnprimaryao.conf
systemctl --now enable openvpn-server@aoservervpnprimaryao
```

=== `/etc/openvpn/server/aoservervpnprimaryao.conf` ===

```code
verb 0  # verbose mode

management localhost 7505
status /var/log/openvpn/openvpn-status.log

port 1194
proto udp
dev tun

dh         /etc/ssl/certs/dhparams.aohostao.pem
ca         /etc/ssl/certs/canameca.crt
crl-verify   /etc/ssl/crl/canameca.crl

cert       /etc/ssl/certs/aodomainao.crt
key      /etc/ssl/private/aodomainao.key
data-ciphers AES-256-CBC:AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305
auth-nocache

tls-version-min 1.2
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256
tls-auth /etc/ssl/private/ta.aodomainao.key 0

topology subnet
server aovpnprimaryipv4addressao aovpnprimaryipv4maskao  # internal tun0 connection IP
#ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
keepalive 10 120
persist-key
persist-tun
client-to-client
explicit-exit-notify 1
```

##### Routing configuration

Before we can adjust the routine of network packages, the kernel parameter `net.ipv4.ip_forward` has to be enabled using `sysctl`.

```console
cp /etc/sysctl.conf /etc/sysctl.conf.ORG
sed -i "s/^#\(net.ipv4.ip_forward=1\)/\1/" /etc/sysctl.conf
sysctl --load
```

Alternative is to add the rule to the new file `99-ipforward.conf`.

```console
echo net.ipv4.ip_forward=1 > /etc/sysctl.d/99-ipforward.conf
sysctl --load
```

=== `/etc/sysctl.conf` ===

```code
:

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

:
```

In the old style, using iptables, the routing is specify in the following.

```console
iptables --table nat --append POSTROUTING --out-interface aohostlanao --jump MASQUERADE
iptables --append FORWARD --in-interface aohostlanao --out-interface tun0 --match state --state RELATED,ESTABLISHED --jump ACCEPT
iptables --append FORWARD --in-interface tun0 --out-interface aohostlanao --jump ACCEPT
```

To store the configuration, the following commands are required.

Note: Anytime your change something on the iptables configuration, make sure you execute the same two `ip*tables-save` commands like above.

```console
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
netfilter-persistent save
```

The last command should have printed the following.

```console
run-parts: executing /usr/share/netfilter-persistent/plugins.d/15-ip4tables save
run-parts: executing /usr/share/netfilter-persistent/plugins.d/25-ip6tables save
```

#### Client

1. First the OpenVPN software has to be installed.
2. In additional to the already deployed `.pfx` PKCS#12-Certificate, the source files `.crt` and `.key` have to be deployed as well, into `/etc/ssl/certs/` and `/etc/ssl/private/`.
2. Copy the file `/etc/ssl/private/ta.aodomainao.key` from the server at the same location.
3. Create the VPN-Client configuration file `client.conf`

```console
apt install --yes openvpn
```

=== `/etc/openvpn/client/client.conf` ===

```code
verb 0
status /var/log/openvpn/primary.out

client
dev tun
proto udp

remote aodomainao 1194
resolv-retry infinite
nobind

persist-key
persist-tun

ca /usr/local/share/ca-certificates/my-custom-ca/ca.crt
cert  "/etc/ssl/certs/client.crt"
key "/etc/ssl/private/client.key"

remote-cert-tls server
tls-auth /etc/ssl/private/ta.aodomainao.key 1
auth-nocache

mute-replay-warnings
```

#### Access DMZ

In case access to other servers in the DMZ is required.

##### Establish Route

```console
ip route add aovpnprimaryipv4addressao/aovpnprimaryipv4suffixao \
             via aovpnprimaryipv4serveraddressao
```

##### Remove Route

```console
ip route del aovpnprimaryipv4addressao/aovpnprimaryipv4suffixao
```

## Lighttp for Maintenance

There are many cases in which you would have to shutdown the Apache Web-Server. During that downtime the clients request will simply timeout. Leaving the user with no glue about what's going on.

The bare minimum is to response at least with an HTTP Error 503 "Service Unavailable".

With Lighttp & Lua such a maintenance page can be provided to the user and Apps. As a side effect the URL `.../.well-known/acme-challenge` remains accessible and allows during maintenance the renewel of the Web-Server Certificates.

To avoid disable/enable Lighttp, the status/ports are controlled by the existance of the file `/tmp/LIGHTTP_MAINTENANCE`. Lighttp change looks for that file at reload and (re-)start.
- file exists: Lighttp will listen on ports 80 and 443
- no file available: Lighttp will listen only on ports 1080 and 1443 

Note: **As `/tmp` is located in a RAM-Disk ... After a restart the file `/tmp/LIGHTTP_MAINTENANCE` and ACNO would operate as normal.**

### Setup

1. install the packages
2. make a backup of the original Lighttp configuration
3. create the following five files as documented below
  - `Error503.lua`
  - `Error503.html`
  - `lighttpd.conf`
  - `lighttpd_include.conf`
  - `lighttpd_include.sh`
4. shutdown Apache
5. start Lighttp
6. test the resposne of the Web-Server
7. disable Lighttp
8. start Apache

```console
apt install --yes lighttpd lighttpd-mod-magnet
mv /etc/lighttpd/lighttpd.conf /etc/lighttpd/lighttpd.conf.ORG
# Create the five files
systemctl stop apache2
touch /tmp/LIGHTTP_MAINTENANCE
systemctl restart lighttpd
```

Test the response of the Lighttp Web-Server by opening the URLs
- `https://aodomainao/` response should be 503
- `https://aodomainao/.well-known/acme-challenge` response should be 404

```console
rm --force /tmp/LIGHTTP_MAINTENANCE
systemctl restart lighttpd
systemctl start apache2
```

### Enable Maintenance

Combine with the Maintenance-Mode of Nextcloud, the commands would look like the following.

```console
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:mode --on
systemctl stop apache2
touch /tmp/LIGHTTP_MAINTENANCE
systemctl restart lighttpd
```

### Disable Maintenance

Combine with the Maintenance-Mode of Nextcloud, the commands would look like the following.

```console
rm --force /tmp/LIGHTTP_MAINTENANCE
systemctl restart lighttpd
systemctl start apache2
sudo -u www-data php /var/www/ncdirectorync/occ maintenance:mode --off
```

=== `/etc/lighttpd/Error503.lua` ===

```lua
local r = lighty.r
local http_status = 503
local pagefile = "/etc/lighttpd/Error503.html" 
r.resp_body:set({ { filename = pagefile } })
return 503
```

=== `/etc/lighttpd/Error503.html` ===

```html
<!DOCTYPE HTML>
<html>
  <head>
    <title>503 Service Unavailable</title>
  </head>
  <body>
    Please excuse the inconvenience.<br>
    We do have noted down your IP-Address to inform you, once the disturbance got resolved.
  </body>
</html>
```

=== `/etc/lighttpd/lighttpd.conf` ===

```code
server.modules += ( "mod_openssl" )
server.modules += ( "mod_magnet" )

server.tag = "Unknown"

mimetype.assign   = ( ".html" => "text/html" )

server.username             = "www-data"
server.groupname            = "www-data"

server.document-root = "/var/www/html"

lighttpd-enable-mod accesslog
accesslog.filename          = "/var/log/lighttpd/access.log"
server.errorlog             = "/var/log/lighttpd/error.log"

include_shell "/etc/lighttpd/lighttpd_include.sh"

$HTTP["url"] !~ "^/.well-known/acme-challenge" {
  magnet.attract-raw-url-to = ( "/etc/lighttpd/Error503.lua" )
}
```


=== `/etc/lighttpd/lighttpd_include.conf` ===

```code
# Changes on the port numbers have to be done in lighttpd_include.conf and lighttpd_include.shnf
#
# default ports are 1080 & 1443, who should not harm anybody
# ports will be changed to 80 & 443, if file "/tmp/LIGHTTP_MAINTENANCE" exists
#

server.port               = 1080

$SERVER["socket"] == ":1080" {
}

$SERVER["socket"] == ":1443" {
  ssl.engine = "enable"
  ssl.verifyclient.activate = "enable"
  #ssl.verifyclient.enforce = "enable"
  ssl.verifyclient.username = "SSL_CLIENT_S_DN_CN"

  ssl.verifyclient.ca-file     = "/etc/ssl/certs/GoP_ca.crt"
  ssl.verifyclient.ca-crl-file = "/etc/ssl/crl/GoP_ca.crl"
  ssl.openssl.ssl-conf-cmd = ("MinProtocol" => "TLSv1.2",
                              "Options" => "ServerPreference",
                              "CipherString" => "HIGH",
                              "DHParameters" => "/etc/ssl/certs/dhparams.aodomain.pem")
  ssl.pemfile           = "/etc/ssl/certs/aodomain.crt"
  ssl.privkey           = "/etc/ssl/private/aodomain.key"
}
```

=== `/etc/lighttpd/lighttpd_include.conf` (Using Let's Encrypt) ===

```code
# Changes on the port numbers have to be done in lighttpd_include.conf and lighttpd_include.shnf
#
# default ports are 1080 & 1443, who should not harm anybody
# ports will be changed to 80 & 443, if file "/tmp/LIGHTTP_MAINTENANCE" exists
#

server.port               = 1080

$SERVER["socket"] == ":1080" {
}

$SERVER["socket"] == ":1443" {
  ssl.engine = "enable"
  ssl.verifyclient.activate = "enable"
  #ssl.verifyclient.enforce = "enable"
  ssl.verifyclient.username = "SSL_CLIENT_S_DN_CN"

  ssl.verifyclient.ca-file     = "/etc/ssl/certs/GoP_ca.crt"
  ssl.verifyclient.ca-crl-file = "/etc/ssl/crl/GoP_ca.crl"
  ssl.openssl.ssl-conf-cmd = ("MinProtocol" => "TLSv1.2",
                              "Options" => "ServerPreference",
                              "CipherString" => "HIGH",
                              "DHParameters" => "/etc/ssl/certs/dhparams.aodomain.pem")
  ssl.pemfile           = "/etc/ssl/certs/aodomain.crt"
  ssl.privkey           = "/etc/ssl/private/aodomain.key"
}

$HTTP["host"] == "aodomain" {
  ssl.pemfile           = "/etc/letsencrypt/live/aodomain/fullchain.pem"
  ssl.privkey           = "/etc/letsencrypt/live/aodomain/privkey.pem"
}

$HTTP["host"] == "coserverco.aodomain" {
  ssl.pemfile           = "/etc/letsencrypt/live/coserverco.aodomain/fullchain.pem"
  ssl.privkey           = "/etc/letsencrypt/live/coserverco.aodomain/privkey.pem"
}

$HTTP["host"] == "ooserveroo.aodomain" {
  ssl.pemfile           = "/etc/letsencrypt/live/ooserveroo.aodomain/fullchain.pem"
  ssl.privkey           = "/etc/letsencrypt/live/ooserveroo.aodomain/privkey.pem"
}

$HTTP["host"] == "ctserverct.aodomain" {
  ssl.pemfile           = "/etc/letsencrypt/live/ctserverct.aodomain/fullchain.pem"
  ssl.privkey           = "/etc/letsencrypt/live/ctserverct.aodomain/privkey.pem"
}
```

=== `/etc/lighttpd/lighttpd_include.sh` ===

```sh
# Changes on the port numbers have to be done in lighttpd_include.conf and lighttpd_include.sh
#
# default ports are 1080 & 1443, who should not harm anybody
# ports will be changed to 80 & 443, if file "/tmp/LIGHTTP_MAINTENANCE" exists
#

if [ -f "/tmp/LIGHTTP_MAINTENANCE" ]; then
  sed --expression="s/1080/80/" --expression="s/1443/443/" /etc/lighttpd/lighttpd_include.conf
else
  cat /etc/lighttpd/lighttpd_include.conf
fi
```

## Scripts (Automation)

From "[GitHub: dietriclX / Automaton](https://github.com/dietriclX/Automaton)" download the content of `Debian 12 - ACNO` into the direcotry `/root/scripts/acno` of the ACNO machine.

Make sure that the entities within `/root/script` have the right attributes.

```console
chown --recursive root:root   /root/scripts
chown             root:backup /root/scripts/data
find /root/scripts -type d                   -exec chmod 770 {} \;
find /root/scripts -type f      -name '*.sh' -exec chmod 770 {} \;
find /root/scripts -type f -not -name '*.sh' -exec chmod 660 {} \;
```


## SSH/OpenSSH

In the following, two kinds of configurations are outlined.

### Loose

Allow even `root` to login directly using ssh, using either local password or public key.

```console
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.BAK
sed --in-place --expression='s/^#\(PermitRootLogin\)\(.*\)$/#\1\2\n\1 yes/' /etc/ssh/sshd_config
systemctl restart ssh
```

=== `/etc/ssh/sshd_config` ===

```code
:

#LoginGraceTime 2m
#PermitRootLogin prohibit-password
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

:
```

### Strict

Forbit login as `root`.
Forbit users to add new public keys to its aohostao account.
Forbit authentication via passwords.

To do this, a general change is required.
- for the existing users with a home-directory `/home/<user>`, move `/home/<user>/.ssh/authorized_keys` to `/etc/ssh/authorized_keys.d/<user>` 
- ensure only `root` will be able to maintain the authorized keys (`/etc/ssh/authorized_keys.d/<user>`)
- allow users only read-access (as required for the authentication)
- the strict rules for ssh are going to be specified in `/etc/ssh/sshd_config.d/using_authorized_keys.d.conf`

```console
mkdir --mode=755 /etc/ssh/authorized_keys.d
ls /home | while read x
do
  mv /home/$x/.ssh/authorized_keys /etc/ssh/authorized_keys.d/$x
  chown root:root /etc/ssh/authorized_keys.d/$x
  chmod 644 /etc/ssh/authorized_keys.d/$x
done
cat <<EOF > /etc/ssh/sshd_config.d/using_authorized_keys.d.conf
# To disable tunneled clear text passwords, change to "no" here!
PasswordAuthentication no

PubkeyAuthentication yes

# Authorized public keys stored at a single place.
# User has only Read-Access.
AuthorizedKeysFile /etc/ssh/authorized_keys.d/%u

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no
EOF
systemctl restart ssh
```

=== `/etc/ssh/sshd_config.d/using_authorized_keys.d.conf` ===
```code
# To disable tunneled clear text passwords, change to "no" here!
PasswordAuthentication no

PubkeyAuthentication yes

# Authorized public keys stored at a single place.
# User has only Read-Access.
AuthorizedKeysFile /etc/ssh/authorized_keys.d/%u

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication no
```

## UnifiedPush

☝ [Notes: Add-On > UnifiedPush](./Notes.md#unifiedpush)

### UnifiedPush Provider

#### Server

Install the Nextcloud App.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install uppush
```

#### Client "**NextPush**"

Prerequisite: Have an App-Password created with "**App-Name**" being named **`<device name> NextPush`**. Store the generated App-Password in your Password Manager. See [**Mobile Devices - OS-App (Replacement/Supplement)**](../../Mobile%20Devices/Readme.md#os-app-replacementsupplement)

Set up the ["**NextPush**"](https://codeberg.org/NextPush/nextpush-android) app on the Smartphone.

- install "**NextPush**" app
- ⬆ start "**NextPush**" app
  - ⬆ tap on "**OK**" in the dialog "**Permissions** This application requires notifications permission to work."
    - ✏️🔙 tap on  **`Allow`** on the Android question to allow NextPush notifications
  - ⬆ tap on "**Manual login**" in the initial start dialog
    - ✏️✓ type user id into "**Username**"
    - ✏️✓ type App-Password into "**Application password**"
    - ✏️✓ type **`https://aodomainao`** into "**Nextcloud root url**"
    - ✓🔙 tap on  **`Sign in`**

After a successful login, you will be greated by the message "To ensure the app functions properly, it is important to disable battery optimization. This will prevent the app from being put to sleep and causing delays in notification."

- ⬆ tap on "**Disable optimization**"
  - ✏️🔙 tap on "**Allow**" on dialog "**Let app always run in background?** Allowing NextPush to always run in the background may reduce battery life. ..."

In addition, disable the option "**Manage app if unused**" in the Androids "**Settings > Apps > NextPush**" (section "**Unused app settings**").

#### Molly-FOSS (Signal chat client)

##### Parameters/Attributes/configuration

- User for MollySocket service: msuserms
- Group for MollySocket service: msgroupms
- Binary name in `/usr/local/bin`: mssrvbinaryms
- Port of MollySockert server: mssrvportms
- Place/Path within Nextcloud directory: msurlpathms

##### Server

Example will download and install version `1.6.0` of MollySocket from [GitHub](https://github.com/mollyim/mollysocket/releases). For newer version adjust the second statement (`...load/1.6.0/moll...`).

In order to include the MollySocket into the Nextcloud instance, add the directive `<Location /msurlpathms/>...</Location>` into the `/etc/apache2/sites-available/nextcloud-include.conf` configuration and restart the Apache2 service.

Check the proper implementation by opening the URL `https://aodomainao/msurlpathms/`. The QR-Code for this instance should be presented.

```console
groupadd --system msgroupms
useradd --no-create-home --gid msgroupms msuserms
wget https://github.com/mollyim/mollysocket/releases/download/1.6.0/mollysocket-linux_amd64
mv mollysocket-linux_amd64 /usr/local/bin/
chmod 755 /usr/local/bin/mollysocket-linux_amd64
ln --symbolic /usr/local/bin/mollysocket-linux_amd64 /usr/local/bin/mssrvbinaryms
cat << EOF > /etc/systemd/system/mollysocket.service
[Unit]
Description=MollySocket
After=network-online.target

[Service]
Type=simple
Environment="RUST_LOG=info"
Environment="MOLLY_CONF=/etc/mollysocket/conf.toml"

LoadCredentialEncrypted=vapid.key:/etc/mollysocket/vapid.key
Environment=MOLLY_VAPID_KEY_FILE=%d/vapid.key

User=msuserms
Group=msgroupms
ConfigurationDirectory=mollysocket::ro
StateDirectory=mollysocket
UMask=0007
ProtectHome=true
ProtectSystem=true

ExecStart=mssrvbinaryms server

Restart=on-failure

# Configures the time to wait before service is stopped forcefully.
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
EOF
chmod 755 /etc/systemd/system/mollysocket.service
mkdir --mode=755 /etc/mollysocket
chown mollysocket:msgroupms /etc/mollysocket
mollysocket vapid gen > /etc/mollysocket/vapid_privkey.txt
chown mollysocket:msgroupms /etc/mollysocket/vapid_privkey.txt
chmod 640 /etc/mollysocket/vapid_privkey.txt
systemd-creds encrypt --pretty \
                      /etc/mollysocket/vapid_privkey.txt \
                      /etc/mollysocket/vapid.key
chown msuserms:msgroupms /etc/mollysocket/vapid.key
chmod 640 /etc/mollysocket/vapid.key
mkdir --mode=755 /var/lib/mollysocket
chown msuserms:msgroupms /var/lib/mollysocket
cat << EOF > /etc/mollysocket/conf.toml
allowed_endpoints = ['*','https://aodomainao']
db = '/var/lib/mollysocket/database.sqlite'
port = mssrvportms
webserver = true
EOF
chown msuserms:msgroupms /etc/mollysocket/conf.toml
chmod 640 /etc/mollysocket/conf.toml
systemctl enable --now mollysocket
systemctl status mollysocket
```

=== `/etc/apache2/sites-available/nextcloud-include.conf` ===
```code
:
  ProxyPass        /push/ws ws://127.0.0.1:7867/ws
  ProxyPass        /push/ http://127.0.0.1:7867/
  ProxyPassReverse /push/ http://127.0.0.1:7867/

  <Location /msurlpathms/>
    ProxyPass http://localhost:mssrvportms/
    ProxyPassReverse http://localhost:mssrvportms/
    RequestHeader set X-Original-URL "/msurlpathms/"
    ProxyPreserveHost On
  </Location>

  RewriteEngine On
  RewriteCond %{REQUEST_METHOD} ^TRACK
  RewriteRule .* - [R=405,L]
```

=== `/etc/systemd/system/mollysocket.service` ===

```code
[Unit]
Description=MollySocket
After=network-online.target

[Service]
Type=simple
Environment="RUST_LOG=info"
Environment="MOLLY_CONF=/etc/mollysocket/conf.toml"

LoadCredentialEncrypted=vapid.key:/etc/mollysocket/vapid.key
Environment=MOLLY_VAPID_KEY_FILE=%d/vapid.key

User=mollysocket
Group=mollysocket
ConfigurationDirectory=mollysocket::ro
StateDirectory=mollysocket
UMask=0007
ProtectHome=true
ProtectSystem=true

ExecStart=mollysocket server

Restart=on-failure

# Configures the time to wait before service is stopped forcefully.
TimeoutStopSec=5

[Install]
WantedBy=multi-user.target
```

##### Client "**Molly-FOSS**"

On the Smartphone, ["**Molly-FOSS**"](https://molly.im) app is already installed and configured.

- ⬆ start "**Molly-FOSS**" app
  - tap on **`⋮`** in the top right corner
  - ⬆ tap on "**Settings**" menu entry
    - ⬆ tap on "**Notifications**"
      - focus on section "**Push notifications**"
      - ⬆ tap on "**Delivery service**"
        - ✏️✓⬆ select "**UnifiedPush**"
          - ✏️✓ tap on "**Scan QR code**"
          - ✓🔙 follow instruction "Scan the QR code displayed on your MollySocket server's homepage or in the console."

## Uninterruptible Power Supply (UPS)

### NUT Client (remote)

When you have already Network UPS Tools (NUT) running on another machine, the ACNO machine can utilitze via network the UPS device.

Prerequisite

- the ACNO machine gets the power of the UPS device
- another system `uphostnameup` is locally connected to the UPS device
- on `uphostnameup`,  is setup and configured to manage the `updevicenameup` UPS device
- on `uphostnameup`,  NUT provides a monitoring account (upmonitoraccountup/upmonitorpasswordup)
- the ACNO machine is capable of communicating via LAN with `uphostnameup`

First the client software of NUT has to be installed.

```console
apt install --yes nut-client
```

#### Test Communication

Execute the following to verify that the ACNO machine is capable of communicating with the NUT service, on the controller system with a local connection to the UPS device.

```console
# upsc -l uphostnameup
Init SSL without certificate database
updevicenameup
# upsc updevicenameup@uphostnameup
Init SSL without certificate database
battery.charge: 100
:
ups.vendorid: 051d
# █
```

#### Configure NUT Client

Make a backup of the original NUT configuration `/etc/nut/upsmon.conf` and `/etc/nut/upsmon.conf` and update the configuration files. And last but not least restart the related services.

```console
cp /etc/nut/nut.conf /etc/nut/nut.conf.ORG
cat << EOF > /etc/nut/nut.conf
MODE=netclient
EOF
cp /etc/nut/upsmon.conf /etc/nut/upsmon.conf.ORG
cat << EOF > /etc/nut/upsmon.conf
RUN_AS_USER root

MONITOR updevicenameup@uphostnameup 1 admin secret slave

MINSUPPLIES 1
SHUTDOWNCMD "/sbin/shutdown -h"
NOTIFYCMD /usr/sbin/upssched
POLLFREQ 2
POLLFREQALERT 1
HOSTSYNC 15
DEADTIME 15
POWERDOWNFLAG /etc/killpower

NOTIFYMSG ONLINE    "UPS %s on line power"
NOTIFYMSG ONBATT    "UPS %s on battery"
NOTIFYMSG LOWBATT   "UPS %s battery is low"
NOTIFYMSG FSD       "UPS %s: forced shutdown in progress"
NOTIFYMSG COMMOK    "Communications with UPS %s established"
NOTIFYMSG COMMBAD   "Communications with UPS %s lost"
NOTIFYMSG SHUTDOWN  "Auto logout and shutdown proceeding"
NOTIFYMSG REPLBATT  "UPS %s battery needs to be replaced"
NOTIFYMSG NOCOMM    "UPS %s is unavailable"
NOTIFYMSG NOPARENT  "upsmon parent process died - shutdown impossible"

NOTIFYFLAG ONLINE   SYSLOG+WALL+EXEC
NOTIFYFLAG ONBATT   SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT  SYSLOG+WALL
NOTIFYFLAG FSD      SYSLOG+WALL+EXEC
NOTIFYFLAG COMMOK   SYSLOG+WALL+EXEC
NOTIFYFLAG COMMBAD  SYSLOG+WALL+EXEC
NOTIFYFLAG SHUTDOWN SYSLOG+WALL+EXEC
NOTIFYFLAG REPLBATT SYSLOG+WALL
NOTIFYFLAG NOCOMM   SYSLOG+WALL+EXEC
NOTIFYFLAG NOPARENT SYSLOG+WALL

RBWARNTIME 43200

NOCOMMWARNTIME 600

FINALDELAY 5
EOF
service nut-client restart
systemctl restart nut-monitor
```

=== `/etc/nut/nut.conf` ===

```code
MODE=netclient
```

=== `/etc/nut/upsmon.conf` ===

```code
RUN_AS_USER root

MONITOR updevicenameup@uphostnameup 1 admin secret slave

MINSUPPLIES 1
SHUTDOWNCMD "/sbin/shutdown -h"
NOTIFYCMD /usr/sbin/upssched
POLLFREQ 2
POLLFREQALERT 1
HOSTSYNC 15
DEADTIME 15
POWERDOWNFLAG /etc/killpower

NOTIFYMSG ONLINE    "UPS %s on line power"
NOTIFYMSG ONBATT    "UPS %s on battery"
NOTIFYMSG LOWBATT   "UPS %s battery is low"
NOTIFYMSG FSD       "UPS %s: forced shutdown in progress"
NOTIFYMSG COMMOK    "Communications with UPS %s established"
NOTIFYMSG COMMBAD   "Communications with UPS %s lost"
NOTIFYMSG SHUTDOWN  "Auto logout and shutdown proceeding"
NOTIFYMSG REPLBATT  "UPS %s battery needs to be replaced"
NOTIFYMSG NOCOMM    "UPS %s is unavailable"
NOTIFYMSG NOPARENT  "upsmon parent process died - shutdown impossible"

NOTIFYFLAG ONLINE   SYSLOG+WALL+EXEC
NOTIFYFLAG ONBATT   SYSLOG+WALL+EXEC
NOTIFYFLAG LOWBATT  SYSLOG+WALL
NOTIFYFLAG FSD      SYSLOG+WALL+EXEC
NOTIFYFLAG COMMOK   SYSLOG+WALL+EXEC
NOTIFYFLAG COMMBAD  SYSLOG+WALL+EXEC
NOTIFYFLAG SHUTDOWN SYSLOG+WALL+EXEC
NOTIFYFLAG REPLBATT SYSLOG+WALL
NOTIFYFLAG NOCOMM   SYSLOG+WALL+EXEC
NOTIFYFLAG NOPARENT SYSLOG+WALL

RBWARNTIME 43200

NOCOMMWARNTIME 600

FINALDELAY 5
```

## Updates (None-Debian software)

Even though you might want to keep your machine up-to-date with regards to security updates, it is not always wise to apply to your None-Debian software the latest greates updates. Especially with regards to nginx, which comes with ONLYOFFICE Docs.

With the following command, you can keep Linux from automatically updating certain package(s).

```console
apt-mark hold <package name>
```

From time to time, you might want to check if packages are on hold, or not.

```console
apt-mark showhold <package name>
```

At any time, the automatic updates can be re-enabled.

```console
apt-mark unhold <package name>
```


### Collabora Online (Developer Edition)

To get controll about update installation, issue the following command to prevent automatic updates.

```console
apt-mark hold collabora-online
```

### ONLYOFFICE Docs

To get controll about update installation, issue the following command to prevent automatic updates.

```console
apt-mark hold onlyoffice-documentserver
```

## Variables

variables.sh (variables.ORG.sh)

## vCard Export (Address Book)

The export of an dedicated Address Book can used by a Phone-System, either a WiFi router with DECT feature or a DECT Base Station.

For private use, the easiest way to manage contact for the phone, is using a separate address book and get the data somehow out of Nextcloud and into the Phone-System.

### Export from Nextcloud

Assuming the Nextcloud user `aovcfownerao` owns the address book `aovcfaddressbookao`, its contacts can be exported via CardDAV using for example [GitHub: ljanyst / carddav-util](https://github.com/ljanyst/carddav-util).

To by-pass the Client Certificate authentication, the Apache has to get a virtual host for listinging on address `127.0.0.1` as Nextcloud server.

```console
mkdir --parents $HOME/scripts/acno
git clone https://github.com/ljanyst/carddav-util
```

```console
cd $HOME/scripts/acno/carddav-util
./carddav-util.py                    \
    --download                       \
    --user=aovcfownerao              \
    --passwd=aovcfownerpasswordao    \
    --file=/tmp/addressbook.vcf      \
    --url=http://127.0.0.1/remote.php/dav/addressbooks/users/aovcfownerao/aovcfaddressbookao
vcf2xml4gigaset.py /tmp/addressbook.vcf /var/www/OnlinePhonebook/addressbook.xml
```

### Import into Phone-System

Once the Address Book had been exported, it can be imported in the Phone-System in various ways.

If required the vCard would have to be converted from an .vcf file into an .xml file. 

#### Manually

Usually the Web UI of a Phone-System has a feature to upload an Address Book.

#### Automatically

Some Phone-Systems can automatically update their Phone Books, based on a file on a Web-Server. For this purpose - publishing an Address Book - the ACNO Web-Server can be used.

Also in this case, to by-pass the Client Certificate authentication, the Apache has to get a virtual host for listinging on server name `OnlinePhonebook.aolocaldomainao` as regulr Web-Server. Publishing the directory, in which the Address Book - for upload - is located.

=== `any.conf` ===

```code
:

<VirtualHost *:80 [::]:80>
  ServerName dummy.aolocaldomainao
  ServerAlias dummy
  Header set Access-Control-Allow-Origin "*"
  DocumentRoot "/var/www/html/"
  TransferLog /var/log/apache2/dummy-access.log
  ErrorLog    /var/log/apache2/dummy-error.log
   LogLevel warn
</VirtualHost>

<VirtualHost *:80 [::]:80>
  ServerName OnlinePhonebook.aolocaldomainao
  ServerAlias OnlinePhonebook
  Header set Access-Control-Allow-Origin "*"
  DocumentRoot "/var/www/OnlinePhonebook"
  TransferLog /var/log/apache2/OnlinePhonebook-access.log
  ErrorLog    /var/log/apache2/OnlinePhonebook-error.log
  LogLevel warn
</VirtualHost>

:
```

# Based on

A loose collection of information, I found on my endeavor to setup the ACNO server. I am very grateful for all the information, from overall picture over step-by-step instructions to tiny little details, left on the internet by all these marvellous human beings. Not to forget the search engines - Qwant and Google - which allowed me to find these bits and pieces.

The following are those articles/postings which helped me most.

## General

- [User Experience Stack Exchange: "We" vs. "You" in documentation](https://ux.stackexchange.com/questions/57305/we-vs-you-in-documentation)

## OS

### antivirus

- [Tec-Bite: Nextcloud Hardening > antivirus](https://www.tec-bite.ch/nextcloud-hardening/#antivirus)
- [Stack Overflow: How to be sure ClamAV database is up to date?](https://stackoverflow.com/questions/43514853/how-to-be-sure-clamav-database-is-up-to-date)

### Email

### Encryption (GnuPG(PGP))

- [Kagel Blog: PGP Schlüssel generieren wie ein Experte](https://www.kagel.ch/artikel/pgp-schluessel-generieren-wie-ein-experte/)
- [Yan Han's blog: GPG - How to trust an imported key](https://yanhan.github.io/posts/2014-03-04-gpg-how-to-trust-imported-key/)
- [Super User: gnupg - gpg remove passphrase - Super User](https://superuser.com/questions/1360324/gpg-remove-passphrase)

### hdparm

- [armbian: (Solved with hd-idle) configuring hdparm.config to spindown hdd's when not in use](https://forum.armbian.com/topic/9889-solved-with-hd-idle-configuring-hdparmconfig-to-spindown-hdds-when-not-in-use/)

### SSD

- [LinuxComminuty: Solid State Drives unter Linux schonend ausreizen (Seite 3)](https://www.linux-community.de/ausgaben/linuxuser/2015/02/pimp-my-ssd/3/)

### S.M.A.R.T.

- [Thomas Krenn: Smartctl](https://www.thomas-krenn.com/de/wiki/Smartctl)
- [cshire.xyz: Smartd and Encrypted Notifications](https://cshire.xyz/smartd-and-encrypted-notifications/)
- [Natural Born Coder: Setting up SMART Monitoring in Proxmox](https://www.naturalborncoder.com/2023/05/setting-up-smart-monitoring-in-proxmox/)
- [Community Help Wiki: Smartmontools](https://help.ubuntu.com/community/Smartmontools)

### Shell Script

- [Unix & Linux StackExchange: How to temporarily save and restore the IFS variable properly?](https://unix.stackexchange.com/questions/640062/how-to-temporarily-save-and-restore-the-ifs-variable-properly)

## Collabora Online/CODE

- [Collabora: How to Install and Test CODE](https://www.collaboraonline.com/code/#learnmorecode)
- [TECHMINT: How to Install ONLYOFFICE Docs on Debian and Ubuntu](https://www.tecmint.com/install-onlyoffice-docs-on-debian-ubuntu/)

## Intrusion Detection/Prevention

- [Andreas 'ads' Scherbaum: Using fail2ban to block unfriendly web requests](https://andreas.scherbaum.la/post/2022-12-09_using-fail2ban-to-block-unfriendly-web-requests/)
- [blog.admin-intelligence.de: Fail2ban zur Absicherung der Nextcloud](https://blog.admin-intelligence.de/fail2ban-fuer-nextcloud/)
- [Nextcloud: Hardening and security guidance > Setup a filter and a jail for Nextcloud](https://docs.nextcloud.com/server/19/admin_manual/installation/harden_server.html?highlight=fail2ban#setup-fail2ban)

## Nextcloud

- [Nextcloud Community: Do not allow auto-create user if not existing](https://help.nextcloud.com/t/do-not-allow-auto-create-user-if-not-existing/165046)
- [Nextcloud Community: How to remove/disable on iOS/Android the “App install banner”?](https://help.nextcloud.com/t/how-to-remove-disable-on-ios-android-the-app-install-banner/90942)

## ONLYOFFICE Docs

- [ONLYOFFICE: Installing ONLYOFFICE Docs for Debian, Ubuntu, and derivatives](https://helpcenter.onlyoffice.com/docs/installation/docs-community-install-ubuntu.aspx)

## OpenSSL

- [How To Setup a CA](https://pages.cs.wisc.edu/~zmiller/ca-howto/)
- [Fabasphere: Create User Certificates via OpenSSL](https://help.cloud.fabasoft.com/index.php?topic=doc/CA-and-User-Certificate-Management/create-user-certificates-via-openssl.htm)
- [Apache: SSL/TLS Strong Encryption: How-To](https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html)
- [GoLinuxCloud: How to add X.509 extensions to certificate OpenSSL](https://www.golinuxcloud.com/add-x509-extensions-to-certificate-openssl/)
- [Jaanus: How to configure development server certificates for iOS 13 and Mac clients](https://jaanus.com/ios-13-certificates/)
- [Apple Support: Requirements for trusted certificates in iOS 13 and macOS 10.15](https://support.apple.com/en-us/103769)

## OpenVPN

- [debian Wiki: OpenVPN](https://wiki.debian.org/OpenVPN)
- [Reintech media: Configuring IPTables Firewall on Debian 12](https://reintech.io/blog/configuring-iptables-firewall-debian-12)

## SSH/OpenSSH

- [linuxize.com: How to Setup Passwordless SSH Login](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)
- [debian Wiki: SSH > Regenerating host keys](https://wiki.debian.org/SSH#Regenerating_host_keys)
- [digital-io.de: SSH – Login für Root erlauben ](https://digital-io.de/ssh-login-fuer-root-erlauben/)

## TURN-Server -coturn

- [gabrieltanner.org: How to set up and configure your own TURN server using Coturn](https://gabrieltanner.org/blog/turn-server/)

## Web-Server - Apache

- [Apache Week Feature: Using Certificate Revocation Lists](http://www.apacheweek.com/features/crl)
- [Carsten Rieger: Nextcloud 23 Installationsanleitung mit Apache2](https://www.c-rieger.de/nextcloud-installationsanleitung-apache2/)
- [InMotion Hosting: How to Hide Your Apache Version and Linux OS From HTTP Headers](https://www.inmotionhosting.com/support/server/apache/hide-apache-version-and-linux-os/)
- [Nick Frostbutter: Redirect http to https with htaccess and mod_rewrite](https://dev.to/nickfrosty/redirect-http-to-https-force-https-with-htaccess-42c9)
- [sslshopper.com: Apache Redirect HTTP to HTTPS using mod_rewrite](https://www.sslshopper.com/apache-redirect-http-to-https.html)

## UPS

- [Techno Tim: Network UPS Tools (NUT) Ultimate Guide](https://technotim.live/posts/NUT-server-guide/)

# Acronym

| Acronym | Stands for |
| ---: | :--- |
| CPU | Central Processing Unit |
| DBMS | Database Management System |
| DDNS | Dynamic Domain Name System |
| DSL | Digital Subscriber Line |
| GPG | GNU Privacy Guard |
| IP | Internet Protocol |
| NAT | Network address translation |
| OS | Operating System |
| PGP | Pretty Good Privacy |
| RAM | Random Access Memory |
| SATA | Serial AT Attachment |
| SBC | Single-Board Computer |
| SSD | Solid Disk Drive |
| STAN Session Traversal Utilities for NAT |
| TURN | Traversal Using Relays around NAT |