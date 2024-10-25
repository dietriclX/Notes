# Hardware

## Unable to boot - black screen with only a blinking cursor

### Hardware

- Fujitsu Futro S920 AMD GX 415GA 4x 1,5GHz
- Intel PRO/1000 ET Ethernet Adapter Quad Port 1GB PCIe x4 Dell 0CWKPJ

### Steps to reproduce

I bought a used S920, with still had the eLux™ RP of the previous owner. A little questionable my decision to buy a Quad Port Network Interface Card (NIC), but that's what I ended up with.

Before I had insert the NIC, I do have verified that the system is working by installating Debian. All good so far.

After I have inserted the NIC and plug in the power cord, a blank screen with only a blinking cursor in the top left corner. Usually, after plug in the power, the POWER LED turns on and OFF after 2-3 seconds. But that has not been the case. the system basically keeped running (stayed power on).

As I bought the hardware from a trustworthy dealer, I was certain that this is not really a hardware issue. So I started playing around with the BIOS setting. Most likely the original settings of the "CSM Configuration" beeing on "Legacy", were causing this issue.

Long story short, at the end I have identified a generic way to solve the issue.

### Solution

#### 1. Preparation of the BIOS (1st Part)

- with the NIC removed, boot into BIOS Setup
- navigate to menu "Boot"
- ✔️✏️ change the configuration to the following

```code
Boot Configuration

Bootup NumLock State            [Off]
Quite Boot                      [Disabled]
Fast On                         [Disabled]
POST Erors                      [Enabled]
Remove Invalid Boot Options     [Enabled]
Prefer USB Boot                 [Enabled]
Boot Removeable Media           [Enabled]
Virus Warning                   [Disaboled]
```

- 🪜 navigate to sub menu "CSM Configuration"
- ✔️✏️ change the configuration to the following

```code
CSM Configuration

Launch CSM                      [Enabled]
Boot option filter              [UEFI only]
Launch PXE OpROM policy         [UEFI only]
Launch Storage OpROM policy     [UEFI only]
Launch Video OpROM policy       [UEFI only]

Other PCI device ROM priority   [UEFI OpROM]
```

- 🛝 navigate back to "Boot" menu
- navigate to menu "Save & Exit"
- ✔️ select option "Exit & Save Changes"
- boot again into BIOS Setup
- navigate to menu "Boot"
- 🪜 navigate to sub menu "CSM Configuration"
- ✔️✏️ ~~disable~~ option "Launch CSM"
- 🛝 navigate back to "Boot" menu
- navigate to menu "Save & Exit"
- ✔️ select option "Exit & Save Changes"
- power off the system, once you see the boot screen
- unplug the power cord
- insert NIC
- plug in the power cord
=> huge different ...

1. after the system got power, the Power LED turned on and OFF after a 2-3 seconds
2. able to boot from external media

#### 2. Install new OS

At this point, I install OPNsense (with GTP for BIOS & UEFI).

Note: After reboot you might experience the system beeing unable to boot, from newly setup internal disk.

#### 3. Preparation of the BIOS (last Part)

- enter BIOS Setup again, with the NIC still installed
- navigate to menu "Boot"
- ✔️✏️ change the configuration to the following

```code
Boot Configuration

Bootup NumLock State            [Off]
Quite Boot                      [Disabled]
Fast On                         [Enabled]
  USB Support                   [Full Initial]
  PS2 Devices Support           [Enabled]
POST Erors                      [Enabled]
Remove Invalid Boot Options     [Enabled]
Prefer USB Boot                 [Disabled]
Boot Removeable Media           [Enabled]
Virus Warning                   [Disaboled]
```

- 🪜 navigate to sub menu "CSM Configuration"
- ✔️✏️ change the configuration to the following

```code
CSM Configuration

Launch CSM                      [Enabled]
Boot option filter              [UEFI & Legacy]
Launch PXE OpROM policy         [UEFI only]
Launch Storage OpROM policy     [UEFI only]
Launch Video OpROM policy       [UEFI only]

Other PCI device ROM priority   [UEFI OpROM]
```

- 🛝 navigate back to "Boot" menu
- navigate to menu "Save & Exit"
- ✔️ select option "Exit & Save Changes"
