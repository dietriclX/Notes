FreeBSD - pfSense

# Used Parameters/Values

| Parameter | Description |
| :--- | :--- |
| pfserialpf | Serial Number of the pfSense installation |
| pfdeviceidpf | Netgate Device ID |
| pfhostnamepf | Hostname of the pfSense system |
| pfdomainpf | Domain name served by the pfSense system |
| pfipaddresspf | IP-Address of the pfSense system (as specified setup) |
| pfadminpasswordpf | Password of the `admin` user (default: pfsense) |
| upsnameups | Name of UPS, as used within pfSense
| upsserialups | Serial number of APC Back-UPS (12 hex-digits) |
| pfupsusbpf | Either `auto` or device name incl. `/dev/` (e.g. `/dev/ugen1.2`) |
| slcanamesl | Name of the own Certificate Authrority |
| slcafilesl | `.crt` file of the own CA |
| slpfcrtpfsl | `.crt` file for the pfSense webConfigurator |
| slpfkeypfsl | `.key` file for the pfSense webConfigurator |

# Installation

## Prerequisites

1. Monitor connected to the machine
2. Keyboard connected to the machine
3. WAN interface connected to the Internet.
4. LAN interface connected to the Workstation!
5. USB-Stick with pfSense install image.

Note: *Once the system is good enough configured to be accessible via the browser, Monitor & Keyboard are no longer needed, but the Workstation needs to be connected to the LAN interface.*

## The goal

For the internal SDD card, we will be using the standard File System ZFS.

The machine has one on-board LAN interface (re0) and a quad port LAN card (igb0-3). The interfaces will be defined as follows.

- WAN  -> re0; DHCP Client
- LAN  -> igb0; Static IP Address 192.168.10.1 with DHCP Server 192.168.10.100…199

Means, you might need to adjust the following values.
- re0
- igb0
- 192.168.10.1
- 192.168.10.100
- 192.168.10.199

Note: Neither interface assignment nor the IP Address information was taken over from the setup (CUI-Wizard). Using the Console, I had to correct the assignment and IP Address.

## Start-UP

With the USB Media inserted, press F12 during boot to get the option to choose a boot partition.

```code
[UEFI: Mass Storage Device 1.00]
Mass Storage Device 1. 00
P1: Innodisk DEMSR-08GB mSATAl
Diagnostic Program
```

- select "**Mass Storage Device 1.  00**"
- 🪜 press ENTER

pfSense Live image boots ...

## Configuration - 1. Part: CUI-Wizard

1. license dialog
     - select "**Accept**“, if you agree to the terms
     - press ENTER

``` code
Install      Install pfSense
Rescue Shell Launch a shell for rescue operations
```

2. dialog "Welcome"
     - select "Install"
     - press ENTER

3. dialog "Network Installation" - Setting up the network to continue the installation.
     - press ENTER
4. dialog "WAN Interface Assignment and Configuration.“ - Please select the WAN interface.
     - select "re0"
     - press ENTER

```code
>>> Continue         Proceed with the installation
M Interface Mode     DHCP(client)
V VLAN Settings      VLAN Tagging disabled
U Use local resolver false
```

5. dialog "WAN (re0) Network Mode Setup" - Adjust the network operation mode for the WAN (re0) interface if necessary.
     - select "Use local resolver"
     - press ENTER  
     => CHANGED TO TRUE
     - select "Continue"
     - press ENTER
6. dialog "LAN Interface Assignment and Configuration"
     - select "igb0"
     - press ENTER

```code
>>> Continue    Proceed with the installation
M Interface Mode    STATIC
V VLAN Settings     VLAN Tagging disabled
I IP Address        192.168.1.1/24
D DHCPD Enabled     true
S DHCPD Range Start 192.168.1.100
E DHCPD Range End   192.168.1.150
```

