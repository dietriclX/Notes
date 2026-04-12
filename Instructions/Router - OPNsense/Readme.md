OPNsense

# Used Parameters/Values

Assuming the user name is `janedoe`, this document can be adjusted - replacing the placeholders by their values - using the following command.

```console
./apply.sh "/home/janedoe/Downloads/Notes-main/Instructions/Router - OPNsense/Readme.md" "/home/janedoe/Documents/OPNsense - Readme.md"
Default Mapping File: /home/janedoe/scripts/fwsense/data/parameter_value.Default.map
Source File: /home/janedoe/Downloads/Notes-main/Instructions/Router - OPNsense/Readme.md
Target File: /home/janedoe/Documents/OPNsense - Readme.md
```

The shell script directory `fwsense` is taken from [GitHub: dietriclX / Automation / Router - OPNsense](https://github.com/dietriclX/Automaton/tree/main/Router%20-%20OPNsense).

## General

- The Machine (becomes the OPNsense system)
     - Administrator
          - Password of the `adfwopnsenseadminaccountfwmin` user: `fwadminaccountfw` (default is `fwopnsenseadminpasswordfw`)
     - Hostname of the OPNsense system: `fwhostnamefw`
     - Domain name served by the OPNsense system: `fwhostdomainfw`
     - Network Devices
          - LAN #1 Interface
               - Name: `fwlan0interfacefw`
               - MAC-Address: `fwlan0macaddressfw`
          - LAN #2 Interface
               - Name: `fwlan1interfacefw`
               - MAC-Address: `fwlan1macaddressfw`
          - LAN #3 Interface
               - Name: `fwlan2interfacefw`
               - MAC-Address: `fwlan2macaddressfw`
          - LAN #3 Interface
               - Name: `fwlan3interfacefw`
               - MAC-Address: `fwlan3macaddressfw`
          - WAN Interface
               - Name: `fwwaninterfacefw`
               - MAC-Address: `fwwanmacaddressfw`
     - IPv4-Addresses
          - Router
               - WAN IPv4 Address: `rtlanipv4addressrt`
          - LAN Gateway
               - LAN IPv4 Address of the OPNsense system: `fwlanipv4addressfw`
               - LAN IPv4 Address range: `fwlandhcpstartipv4addressfw` - `fwlandhcpendipv4addressfw`
          - WAN IPv4 Address of the OPNsense system: `fwwanipaddressfw`
- The Workstation (used for administration of the pfSense system)
     - Host name: `wshostws`
     - Software
  	     - OS: Debian 12 "bookworm"
     - OS Users
          - Regular User
               - Account: `wssysuserws`
 
## Optional Parameters/Values

- IPv6-Addresses
     - LAN Gateway
          - LAN IPv6 Address of the OPNsense system: `fwlanipv6addressfw`
          - LAN IPv6 Address range: `fwlandhcpstartipv6addressfw` - `fwlandhcpendipv6addressfw`
- Demilitarized zone (DMZ)
     - IP-Addresses
          - DMZ Gateway
               - DMZ IPv4 Address: `fwdmzipv4addressfw`
               - DMZ IPv4 Range: `fwdmzdhcpstartipv4addressfw` - `fwdmzdhcpendipv4addressfw`
               - DMZ IPv6 Address: `fwdmzipv6addressfw`
               - DMZ IPv6 Range: `fwdmzdhcpstartipv6addressfw` - `fwdmzdhcpendipv6addressfw`
          - Web-Servers
               - DMZ IPv4 Address: `aohostipv4addressao`
               - DMZ IPv6 Address: `aohostipv6addressao`
- Uninterruptible Power Supply (UPS)
     - USB Device Name (either `auto` or device name incl. `/dev/`): `fwupsusbfw`
     - Name of UPS, as used within OPNsense: `fwupsnamefw`
     - Serial number of APC Back-UPS (12 hex-digits): `fwupsserialnrfw`
- Certificates
     - Name (CN part) of the own Certificate Authrority: `cacommonnameca`
     - `.crt` file name of the own CA: `cacrtfileca`
     - Name (CN part) of the Intermediate CA from your own CA: `cicommonnameci`
     - `.crt` file name of the Intermediate CA from your own CA: `cicrtfileci`
     - `.key` file name of the Intermediate CA from your own CA: `cikeyfileci`
     - `.crt` file name for the pfSense webConfigurator: `fwcrtfilefw`
     - `.key` file name for the pfSense webConfigurator: `fwkeyfilefw`

# Installation

Open URL [Download - OPNsense](https://opnsense.org/download/). In section "**Fast download selector**" do the following selection and click on "**Download OPNsense**" to download the OPNsense install image.

System architecture: amd64
Select the image type: vga
Mirror Location: OPNsense

## Prerequisites

- A Router
  - LAN interface - LAN Cable #1
- A Workstation
  - LAN interface - LAN Cable #2
- An Intel/AMD 64-bit machine
  - DVI/HDMI/VGA interface - Monitor
  - LAN interface fwwaninterfacefw- LAN Cable #1
  - LAN interface fwlan0interfacefw - n/a
  - LAN interface fwlan1interfacefw - n/a
  - LAN interface fwlan2interfacefw - n/a
  - LAN interface fwlan3interfacefw - LAN Cable #2
  - USB interface - Keyboard
  - USB interface - USB-Stick with OPNsense install image.

Note #1: *The "LAN interface fwwaninterfacefw" is the onboard LAN interface. The "LAN interface fwlan0interfacefw" to "LAN interface fwlan3interfacefw" are those from the LAN PCI Express card. Use the "LAN interface fwlan3interfacefw" to avoid mixing up the order of interfaces, once the "LAN Bridge" got setup.*

Note #2: *Once the system is good enough configured to be accessible via the browser, Monitor & Keyboard are no longer required, but the Workstation needs to be connected to the LAN interface.*

## The goal

For the internal SDD card, we will be using the standard File System ZFS.

The machine has one on-board LAN interface (fwwaninterfacefw) and a quad port LAN card (fwlan0interfacefw-fwlan3interfacefw). The interfaces will be defined as follows.

- WAN  -> fwwaninterfacefw; DHCP Client
- LAN  -> fwlan3interfacefw; Static IP Address `fwlanipv4addressfw` with DHCP Server service IP-Range `fwlandhcpstartipv4addressfw` to `fwlandhcpendipv4addressfw`

## Start-UP

In the BIOS Setting, enable boot from USB & removable media.

With the USB Media inserted, press F12 during boot to get the option to choose a boot partition.

```code
[UEFI: Mass Storage Device 1.00]
Mass Storage Device 1. 00
P1: Innodisk DEMSR-08GB mSATAl
Diagnostic Program
```

- select **`Mass Storage Device 1.  00`**
- ⬆ press **ENTER**

OPNsense Live image boots ...

## Configuration - 1. Part: CUI-Wizard

```code
:
Welcome!  OPNsense is running in live mode from install media. Please
login as 'fwopnsenseadminaccountfw' to continue in live mode, or as 'fwopnsenseinstalleraccountfw' to start the
installation.  Use the default or previously-imported root password for
both accounts.  Remote login via ssh is also enabled.

FreeBSD/amd64 (OPNsense.internal) (ttyv0)

login: █
```

- ✏️✓ type **`fwopnsenseinstalleraccountfw`** as login name
- ✓ press **ENTER**
- ✏️✓ type **`fwopnsenseinstallerpasswordfw`** as password
- ✓ press **ENTER**

New dialog `**Keymap Selection**` appears.

``` code
The system console driver for FreeBSD defaults to standard "US" keyboard map. Other keymaps can be chosen below.

>>> Continue with default keymap
->- Test default keymap
:
```

- ✏️✓ select your keyboard layout
- ✏️✓ press **ENTER**
- ✏️✓ **`>>> Continue with *.kbd keymap`**
- ✓ press **ENTER**

New dialog `**OPNsense 25.7**` appears.

``` code
Choose one of the following tasks to perform.

Install (ZFS)   ZFS GPT/UEFI Hybrid
Install (UFS)   UFS GPT/UEFI Hybrid
Other Modes >>  Extended Installation
Import Config   Load Configuration
Password Reset  Recover Installation
Force Reboot    Reboot System
Force Halt      Power Down System
```

- ✏️✓ select **`Other Modes >>`**
- ✓ press **ENTER**

New dialog `**Select Task**` appears.

``` code
Choose one of the following tasks to perform.

Auto (UFS)  Guided Disk Setup
Auto (ZFS)  Guided Root-on-ZFS
Manual      Manual Disk Setup (experts)
```

- ✏️✓ select **`Auto (ZFS)`**
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Configure Options:

>>> Install          Proceed with Installation
T Pool Type/Disk:    stripe: 0 disks
- Rescan Devices     *
- Disk Info          *
N Pool Name          zroot
4 Force 4K Sectors?  YES
E Encrypt Disks?     NO
P Partition Scheme   GPT (BIOS)
S Swap Size          2g
M Mirror Swap?       NO
W Encrypt Swap?      NO
O ZFS Pool Options   -O compress=lz4 -O atime=off
```

- ✏️✓ select **`T Pool Type/Disk:`**
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Select Virtual Device type:

stripe  Stripe - No Redundancy
mirror  Mirror - n-Way Mirroring
raid10  RAID 1+0 - n x 2-Way Mirrors
raidz1  RAID-Z1 - Single Redundant RAID
raidz2  RAID-Z2 - Double Redundant RAID
raidz3  RAID-Z3 - Triple Redundant RAID
```

- ✏️✓ select **`stripe`**
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Please select one or more disks to create a pool:

[ ] ada0  Innodisk DEMSR - 08GB mSATA 3ME A
```

- ✏️✓ press **SPACE**
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Configure Options:

>>> Install          Proceed with Installation
T Pool Type/Disk:    stripe: 1 disk
:
```

- ✏️✓ select **`S Swap Size`**
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Please enter amount of swap space (SI-Unit suffixes
recommended; e.g., `2g` for 2 Gigabytes):

2g
```

- ✏️✓ remove value, leaving the field empty
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Configure Options:

>>> Install          Proceed with Installation
T Pool Type/Disk:    stripe: 1 disk
:
S Swap Size          0
:
```

- ✏️✓ select **`>>> Install`**
- ✓ press **ENTER**

New dialog `**ZFS Configuration**` appears.

``` code
Last Chance! Are you sure you want to destroy
the current contents of the following disks:

 ada0
```

- ✏️✓ select **`< YES >`**
- ✓ press **ENTER**

New dialog `**Installation Progress**` appears.

``` code
Cloning current system             [     0%      ]

Overall Progress
                      0%
```

simply wait until its finished the task

New dialog `**Final Configuration**` appears.

``` code
Setup of your OPNsense system is nearly
complete.

Root Password     Change root password
Complete Install  Confirm and exit
```

- ✏️✓ select **`Complete Install`**
- ✓ press **ENTER**

### 1st Reboot

New dialog `**Installation Complete**` appears.

``` code
The system may boot back into the
installation media when not ejected
properly.

Reboot now  Reboot system
Halt now    Power down system
```

- ✏️✓ select **`Halt now`**
- ✓ press **ENTER**

System gets powered off.

- the move the install media.

## Configuration - 2. Part: Console

After the first boot, you will be greeted with the following information and available menu options.

```code
*** OPNsense.internal: OPNsense 25.7 (amd64) ***

 LAN (igb0)      -> v4: fwopnsenselanipv4addressfw/fwopnsenselanipv4cidrfw
 WAN (igb1)      -> 

 HTTPS: sha256 ...
               ...

FreeBSD/amd64 (OPNsense.internal) (ttyv0)

login:
```

- ✏️✓ type **`fwopnsenseadminaccountfw`** for login account
- ✏️✓ press **ENTER**
- ✏️✓ type **`fwopnsenseadminpasswordfw`** for login password
- ✓ press **ENTER**


```code
-----------------------------------------------
|      Hello, this is OPNsense 25.7           |           :::::::.
|                                             |           :::::::::.
| Website:     https://opensense.org/         |        :::        :::
| Handbook:    https://docs.opnsense.org/     |        :::        :::
| Forums:      https://forum.opnsense.org/    |        :::        :::
| Code:        https://github.com/opnsense    |         `:::::::::
| Reddit:      https://reddit.com/r/opnsense  |           `:::::::
-----------------------------------------------

*** OPNsense.internal: OPNsense 25.7 (amd64) ***

 LAN (igb0)      -> v4: fwopnsenselanipv4addressfw/fwopnsenselanipv4cidrfw
 WAN (igb1)      -> 

 HTTPS: sha256 ...
               ...

0) Logout                                7) Ping host
1) Assign interfaces                     8) Shell
2) Set interface IP address              9) pfTop
3) Reset the root password              10) Firewall Logs
4) Reset to factory defaults            11) Reload all services
5) Power off system                     12) Update from console
6) Reboot system                        13) Restore a backup

