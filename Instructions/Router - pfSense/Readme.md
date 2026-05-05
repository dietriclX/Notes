FreeBSD - pfSense (Version 1.0 from [dietriclX](https://github.com/dietriclX) on GitHub)

# Used Parameters/Values

Assuming the user name is `janedoe`, this document can be adjusted - replacing the placeholders by their values - using the following command.

```console
./apply.sh "/home/janedoe/Downloads/Notes-main/Instructions/Router - pfSense/Readme.md" "/home/janedoe/Documents/pfSense - Readme.md"
Source File: /home/janedoe/Downloads/Notes-main/Instructions/Router - pfSense/Readme.md
Target File: /home/janedoe/Documents/pfSense - Readme.md
```

The shell script directory `fwsense` is taken from [GitHub: dietriclX / Automation / Router - pfSense](https://github.com/dietriclX/Automaton/tree/main/Router%20-%20pfSense).

## General

- The Machine (becomes the OPNsense system)
     - Netgate Device
          - ID: fwpfsensedeviceidfw
          - Serial Number: fwpfsenseserialfw
     - Administrator
          - Password of the `fwpfsenseadminaccountfw` user: `fwfwsenseadminpasswordfw` (default is `fwpfsenseadminpasswordfw`)
     - Hostname of the pfSense system: `fwhostnamefw`
     - Domain name served by the pfSense system: `fwhostdomainfw`
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
               - LAN IPv4 Address of the pfSense system: `fwlanipv4addressfw`
               - LAN IPv4 Address range: `fwlandhcpstartipv4addressfw` - `fwlandhcpendipv4addressfw`
          - WAN IPv4 Address of the pfSense system: `fwwanip4addressfw`
- The Workstation (used for administration of the pfSense system)
     - Host name: `wshostws`
     - Software
  	     - OS: Debian 12 "bookworm"1G/
     - OS Users
          - Regular User
               - Account: `wssysuserws`
 
## Optional Parameters/Values

- IPv6-Addresses
     - LAN Gateway
          - LAN IPv6 Address of the pfSense system: `fwlanipv6addressfw`
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
     - Name of UPS, as used within pfSense: `fwupsnamefw`
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

Download image "**AMD64 Memstick USB (..., All Other Intel/AMD 64-bit)**" from "[netgate: > NETGATE INSTALLER](https://shop.netgate.com/products/netgate-installer)".

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
  - USB interface - USB-Stick with pfSense install image.

Note #1: *The "LAN interface fwwaninterfacefw" is the onboard LAN interface. The "LAN interface fwlan0interfacefw" to "LAN interface fwlan3interfacefw" are those from the LAN PCI Express card. Use the "LAN interface fwlan3interfacefw" to avoid mixing up the order of interfaces, once the "LAN Bridge" got setup.*

Note #2: *For the initial installation the "**LAN Cable #2**" needs not to be connected. Establish the connection to the workstation, once you start with section "**[Configuration - 2. Part: Console](#configuration---2-part-console)**".*

Note #3: *Once the system is good enough configured to be accessible via the browser, Monitor & Keyboard are no longer required, but the Workstation needs to be connected to the LAN interface.*

## The goal

For the internal SDD card, we will be using the standard File System ZFS.

The machine has one on-board LAN interface (fwwaninterfacefw) and a quad port LAN card (fwlan0interfacefw-fwlan3interfacefw). The interfaces will be defined as follows.

- WAN  -> fwwaninterfacefw; DHCP Client
- LAN  -> fwlan3interfacefw; Static IP Address `fwlanipv4addressfw` with DHCP Server service IP-Range `fwlandhcpstartipv4addressfw` to `fwlandhcpendipv4addressfw`

Note: *Neither interface assignment nor the IP Address information was taken over from the setup (CUI-Wizard). Using the Console, We have to correct the assignment and IP Address.*

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

pfSense Live image boots ...

## Configuration - 1. Part: CUI-Wizard

New dialog `Copyright and Distribution Notice` appears.

```code
Copyright and Trademark Notices.

Copyright(c) 2004-2016. Electric Sheep Fencing, LCC ("ESF").
All Rights Reserved.

Copyright(c) 2014-2024. Rubicon Communications, LLC d/b/a Netgate ("Netgate").
All Rights Reserved.

All logos, text, and contentn of ...
:
```

- ✏️✓ select **`Accept`**, if you agree to the terms
- ✓ press **ENTER**

New dialog `Welcome` appears.

``` code
Welcome to pfSense!

Select configuration file: default (blank) configuration.

Install                Install pfSense
Rescue Shell           Launch a shell for rescue operations
```

- ✏️✓ select **`Install`**
- ✓ press **ENTER**

New dialog `Network Installation` appears.

```code
Setting up network to continue the installation.
```

- ✓ press **ENTER**

### WAN Interface

New dialog `WAN Interface Assignement and Configuration` appears.

```code
Please select the WAN interface.

fwlan0interfacefw fwlan0macaddressfw (no carrier)
fwlan1interfacefw fwlan1macaddressfw (no carrier)
fwlan2interfacefw fwlan2macaddressfw (no carrier)
fwlan3interfacefw fwlan3macaddressfw (active)
fwwaninterfacefw fwwanmacaddressfw (active)
```

- ✏️✓ select **`fwwaninterfacefw`**
- ✓ press **ENTER**

New dialog `WAN (fwwaninterfacefw) Network Mode Setup` appears.

```code
Adjust the network operation mode for the WAN (fwwaninterfacefw) interface if necessary.

>>> Continue         Proceed with the installation
M Interface Mode     DHCP(client)
V VLAN Settings      VLAN Tagging disabled
U Use local resolver false
```

- keep `DHCP (client)` as "**M Interface Mode**"
- keep `VLAN Tagging disabled` as "**V VLAN Settings**"
- keep `false` as "**Use local resolver**"
- ✏️✓ select **`Continue`**
- ✓ press **ENTER**

### LAN Interface

New dialog `LAN Interface Assignment and Configuration` appears.

```code
Please select the LAN interface.

None Do not assign the LAN interface
fwlan0interfacefw fwlan0macaddressfw (no carrier)
fwlan1interfacefw fwlan1macaddressfw (no carrier)
fwlan2interfacefw fwlan2macaddressfw (no carrier)
fwlan3interfacefw fwlan3macaddressfw (active)
```

- ✏️✓ select **`None Do not assign the LAN interface`**
- ✓ press **ENTER**

### LAN/WAN Interface Confirmation

New dialog `Interface Assignment and Configuration` appears.

```code
Please confirm the interface assignment to continue with the installation.

LAN  (not assigned)
WAN fwwaninterfacefw (active)
```

- ✏️✓ select **`Continue`**
- ✓ press **ENTER**

New dialog `Connectivity Check` appears.

```code
Verify the Internet connection...

Try to reach the Netgate Servers, please wait (this can take a while)...
```

### Software License Selection

New dialog `Active Subscription Validation` appears.

```code
This device does not have an active pfSense Plus subscription.

To purchase pfSense Plus:
 Software Store: https://www.netgate.com/purchase-plus
 NDI: 0123 4567 89ab cdef 0123
and select 'Retry Validation' once purchase is complete.

Or select 'Install CE' to install pfSense CE software.
```
- ✏️✓ select **`Install CE`**
- ✓ press **ENTER**

### Disk

New dialog `Installation Options` appears.

```code
Please select the File System type and the Partition Scheme.

>>> Continue       Proceed with the installation
F File System      ZFS (recommended default)
P Partition Schene GPT (compatible with MBR)
```

- ✏️✓ select **`Continue`**
- ✓ press **ENTER**

New dialog `ZFS Virtual Device Type Configuration` appears.

```code
Select the ZFS Virtual Device configuration type.

stripe Stripe - No Redundancy
```

- ✏️✓ select **`stripe`**
- ✓ press **ENTER**

New dialog `Disk Selection` appears.

```code
Select the disk(s) for software installation.

[X] ada0 7.4G <Innodisk DEMSR - 08GB mSATA 3ME A>
```

- ✓ press **ENTER**

New dialog `Confirmation` appears.

```code
Last Chance! Are you sure you want to destroy the current contents of the following disks:

     ada0
```

- ✏️✓ select **`Yes`**
- ✓ press **ENTER**

New Dialog `Partitioning` appears.

```code
commiting the changes to disk, please ...
```

### Software Version Selection

New dialog `Software Version to Install` appears.

```code
Select the version of pfSense CE to install.

0000-2_8_1-preview Beta version 2.8.1
0001-2_8_0         Current Stable Version (2.8.0)
0002-2.7.2         Previous Stable Release (2.7.2)
```

- ✏️✓ select **`0001-2_8_0`**
- ✓ press **ENTER**

### Software Installation Process

New dialog `Installation Details` appears, showing progress of installation.

Wait while software gets downloaded and installed. Once the installation is finished the last lines of the dialog look like the following.

```code
:

pfSense Post Installation setup

pfSense Post Installation setup .. done
```

- ✓ press **ENTER**

### 1st Reboot

New dialog "Complete" appears.

```code
Installation of pfSense complete! Would you like to reboot into the installed system now?
```

- ✏️✓ select **`Reboot`**
- ✓ press **ENTER**

Enter the BIOS Setting and disable the boot from USB & removable media.

## Configuration - 2. Part: Console

Start the configuration by ...

- ✏️ connect workstation with pfSense system, using "**LAN Cable #2**" 
- ✏️ start pfSense system

After the first boot, you will be greeted with the following information and available menu options.

```code
FreeBSD/amd64 (pfSense.home.arpa) (ttyv0)

pfSense - Serial: fwserialfw - Netgate Device ID: fwdeviceidfw

*** Welcome to pfSense 2.8.0-RELEASE (amd64) on pfSense ***

 WAN (wan) -> fwwaninterfacefw -> v4/DHCP4: 192.168.22.104/24
                                  v6/DHCP6: fd00::192:168:2200:108/128

0) Logout / Disconnect SSH            9) pfTop
1) Assign Interfaces                 10) Filter Logs
2) Set interface(s) IP address       11) Restart GUI
3) Reset admin account and password  12) PHP shell + pfSense tools
4) Reset to factory defaults         13) Update from console
5) Reboot system                     14) Enable Secure Shell (sshd)
6) Halt system                       15) Restore recent configuration
7) Ping host                         16) Restart PHP-FPM
8) Shell

Enter an option:
```

The IP Addresses (or missing of the same) are an indicator, that none of the configuration has been taken over.

### LAN/WAN Interface

- type **`1`** for menu option "**1) Assign Interfaces**"
- ⬆ press **ENTER**

The entry details of option "Assign Interfaces" are shown.

```code
Valid interfaces are:

fwlan0interfacefw    fwlan0macaddressfw (down) ...
fwlan1interfacefw    fwlan1macaddressfw (down) ...
fwlan2interfacefw    fwlan2macaddressfw (down) ...
fwlan3interfacefw    fwlan3macaddressfw (down) ...
fwwaninterfacefw     fwwanmacaddressfw (down) ...

Do VLANs need to be setup first?
If VLABs will not be used, or only for optional interfaces, it is typical to say no here and use the webConfigurator to configure VLANs later, if required.

Should VLANs be set up now [y|n]? 
```

- ✏️✓ type **`n`**
- press **ENTER**

```code
If the names of the interfaces are not known, auto-detection can be used instead. To use auto-detection, please disconnect all interfaces before pressing ‚a‘ to begin the process.

Enter the WAN interface name or ‚a‘ for auto-detection
(fwlan0interfacefw fwlan1interfacefw fwlan2interfacefw fwlan3interfacefw fwwaninterfacefw or a): 
```

- ✏️✓ type **`fwwaninterfacefw`**
- press **ENTER**

```code
Enter the LAN interface name or ‚a‘ for auto-detection
NOTE: this enables full Firewalling/NAT mode.
(fwlan0interfacefw fwlan1interfacefw fwlan2interfacefw fwlan3interfacefw a or nothing if finished): 
```

- ✏️✓ type **`fwlan3interfacefw`**
- press **ENTER**

```code
Enter the Optional 1 interface name or ‚a‘ for auto-detection
(fwlan0interfacefw fwlan1interfacefw fwlan2interfacefw a or nothing if finished): 
```

- press **ENTER**

```code
The interfaces will be assigned as follows:

WAN  -> fwwaninterfacefw
LAN  -> fwlan3interfacefw

Do you want to proceed [y|n]? 
```

- ✏️✓ type **`y`**
- ✓🔙 press **ENTER**

Leaving the "**Assign Interfaces**" and pfSense taking the configuration to update the WAN and LAN interfaces.

```code
Writing configuration…done.
One moment while the settings are reloading... done!

:

*** Welcome to pfSense 2.8.0-RELEASE (amd64) on pfSense ***

 WAN (wan) -> fwwaninterfacefw  -> v4/DHCP4: 192.168.22.104/24
                                   v6/DHCP6: fd00::192:168:2200:108/128
 LAN (lan) -> fwlan3interfacefw -> 

:

Enter an option:
```

### LAN Address(es)

- type **`2`** for menu option "**2) Set interface(s) IP address**"
- ⬆ press **ENTER**

The entry details of option "Set interface(s) IP address" are shown.

```code
Available interfaces:

1 - WAN (re0 - dhcp, dhcp6)
2 - LAN (igb3 - static)

Enter the number of the interface you wish to configure: 
```

- type **`2`** to configure the LAN interface
- ⬆ press **ENTER**

#### LAN IPv4

```code
Configure IPv4 address LAN interface via DHCP? (y/n)
```

- type **`n`**
- press **ENTER**

```code
Enter the new LAN IPv4 address.  Press <ENTER> for none:
> 
```

- ✏️✓ type **`fwlanipv4addressfw`**
- press **ENTER**

```code
Subnet masks are entered as bit counts (as in CIDR notation) in pfSense.
e.g. 255.255.255.0 = 24
     255.255.0.0   = 16
     2555.0.0.0    = 8

Enter the new LAN IPv4 subnet bit count (1 to 32):
> 
```

- ✏️✓ type **`24`**
- press **ENTER**

```code
For a WAN, enter the new LAN IPv4 upstream gateway address.
For a LAN, press <ENTER> for none:
> 
```

- press **ENTER**

#### LAN IPv6

If no IPv6 on the LAN is wanted, do not enter anything at ".... LAN IPv6 address ...**". IPv6 can be enabled at a later point in time. See section "[IPv6 Enablement](#ipv6-enablement)"

```code
Configure IPv6 address LAN interface via DHCP6? (y/n)
```

- type **`n`**
- press **ENTER**

```code
Enter the new LAN IPv6 address.  Press <ENTER> for none:
>
```

Either enter an IPv6 Address or leave it empty.

- ✏️✓ type **`fwlanipv6addressfw`**
- press **ENTER**

If left empty, continue with section "[LAN DHCP (IPv4)](#lan-dhcp-ipv4)"

```code
Subnet masks are entered as bit counts (as in CIDR notation) in pfSense.
e.g. ffff:ffff:ffff:ffff:ffff:ffff:ffff:ff00 = 120
     ffff:ffff:ffff:ffff:ffff:ffff:ffff:0    = 112
     ffff:ffff:ffff:ffff:ffff:ffff:0:0       =  96
     ffff:ffff:ffff:ffff:ffff:0:0:0          =  80
     ffff:ffff:ffff:ffff:0:0:0:0             =  64

Enter the new LAN IPv4 subnet bit count (1 to 128):
> 
```

- ✏️✓ type **`fwlanipv6cidrfw`**
- press **ENTER**

```code
For a WAN, enter the new LAN IPv6 upstream gateway address.
For a LAN, press <ENTER> for none:
> 
```

- press **ENTER**

### LAN DHCP (IPv4)

```code
Do you want to enable the DHCP server on LAN? (y/n) 
```

- ✏️✓ type **`y`**
- press **ENTER**

```code
Enter the start address of the IPv4 client address range: 
```

- ✏️✓ type **`fwlandhcpstartipv4addressfw`**
- press **ENTER**

```code
Enter the end address of the IPv4 client address range: 
```

- ✏️✓ type **`fwlandhcpendipv4addressfw`**
- press **ENTER**


### LAN DHCP6 (IPv6)

Only if IPv6 for LAN got enabled.

```code
Do you want to enable the DHCP6 server on LAN? (y/n) 
```

- ✏️✓ type **`y`**
- press **ENTER**

```code
Enter the start address of the IPv6 client address range: 
```

- ✏️✓ type **`fwlandhcpstartipv6addressfw`**
- press **ENTER**

```code
Enter the end address of the IPv6 client address range: 
```

- ✏️✓ type **`fwlandhcpendipv6addressfw`**
- press **ENTER**

### webConfigurator procotol

```code
Do you wanrt to revert to HTTP as the webConfigurator protocol? (y/n) 
```

- ✏️✓ type **`n`**
- ✓ press **ENTER**

```code
Please wait while the changes are saved to LAN...
 Reloading filter...
 Reloading routing configuration...
 DHCPD...

 The IPv4 LAN address has been set to fwlanipv4addressfw/24


 The IPv6 LAN address has been set to fwlanipv6addressfw/fwlanipv6cidrfw
You can now access the webConfigurator by opening the following URL in your web browser:
                https://fwlanipv4addressfw/
                https://[fwlanipv6addressfw]/

 Press <ENTER> to continue.
```

- press **ENTER**

```code
:

*** Welcome to pfSense 2.8.0-RELEASE (amd64) on pfSense ***

 WAN (wan) -> fwwaninterfacefw  -> v4/DHCP4: 192.168.22.104/24
                                   v6/DHCP6: fd00::192:168:2200:108/128
 LAN (lan) -> fwlan3interfacefw -> v4: fwlanipv4addressfw/24
                             v6/DHCP6: fwlanipv6addressfw/fwlanipv6cidrfw

:

Enter an option:
```

- type **`6`** to shutdown the system
- ⬆ press **ENTER**

```code
pfSense will shutdown and halt system. This may take a few minutes, depending on your hardware.
Do you want to proceed [y|n]? 
```

- type **`y`** to confirm the shutdown
- ✓🔙 press **ENTER**

System shutdown and powered off.

- ✏️ disconnect keyboard to test that even without a keyboard the system boot without any trouble
- start system once more to check the interface assignment

```code
:

*** Welcome to pfSense 2.8.0-RELEASE (amd64) on pfSense ***

 WAN (wan) -> fwwaninterfacefw   -> v4/DHCP4: <IP Address>/24
                       v6/DHCP6: <IP Address>/112
 LAN (lan) -> fwlan3interfacefw  -> v4: fwlanipv4addressfw/24
                       v6/DHCP6: fwlanipv6addressfw/fwlanipv6cidrfw

:

Enter an option:
```

It is correct, so you can turn off the system.

- briefly press the "power-on“ button

System shutdown and powered off.

## Configuration - 3. Part: webConfigurator

- start the pfSense machine

- open URL: https://fwlanipv4addressfw
- login with `fwpfsenseadminaccountfw`/`fwpfsenseadminpasswordfw`

It is the first time, therfore the wizard shows up

New dialog "**Wizard** > **pfSense Setup**" appears.

> Welcome to pfSense® software!

> This wizard will provide guidance through the initial configuration of pfSense.

> The wizard may be stopped at any time by clicking the logo image at the top of the screen.

> pfSense® software is developed and maintained by Netgate®

- click on "**Next**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**Netgate® Global Support is available 24/7**" appears.

> Our 24/7 worldwide team of support engineers are the most qualified to diagnose your issue and resolve it quickly, from branch office to enterprise — on premises to cloud.

> We offer several support subscription plans tailored to fit different environment sizes and requirements. Many companies around the world choose Netgate support because:

>     - Support is available 24 hours a day, seven days a week, including holidays.
>     - Support engineers are located around the world, ensuring that no support call is missed.
>     - Our support engineers hold many prestigious network engineer certificates and have years of hands-on experience with networking.

- click on "**Next**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**General Information**" appears.

- ✏️✓ type **`fwhostnamefw`** as "**Hostname**"
- ✏️✓ type **`fwhostdomainfw`** as "**Domain**"
- keep `Overide DNS` enabled
- ✓ click on "**Next**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**Time Server Information**" appears.

- ✏️✓ type **`fwtimeserverfw`** as "**Time server hostname**"
- ✏️✓ select your timezone from listbox "**Timezone**"
- ✓ click on "**Next**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**Configure WAN Interface**" appears.

- click on "**Next**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**Configure LAN Interface**" appears.

- click on "**Next**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**Set Admin WebGUI Password**" appears

- ✏️✓ type the new **`fwfwsenseadminpasswordfw`** for the webConfigurator
- ✓ click on "**Next**"

no longer shown the message "WARNING: The 'fwpfsenseadminaccountfw' account password is set to the default value. Change the password in the User Manager."

New dialog "**Wizard**" > "**pfSense Setup**" > "**Realod configuration**" appears.

- ✓ click on "**Reload**"

New dialog "**Wizard**" > "**pfSense Setup**" > "**Wizard completed.**" appears.

- click on "**Check for update**"

New dialog "**System**" > "**Update**" > "**System update**" appears.

Note: *"**Status**" should say "Up to date".

# Basic Configuration - 4. Part: webConfigurator

## Create own admin user

- ⬆ navigate to menu option "**System** > **User Manager**"
     - navigate to tab "**Users**"
     - ⬆ click on "**➕ Add**"
          - ✔️✏️ type **`fwadminfw`** in "**Username**"
          - ✔️✏️ type **`fwadminpasswordfw`** in "**Password**"
          - ✔️✏️ type **`Administrator Product Idenpendent`** in "**Full name**"
          - keep "**Expiration date**" empty
          - ✔️✏️ select **`admins`** from list "Not member of" from "**Group membership**"
          - ⬆ click on "**≫ Move to 'Member of' list**"
          - ✔️🔙 click on "**💾 Save**"

## Enable RAM Disk

Such a change requires a reboot of the whole system.

Note: *RAM Disk is going to use 5120 MB of the installed 8192 MB RAM.*

- ⬆ select menu option "**System** > **Advanced**"
     - navigate to tab "**Miscellaneous**"
     - focus on section "**RAM Disk Settings (Reboot to Apply Changes)**"
     - ✏️✓ enable option "**Use RAM Disks**"
     - ✏️✓ type **`fwramdisksizetmpfw`** for "**/tmp RAM Disk**" at "**RAM Disk Size**"
     - ✏️✓ type **`fwramdisksizevarfw`** for "**/var RAM Disk**" at "**RAM Disk Size**"
     - ✓ click on "**💾 Save**"

```code
The "Use RAM Disks" setting has been changed.
This requires the firewall to reboot.

Reboot now?
```

- ⬆ click on "**OK**"
- got navigates to menu option "**Diagnostics > Reboot**"
- keep `Normal Reboot` as "**Reboot Method**"
- ⬆ click on "**🔧 Submit**" (wrench)

## Network Time Protocol (NTP) Server

- ⬆ select menu option "**Services** > **NTP**"
	- focus on section "**NTP Server Configuration**"
	- ✏️✓ type **`ptbtime1.ptb.de`** in "**Time servers**" at first line
	- click on "**💾 Save**"

## DNS Server of Router

- ⬆ select menu option "**Services** > **DNS Resolver**"
     - ✏️✓ enable "**DNS Query Forwarding**" ("Enable Forwarding Mode")
     - ✓ click on "**Save**"
     - click on "**✔️ Apply Changes**"

Only if a static IP address has been assigned to the WAN interface, this configuration is required.

- ⬆ select menu option "**System** > **General Setup**"
     - focus on section "**DNS Server Settings**"
     - ✏️✓ type **`rtlanipv4addressrt`** at "**DNS Servers**"
     - ✓ click on "**Save**"

## Switch DHCP product

The product "ISC DHCP" is  deprecated, thus makes it useful to switch to the alternative.

- ⬆ choose menu option  "**System** > **Advanced**"
     - navigate to tab "**Networking**"
     - focus on section "**DHCP Options**"
     - ✏️✓ select  option "**Kea DHCP**" as "**Server Backend**"
     - ✓ click on "**Save**"

## Specify LAN Domain

This section is relevant, in case the "**LAN**" domain "**fwlandomainfw**" is different to the routers domain "**fwhostdomainfw**".

- ⬆ select menu option "**Services > DHCP Server**"
     - navigate to tab "**LAN**"
     - focus on "**Other DHCP Options**"
     - ✏️✓ type **`fwlandomainfw`** into "**Domain Name**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

## Enable Hardware Crypto

First check the status of the Hardware crypto and the CPUs supported Crypto features.

### Check available support by CPU

#### Look at "**CPU Type**"

- ⬆ select menu option "**Status** > **Dashboard**"
     - focus on section "**System Information**"
     - at  "**CPU Type**" look for an "**...Crypto...**" entry. At the end of that line it says either "**(inactive)**" or "**(active)**".

Example information at "**CPU Type**"

```code
AMD GX-415GA SOC with Radeon(tm) HD Graphics
4 CPUs: 1 package(s) x 4 core(s)
AES-NI CPU Crypto: Yes (inactive)
QAT Crypto: No
```

#### Look at "**Hardware crypto**"

- ⬆ select menu option "**Status** > **Dashboard**"
     - focus on section "**System Information**"
     - at "**Hardware crypto**" it says "**Inactive**" or lists a number of ciphers.

### Enable Hardware crypto

- ⬆ select menu option "**System** > **Advanced**"
     - navigate to tab "**Miscellaneous**"
     - focus on section "**Cryptographic & Thermal Hardware**"
     - ✏️✓ select **`AES-NI CPU-based Acceleration`** for "**Cryptographic Hardware**"
     - ✓ click on "**Save**"

## Enable System Patching

- ⬆ select menu option "**System** > **Package Manager**"
     - navigate to tab "**Available Packages**"
     - type **`System_Patches`** in field "**Search term**"
     - click on "**🔍 Search**"
     - ✏️✓ click on "**➕ Install**" of package line "**System_Patches**"
     - ✓ click "**✓ Confirm**" in tab "**Package Installer**"

## Adjust Dashboard

Get to the dashboard and adjust the tiles.

- ⬆ click on the "pfsense" logo int the left top corner
     - ✏️ click on the "**ⓧ**" of tile "**Netgate Services And Support**"
     - click on the "**➕**" above the tiles
     - ✏️  click on "**➕ Service Status**"

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

- ⬆ select menu option "**System** > **Certificates**"
     - navigate to tab "**Authorities**"
     - ⬆ click "**➕ Add**"
          - ✏️✓ type **`cacommonnameca`** (the CN part of the certificate) as "**Descriptive name**"
          - ✏️✓ select **`Import an existing Certificate Authority`** as "**Method**"
          - ✏️✓ enable "**Trust Store**"
          - open the `cacrtfileca` file and copy the content into the clipboard
          - ✏️✓ paste the content of the clipboard into "**Certificate data**"
          - ✓🔙 click "**💾 Save**"

### Configure Intermediate Certificate Authority

When you have setup already ACNO, only a few more steps are required to create those files required.

See [Add-On > Certificates > Intermediate Certificate Authority](../Debian%2012%20-%20ACNO/Setup/Readme.md#intermediate-certificate-authority)

| Input File | From the ACNO instruction |
| :--- | :--- |
| `cicrtfileci` | `~/SSL.canameca/CA.intermediate/SSL.canameca.intermediate.crt` |
| `cikeyfileci` | `~/SSL.canameca/CA.intermediate/SSL.canameca.intermediate.unencrypted.key` |

- ⬆ select menu option "**System** > **Certificates**"
     - navigate to tab "**Authorities**"
     - ⬆ click "**➕ Add**"
          - ✏️✓ type **`cicommonnameci YYMMDD`** as "**Descriptive name**"¹
          - ✏️✓ select **`Import an existing Certificate Authority`** as "**Method**"
          - ✏️✓ enable "**Trust Store**"
          - ✏️✓ enable "**Randomize Serial**"
          - open the `cicrtfileci` file and copy the content into the clipboard
          - ✏️✓ paste the content of the clipboard into "**Certificate data**"
          - open the `cikeyfileci` file and copy the content into the clipboard
          - ✏️✓ paste the content of the clipboard into "**Certificate Private Key (optional)**"
          - ✓🔙 click "**💾 Save**"

¹ **`YYMMDD`** representing the "Valid From" date.

### Configure Web Certificate

- ⬆ select menu option "**System** > **Certificates**"
     - navigate to tab "**Certificates**"
     - ⬆ click "**➕ Add/Sign**"
          - focus on section "**Add/Sign a New Certificate**"
          - ✏️✓ select **`Create an internal Certificate`** as "**Method**"
          - ✏️✓ type "**webConfigurator fwhostnamefw (fwhostdomainfw) YYMMDD**" into "**Descriptive name**"¹
          - focus on section "**Internal Certificate**"
          - ✏️✓ select **`cicommonnameci`** as "**Certificate authority**"
          - ✏️✓ select **`RSA`** as "**Key type**"
          - ✏️✓ select **`4096`** as "**Key length**"
          - ✏️✓ type **`398`** into "**Lifetime (days)**"
          - ✏️✓ type **`fwhostnamefw.fwhostdomainfw`** into "**Common Name**"
          - focus on section "**Certificate Attributes**"
          - ✏️✓ select **`Server Certificate`** as "**Certificate Type**"
          - ✏️✓ select **`FQDN or Hostname`** as "**Alternative Names > Type**" (1. row)
          - ✏️✓ type **`fwhostnamefw.fwhostdomainfw`** into "**Alternative Names > Value**" (1. row)
          - ✏️✓ click "**➕ Add SAN Row**"
          - ✏️✓ select **`FQDN or Hostname`** as "**Alternative Names > Type**" (2. row)
          - ✏️✓ type **`fwhostnamefw`** into "**Alternative Names > Value**" (2. row)
          - ✓🔙 click "**💾 Save**"

¹ **`YYMMDD`** representing the "Valid From" date.

### Activate Own Web Certificate

- ⬆ select menu option "**System** > **Advanced**"
     - navigate to tab "**Admin Access**"
     - focus on section "**webConfigurator**"
     - ✏️✓ select newly created Certificate as "**SSL/TLS Certificate**"
     - ✓ click on "**💾 Save**"

### Test Own Web Certificate

- close browser tab
- in new browser tab, open URL: https://fwhostnamefw.fwhostdomainfw
=> Page got loaded with the new Certificate.
- change URL to: https://fwhostnamefw
=> Page got loaded with the new Certificate.

## DHCP Client - Fix IP-Address

In some cases it is desired to have a system within the DMZ/LAN network with a fixed IP-Address. Usually we would specify the IP-Address on the device itself. As an alternative, the pfSense own DHCP/DHCPv6 Server can be used to achieve the same.

Beside the IP-Address(es), another prerequisite is to have the MAC address and/or DHCP Unique Identifier (DUID) at hand.

- DHCP Server: the MAC address¹ of the client device
- DHCPv6 Server: DHCP Unique Identifier (DUID)¹ of the client device
- DHCP/DHCPc6 Server: the client device IP-Address(es)

¹ Especially DUID has to be taken from the lease page of pfSense system; See menu option "**Status > DHCP Leases**" and "**Status > DHCPv6 Leases**" respectively.

Once the static mapping is completed, the client device has to be rebooted to get the assigned IP-Address.

Note: *In case of an Wifi Interface, used by the client device, not randmon MAC address should be used!*

### DHCP Static Mappings

- ⬆ select menu option "**Services > DHCP Server**"
     - navigate to tab "**DMZ**" or "**LAN**"
     - focus on "**DHCP Static Mappings**"
     - ⬆ click on "**➕ Add Static Mapping**"
          - ✏️✓ type MAC address (Client Device) as "**MAC Address**"
          - ✏️✓ type IPv4-Address (Client Device) as "**IP-Address**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**Apply Changes**"

### DHCPv6 Static Mappings

- ⬆ select menu option "**Services > DHCPv6 Server**"
     - navigate to tab "**DMZ**" or "**LAN**"
     - focus on "**DHCPv6 Static Mappings**"
     - ⬆ click on "**➕ Add Static Mapping**"
          - ✏️✓ type DUID-Address (Client Device) as "**DHCP Unique Identifier**"
          - ✏️✓ type IPv6-Address (Client Device) as "**IPv6-Address**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**Apply Changes**"

## DMZ creation

Makes only sense with a DSL-modem setup. See section "[DSL-Modem usage (as replacement for DSL-Router)](#dsl-modem-usage-as-replacement-for-dsl-router)"

Requires to have IPv6 enabled on pfSense. See section "[IPv6 Enablement](#ipv6-enablement)"

### Define Interface

Take **`pfdmzinterfacepf`** for the DMZ interface.  If necassary, take the interface out of the Bridge configuration. See instruction [Remove LAN3 from LAN Bridge ...](#remove-lan3-from-lan-bridge-quad-port-bridge-lan0-3).

In case pfdmzinterfacepf is not assigned yet, do the following steps.

- ⬆ select menu option "**Interfaces > Assignments**"
     - ✏️✓ select **`pfdmzinterfacepf (pfdmzmacaddresspf)`** from listbox "**Available network ports**"
     - ✏️✓ click on "**➕ Add**"
     - ✓ click on "**💾 Save**"

Make the required changed to the configuration of the assigned interface.

- ⬆ select menu option "**Interfaces > Assignments**"
     - ⬆ click on the name (hyperlink) of interface pfdmzinterfacepf
          - focus on "**General Configuration**"
          - ✏️✓ enable option "**Enable Interface**"
          - ✏️✓ type **`DMZ`** into "**Description**"
          - ✏️✓ select **`Static IPv4`** as "**IPv4 Configuration Type**"
          - ✏️✓ select **`Static IPv6`** as "**IPv6 Configuration Type**"
          - focus on "**Static IPv4 Configuration**"
          - ✏️✓ type **`fwdmzipv4addressfw`** into "**IPv4 Address**"
          - ✏️✓ select **`fwdmzipv4cidrfw`** (CIDR) for "**IPv4 address**"
          - focus on "**Static IPv6 Configuration**"
          - ✏️✓ type **`fwdmzipv6addressfw`** into "**IPv6 Address**"
          - ✏️✓ select **`fwdmzipv6cidrfw`** (CIDR) for "**IPv6 address**"
          - ✓ click on "**💾 Save**"
          - click on "**✔️ Apply Changes**"

### Enable the DHCP IPv4 server.,

- ⬆ select menu option "**Services > DHCP Server**"
     - navigate to tab "**DMZ**"
     - focus on "**General Settings**"
     - ✏️✓ enable option "**Enable**" ("Enable DHCP server on DMZ interface")
     - focus on "**Primary Address Pool**"
     - ✏️✓ type **`fwdmzdhcpstartipv4addressfw`** and **`fwdmzdhcpendipv4addressfw`** into "**Address Pool Range**" ("From" and "To")
     - focus on "**Other DHCP Options**"
     - ✏️✓ type **`fwdmzdomainfw`** into "**Domain Name**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Enable the DHCP IPv6 server.,

- ⬆ select menu option "**Services > DHCP6 Server**"
     - navigate to tab "**DMZ**"
     - focus on "**General Settings**"
     - ✏️✓ enable option "**Enable**" ("Enable DHCP server on DMZ interface")
     - focus on "**Primary Address Pool**"
     - ✏️✓ type **`fwdmzdhcpstartipv6addressfw`** and **`fwdmzdhcpendipv6addressfw`** into "**Address Pool Range**" ("From" and "To")
     - focus on "**Other DHCPv6 Options**"
     - ✏️✓ type **`fwdmzdomainfw`** into "**Domain Name**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

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
     - navigate to tab "**IP**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Name**"
          - ✏️✓ type **`Web Server IP in DMZ`** into "**Description**"
          - keep **`Host(s)`** as "**Type**"
          - focus on "**Host(s)**"
          - ✏️✓ type **`aohostipv4addressao`** into "**IP or FQDN**"
          - ✏️✓ type **`DMZ Web Server IPv4`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Name**"
          - focus on "**Host(s)**"
          - ✏️✓ type **`aohostipv6addressao`** in "**IP or FQDN**"
          - ✏️✓ type **`DMZ Web Server IPv6`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Alias: Ports of Web Server in DMZ

Define some Aliases for all ports used by the web server. This Alias will be used to allow/block communication.

The ports in this example are for a system with Nextcloud, coturn and OpenVPN.

- ⬆ select menu option "**Firewall > Aliases**"
     - navigate to tab "**Ports**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Name**"
          - ✏️✓ type **`All ports served by the web server.`** into "**Description**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`80`** into "**Port**"
          - ✏️✓ type **`HTTP`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`443`** into "**Port**"
          - ✏️✓ type **`HTTPS`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`10000:20000`** into "**Port**"
          - ✏️✓ type **`TURN Server ports`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`3478`** into "**Port**"
          - ✏️✓ type **`TURN Server`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`5349`** into "**Port**"
          - ✏️✓ type **`TURN Server`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`1194`** into "**Port**"
          - ✏️✓ type **`OpenVPN`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Name**"
          - ✏️✓ type **`All ports served by the web server.`** into "**Description**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`10000:20000`** into "**Port**"
          - ✏️✓ type **`TURN Server ports`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`3478`** into "**Port**"
          - ✏️✓ type **`TURN Server`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`5349`** into "**Port**"
          - ✏️✓ type **`TURN Server`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`1194`** into "**Port**"
          - ✏️✓ type **`OpenVPN`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Outbound_TCP`** into "**Name**"
          - ✏️✓ type **`All ports to the internet requied by the web server.`** into "**Description**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`80`** into "**Port**"
          - ✏️✓ type **`HTTP`** into "**Description**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`443`** into "**Port**"
          - ✏️✓ type **`HTTPS`** into "**Description**"
          - click on "**➕ Add Port**"¹
          - ✏️✓ type **`465`** into "**Port**"
          - ✏️✓ type **`SMTP`** into "**Description**"
          - click on "**➕ Add Port**"¹
          - ✏️✓ type **`587`** into "**Port**"
          - ✏️✓ type **`SMTP`** into "**Description**"
          - click on "**➕ Add Port**"¹
          - ✏️✓ type **`993`** into "**Port**"
          - ✏️✓ type **`IMAP`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`DMZ_Outbound_UDP`** into "**Name**"
          - ✏️✓ type **`All ports to the internet requied by the web server.`** into "**Description**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`123`** into "**Port**"
          - ✏️✓ type **`NTP`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ Example for allowing as well the communication with IMAP and SMTP servers.

The first colums of list "Firewall Aliases Ports" will now look like:

| Name | Type | Values | Description |
| :--- | :--- | --- | :--- |
| DMZ_Inbound_TCP | Port(s) | 80, 443, 10000:20000, 3478, 5349, 1194 | All ports served by the web server. |
| DMZ_Inbound_TCP | Port(s) | 10000:20000, 3478, 5349, 1194 | All ports served by the web server. |
| DMZ_Outbound_Ports | Port(s) | 80, 443 | All ports to the internet requied by the web server. |

### Block DMZ to LAN communication

Prevent the DMZ to communicate with the LAN.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**DMZ**"
     - ⬆ click on "**⤴ Add**"
          - focus on "**Edit Firewall Rule**"
          - ✏️✓ select **`Block`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - ✏️✓ select **`IPv4+IPv6`** as "**Address Family**"
          - ✏️✓ select **`Any`** as "**Protocol**"
          - focus on "**Source**"
          - ✏️✓ select **`DMZ address`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`LAN address`** as "**Destination**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Allow DMZ to Firewall and Internet (minimum)

Allow the ICMP > Echo request from the web server to the internet.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**DMZ**"
     - ⬆ click on "**⤵ Add**"¹
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - keep **`IPv4+IPv6`** as "**Address Family**"
          - ✏️✓ select **`TCP/UDP`** as "**Protocol**"
          - focus on "**Source**"
          - ✏️✓ select **`DMZ subnets`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`This Firewall (self)`** as "**Destination**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⤵ Add**"¹
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - keep **`IPv4+IPv6`** as "**Address Family**"
          - ✏️✓ select **`ICMP`** as "**Protocol**"
          - ✏️✓ select only **`Echo request`** as "**ICMP Subtypes**"
          - focus on "**Source**"
          - ✏️✓ select **`DMZ subnets`** as "**Source**"
          - focus on "**Destination**"
          - keep **`Any`** as "**Destination**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ Using the second button, will ensure that the new rule will be added to the end of the list.

### Allow DMZ Internet communication (solution specific)

Allow TCP/IP communication from the web server to the internet.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**DMZ**"
     - ⬆ click on "**⤵ Add**"¹
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - keep **`IPv4+IPv6`** as "**Address Family**"
          - keep **`TCP`** as "**Protocol**"
          - focus on "**Source**"
          - ✏️✓ select **`DMZ subnets`** as "**Source**"
          - focus on "**Destination**"
          - keep **`Any`** as "**Destination**"
          - keep **`(other)`** as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Outbound_TCP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`(other)`** as "**To**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Outbound_TCP`** into "**Custom**" of "**Destination Port Range**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⤵ Add**"¹
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`DMZ`** as "**Interface**"
          - keep **`IPv4+IPv6`** as "**Address Family**"
          - keep **`UDP`** as "**Protocol**"
          - focus on "**Source**"
          - ✏️✓ select **`DMZ subnets`** as "**Source**"
          - focus on "**Destination**"
          - keep **`Any`** as "**Destination**"
          - keep **`(other)`** as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Outbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`(other)`** as "**To**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Outbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ Using the second button, will ensure that the new rule will be added to the end of the list.

The first colums of list "Rules (Drag to Change Order)" will now look like:

| ☐ |  | States | Protocol | Source | Port | Destination | Port |
| :---: | :---: | ---: | :--- | :--- | :---: | :--- | :--- |
| ☐ | ❌ | 0/0 B | IPv4+6 * | DMZ subnets | * | LAN address | * |
| ☐ | ✔ | 0/0 B | IPv4+6 TCP/UDP | DMZ subnets | * | This Firewall (self) | 53 (DNS) |
| ☐ | ✔ | 0/0 B | IPv4+6 ICMP echoreq | DMZ subnets | * | * | * |
| ☐ | ✔ | 0/0 B | IPv4+6 TCP | DMZ subnets | * | * | DMZ_Outbound_TCP |

Note: *The rule, for blocking the communication to the LAN and following the others.*

### Enable Port Forwarding (WAN->DMZ)

Activate the port forwording (TCP/UDP) to the web server in the DMZ. The IPv6 entries are basically duplicates of the IPv4 entries, with the "4" changed to "6" in fields "**Address Family**"and "**Address**".

- ⬆ select menu option "**Firewall > NAT**"
     - stay to tab "**Port Forward**"
     - ⬆ click on "**⤴ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`TCP`** as "**Protocol**"
          - keep **`WAN address`** as "**Destination**"
          - ✏️✓ keep **`(Other)`** as "**From port**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`(Other)`** as "**To port**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`Address or Alias`** as "**Redirect target IP**" ("Type")
          - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Address**" of "**Redirect target IP**"
          - keep **`Other`** as "**Redirect target port**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Redirect Internet TCP traffic to Web Server.`** into "**Description**"
          - keep **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Redirect target IP**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - keep **`WAN address`** as "**Destination**"
          - ✏️✓ keep **`(other)`** as "**From port**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`(other)`** as "**To port**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`Address or Alias`** as "** Redirect target IP **"
          - ✏️✓ type "DMZ_Web_Server_IPv4` into "**Address**" of "**Redirect target IP**"
          - keep **`Other`** as "**Redirect target port**"
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Redirect Internet UDP traffic to Web Server.`** into "**Description**"
          - keep **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Redirect target IP**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ This option will automatically generate the Firewall Rule to allow Inbound-Traffic. See below section "[Inbound NAT associated filter rules](#inbound-nat-associated-filter-rules)"

Note: *For the last rule the value for "**Filter rule association**" might be shown as "Rule NAT Redirect Internet TCP traffic to Web Server.", however once saved the value will be "Rule NAT Redirect Internet UDP traffic to Web Server.".*

### Enable Port Forwarding (LAN->DMZ)

Route the traffic, to the Web-Server, from the LAN directly to the Web-Server. The IPv6 entries are basically duplicates of the IPv4 entries, with the "4" changed to "6" in fields "**Address Family**"and "**Address**".

- ⬆ select menu option "**Firewall > NAT**"
     - stay to tab "**Port Forward**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`LAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`Any`** as "**Protocol**"
          - keep **`WAN address`** as "**Destination**"
          - keep **`Address or Alias`** as "**Redirect target IP**" ("Type")
          - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Address**" of "**Redirect target IP**"
          - keep **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Redirect target IP**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ This option will automatically generate the Firewall Rule to allow Inbound-Traffic. See below section "[Inbound NAT associated filter rules](#inbound-nat-associated-filter-rules)"

Note: *For the last rule the value for "**Filter rule association**" might be shown as "Rule NAT Redirect Internet TCP traffic to Web Server.", however once saved the value will be "Rule NAT Redirect Internet UDP traffic to Web Server.".*



The creation of the NAT - Port Forward rules should have created the corresponding rules to allow internet traffic to passed to the Web Server.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"

The first colums of list "Firewall Aliases Ports" will now look like:

| ☐ |  | States | Protocol | Source | Port | Destination | Port |
| :---: | :---: | ---: | :--- | :--- | :---: | :--- | :--- |
| | ❌ | 0/0 B | * | RFC 1918 networks | * | * | * |
| | ❌ | 0/0 B | * | Reserved Not assigned by IANA | * | * | * |
| ☐ | ✔ | 0/0 B | IPv4 TCP | * | * | DMZ_Web_Server_IPv4 | DMZ_Inbound_TCP |
| ☐ | ✔ | 0/0 B | IPv6 TCP | * | * | DMZ_Web_Server_IPv6 | DMZ_Inbound_TCP |
| ☐ | ✔ | 0/0 B | IPv4 UDP | * | * | DMZ_Web_Server_IPv4 | DMZ_Inbound_UDP |
| ☐ | ✔ | 0/0 B | IPv6 UDP | * | * | DMZ_Web_Server_IPv4 | DMZ_Inbound_UDP |

Note: *The column "**Description**" should be "NAT" followed by the description of the corresponding Port Forward definition.*

### Inbound NAT associated filter rules

In the case the rules were not autmatically created or got deleted at a later point in time, the following shows how to create those rules manually. Or take the instruction to verify the automatically created rules.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`Pass`** as "**Action**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`TCP`** as "**Protocol**"
          - focus on "**Source**"
          - keep **`Any`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`Address or Alias`** as "**Destination**"
          - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Destination Address**" of "**Destination**"
          - keep **`(other)`** as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Custom**" of "**Redirect target port**"
          - keep **`(other)`** as "**To**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_TCP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Allow Internet TCP traffic to Web Server.`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - focus on "**Destination**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Destination**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`Pass`** as "**Action**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - focus on "**Source**"
          - keep **`Any`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`Address or Alias`** as "**Destination**"
          - ✏️✓ type **`DMZ_Web_Server_IPv4`** into "**Destination Address**" of "**Destination**"
          - keep **`(other)`** as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - keep **`(other)`** as "**To**" of "**Destination Port Range**"
          - ✏️✓ type **`DMZ_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Allow Internet UDP traffic to Web Server.`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - focus on "**Destination**"
          - ✏️✓ type **`DMZ_Web_Server_IPv6`** into "**Address**" of "**Destination**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

## DNS restriction

Redirect all unencrypted DNS requests to the OPNsense system, regardless of the chosen DNS server. This will avoid on IPv4 the bypass of the OPNsense Systems DNS.

- ⬆ select menu option "**Firewall > NAT**"
     - navigate to tab "**Port Forward**"
     - ⬆ click on "**? Add**"¹
          - ✏️✓ select **`LAN`** as "**Interface**"
          - keep selected **`IPv4`** as "**TCP/IP Version**"
          - keep **`TCP`** as "**Protocol**"
          - ✏️✓ select **`Any`** as "**Destination**"
          - ✏️✓ select **`DNS`** as "**From port**" of "**Destination port range**"
          - keep selected **`DNS`** as "**To port**" of "**Destination port range**"
          - keep selected **`Address or Alias`** as "**Type**" of "**Redirect target IP**"
          - ✏️✓ type **`127.0.0.1`** as "**Address**" of "**Redirect target IP**"
          - ✏️✓ select **`DNS`** as "**Redirect target port**"
          - ✏️✓ type **`Redirect all DNS requests to Firewall`** as "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ✓ click on "**✔️ Apply Changes**"

¹ At this point not relevant, which add button to use.

## DSL-Modem usage (as replacement for DSL-Router)

At a later point in time, the DSL-Router can be replaced by a DSL-Modem, allowing pfServer to direct communicate with the internet.

Before we forget it, the changes of [**Allow DSL-Router LAN traffic**](#allow-dsl-router-lan-traffic) have to be revoked.

- ⬆ select menu option "**Interfaces > WAN**"
     - focus on "**Reserved Networks**"
     - ✏️✓ enable "**Block private networks and loopback addresses**" ("Blocks traffic from IP addresses that are reserved for private networks ...")
     - ✏️✓ enable "**Block bogon networks**" ("Blocks traffic from reserved IP addresses (but not RFC 1918) or not yet assigned by IANA. ...")
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

The biggest change to make in the pfSense configuration is from a simple "WAN -> re0 (...)" to a chain of configurations "WAN -> PPPoE -> VLAN -> re0 (...)".

- ⬆ select menu option "**Interfaces > Assignments**"
     - navigate to tab "**VLANs**"
     - ⬆ click on "**➕ Add**"
          - ✏️✓ select **`fwwaninterfacefw (fwwanmacaddressfw)`** as "**Parent Interface**"
          - ✏️✓ type **`dlvlaniddl`**¹ as "**VLAN Tag**"
          - ✏️✓ type **`ISP VLAN ID dlvlaniddl`** as "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - navigate to tab "**PPPs**"
     - ⬆ click on "**➕ Add**"
          - ✏️✓ select **`PPPoE`** as "**Link Type**"
          - ✏️✓ select **`ISP VLAN ID dlvlaniddl`** as "**Link Interface(s)**"
          - ✏️✓ type **`ISP DSL PPPoE`** as "**Description**"
          - ✏️✓ type **`dluserdl`**² as "**"Username**"
          - ✏️✓ type **`dlpassworddl`**² as "**Password**"
          - ✏️✓ select **`Custom`** as "**Periodic reset**"
          - ✏️✓ type **`dlresthourdl`** as "**Hour**" of "**Custom reset**"
          - ✏️✓ type **`dlrestminutedl`** as "**Minute**" of "**Custom reset**"
          - ✓🔙 click on "**💾 Save**"
     - navigate to tab "**Interface Assignments**"
     - ✏️✓ select **`PPPoE`** as "**WAN > Network port**"
     - ✓ click on "**💾 Save**"
     - ⬆ click on "**WAN**"
          - focus on "**General Configuration**"
          - ✏️✓ select **`PPPoE`** as "**IPv4 Configuration Type**"
          - ✏️✓ select **`DHCP6`** as "**IPv6 Configuration Type**"
          - focus on "**DHCP6 Client Configuration**"
          - ✏️✓ enable option "**Use IPv4 connectivity as parent interface**" ("Request a IPv6 prefix/information through the IPv4 connectivity link")
          - focus on "**PPPoE Configuration**"
          - ✏️✓ type DSL-Password² as "**Password**"
          - keep empty "**Specific date**" of "**Custom reset**"
          - ✓ click on "**💾 Save**"
          - click on "**Apply Changes**"

¹ *In case of a Telekom DSL-Line, it is the VLAN-ID `7`.*

² *The DSL-User/Password has to be provided by the ISP. In case of the Telekom AG, the user name is `<Anschlusskennung><Internet Zugangsnummer><Mitbenutzernummer>@t-online.de` and the password of the Telekom-User.*

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

- ⬆ select menu option "**System > General Setup**"
     - focus on "**DNS Server Settings**"
     - ✏️✓ remove entries for "**DNS Server**"
     - ✏️✓ enable option "**DNS Server Override**" ("Allow DNS server list to be overridden by DHCP/PPP on WAN or remote OpenVPN server")
     - ✓ click on "**💾 Save**"

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

## Dynamic DNS (DDNS)

### Service Type available

- ⬆ select menu option "**Service** > **Dynamic DNS**"
- ⬆ click "**➕ Add**"

In case your DDNS Provider exists in the listbox "**Service Type**", there is no special instruction needed.

Create two entries, in case listbox includes **`<provider> (v6)`** as well.

### Service Type **`Custom`**/**`Custom (v6)`**

In case your DDNS Provider is not in the listbox "**Service Type**", the **`Custom*`** entry has to be chosen and provider specific configuration have to be made.

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

## Geo-Fencing

### Package Installation

- ⬆ select menu option "**System** > **Package Manager**"
     - navigate to tab "**Available Packages**"
     - type **`pfBlockerNG`** in field "**Search term**"
     - click on "**🔍 Search**"
     - ✏️✓ click on "**➕ Install**" of package line "**nut**"
     - ✓ click "**✓ Confirm**" in tab "**Package Installer**"

### Setup

Without a previous installationo of the package, a Wizard will guide you through the initial configuration.

- ⬆ select menu option "**Firewall** > **pfBlockerNG**"
     - dialog "Wizard > pfBlockerNG Setup"
     - click on "**≫ Next**"
     - click on "**≫ Next**"
     - ✏️✓ select **`WAN`** as "**Select Inbound Firewall Interface**"
     - ✏️✓ select **`DMZ`** as "**Select Outbound Firewall Interface**"
     - click on "**≫ Next**"
     - keep `10.10.10.1`¹ as "**VIP Address**"
     - keep `8081` as "**Port**"
     - keep `8443` as "**SSL Port**"
     - ✏️✓ enable "**IPv6 DNSBL**"
     - keep "**DNSBL Whitelist**" enabled
     - click on "**≫ Next**"
     - click on "**≫ Finish**"

¹ Default value can be keeped, if there is nothing else on that IP-Address.

     - navigate to "**Update**"
     - keep `Update` as "**Select 'Force' option**"
     - keep `All` as "**Select 'Reload' option**"

     - navigate to "**IP**"
     - focus on "**MaxMind GeoIP configuration**"
     - ✏️✓ type Account-ID as "**MaxMind Account ID**"
     - ✏️✓ type License Key as "**MaxMind License Key**"
     - click on "**💾 Save IP Settings**"
     - navigate to "**Update**"
     - click on "**⮊ Run**"
     - navigate to "**IP > GeoIP**"

#### Germany Allowed Only

- ⬆ select menu option "**Firewall** > **pfBlockerNG**"
     - navigate to "**IP** > **GeoIP**"
     - ✏️✓ select **`Deny Inbound`** for "**Action**" for all lines except "**Europe**"
     - ✏️✓ select **`Permit Inbound`** for "**Action**" in line for "**Europe**"
     - click on "**💾 Save**"
     - ✓ click on "**OK**" regarding "Save settings and/or page 'Order' changes?"
     - click on "**🖉**" in line for "**Europe**"
     - navigated to "**IP > GeoIP > Europe**"
     - ✏️✓ select all **`Germany (*)`** entries for IPv4
     - ✏️✓ select all **`Germany (*)`** entries for IPv6
     - expand section "**Advanced Inbound Firewall Rule Settings**"
     - keep ~~disabled~~ "**nvert Source**"
     - ✏️✓ enable "**Custom DST Port**" ("Enable")
     - type **`DMZ_Inbound`** in text field next to checkbox
     - ✏️✓ select **`TCP/UDP`** for "**Customer Protocol**"
     - ✓ click on "**💾 Save**"

## Guest Network

We are going to setup a new network, which has Internet-Access only. It is a different approach than [Internet Access Control](#internet-access-control), as any system within the Guest Network will have **only** Internet-Access and being completely isolated from the other networks.

The Guest Network can be either used directly via LAN or Wi-Fi, if conneced to a Wi-Fi Access Point (AP). As AP even a participation with a wireless community network e.g. Freifunk is possible. Especially for the last case such a Guest Network should be used instead of a regular LAN, as the AP is not fully under controll of ourself.

- ⬆ select menu option "**Interfaces > Assignments**"
     - ✏️✓ select **`pfguestinterfacepf (pfguestmacaddresspf)`** from listbox "**Available network ports**"
     - ✏️✓ click on "**➕ Add**"
     - ✓ click on "**💾 Save**"

Make the required changed to the configuration of the assigned interface.

- ⬆ select menu option "**Interfaces > Assignments**"
     - ⬆ click on the name (hyperlink) of interface pfguestinterfacepf
          - focus on "**General Configuration**"
          - ✏️✓ enable option "**Enable Interface**"
          - ✏️✓ type **`Guest-LAN`** into "**Description**"
          - ✏️✓ select **`Static IPv4`** as "**IPv4 Configuration Type**"
          - ✏️✓ select **`Static IPv6`** as "**IPv6 Configuration Type**"
          - focus on "**Static IPv4 Configuration**"
          - ✏️✓ type **`fwguestipv4addressfw`** into "**IPv4 Address**"
          - ✏️✓ select **`fwguestipv4cidrfw`** (CIDR) for "**IPv4 address**"
          - focus on "**Static IPv6 Configuration**"
          - ✏️✓ type **`fwguestipv6addressfw`** into "**IPv6 Address**"
          - ✏️✓ select **`fwguestipv6cidrfw`** (CIDR) for "**IPv6 address**"
          - ✓ click on "**💾 Save**"
          - click on "**✔️ Apply Changes**"

### Enable the DHCP IPv4 server.,

- ⬆ select menu option "**Services > DHCP Server**"
     - navigate to tab "**Guest-LAN**"
     - focus on "**General Settings**"
     - ✏️✓ enable option "**Enable**" ("Enable DHCP server on DMZ interface")
     - focus on "**Primary Address Pool**"
     - ✏️✓ type **`fwguestdhcpstartipv4addressfw`** and **`fwguestdhcpendipv4addressfw`** into "**Address Pool Range**" ("From" and "To")
     - focus on "**Other DHCP Options**"
     - ✏️✓ type **`fwguestdomainfw`** into "**Domain Name**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Enable the DHCP IPv6 server.,

- ⬆ select menu option "**Services > DHCP6 Server**"
     - navigate to tab "**DMZ**"
     - focus on "**General Settings**"
     - ✏️✓ enable option "**Enable**" ("Enable DHCP server on DMZ interface")
     - focus on "**Primary Address Pool**"
     - ✏️✓ type **`fwguestdhcpstartipv6addressfw`** and **`fwguestdhcpendipv6addressfw`** into "**Address Pool Range**" ("From" and "To")
     - focus on "**Other DHCPv6 Options**"
     - ✏️✓ type **`fwguestdomainfw`** into "**Domain Name**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

Note: Leave the Router Mode for the Guest-LAN disabled.

### Remove **`Guest-LAN`** from DNS Resolver

- ⬆ select menu option "**Services** > **DNS Resolver**"
     - navigate to tab "**General Settings**"
     - ✏️✓ un-select **`Guest-LAN`** from "**Network Interfaces**" ¹
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ In case **`All`** is currently selected, do the following to un-select **`Guest-LAN`**

- click on the first entry in the list
- scroll down to the end of the list
- press and hold down SHIFT
- click on the last entry
- release SHIFT
- press and hold down CTRL
- click on **`All`**
- click on **`Guest-LAN`**
- release CTRL

### Add **`Guest-LAN`** to DNS Forwarder

- ⬆ select menu option "**Services** > **DNS Forwarder**"
     - focus on "**General DNS Forwarder Options**"
     - ✏️✓ enable "**Enable**" ("Enable DNS forwarder")
     - ✏️✓ type **`53`** in "**Listen Port**"
     - ✏️✓ select **`Guest-LAN`** from "**Interfaces**" ¹
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ In case **`All`** is currently selected, do the following to select **`Guest-LAN`**

- click on **`Guest-LAN`**

### Restict Firewall Rules for **`Guest-LAN`**

First allow the DNS-Access (to the DNS Forwarder).

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**Guest-LAN**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`Guest-LAN`** as "**Interface**"
          - ✏️✓ select **`IPv4+IPv6`** from "**Address Family**"
          - ✏️✓ select **`TCP/UDP`** from "**Protocol**"
          - focus on "**Source**"
          - keep **`Any`** from "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`This Firewall (self)`** from "**Destination**"
          - ✏️✓ select **`DNS (53)`** from "**From**" of "**Destination Port Range**"
          - focus on "**Extra Options**"
          - ✏️✓ type **`Allow DNS`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

For every interface, to be isolate from the Guest-LAN, execute the following steps.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**Guest-LAN**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Firewall Rules**"
          - ✏️✓ select **`Block`** from "**Action**"
          - keep **`Guest-LAN`** as "**Interface**"
          - ✏️✓ select **`IPv4+IPv6`** from "**Address Family**"
          - ✏️✓ select **`Any`** from "**Protocol**"
          - focus on "**Source**"
          - keep **`Any`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`??? address`** from "**Destination**" ¹
          - focus on "**Extra Options**"
          - ✏️✓ type **`Isolate Guest-LAN from others`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ With **`???`** to be the interface name, which should be isolated from the Guest-LAN.

Finally grant access to anyone, except to the Firewall.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**Guest-LAN**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`Guest-LAN`** as "**Interface**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ select **`Any`** from "**Protocol**"
          - focus on "**Source**"
          - keep **`Any`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ enable **`Invert match`** from "**Destination**"
          - ✏️✓ select **`This Firewall (self)`** from "**Destination**" ¹
          - focus on "**Extra Options**"
          - ✏️✓ type **`Allow all kind of Traffic`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

With for example LAN- and DMZ-Interfaces defined, the rules for Guest-LAN should look like the following. Those reules to block traffic to the private interfaces have to be between the first allowed rule (DNS) and the last allowed rule (except Firewall).

| State | Protocol       | Source | Port | Destination            | Port     | Gateway | Queue | Schedule | Description |
| :---- | :------------- | :----- | :--- | :--------------------- | :------- | :------ | :---- | :------- | :---------- |
| ✔️ ... | IPv4+6 TCP/UDP | *      | *    | This Firewall (self)   | 53 (DNS) | *       | none  |          | Allow DNS   |
| ❌ ... | IPv4+6 *       | *      | *    | DMZ address            | *        | *       | none  |          | Isolate     |
| ❌ ... | IPv4+6 *       | *      | *    | LAN address            | *        | *       | none  |          | Isolate     |
| ✔️ ... | IPv4+6 *       | *      | *    | ! This Firewall (self) | *        | *       | none  |          | Allow All   |




## Internet Access Control

QQQ TODO QQQ

With more and more devices - black boxes - coming with a WiFi and/or LAN connection, the device owner do want to have controll on the devices communication. Either to prevent automatic updates or calling home.

In case a network connection and/or App is required for the activation of a device ... most likely such requirement is not advertised nor mentioned anywhere ... DO RETURN THE DEVICE and request your money back.

Usually these devices have a known¹ MAC address and use DHCP to aquire an IP-Address.

¹ If the device has a label with the MAC address, that is easy. But even without such label and/or usage of IPv6, the MAC address and/or DUID can be identified by looking at the DHCP leases (**Status > DHCP Leases**" & "**Status > DHCPv6 Leases**"). Looking at the leases and connecting a device should be done with the COMPLETE traffic from the LAN being blocked.

### General Allow-List



Alias > IP
focus on section "**Properties**"
**`LAN0TOP_Allow_Internet`** in "**Name**"
**`192.168.20.1 - 127 / fd00:0:0:0:0192:0168:2000:0001-01ff`** in "**Description**"
Type: Network(s)
focus on section "**Network(s)**"
**`fd00:0000:0000:0000:0192:0168:2000:0000`** **`119`**	**`IPv6 LAN0`** in line "**Network or FQDN**"
click on "**I Add Network**"
**`192.168.20.0`** **`25`**	**`IPv4 LAN0`** in new line "**Network or FQDN**"
click on "**I Save**"

**`LAN1EGP_Allow_Internet`** in "**Name**"

**`LAN2UG_Allow_Internet`** in "**Name**"

**`LAN3DG_DMZ_Allow_Internet`** in "**Name**"





### Dedicated Deny-List

Basically the steps are the following.

1. Create a DHCP Static Mapping based on the MAC address / DUID (identifier). See [**DHCP Client - Fix IP-Address**](#dhcp-client---fix-ip-address)
2. Create/maintaine a Host(s) Alias list called "IoT Devices" with the devices name (see Step 1.)
3. Create/maintaine a Block Rule list called "Deny IoT Devices Traffic" with the Source Alias "IoT Devices" (see Step 2.)
4. In case certain Internet addresses should be allowed, create/maintaine a Allow Rule list called "Allow IoT Devices Traffic" with the Source Alias "IoT Devices" (see Step 2.)

Note: Businesses should buy a pfSense+ subscription, as this product variant comes with the feature "**Ethernet (Layer 2) Rules**" (incl. filtering on MAC addresses).

#### Access Control List (ACL) - Alias "IoT Devices" 

- ⬆ select menu option "**Firewall > Aliases**"
     - navigate to tab "**IP**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`ACL_IoT_Devices`** into "**Name**"
          - ✏️✓ type **`ACL: IoT Devices`** into "**Description**"
          - keep `Host(s)` as "**Type**"
          - focus on "**Host(s)**"
          - ✏️✓ type host FQDN into "**IP or FQDN**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

#### Rule "Deny IoT Devices Traffic"

#### Rule "Allow IoT Devices Traffic"

## Internet Disable (temporarily)

Under some circumstances, e.g. configuring the [Internet Access Control](#internet-access-control), it might be required to temporarily turn off the internet at all. For example to identify MAC-Address of a device, without allowing internet access during the time of discovery.

### Turn OFF

- ⬆ select menu option "**Interface > WAN**"
     - focus on section "**General Configuration**"
     - ✏️✓ ~~disable~~ "**Enable**" ("Enable interface")
     - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Turn ON

- ⬆ select menu option "**Interface > WAN**"
     - focus on section "**General Configuration**"
     - ✏️✓ enable "**Enable**" ("Enable interface")
     - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

## IP-Phone (SIP-Gateway)

Several different configurations are required.

- define Outbound Rules (SIP-Gateway -> Internet)
- define Inbound Rules (Internet -> SIP-Gateway)
- enable 1:1 mapping of the ports (Hybrid Outbound NAT + Mapping)

### Alias: IP-Address of SIP-Gateway

Define an Alias for IP-Address of the SIP-Gateway. This Alias will be used to allow/block communication.

- ⬆ select menu option "**Firewall** > **Aliases**"
     - navigate to tab "**IP**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Name**"
          - focus on "**Host(s)**"
          - ✏️✓ type **`vpsipipv4addressvp`** into "**IP or FQDN**"
          - ✏️✓ type **`SIP Gateway IPv4`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Properties**"
          - ✏️✓ type **`SIP_Gateway_IPv6`** into "**Name**"
          - focus on "**Host(s)**"
          - ✏️✓ type **`vpsipipv6addressvp`** in "**IP or FQDN**"
          - ✏️✓ type **`SIP Gateway IPv6`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### Alias: Ports of SIP-Gateway

Define some Aliases for all ports used by the SIP-Gateway. This Alias will be used to allow/block communication.

The ports in the following instruction are taken from the Telekom AG website and might be different to your ISP or SIP-Provider. See [Telekom: Wie kann ich meinen Festnetz-Anschluss mit Voice over IP bzw. einem SIP-Client nutzen?](https://www.telekom.de/hilfe/internet-telefonie/telefonie/voice-over-ip-sip-client?samChecked=true)

- ⬆ select menu option "**Firewall > Aliases**"
     - navigate to tab "**Ports**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Name**"
          - ✏️✓ type **`SIP UDP Ports inwards`** into "**Description**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`5070`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`5080`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`30000:31000`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`40000:41000`** into "**Port**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`SIP_Outbound_TCP`** into "**Name**"
          - ✏️✓ type **`SIP TCP Ports outwards`** into "**Description**"
          - focus on "**Port(s)**"
          - ✏️✓ type **`80`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`443`** into "**Port**"
          - click on "**➕ Add Port**"
          - ✏️✓ type **`65002`** into "**Port**"
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**➕ Add**"
          - focus on "**Properties**"
          - ✏️✓ type **`SIP_Outbound_UDP`** into "**Name**"
          - ✏️✓ type **`IP-Phone UDP Ports outwards`** into "**Description**"
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
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

The first colums of list "Firewall Aliases Ports" will now look like for the newly created aliases:

| Name | Type | Values | Description |
| :--- | :--- | --- | :--- |
| SIP_Inbound_UDP | Port(s) | 5070, 5080, 30000:31000, 40000:41000 | IP-Phone UDP Ports inwards |
| SIP_Outbound_TCP | Port(s) | 80, 443 | IP-Phone TCP Ports outwards |
| SIP_Outbound_UDP | Port(s) | 5060, 30000:31000, 40000:41000, 3478, 3479 | IP-Phone TCP Ports outwards |

### Allow SIP-Gateway Internet communication (SIP-Gateway specific)

Allow TCP/IP communication from the SIP-Gateway to the internet.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**LAN**"
     - ⬆ click on "**⤵ Add**"¹
          - focus on "**Edit Firewall Rules**"
          - keep **`Pass`** as "**Action**"
          - keep **`LAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - keep **`UDP`** as "**Protocol**"
          - focus on "**Source**"
          - ✏️✓ select **`Address or Alias`** as "**Source**"
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Source Address**" of "**Source**"
          - focus on "**Destination**"
          - keep **`Any`** as "**Destination**"
          - keep **`(other)**` as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Outbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`(other)`** as "**To**" of "**Destination Port Range**"
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

The first colums of list "Rules (Drag to Change Order)" will now look like for the newly created rule:

| ☐ |  | States | Protocol | Source | Port | Destination | Port |
| :---: | :---: | ---: | :--- | :--- | :---: | :--- | :--- |
| ☐ | ✔ | 0/0 B | IPv4 UDP | SIP_Gateway_IPv4 | * | * | SIP_Outbound_UDP |
| ☐ | ✔ | 0/0 B | IPv6 UDP | SIP_Gateway_IPv6 | * | * | SIP_Outbound_UDP |

### Enable Port Forwarding (WAN->SIP-Gateway)

Activate the port forwording (UDP) to the SIP-Gateway. The IPv6 entries are basically duplicates of the IPv4 entries, with the "4" changed to "6" in fields "**Address Family**"and "**Address**".

- ⬆ select menu option "**Firewall > NAT**"
     - stay to tab "**Port Forward**"
     - ⬆ click on "**⤴ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - keep **`WAN address`** as "**Destination**"
          - ✏️✓ keep **`(Other)`** as "**From port**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`(Other)`** as "**To port**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Destination Port Range**"
          - keep **`Address or Alias`** as "**Redirect target IP**" ("Type")
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Address**" of "**Redirect target IP**"
          - keep **`Other`** as "**Redirect target port**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - ✏️✓ type **`Redirect Internet UDP traffic to SIP-Gateway.`** into "**Description**"
          - keep **`Add associated filter rule`** as "**Filter rule association**"¹
          - ✓🔙 click on "**💾 Save**"
     - ⬆ click on "**⮺**" ("Add a new NAT based on this one") at the right of the newly created entry
          - focus on "**Edit Redirect Entry**"
          - ✏️✓ select **`IPv6`** as "**Address Family**"
          - ✏️✓ type **`SIP_Gateway_IPv6`** into "**Address**" of "**Redirect target IP**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

¹ This option will automatically generate the Firewall Rule to allow Inbound-Traffic. See below section "[Inbound NAT associated filter rules](#inbound-nat-associated-filter-rules)"

The first colums of list "Rules" (Port Forward) will now look like for the newly created rules:

| ☐ |  |  | Interface | Protocol | Source Address | Source Ports | Dest. Address | Dest. Ports | NAT IP | NAT Ports | Description |
| :---: | :---: | :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| ☐ | ✔ | 🔀 | WAN | UDP | * | * | WAN address | SIP_Inbound_UDP | SIP_Gateway_IPv4 | SIP_Inbound_UDP | Redirect Internet UDP traffic to SIP-Gateway. |
| ☐ | ✔ | 🔀 | WAN | UDP | * | * | WAN address | SIP_Inbound_UDP | SIP_Gateway_IPv6 | SIP_Inbound_UDP | Redirect Internet UDP traffic to SIP-Gateway. |

The creation of the NAT - Port Forward rules should have created the corresponding rules to allow internet traffic to passed to the Web Server.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"

The first colums of list "Firewall Aliases Ports" will now look like for the newly created rules:

| ☐ |  | States | Protocol | Source | Port | Destination | Port |
| :---: | :---: | ---: | :--- | :--- | :---: | :--- | :--- |
| ☐ | ✔ | 0/0 B | IPv4 UDP | * | * | SIP_Gateway_IPv4 | SIP_Inbound_UDP |
| ☐ | ✔ | 0/0 B | IPv6 UDP | * | * | SIP_Gateway_IPv6 | SIP_Inbound_UDP |

Note: *The column "**Description**" should be "NAT" followed by the description of the corresponding Port Forward definition.*

### Inbound NAT associated filter rules

In the case the rules were not autmatically created or got deleted at a later point in time, the following shows how to create those rules manually. Or take the instruction to verify the automatically created rules.

- ⬆ select menu option "**Firewall > Rules**"
     - navigate to tab "**WAN**"
     - ⬆ click on "**⤵ Add**"
          - focus on "**Edit Redirect Entry**"
          - keep **`Pass`** as "**Action**"
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4`** as "**Address Family**"
          - ✏️✓ select **`UDP`** as "**Protocol**"
          - focus on "**Source**"
          - keep **`Any`** as "**Source**"
          - focus on "**Destination**"
          - ✏️✓ select **`Address or Alias`** as "**Destination**"
          - ✏️✓ type **`SIP_Gateway_IPv4`** into "**Destination Address**" of "**Destination**"
          - keep **`(other)`** as "**From**" of "**Destination Port Range**"
          - ✏️✓ type **`SIP_Inbound_UDP`** into "**Custom**" of "**Redirect target port**"
          - keep **`(other)`** as "**To**" of "**Destination Port Range**"
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
          - keep **`WAN`** as "**Interface**"
          - keep **`IPv4+IPv6`** as "**Address Family**"
          - ✏️✓ select **`TCP/UDP`** as "**Protocol**"
          - keep **`Any`** as "**Source**"
          - keep **`Any`** as "**Destination**"
          - focus on "**Translation**"
          - keep **`WAN address`** as "**Address**"
          - keep "**Port or Range**" unset
          - ✏️✓ enable "**Static Port**"
          - ✏️✓ type **`Required by STUN/TURN traffic`** into "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

## IP-Range change

In case of a planned IP-Range change for the DMZ/LAN network, the order of operation is important. Click on "**✔️ Apply Changes**" only when instructioned to do so, as it is important to apply the changes on the DHCP/DHCP6 Server(s) first.

1. open a connection to those server with a fix IP-Address (see Step 6.)
2. adjust the IP address(es) of the DMZ/LAN interface(s)
3. adjust the IP-Range of the DHCP/DHCP6 Server(s)
4. to apply the changes on the DHCP/DHCP6 Server(s), click on "**✔️ Apply Changes**"
5. navigate back to the DMZ/LAN interface(s), click on "**✔️ Apply Changes**"
6. for those server with a fix IP-Address, apply the change in taking the new IP-Range into account
7. disconnect and connect again the Ethernet Cable of the workstation

## IPv6 Enablement

Quick check that pfSense has IPv6 enable.

- ⬆ select menu option "**System** > **Advanced**"
     - navigate to tab "**Networking**"
     - focus on "**IPv6 Options**"
     - ✏️✓ enable "**Allow IPv6**" ("All IPv6 traffic will be blocked by the firewall unless this box is checked")
     - ✓🔙 click on "**💾 Save**" (if change was made)

### IPv6 for WAN

Specify IPv6 of WAN-Router.

- ⬆ select menu option "**Interfaces** > **WAN**"
     - focus on "**General Configuration**"
     - ✏️✓ select **`DHCP6`** as "**IPv6 Configuration Type**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

### IPv6 for LAN

The easiest way to create an IPv6 range is using `fc00` with the same numbers as used for the IPv4 range.

Specify IPv6 address for the LAN-Gateway.

- ⬆ select menu option "**Interfaces** > **LAN**"
     - focus on "**General Configuration**"
     - ✏️✓ select **`Static IPv6`** as "**IPv6 Configuration Type**"
     - focus on "**Static IPv6 Configuration**"
     - ✏️✓ type **`fwlanipv6addressfw`** into "**IPv6 address**"
     - ✏️✓ select **`fwlanipv6cidrfw`** (address block) for "**IPv6 address**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"

Specify the Router Mode for the LAN (prerequisite for DHCP6).

- ⬆ select menu option "**Services** > **Router Advertisement**"
     - navigate to tab "**LAN**"
     - focus on "**Router Advertisement**"
     - ✏️✓ select **`Assisted - RA Flags [managed, other stateful], Prefix Flags [onlink, auto, router]`** as "**Router Mode**"
     - ✓ click on "**💾 Save**"

Specify for LAN DHCPv6 server the IPv6 range.

- ⬆ select menu option "**Services** > **ServicesDHCVPv6 Server**"
     - navigate to tab "**LAN**"
     - focus on "**General Settings**"
     - ✏️✓ enable "**Enable**" ("Enable DHCPv6 server on LAN interface")
     - focus on "**Primary Address Pool**"
     - ✏️✓ type **`fwlandhcpstartipv6addressfw`** into "**From**" of "**Address Pool Range**"
     - ✏️✓ type **`fwlandhcpendipv6addressfw`** into "**To**" of "**Address Pool Range**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**"


## LAN Bridge for all/some LAN interfaces

With an installed 2/4 LAN card installed, a Bridge is capable of bundling multiple LAN interfaces to one.

Note: *That is for none WAN interfaces*

### LAN Bridge "Quad-Port-Bridge LAN0-2"

Note: *With currently the LAN3 interface in use, you will - at the end - have to change **physically** the LAN connector on the pfSense machine!*

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - ✏️✓ select **`fwlan0interfacefw (fwlan0macaddressfw)`** from listbox "**Available network ports**"
     - ✏️✓ click on "**➕ Add**"
     - ✏️✓ select **`fwlan1interfacefw (fwlan1macaddressfw)`** from listbox "**Available network ports**"
     - ✏️✓ click on "**➕ Add**"
     - ✏️✓ select **`fwlan2macaddressfw (fwlan2interfacefw)`** from listbox "**Available network ports**"
     - ✏️✓ click on "**➕ Add**"
     - ✓ click on "**💾 Save**"

- ⬆ select menu option "**Interfaces** > **OPT1 (fwlan0interfacefw)**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✏️✓ type **`LAN0`** into "**Description**"
     - ✓ click on "**💾 Save**"
     - ✏️ click on "**Apply Changes**"

- ⬆ select menu option "**Interfaces** > **OPT2 (fwlan1interfacefw)**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✏️✓ type **`LAN1`** into "**Description**
     - ✓ click on "**💾 Save**"
     - ✏️ click on "**Apply Changes**"

- ⬆ select menu option "**Interfaces** > **OPT3 (fwlan2interfacefw)**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✏️✓ type **`LAN2`** into "**Description**"
     - ✓ click on "**💾 Save**"
     - ✏️ click on "**Apply Changes**"

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - navigate to tab "**Bridges**"
     - ⬆✏️✓ click on "**➕ Add**"
          - ✏️✓ select **`LAN0`**, **`LAN1`** and **`LAN2`** as "**Member Interfaces**"
          - ✏️✓ type **`Quad-Port-Bridge LAN0-2`** in "**Description**"
          - ✓🔙 click on "**💾 Save**"
     - navigate to tab "**Interface Assignments**"
     - ✏️✓ select option "BRIDGE0 (Quad-Port-Bridge LAN0-2)" as "**LAN**" Interface
     - ✓ click on "**💾 Save**"

After the change got saved, from "fwlan3interfacefw (fwlan3macaddressfw)" is no longer working, as the interface is no longer assigned.

Go to the pfSense machine and switch the LAN cable from "fwlan3interfacefw" to one of those from the newly created bridge ("fwlan0interfacefw", "fwlan1interfacefw" or "fwlan2interfacefw").

Back on your workstation, refresh the page in the browser.

In case the interface "fwlan3interfacefw" is planned to be used for a DMZ, the following steps can be skipped and LAN Bridge configuration is finished.

### Add LAN3 to LAN Bridge "Quad-Port-Bridge LAN0-3"

In order to assign "fwlan3interfacefw" to the LAN Bridge, the following steps are required.

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - ✏️✓ select **`fwlan3interfacefw (fwlan3macaddressfw)`** from listbox "**Available network ports**"
     - ✏️✓ click on "**Add**"
     - ✓ click on "**💾 Save**"

- ⬆ select menu option "**Interfaces** > **OPT4 (fwlan3interfacefw)**"
     - ✏️✓ enable option "**Enable Interface**"
     - ✏️✓ type **`LAN3`** into "**Description**"
     - ✓ click on "**💾 Save**"
     - ✏️ click on "**Apply Changes**"

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - navigate to tab "**Bridges**"
     - ⬆ click on "**🖉**" (Pencil icon) to edit interface "**BRIDGE0**"
          - ✏️✓ select **`LAN3`** in addition the already select `LAN0`, `LAN1` and `LAN2` of "**Member Interfaces**"
          - ✏️✓ type **`Quad-Port-Bridge LAN0-3`** in "**Description**"
          - ✓🔙 click on "**💾 Save**"

Go to the pfSense machine and switch the LAN cable back to igb3. So you have the same physical configuration as during setup.

### Remove LAN3 from LAN Bridge "Quad-Port-Bridge LAN0-3"

In order to remove **`fwlan3interfacefw`** to the LAN Bridge, the following steps are required.

- ⬆ select menu option "**Interfaces** > **Assignments**"
     - navigate to tab "**Bridges**"
     - ⬆ click on "**🖉**" (Pencil icon) to edit interface "**BRIDGE0**"
          - ✏️✓ ~~de-select~~ **`LAN3`** leaving only `LAN0`, `LAN1` and `LAN2` selected of "**Member Interfaces**"
          - ✏️✓ type **`Quad-Port-Bridge LAN0-2`** in "**Description**"
          - ✓🔙 click on "**💾 Save**"

## NTP-Server

In order to make the NTP-Server on the pfSense System available to the LAN, those related interfaces have to be selected.

- ⬆ select menu option "**Services** > **NTP**"
     - navigate to tab "**Settings**"
          - focus on section "**NTP Server Configuration**"
          - ✏️✓ enable **`Enable NTP Server`** of "**Enable**"
          - ✏️✓ select all **`LAN*`** entries of "**Interface**"
          - ✓🔙 click on "**💾 Save**"

## OpenVPN Setup

### Adjust Dashboard

Get to the dashboard and add the OpenVPN tile.

- ⬆ click on the "pfsense" logo int the left top corner
     - click on the "➕" above the tiles
     - ✏️  click on "**➕ OpenVPN**"

QQQ TODO QQQ

## Serial Communications

- ⬆ select menu option "**System** > **Advanced**"
     - focus on section "**Serial Communications**"
     - ✏️✓ enable "**Serial Terminal**" ("Enables the first serial port with 115200/8/N/1 by default, or another speed selectable below.")
     - keep **`115200`** as "**Serial Speed**"
     - keep **`Serial Console`** as "**Primary Console**"¹
     - ✓ click on "**💾 Save**"

¹ We change this to **`Serial Console`**, in case we are going to use the serial interface primarily to monitor the pfSense system well being. 

## SSH Setup

In order to use ssh, to connect with the pfSense system, it is highly recommended to use keys instead of the accounts password. See [User - Assign SSH Public Key](#user---assign-ssh-public-key)

- ⬆ select menu option "**System** > **Advanced**"
     - focus on section "**Secure Shell**"
     - ✏️✓ enable "**Secure Shell Server**"
     - ✏️✓ select "Public Key Only" as "**Secure Shell Server**"
     - ✓ click on "**💾 Save**"

### Check Login

From your Workstation you can check is SSH was setup correctly.

```console
$ ssh fwadminfw@fwhostnamefw.fwhostdomainfw
(fwadminfw@fwhostnamefw.fwhostdomainfw) Password for fwadminfw@fwhostnamefw.fwhostdomainfw:
pfSense - Serial: fwserialfw - Netgate Device ID: fwdeviceidfw

*** Welcome to pfSense 2.8.0-RELEASE (amd64) on fwhostnamefw ***

 WAN (wan)       -> re0        -> v4: fwwanip4addressfw/24
                                  v6/DHCP6: .../...
 LAN (lan)       -> igb3       -> v4: fwlanipv4addressfw/24

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

- type **`0`** (for logout)
- press **ENTER**

Press CTRL+C to terminate ssh session.

## UPS

### Package Installation

- ⬆ select menu option "**System** > **Package Manager**"
     - navigate to tab "**Available Packages**"
     - type **`Network UPS Tools`** in field "**Search term**"
     - click on "**🔍 Search**"
     - ✏️✓ click on "**➕ Install**" of package line "**nut**"
     - ✓ click "**✓ Confirm**" in tab "**Package Installer**"

### UPS Setup

- ⬆ select menu option "**Services** > **UPS**"
     - navigate to tab "**UPS Settings**"
     - focus on section "**General Settings**"
     - ✏️✓ select **`Local USB`** as "**UPS Type**"
     - ✏️✓ type fwupsnamefw¹ in field "**UPS Name**" 
     - focus on section "**Driver Settings**"
     - keep **`usbhid`** as "**Driver**"
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
fwadminfw@iThink:~$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/fwadminfw/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/fwadminfw/.ssh/id_ed25519
Your public key has been saved in /home/fwadminfw/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:WzMNibmeNE8UuiMKpX1ZtfIBvo3vg7uO9bMH2a4VJHs fwadminfw@iThink
The key's randomart image is:
+--[ED25519 256]--+
|        . o      |
|       . * +     |
|    .   B B .    |
|   +   o X B     |
|  o . + S OoE    |
|   . o + Xo+..   |
|    .   =.oo.    |
|       o.oo.o    |
|      ..+o=*     |
+----[SHA256]-----+
```

```console
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

#### Make Adjustment

- ⬆ select menu option "**System** > **User Management**"
     - navigate to tab "**Users**"
     - ⬆ click "**🖉**" ("Edit") of the users line
          - focus on section "**Keys**"
          - ✏️✓ insert the public key into "**Authorized SSH Keys**"¹
          - ✓🔙 click on "**💾 Save**"

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
          - keep **`Default (none for all but root)`** as "**Login shell**"
          - keep "**Expiration date**" empty
          - ✔️✏️ select **`admins`** as "**Group membership**"
          - ✔️✏️ select **`All pages`** as "**Provoleges**"
          - ✔️🔙 click on "**Save**"

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

## VPN

### IPsec

#### Prerequisite

1. Users got created. See [User - Create](#user---create)
2. Users Client Certificate got imported. See [User - Import Client Certificate](#user---import-client-certificate)
3. Users got Client Certificates assigned. See [User - Assign Client Certificate](#user---assign-client-certificate)
4. User-Group got created with the following porperties. See [Group - Create](#group---create)

- Short Name: **`IPsec`** as **`<group-short-name>`**
- Description: **`Users able to use IPsec(IPsec)`** as **`<group-description>`**
- Privilege: **`User - VPN: IPsec xauth Dialin`** as **`<privilege>`**
- Member: Those from 1. Step

#### Setup

- ⬆ select menu option "**VPN** > **IPsec**"
     - navigate to tab "**Mobile Clients**"
     - focus on section "**Enable IPsec Mobile Client Support**"
     - ✏️✓ enable "**IKE Extensions**" ("Enable IPsec Mobile Client Support")
     - focus on section "**Extended Authentication (Xauth)**"
     - ✏️✓ select **`Local Database`** as "**User Authentication**"
     - ✏️✓ enable "**Group Authentication**" ("Group Authentication")
     - ✏️✓ select **`Users able to use IPsec(IPsec)`** as "**Authentication Groups**"
     - keep ~~disabled~~ "**RADIUS Accounting**"
     - focus on section "**Client Configuration (mode-cfg)**"
     - ✏️✓ enable "**Virtual Address Pool**" ("Provide a virtual IP address to clients")
     - ✏️✓ type **`10.9.4.0`** for **Network**
     - ✏️✓ select **`24`** for **Subnet**
     - keep ~~disabled~~ "**Virtual IPv6 Address Pool**"
     - keep ~~disabled~~ "**RADIUS IP address priority**"
     - keep ~~disabled~~ "**RADIUS Advanced Parameters**"
     - ✏️✓ enable "**Network List**" ("Provide a list of accessible networks to clients")
     - keep ~~disabled~~ "**Save Xauth Password**"
     - keep ~~disabled~~ "**DNS Default Domain**"
     - ✏️✓ enable "**Split DNS**" ("Provide a list of split DNS domain names to clients. Enter a space separated list.")
     - ✏️✓ type **`aodomainao`** in the field below the checkbox
     - keep ~~disabled~~ "**DNS Servers**"
     - keep ~~disabled~~ "**WINS Servers**"
     - keep ~~disabled~~ "**Phase2 PFS Group**"
     - keep ~~disabled~~ "**Login Banner**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**" in message area "**The IPsec tunnel configuration has been changed. ...**"
     - click on "**➕ Create Phase 1**" in message area "**Support for IPsec Mobile Clients is enabled but a Phase 1 definition was not found. ...**"

- still within menu option "**VPN** > **IPsec**"
     - automatically navigates to tab "**Tunnels**"
     - focus on section "**General Information**"
     - ✏️✓ type **`Tunnel for Mobile Clients (ddvpndd.ddprefixdomaindd.ddbasedomain2dd)`** in "**Description**"
     - keep ~~disabled~~ "**Disabled**"
     - focus on section "**IKE Endpoint Configuration**"
     - keep selected **`IKEv2`** as "**Key Exchange version**"
     - keep selected **`IPv4`** as "**Internet Protocol**"
     - keep selected **`WAN`** as "**Interface**"
     - focus on section "**Phase 1 Proposal (Authentication)**"
     - keep selected **`Mutual Certificate`** as "**Authentication Method**"
     - ✏️✓ select **`Fully qualified domain name`** as "**My identifier**"
     - ✏️✓ type **`ddvpndd.ddprefixdomaindd.ddbasedomain2dd`**¹ in field next to the listbox
     - ✏️✓ select **`Any`** as "**Peer identifier**"
     - ✏️✓ select **`Server ddprefixdomaindd.ddbasedomain2dd`** in "**My Certificate**"
     - ✏️✓ select **`Certificate Authority cacommonnameca`** as "**Peer Certificate Authority**"
     - focus on section "**Phase 1 Proposal (Encryption Algorithm)**"
     - keep selected **`AES`** as "**Algorithm**" ("**Encryption Algorithm**")
     - ✏️✓ select **`256 bits`** as "**Key length**" ("**Encryption Algorithm**")
     - keep selected **`SHA256`** as "**Hash**" ("**Encryption Algorithm**")
     - keep selected **`14 (2048 bit)`** as "**DH Group**" ("**Encryption Algorithm**")
     - click on "**➕ Add Algorithm**"
     - keep selected **`AES`** as "**Algorithm**" ("**Encryption Algorithm**")
     - keep selected **`256 bits`** as "**Key length**" ("**Encryption Algorithm**")
     - keep selected **`SHA256`** as "**Hash**" ("**Encryption Algorithm**")
     - keep selected **`19 (nist ecp256)`** as "**DH Group**" ("**Encryption Algorithm**")
     - click on "**➕ Add Algorithm**"
     - keep selected **`AES`** as "**Algorithm**" ("**Encryption Algorithm**")
     - keep selected **`256 bits`** as "**Key length**" ("**Encryption Algorithm**")
     - keep selected **`SHA256`** as "**Hash**" ("**Encryption Algorithm**")
     - keep selected **`2 (1024 bit)`** as "**DH Group**" ("**Encryption Algorithm**")
     - focus on section "**Expiration and Replacement**"
     - keep selected **`28800`** as "**Life Time**"
     - keep selected **`25920`** as "**Rekey Time**"
     - keep selected **`0`** as "**Reauth Time**"
     - keep selected **`2880`** as "**Rand Time**"
     - focus on section "**Advanced Options**"
     - keep selected **`Default`** as "**Child SA Close Action**"
     - keep selected **`Auto`** as "**NAT Traversal**"
     - ✏️✓ select **`Enable`** as "**MOBIKE**"
     - keep ~~disabled~~ "**Split connections**"
     - keep ~~disabled~~ "**PRF Selection**"
     - keep selected **`Remote IKE Port`** as "**Custom IKE/NAT-T Ports**" ("UDP port for IKE on the remote gateway. Leave empty for default automatic behavior (500/4500).")
     - keep selected **`Remote NAT-T Port`** as "**Custom IKE/NAT-T Ports**" ("UDP port for NAT-T on the remote gateway.")
     - keep enabled "**Dead Peer Detection**"
     - ✏️✓ type **`30`** in "**Delay**"
     - keep **`5`** in "**Max failures**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**" in message area "**The IPsec tunnel configuration has been changed. ...**"

¹ A heading or trailing space will result in error "**A valid FQDN for 'My identifier' must be specified.**".

- still within menu option "**VPN** > **IPsec**"
     - navigate to tab "**Tunnels**" (should be still here)
     - click on "**⊕ Show Phase 2 Entries (0)**"
     - click on "**➕ Add P2**"
     - automatically navigates to tab "**Mobile Clients**"
     - focus on section "**General Information**"
     - ✏️✓ type **`Tunnel for Mobile Clients (ddvpndd.ddprefixdomaindd.ddbasedomain2dd)`** in "**Description**"
     - keep ~~disabled~~ "**Disabled**"
     - keep selected **`Tunnel IPv4`** for "**Mode**"
     - keep  **`Tunnel for Mobile Clients (ddvpndd.ddprefixdomaindd.ddbasedomain2dd)`** as "**Phase 1**"
     - focus on section "**Networks**"
     - ✏️✓ select **`DMZ subnet`** as "**Local Network**"
     - keep selected **`None`** as "**NAT/BINAT translation**"
     - focus on section "**Phase 2 Proposal (SA/Key Exchange)**"
     - keep selected **`ESP`** as "**Protocol**"
     - ✏️✓ keep enabled "**AES**" with **`256 bits`** ("**Encryption Algorithms**")
     - keep ~~disabled~~ "**AES128-GCM**" with **`128 bits`** ("**Encryption Algorithms**")
     - keep ~~disabled~~ "**AES192-GCM**" with **`Auto`** ("**Encryption Algorithms**")
     - keep enabled "**AES256-GCM**" with **`128 bits`** ("**Encryption Algorithms**")
     - keep ~~disabled~~ "CHACHA20-POLY1305**" ("**Encryption Algorithms**")
     - keep enabled "**SHA256**" ("**Hash Algorithms**")
     - ✏️✓ enabled "**SHA1**", "**SHA384**", "**SHA512**" ("**Hash Algorithms**")
     - keep ~~disabled~~ "**AES-XCBC**" ("**Hash Algorithms**")
     - ✏️✓ select **`off`** as "**PFS key group**"
     - focus on section "**Expiration and Replacement**"
     - keep selected **`3600`** as "**Life Time**"
     - keep selected **`3240`** as "**Rekey Time**"
     - keep selected **`360`** as "**Rand Time**"
     - ✓ click on "**💾 Save**"
     - click on "**✔️ Apply Changes**" in message area "**The IPsec tunnel configuration has been changed. ...**"

#### Stop IPsec

- still within menu option "**VPN** > **IPsec**"
     - navigate to tab "**Tunnels**" (should be still here)
     - ✏️ click on "**Disable**" (to enable) "**Tunnel for Mobile Clients (ddvpndd.ddprefixdomaindd.ddbasedomaindd)**"
     - click on "**✔️ Apply Changes**" in message area "**The IPsec tunnel configuration has been changed. ...**"

#### Start IPsec

- still within menu option "**VPN** > **IPsec**"
     - navigate to tab "**Tunnels**" (should be still here)
     - ✏️ click on "**Enable**" (to disable) "**Tunnel for Mobile Clients (ddvpndd.ddprefixdomaindd.ddbasedomaindd)**"
     - click on "**✔️ Apply Changes**" in message area "**The IPsec tunnel configuration has been changed. ...**"

## Adjust Dashboard

Get to the dashboard and adjust the tiles.

- ⬆ click on the "pfsense" logo int the left top corner
     - click on the "**➕**" above the tiles
     - ✏️  click on "**➕ UPS Status**"

# Issue

## DNBS Resolver: Unable to resolve name

### Syptom

```console
$ nslookup google.com
;; Got SERVFAIL reply from fwlanipv4addressfw
:
```

### Solution

- ⬆ select menu option "**Service** > ** DNS Resolver**"
     - ✏️✓ enable option "**DNS Query Forwarding**" ("Enable Forwarding Mode")
     - ✓ click on "**💾 Save**"
     - ✏️ click on "**Apply Changes**"

## UPS Setup: Driver not connected 

### Syptom

The system was running, when the UPS USB-Cable was plugged in. After configuring the UPS service, data was not aquired.

- ⬆ select menu option "**Status** > **System Logs**"
     - navigate to tab "**System > General**"

```code
timestamp 	upsd 	80423 	Can't connect to UPS [fwupsnamefw] (usbhid-ups-fwupsnamefw): No such file or directory
timestamp 	upsd 	80423 	Found 1 UPS defined in ups.conf
timestamp 	upsd 	80517 	Startup successful
timestamp 	upsmon 	81837 	Startup successful
timestamp 	upsd 	80517 	User local-monitor@::1 logged into UPS [fwupsnamefw]
timestamp 	upsmon 	81876 	Poll UPS [fwupsnamefw] failed - Driver not connected
timestamp 	upsmon 	81876 	Communications with UPS fwupsnamefw lost
timestamp 	kernel 		ugen1.2: <American Power Conversion Back-UPS ...> at usbus1 (disconnected)
timestamp 	kernel 		ugen1.2: <American Power Conversion Back-UPS ...> at usbus1
timestamp 	upsmon 	81876 	Poll UPS [fwupsnamefw] failed - Driver not connected
timestamp 	upsmon 	81876 	UPS fwupsnamefw is unavailable
timestamp 	upsmon 	81876 	Poll UPS [fwupsnamefw] failed - Driver not connected 
```

### Solution

Shutdown the pfSense system, wait 3 seconds and start again.

---

# Based on

A loose collection of information, I found on my endeavor to setup the ACNO server. I am very grateful for all the information, from overall picture over step-by-step instructions to tiny little details, left on the internet by all these marvellous human beings. Not to forget the search engines - Qwant and Google - which allowed me to find these bits and pieces.

- [Administrator: Mobile Clients mit Client Certificate authentifizieren](https://administrator.de/forum/mobile-clients-mit-client-certificate-authentifizieren-5745519444.html#comment-5774865542)
- [CODER'S TOOL: IPv6 Subnet Calculator](https://www.coderstool.com/ipv6-subnet-calculator)
- [Home Netwokrk Guy: How to Redirect Unencrypted DNS Requests to Your Local DNS Resolver in OPNsense](https://homenetworkguy.com/how-to/redirect-all-dns-requests-to-local-dns-resolver/)
- [IT Blog (written by Zeljko Medic): How to setup DMZ on pFSense](https://www.informaticar.net/how-to-setup-dmz-on-pfsense/)
- [Jaspreet Singh: IPv6 Network Ranges Equivalent to IPv4 Private Networks](https://www.jaspreet.net/2024/11/16/2845/ipv6-network-ranges-equivalent-to-ipv4-private-networks/)
- [LisaNet: VPN on demand auf dem Mac und iOS](https://lisanet.de/vpn-on-demand-auf-dem-mac-und-ios/)
- [Luke Davidson: GeoIP on pfSense with pfBlockerNG](https://blog.lukedavidson.org/pfsense/2022/08/23/geoip-on-pfsense-with-pfblockerng.html)
- [Netgate Forum: A brief manual: NUT primary (master) on pfSense with an external NUT client (e.g. Synology)](https://forum.netgate.com/topic/182329/a-brief-manual-nut-primary-master-on-pfsense-with-an-external-nut-client-e-g-synology)
- [Netgate Forum: IOS On Demand VPN](https://forum.netgate.com/topic/181588/ios-on-demand-vpn)
- [Netgate Forum: IPsec with EAP-TLS client cert auth failing [SOLVED]](https://forum.netgate.com/topic/120507/ipsec-with-eap-tls-client-cert-auth-failing-solved/6)
- [Netgate Forum: Using pfSense Software System Patches](https://www.netgate.com/blog/using-pfsense-software-system-patches)
- [reddit: pfSense 2.8.0 CE and Dynamic DNS with Linode API token ](https://www.reddit.com/r/PFSENSE/comments/1l1x7wd/pfsense_280_ce_and_dynamic_dns_with_linode_api/) answer from [brosferatu_](https://www.reddit.com/user/brosferatu_/)
- [RFC6762 - Multicast DNS > Appendix G.  Private DNS Namespaces](https://www.rfc-editor.org/rfc/rfc6762#appendix-G)
- [RFC8375 - Special-Use Domain 'home.arpa.'](https://www.rfc-editor.org/rfc/rfc8375)
- [Telesphoreo's Blog: NUT Server on pfSense with APC UPS](https://blog.telesphoreo.me/2024/11/24/nut-server-on-pfsense-with-apc-ups/)
- [Victor's Blog: How to Setup a USB UPS on pfSense](https://blog.victormendonca.com/2020/10/28/how-to-setup-ups-on-pfsense/)
- [Wikipedia: Reserved IP addresses](https://en.wikipedia.org/wiki/Reserved_IP_addresses)
- [YouTube: Let's Bridge These Ports - OPNsense (Jason's Lab)](https://www.youtube.com/watch?v=q1Rv4gB8fkI)

# Further reading

- [debianforum.de: wirksame DNS-Block-Listen für uBlock Origin, piHole, pfSense](https://debianforum.de/forum/viewtopic.php?t=170376)
- [LINUX INCLUDED: Block Ads & Malvertising on pfSense Using pfBlockerNG (DNSBL)](https://linuxincluded.com/block-ads-malvertising-on-pfsense-using-pfblockerng-dnsbl/)
- [reddit: How to test pfblokerNG is working.](https://www.reddit.com/r/PFSENSE/comments/6wju5e/how_to_test_pfblokerng_is_working/?rdt=40523)
- [zenarmor: pfSense® Software Best Practices Leitfaden](https://www.zenarmor.com/docs/de/netzwerksicherheitstutorials/pfsense-sicherheits-und-hartungs-best-practice-leitfaden)


----

## Host name at login

- ⬆ select menu option "**System** > **General Setup**"
     - focus on section "**webConfigurator**"
     - ✏️✓ enable option "**Login hostname**"
     - ✓ click on "**Save**"


### Issue: There was an error trying to determine the public IP for interface - wan (pppoe0)

Disabling the option "**Disable Gateway Monitoring**" solved my issue.

- ⬆ select menu option "**System** > **Routing**"
     - ⬆ click "**🖉**" ("Edit gateway") of line "**WAN_DHCP**"
          - ✏️✓ enable "**Gateway Monitoring**" ("Disable Gateway Monitoring")
          - ✓🔙 click "**💾 Save**"
     - ⬆ click "**🖉**" ("Edit gateway") of line "**WAN_DHCP6**"
          - ✏️✓ enable "**Gateway Monitoring**" ("Disable Gateway Monitoring")
          - ✓🔙 click "**💾 Save**"
     - click on "**✔️ Apply Changes**"