7. dialog "LAN (igb0) Network Mode Setup"
     - select "IP Address"
     - 🪜 press ENTER
     8. dialog "LAN (igb0) IP Address Setup"
          - enter "192.168.10.1/24"
          - 🛝 press ENTER
     - select "DHCPD Range Start"
     - 🪜 press ENTER
     9. dialog "LAN (igb0) DHCPD Range Start Setup"
          - enter "192.168.10.100"
          - 🛝 press ENTER
     - select "DHCPD Range End"
     - 🪜 press ENTER
     10. dialog "LAN (igb0) DHCPD Range End Setup"
          - enter "192.168.10.199”
          - 🛝 press ENTER
     - select "Continue"
     - press ENTER

```code
LAN igb0 (no carrier)
WAN re0 (active)
```

11. dialog "Interface Assignment and Configuration" - Please confirm the interface assignment to continue with the installation.
     - press ENTER
12. dialog "Connectivity Check“
     tries to to connect to Netgate  server....
13. dialog "Active Subscription Validation" - The device does not have an active pfSense Plus subscription.
     - select "Install CE"
     - press ENTER

```code
>>> Continue       Proceed with the installation
F File System      ZFS (recommended default)
P Partition Schene GPT (compatible with MBR)
```

14. dialog "Installation Options" - Please select the File System type and the Partition Scheme.
     - select "Continue“
     - press ENTER

```code
stripe Stripe - No Redundancy
```

15. dialog "ZFS Virtual Device Type Configuration" - Select the ZFS Virtual Device configuration type.
     - select "stripe“
     - press ENTER

```code
[X] ada0 7.4G <Innodisk DEMSR - 08GB mSATA 3ME A>
```

16. dialog "Disk Selection" - Select the disk(s) for software installation.
     - press ENTER

```code
ada0
```

17. dialog "Confirmation" - Last Chance! Are you sure you want to destroy the current contents of the follow disks:
     - select "Yes"
     - press ENTER

commiting the changes to disk

```code
0000-2.7.2 Current Stable Release (2.7.2)
0001-2.7.0 Deprecated Version (2.7.0)
```

18. dialog "Software Version to Install" - Select.the version of pfSense CE to install.
     - select "0000-2.7.2"
     - press ENTER
19. dialog "Installation Details"
     ... progress of installation ...

```code
:
pfSense Post Installation setup .. done
```

19. dialog "Installation Details"
     - press ENTER
20. dialog "Complete" - Installation of pfSense complete! Would you like to reboot into the installed system now?
     - select "Reboot"
     - press ENTER

In the BIOS Setting, disable boot from USB & removable media.

boot …

## Configuration - 2. Part: Console

After the first boot, you will be greated with the following information.

```code
FreeBSD/amd64 (pfSense.home.arpa) (ttyv0)

pfSense - Serial: pfserialpf - Netgate Device ID: pfdeviceidpf

*** Welcome to pfSense 2.7.2-RELEASE (amd64) on pfSense ***

 WAN (wan)       -> igb0       ->
 LAN (lan)       -> igb1       -> v4: 192.168.1.1/24

0) Logout (SSH only)     9) pfTop
1) Assign Interfaces   10) Filter Logs
2) Set interface(s) IP address   11) Restart webConfigurator
3) Reset webConfigurator password    12) PHP shell + pfSense tools
4) Reset to factory defaults   13) Update from console
5) Reboot system   14) Enable Secure Shell (sshd)
6) Halt system   15) Restore recent configuration
7) Ping host   16) Restart PHP-FPM
8) Shell

Enter an option: 
```

- enter "1"
- 🪜 press ENTER

```
Valid interfaces are:

igb0   <MAC Address #1>
igb1   <MAC Address #2>
igb2   <MAC Address #3>
igb3   <MAC Address #4>
re0    <MAC Address #1>

Do VLANs need to be setup first?
If VLABs will not be used, or only for optional interfaces, it is typical to say no here and use the webConfigurator to configure VLANs later, if required.

Should VLANs be set up now [y|n]? 
```