Enter an option: █
```

### LAN/WAN Interface

- type **`1`** for menu option "**1) Assign interfaces**"
- ⬆ press **ENTER**

```code
Do you want to configure LAGGs now? [y/N]: █
```

- ✏️✓ type **`n`**
- press **ENTER**

```code
Do you want to configure VLANs now? [y/N]: █
```

- ✏️✓ type **`n`**
- press **ENTER**

The entry details of option "Assign Interfaces" are shown.

```code
Valid interfaces are:

igb0             fwlan0macaddressfw ...
igb1             fwlan1macaddressfw ...
igb2             fwlan2macaddressfw ...
igb3             fwlan3macaddressfw ...
re0              fwwanmacaddressfw ...

If you do not know the names of your interfaces, you may choose to use
auto-detection. In that case, disconnect all interfaces now before
hitting 'a' to initiate auto detection.

Enter the WAN interface name or 'a' for auto-detection: █
```

- ✏️✓ type **`re0`**
- press **ENTER**

```code
Enter the LAN interface name or 'a' for auto-detection
NOTE: this enables full Firewalling/NAT mode.
(or nothing if finished): █
```

- ✏️✓ type **`igb3`**
- press **ENTER**

```code
Enter the Optional 1 interface name or ‚a‘ for auto-detection
(igb0 igb1 igb2 a or nothing if finished): █
```

- press **ENTER**

```code
The interfaces will be assigned as follows:

WAN  -> re0
LAN  -> igb3

Do you want to proceed [y|n]? █
```

- ✏️✓ type **`y`**
- ✓🔙 press **ENTER**

Leaving the "**Assign Interfaces**" and pfSense taking the configuration to update the WAN and LAN interfaces.

```code
Writing configuration…done.
:

*** OPNsense.internal: OPNsense 25.7 (amd64) ***

 LAN (igb3)      -> v4: fwopnsenselanipv4addressfw/fwopnsenselanipv4cidrfw
 WAN (re0)       -> v4/DHCP4: rtdhcpwanradnomipv4addressrt/rtlanipv4cidrrt
                    v6/DHCP6: rtdhcpwanradnomipv6addressrt/rtlanipv6cidrrt

 HTTPS: sha256 ...
               ...

0) Logout                                7) Ping host
1) Assign interfaces                     8) Shell
2) Set interface IP address              9) pfTop
3) Reset the root password              10) Firewall Logs
4) Reset to factory defaults            11) Reload all services
5) Power off system                     12) Update from console
6) Reboot system                        13) Restore a backup

Enter an option: █
```

### LAN Address(es)

- type **`2`** for menu option "**2) Set interface IP address**"
- ⬆ press **ENTER**

The entry details of option "Set interface(s) IP address" are shown.

```code
Available interfaces:

1 - LAN (igb3 - static, track6)
2 - WAN (re0 - dhcp, dhcp6)

Enter the number of the interface you wish to configure: █
```

- type **`1`** to configure the LAN interface
- ⬆ press **ENTER**

#### LAN IPv4

```code
Configure IPv4 address LAN interface via DHCP? [y/N] █
```

- type **`n`**
- press **ENTER**

```code
Enter the new LAN IPv4 address.  Press <ENTER> for none:
> █
```

- ✏️✓ type **`fwlanipv4addressfw`**
- press **ENTER**

```code
Subnet masks are entered as bit counts (as in CIDR notation).
e.g. 255.255.255.0 = 24
     255.255.0.0   = 16
     2555.0.0.0    = 8

Enter the new LAN IPv4 subnet bit count (1 to 32):
> █
```

- ✏️✓ type **`fwhostdomainfw`**
- press **ENTER**

```code
For a WAN, enter the new LAN IPv4 upstream gateway address.
For a LAN, press <ENTER> for none:
> █
```

- press **ENTER**

#### LAN IPv6

```code
Configure IPv6 address LAN interface via WAN tracking? [Y/n] █
```

- type **`n`**
- press **ENTER**

```code
Configure IPv6 address LAN interface via DHCP6? [y/N] █
```

- type **`n`**
- press **ENTER**

```code
Enter the new LAN IPv6 address.  Press <ENTER> for none:
> █
```

- press **ENTER**

### LAN DHCP (IPv4)

```code
Do you want to enable the DHCP server on LAN? [y/N] █
```

- ✏️✓ type **`y`**
- press **ENTER**

```code
Enter the start address of the IPv4 client address range: █
```

- ✏️✓ type **`fwlandhcpstartipv4addressfw`**
- press **ENTER**

```code
Enter the end address of the IPv4 client address range: █
```

- ✏️✓ type **`fwlandhcpendipv4addressfw`**
- press **ENTER**

### Web GUI procotol

```code
Do you want to change the web GUI protocol from HTTPS to HTTP? [y/N] █
```

- ✏️✓ type **`n`**
- ✓ press **ENTER**

```code
Do you want to generate a new self-signed web GUI certificate? [y/N] █
```

- ✏️✓ type **`y`**
- ✓ press **ENTER**

```code
Restore web GUI access defaults? [y/N] █
```

- ✏️✓ type **`y`**
- ✓ press **ENTER**

Leaving the "**Set interface IP address**" and OPNsense taking the configuration to update the WAN and LAN interfaces.

```code
Writing configuration…done.
:

You can now access the web GUI by opening
the following URL in your web browser:

    https://fwlanipv4addressfw

*** OPNsense.internal: OPNsense 25.7 (amd64) ***

 LAN (igb3)      -> v4: fwhostdomainfw
 WAN (re0)       -> v4/DHCP4: rtdhcpwanradnomipv4addressrt/rtlanipv4cidrrt
                    v6/DHCP6: rtdhcpwanradnomipv6addressrt/rtlanipv6cidrrt

 HTTPS: sha256 ...
               ...

0) Logout                                7) Ping host
1) Assign interfaces                     8) Shell
2) Set interface IP address              9) pfTop
3) Reset the root password              10) Firewall Logs
4) Reset to factory defaults            11) Reload all services
5) Power off system                     12) Update from console
6) Reboot system                        13) Restore a backup

Enter an option: █
```

- type **`5`** to shutdown the system
- ⬆ press **ENTER**

```code
The system will halt and power off. Do you want to proceed? [y/N]: █
```

- type **`y`** to confirm the shutdown
- ✓🔙 press **ENTER**

System shutdown and powered off.

- ✏️ disconnect keyboard to test that even without a keyboard the system boot without any trouble
- start system once more to check the interface assignment

```code
*** OPNsense.internal: OPNsense 25.7 (amd64) ***

 LAN (igb3)      -> v4: fwlanipv4addressfw/fwlanipv4cidrfw
 WAN (re0)       -> v4/DHCP4: rtdhcpwanradnomipv4addressrt/rtlanipv4cidrrt
                    v6/DHCP6: rtdhcpwanradnomipv6addressrt/rtlanipv6cidrrt

 HTTPS: sha256 ...
               ...

FreeBSD/amd64 (OPNsense.internal) (ttyv0)

login:
```

It is correct, so you can turn off the system.

- briefly press the "power-on“ button

System shutdown and powered off.

## Configuration - 3. Part: Web GUI

- start the OPNsense machine

- open URL: https://fwlanipv4addressfw
- login with `fwopnsenseadminaccountfw`/`fwopnsenseadminpasswordfw`

It is the first time, therefore the wizard shows up

New dialog "**System**" > "**Configuration**" > "**Wizard**" appears with tab "**Welcome**".

> Welcome
> This wizard will guide you through the initial system configuration. The wizard may be stopped at any time by clicking the logo image at the top of the screen.

- click on "**Next**"

New tab "**General Information**" appears.

- ✏️✓ type **`fwhostnamefw`** as "**Hostname**"
- ✏️✓ type **`fwhostdomainfw`** as "**Domain**"

- keep **`Etc/UTC`** as "**Timezone**"
- keep "**DNS Server**" empty
- keep **`Overide DNS`** enabled
- keep **`Enable Resolver`** enabled
- click on "**Next**"
- 
New tab "**Network [WAN]**" appears.

- keep **`DHCP`** as "**Type**"
- keep **`Block RFC1918 Private Networks`** enabled
- keep **`Block bogon networks`** enabled
- click on "**Next**"

New tab "**Network [LAN]**" appears.

- keep **`Disable LAN`** ~~disabled~~
- keep **`fwlanipv4addressfw/fwlanipv4cidrfw`** as "**IP Address**"
- keep **`Configure DHCP server`** enabled
- click on "**Next**"

New tab "**Set initial password**" appears.

- ✏️✓ type **`fwfwsenseadminpasswordfw`** as "**Root Password**"
- ✏️✓ type **`fwfwsenseadminpasswordfw`** as "**Root Password Confirmation**"
- click on "**Next**"

New tab "**Finish**" appears.

- ✓ click on "**Apply**"

Message "**Finished initial configuration!**" appears.

> Congratulations! OPNsense is now configured.
> 
> Please consider donating to the project to help us with our overhead costs. See [our website](https://opnsense.org/) to donate or purchase available OPNsense support services.
> 
> Click to [continue to the dashboard](https://fwlanipv4addressfw/). Or click to [check for updates](https://fwlanipv4addressfw/ui/core/firmware#checkupdate).

- click on "**[check for updates](https://fwlanipv4addressfw/ui/core/firmware#checkupdate)**"

New dialog "**System**" > "**Firmware**" > "**Updates**" appears.

New message "**Firmware status**" appears.

> The release type "opnsense" is not available on this repository.

- click on "**Close**"

```code
***GOT REQUEST TO CHECK FOR UPDATES***
Currently running OPNsense 25.7 (amd64) at Sat Dec 13 11:06:14 UTC 2025
Fetching changelog information, please wait... done
Updating OPNsense repository catalogue...
Fetching meta.conf: . done
Fetching packagesite.pkg: .......... done
Processing entries: .......... done
OPNsense repository update completed. 909 packages processed.
All repositories are up to date.
Updating database digests format: .......... done
New version of pkg detected; it needs to be installed first.
The following 1 package(s) will be affected (of 0 checked):

