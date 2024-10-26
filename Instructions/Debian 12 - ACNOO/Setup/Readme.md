Apache coturn Nextcloud ONLYOFFICE OpenVPN / Acronym "ACNOO" or just "ao".

# Introduction

You are a Linux-User with some basic knowledge, on terminal session based interaction with a server?
You would like to run your own Nextcloud server?
This is the right guide for you, to setup a server with Nextcloud, ONLYOFFICE and OpenVPN at home.

The server will be made accessible from the Internet, however the access gets restricted at the frontline of defense by the Web-Server Apache. This will be achieved by using Client-Certificates, means no Client-Certificate ⇒ no access at all.

This approach has some limitations, as not all Nextcloud related Apps do fully support Client-Certificates. If Client-Certificate is supported, the current implementation does expect that at least the first communication can be done without a Client-Certificate. Which of course isn't the case.

On the other hand, the Web-Interface of Nextcloud and ONLYOFFICE will be accessible - through the Internet - on those devices with the Client-Certificate installed.

I went through several iterations, of setting up, testing, evaluating. Checked out the different solutions/software/technologies, like Collabora, Let's Encrypt, nginx and so on.

- Collabora: The License of the "Development Edition" does not fit to my planned usage. The "Try the development edition for free" is basically what license text says. No, thanks.
- Let's Encrypt: I do have to deploy the client certificates, together with my own CA Certificate, anyway ... so there is no need to have the Server-Certificate signed by an official CA.
- nginx: I didn't like the way of configuration. Sorry, I had already experiences with Apache from the past. And to be honest, nginx doesn't come with the same reputation as Apache. Also in this case, the licensing model ... not impressed, not interested at all.

At the end, I decided to go with the following "Basis" setup.

## Basis

- Connectivity
	- DSL
		- Download: 50 MBit/s max.
		- Upload: 20 MBit/s max.
	- DSL-Router
		- Support DDNS: ✓
		- Support Port Forwarding: ✓
	- Wireless Router
		- OS: OpenWrt
	- DDNS Service Provider: *your choice*
- Hardware
	- CPU: Intel Core i5
	- RAM: 8GB DDR3
	- SSD: 128GB SATA
- Software
	- OS: Debian 12 "bookworm"
		- DBMS: PostgreSQL 15 (via Debian)
		- In-Memory Storage System: Redis/7.0.15-1 (via Debian)
		- Office Application
			- Nextcloud/29.0.5 (via GitHub - Nextcloud)
			- ONLYOFFICE/8.1.1-26 (via ONLYOFFICE Repository)
		- STAN Server: coturn/4.6.1-1 (via Debian)
		- TURN Server: coturn/4.6.1-1 (via Debian)
		- Web-Server: Apache/2.4.61 (via Debian)

## Used parameters/attributes/configuration

### General

Independent of the instance.

- Users
	- User: `aosysuserao`
	- Email: `aoadmin@addressao`
	- Phone Region: `aophoneregionao`
- Router
	-  Local IP address: `aorouteraddressao` 
- Host
	- Host name: `nchostnc`
- Web-Server
	- Full Qualified Domain Name (FQDN) `aodomainao`
	- SSL via own Certificate Authority (CA)
- PostgreSQL
	- Host name: `ncpostgresqlnc`
	- Database directory: `/var/lib/postgresql/15/main`
	- Network port: `5432`

### Active Instance

The active Instance of the Office Solution.

- PostgreSQL
- Redis
	- Nextcloud DB Index: ncredisdbindexnc
- Nextcloud
	- host name `nchostnc`
	- local directory `/var/www/ncdirectorync`
	- data directory `/var/lib/nextcloud/ncdatadirectorync`
	- database name `ncdatabasenc`
	- database owner `ncdbonc`
	- database owners password `ncdbopasswordnc`
	- URL `https://aodomainao`
	- administrator `ncadminnc`
	- administrator password `ncadminpasswordnc`
- ONLYOFFICE
	- host name `oohostoo`
	- local directory `/var/www/onlyoffice`
	- database name `oodatabaseoo`
	- database owner `oodbooo`
	- database owners password `oodbopasswordoo`
	- public server name `ooserveroo`
	- URL: `https://ooserveroo.aodomainao`
- coTURN
	- host name `cthostct`
	- database name `ctdatabasect`
	- database owner `ctdboct`
	- database owners password `ctdbopasswordct`
	- public server name `ctserverct`
	- URL: `https://ctserverct.aodomainao`

### Backup Instance

The active Instance of the Office Solution.

*left out this part ... at least for the moment, this aspect is not covered ..*

# DDNS Configuration

First thing, do register your DSL-Router under a URL of your choice. There are several service provider for this kind of DNS.

1. Register yourself at a service provider for DDNS.
2. At the DDNS Service Provider,
	1. reserve the `aodomainao` domain.
	2. add two CNAME entries to your domain.
		- for ONLYOFFICE `ooserveroo`
		- for coturn (STUN/TURN) `ctserverct`
3. Configure your DSL-Router, to daily announce its Internet IP address to the DDNS Service Provider.

## DSL-Router

### Configuration: DDNS

The location of the configuration depends on the Router model. For mine, I found the configuration at "Internet > Dynamic DNS".

- 🪜 navigate to the DDNS settings
- ✔️✏️ tick the checkbox for "**Use dynamic DNS**"
- ✔️✏️ for "**Provider**" select "**Other provider**"
- ✔️✏️ enter the following information
	- Domain name: `aodomainao`
	- User name: `<account login for DDNS-Provider>`
	- Password: `<account password for DDNS-Provider>`
	- Updateserver-Address: `<check documentation of DDNS-Provider>`
	- Protocol: `HTTPS`
	- Port: `443`
- ✔️ click on "**Save**"
- 🛝 reboot the router

# Certificates

An essential part of this server setup is the usage of certificates. Therefore the creation of the same, should be done right at the beginning.

In order to learn the process of revoking a Client-Certificate, in case of a stolen device, do all the steps up to the end of this section.

## Files

At the end you will have the following files.

| file name | purpose |
| :--- | :--- |
| `own_ca.crl` | Certificate Revocation List (CRL) of own_ca |
| `own_ca.crt` | Certificate of own_ca |
| `own_ca.key` | Key of own_ca |
| `nchostnc/nchostnc.crt` | to enable and authenticate the Web-Server locally |
| `nchostnc/nchostnc.key` | / |
| `nchostnc/aodomainao.crt` | to enable and authenticate the Web-Server Internet |
| `nchostnc/aodomainao.key` | / |
| `nchostnc/YYYYMMDD.bak/client A.crt` | certificate and key for Client Device A |
| `nchostnc/YYYYMMDD.bak/client A.key` | / which certificate gets revoked |
| `nchostnc/YYYYMMDD.bak/client A.pfx` | PKCS#12 format of certificate for Client Device A |
| `nchostnc/client A.crt` | certificate and key for Client Device A |
| `nchostnc/client A.key` | / which certificate gets renewed |
| `nchostnc/client A.pfx` | PKCS#12 format of certificate for Client Device A |

As your CA-Certificate gets to deployed on the client device anyway, means it is unnecessary to have the Web-Server-Certificates created/signed by an official CA.

## OpenSSL for Web-Server & -Client-Authentication

The following has to be done on a different system than your server! However certain files have to be - later - transferred on to the server.

You are about to create a CA, Under no circumstances the CA files (`own_ca.key` & `own_ca.crt`) should get lost, or even worse, get stolen.

### Pass Phrase & Password

Make sure that you write down this information and do not loose it ... like with the CA files, the information is important/critical.

#### pass phrase of CA key file

During the first steps (becoming CA), you will have to enter a pass phrase for the CA key. This is indicated by `Enter PEM pass phrase:`  and `Verifying - Enter PEM pass phrase:`. Than later and each time a Certificate Request (`.csr`) has to be signed, you are asked to enter the exact same pass phrase as specified during the first steps (creating your CA). 

#### export/import password

The Client-Certificate for the initial user gets created and has to have a password. This is indicated by `Enter Export Password:`  and `Verifying - Enter Export Password:`. The so called Export-/Import-Password is later required when importing the Client-Certificate onto the client device.

### Preparation

A small directory structure needs to be created.

- the main directory `SSL.own_ca` will contain everything of your ca
- the config & data directory `coda` 
- the directory `nchostnc` for all file related to this server

In addition several files in `coda` are required for using OpenSSL properly, one of which you can create immediately from command line.

```console
cd ~
mkdir SSL.own_ca
cd SSL.own_ca
mkdir --parents coda/certs coda/crl coda/newcerts
touch coda/index.txt coda/crl.pem
echo 50 > coda/serial
echo 01 > coda/crlnumber
cat << EOF > coda/env.cnf
dir           = `pwd`/coda                  # Where everything is kept
certs         = \$dir/certs            # Where the issued certs are kept
crl_dir       = \$dir/crl              # Where the issued crl are kept
database      = \$dir/index.txt        # database index file.
new_certs_dir = \$dir/newcerts         # default place for new certs.
serial        = \$dir/serial           # The current serial number
crlnumber     = \$dir/crlnumber        # the current crl number
crl           = \$dir/crl.pem          # The current CRL
EOF
mkdir nchostnc
```

The newly created directory `~/SSL.own_ca/nchostnc` is now prepared. Certain files got created and the reference to those is stored in the file `env.cnf`, which gets included by all the configuration files.

In the new directory `SSL.own_ca/nchostnc`, create the following configuration files (see below).

| file  | used for creating |
| :--- | :--- |
| `coda/ca.cnf` | Certificate Authority (CA) |
| `coda/client.cnf` | Client-Certificate request |
| `coda/server.cnf` | Server-Certificate request |
| `coda/ca_client.cnf` | Client-Certificate |
| `coda/ca_server.cnf` | Server-Certificate |
| `nchostnc/ext_client.cnf` | Client-Certificate |
| `nchostnc/ext_server_domain.cnf` | Server-Certificate for nchostnc |
| `nchostnc/ext_server_local.cnf` | Server-Certificate for aodomainao |