- enter "n"
- press ENTER

```code
If the names of the interfaces are not known, auto-detection can be used instead. To use auto-detection, please disconnect all interfaces before pressing ‚a‘ to begin the process.

Enter the WAN interface name or ‚a‘ for auto-detection
(igb0 igb1 igb2 igb3 re0 or a): 
```

- enter "re0“
- press ENTER

```code

Enter the LAN interface name or ‚a‘ for auto-detection
NOTE: this enables full Firewalling/NAT mode.
(igb0 igb1 igb2 igb3 a or nothing if finished): 
```

- enter "igb0“
- press ENTER

```code
Enter the Optional 1 interface name or ‚a‘ for auto-detection
(igb1 igb2 igb3 a or nothing if finished): 
```

- press ENTER

```code
The interfaces will be assigned as follows:

WAN  -> re0
LAN  -> igb0

Do you want to proceed [y|n]? 
```

- enter "y“
- 🛝 press ENTER

```code
Writing configuration…done.
One moment while the settings are reloading… done!
pfSense - Serial …
:
Enter an option:
```

- enter "2“
- 🪜 press ENTER

```code
Available interfaces:

1 - WAN (re0 - dhcp, dhcp6)
2 - LAN (igb0 - static)

Enter the number of the interfaace you wish to coonfigure: 
```

- enter "2“
- press ENTER

```code
Configure IPv4 address LAN interface via DHCP? (y/n) 
```

- enter "n“
- press ENTER

```code
Enter the new LAN IPv4 address. Press <ENTER> for none:
> 
```

- enter "192.168.10.1“
- press ENTER

```code
Subnet masks are entered as bit counts (as in CIDR notation) in pfSense.
e.g. 255.255.255.0 = 24
     255.255.0.0   = 16
     255.0.0.0     = 8

Enter the new LAN IPv4 subnet bit count (1 to 32):
>
```

- enter "24“
- press ENTER

```code
For a WAN, enter the new LAN IPv4 upstream gateway address.
For a LANN, press <ENTER> for none:
> 
```

- press ENTER

```code
Configure IPv6 address LAN interface via DHCP6? (y/n) 
```

- enter "n“
- press ENTER

```code
Enter the new LAN IPv6 address. Press <ENTER> for none:
> 
```

- press ENTER

```code
Do you want to enable the DHCP server on LAN? (y/n) 
```

- enter "y“
- press ENTER

```code
Enter the start address of the IPv4 client address range: 
```

- enter "192.168.10.100“
- press ENTER

```code
Enter the end address of the IPv4 client address range: 
```

- enter "192.168.10.199“
- press ENTER

```code
Do you want to revert to HTTP as the webConfigurator protocol? (y/n) 
```

- enter "n“
- press ENTER

```code
Please wait while the changes are saved to LAN…
 Reloading filter…
 Reloading routing configuration…
 DHCPD…

The IPv4 LAN address has been set to 192.168.10.1/24
You can now access the webConfigurtor by opening the following URL in your web browser:
                https://192.168.10.1/

Press <ENTER to continue.
```

- 🛝 press ENTER

```code
pfSense - Serial: pfserialpf - Netgate Device ID: pfdeviceidpf

*** Welcome to pfSense 2.7.2-RELEASE (amd64) on pfSense ***

 WAN (wan)       -> re0        -> v4/DHCP4: 192.168.30.109/24
 LAN (lan)       -> igb0       -> v4: 192.168.10.1/24
:
Enter an option:
```

- enter "6“
- 🪜 press ENTER

```code
pfSense will shutdown and halt system. This may take a few minutes, depending on your hardware.
Do you want to proceed [y|n]? 
```

- enter "y“
- 🛝 press ENTER

System shutdown and powered off.

- disconnect keyboard!!!
- start system once more to check the interface assignment