Installed packages to be UPGRADED:
	pkg: 1.19.2_5 -> 2.3.1_1

Number of packages to be upgraded: 1

The process will require 9 MiB more space.
6 MiB to be downloaded.
[1/1] Fetching pkg-2.3.1_1.pkg: .......... done
Checking integrity... done (0 conflicting)
[1/1] Upgrading pkg from 1.19.2_5 to 2.3.1_1...
[1/1] Extracting pkg-2.3.1_1: .......... done
pkg: Repository OPNsense cannot be opened. 'pkg update' required
Checking integrity... done (0 conflicting)
Your packages are up to date.
self: No packages available to install matching 'opnsense'
***DONE***
```

The line **`New version of pkg detected; it needs to be installed first.`** indicates, that the following additional steps are required to update the firmware.

- navigate to tab "**Status**"
- click on "**🗘 Check for updates**"

New message "**25.7.9**" appears.

> What is up everyone,
> 
> A bug snuck into the last release that did not properly disable the caching of DNS entries when using multiple blocklists with different network restrictions. We have used the opportunity to polish the notification code and apply behaviour during the migration of the old blocklist to the new format.
> 
> The saga around safe command execution continues in this release as well. Otherwise it is a rather quiet release and 2025 is almost over. Happy holidays!
> 
> Here are the full patch notes:

with a list of packages ...

- click on "**Close**"
- scroll to the bottom of the page
- click on "**✔️ Update**"

New message "**Reboot required**" appears.

> The firewall will reboot directly after this firmware update.

- click on "**OK**"

Start of applying the updates.

In case of an issue, a message "**Danger**"appears.

> Unexpected error, check log for details

- click on "**Close**"

Do check the log for any suspicious messages. 

New message "**Your system is rebooting**" appears.

> The upgrade has finished and the system is being rebooted at the moment, please wait...

The played tunes will indicate the shutdown and later the startup of the system.

Open menu "**System** > **Firmware** > **Status**"

- click on "**🗘 Check for Updates**"

Just to be on the save side, that the system is really up-to-date.

# Basic Configuration - 4. Part: Web GUI

## Create own admin user

See [User - Create Own Admin User](#user---create-own-admin-user)

## RAM Disk (tmpfs)

- ⬆ navigate to menu option "**System** > **Settings** > **Miscellaneous**"
     - scroll down to section "Disk / Memory Settings (reboot to apply changes)"
     - ✔️✏️ enable option "**/var/log RAM disk**" ("Use memory file system for /var/log")
     - ✔️✏️ type **`25`** in field "**/var/log RAM usage**"
     - ✔️✏️ enable option "**/tmp RAM disk**" ("Use memory file system for /tmp")
     - ✔️✏️ type **`38`** in field "**/tmp RAM usage**"
     - ✔️ click on "**Save**"

- ⬆ navigate to menu option "**Power** > **Reboot**"
     - ✔️ click on "**Yes**" in regards to question "Are you sure you want to reboot the system?"

## Network Time Protocol (NTP) Server

- ⬆ select menu option "**Services** > **Network Time** > **General**"
	- focus on section "**NTP Server Configuration**"
	- ✏️✓ type **`ptbtime1.ptb.de`** in "**Time servers**" at first line
	- click on "**Save**"

## DNS Server of Router

- ⬆ select menu "**Services** > **Unbound DNS** > **Query Forwarding**"
     - ✏️✓ enable "**Use System Nameservers**"
     - ✓ click on "**Apply**"

Only if a static IP address has been assigned to the WAN interface, this configuration is required.

- ⬆ select menu option "**System** > **Settings** > **General**"
     - focus on section "**Networking**"
     - ✏️✓ type **`rtlanipv4addressrt`** at "**DNS Servers**"
     - ✓ click on "**Save**"

## Specify LAN Domain

This section is relevant, in case the "**LAN**" domain "**fwlandomainfw**" is different to the routers domain "**fwhostdomainfw**".

- ⬆ select menu option "**Services > Dnsmasq DNS & DHCP** > **DHCP ranges**"
     - navigate to tab "**DHCP ranges**"
     - click on "**🞂**" of interface "**LAN**" to see its ip range
     - ⬆ click on "**🖉**" of interface "**LAN**" (to be on the right)
          - ✏️✓ type **`fwlandomainfw`** into "**Domain Name**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply**"


# Add-On

## Certificate Usage

Depending on the scenario, for which the router is planned to be used, the (Intermediate) Certificate Authority and Certificate for the webConfigurator are configurated differently.

### Scenario: Router without OpenVPN

It is planned to 

- not setup and manage an OpenVPN server on the pfSense system
- not use Client Certificates for user authentication on the pfSense system
- create the certfiate for the webConfigurator outside the pfSense system

Therefore only the CA Certificate without the key has to be installed.

Steps to follow are

1. "Configure Certificate Authority"
2. "Configure Web Certificate"
3. "Activate Own Web Certificate"
4. "Test Own Web Certificate"

### Scenario: pfSense system with OpenVPN

It is planned to 

- setup and manage an OpenVPN server on the pfSense system
- use Client Certificates for user authentication on the pfSense system
- maintain the (Client) Certfiate within the pfSense system

Therefore an Intermediate CA Certificate has to be prepared on the workstation and installed on the pfSense system. In result the pfSense system will become capable of creating its required certificates.

Steps to follow are

1. "Configure Certificate Authority"
2. "Configure Intermediate Certificate Authority"
3. "Configure Web Certificate"
4. "Activate Own Web Certificate"
5. "Test Own Web Certificate"

### Configure Certificate Authority

- ⬆ select menu option "**System** > **Trust** > **Authorities**"
     - navigate to tab "**Certificates**"
     - ⬆ click "**➕**"
          - ✏️✓ select **`Import an existing Certificate`** as "**Method**"
          - ✏️✓ type **`cacommonnameca`** (the CN part of the certificate) as "**Descriptive name**"
          - open the `cacrtfileca` file and copy the content into the clipboard
          - ✏️✓ paste the content of the clipboard into "**Certificate data**"
          - ✓🔙 click "**Save**"

### Configure Intermediate Certificate Authority

When you have setup already ACNO, only a few more steps are required to create those files required.

See [Add-On > Certificates > Intermediate Certificate Authority](../Debian%2012%20-%20ACNO/Setup/Readme.md#intermediate-certificate-authority)

| Input File | From the ACNO instruction |
| :--- | :--- |
| `cicrtfileci` | `~/SSL.canameca/CA.intermediate/SSL.canameca.intermediate.crt` |
| `cikeyfileci` | `~/SSL.canameca/CA.intermediate/SSL.canameca.intermediate.unencrypted.key` |

- ⬆ select menu option "**System** > **Trust** > **Certificates**"
     - navigate to tab "**Certificates**"
     - ⬆ click "**➕**"
          - ✏️✓ type **`cicommonnameci YYMMDD`** as "**Descriptive name**"¹
          - ✏️✓ select **`Import an existing Certificate`** as "**Method**"
          - open the `cicrtfileci` file and copy the content into the clipboard
          - ✏️✓ paste the content of the clipboard into "**Certificate data**"
          - open the `cikeyfileci` file and copy the content into the clipboard
          - ✏️✓ paste the content of the clipboard into "**Private Key data**"
          - ✓🔙 click "**Save**"

¹ **`YYMMDD`** representing the "Valid From" date.

### Configure Web Certificate

- ⬆ select menu option "**System** > **Trust** > **Certificates**"
     - navigate to tab "**Certificates**"
     - ⬆ click "**➕**"
          - ✏️✓ select **`Create an internal Certificate`** as "**Method**"
          - ✏️✓ type "**Web GUI fwhostnamefw (fwhostdomainfw) YYMMDD**" into "**Descriptive name**"¹
          - focus on section "**Key**"
          - ✏️✓ select **`Server Certificate`** as "**Type**"
          - keep **`Save on this firewall`** as "**Private key location**"
          - ✏️✓ select **`RSA-4096`** as "**Key type**"
          - keep **`SHA-256`** as "**Digest Algorithm**"
          - ✏️✓ select **`cicommonnameci`** as "**Issuer**"
          - ✏️✓ type **`398`** into "**Lifetime (days)**"
          - focus on section "**General**"
          - ✏️✓ type **`ciorgunitci`** into "**Organisational Unit**"
          - ✏️✓ type **`fwhostnamefw.fwhostdomainfw`** into "**Common Name**"
          - focus on section "**Alternative Names**" (expand the section by a click on the name)
          - ✏️✓ type **`fwhostnamefw`** into "** DNS domain names**"²
          - ✏️✓ add **`fwhostnamefw.fwhostdomainfw`** into "** DNS domain names**"²
          - ✓🔙 click "**Save**"

¹ **`YYMMDD`** representing the "Valid From" date.

² Other host/domain names can be added as well. One line per entry. For example from a backup machine.

### Activate Own Web Certificate

- ⬆ select menu option "**System** > **Settings** > **Administration**"
     - focus on section "**Web GUI**"
     - ✏️✓ select newly created Certificate as "**SSL Certificate**"
     - ✓ click on "**Save**"

### Test Own Web Certificate

- close browser tab
- in new browser tab, open URL: https://fwhostnamefw.fwhostdomainfw
=> Page got loaded with the new Certificate.
- change URL to: https://fwhostnamefw
=> Page got loaded with the new Certificate.

## DHCP Client - Fix IP-Address

In some cases it is desired to have a system within the DMZ/LAN network with a fixed IP-Address. Usually we would specify the IP-Address on the device itself. As an alternative, the pfSense own DHCP/DHCPv6 Server can be used to achieve the same.

Beside the IP-Address(es), another prerequisite is to have the MAC address for IPv4 and/or DHCP Unique Identifier (DUID) for IPv6 at hand.

- DHCP Server: the MAC address¹ of the client device
- DHCPv6 Server: DHCP Unique Identifier (DUID)¹ of the client device
- DHCP/DHCPc6 Server: the client device IP-Address(es)

¹ Especially DUID has to be taken from the lease page of OPNsense system; See menu option "**Services > Dnsmasq DNS & DHCP** > **Leases**".

Once the static mapping is completed, the client device has to be rebooted to get the assigned IP-Address.

Note: *In case of an Wifi Interface, used by the client device, not randmon MAC address should be used!*

### DHCP Static Mappings (IPv4/IPv6)

- ⬆ select menu option "**Services** > **Dnsmasq DNS & DHCP** > **Hosts**"
     - navigate to tab "**Hosts**"
     - ⬆ click on "**➕**"
          - focus on section "**DNS**"
          - ✏️✓ type Host-Name as "**Host**"
          - ✏️✓ type Domain-Name as "**Domain**"
          - ✏️✓ type IP-Address as "**IP-Address**"
          - focus on section "**DHCP**"
          - ✏️✓ type DUID-Address as "**Hardware addresses**"
          - ✓🔙 click on "**Save**"
          - focus on section "**Informational**"
          - ✏️✓ type meaningful name as "**Description**"¹
     - click on "**Apply Changes**"

¹ Use either a descriptive name - everyone allows to identify the device - or the device brand and product name.

## DMZ creation

Makes only sense with a DSL-modem setup. See section "[DSL-Modem usage (as replacement for DSL-Router)](#dsl-modem-usage-as-replacement-for-dsl-router)"

Requires to have IPv6 enabled on pfSense. See section "[IPv6 Enablement](#ipv6-enablement)"

### Define Interface

Take **`pfdmzinterfacepf`** for the DMZ interface.  If necassary, take the interface out of the Bridge configuration. See instruction [Remove LAN3 from LAN Bridge ...](#remove-lan3-from-lan-bridge-quad-port-bridge-lan0-3).

In case pfdmzinterfacepf is not assigned yet, do the following steps.

- ⬆ select menu option "**Interfaces > Assignments**"
     - focus on section "**➕ Assign a new interface**"
     - ✏️✓ select **`pfdmzinterfacepf (pfdmzmacaddresspf)`** from listbox "**Device**"
     - ✏️✓ type **`DMZ`** into "**Description**"
     - ✏️✓ click on "**Add**"

Make the required changed to the configuration of the assigned interface.

- ⬆ select menu option "**Interfaces > Assignments**"
     - ⬆ click on the name (hyperlink) of interface pfdmzinterfacepf
          - focus on "**Basic configuration**"
          - ✏️✓ enable option "**Enable Interface**"
          - focus on "**Generic configuration**"
          - keep ~~disabled~~ "**Block private networks**"
          - keep ~~disabled~~ "**Block bogon networks**"
          - ✏️✓ select **`Static IPv4`** as "**IPv4 Configuration Type**"
          - ✏️✓ select **`Static IPv6`** as "**IPv6 Configuration Type**"
          - focus on "**Static IPv4 configuration**"
          - ✏️✓ type **`fwdmzipv4addressfw`** into "**IPv4 Address**"
          - ✏️✓ select **`fwdmzipv4cidrfw`** (CIDR) for "**IPv4 address**"
          - focus on "**Static IPv6 configuration**"
          - ✏️✓ type **`fwdmzipv6addressfw`** into "**IPv6 Address**"
          - ✏️✓ select **`fwdmzipv6cidrfw`** (CIDR) for "**IPv6 address**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

### Enable the DHCP IPv4 server

- ⬆ select menu option "**Services > Dnsmasq DNS & DHCP** > **DHCP ranges**"
     - navigate to tab "**DHCP ranges**"
     - ⬆ click on "**➕**"
          - ✏️✓ select **`DMZ`** as "**Interface**"
          - keep **`None`** as "**Block bogon networks**"
          - ✏️✓ type **`fwdmzdhcpstartipv4addressfw`** into "**Start address**"
          - ✏️✓ type **`fwdmzdhcpendipv4addressfw`** into "**End address**"
          - keep **`automatic`** as "**Subnet mask**"
          - ✏️✓ type **`fwdmzdomainfw`** into "**Domain Name**"
          - ✏️✓ type **`DMZ: IPv4`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply Changes**"

### Enable the DHCP IPv6 server

- ⬆ select menu option "**Services > Dnsmasq DNS & DHCP** > **DHCP ranges**"
     - navigate to tab "**DHCP ranges**"
     - ⬆ click on "**➕**"
          - ✏️✓ select **`DMZ`** as "**Interface**"
          - keep **`None`** as "**Block bogon networks**"
          - ✏️✓ type **`fwdmzdhcpstartipv6addressfw`** into "**Start address**"
          - ✏️✓ type **`fwdmzdhcpendipv6addressfw`** into "**End address**"
          - keep **`automatic`** as "**Subnet mask**"
          - ✏️✓ type **`fwdmzdomainfw`** into "**Domain Name**"
          - ✏️✓ type **`DMZ: IPv6`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply Changes**"

Note: *A red message will inform about the missing the following configuration.*

Specify the Router Mode for the DMZ (prerequisite for DHCP6).

- ⬆ select menu option "**Services** > **Router Advertisement**"
     - navigate to tab "**DMZ**"
     - focus on "**Router Advertisement**"
     - ✏️✓ select **`Assisted - RA Flags [managed, other stateful], Prefix Flags [onlink, auto, router]`** as "**Router Mode**"
     - ✓ click on "**💾 Save**"

### Alias: IP-Address of Web Server in DMZ

Define two Aliases for IP-Addresses of the web server. These Aliases will be used to allow/block communication.

- ⬆ select menu option "**Firewall > Aliases**"
     - ⬆ click on "**➕**"
          - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Name**"
          - keep **`Host(s)`** as "**Type**"
          - ✏️✓ type **`aohostipv4addressao`** into "**Content**"
          - ✏️✓ type **`Web Server IP in DMZ`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**⮺**" ("Clone") at the right of the newly created entry
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Name**"
          - ✏️✓ type **`aohostipv6addressao`** in "**Content**"
          - ✏️✓ type **`DMZ Web Server IPv6`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply**"

### Alias: Ports of Web Server in DMZ

Define some Aliases for all ports used by the web server. This Alias will be used to allow/block communication.

The ports in this example are for a system with Nextcloud, coturn and OpenVPN.

- ⬆ select menu option "**Firewall > Aliases**"
     - ⬆ click on "**➕**" ("Add")
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Name**"
          - ✏️✓ select **`Ports(s)`** as "**Type**"
          - ✏️✓ type the following ports into "**Content**"
               - 80
               - 443
               - 10000:20000
               - 3478
               - 5349
               - 1194
          - ✏️✓ type **`All ports served by the web server.`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Name**"
          - ✏️✓ select **`Ports(s)`** as "**Type**"
          - ✏️✓ type the following ports into "**Content**"
               - 10000:20000
               - 3478
               - 5349
               - 1194
          - ✏️✓ type **`All ports served by the web server.`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ type **`DMZ_Outbound_TCP`** into "**Name**"
          - ✏️✓ select **`Ports(s)`** as "**Type**"
          - ✏️✓ type the following ports into "**Content**"
               - 80
               - 443
               - 465
               - 587¹
               - 993¹
          - ✏️✓ type **`All ports to the internet requied by the web server.`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**" ("Add")
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Outbound_UDP`** into "**Name**"
          - ✏️✓ select **`Ports(s)`** as "**Type**"
          - ✏️✓ type the following ports into "**Content**"
               - 123
          - ✏️✓ type **`All ports to the internet requied by the web server.`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply**"