In section `req_distinguished_name`  make sure to update the default information, to fit to your needs.

=== `env.cnf` ===

```code
dir           = ./coda                     # Where everything is kept
certs         = $dir/coda/certs            # Where the issued certs are kept
crl_dir       = $dir/coda/crl              # Where the issued crl are kept
database      = $dir/coda/index.txt        # database index file.
new_certs_dir = $dir/coda/newcerts         # default place for new certs.
serial        = $dir/coda/serial           # The current serial number
crlnumber     = $dir/coda/crlnumber        # the current crl number
crl           = $dir/coda/crl.pem          # The current CRL
```

=== `coda/ca.cnf` ===

```code
[ req ] ############
prompt = no

default_bits       = 4096
distinguished_name = req_distinguished_name
attributes         = req_attributes
x509_extensions    = v3_ca
string_mask        = utf8only
policy             = policy_anything

[ req_distinguished_name ] ############
countryName                    = DE
stateOrProvinceName            = Berlin
localityName                   = Berlin
0.organizationName             = Beispiel GmbH
organizationalUnitName         = Certificate Authority for Beispiel GmbH
commonName                     = CA Beispiel GmbH

[ req_attributes ] ############

[ policy_anything ] ############
countryName            = optional
stateOrProvinceName    = optional
localityName           = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ v3_ca ] ############
basicConstraints       = critical,CA:true
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid:always,issuer
keyUsage               = critical, keyCertSign, cRLSign
```

===`coda/ca_client.cnf` ===

```code
[ ca ] ############
default_ca = CA_default        # The default ca section

[ CA_default ] ############
.include coda/env.cnf

certificate      = own_ca.crt
private_key      = own_ca.key

default_crl_days = 30                 # how long before next CRL
default_md       = sha256

policy           = policy_match       # match of country state organization
email_in_dn      = no                 # Email was required for the request, but no longer needed.

name_opt         = ca_default         # Subject Name options
cert_opt         = ca_default         # Certificate field options
x509_extensions  = usr_cert           # The extensions to add to the cert
preserve         = no                 # keep passed DN ordering

[ policy_match ] ############
countryName            = match
stateOrProvinceName    = match
organizationName       = match
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ usr_cert ] ############
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, clientAuth
nameConstraints        = critical, permitted;DNS:nchostnc, permitted;DNS:aodomainao, permitted;DNS:.aodomainao
```

=== `coda/ca_server.cnf` ===

```code
[ ca ] ############
default_ca = CA_default        # The default ca section

[ CA_default ] ############
.include coda/env.cnf

certificate      = own_ca.crt
private_key      = own_ca.key

default_md       = sha256

policy           = policy_match       # match of country state organization
email_in_dn      = no                 # Email was required for the request, but no longer needed.

name_opt         = ca_default         # Subject Name options
cert_opt         = ca_default         # Certificate field options
preserve         = no                 # keep passed DN ordering
x509_extensions  = usr_cert           # The extensions to add to the cert

[ policy_match ] ############
countryName            = match
stateOrProvinceName    = match
organizationName       = match
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional

[ usr_cert ] ############
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, serverAuth
subjectAltName         = DNS:localhost,DNS:nchostnc,DNS:aodomainao,DNS:ooserveroo.aodomainao,DNS:ctserverct.aodomainao
```

=== `coda/client.cnf` ===

```code
[ req ] ############
prompt = yes

default_bits       = 4096
distinguished_name = req_distinguished_name
string_mask        = utf8only

[ req_distinguished_name ] ############
countryName                    = Country Name (2 letter code)
countryName_default            = DE
countryName_min                = 2
countryName_max                = 2
stateOrProvinceName            = State or Province Name (full name)
stateOrProvinceName_default    = Berlin
localityName                   = Locality Name (eg, city)
localityName_default           = Berlin
0.organizationName             = Organization Name (eg, company)
0.organizationName_default     = Beispiel GmbH
organizationalUnitName         = Organizational Unit Name (eg, section)
organizationalUnitName_default = 
commonName                     = Common Name (of User)
commonName_max                 = 64
emailAddress                   = Email Address (of User)
emailAddress_default           = aoadmin@addressao
emailAddress_max               = 64
```

=== `coda/server.cnf` ===

```code
[ req ] ############
prompt             = yes
default_bits       = 4096
distinguished_name = req_distinguished_name
string_mask        = utf8only

[ req_distinguished_name ] ############
countryName                    = Country Name (2 letter code)
countryName_default            = DE
countryName_min                = 2
countryName_max                = 2
stateOrProvinceName            = State or Province Name (full name)
stateOrProvinceName_default    = Berlin
localityName                   = Locality Name (eg, city)
localityName_default           = Berlin
0.organizationName             = Organization Name (eg, company)
0.organizationName_default     = Beispiel GmbH
organizationalUnitName         = Organizational Unit Name (eg, section)
organizationalUnitName_default = 
commonName                     = Common Name (of Server)
commonName_max                 = 64
emailAddress                   = Email Address (of User)
emailAddress_default           = aoadmin@addressao
emailAddress_max               = 64
```

=== `nchostnc/ext_client.cnf` ===

```code
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, clientAuth
nameConstraints        = critical, permitted;DNS:nchostnc, permitted;DNS:aodomainao, permitted;DNS:.aodomainao
```

=== `nchostnc/ext_server_domain.cnf` ===

```code
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, serverAuth
subjectAltName         = DNS:aodomainao,DNS:*.aodomainao
```

=== `nchostnc/ext_server_local.cnf` ===

```code
basicConstraints       = critical, CA:FALSE
subjectKeyIdentifier   = hash
authorityKeyIdentifier = keyid,issuer
keyUsage               = critical, digitalSignature, keyEncipherment, keyAgreement
extendedKeyUsage       = critical, serverAuth
subjectAltName         = DNS:localhost,DNS:nchostnc
```

### Keys of Certificate

```console
openssl rsa -check -in /etc/letsencrypt/live/aodomainao/privkey.pem

```

### CA-Certificate

To become a technical CA, the first steps are to create the key and create your own certificate.

```console
# cd ~/SSL.own_ca
# openssl genrsa -utf8 -aes256 -out own_ca.key 4096
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
# openssl req -config coda/ca.cnf -days 3650 -extensions v3_ca -new -x509 -key own_ca.key -out own_ca.crt
Enter pass phrase for own_ca.key:
```

### Server-Certificate

More precisely, creating two kinds of Server-Certificates, one for the local network communication and the other via the Internet.

Server-Certificate for access the Web-Server inside the local network or the host itself. Addressing the server by the names `localhost` or `nchostnc`.

```console
openssl genrsa -utf8 -aes256 -passout pass:secret -out nchostnc/nchostnc-pass.key 4096
openssl rsa -passin pass:secret -in nchostnc/nchostnc-pass.key -out nchostnc/nchostnc.key
openssl req -config coda/server.cnf -new -key nchostnc/nchostnc.key -out nchostnc/nchostnc.csr
openssl ca -config coda/ca_server.cnf -extfile nchostnc/ext_server_local.cnf -days 365 -out nchostnc/nchostnc.crt -infiles nchostnc/nchostnc.csr
```

Server-Certificate for access the Web-Server from the Internet. Addressing the server by the names `aodomainao` or `<something>.aodomainao`.

```console
openssl genrsa -utf8 -aes256 -passout pass:secret -out nchostnc/aodomainao-pass.key 4096
openssl rsa -passin pass:secret -in nchostnc/aodomainao-pass.key -out nchostnc/aodomainao.key
openssl req -config coda/server.cnf -new -key nchostnc/aodomainao.key -out nchostnc/aodomainao.csr
openssl ca -config coda/ca_server.cnf -extfile nchostnc/ext_server_domain.cnf -days 365 -out nchostnc/aodomainao.crt -infiles nchostnc/aodomainao.csr
```

### Client-Certificate

Note: Each user should have for every device an individual Client-Certificate.
Note: Even PHP is acting as a user and as such requires a Client-Certificate.

```console
openssl genrsa -utf8 -aes256 -passout pass:secret -out "nchostnc/client-pass.key" 4096
openssl rsa -passin pass:secret -in "nchostnc/client-pass.key" -out "nchostnc/client A.key"
openssl req -config coda/client.cnf -addext "subjectAltName = email:myEmail@email.com" -new -key "nchostnc/client A.key" -out "nchostnc/client A.csr"
openssl ca -config coda/ca_client.cnf  -extfile nchostnc/ext_client.cnf -days 365 -out "nchostnc/client A.crt" -infiles "nchostnc/client A.csr"
openssl pkcs12 -export -out "nchostnc/client A.pfx" -inkey "nchostnc/client A.key" -in "nchostnc/client A.crt" -certfile "own_ca.crt"
```

In case of using an iOS device as client, the PKCS#12-Certificate has to be created using the `-legacy` parameter.

```
openssl pkcs12 -export -legacy -out "nchostnc/client A.pfx" -inkey "nchostnc/client A.key" -in "nchostnc/client A.crt" -certfile own_ca.crt
```

### Revoke Client-Certificate

Step to revoke a Client-Certificate. The crl information gets added to the file `own_ca.crl`.

```console
openssl ca -config coda/ca_client.cnf -revoke "nchostnc/client A.crt"
openssl ca -config coda/ca_client.cnf -gencrl -out "own_ca_$(date +'%Y%m%d') client A.crt.crl"
cat "own_ca_$(date +'%Y%m%d') client A.crt.crl" >> own_ca.crl
```

When looking at the Certificate Revocation List, the most import parts are the date/time and serial number of the original certificate.

```console
# openssl crl -text -in 'own_ca_20240922 client A.crt.crl'
:
Revoked Certificates:
    Serial Number: 51
        Revocation Date: Sep 22 17:47:51 2024 GMT
:
```