```code
pfSense - Serial: pfserialpf - Netgate Device ID: pfdeviceidpf

*** Welcome to pfSense 2.7.2-RELEASE (amd64) on pfSense ***

 WAN (wan)       -> re0        -> v4/DHCP4: 192.168.30.109/24
 LAN (lan)       -> igb0       -> v4: 192.168.10.1/24
:
Enter an option:
```

It is correct, so you can turn off the system.

- briefly press the "power-on“ button

System shutdown and powered off.

## Configuration - 3. Part: webConfigurator

- start the pfSense machine

- open URL: https://pfipaddresspf
- login with `admin`/`pfsense`

It is the first time, you access the webConfigurator, therefore you are greated with the "**Wizard** > **pfSense Setup**".

- click on "**Next**"
- click on "**Next**"

dialog "General Information"

- enter "pfhostnamepf" as "**Hostname**"
- enter "pfdomainpf" as "**Domain**"
- click on "**Next**"

dialog "Time Server Information"

- select your timezone from listbox "**Timezone**"
- click on "**Next**"

dialog "Configure WAN Interface"

- click on "**Next**"

dialog "Configure LAN Interface"

- click on "**Next**"

dialog "Set Admin WebGUI Password"

- enter you new pfadminpasswordpf for the webConfigurator
- click on "**Next**"

dialog "Reload configuration"

- click on "**Reload**"

dialog "Wizard completed."

- click on "**Check for update**"

dialog "Confirmation Required to update pfSense system."

Note: *"**Status**" should say "Up to date", if you are not having used a very old USB media*

# Basic Configuration - 4. Part: webConfigurator

## Enable RAM Disk

- select menu option "**System** > **Advanced**"
- navigate to tab "**Miscellaneous**"
- focus on section "**RAM Disk Settings (Reboot to Apply Changes)**"
- enable option "**Use RAM Disks**"
- enter "2048" for "**/tmp RAM Disk**" at "**RAM Disk Size**"
- enter "3072" for "**/var RAM Disk**" at "**RAM Disk Size**"
- click on "**Save**"

Note: *RAM Disk is going to use 5.12 GB of the installed 8 GB.*

dialog "The "Use RAM Disks" setting has been changed.
This requires the firewall to reboot."

- click on "OK" to answer "Reboot now?" with yes

## Switch DHCP product

The product "ISC DHCP" is  deprecated, thuse makes it useful to switch to the alternative.

- choose menu option  "**System** > **Advanced**"
- navigate to tab "**Networking**"
- focus on section "**DHCP Options**"
- select  option "**Kea DHCP**" as "**Server Backend**"
- click on "**Save**"

## Enable Hardware Crypto

First check the status of the Hardware crypto and the CPUs supported Crypto features.

### Check Status

#### check at "**CPU Type**"

- choose menu option "**Status** > **Dashboard**"
- focus on section "**System Information**"
- at  "**CPU Type**" look for an "**...Crypto...**" entry. At the end of that line it says either "**(inactive)**" or "**(active)**".

Example information at "**CPU Type**"

```code
AMD GX-415GA SOC with Radeon(tm) HD Graphics
4 CPUs: 1 package(s) x 4 core(s)
AES-NI CPU Crypto: Yes (inactive)
QAT Crypto: No
```

#### check at "**Hardware crypto**"

- choose menu option "**Status** > **Dashboard**"
- focus on section "**System Information**"
- at  "**Hardware crypto**" it says "**Inactive**" or lists a number of ciphers.

### change

- open menu option "**System** > **Advanced**"
- navigate to tab "**Miscellaneous**"
- focus on section "**Cryptographic & Thermal Hardware**"
- select "AES-NI CPU-based Acceleration" for "**Cryptographic Hardware**"
- click on "**Save**"

## Enable SSH

### change