¹ Example for allowing as well the communication with IMAP and SMTP servers.

### Block DMZ to LAN communication

Prevent the DMZ to communicate with the LAN.

- ⬆ select menu option "**Firewall > Rules** > **DMZ**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ select **`Block`** as "**Action**"
          - keep enabled "**Quick**" ("Apply the action immediately on match.")
          - keep selected **`DMZ`** as "**Interface**"
          - keep selected **`in`** as "**Direction**"
          - ✏️✓ select **`IPv4+IPv6`** as "**TCP/IP Version**"
          - keep selected **`any`** as "**Protocol**"
          - ✏️✓ select **`DMZ address`** as "**Source**"
          - ✏️✓ select **`LAN address`** as "**Destination**"
          - keep selected **`any`** for "**from:**" and "**to:**" of "**Destination port range**"
          - ✏️✓ type **`Block DMZ to LAN communication`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - tick checkbox of the newly created line¹
     - click on **`🡐`** ("Move selected rules before this rule") of the first line²
     - click on "**Apply changes**"

¹ The checkbox with a checkmark in it, marks a line as selected.

### Allow DMZ to Firewall and Internet (minimum)

Allow the ICMP > Echo request from the web server to the internet.

- ⬆ select menu option "**Firewall > Rules** > **DMZ**"
     - ⬆ click on "**➕**" ("Add")
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - keep selected **`in`** as "**Direction**"
          - ✏️✓ select **`IPv4+IPv6`** as "**TCP/IP Version**"
          - ✏️✓ select **`TCP/UDP`** as "**Protocol**"
          - ✏️✓ select **`DMZ address`** as "**Source**"
          - ✏️✓ select **`This Firewall`** as "**Destination**"
          - keep selected **`any`** for "**from:**" and "**to:**" of "**Destination port range**"
          - ✏️✓ type **`Allow DMZ to Firewall communication`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**" ("Add")
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - keep selected **`in`** as "**Direction**"
          - keep selected **`IPv4`** as "**TCP/IP Version**"
          - ✏️✓ select **`ICMP`** as "**Protocol**"
          - ✏️✓ select only **`Echo Request`** as "**ICMP types**"
          - ✏️✓ select **`DMZ address`** as "**Source**"
          - keep **`any`** as "**Destination**"
          - keep selected **`any`** for "**from:**" and "**to:**" of "**Destination port range**"
          - ✏️✓ type **`Allow DMZ to Firewall communication`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

### Allow DMZ Internet communication (solution specific)

Allow TCP/IP communication from the web server to the internet.

- ⬆ select menu option "**Firewall > Rules** > **DMZ**"
     - ⬆ click on "**➕**" ("Add")
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - ✏️✓ select **`IPv4+IPv6`** as "**TCP/IP Version**"
          - keep **`TCP`** as "**Protocol**"
          - ✏️✓ select **`DMZ address`** as "**Source**"
          - keep **`any`** as "**Destination**"
          - ✏️✓ select **`DMZ_Outbound_TCP`** as "**from:**" of "**Destination Port Range**"
          -keep selected **`DMZ_Outbound_TCP`** into "**to:**" of "**Destination Port Range**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**" ("Add")
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - ✏️✓ select **`IPv4+IPv6`** as "**TCP/IP Version**"
          - keep **`UDP`** as "**Protocol**"
          - ✏️✓ select **`DMZ address`** as "**Source**"
          - keep **`any`** as "**Destination**"
          - ✏️✓ select **`DMZ_Outbound_UDP`** as "**from:**" of "**Destination Port Range**"
          - keep selected **`DMZ_Outbound_UDP`** into "**to:**" of "**Destination Port Range**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

### Enable Port Forwarding (WAN->DMZ)

Activate the port forwording (TCP/UDP) to the web server in the DMZ. The IPv6 entries are basically duplicates of the IPv4 entries, with the "4" changed to "6" in fields "**Address Family**"and "**Address**".

- ⬆ select menu option "**Firewall** > **NAT** > **Port Forward**"
     - ⬆ click on "**➕**" ("Add")
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - keep selected **`TCP`** as "**Protocol**"
          - ✏️✓ select **`WAN address`** as "**Destination**"
          - ✏️✓ select **`DMZ_Inbound_TCP`** as "**from:**" of "**Destination Port Range**"
          - keep selected **`DMZ_Inbound_TCP`** as "**to:**" of "**Destination Port Range**"
          - ✏️✓ select **`DMZ_Web_Server_IPv4`** as "**Redirect target IP**"
          - keep selected **`DMZ_Inbound_TCP`** as "**Redirect target port**"
          - ✏️✓ type **`Redirect Internet TCP traffic to Web Server.`** into "**Description**"
          - keep selected **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "** Save**"
     - ⬆ click on "**⮺**" ("Clone") at the right of the newly created entry
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ select **`DMZ_Web_Server_IPv6`** as "**Redirect target IP**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - ✏️✓ select **`WAN address`** as "**Destination**"
          - ✏️✓ select **`DMZ_Inbound_UDP`** as "**from:**" of "**Destination Port Range**"
          - keep selected **`DMZ_Inbound_UDP`** as "**to:**" of "**Destination Port Range**"
          - ✏️✓ select **`DMZ_Web_Server_IPv4`** as "**Redirect target IP**"
          - keep selected **`DMZ_Inbound_UDP`** as "**Redirect target port**"
          - ✏️✓ type **`Redirect Internet UDP traffic to Web Server.`** into "**Description**"
          - keep selected **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**⮺**" ("Clone") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Redirect target IP**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