In case you would have forgotten the related certificate, based on this example look into the certificate `51.pem` by executing command `openssl x509 -text -noout -in coda/newcerts/51.pem`

=== `own_ca_20240922 client A.crt.crl` ===

```code
-----BEGIN X509 CRL-----
MIIDBDCB7QIBATANBgkqhkiG9w0BAQsFADCBlDELMAkGA1UEBhMCREUxDzANBgNV
:
65dbmB4e2iU=
-----END X509 CRL-----
```

### Renew Client-Certificate

Keep the original files, that will be used for one of our tests.

```console
mkdir nchostnc/$(date +'%Y%m%d').bak
mv nchostnc/client\ A.* nchostnc/$(date +'%Y%m%d').bak
```

Like the initial creation of the Client-Certificate.

```console
openssl genrsa -utf8 -aes256 -passout pass:secret -out "nchostnc/client-pass.key" 4096
openssl rsa -passin pass:secret -in "nchostnc/client-pass.key" -out "nchostnc/client A.key"
openssl req -config coda/client.cnf -addext "subjectAltName = email:myEmail@email.com" -new -key "nchostnc/client A.key" -out "nchostnc/client A.csr"
openssl ca -config coda/ca_client.cnf  -extfile nchostnc/ext_client.cnf -days 365 -out "nchostnc/client A.crt" -infiles "nchostnc/client A.csr"
openssl pkcs12 -export -out "nchostnc/client A.pfx" -inkey "nchostnc/client A.key" -in "nchostnc/client A.crt" -certfile "own_ca.crt"
```

# Server Installation

## OS installation

Have the image for Debian 12 ready on an USB Stick.

1. insert USB Stick
2. boot machine (ensure the machine will boot from the USB Stick)
3. select "Installation" from the GRUB boot menu
4. dialog "Select a language"
	- select "English"
5. dialog "Select your location
	- select "other > Europe > Germany
6. dialog "Configure locales
	- select "United States"
7. dialog "Configure the keyboard"
	- select "German"
8. dialog "Configure the network"
	- select "eno1: Intel Corporation Ethernet Connection I217-V"
	- specify "Hostname"
	- set "Domain name" to be empty
9. dialog "Set up users and passwords"
	- specify root password (note it down)
10. dialog "Set up users and passwords"
	- specify "Full name for the new user"
	- specify "Username for your account"
	- specify "Choose a password for the new user"
11. dialog "Partition disks"
	- select "Manual"
	- under the specific drive, select "FREE SPACE" entry
	- select "Create a new partition"
	- specify "New partition size"
	- select "Primary"
	- specify "Use as" to be "Ext2 file system
	- specify "Format the partition" to be "yes, format it"
	- specify "Mount options" to be "noatime,relatime"
	- specify "Label" to be "Nextcloud"
	- specify "Bootable flag" to be "on"
	- select "Finish partitioning and write changes to disk"
	- select "No" on the question about "swap space"
	- select "Yes" to get the changes written onto the disk
12. dialog "Configure the package manager"
	- select "Germany" as "Debian archive mirror country"
	- select "ftp.gwdg.de" as "Debian archive mirror"
	- specify no "HTTP proxy ..."
13. dialog "Configuring popularity-contest"
	- select "Yes"
14. dialog "Software selection"
	- disable "Debian desktop environment"
	- disable "GNOME"
	- enable "SSH server"
	- select "Continue"
15. dialog "grub-pc"
	- select "Yes" to install GRUB boot loader
	- select drive (used to previously create partition)
16. dialog "Finish the installation"
	- remove USB stick
	- select "Continue"
17. reboot take place, ending with login prompt

From now on, the communication with the server will be done using ssh and http(s).

### Post activities

#### GRUB

Hide the GRUB menu during boot time. If need at a later point in time, keep the SHIFT key press right after start.

```console
cp /etc/default/grub /etc/default/grub.ORG
sed -i -e "s/GRUB_TIMEOUT=5/GRUB_TIMEOUT=0\nGRUB_TIMEOUT_STYLE=hidden/" /etc/default/grub
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

##### Client-Keys

Like for the CA files ... do not loose the Client-Keys. Otherwise you do have to get physical access to the server `nchostnc`, in order to login in again.

The following steps have to be done on the machine, used to remotely login to the server `nchost`.

###### As regular user

In case you no ssh-key pair ... create it using the following command (and press ENTER when asked for passphrase)

```console
$ ls ~/.ssh
known_hosts  known_hosts.old
$ ssh-keygen -t rsa -b 4096 -C "aosysuserao" -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/aosysuserao/.ssh/id_rsa
Your public key has been saved in /home/aosysuserao/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:z5s8Vby75poeVE4YfRK3jnAZ/RFaHjMPIn5fLRj2xZI aosysuserao
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

Copy the key to the `nchostnc` server.

```console
$ ssh-copy-id aosysuserao@nchostnc
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/aosysuserao/.ssh/id_rsa.pub"
The authenticity of host 'nchostnc (10.0.0.20)' can't be established.
ED25519 key fingerprint is SHA256:td4yIx6Ap/66c5rweUl/HxGC0W1gLww3rwdoHeR9Ang.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
aosysuserao@nchostnc's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'aosysuserao@nchostnc'"
and check to make sure that only the key(s) you wanted were added.
```

Check out if the login works, using the key.

```console
$ ssh aosysuserao@nchostnc
Linux nchostnc 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
aosysuserao@nchostnc:~$ 
```

##### Configure disable Password usage
PasswordAuthentication no
ChallengeResponseAuthentication no
UsePAM no
To be done on the server `nchostnc`.

1. create a backup of the current configuration, using command `cp /etc/ssh/sshd_config /etc/ssh/sshd_config.ORG`
2. modify `sshd_config` by setting the following parameters with value `no`
	- PasswordAuthentication
	- ChallengeResponseAuthentication
	- UsePAM
3. restart the ssh service, using command `systemctl restart ssh`

===`/etc/ssh/sshd_config` ===

```code
:

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no

:

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the KbdInteractiveAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via KbdInteractiveAuthentication may bypass
# the setting of "PermitRootLogin prohibit-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and KbdInteractiveAuthentication to 'no'.
UsePAM yes

:

ChallengeResponseAuthentication no
```

##### Test as root

Try out the root user ... you will not get in.

```console
# ssh nchostnc
The authenticity of host 'nchostnc (10.0.0.20)' can't be established.
ED25519 key fingerprint is SHA256:td4yIx6Ap/66c5rweUl/HxGC0W1gLww3rwdoHeR9Ang.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'nchostnc' (ED25519) to the list of known hosts.
root@nchostnc: Permission denied (publickey).
# ssh aosysuserao@nchostnc
aosysuserao@nchostnc: Permission denied (publickey).
```

#### SSD

To reduce wear of the SSD, the disk already gets mounted with specific parameter that reduce write operations. In addition, some information are temporary and might not be required to be stored on disk and therefore RAM-Disk directories will be prepared.

##### RAM-Disk

Create some OS folders as RAM-Disk, using the following

```code
cat << EOF >> /etc/fstab

# Temporary files onto tmpfs
none /tmp tmpfs defaults,noatime,mode=1777 0 0
none /var/tmp tmpfs defaults,noatime 0 0
none /var/log tmpfs defaults,noatime 0 0
EOF
```

=== `/etc/fstab` ===

```code
:

# Temporary files onto tmpfs
none /tmp tmpfs defaults,noatime,mode=1777 0 0
none /var/tmp tmpfs defaults,noatime 0 0
none /var/log tmpfs defaults,noatime 0 0
```

Note: The directory `/run` is a tmpfs and available Debian 10 and higher.

###### `/var/log` folders

The installed software expects to have always the needed log-directories accessible for its account. This can be achieved by making use of  `/etc/tmpfiles.d/log-subfolder.conf`. Once done, reboot the server and the folders get created.

```code
cat << EOF >> /etc/tmpfiles.d/log-subfolder.conf
d       /var/log/nextcloud                            0755 www-data www-data - -
d       /var/log/apache2                              0755 www-data www-data - -
d       /var/log/nginx                                0755 www-data www-data - -
d       /var/log/onlyoffice                           0755 ds ds - -
d       /var/log/onlyoffice/documentserver            0755 ds ds - -
d       /var/log/onlyoffice/documentserver/metrics    0755 ds ds - -
d       /var/log/onlyoffice/documentserver/converter  0755 ds ds - -
d       /var/log/onlyoffice/documentserver/docservice 0755 ds ds - -
d       /var/log/onlyoffice/documentserver-example    0755 ds ds - -
d       /var/log/openvpn                              0755 root root - -
d       /var/log/rabbitmq                             0755 rabbitmq rabbitmq - -
d       /var/log/redis                                0755 redis redis - -
d       /var/log/turn                                 0755 turnserver turnserver - -
EOF
```

=== `/etc/tmpfiles.d/log-subfolder.conf` ===

```code
d       /var/log/nextcloud                            0755 www-data www-data - -
d       /var/log/apache2                              0755 www-data www-data - -
d       /var/log/nginx                                0755 www-data www-data - -
d       /var/log/onlyoffice                           0755 ds ds - -
d       /var/log/onlyoffice/documentserver            0755 ds ds - -
d       /var/log/onlyoffice/documentserver/metrics    0755 ds ds - -
d       /var/log/onlyoffice/documentserver/converter  0755 ds ds - -
d       /var/log/onlyoffice/documentserver/docservice 0755 ds ds - -
d       /var/log/onlyoffice/documentserver-example    0755 ds ds - -
d       /var/log/openvpn                              0755 root root - -
d       /var/log/rabbitmq                             0755 rabbitmq rabbitmq - -
d       /var/log/redis                                0755 redis redis - -
d       /var/log/turn                                 0755 turnserver turnserver - -
```

#### Locales

Add additional locals, by starting `dpkg-reconfigure locales` and selecting the local, in addition to the existing one.

#### Tools