- choose menu option  "**System** > **Advanced**"
- scroll to section "**Secure Shell**"
- enable "**Secure Shell Server**"
- click on "Save"

### check

From your Workstation you can check is SSH was setup correctly.

```console
$ ssh admin@pfhostnamepf.pfdomainpf
(admin@pfhostnamepf.pfdomainpf) Password for admin@pfhostnamepf.pfdomainpf:
pfSense - Serial: pfserialpf - Netgate Device ID: pfdeviceidpf

*** Welcome to pfSense 2.7.2-RELEASE (amd64) on pfhostnamepf ***

 WAN (wan)       -> re0        -> v4/DHCP4: 192.168.30.109/24
 LAN (lan)       -> igb0       -> v4: 192.168.10.1/24

 0) Logout (SSH only)                  9) pfTop
 1) Assign Interfaces                 10) Filter Logs
 2) Set interface(s) IP address       11) Restart webConfigurator
 3) Reset webConfigurator password    12) PHP shell + pfSense tools
 4) Reset to factory defaults         13) Update from console
 5) Reboot system                     14) Disable Secure Shell (sshd)
 6) Halt system                       15) Restore recent configuration
 7) Ping host                         16) Restart PHP-FPM
 8) Shell

Enter an option: 
```

- enter "0" (for logout)
- press ENTER

## Create LAN Bridge for igb0, igb1, igb2

- select menu option "**Interfaces** > **Assignments**"
- select "igb1 (<MAC Address>)" from listbox "**Available network ports**"
- click on "**Add**"
- select "igb2 (<MAC Address>)" from listbox "**Available network ports**"
- click on "**Add**"
- click on "**Save**"

- select menu option "**Interfaces** > **OPT1 (igb1)**"
- enable option "**Enable Interface**"
- type "LAN2" into "**Description**" of interface "igb1"
- click on "**Save**"
- click on "**Apply Changes**"

- select menu option "**Interfaces** > **OPT2 (igb1)**"
- enable option "**Enable Interface**"
- type "LAN3" into "**Description**" of interface "igb2"
- click on "**Save**"
- click on "**Apply Changes**"

- select menu option "**Interfaces** > **Assignments**"
- navigate to tab "**Bridges**"
- click on "**Add**"

dialog "**Interfaces / Bridges / Edit**"

- select "LAN2" & "LAN3" as "**Member Interfaces**"
- type "Quad-Port-Bridge LAN1-3" in "**Description**"
- click on "**Save**"
- navigate to tab "**Interface Assignments**"
- select option "BRIDGE0 (Quad-Port-Bridge LAN1-3)" as "**Network port**"
- click on "**Save**"

Go to the pfSense machine and switch the LAN cable from igb0 to one of those from the bridge (igb1-2).

- select menu option "**Interfaces** > **Assignments**"
- select "igb0 (<MAC Address>)" from listbox "**Available network ports**"
- click on "**Add**"
- click on "**Save**"

- select menu option "**Interfaces** > **OPT3 (igb0)**"
- enable option "**Enable Interface**"
- type "LAN1" into "**Description**" of interface "igb0"
- click on "**Save**"
- click on "**Apply Changes**"

- select menu option "**Interfaces** > **Assignments**"
- navigate to tab "**Bridges**"
- click on "**🖉**" (Pencil icon) to edit interface "**BRIDGE0**"
- select "LAN1" in addition the already select"LAN2" & "LAN3" of "**Member Interfaces**"
- click on "**Save**"

Go to the pfSense machine and switch the LAN cable back to igb0.

Note: *Even after a reboot the order of the interfaces is still the same. WAN, LAN, LAN2, LAN3, LAN1*

## Configure Certificates

### Import Certificate Authority (own CA)

- open the `slcafilesl` file in a text editor
- copy the content into the Clipboard
- open menu option "**System** > **Certificates**"
- click on "**Add**"

dialog "**System / Certificate / Authorities / Edit**"