¹ This option will automatically generate the Firewall Rule to allow Inbound-Traffic. See below section "[Inbound NAT associated filter rules](#inbound-nat-associated-filter-rules)"

### Enable Port Forwarding (LAN->DMZ)

Route the traffic, to the Web-Server, from the LAN directly to the Web-Server. The IPv6 entries are basically duplicates of the IPv4 entries, with the "4" changed to "6" in fields "**Address Family**"and "**Address**".

- ⬆ select menu option "**Firewall** > **NAT** > **Port Forward**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ select **`LAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`any`** as "**Protocol**"
          - ✏️✓ select **`WAN address`** as "**Destination**"
          - ✏️✓ select **`DMZ_Web_Server_IPv4`** as "**Redirect target IP**"
          - ✏️✓ type **`Redirect LAN TCP traffic to Web Server.`** into "**Description**"
          - keep selected **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "** Save**"
     - ⬆ click on "**⮺**" ("Clone") at the right of the newly created entry
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ select **`DMZ_Web_Server_IPv6`** as "**Redirect target IP**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

¹ This option will automatically generate the Firewall Rule to allow Inbound-Traffic. See below section "[Inbound NAT associated filter rules](#inbound-nat-associated-filter-rules)"

Note: *For the last rule the value for "**Filter rule association**" might be shown as "Rule NAT Redirect Internet TCP traffic to Web Server.", however once saved the value will be "Rule NAT Redirect Internet UDP traffic to Web Server.".*

The creation of the NAT - Port Forward rules should have created the corresponding rules to allow internet traffic to passed to the Web Server.

- ⬆ select menu option "**Firewall > Rules** > **WAN**"
     - click on (/) of the first line "*Automatically generated rules*"

## DSL-Modem usage (as replacement for DSL-Router)

At a later point in time, the DSL-Router can be replaced by a DSL-Modem, allowing pfServer to direct communicate with the internet.

