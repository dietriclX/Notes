OPNsense

# Installation

With the USB Media inserted, press F12 during boot to get the option to choose a boot partition.

```code
[UEFI: Mass Storage Device 1.00]
Mass Storage Device 1. 00
P1: Innodisk DEMSR-08GB mSATAl
Diagnostic Program
```

- ✔️✏️ select "**[UEFI: Mass Storage Device 1.00]**"
- ✔️🪜 press ENTER

OPNsense Live image boots. During the process the network interfaces get identified and automatically assigned to the WAN and LAN roles.

```code

The interfaces will be assigned as follows:

WAN  -> igb1
LAN  -> igb0

```

At the end of the boot process, the login messages appear.

- 🪜 login with installer/opnsense

The installation wizard guides you through the installation steps.

Dialog "Keymap Selection"

- ✔️✏️ enable/select  "**German**' keyboard layout
- ✔️ select "**Continue with de.kbd keymap**"

Dialog "OPNsense 24.7"

```code
Choose one of the following tasks to perform.
Install (UFS)   UFS GTP/UEFI Hybrid
Install (ZFS)   ZFS GTP/UEFI Hybrid
Other Modes >>  Extended Installation
Import Config   Load Configuration
Password Reset  Recover Installation
Force Reboot    Reboot System
```

- 🪜 select "**Other Modes >>**"

Dialog "Select Task"

```code
Choose one of the following tasks to perform.
Auto (UFS)  Guided Disk Setup
Auto (ZFS)  Guided Root-on-ZFS
Manual      Manual Desk Setup (experts)
```

- 🪜 select "**Auto (ZFS)**"

Dialog "ZFS Configuration"

```code
Configure Options:
>>> Install          Proceed with Installation
T Pool Type/Disks:   stripe: 0 disks
- Rescan Devices     *
- Disk Info          *
N Pool Name          zroot
4 Force 4K Sectors?  YES
E Encke yot Disks?   NO
P Partition Scheme   GTP (BIOS+UEFI)
S Swap Size          2GB
M Mirror Swap?       NO
W Encrypt Swap?      NO
```

- 🪜 select "**T Pool Type/Disks:**"

Dialog "ZFS Configuration"

```code
Select Virtual Device type:
stripe  Stripe - No Redundancy
mirror  Mirror - n-Way Mirroring
raid10  RAID 1+0 - n x 2-Way Mirrors
raidz1  RAID-Z1 - Single Redundant RAID
raidz2  RAID-Z2 - Double Redundant RAID
raidz3  RAID-Z3 = Tripte Redundant RAID
```

- 🪜 select "**stripe**"

Dialog "ZFS Configuration"

```code
[ ] ada  Innodisk DEMSR- 08GB mSATA 3ME A

```

- ✔️✏️ enable/select "**ada**"
- ✔️🛝 press ENTER

Dialog "ZFS Configuration"

```code
Configure Options:
>>> Install          Proceed with Installation
T Pool Type/Disks:   stripe: 1 disks
- Rescan Devices     *
- Disk Info          *
N Pool Name          zroot
4 Force 4K Sectors?  YES
E Encke yot Disks?   NO
P Partition Scheme   GTP (BIOS+UEFI)
S Swap Size          2GB
M Mirror Swap?       NO
W Encrypt Swap?      NO
```

- 🪜 select "**S Swap Size**"

Dialog "ZFS Configuration"

```code
Please enter amount of swap space (SI-Unit suffixes recommended; e.g., `2g' for 2 Gigabytes):
2g
```

- ✔️✏️ enter "0"
- ✔️🛝 press ENTER

Dialog "ZFS Configuration"

```code
Configure Options:
>>> Install          Proceed with Installation
T Pool Type/Disks:   stripe: 1 disks
- Rescan Devices     *
- Disk Info          *
N Pool Name          zroot
4 Force 4K Sectors?  YES
E Encke yot Disks?   NO
P Partition Scheme   GTP (BIOS+UEFI)
S Swap Size          0
M Mirror Swap?       NO
W Encrypt Swap?      NO
```

- 🪜 select">>> Install"

Dialog "ZFS Configuration"
Last Chance! Are you sure you want to destroy the current contents of the following disks:
ada0

- ✔️✏️ select "<YES>"
- ✔️ press ENTER

The installation happens now ...

At the end you have the option to change the root password and to finish the installation and reboot the system.

Dialog "Final Configuration"

```code
Root Password Change root password
Complete Install Exit and reboot
```

# Configuration (1st time)

From the boot of the Live Image, I know that the LAN device is `igb0` which is the top RF45 connector on the card. With a PC connected to the new OPNsense box, the system received an IP-Address (checked with `ip a`) and be able to open URL "https://192.168.1.1".

## Hostname & Domain

- navigate to menu "**System** > **Settings** > **General**"
- ✔️✏️ enter host name, into field "**Hostname**"
- ✔️✏️ enter domain (e.g. "private"), into field "**Domain**"
- ✔️ click on "Save"

## RAM Disk (tmpfs)

- navigate to menu "**System** > **Settings** > **Miscellaneous**"
- scroll down to section "Disk / Memory Settings (reboot to apply changes)"
- ✔️✏️ enable option "**/var/log RAM disk**"
- ✔️✏️ enter "2", into field "**/var/log RAM usage**"
- ✔️✏️ enable option "**/tmp RAM disk**"
- ✔️✏️ enter "2", into field "**/tmp RAM usage**"
- ✔️ click on "Save"

## WAN Interface

 navigate to menu "**Interfaces** > **Assignments**"
- ✔️✏️ select entry from listbox, in line of "**[WAN]    wan**"
- ✔️ click on "Save"

## Server in DMZ

In case you have create a kind of DMZ, by running OPNsense behind the DSL Router and with a server (nchostnc with Nextcloud) connected to the DSL Router ... In order to access this server by name, do the following.

- navigate to menu "**Services** > **Unbound DNS** > **Overrides**"
- select tab "Host Overrides"
- 🪜 click on "**+**", under the top list, to add new entry
    - ✔️✏️ enter "nchostnc" in field "**Host**"
    - ✔️✏️ enter "private" in field "**Domain**"
    - ✔️✏️ select "A (IPv4 address)" as "**Type**"
    - ✔️✏️ enter IP-Address of nchostnc in field "**IP address**"
    - 🛝 click on "**Save**"
- in top list, select newly created entry
- focus on section "**Aliases**"
- 🪜 click on "**+**" (section "**Aliases**"), under bottom list, to add new entry
    - ✔️✏️ select "nchostnc.private" as "**Host override**"
    - ✔️✏️ enter "nchostnc" in field "**Host**"
    - 🛝 click on "**Save**"
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