```console
apt install -y curl git gpg sudo unzip
```

#### Finish OS Setup

In order to find your system IP address based on the MAC-Address, checkout command `ip a`.

- shutdown the system
- detach monitor and keyboard
- start the system
- ensure the system can be still reached via ssh
- shutdown the system
- put it at its place ... in the DMZ
- start the system

## Nextcloud Prerequisites

### OS

#### Repositories

Add `contrib` and `non-free` area to the Repository.

```console
sed -i 's/main non-free-firmware/main contrib non-free non-free-firmware/g' /etc/apt/sources.list
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

Add repository for ONLYOFFICE.

```console
mkdir -p -m 700 ~/.gnupg
curl -fsSL https://download.onlyoffice.com/GPG-KEY-ONLYOFFICE | gpg --no-default-keyring --keyring gnupg-ring:/tmp/onlyoffice.gpg --import
chmod 644 /tmp/onlyoffice.gpg
mv /tmp/onlyoffice.gpg /usr/share/keyrings/onlyoffice.gpg
echo "deb [signed-by=/usr/share/keyrings/onlyoffice.gpg] https://download.onlyoffice.com/repo/debian squeeze main" | tee /etc/apt/sources.list.d/onlyoffice.list

apt update
```

#### Software

Install more of the required software. 
During the install process a dialog pop-up, asking for information, in regards to the ONLYOFFICE database ... press ENTER to continue. As a result the installation ends with an error, but that's fine ... this will be handled later.

Note: Due to some dependencies, two Web-Server (apache & nginx) will be installed. That's fine ... we will handle that aspect later.

```console
apt install -y imagemagick libapache2-mod-php php php-apcu php-bcmath php-bz2 php-cli php-common php-curl php-dev php-dompdf php-fpm php-gd php-gmp php-imagick php-intl php-json php-mbstring php-pear php-pgsql php-redis php-soap php-xml php-zip postgresql redis ttf-mscorefonts-installer rabbitmq-server onlyoffice-documentserver coturn
Errors were encountered while processing:
 onlyoffice-documentserver
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

#### Certificates & Co.

First we will have to create a directory for the CRLs.

```console
mkdir /etc/ssl/crl
```

Get the following files, created in section "Certificates Creation", onto this machine.

```console
/etc/ssl/certs/own_ca.crt
/etc/ssl/certs/nchostnc.crt
/etc/ssl/certs/aodomainao.crt

/etc/ssl/private/nchostnc.key
/etc/ssl/private/aodomainao.key

/etc/ssl/crl/own_ca.crl
```

Add your own ca to the list of Certificate Authorities of the system (incl. Mono environment)

```console
mkdir /usr/local/share/ca-certificates/my-custom-ca
chmod 700 /usr/local/share/ca-certificates/my-custom-ca
ln -s /etc/ssl/certs/own_ca.crt /usr/local/share/ca-certificates/my-custom-ca
update-ca-certificates
```

### Nginx

Even though it is not required and used, for this server, it gets installed as dependency of ONLYOFFICE. We just have to "disable" it.

#### "Disable" nginx

Disable the include statements in the nginx configuration. This way, the service starts, but does not listen on any ports. In a later step the service of nginx can be disabled.

```console
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.ORG
sed -i -e 's/^\tinclude \/etc\/nginx\/conf.d\/\*\.conf;/\t#include \/etc\/nginx\/conf.d\/\*\.conf;/' -e 's/\tinclude \/etc\/nginx\/sites-enabled\/\*;/\t#include \/etc\/nginx\/sites-enabled\/\*;/' /etc/nginx/nginx.conf
```

=== `/etc/nginx/nginx.conf` ===

```code
:

        ##
        # Virtual Host Configs
        ##

        #include /etc/nginx/conf.d/*.conf;
        #include /etc/nginx/sites-enabled/*;
}

:
```

After the change, restart the service.

```console
systemctl restart nginx
```

Optional, the nginx Web-Server can be disabled.

```console
# systemctl stop nginx
# systemctl disable nginx
Synchronizing state of nginx.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install disable nginx
Removed "/etc/systemd/system/multi-user.target.wants/nginx.service".
```

### Script-Runtime - PHP

#### `php.ini` modifications

Check the existance of the `php.ini` files.

```console
# find / -name php.ini
/etc/php/8.2/apache2/php.ini
/etc/php/8.2/fpm/php.ini
/etc/php/8.2/cli/php.ini
```

Make the modification either automatically or manually. However at the end the changes have to look like the following.

```console
# grep -e "^output_buffering" -e "^max_execution_time" -e "^post_max_size" -e "^upload_max_filesize" -e "^date.timezone" -e "^opcache.enable" -e "^opcache.memory_consumption" -e "^opcache.interned_strings_buffer" -e "^opcache.max_accelerated_files" -e "^opcache.revalidate_freq" -e "^opcache.save_comments" -e "^\[APCu\]" -e "^apc.enable_cli" /etc/php/8.2/cli/php.ini
output_buffering = Off
max_execution_time = 300
post_max_size = 600M
upload_max_filesize = 500M
date.timezone = Europe/Amsterdam
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.revalidate_freq=1
opcache.save_comments=1
[APCu]
apc.enable_cli=1
# 
```

##### Automated

The easiest way is to, to create file `php.ini.diff` and patch all three `php.ini` files.

```console
cp /etc/php/8.2/apache2/php.ini /etc/php/8.2/apache2/php.ini.ORG
cp /etc/php/8.2/fpm/php.ini /etc/php/8.2/fpm/php.ini.ORG
cp /etc/php/8.2/cli/php.ini /etc/php/8.2/cli/php.ini.ORG
patch /etc/php/8.2/cli/php.ini < php.ini.diff
patch /etc/php/8.2/apache2/php.ini < php.ini.diff
patch /etc/php/8.2/fpm/php.ini < php.ini.diff
```

=== `~/php.ini.diff` ===

```code
226c226
< output_buffering = 4096
---
> output_buffering = Off
409c409
< max_execution_time = 30
---
> max_execution_time = 300
703c703
< post_max_size = 8M
---
> post_max_size = 600M
855c855
< upload_max_filesize = 2M
---
> upload_max_filesize = 500M
979c979
< ;date.timezone =
---
> date.timezone = Europe/Amsterdam
1786c1786
< ;opcache.enable=1
---
> opcache.enable=1
1792c1792
< ;opcache.memory_consumption=128
---
> opcache.memory_consumption=256
1795c1795
< ;opcache.interned_strings_buffer=8
---
> opcache.interned_strings_buffer=8
1799c1799
< ;opcache.max_accelerated_files=10000
---
> opcache.max_accelerated_files=10000
1817c1817
< ;opcache.revalidate_freq=2
---
> opcache.revalidate_freq=1
1824c1824
< ;opcache.save_comments=1
---
> opcache.save_comments=1
1974a1975,1977
> 
> [APCu]
> apc.enable_cli=1
```

##### Manually

Adjust the PHP configuration.

| Property | original | modified |
| ---: | :---: | :---: |
| display_errors | n/a | Off² |
| output_buffering | n/a | Off |
| max_execution_time | 30 | 300 |
| post_max_size | 8M | 600M |
| upload_max_filesize | 2M | 500M |
| zend_extension | n/a | opcache¹ |
| date.timezone | n/a | Europe/Amsterdam |
| opcache.enable | n/a | 1 |
| opcache.memory_consumption | n/a | 256 |
| opcache.interned_strings_buffer | n/a | 8 |
| opcache.max_accelerated_files | n/a | 10000 |
| opcache.revalidate_freq | n/a | 1 |
| opcache.save_comments | n/a | 1 |

¹ *Change not required for Debian.*
² *It is the default value.*
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