Before we forget it, the changes of [**Allow DSL-Router LAN traffic**](#allow-dsl-router-lan-traffic) have to be revoked.

- ⬆ select menu option "**Interfaces** > **[WAN]**"
     - focus on "**Generic configuration**"
     - keep enableed "**Block private networks**" 
     - keep enableed "**Block bogon networks**"
     - ✓ click on "**Save**"
     - click on "**✔️ Apply Changes**"

The biggest change to make in the pfSense configuration is from a simple "WAN -> re0 (...)" to a chain of configurations "WAN -> PPPoE -> VLAN -> re0 (...)".

- ⬆ select menu option "**Interfaces** > **Devices** > **VLANs**"
     - ⬆ click on "**➕**" ("Add")
          - keep "**Device**" empty¹
          - ✏️✓ select **`fwwaninterfacefw (fwwanmacaddressfw)`** as "**Parent**"
          - ✏️✓ type **`dlvlaniddl`**² into "**VLAN Tag**"
          - ✏️✓ type **`ISP VLAN ID dlvlaniddl`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - navigate to tab "**PPPs**"
- ⬆ select menu option "**Interfaces** > **Devices** > **Point-to-Point**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ select **`PPPoE`** as "**Link Type**"
          - ✏️✓ select **`vlan01`** as "**Link Interface(s)**"
          - ✏️✓ type **`ISP DSL PPPoE`** as "**Description**"
          - ✏️✓ type **`dluserdl`**² as "**"Username**"
          - ✏️✓ type **`dlpassworddl`**² as "**Password**"
          - click on "**Show advanced options**"
          - keep ~~disabled~~ "**Dial On Demand**"
          - ✓🔙 click on "**Save**"
- ⬆ select menu option "**Interfaces** > **Assignments**"
     - ✏️✓ select **`pppoe0 (vlan01) - ISP DSL PPPoE dluserdl`** as "**Device**" of line **[WAN]**"
     - ✓ click on "**Save**"
     - ⬆ click on "**[WAN]**"
          - focus on "**General configuration**"
          - keep selected **`PPPoE`** as "**IPv4 Configuration Type**"
          - keep selected **`DHCPv6`** as "**IPv6 Configuration Type**"
          - focus on "**DHCPv6 client configuration**"
          - keep ~~disabled~~ "**Request prefix only**"
          - ✓ click on "**Save**"
          - click on "**Apply changes**"

¹ The device will become **`vlan01`**.

² In case of a Telekom DSL-Line, it is the VLAN-ID `7`.

² The DSL-User/Password has to be provided by the ISP. In case of the Telekom AG, the user name is `<Anschlusskennung><Internet Zugangsnummer><Mitbenutzernummer>@t-online.de` and the password of the Telekom-User.

- ⬆ select menu option "**Interfaces** > **Assignments**"

From the DNS perspective additional changes are required, in order to forward requests on the public domain to the server in the DMZ.

The following steps are required to a domain specifc "**Host Overrides**" entry in the "**DNS Resolver,

| Host | Parent domain of host | IP to return from host | Description | Actions |
| :-- | :--- | :--- | :--- | :--- |
| ddprefixdomaindd | ddbasedomaindd | aohostipv4addressao | DynDNS Provider Name | 🖉 🗑 |

with "**Additional Names for this Host**" entry in the "**Host Override**" definition.

| | | | |
| :--- | :--- | :--- | :--- |
| ddofficedd | ddprefixdomaindd.ddbasedomaindd | Online Office Solution | 🗑 Delete |
| ddturndd | ddprefixdomaindd.ddbasedomaindd | STURN/TURN Server | 🗑 Delete |

Note: Usually the DynDNS provider has a set of domain names, to be used as the base domain name (e.g. "ddbasedomaindd") and the own - individual - part is the prefix/host ("ddprefixdomaindd") to it. As IP is only the IPv4 address given, in case of IPv4+IPv6 the have to be specified by `aohostipv4addressao,aohostipv6addressao`.

- ⬆ select menu option "**Services** > **DNS Resolver**"
     - focus on "**Host Overrides**"
     - ⬆ click "**➕ Add**"
          - focus on "**Host Override Options**"
          - ✏️✓ type **`ddprefixdomaindd`** in "**Hosts**"
          - ✏️✓ type **`ddbasedomaindd`** in "**Domain**"
          - ✏️✓ type **`aohostipv4addressao`** in "**Domain**"
          - ✏️✓ type **`ddproviderdd`** in "**Description**"
          - focus on "**Additional Names for this Host**"
          - type **`<host>`**¹ in "**Host name**"
          - type **`ddprefixdomaindd.ddbasedomaindd`** in "**Domain**"
          - type solution name in "**Description**"
          - repeat the previous three step after a click on "**➕ Add**" for every other registered host name
          - ✓ click on "**Save**"
     - click on "**✔️ Apply Changes**"

¹ That is the solution specific host name e.g. **`ddofficedd`**, **`ddturndd`**.

Note: *In case of missing entries in the "**DNS Resolver**", pfSense will report the following.*

    Potential DNS Rebind attack detected, see http://en.wikipedia.org/wiki/DNS_rebinding
    Try accessing the router by IP address instead of by hostname.

The previously used DSL-Router has to be removed as DNS-Server from the configuration. Instead the DNS-Server (PRIDNS/SECDNS) from the PPPoE connection have to be taken.

- ⬆ select menu option "**System** > **Settings** > **General**"
     - focus on "**Networking**"
     - ✏️✓ remove entries for "**DNS server**"
     - ✏️✓ enable option "**DNS server options**" ("Allow DNS server list to be overridden by DHCP/PPP on WAN")
     - ✓ click on "**Save**"

## DSL-Router usage (default scenario)

### Allow DSL-Router LAN traffic

In case of having other systems connected to the DSL-Router, beside of pfSense, it might be needed to allow those other systems to communicate with pfSense and/or other systems on the pfSense LAN.

By default this kind of traffic is blocked by pfSense. See Firewall Rules

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"

The first colums of list "Rules (Drag to Change Order)" looks like the following for the first two lines:

| ☐ |  | States | Protocol | Source | Port |
| :---: | :---: | ---: | :--- | :--- | :---: |
| ☐ | ❌ | 0/0 B | * | RFC 1918 networks | * |
| ☐ | ❌ | 0/0 B | * | Reserved Not assigned by IANA | * |

Note: *These rule got automatically created based on the WAN interface definition. In order to remove these rules, the interface configuration has to be adjusted.*

#### Adjust WAN Interface configuration

- ⬆ select menu option "**Interfaces > WAN**"
     - focus on "**Reserved Networks**"
     - ✏️✓ ~~disable~~ "**Block private networks and loopback addresses**" ("Blocks traffic from IP addresses that are reserved for private networks ...")
     - ✏️✓ ~~disable~~ "**Block bogon networks**" ("Blocks traffic from reserved IP addresses (but not RFC 1918) or not yet assigned by IANA. ...")
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

## DNS restriction

Redirect all unencrypted DNS requests to the OPNsense system, regardless of the chosen DNS server. This will avoid on IPv4 the bypass of the OPNsense Systems DNS.

- ⬆ select menu option "**Firewall > NAT** > **Port Forward**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ select **`LAN`** as "**Interface**"
          - keep selected **`IPv4`** as "**TCP/IP Version**"
          - keep **`TCP`** as "**Protocol**"
          - ✏️✓ select **`any`** as "**Destination**"
          - ✏️✓ type **`127.0.0.1`** as "**Redirect target IP**"
          - ✏️✓ select **`DNS`** as "**Redirect target port**"
          - ✏️✓ type **`Redirect all DNS requests to Firewall`** as "**Description**"
          - ✓🔙 click on "**Save**"
     - ✓ click on "**Apply changes**"

## Dynamic DNS (DDNS)

### Package Installation

- ⬆ select menu option "**System** > **Firmware** > **Packages**"
     - navigate to tab "**Plugins**"
     - tick checkbox "**Show community plugins**"
     - type **`os-ddclient`** in the search field
     - ✏️ click on "**➕**" ("Install") of "**os-ddclient**" package line

In order to update the "**Services**" menu, refresh the browser page once the installation is finished.

### Service Type available

- ⬆ select menu option "**Service** > **Dynamic DNS** > "**Settings**"
     - navigate to tab "**General settings**"
     - ✏️✓ select **`ddclient`** as "**Backend**"
     - ✓ click "**Apply**"
     - navigate to tab "**Accounts**"
     - ⬆ click "**➕**" ("Add")
          - ✏️✓ type **`ddproviderdd`** into "**Description**"
          - ✏️✓ select **`ddproviderdd`** as "**Service**"
          - ✓🔙 click on "**Save**"
     - ✓ click on "**Apply**"

In case your DDNS Provider exists in the listbox "**Service Type**", there is no special instruction needed.

Create two entries, in case listbox includes **`<provider> (v6)`** as well.

### Service Type **`custom`**/**`custom (v6)`**

In case your DDNS Provider is not in the listbox "**Service Type**", the **`Customer*`** entry has to be chosen and provider specific configuration have to be made.

#### Service Provider: ChangeIP Inc.

##### for IPv4

- ⬆ select menu option "**Service** > **Dynamic DNS**"
- ⬆ click "**➕ Add**"
- ✏️✓ select **`Custom`** for "**Service Type**"
- keep selected **`WAN`** for "**Interface to monitor**"
- keep selected **`WAN`** for "**Interface to send update from**"
- keep selected **`Automatic (default)`** for "**Check IP Mode**"
- ✏️✓ enable "**Verbose logging**" ("Enable verbose logging*)
- ✏️✓ enable "**HTTP API DNS Options**" ("Force IPv4 DNS Resolution")
- ✏️✓ type **`https://nic.ChangeIP.com/nic/update?hostname=ddprefixdomaindd.ddbasedomaindd&u=dduserdd&p=dduserpassworddd`** in field "**Update URL**"
- ✏️✓ type **`IPv4: ChangeIP Inc. (ddprefixdomaindd.ddbasedomaindd)`** in field "**Description**"
- ✓🔙 click "**💾 Save**"

##### for IPv6

Requires to have IPv6 enabled on pfSense. See section "[IPv6 Enablement](#ipv6-enablement)"

- ⬆ select menu option "**Service** > **Dynamic DNS**"
- ⬆ click "**➕ Add**"
- ✏️✓ select **`Custom (v6)`** for "**Service Type**"
- keep selected **`WAN`** for "**Interface to monitor**"
- keep selected **`WAN`** for "**Interface to send update from**"
- keep selected **`Automatic (default)`** for "**Check IP Mode**"
- ✏️✓ enable "**Verbose logging**" ("Enable verbose logging*)
- ✏️✓ enable "**HTTP API DNS Options**" ("Force IPv4 DNS Resolution")
- ✏️✓ type **`https://nic.ChangeIP.com/nic/update?hostname=ddprefixdomaindd.ddbasedomaindd&ip6=%IP%&u=dduserdd&p=dduserpassworddd`** in field "**Update URL**"
- ✏️✓ type **`IPv6: ChangeIP Inc. (ddprefixdomaindd.ddbasedomaindd)`** in field "**Description**"
- ✓🔙 click "**💾 Save**"

## Internet Access Control

With more and more devices - black boxes - coming with a WiFi and/or LAN connection, the device owner do want to have controll on the devices communication. Either to prevent automatic updates or calling home.

In case a network connection and/or App is required for the activation of a device ... most likely such requirement is not advertised nor mentioned anywhere ... DO RETURN THE DEVICE and request your money back.

Usually these devices have a known¹ MAC address and use DHCP to aquire an IP-Address.

¹ If the device has a label with the MAC address, that is easy. But even without such label and/or usage of IPv6, the MAC address and/or DUID can be identified by looking at the DHCP leases (**Status > DHCP Leases**" & "**Status > DHCPv6 Leases**"). Looking at the leases and connecting a device should be done with the COMPLETE traffic from the LAN being blocked.

### Using Fixed IP-Address

For this approach we split up our LAN IP-Address in half. The lower part will get always access to the internet, whereas for the upper part any attempt to the internet will be blocked.

With right CIDR prefix, alias for the firewall can be easily created. Like for example and alias "LAN_Allow_Internet" with the network(s) `fc00::/119, 192.168.0.0/25`.

Example IPv4:

With a usual network `192.168.0.0/24`, the IP range covers the IPs `192.168.0.1` to `192.168.0.254`. The lower part of ~126 IPs could get access to the internet. A network `192.168.0.0/25` with `192.168.0.1` to `192.168.0.126` would fit. Any device with IP-Address bigger than `192.168.0.126` gets the internet blocked.

Example IPv6:

With a usual network `fc00::/112`, the IP range covers the IPs `fc00::1` to `fc00::ffff`. The lower part of ~126 IPs could get access to the internet. A network `fc00::/119` with `fc00::1` to `fc00::1ff` would fit. Any device with IP-Address bigger than `fc00::1ff` gets the internet blocked.

#### Alias Allow Internet

- ⬆ select menu option "**Firewall > Aliases**"
     - ⬆ click on "**➕**"
          - keep enabled "**Enabled**"
          - ✏️✓ type **`LAN_Allow_Internet`** into "**Name**"
          - ✏️✓ select **`Network(s)`** as "**Type**"
          - ✏️✓ type **`fc00::/119, 192.168.0.0/25`** into "**Content**"
          - ✏️✓ type **`LAN: lower part allows internet access`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"

We do repeat these steps for every LAN network, the OPNsense system manages.

¹ As soon as one network ends with the prefix press ENTER and the text gets converted into a block with an `❌` at the right.

#### Rule Block Internet

Test current access of the Workstation to the Internet, by open a wellknown URL in the browser. Web Page should be presented in the browser.

- ⬆ select menu option "**Firewall > Rules** > **WAN**"
     - ⬆ click on "**➕**"
          - ✏️✓ select **`Block`** as "**Action**"
          - keep ~~disabled~~ "**Disabled**" ("Disable this rule")
          - keep enabled "**Quick**" ("Apply the action immediately on match.")
          - keep selected **`WAN`** as "**Interface**"
          - ✏️✓ selected **`out`** as "**Direction**"
          - ✏️✓ selected **`IPv4+IPv6`** as "**TCP/IP Version**"
          - keep select **`any`** as "**Protocol**"
          - keep ~~disableed~~ **`Source / Invert`** ("Use this option to invert the sense of the match.")
          - keep select **`any`** as "**Source**"
          - keep ~~disabled~~ **`Destination / Invert`** ("Use this option to invert the sense of the match.")
          - keep selected **`any`** as "**Destination**"
          - keep selected **`any`** as "**from:**" and "**to:**" of "**Destination port range**"
          - ✏️✓ type **`Block Clients with a upper part IP`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - tick checkbox of the newly created line¹
     - click on **`🡐`** ("Move selected rules before this rule") of the first line²
     - click on "**Apply Changes**"

¹ The checkbox with a checkmark in it, marks a line as selected.

Test current access of the Workstation to the Internet, by open a wellknown URL in the browser. The browser should be unable to open the page.

- ⬆ select menu option "**Firewall > Rules** > **WAN**"
     - ⬆ click on "**➕**"
          - ✏️✓ select **`Pass`** as "**Action**"
          - keep ~~disabled~~ "**Disabled**" ("Disable this rule")
          - keep enabled "**Quick**" ("Apply the action immediately on match.")
          - keep selected **`WAN`** as "**Interface**"
          - ✏️✓ selected **`out`** as "**Direction**"
          - ✏️✓ selected **`IPv4+IPv6`** as "**TCP/IP Version**"
          - keep select **`any`** as "**Protocol**"
          - keep ~~disabled~~ **`Source / Invert`** ("Use this option to invert the sense of the match.")
          - ✏️✓ select **`Aliases > LAN_Allow_Internet`** as "**Source**"¹
          - keep ~~disabled~~ **`Destination / Invert`** ("Use this option to invert the sense of the match.")
          - keep selected **`any`** as "**Destination**"
          - keep selected **`any`** as "**from:**" and "**to:**" of "**Destination port range**"
          - ✏️✓ type **`Allow Clients with a lower part IP the access`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - tick checkbox of the newly created line²
     - click on **`🡐`** ("Move selected rules before this rule") of the first line
     - click on "**Apply Changes**"

Test current access of the Workstation to the Internet, by open a wellknown URL in the browser. Web Page should be presented in the browser.

¹ Add all Aliases which should get access to the internet.

² The checkbox with a checkmark in it, marks a line as selected.

#### Add Device without Internet Access

- ⬆ select menu option "**Services** > **Dnsmasq DNS & DHCP**"
     - ⬆ click on "**➕**" ("Add")
          - ✏️✓ type **`<host name>`** into "**Host**"
          - ✏️✓ type **`<domain name>`** into "**Domain**"
          - ✏️✓ type **`<IP address>`** into "**IP address**"
          - ✏️✓ type **`<MAC address>`** into "**Hardware addresses**"
          - ✏️✓ type **`<description>`** into "**Hardware addresses**"
          - ⬆ click on "**Save**"
     - ⬆ click on "**Apply**"

## Internet Disable (temporarily)

Under some circumstances, e.g. configuring the [Internet Access Control](#internet-access-control), it might be required to temporarily turn off the internet at all. For example to identify MAC-Address of a device, without allowing internet access during the time of discovery.

### Turn OFF

- ⬆ select menu option "**Interface > [WAN]**"
     - focus on section "**Basic configuration**"
     - ✏️✓ ~~disable~~ "**Enable**" ("Enable Interface")
     - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

### Turn ON

- ⬆ select menu option "**Interface > [WAN]**"
     - focus on section "**Basic configuration**"
     - ✏️✓ enable "**Enable**" ("Enable Interface")
     - ✓🔙 click on "**Save**"
     - click on "**Apply changes**"

## IP-Phone (SIP-Gateway)

Several different configurations are required.

- define Outbound Rules (SIP-Gateway -> Internet)
- define Inbound Rules (Internet -> SIP-Gateway)
- enable 1:1 mapping of the ports (Hybrid Outbound NAT + Mapping)

### Alias: IP-Address of SIP-Gateway

Define an Alias for IP-Address of the SIP-Gateway. This Alias will be used to allow/block communication.

- ⬆ select menu option "**Firewall** > **Aliases**"
     - ⬆ click on "**➕**"
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Name**"
          - keep selected **`Host(s)`** as "**Type**"
          - ✏️✓ type **`vpsipipv4addressvp`** into "**Content**"
          - ✏️✓ type **`SIP Gateway IPv4`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Properties**"
          - ✏️✓ type **`SIP_Gateway_IPv6`** into "**Name**"
          - focus on "**Host(s)**"
          - ✏️✓ type **`vpsipipv6addressvp`** in "**Content**"
          - ✏️✓ type **`SIP Gateway IPv6`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply**"

### Alias: Ports of SIP-Gateway

Define some Aliases for all ports used by the SIP-Gateway. This Alias will be used to allow/block communication.

The ports in the following instruction are taken from the Telekom AG website and might be different to your ISP or SIP-Provider. See [Telekom: Wie kann ich meinen Festnetz-Anschluss mit Voice over IP bzw. einem SIP-Client nutzen?](https://www.telekom.de/hilfe/internet-telefonie/telefonie/voice-over-ip-sip-client?samChecked=true)

- ⬆ select menu option "**Firewall > Aliases**"
     - ⬆ click on "**➕**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Name**"
          - ✏️✓ select **`Port(s)`** as "**Type**"
          - ✏️✓ type the following ports into "**Content**"
               - **`5070`**
               - **`5080`**
               - **`30000:31000`**
               - **`40000:41000`**
          - ✏️✓ type **`SIP UDP Ports inwards`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**"
          - ✏️✓ type **`SIP_Outbound_TCP`** into "**Name**"
          - ✏️✓ select **`Port(s)`** as "**Type**"
          - ✏️✓ type the following ports into "**Content**"
               - **`80`**
               - **`443`**
               - **`65002`**
          - ✏️✓ type **`SIP TCP Ports outwards`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - ⬆ click on "**➕**"
          - ✏️✓ type **`SIP_Outbound_UDP`** into "**Name**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`5060`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`30000:31000`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`40000:41000`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`3478`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`3479`** into "**Port**"
          - ✏️✓ type **`SIP UDP Ports outwards`** into "**Description**"
          - ✓🔙 click on "**Save**"
     - click on "**Apply**"

### Allow SIP-Gateway Internet communication (SIP-Gateway specific)

Allow TCP/IP communication from the SIP-Gateway to the internet.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**LAN**"
     - ⬆ click on "**⤵ Add**"¹
          - focus on "**Edit Firewall Rules**"
          - keep `Pass` as "**Action**"
          - keep `LAN` as "**Interface**"
          - keep `IPv4` as "**Address Family**"
          - keep `UDP` as "**Protocol**"
          - focus on "**Source**"
          - ✏️✓ select **`Address or Alias`** as "**Source**"
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Source Address**" of "**Source**"
          - focus on "**Destination**"
          - keep `Any` as "**Destination**"
          - keep `(other)` as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Outbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep `(other)` as "**To**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Outbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Copy") at the right of the newly created entry
          - focus on "**Edit Firewall Rules**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - focus on "**Source**"
          - ✏️✓ type **`SIP_Gateway_IPv6`** into "**Source Address**" of "**Source**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ Using the second button, will ensure that the new rule will be added to the end of the list.

### Enable Port Forwarding (WAN->SIP-Gateway)

Activate the port forwording (UDP) to the SIP-Gateway. The IPv6 entries are basically duplicates of the IPv4 entries, with the "4" changed to "6" in fields "**Address Family**"and "**Address**".

- ⬆ select menu option "**Firewall > NAT**"
     - stay to tab "**Port Forward**"
     - ⬆ click on "**⤴ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep `WAN` as "**Interface**"
          - keep `IPv4` as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - keep `WAN address` as "**Destination**"
          - ✏️✓ keep `(Other)` as "**From port**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep `(Other)` as "**To port**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep `Address or Alias` as "**Redirect target IP**" ("Type")
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Address**" of "**Redirect target IP**"
          - keep `Other` as "**Redirect target port**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Redirect Internet UDP traffic to SIP-Gateway.`** into "**Description**"
          - keep `Add associated filter rule`¹ as "**Filter rule association**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ type **`SIP_Gateway_IPv6`** into "**Address**" of "**Redirect target IP**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ This option will automatically generate the Firewall Rule to allow Inbound-Traffic. See below section "[Inbound NAT associated filter rules](#inbound-nat-associated-filter-rules)"

The creation of the NAT - Port Forward rules should have created the corresponding rules to allow internet traffic to passed to the Web Server.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"

### Inbound NAT associated filter rules

In the case the rules were not autmatically created or got deleted at a later point in time, the following shows how to create those rules manually. Or take the instruction to verify the automatically created rules.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep `Pass` as "**Action**"
          - keep `WAN` as "**Interface**"
          - keep `IPv4` as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - focus on "**Source**"
          - keep `Any` as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`Address or Alias`** as "**Destination**"
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Destination Address**" of "**Destination**"
          - keep `(other)` as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - keep `(other)` as "**To**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Allow Internet UDP traffic to SIP-Gateway.`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - focus on "**Destination**"
          - ✏️✓ type **`SIP_Gateway_IPv6`** into "**Address**" of "**Destination**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Enable Hybrid Outbound NAT

- ⬆ select menu option "**Firewall > NAT**"
     - navigate to tab "**Outbound**"
     - focus on "**Outbound NAT Mode**"
     - ✏️✓ select **`Hybrid Outbound NAT rule generation.`** as "**Mode**"
     - ✓ click on "**💾 Save**"
     - ⬆ click on "**⤴ Add**"
          - focus on "**Edit Advanced Outbound NAT Entry**"
          - keep `WAN` as "**Interface**"
          - keep `IPv4+IPv6` as "**Address Family**"
          - ✏️✓ select **`TCP/UDP`** as "**Protocol**"
          - keep `Any` as "**Source**"
          - keep `Any` as "**Destination**"
          - focus on "**Translation**"
          - keep `WAN address` as "**Address**"
          - keep "**Port or Range**" unset
          - ✏️✓ enable "**Static Port**"
          - ✏️✓ type **`Required by STUN/TURN traffic`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

## IPv6 Enablement

### IPv6 for WAN

Specify IPv6 of WAN-Router.

- ⬆ select menu option "**Interfaces** > **[WAN]**"
     - focus on "**General configuration**"
     - ✏️✓ select **`DHCPv6`** as "**IPv6 Configuration Type**"
     - ✓ click on "**Save**"
     - click on "**Apply changes**"

### IPv6 for LAN

Specify IPv6 address for the LAN-Gateway.

- ⬆ select menu option "**Interfaces** > **[LAN]**"
     - focus on "**General Configuration**"
     - ✏️✓ select **`Static IPv6`** as "**IPv6 Configuration Type**"
     - focus on "**Static IPv6 Configuration**"
     - ✏️✓ type **`fwlanipv6addressfw`** into "**IPv6 address**"
     - ✏️✓ select **`fwlanipv6cidrfw`** (address block) for "**IPv6 address**"
     - keep selected **`Disabled`** as "**IPv6 gateway rules**"
     - ✓ click on "**Save**"
     - click on "**Apply changes**"

Specify for LAN DHCPv6 server the IPv6 range.

- ⬆ select menu option "**Services** > **Dnsmasq DNS & DHCP** > **DHCP ranges**"
     - navigate to tab "**DHCP ranges**"
     - ⬆ click on **`➕`**
     - ✏️✓ select **`LAN`** as "**Interface**"
     - ✏️✓ type **`fwlandhcpstartipv6addressfw`** into "**Start address**"
     - ✏️✓ type **`fwlandhcpendipv6addressfw`** into "**End address**"
     - keep either empty or **`64`** for "**Prefix length (ipv6)**"
     - ✏️✓ select **`ra-only`** as "**RA mode**"
     - ✏️✓ type **`fwlandomainfw`** into "**Domain**"
     - ✓ click on "**Save**"
     - click on "**Apply**"

## LAN Bridge for all/some LAN interface

With an installed 2/4 LAN card installed, a Bridge is capable of bundling multiple LAN interfaces to one.

Note: *That is for none WAN interfaces*

### LAN Bridge "Quad-Port-Bridge LAN0-2"

Note: *With currently the LAN3 interface in use, you will - at the end - have to change **physically** the LAN connector on the OPNsense machine!*

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - ✏️✓ select **`fwlan0interfacefw (fwlan0macaddressfw)`** from listbox "**Device**"
     - ✏️✓ type **`LAN0`** into "**Description**"
     - ✏️✓ click on "**Add**"
     - ✏️✓ select **`fwlan1interfacefw (fwlan1macaddressfw)`** from listbox "**Device**"
     - ✏️✓ type **`LAN1`** into "**Description**"
     - ✏️✓ click on "**Add**"
     - ✏️✓ select **`fwlan2macaddressfw (fwlan2interfacefw)`** from listbox "**Device**"
     - ✏️✓ type **`LAN2`** into "**Description**"
     - ✏️✓ click on "**Add**"

- ⬆ select menu option "**Interfaces** > **[LAN0]**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✓ click on "**Save**"
- ⬆ select menu option "**Interfaces** > **[LAN1]**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✓ click on "**Save**"
- ⬆ select menu option "**Interfaces** > **[LAN2]**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✓ click on "**Save**"
     - ✏️ click on "**Apply Changes**"

- ⬆ select menu option "**Interfaces** > **Devices** > **Bridge**"
     - ⬆✏️✓ click on "**➕**"
          - ✏️✓ select **`LAN0`**, **`LAN1`** and **`LAN2`** as "**Member Interfaces**"
          - ✏️✓ type **`Quad-Port-Bridge LAN0-2`** in "**Description**"
          - keep ~~disabled~~ "**Enable link-local address**"
          - ✓🔙 click on "**Save**"
     - ✏️ click on "**Apply**"
- ⬆ select menu option "**Interfaces** > **Assignments**"
     - ✏️✓ select option "bridge0 (Quad-Port-Bridge LAN0-2)" as "**LAN**" Interface
     - ✓🔙 click on "**Save**"

After the change got saved, from "fwlan3interfacefw (fwlan3macaddressfw)" is no longer working, as the interface is no longer assigned.

Go to the pfSense machine and switch the LAN cable from "fwlan3interfacefw" to one of those from the newly created bridge ("fwlan0interfacefw", "fwlan1interfacefw" or "fwlan2interfacefw").

Back on your workstation, refresh the page in the browser.

In case the interface "fwlan3interfacefw" is planned to be used for a DMZ, the interface must not be assigned to the LAN Bridge "Quad-Port-Bridge LAN0-2".

### Add LAN3 to LAN Bridge "Quad-Port-Bridge LAN0-2"

In order to assign "fwlan3interfacefw" to the LAN Bridge, the following steps are required.

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - ✏️✓ select **`fwlan3interfacefw (fwlan3macaddressfw)`** from listbox "**Device**"
     - ✏️✓ type **`LAN3`** into "**Description**"
     - ✏️✓ click on "**Add**"
- ⬆ select menu option "**Interfaces** > **[LAN3]**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✓ click on "**Save**"
     - ✏️ click on "**Apply changes**"
- ⬆ select menu option "**Interfaces** > **Assignments**"
     - navigate to tab "**Bridges**"
     - ⬆ click on "**🖉**" (Pencil icon) to edit interface "**bridge0**"
          - ✏️✓ select **`LAN3`** in addition the already select `LAN0`, `LAN1` and `LAN2` of "**Member Interfaces**"
          - ✏️✓ type **`Quad-Port-Bridge LAN0-3`** in "**Description**"
          - ✓🔙 click on "**Save**"

Go to the OPNsense machine and switch the LAN cable back to igb3. So you have the same physical configuration as during setup.

### Remove LAN3 from LAN Bridge "Quad-Port-Bridge LAN0-3"

In order to remove **`fwlan3interfacefw`** to the LAN Bridge, the following steps are required.

- ⬆ select menu option "**Interfaces** > **Devices** > "**Bridge**""
     - ⬆ click on "**🖉**" ("**Edit**") to edit interface "**bridge0**"
          - ✏️✓ ~~de-select~~ **`LAN3`** leaving only `LAN0`, `LAN1` and `LAN2` selected of "**Member Interfaces**"
          - ✏️✓ type **`Quad-Port-Bridge LAN0-2`** in "**Description**"
          - ✓🔙 click on "**Save**"
     - ✓🔙 click on "**Apply**"
- ⬆ select menu option "**Services** > **Dnsmasq DNS & DHCP** > "**General**""
     - ✏️✓ select **`LAN3`** as "**Interface**" in addition to the **`LAN`**
     - ✓🔙 click on "**Save**"

## Serial Communications

- ⬆ select menu option "**System** > **Settings** > **Administration**"
     - focus on section "**Console**"
     - ✏️✓ select **`Serial Console`** as "**Secondary Console**"
     - keep **`115200`** as "**Serial Speed**"
     - ✏️✓ enable "**USB-based serial**" ("Use USB-based serial ports") ²
     - keep enabled "**Console menu**" ("Password protect the console menu")
     - ✓ click on "**Save**"

¹ We change **`Serial Console`** as "**Primary Console**" and **`VGA Console`** as "**Secondary Console**", in case we are going to use the serial interface primarily to monitor the pfSense system well being. 

² We do want to avoid USB-CDC Interface (serial) and use this only in case of **`Serial Console`** as "**Secondary Console**".

## SSH Setup

In order to use ssh, to connect with the pfSense system, it is highly recommended to use keys instead of the accounts password. See [User - Assign SSH Public Key](#user---assign-ssh-public-key)

- ⬆ select menu option "**System** > **Settings** > **Administration**"
     - focus on section "**Secure Shell**"
     - ✏️✓ enable "**Secure Shell Server**" ("Enable Secure Shell")
     - keep ~~disabled~~ "**Root Login**" ("Permit root user login")
     - keep ~~disabled~~ "**Authentication Method**" ("Permit password login ")¹
     - focus on section "**Authentication**"
     - ✏️✓ select **`Local Database`** for "**Server**"
     - ✏️✓ select **`Ask password`** for "**Sudo**"
     - ✓ click on "**Save**"

¹ Only for testing, this option can be enabled for a short period of time.

### Check Login

From your Workstation you can check is SSH was setup correctly.

```console
fwadminfw@wshostws:~$ ssh fwadminfw@fwhostnamefw.fwhostdomainfw
------------------------------------------------
|       Hello, this is OPNsense 25.7           |           :::::::.
|                                              |           :::::::::.
|  Website:     https://opnsense.org/          |        :::        :::
|  Handbook:    https://docs.opnsense.org/     |        :::        :::
|  Forums:      https://forum.opnsense.org/    |        :::        :::
|  Code:        https://github.com/opnsense    |         `:::::::::
|  Reddit:      https://reddit.com/r/opnsense  |           `:::::::
------------------------------------------------
fwadminfw@fwhostfw:~ $ █
```

- type **`exit`** (for logout)
- press **ENTER**

## UPS

### Package Installation

- ⬆ select menu option "**System** > **Firmware** > **Packages**"
     - navigate to tab "**Plugins**"
     - tick checkbox "**Show community plugins**"
     - type **`os-nut`** in the search field
     - ✏️ click on "**➕**" ("Install") of "**os-nut**" package line

In order to update the "**Services**" menu, refresh the browser page once the installation is finished.

### UPS Setup

- ⬆ select menu option "**Services** > **Nut** > **Configuration**"
     - navigate to tab "**General Settings** > **General Nut Settings**"
     - ✏️✓ enable "**Enable Nut**"
     - keep selected **`standalone`** as "**Service Mode**"
     - ✓ click on "**Apply**"
     - navigate to tab "**General Settings** > **Nut Account Settings**"

     - navigate to tab "**UPS Type** > **USBHID-Driver**"
     - ✏️✓ enable "**Enable**"
     - keep option **`port=auto`** for "** Extra Arguments**"
     - ✓ click on "**Apply**"


     - focus on section "**General Settings**"
     - ✏️✓ select **`Local USB`** as "**UPS Type**"
     - ✏️✓ type fwupsnamefw¹ in field "**UPS Name**" 
     - focus on section "**Driver Settings**"
     - keep `usbhid` as "**Driver**"
     - click on "**⚙ Display Advanced**"
     - focus on section "**Advanced settings**"
     - ✏️✓ type **`LISTEN * 3493`** in field "**Additional configuration lines for upsd.conf**"
     - ✏️✓ type the following three lines in field "**Additional configuration lines for upsd.users**"
          - [fwnutmonitoraccountfw]
          -    password = fwnutmonitorpasswordfw
          -    upsmon secondary
     - ✓ click on "**💾 Save**"

¹ Name used by all other systems, accessing the UPS-Service on pfSense.

Note: *The `LISTEN * 3493` part enables the NUT service to listen on all interfaces. Hence ever system in any network will be allowed to use the NUT service on the pfSense system*

### 
### Test Remote Communication

With the software package `nut-client` installed on another system, that one can make use of NUT on pfSense.

```console
# upsc -l fwhostnamefw
Init SSL without certificate database
fwupsnamefw
# upsc fwupsnamefw@fwhostnamefw
Init SSL without certificate database
battery.charge: 100
:
ups.vendorid: 051d
# █
```

Note: *The data (output) of the last command is identical with the information shown in the pfSense webConfigurator on "**Services / UPS / Status**" section "**UPS Detail**".*

## User/Group Management

### Group - Create

Have a User-Group available, with members been able to use the IPsec VPN connection.

- ⬆ select menu option "**System** > **User Manager**"
     - navigate to tab "**Groups**"
     - ⬆ click on "**➕ Add**"
          - ✏️✓ type **`<group-short-name>`** in "**Group name**"
          - keep selected **`Local`** for **Scope**
          - ✏️✓ type **`<group-description>`** in "**Description**"
          - ✏️✓ add all necessary users to the list "**Members**"
          - ⬆ click on "**➕ Add**"¹
               - select **`<privilege>`** for **Assigned privileges**¹
               - ✓🔙 click on "**💾 Save**"¹
          - ✓🔙 click on "**💾 Save**"

¹ In case the goup should have a specific privilege assigned.

### User - Assign SSH Public Key

#### Prerequisites

- User already exists
- the public key of the workstation exists

In case you have no ssh-key pair yet ... create it using the following command (and press ENTER when asked for passphrase)

As long as we use the `@` in the `ssh fwadminfw@fwhostnamefw.fwhostdomainfw` command, the name of the workstations account is irrelevant. All what counts is the public key `id_rsa.pub` for authentication.

```console
$ ls ~/.ssh
known_hosts  known_hosts.old
```

If files `id_rsa` and `id_rsa.pub` are listed as well, you have already prepared your workstation and therefor have to skip the following commands.

```console
$ ssh-keygen -t rsa -b 4096 -C "wssysuserws wshostws" -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/wshostws/.ssh/id_rsa
Your public key has been saved in /home/wshostws/.ssh/id_rsa.pub
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

#### Make Adjustment

- ⬆ select menu option "**System** > **Access** > **Users**"
     - navigate to tab "**Users**"
     - ⬆ click "**🖉**" ("Edit") of the users line
          - ✏️✓ insert the public key into "**Authorized Keys**"¹
          - ✓🔙 click on "**Save**"

¹ To be taken from the workstation accounts file `/home/wssysuserws/.ssh/id_rsa.pub`.

### User - Assign Client Certificate

Prerequisite:

- User already exists
- Client Certificate has been already created/imported

- ⬆ select menu option "**System** > **User Manager**"
     - navigate to tab "**Users**"
     - ⬆ click "**🖉**" ("Edit user") of the users line
          - focus on section "**User Certificates**"
          - ⬆ click on "**➕ Add**"
               - ✏️✓ select **`Choose an existing certificate`** for **Method**
               - ✏️✓ select users certificate for **Existing Certificates**
               - ✓🔙 click on "**💾 Save**"
          - ✓🔙 click on "**💾 Save**"

## User - Create Own Admin User

- ⬆ navigate to menu option "**System** > **Access** > **Users**"
     - ⬆ click on "**➕**"
          - ✔️✏️ type **`fwadminfw`** in "**Username**"
          - ✔️✏️ type **`fwadminpasswordfw`** in "**Password**"
          - ✔️✏️ type **`Administrator Product Idenpendent`** in "**Full name**"
          - keep **`Default`** as "**Language**"
          - ✔️✏️ select **`/bin/sh`** as "**Login shell**"¹
          - keep "**Expiration date**" empty
          - ✔️✏️ select **`admins`** as "**Group membership**"
          - ✔️✏️ select **`All pages`** as "**Provoleges**"
          - ✔️🔙 click on "**Save**"

¹ This change is required, to avoid `Permission denied (publickey).` error when trying to access the OPNsense system via SSH.

### User - Create Regular User

- ⬆ select menu option "**System** > **User Manager**"
     - navigate to tab "**Users**"
     - ⬆ click on "**➕ Add**"
     - keep ~~disabled~~ "**Disabled**" ("This user cannot login")
     - ✏️✓ type **`<user-id>`**¹ in "**Username**"
     - ✏️✓ type **`<password>`**¹ in "**Password**"
     - ✏️✓ type **`<user-full-name>`**¹ in "**Full name**"
     - keep empty field "**Expiration date**"
     - keep ~~disabled~~ "**Custom Settings**" ("Use individual customized GUI options and dashboard layout for this user.")
     - ✏️✓ add all necessary groups to the list "**Member of**"
     - ⬆ click on "**➕ Add**"²
          - select **`<privilege>`** for **Assigned privileges**²
          - ✓🔙 click on "**💾 Save**"²
     - keep ~~disabled~~ "**Certificate**" ("Click to create a user certificate")
     - keep empty field "**Authorized SSH Keys**"
     - keep empty field "**IPsec Pre-Shared Key**"
     - keep ~~disabled~~ "**Keep Command History**" ("Keep shell command history between login sessions")
     - ✓🔙 click on "**💾 Save**"

¹ To be identical with CN field of the certificate.

² In case the user should have a specific privilege assigned.

### User - Import Client Certificate

Prerequisite: Client Certificate of a user available as a **`.crt`** file.

- ⬆ select menu option "**System** > **Certificates**"
     - navigate to tab "**Certificates**"
     - ⬆ click on "**➕ Add/Sign**"
          - select **`Import an existing Certificate`** for **Method**
          - ✏️✓ type **`<user-id> Authentication`** in "**Description name**"
          - keep selected **`X.509 (PEM)`** for "**Certificate Type**"
          - ✏️✓ copy in "**Certificate data**", from the users **`.crt`** file, the part starting with **`-----BEGIN CERTIFICATE-----`** and ending with **`-----END CERTIFICATE-----`**
          - keep empty field "**Private key data**"
          - ✓🔙 click on "**💾 Save**"

## Web GUI

### Limit Access to a few LAN interfaces

- ⬆ select menu option "**System** > **Settings** > "**Administration**"
     - focus on section "**Web GUI**"
     - ✏️✓ select only "**Listen Interfaces**" necessary 
     - ✓🔙 click on "**Save**"




---



## Server in DMZ

In case you have create a kind of DMZ, by running OPNsense behind the DSL Router and with a server (nchostnc with Nextcloud) connected to the DSL Router ... In order to access this server by name, do the following.

- navigate to menu "**Services** > **Unbound DNS** > **Overrides**"
- select tab "Host Overrides"
- ⬆ click on "**+**", under the top list, to add new entry
    - ✔️✏️ enter **`nchostnc`** in field "**Host**"
    - ✔️✏️ enter **`fwhostdomainfw`** in field "**Domain**"
    - ✔️✏️ select "A (IPv4 address)" as "**Type**"
    - ✔️✏️ enter IP-Address of nchostnc in field "**IP address**"
    - 🔙 click on "**Save**"
- in top list, select newly created entry
- focus on section "**Aliases**"
- ⬆ click on "**+**" (section "**Aliases**"), under bottom list, to add new entry
    - ✔️✏️ select "nchostnc.fwhostdomainfw" as "**Host override**"
    - ✔️✏️ enter "nchostnc" in field "**Host**"
    - 🔙 click on "**Save**"
- ✔️ click on "Apply" at the bottom

# Configuration - Proxy

## Installation

- navigate to menu "**System** > **Firmware** > **Plugins**"
- select tab "Plugins"
- ✏️ click on "**+**", of line "**os-squid**"

## Configure

- specify Memory Cache only with 12GB (16GB installed RAM)
- navigate to menu "**Services** > **Squid Web Proxy** > **Administration**"
- choose tab menu option "**Gerneral Proxy Settings** > **Local Cache Settings**"
- ✔️✏️ specify "12288" in field "**Memory Cache size in Megabytes**" (12GB)
- ✔️✏️ ~~disable~~ "**Enable local cache**"
- ✔️ click on "Apply" at the bottom
- choose tab menu option "**Forward Proxy** > **General Forward Settings**"
- ✔️✏️ select "LAN" from listbox "**Proxy interface**"
- ✔️ click on "Apply" at the bottom
- choose menu option "**Proxy Auto-Config** > **Matches**"
- ✔️✏️ select "LAN" from listbox "**Proxy interface**"
- ✔️ click on "Apply" at the bottom
