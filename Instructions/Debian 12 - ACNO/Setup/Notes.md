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
- Apache `SSLVerifyClient require`: Resticting the access on the Web-Server level, works extremly well. At least from the security aspect. However you cannot use any of the Android/iOS apps. The compromise is to use the Client-Certificates for authentication and allow access of Nextcloud also for others.

At the end, I decided to go with the following "Basis" setup.

## Basis

Network

```code
Internet <---> DSL-Router <---> Wireless Router <---> Workstation
                            |                     |
                            |                     +--> machine `aohostao` (during setup)
                            |
                            +--> machine `aohostao` (DMZ)
```

# Certificates

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

## Nextcloud Prerequisites

### Proxy-Server - Nginx

Unfortunately `nginx` is tightly bound with the ONLYOFFICE installation. I first tried to ignore this circumstance and had disabled the `nginx` Web-Server Proxy, but after an upgrade from version 8.1.3-4 to 8.2.0-143 ... I had been unable to open the documents (error `Cannot GET /8.2.0-88d72bc60b1e714a35888b9a72c76d20/web-apps/... `). So for the moment, we will leave `nginx` acting as Proxy but on port `8080`.

In file `ds.conf` replace the port definition from `80` to `8080`.

Note: Have not figured out yet, if the same step is required for each update of ONLYOFFICE. See also section "Issues" to handle issues during update.

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