The Check: Open the URL "http://nchostnc/info.php". Verify that the most relevant parts are available. See [Installation and server configuration / PHP Modules & Configuration](https://docs.nextcloud.com/server/latest/admin_manual/installation/php_configuration.html)

Note: Do not forget to delete the file, once your installation is running.

### Database - PostgreSQL 15

Default locations
Database `/var/lib/postgresql/15/main`
Configuration `/etc/postgresql/15/main`

1. modify the `postgresql.conf` file
	1. by removing leading "#" of line `listen_addresses = 'localhost'` definition
	2. by changing the socket permission from `0777` to `0770`
2.restart the PostgreSQL service, using command `systemctl restart postgresql`

```console
cp /etc/postgresql/15/main/postgresql.conf /etc/postgresql/15/main/postgresql.conf.org
sed -i -e "s/#listen_addresses = 'localhost'/listen_addresses = 'localhost'/" -e "s/#unix_socket_permissions = 0777/unix_socket_permissions = 0770/" /etc/postgresql/15/main/postgresql.conf
systemctl restart postgresql
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
unix_socket_directories = '/var/run/postgresql' # comma-separated list of directories
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

### Database - Redis

1. modify the `redis.conf` file
	1. by removing leading "#" of line `unixsocket ...` definition
	2. by changing the socket permission from `0700` to `0770`
2.restart the Redis service, using command `systemctl restart redis`

```console
cp /etc/redis/redis.conf /etc/redis/redis.conf.org
sed -i -e "s/^# unixsocket/unixsocket/" -e "s/^# unixsocketperm 700/unixsocketperm 770/" /etc/redis/redis.conf
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

### ONLYOFFICE

#### Create database

```console
sudo --user=postgres psql --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# CREATE USER oodbooo WITH PASSWORD 'oodbopasswordoo';
CREATE ROLE
postgres=# CREATE DATABASE oodatabaseoo OWNER oodbooo;
CREATE DATABASE
postgres=# GRANT ALL privileges ON DATABASE oodatabaseoo TO oodbooo;
postgres=# \q
```

#### Prepare database

```console
# sudo --user=postgres psql --host=localhost --dbname=oodatabaseoo --username=oodbooo --password -f /var/www/onlyoffice/documentserver/server/schema/postgresql/createdb.sql
could not change directory to "/root": Permission denied
Password: 
CREATE TABLE
CREATE TABLE
CREATE FUNCTION
```

#### Adjust debconf parameters

Adjust the ONLYOFFICE install to fit your configuration.

```console
echo onlyoffice-documentserver onlyoffice/db-host string localhost | sudo debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-user string oodbooo | sudo debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-pwd password oodbopasswordoo | debconf-set-selections
echo onlyoffice-documentserver onlyoffice/db-name string oodatabaseoo | sudo debconf-set-selections
```

#### Rerun installation

Run the configuration of the install package. This has two effect.

1. It resolves the issue `E: Sub-process /usr/bin/dpkg returned an error code (1)`
2. Completes the configuration of ONLYOFFICE

```console
# apt install -y onlyoffice-documentserver
:
Congratulations, the ONLYOFFICE documentserver has been installed successfully!

Processing triggers for libc-bin (2.36-9+deb12u8) ...
# 
```

### coturn

#### Create database

```console
sudo --user=postgres psql --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# CREATE USER ctdboct WITH PASSWORD 'ctdbopasswordct';
CREATE ROLE
postgres=# CREATE DATABASE ctdatabasect OWNER ctdboct;
CREATE DATABASE
postgres=# GRANT ALL privileges ON DATABASE ctdatabasect TO ctdboct;
postgres=# \q
```

#### Prepare database

```console
# sudo --user=postgres psql --host=localhost --dbname=ctdatabasect --username=ctdboct --password -f /usr/share/coturn/schema.sql
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
cd /usr/local/etc
ln -s /etc/ssl/certs/aodomainao.crt server.crt
ln -s /etc/ssl/private/aodomainao.key server.key
ln -s /etc/ssl/certs/own_ca.crt ca.crt
cp /etc/turnserver.conf /etc/turnserver.conf.ORG
cp /lib/systemd/system/coturn.service /lib/systemd/system/coturn.service.ORG
sed -i -e 's/turnserver.conf --pidfile=/turnserver.conf --psql-userdb="host=localhost dbname=ctdatabasect user=ctdboct password=ctdbopasswordct connect_timeout=30" --pidfile=/' /lib/systemd/system/coturn.service
cat << EOF > /etc/turnserver.conf
log-file=/var/log/turn/server.log
verbose

realm=aodomainao
server-name=turn.aodomainao
listening-ip=0.0.0.0
listening-port=3478
tls-listening-port=5349
min-port=10000
max-port=20000

CA-file=/usr/local/etc/own_ca.crt
cert=/usr/local/etc/nchostnc.crt
pkey=/usr/local/etc/nchostnc.key
cipher-list="ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384"
no-loopback-peers
no-multicast-peers
no-tlsv1
no-tlsv1_1
no-stdout-log

fingerprint
use-auth-secret
static-auth-secret=215f6146b6ed6446bd6c093af4911b34ba1b3e36bc3b0dbc662384e29edd6ee6
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
verbose

realm=aodomainao
server-name=ctserverct.aodomainao
listening-ip=0.0.0.0
listening-port=3478
tls-listening-port=5349
min-port=10000
max-port=20000

CA-file=/usr/local/etc/ca.crt
cert=/usr/local/etc/server.crt
pkey=/usr/local/etc/server.key
cipher-list="ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384"
no-loopback-peers
no-multicast-peers
no-tlsv1
no-tlsv1_1
no-stdout-log

fingerprint
use-auth-secret
static-auth-secret=<your secret>
total-quota=100
bps-capacity=0
stale-nonce=600
```

=== `/lib/systemd/system/coturn.service` ===

```code
:
ExecStart=/usr/bin/turnserver -c /etc/turnserver.conf --psql-userdb="host=localhost dbname=ctdatabasect user=ctdboct password=ctdbopasswordct connect_timeout=30" --pidfile=
:
```

### Web-Server - Apache2

#### Hide Server Information

Adjust the settings for `ServerTokens` and `ServerSignature` to hide information about service Web-Server and underlying OS.

```console
cp /etc/apache2/conf-enabled/security.conf /etc/apache2/conf-enabled/security.conf.org
sed -i -e "s/^ServerTokens OS/#ServerTokens OS/" -e "s/^#ServerTokens Full/ServerTokens Full/" /etc/apache2/conf-enabled/security.conf
```

=== `/etc/apache2/conf-enabled/security.conf` ===

```code
:

#
# ServerTokens
# This directive configures what you return as the Server HTTP response
# Header. The default is 'Full' which sends information about the OS-Type
# and compiled in modules.
# Set to one of:  Full | OS | Minimal | Minor | Major | Prod
# where Full conveys the most information, and Prod the least.
#ServerTokens Minimal
#ServerTokens OS
#ServerTokens Full
ServerTokens Prod

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

Now over to the configuration...
Start by doing the following steps

1. stop the Web-Server
2. delete the `index.*` files from apache and nginx
3. enable  the required modules
4. add user `www-data` to group `redis`
5. remove (but keep) the original the default Web-Site configuration
6. create a stub Web-Site configuration and enable this
7. create an "error" page in case a user access the Web-Server without a valid certitificate
8. create `phpinfo.php` for checking at any time the configuration of php
9. keep the Web-Server stopped until Nextcloud is installed
10. create the `html` directories for ONLYOFFICE and coturn

```console
systemctl stop apache2
rm var/www/html/index.*
a2enmod headers proxy proxy_http proxy_wstunnel rewrite ssl
usermod --append --groups redis www-data
cd /etc/apache2/sites-available
mv 000-default.conf 000-default.conf.org
mv default-ssl.conf default-ssl.conf.org
touch any.conf
cd ../sites-enabled/
rm 000-default.conf
ln -s ../sites-available/any.conf any.conf
mkdir /var/www/html.ooserveroo
mkdir /var/www/html.ctserverct
```

Create all files for the Web-Sites served by the Apache Web-Server.

=== `/etc/apache2/sites-available/any.conf` ===

```code
ServerName nchostnc

<VirtualHost *:80>
  ServerName nchostnc
  ServerName localhost
  DocumentRoot "/var/www/ncdirectorync"

  CustomLog /var/log/apache2/nchostnc-access-http.log combined
  ErrorLog /var/log/apache2/nchostnc-error-http.log
  LogLevel warn

Include /etc/apache2/sites-available/nextcloud-include.conf
</VirtualHost>

<VirtualHost *:443>
  ServerName nchostnc
  DocumentRoot "/var/www/ncdirectorync"

  SSLCertificateFile /etc/ssl/certs/nchostnc.crt
  SSLCertificateKeyFile /etc/ssl/private/nchostnc.key
Include /etc/apache2/sites-available/ssl-include.conf

  CustomLog /var/log/apache2/nchostnc-access-https.log "%t %h Issuer: %{SSL_CLIENT_I_DN}x Client: %{SSL_CLIENT_S_DN}x \"%r\" %b"
  ErrorLog /var/log/apache2/nchostnc-error-https.log
  LogLevel warn

Include /etc/apache2/sites-available/nextcloud-include.conf
</VirtualHost>

<VirtualHost *:80>
  ServerName aodomainao
  DocumentRoot "/var/www/html"
  Redirect permanent / https://aodomainao/
</VirtualHost>

<VirtualHost *:443>
  ServerName aodomainao
  DocumentRoot "/var/www/ncdirectorync"

  SSLCertificateFile /etc/ssl/certs/aodomainao.crt
  SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
Include /etc/apache2/sites-available/ssl-include.conf

  CustomLog /var/log/apache2/aodomainao-access-https.log "%t %h Issuer: %{SSL_CLIENT_I_DN}x Client: %{SSL_CLIENT_S_DN}x \"%r\" %b"
  ErrorLog /var/log/apache2/aodomainao-error-https.log
  LogLevel warn

Include /etc/apache2/sites-available/nextcloud-include.conf
</VirtualHost>

<VirtualHost *:80>
  ServerName ooserveroo.aodomainao
  DocumentRoot "/var/www/html.ooserveroo"
  Redirect permanent / https://ooserveroo.aodomainao/
</VirtualHost> # ooserveroo.aodomain

<VirtualHost *:443>
  ServerName ooserveroo.aodomainao
  DocumentRoot "/var/www/html.ooserveroo"

  SSLCertificateFile /etc/ssl/certs/aodomainao.crt
  SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
Include /etc/apache2/sites-available/ssl-include.conf

  CustomLog /var/log/apache2/ooserveroo-access-https.log "%t %h Issuer: %{SSL_CLIENT_I_DN}x Client: %{SSL_CLIENT_S_DN}x \"%r\" %b"
  ErrorLog /var/log/apache2/ooserveroo-error-https.log
  LogLevel warn

Include /etc/apache2/sites-available/onlyoffice-include.conf
</VirtualHost>

<VirtualHost *:80>
  ServerName ctserverct.aodomainao
  DocumentRoot "/var/www/html.ctserverct"
  Redirect permanent / https://ctserverct.aodomainao/
</VirtualHost> # ctserverct.aodomainao

<VirtualHost *:443>
  ServerName ctserverct.aodomainao
  DocumentRoot /var/www/html

  SSLCertificateFile /etc/ssl/certs/aodomainao.crt
  SSLCertificateKeyFile /etc/ssl/private/aodomainao.key
Include /etc/apache2/sites-available/ssl-include.conf

  CustomLog /var/log/apache2/ctserverct-access.log "%t %h Issuer: %{SSL_CLIENT_I_DN}x Client: %{SSL_CLIENT_S_DN}x \"%r\" %b"
  ErrorLog /var/log/apache2/ctserverct-error.log
  LogLevel warn

  ProxyPreserveHost On
  ProxyPass / http://127.0.0.1:8080/
  ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```

=== `/etc/apache2/sites-available/coturn-include.conf` ===

```code
  ProxyPreserveHost On
  ProxyPass / http://127.0.0.1:8080/
  ProxyPassReverse / http://127.0.0.1:8080/
```

=== `/etc/apache2/sites-available/nextcloud-include.conf` ===

```code
  Alias "/" "/var/www/ncdirectorync/"

  <IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteRule ^\.well-known/carddav /ncdirectorync/remote.php/dav [R=301,L]
    RewriteRule ^\.well-known/caldav /ncdirectorync/remote.php/dav [R=301,L]
    RewriteRule ^\.well-known/webfinger /ncdirectorync/index.php/.well-known/webfinger [R=301,L]
    RewriteRule ^\.well-known/nodeinfo /ncdirectorync/index.php/.well-known/nodeinfo [R=301,L]
    RewriteRule ^/\.well-known/host-meta /ncdirectorync/public.php?service=host-meta [QSA,L]
    RewriteRule ^/\.well-known/host-meta\.json /ncdirectorync/public.php?service=host-meta-json [QSA,L]
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

  ProxyPass        /push/ws ws://127.0.0.1:7867/ws
  ProxyPass        /push/ http://127.0.0.1:7867/
  ProxyPassReverse /push/ http://127.0.0.1:7867/

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
  RequestHeader setifempty X-Forwarded-Proto https
  RequestHeader setifempty X-Forwarded-Host %{THE_HOST}e
  ProxyAddHeaders Off

  RewriteEngine on
  RewriteCond %{HTTP:Upgrade} websocket [NC]
  RewriteCond %{HTTP:Connection} upgrade [NC]
  RewriteRule ^/?(.*) "ws://localhost:8000/$1" [P,L]
  ProxyPass / "http://localhost:8000/"
  ProxyPassReverse / "http://localhost:8000/"
```

=== `/etc/apache2/sites-available/ssl-include.conf` ===

```code
  SSLOpenSSLConfCmd DHParameters /etc/ssl/certs/dhparams.aodomainao.pem
  SSLCACertificateFile /etc/ssl/certs/own_ca.crt
  SSLCARevocationFile  /etc/ssl/crl/own_ca.crl
  SSLCARevocationCheck chain

  Protocols h2 h2c http/1.1
  Header add Strict-Transport-Security: "max-age=15552000;includeSubdomains"
  ServerAdmin mail@example.com

  SSLVerifyClient optional
  SSLVerifyDepth 1

  SSLEngine on
  SSLCompression off
  SSLOptions +StrictRequire
#  SSLOptions +StdEnvVars
  SSLProtocol -all +TLSv1.3 +TLSv1.2
  SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
  SSLHonorCipherOrder off
  SSLSessionTickets off
  ServerSignature off
  SSLOpenSSLConfCmd Curves X448:secp521r1:secp384r1:prime256v1
  SSLOpenSSLConfCmd ECDHParameters secp384r1

  SSLUserName SSL_CLIENT_SAN_Email_0

  RewriteCond %{SSL:SSL_CLIENT_VERIFY} !=SUCCESS
  RewriteCond %{REMOTE_ADDR} !^127\.0\.0\.1
  RewriteCond %{REMOTE_ADDR} !^::1
  RewriteCond %{REMOTE_HOST} !^127\.0\.0\.1
  RewriteRule . incorrect_client_certificate.php [L]
```

## Nextcloud installation

### Install directory

Get the Nextcloud Server directory Version 29.0.4 from GitHub.

```console
cd /var/www
git clone --depth 1 --shallow-submodules --recurse-submodules --branch v29.0.8 https://github.com/nextcloud/server.git
touch server/config/config.php
mv server ncdirectorync
chown --recursive --no-dereference www-data:www-data ncdirectorync
```

Create symbolic links to the two files in the `html` directory.

```console
cd /var/www/ncdirectorync
ln -s ../html/phpinfo.php phpinfo.php
```

### Database

Create the database, by executing the following SQL Statements.

```sql
CREATE DATABASE ncdatabasenc TEMPLATE template0 ENCODING 'UNICODE';
CREATE USER ncdbonc WITH PASSWORD 'ncdbopasswordnc';
ALTER DATABASE ncdatabasenc OWNER TO ncdbonc;
GRANT ALL PRIVILEGES ON DATABASE ncdatabasenc TO ncdbonc;
```

To be done in the following way.

```console
sudo --user=postgres psql --username=postgres
psql (15.8 (Debian 15.8-0+deb12u1))
Type "help" for help.

postgres=# CREATE USER ncdbonc WITH PASSWORD 'ncdbopasswordnc';
CREATE ROLE
postgres=# CREATE DATABASE ncdatabasenc TEMPLATE template0 ENCODING 'UNICODE' OWNER ncdbonc;
CREATE DATABASE
postgres=# GRANT ALL PRIVILEGES ON DATABASE ncdatabasenc TO ncdbonc;
GRANT
postgres=# \q
```

### Data directory

Create Data-Directory

```console
# mkdir --parents /var/lib/nextcloud/ncdatadirectorync
# chown --recursive www-data:www-data /var/lib/nextcloud
```

### Configuration

#### Start Web-Server

Start

```console
systemctl start apache2
```

#### Basis

Specify base base configuration.

```console
# sudo -u www-data php /var/www/ncdirectorync/occ maintenance:install --database "pgsql" --database-name "ncdatabasenc" --database-user "ncdbonc" --database-pass "ncdbopasswordnc" --admin-user "ncadminnc" --admin-pass "ncadminpasswordnc" --data-dir "/var/lib/nextcloud/ncdatadirectorync"
Nextcloud was successfully installed
# 
```

#### Redis

Enable Redis usage, with dbindex = 3.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.distributed --value "\\OC\\Memcache\\Redis"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set memcache.locking --value "\\OC\\Memcache\\Redis"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis dbindex --value "3"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis host --value "/run/redis/redis-server.sock"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set redis port --value "0"
```

#### Trusted domains

Specify the trusted_domains.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 0 --value "localhost"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 1 --value "127.0.0.1"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 2 --value "::1"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 3 --value "aodomainao"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 4 --value "aorouteraddressao"
```

Can be check at any time.

```console
# sudo --user=www-data php /var/www/ncdirectorync/occ  config:system:get trusted_domains
localhost
127.0.0.1
::1
aodomainao
aorouteraddressao
# 
```

During the tests in your local network, with direct communication between client and server, you would have to add as well the clients IP address. Otherwise you will get the error:

Access through untrusted domain

Please contact your administrator. If you are an administrator, edit the "trusted_domains" setting in config/config.php like the example in config.sample.php.

Further information how to configure this can be found in the [documentation](https://docs.nextcloud.com/server/29/go.php?to=admin-trusted-domains).

```console
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set trusted_domains 5 --value "10.0.0.12"
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
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set filelocking.enabled --value "true"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set maintenance_window_start --value "1"
sudo --user=www-data php /var/www/ncdirectorync/occ config:system:set default_phone_region --value "DE"
sudo --user=www-data php /var/www/ncdirectorync/occ config:app:set workflowengine logfile --value=/var/log/nextcloud/flow.log
```

#### The `config.php` file

At the end we will have a `config.php` looking like the following.

=== `/var/www/ncdirectorync/config/config.php` ===

```code
<?php
$CONFIG = array (
  'instanceid' => 'ocmu53th6kbo',
  'passwordsalt' => 'LCrY0KFd0kg1Ak7YPdnMqCmpAAWkhA',
  'secret' => 'VYfzVtwcxsLxZ48yBugcbeoZrA5l73CyuuRdXXt8gAQJzty9',
  'datadirectory' => '/var/lib/nextcloud/ncdatadirectorync',
  'dbtype' => 'pgsql',
  'version' => '29.0.5.1',
  'overwrite.cli.url' => 'http://localhost',
  'dbname' => 'ncdatabasenc',
  'dbhost' => 'localhost',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'dbuser' => 'ncdbonc',
  'dbpassword' => 'Kh8JdDPK9gh5HSRikPfHg5uGXyHA_77R',
  'updater.release.channel' => 'git',
  'installed' => true,
  'memcache.local' => '\\OC\\Memcache\\APCu',
  'filelocking.enabled' => 'true',
  'memcache.distributed' => '\\OC\\Memcache\\Redis',
  'memcache.locking' => '\\OC\\Memcache\\Redis',
  'redis' => 
  array (
    'dbindex' => '3',
    'host' => '/run/redis/redis-server.sock',
    'port' => '0',
  ),
  'maintenance_window_start' => '1',
  'default_phone_region' => 'DE',
  'trusted_domains' => 
  array (
    0 => 'aorouteraddressao',
    1 => '127.0.0.1',
    2 => '::1',
  ),
);
```

#### Establish Background Job

Add the "Background jobs" to the cron jobs of user `wwww-data`.

```console
crontab -u www-data -e
```

Add line

```code
*/5  *  *  *  * php -f /var/www/ncdirectorync/cron.php --define apc.enable_cli=1
```

Get rid of unwanted Nextcloud-Apps.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:remove testing
sudo --user=www-data php /var/www/ncdirectorync/occ app:disable weather_status
```

Install the usual Nextcloud-Apps.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:install calendar
sudo --user=www-data php /var/www/ncdirectorync/occ app:install camerarawpreviews
sudo --user=www-data php /var/www/ncdirectorync/occ app:install contacts
sudo --user=www-data php /var/www/ncdirectorync/occ app:install groupfolders
sudo --user=www-data php /var/www/ncdirectorync/occ app:install memories
sudo --user=www-data php /var/www/ncdirectorync/occ memories:places-setup
sudo --user=www-data php /var/www/ncdirectorync/occ app:install notes
sudo --user=www-data php /var/www/ncdirectorync/occ app:install notify_push
sudo --user=www-data php /var/www/ncdirectorync/occ app:install onlyoffice
sudo --user=www-data php /var/www/ncdirectorync/occ app:install previewgenerator
sudo --user=www-data php /var/www/ncdirectorync/occ app:install snappymail
sudo --user=www-data php /var/www/ncdirectorync/occ app:install spreed
sudo --user=www-data php /var/www/ncdirectorync/occ app:install tasks
```

At any point in time, you are able to update the Nextcloud-Apps from command line, using command `sudo --user=www-data php /var/www/ncdirectorync/occ app:update --all`. Right after the installation you should get the message `All apps are up-to-date or no updates could be found`.

Open URL `https://aodomainao/`. That way you a.) verify that the communication to the Internet is working and b.) will be able to enter the missing parameters.

## Nextcloud configuration

### App-Install Banner

Are you annoyed as well, when each web-site ask you to install their app. Even worst if it is your own. For iOS this banner can be de-activated using SQL Statement `INSERT INTO oc_appconfig(appid,configkey,configvalue) VALUES ('theming', 'iTunesAppId', '');`.

```console
# sudo --user=postgres psql --host=localhost --dbname=ncdatabasenc --username=ncdbonc --password
Password: 
psql (15.8 (Debian 15.8-0+deb12u1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
Type "help" for help.

ncdatabasenc=> INSERT INTO oc_appconfig(appid,configkey,configvalue) VALUES ('theming', 'iTunesAppId', '');
INSERT 0 1
ncdatabasenc=> select * from oc_appconfig where appid = 'theming';
  appid  |     configkey     |            configvalue             | type | lazy 
---------+-------------------+------------------------------------+------+------
 theming | installed_version | 2.4.0                              |    2 |    0
 theming | types             | logging                            |    2 |    0
 theming | enabled           | yes                                |    2 |    0
 theming | iTunesAppId       |                                    |    2 |    0
 theming | logoDimensions    | 175x58                             |    2 |    0
 theming | faviconMime       | image/png                          |    2 |    0
 theming | logoheaderMime    | image/png                          |    2 |    0
 theming | slogan            | Apache coturn Nextcloud ONLYOF...  |    2 |    0
 theming | url               | https://aodomainao                 |    2 |    0
 theming | name              | acnoo                              |    2 |    0
 theming | cachebuster       | 26                                 |    2 |    0
(11 rows)
```

### Client Push (notify_push)

With Client Push installed as Nextcloud App, the Client Push service is neither installed, enabled nor configured. To get the service setup properly the following steps are required.

1. Create service file
2. Enable service
3. Configure service

Create file `notify_push.service` for systemd service.

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

Ensure `notify_push` is executeable.

```console
# ls -la /var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push
-rw-r----- 1 www-data www-data 21416808 Oct 19 14:09 /var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push
# chmod 740 /var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push
ls -la /var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push
-rwxr----- 1 www-data www-data 21416808 Oct 19 14:09 /var/www/ncdirectorync/apps/notify_push/bin/x86_64/notify_push
```

Enable and start the service.

```console
systemctl enable notify_push
systemctl start notify_push
```

Setup the configuration.

```console
# sudo --user=www-data php /var/www/ncdirectorync/occ notify_push:setup https://aodomainao/push
✓ redis is configured
✓ push server is receiving redis messages
✓ push server can load mount info from database
✓ push server can connect to the Nextcloud server
✓ push server is a trusted proxy
✓ push server is running the same version as the app
  configuration saved
# 
```

### Talk (spreed)




## DSL-Router

Once your have tested the Web-Server in your local network, it is time to make it available for the clients on the Internet.

### Reboot

As part of your setup/test, you should reboot the Router to ensure that the configuration got stored properly.

The location of the configuration depends on the Router model. For mine, I found the configuration at "Settings > Problem handling > Restart".

click on "Restart"

### Configuration

#### Email SMTP-Server

Some router do block SMTP communication in general and allow communication only to those maintain in a list of E-Mail servers.

The location of the configuration depends on the Router model. For mine, I found the configuration at "Internet > E-mail abuse detection".

Add those SMTP-Server, you are certain about beeing used by the SnappyMail users, but not in the defined list of servers.

#### Port Forwarding

The location of the configuration depends on the Router model. For mine, I found the configuration at "Internet > Port activiation > Port redirections and port forwardings".

Three rules have to be created. One for the Web-Server and the other two for the TURN-Server.

Name of the redirection: Web-Server 443 only
Applies to the following device: nchostnc
Use template: Web-Server
Ports to redirect:
    TCP: 443 🞂 443

Name of the redirection: TURN-Server
Applies to the following device: nchostnc
Use template: No template
Ports to redirect:
    TCP: 5349 🞂 5349
    UCP: 5349 🞂 5349

Name of the redirection: TURN-Server ports
Applies to the following device: nchostnc
Use template: No template
Ports to redirect:
    TCP: 10000-20000 🞂 10000-20000
    UCP: 10000-20000 🞂 10000-20000

# Add-On

## Client-Certificate authentication

Instead of typing in user/password, the authentication can be established based on the Client-Certificate. Which got deployed anyway and is secure enough to authenticate a user and its device.

### Prerequisites

You have to have **two** different Client-Certificate. One for the regular user account and another for the administrator.

In file `ssl-include.conf` the following line must exist, otherwise the environment variable `REMOTE_USER` cannot be used for identifying the right user account.

```code
  SSLUserName SSL_CLIENT_SAN_Email_0
```

### Login change

As Administrator

- 🪜 open a "**New private window**", in the web browser
	- 🪜 open URL `aodomainao`
		- 🪜 login with the administrator account `ncadminnc`
			- 🪜 select "**+ Apps**" from the menu
				- 🪜 select "Featured apps" in the left panel
					- ✏️ click on "**Download and enable**" in line of "SSO & SAML authentication"
			- 🪜 select "**Adiministration settings**" from the menu
				- 🪜 select "SSO & SAML authentication" in the left panel
					- ✏️ enter "REMOTE_USER" in field under "General > Attribute to map the UID to."
					- ✏️ ~~disable~~ "**Only allow authentication if an account exists on some other backend (e.g. LDAP).**"

Stay logged in as Administrator, until the test is finished and everything setup properly.

### Test

As User

- 🪜 open another "**New private window**", in the web browser
	- 🪜 open URL `aodomainao`
		- ✏️ select the Client-Certificate of the **user**

You got automatically logged in.

As Administrator

- 🪜 open another "**New private window**", in the web browser
	- 🪜 open URL `aodomainao`
		- ✏️ select the Client-Certificate of the **Administrator**

You got automatically logged in.

The login process was working as expected? The you are save to close the web browser window/tab of section "Login change".

### Revert change

At anytime you can disable the App and everyone has to login like before. However, those users created automatically via SSO are gone.

```console
sudo --user=www-data php /var/www/ncdirectorync/occ app:disable user_saml
```

## Let's encrypt

For the initial creation of the Certificates, use the content below of `any.conf`. In the meantime, make a backup the original version.

```console
mv /etc/apache2/sites-available/any.conf /etc/apache2/sites-available/any.conf.ORG
vi /etc/apache2/sites-available/any.conf
cp /etc/apache2/sites-available/any.conf /etc/apache2/sites-available/any.conf.NEW
```

=== `/etc/apache2/sites-available/any.conf` (temporary) ===

```code
ServerName nchostnc
  
<VirtualHost *:80>
  ServerName nchostnc
  ServerName localhost
  DocumentRoot "/var/www/html"
</VirtualHost>
  
<VirtualHost *:80>
  ServerName aodomainao
  DocumentRoot "/var/www/html"
</VirtualHost>

<VirtualHost *:80>
  ServerName ooserveroo.aodomainao
  DocumentRoot "/var/www/html.ooserveroo"
</VirtualHost>

<VirtualHost *:80>
  ServerName ctserverct.aodomainao
  DocumentRoot "/var/www/html.ctserverct"
</VirtualHost>
```

Create the Certificates/Keys for the three Web-Site.

Note: Later I figured out that the key type and length does not match the original configuration. That's why I added the following step to change to RSA 4098 bits. Maybe at the creation of the certificates, using `--key-type rsa --rsa-key-size 4096` the keys get directly created in the right format.

```console
# certbot --agree-tos --email aoadmin@addressao -d aodomainao
:

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/aodomainao/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/aodomainao/privkey.pem
This certificate expires on <date>.
:
# certbot --agree-tos --email aoadmin@addressao -d ooserveroo.aodomainao
:

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ooserveroo.aodomainao/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ooserveroo.aodomainao/privkey.pem
This certificate expires on <date>.
:
# certbot --agree-tos --email aoadmin@addressao -d ctserverct.aodomainao
:

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ctserverct.aodomainao/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ctserverct.aodomainao/privkey.pem
This certificate expires on <date>.
:
# 
```

Change from ECDSA to RSA 4096 bit

Once created, you can change to the 4096 bit length with RSA.
During the process, you have to answer the question "(U)pdate key type/(K)eep existing key type:" with "U".

```console
# certbot --agree-tos --key-type rsa --rsa-key-size 4096 --email aoadmin@addressao -d aodomainao
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
An ECDSA certificate named aodomainao already exists. Do you want to
update its key type to RSA?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(U)pdate key type/(K)eep existing key type: U
Renewing an existing certificate for aodomainao

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/aodomainao/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/aodomainao/privkey.pem
This certificate expires on 2025-01-21.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for aodomainao to /etc/apache2/sites-enabled/any.conf
Your existing certificate has been successfully renewed, and the new certificate has been installed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# certbot --agree-to --key-type rsa --rsa-key-size 4096 --email aoadmin@addressao -d ooserveroo.aodomainao
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
An ECDSA certificate named ooserveroo.aodomainao already exists. Do you want to
update its key type to RSA?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(U)pdate key type/(K)eep existing key type: U
Renewing an existing certificate for ooserveroo.aodomainao

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ooserveroo.aodomainao/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ooserveroo.aodomainao/privkey.pem
This certificate expires on 2025-01-21.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for ooserveroo.aodomainao to /etc/apache2/sites-enabled/any.conf
Your existing certificate has been successfully renewed, and the new certificate has been installed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# certbot --agree-tos --key-type rsa --rsa-key-size 4096 --email aoadmin@addressao -d ctserverct.aodomainao
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
An ECDSA certificate named ctserverct.aodomainao already exists. Do you want to
update its key type to RSA?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(U)pdate key type/(K)eep existing key type: U
Renewing an existing certificate for ctserverct.aodomainao

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/ctserverct.aodomainao/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/ctserverct.aodomainao/privkey.pem
This certificate expires on 2025-01-21.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.

Deploying certificate
Successfully deployed certificate for ctserverct.aodomainao to /etc/apache2/sites-enabled/any.conf
Your existing certificate has been successfully renewed, and the new certificate has been installed.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
If you like Certbot, please consider supporting our work by:
 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
 * Donating to EFF:                    https://eff.org/donate-le
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# 
```

Revert the change of `any.conf`.

```console
cp /etc/apache2/sites-available/any.conf.ORG /etc/apache2/sites-available/any.conf
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

## OpenVPN

### Prerequisites

This server process as well, needs some certificates, most of which can be "-reused" from the existing Apache configuration.

- /etc/ssl/certs/own_ca.crt
- /etc/ssl/crl/own_ca.crl
- /etc/ssl/certs/aodomainao.crt
- /etc/ssl/private/aodomainao.key
- /etc/ssl/certs/dhparams.aodomainao.pem

#### TLS authentication (TA)

The OpenVPN connection, in this configuration, is intended to be used by the administrator only. Also for simplicity, to avoid using TLS Crypt v2.

```console
openvpn --genkey secret /etc/ssl/private/ta.aodomainao.key
```

### Server

#### Installation

1. Beside OpenVPN, iptables is required to allow communication from the VPN-Tunnel to the Internet. And to store the iptables configuration, the two `*-persitent` packages are required as well.
2. Create the VPN-Server configuration file `server.conf`.
3. Enable and start the OpenVPN Server.

Note: During the installation of `iptables-persistent`, you will be asked some questions. Ignore these (select "<No>"), we will deal with this aspect later

```console
apt install -y openvpn iptables netfilter-persistent iptables-persistent
vi /etc/openvpn/server/server.conf
systemctl enable openvpn-server@server
```

=== `/etc/openvpn/server/server.conf` ===

```code
verb 0  # verbose mode

port 1194
proto udp
dev tun

dh         /etc/ssl/certs/dhparams.aodomainao.pem
ca         /etc/ssl/certs/own_ca.crt
crl-verify   /etc/ssl/crl/own_ca.crl

cert       /etc/ssl/certs/aodomainao.crt
key      /etc/ssl/private/aodomainao.key
data-ciphers AES-256-CBC:AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305
auth-nocache

tls-version-min 1.2
tls-cipher TLS-ECDHE-ECDSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-RSA-WITH-CHACHA20-POLY1305-SHA256:TLS-ECDHE-ECDSA-WITH-AES-128-GCM-SHA256:TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256
tls-auth /etc/ssl/private/ta.aodomainao.key 0

topology subnet
server 10.9.8.0 255.255.255.0  # internal tun0 connection IP
#ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
keepalive 10 120
persist-key
persist-tun
client-to-client
explicit-exit-notify 1
```

#### Routing configuration

Before we can adjust the routine of network packages, the kernel parameter `net.ipv4.ip_forward` has to be enabled using `sysctl`.

```console
sed -i "s/^#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/" /etc/sysctl.conf
sysctl -p
```

=== `/etc/sysctl.conf` ===

```code
:

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

:
```

In the old style, using iptables, the routing is specify in the following.
Note: The interface `eth0` has to be replaced by the real one, of your server.

```console
iptables --table nat --append POSTROUTING --out-interface eth0 --jump MASQUERADE
iptables --append FORWARD --in-interface eth0 --out-interface tun0 --match state --state RELATED,ESTABLISHED --jump ACCEPT
iptables --append FORWARD --in-interface tun0 --out-interface eth0 --jump ACCEPT
```


To store the configuration, the following commands are required.

```console
iptables-save > /etc/iptables/rules.v4
ip6tables-save > /etc/iptables/rules.v6
netfilter-persistent save
```

Note: Anytime your change something on the iptables configuration, make sure you execute the same two `ip*tables-save` commands like above.

Commands to review changes

```console
# iptables --list
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
# iptables --tables nat --list
iptables v1.8.9 (nf_tables): unknown option "--tables"
Try `iptables -h' or 'iptables --help' for more information.
# iptables --table nat --list
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         

Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination         
MASQUERADE  all  --  anywhere             anywhere  
```

#### Status

```console
# systemctl status openvpn-server@server.service
● openvpn-server@server.service - OpenVPN service for server
     Loaded: loaded (/lib/systemd/system/openvpn-server@.service; enabled; preset: enabled)
     Active: active (running) since Mon 2024-09-30 14:53:42 CEST; 1 day 3h ago
       Docs: man:openvpn(8)
             https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
             https://community.openvpn.net/openvpn/wiki/HOWTO
```

### Client

1. First the OpenVPN software has to be installed.
2. In additional to the already deployed `.pfx` PKCS#12-Certificate, the source files `.crt` and `.key` have to be deployed as well, into `/etc/ssl/certs/` and `/etc/ssl/private/`.
2. Copy the file `/etc/ssl/private/ta.aodomainao.key` from the server at the same location.
3. Create the VPN-Client configuration file `client.conf`

```console
apt install -y openvpn
```

=== `/etc/openvpn/client/client.conf` ===

```code
verb 0

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

# This and That

## HDD - idle => power off

In case of a real HDD, you might to want the backup device to go into sleep mode ... once the backup is done. Or does it go into power off, once un-mounted?

```console
# lsusb
Bus 004 Device 003: ID 174c:55aa ASMedia Technology Inc. ASM1051E SATA 6Gb/s bridge, ASM1053E SATA 6Gb/s bridge, ASM1153 SATA 3Gb/s bridge, ASM1153E SATA 6Gb/s bridge
:
# cat  /sys/bus/usb/devices/4-2/product 
ASM105x
# cat  /sys/bus/usb/devices/4-2/power/control 
on
# cat  /sys/bus/usb/devices/4-2/power/autosuspend_delay_ms
2000
# echo "auto" > /sys/bus/usb/devices/4-2/power/control 
```

## Nextcloud: Errors, Infos, Warning

### Administrative settings: Overview > Security & setup warnings

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

INFO: Your installation has no default phone region set. This is required to validate phone numbers in the profile settings without a country code. To allow numbers without a country code, please add "default_phone_region" with the respective ISO 3166-1 code of the region to your config file. For more details see the [documentation ↗](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2#Officially_assigned_code_elements).
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

## S.M.A.R.T

Usually every disk/flash drive support this feature. Some useful command ... at least for the beginning.

```
apt-get install -y smartmontools
smartctl -i /dev/sda
SMART support is: Disabled
smartctl -s on /dev/sda
smartctl -a /dev/sda | grep Temp | cut -d" " -f 2,37
```


# Based on

While looking for the right way, to set up the server, I search a lot on the Internet. The following are those articles/postings which helped me most.

## OS

### SSD

- [Solid State Drives unter Linux schonend ausreizen (Seite 3)](https://www.linux-community.de/ausgaben/linuxuser/2015/02/pimp-my-ssd/3/)

## Nextcloud

- [Do not allow auto-create user if not existing](https://help.nextcloud.com/t/do-not-allow-auto-create-user-if-not-existing/165046)
- [How to remove/disable on iOS/Android the “App install banner”?](https://help.nextcloud.com/t/how-to-remove-disable-on-ios-android-the-app-install-banner/90942)

## OpenSSL

- [How To Setup a CA](https://pages.cs.wisc.edu/~zmiller/ca-howto/)
- [Create User Certificates via OpenSSL](https://help.cloud.fabasoft.com/index.php?topic=doc/CA-and-User-Certificate-Management/create-user-certificates-via-openssl.htm)
- [Apache: SSL/TLS Strong Encryption: How-To](https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html)
- [How to add X.509 extensions to certificate OpenSSL](https://www.golinuxcloud.com/add-x509-extensions-to-certificate-openssl/)
- [How to configure development server certificates for iOS 13 and Mac clients](https://jaanus.com/ios-13-certificates/)
- [Requirements for trusted certificates in iOS 13 and macOS 10.15](https://support.apple.com/en-us/103769)

## OpenSSH

- [How to Setup Passwordless SSH Login](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)

## OpenVPN

- [debian Wiki: OpenVPN](https://wiki.debian.org/OpenVPN)
- [Configuring IPTables Firewall on Debian 12](https://reintech.io/blog/configuring-iptables-firewall-debian-12)

## TURN-Server -coturn

- [How to set up and configure your own TURN server using Coturn](https://gabrieltanner.org/blog/turn-server/)

## Web-Server - Apache

- [Nextcloud 23 Installationsanleitung mit Apache2](https://www.c-rieger.de/nextcloud-installationsanleitung-apache2/)
- [Redirect http to https with htaccess and mod_rewrite](https://dev.to/nickfrosty/redirect-http-to-https-force-https-with-htaccess-42c9)
- [InMotion Hosting: How to Hide Your Apache Version and Linux OS From HTTP Headers](https://www.inmotionhosting.com/support/server/apache/hide-apache-version-and-linux-os/)
- [Apache Week Feature: Using Certificate Revocation Lists](http://www.apacheweek.com/features/crl)

# Acronym

| Acronym | Stands for |
| ---: | :--- |
| CPU | Central Processing Unit |
| DBMS | Database Management System |
| DDNS | Dynamic Domain Name System |
| DSL | Digital Subscriber Line |
| IP | Internet Protocol |
| NAT | Network address translation |
| OS | Operating System |
| RAM | Random Access Memory |
| SATA | Serial AT Attachment |
| SSD | Solid Disk Drive |
| STAN Session Traversal Utilities for NAT |
| TURN | Traversal Using Relays around NAT |