- focus on section "**Create / Edit CA**"
- enter "slcanamesl" in "**Description**"
- select "Import an existing Certificate Authority" as "**Method**"
- enable option "**Trust Store**"
- insert Clipboard content into "**Certificate data**"
- click on "**Save**"

### Import Certificate (Own Web Certificate)

- open menu option "**System** > **Certificates**"
- navigate to tab "**Certificates**"
- click on "**Add**"

dialog "**System / Certificate / Certificates / Edit**"

- focus on section "**Add/Sign a New Certificate**"
- select "Import an existing Certificate" as "**Method**"
- enter "Own Web Certificate" in "**Description**"
- focus on section "**Import Certificate**"
- open the `slpfcrtpfsl` file in a text editor
- copy the content into the Clipboard
- insert Clipboard content into "**Certificate data**"
- open the `slpfkeypfsl` file in a text editor
- copy the content into the Clipboard
- insert Clipboard content into "**Private key data**"
- click on "**Save**"

### Activate Own Web Certificate

- select menu option "**System** > **Advanced**"
- navigate to tab "**Admin Access**"
- focus on section "**webConfigurator**"
- select "Own Web Certificate" as "**SSL/TLS Certificate**"
- click on "**Save**"

### Test Own Web Certificate

- close browser tab
- in new browser tab, open URL: https://pfipaddresspf
=> Page got loaded with the new Certificate.

### Set RAM Disk

1. dialog "**Status / Dashboard**"
     - select menu option "**System** > **Advanced**"
2. dialog "**System / Advanced / Admin Access**"
     - navigate to tab "**Networking**"
3. dialog "**System / Advanced / Networking**"
     - focus on section "**DHCP Options**"
     - select "**Kea DHCP**" as "**Server Backend**"
     - focus on section "**IPv6 Options**"
     - ~~disable~~ option "**Allow IPv6**"

tab "**Miscellaneous**"

---

# Show host name at login

menu option "**System** > **General Setup**"

section "**webConfigurator**"

enable option "**Login hostname**"

click "Save"


# Upload Certificate for webconfigurator

menu option "**System** > **Certificates**"

tab "**Authorities**"

section "**Create / Edit CA**"

enter "own CA" into "**Descriptive name**"

select "Import an existing Certificate Authority" for "**Method**"

enable option "**Trust Store**"

section "**Existing Certificate Authority**"

in field "**Certificate data**" insert content of file ".crt"

click on "**Save**"

tab "**Certificates**"

click "**Add/Sign**"

in section "**Add/Sign a New Certificate**"

select "**Import an existing Certificate**" for "**Method**"

type in "Fuji" for "**Description name**"

in section "**Import Certificate**"

in field "**Certificate data**" insert content of file ".crt"

in field "**Private key data**" insert content of file ".key" (must not have a passphrase)

click on "**Save**"

menu option "**System** > **Advanced**"

tab "**Admin Access**"

section "**webConfigurator**"

select "Fuji" (from previous step) as "**SSL/TLS Certificate**"

enable option "**WebGUI redirect**"

click on "**Save**"

---

# Block Adv & Porn

pfBlockerNG

https://linuxincluded.com/block-ads-malvertising-on-pfsense-using-pfblockerng-dnsbl/

https://debianforum.de/forum/viewtopic.php?t=170376

https://www.reddit.com/r/PFSENSE/comments/6wju5e/how_to_test_pfblokerng_is_working/?rdt=40523

---

# UPS

# Wake on LAN

[Docs > pfSense® software > Services > Wake on LAN](https://docs.netgate.com/pfsense/en/latest/services/wake-on-lan.html)


# Based on

YouTube: Let's Bridge These Ports - OPNsense (Jason's Lab)
https://www.youtube.com/watch?v=q1Rv4gB8fkI

[How to setup DMZ on pFSense](https://www.informaticar.net/how-to-setup-dmz-on-pfsense/)
