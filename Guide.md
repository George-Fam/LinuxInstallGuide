# Linux Install Guide - EN

### Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Where to Start](#where-to-start)
- [Prep](#prep)
  - [VM](#vm)
    - [Windows](#windows)
    - [Mac](#mac)
  - [Dual/Single Boot](#dualsingle-boot)
- [Intel RST](#intel-rst)
  - [Disabling RST](#disabling-rst)
- [BitLocker](#bitlocker)
- [Internet](#internet)
- [Debian Installation](#debian-installation)
- [Custom Partitioning (Optional / Dual Boot)](#custom-partitioning-optional--dual-boot)
  - [Partition Scheme](#partition-scheme)
  - [Dual Boot](#dual-boot)
    - [Recommended Guides](#recommended-guides)
- [Graphics Drivers](#graphics-drivers)
- [Printers (UQAM Setup)](#printers-uqam-setup)
- [Debian Root & Sudo](#debian-root--sudo)
- [Installation of Virtualbox Guest Additions (only on VM)](#installation-of-virtualbox-guest-additions-only-on-vm)
- [Common Errors](#common-errors)
  - [No Mirrors Available](#no-mirrors-available)
  - [Old usermod command](#old-usermod-command)
  - [Wrong shell (sh instead of bash)](#wrong-shell-sh-instead-of-bash)
  - [VirtualBox Dependencies (Visual C++)](#virtualbox-dependencies-visual-c)
  - [Wrong Username](#wrong-username)
  - [Troubleshooting Boot Issues (Secure Boot)](#troubleshooting-boot-issues-secure-boot)
- [Further Reading](#further-reading)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### Where to Start

```
What are you setting up?
├── Virtual Machine (VM) ───────────────────────────────────────────────────────┐
│   ├── Windows host -> enable virtualization -> Prep > VM > Windows            │
│   └── Mac host     -> use UTM (Apple Silicon) or VirtualBox -> Prep > VM > Mac│
│                                                                               │
│   Then jump straight to: Debian Installation -> Guest Additions               │
│   (Skip Intel RST, BitLocker, and Custom Partitioning)                        │
│                                                                               │
└── Dual / Single Boot (installing directly on your machine) ───────────────────┘
    ├── Windows machine?
    │   ├── Check Intel RST  (NVMe/Optane users: skip if unsure, come back if install fails)
    │   └── Disable BitLocker before shrinking your drive
    │
    ├── Prep > Dual/Single Boot  ->  Custom Partitioning  ->  Debian Installation
    └── After install: Graphics Drivers (Nvidia users), Printers, Root & Sudo
```

### Prep

#### VM

##### Windows

- **Enable virtualization in your BIOS**:
  - Look for and enable the option **Virtualization Technology** (Intel VT-x, AMD-V, or similar).
- **Enable virtualization in the operating system (Windows)**.
  See this [Microsoft](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1) article for more details.

##### Mac

Recent Macs support virtualization by default.

- For Apple Silicon Macs :
  - You must download the **ARM** image of your preferred Linux distribution instead of the standard **x86 (amd64)** version.
  - [Debian ARM Image](https://cdimage.debian.org/debian-cd/current/arm64/iso-cd/)
- See this video guide: [Installation d’une MV pour les Mac Apple Silicon](https://ena01.uqam.ca/pluginfile.php/5473202/course/section/908922/InstallVMUbuntuAvecUTM.mp4), by Mélanie Lord (Fall 2024).
- Instead of using VirtualBox, use [UTM](https://mac.getutm.app/).

#### Dual/Single Boot

- **This guide assumes a UEFI system**, which is standard on virtually all laptops sold in the last 10+ years. If your machine uses Legacy/CSM boot (older hardware), see the [Arch Wiki BIOS/CSM section](<https://wiki.archlinux.org/title/Installation_guide#Boot_mode_(UEFI_vs._BIOS)>) for differences.
- Instead of installing Linux in a virtual machine, it is generally better to install it directly on your system (better performance overall). However, this is a bit more complex and carries a higher risk of data loss. If you're not comfortable following installation steps on your own, it's recommended to use a virtual machine for now and/or attend the [Install Party](https://info.uqam.ca/linux/) organized by the computer science department for a guided setup.

1. **Preparation:**
   - Make sure your hard drive/SSD has enough free space (at least 20–30 GB recommended for Linux).
   - Perform a full backup of your important files to avoid any accidental data loss.
2. **Distro Choice:**
   - **Debian 13** is recommended for _INF1070_.
   - However, if your machine requires more recent drivers (e.g., gaming laptops or newer hardware), you may want to use a **rolling-release** (more recent but potentially less stable) or **semi-rolling** distribution such as: [_Debian Unstable_](https://wiki.debian.org/fr/DebianUnstable), [Arch Linux](https://archlinux.org/), [Manjaro](https://manjaro.org/), [Fedora](https://fedoraproject.org/), etc.
3. **Create a bootable USB drive** using [Rufus](https://rufus.ie/en/) or a similar tool (e.g., Etcher for macOS/Linux).

### Intel RST

> **Warning:** The following steps involve editing the Windows registry and BIOS settings. If you are not comfortable doing this, bring your laptop to the [Install Party](https://info.uqam.ca/linux/) for guided help.

- Intel RST (Rapid Storage Technology) is a proprietary RAID/SSD caching solution that causes major compatibility issues with Linux. It is **not supported** by kernel developers and **should be disabled**.
  - Not supported due to:
    - No NVMe device power management
    - No NVMe reset support
    - No NVMe quirks based on PCI ID
    - No SR-IOV virtual functions
    - Reduced performance due to shared, legacy interrupts
    - [Source: linux-pci mailing list](https://lore.kernel.org/linux-pci/20190620061038.GA20564@lst.de/T/)

#### Disabling RST

1. Disable RST in intel Optane if installed and seen in Optane
2. Try Installing Linux
3. If step 2 doesn't work, re-configure windows to use AHCI _(If AHCI isn't supported by your BIOS, nothing can be done)_
   1. Open Regedit
   2. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV\`
   3. Change value of Start to 0
   4. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV\StartOverride`
   5. Change value of 0 to 0
   6. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\storahci\` and repeat steps 3-5
   7. Reboot to BIOS and change to AHCI
   8. **If you get a blue screen**, boot into Windows Recovery (CMD) and run the following:
      1. Open `diskpart` and assign letters to your Windows and EFI partitions
      2. Remove the safe boot flag:

         ```
         bcdedit /deletevalue {default} safeboot
         ```

         - If that fails, try `{current}` or omit the identifier entirely

      3. Reboot. If Windows still won't boot, rebuild the BCD (UEFI only):

         ```
         cd \d A:\EFI\Microsoft\Boot
         ren BCD BCD.bak
         bcdboot C:\Windows /l en-us /s A: /f ALL
         ```

         - `A:` = EFI partition (FAT32), `C:` = Windows partition — use the letters you assigned in `diskpart`
         - Replace `en-us` with your language code (e.g. `fr-ca`)
         - If the BCD is at `A:\Boot` instead of `A:\EFI\Microsoft\Boot`, adjust the path accordingly

4. Proceed with Linux install

### BitLocker

1. Get Recovery Key from [Microsoft](https://account.microsoft.com/devices/recoverykey) - **IMPORTANT**
2. Disable Bitlocker _(Decryption may take hours depending on drive size - best done ahead of time)_
   - Run in Powershell as Admin: `Get-BitLockerVolume | foreach { Disable-BitLocker -MountPoint $_.MountPoint }`

### Internet

- If no ethernet connection is available, use the Debian DVD/USB full image
  - For Debian, may need to fix _sources.list_
- Wifi
  - PEAP
  - NO CA Certificate
  - user: `yourusername@ens.uqam.ca` (replace with your UQAM email)
  - pass: `yourpassword` (replace with your UQAM password)

### Debian Installation

1. Select "Graphical install" at the boot menu.
2. Install Debian in **French** to make future course work easier (all course materials and lab instructions are in French).
3. You can leave "debian" as the machine name and leave the domain name blank.
4. **Do not set a password for the root user**. This ensures the first user created will have `sudo` privileges.
5. (**For INF1070**) When prompted for full name, enter your actual name. For the username, **you must** use your UQAM [permanent code](https://etudier.uqam.ca/code-permanent-uqam) in lowercase. **Do not forget your password.**
6. Choose to use the entire disk and put everything in a single partition.
   - If you're setting up a **dual boot** or want more control over your partitions, see the [Custom Partitioning](#custom-partitioning-optional--dual-boot) section below.
7. At the software selection step, choose your desired desktop environment (GNOME, KDE Plasma, or Xfce are recommended) along with base system utilities.

### Custom Partitioning (Optional / Dual Boot)

If you don’t want to use the entire disk or if you’re installing Debian alongside another OS (dual boot), you can opt for manual partitioning.

#### Partition Scheme

| Mount Point | Suggested Size   | File System   | Notes                                     |
| ----------- | ---------------- | ------------- | ----------------------------------------- |
| `/boot/efi` | 512MB - 1 GB     | FAT32         |                                           |
| `swap`      | 1-2 x RAM size   | swap          | Useful for Hibernation                    |
| `/` (root)  | 20-30 GB Minimum | ext4 or btrfs | Contains everything if /home is omitted   |
| `/home`     | Rest of Disk     | ext4 or btrfs | Optional, useful for separating user data |

- You can put everything in `/` if you prefer a simpler setup.
  - Having a separate `/home` is useful but not required. However, it can cause issues if the `/` partition runs out of space.

#### Dual Boot

- Shrink your Windows partition **before** booting the Debian installer using **Windows Disk Management** (`diskmgmt.msc`): right-click your Windows partition -> _Shrink Volume_.
- In a dual boot setup, it’s recommended to install Linux on a separate disk.
  - If you do so, you must create a new EFI partition on that disk.
  - If using the same disk as Windows, **do not create a new EFI partition** — use the existing one. **Resize it to at least 1 GB** beforehand using [GParted](https://gparted.org/) (bootable from a live USB), then **mount it as `/boot/efi`** during installation.
  - Before booting GParted to resize any partition, ensure Windows has released its lock on the NTFS filesystem:
    1. **Disable Fast Startup**: Settings -> System -> Power & Sleep -> Additional power settings -> _Choose what the power buttons do_ -> uncheck _Turn on fast startup_
    2. **Disable Hibernation** (run in PowerShell as Admin): `powercfg /h off`
    3. **Shut down Windows fully** (not restart) before booting GParted — otherwise the NTFS partition will be in a dirty/locked state and GParted may refuse to resize it.

##### Recommended Guides

- [Debian Dual Boot Guide](https://wiki.debian.org/DualBoot/Windows) (English)
- [Arch Dual Boot Guide](https://wiki.archlinux.org/title/Dual_boot_with_Windows) (Available in English and [French](<https://wiki.archlinux.org/title/Dual_boot_with_Windows_(Fran%C3%A7ais)>))
  - Very detailed and relevant for all Linux distributions.

### Graphics Drivers

- See the [Nvidia Debian Wiki](https://wiki.debian.org/fr/NvidiaGraphicsDrivers#Debian_13_.2BAKs_Trixie_.2BALs-) for installation instructions.

### Printers (UQAM Setup)

- Follow [Ryan Kavanagh's guide](https://rak.ac/blog/2024-01-17-imprimer-sous-linux-uqam-informatique/) for printing on Linux at UQAM.
- Additional info for UQAM
  - Use `esquisse.ens.uqam.ca` instead of `Fresque.adm.gst.uqam.ca`.
  - Install `cups` and `smbclient` or `samba`.
  - Enable CUPS: `sudo systemctl enable --now cups`
  - Use the [latest Kyocera Linux drivers](https://www.kyoceradocumentsolutions.us/content/download-center-americas/us/drivers/drivers/KyoceraLinuxPackages_20240521_tar_gz.download.gz).
- Example Printer URL: `smb://CODEMS%40ens.uqam.ca:YOUR_MS_PASSWORD@esquisse.ens.uqam.ca/Impression_Mono_Kyocera`
  - Replace `YOUR_MS_PASSWORD` with your Microsoft/AD password
  - Change `Mono` to `Couleur` if you want color printing

### Debian Root & Sudo

- If you did set a password for `root`, your regular user won’t have sudo rights by default.

```sh
su -l # Log in as root

apt update
apt install sudo # May already be installed

usermod -aG sudo username
```

### Installation of Virtualbox Guest Additions (only on VM)

To enhance VM integration (better resolution, mouse integration, etc.), install the VirtualBox Guest Additions.

1. In the VirtualBox menu (VM window):
   - Peripherals -> Insert Guest Additions CD image...
   - The CD will be automatically mounted in Debian.
2. Autorun will be proposed
   - Click Run and enter your password when prompted

### Common Errors

#### No Mirrors Available

1. If you used the DVD image or didn’t select a mirror like deb.debian.org.
   - You may see an error like:
     - `The repository 'cdrom://[Debian GNU/Linux 13.0.0 ...] trixie Release' does not have a Release file.`
2. Erase `sources.list` as sudo or root:
   - `sudo rm /etc/apt/sources.list` or
   - `su -l` and then `rm /etc/apt/sources.list`
3. Make a new file `/etc/apt/sources.list.d/debian.sources` :

```sh
Types: deb deb-src
URIs: https://deb.debian.org/debian
Suites: trixie trixie-updates
Components: main contrib non-free non-free-firmware
Enabled: yes
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb deb-src
URIs: https://security.debian.org/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Enabled: yes
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

4. Update your list of packages: `sudo apt update`

#### Old usermod command

- If `su -c "usermod -aG sudo $USER"` returns an error:
  - Use `su -l` and then `usermod -aG sudo username`

#### Wrong shell (sh instead of bash)

- If your prompt only shows a dollar sign (`$`) and not `username@machine:~$`, you’re using `sh`.
  - Run : `sudo chsh -s /bin/bash`
  - If `sudo` doesn't work: `su -l` and then `chsh -s /bin/bash`

#### VirtualBox Dependencies (Visual C++)

- If installing VirtualBox on Windows fails with a VC++ error, install:
  - `https://aka.ms/vs/17/release/vc_redist.x64.exe`
  - `https://aka.ms/vs/17/release/vc_redist.x86.exe`

#### Wrong Username

- Either create a new user from the GUI and add them to the `sudo` group, or:

```sh
sudo useradd -m username -s /bin/bash
sudo passwd username
sudo usermod -aG sudo username
```

#### Troubleshooting Boot Issues (Secure Boot)

- **Stuck at boot after install**: Try disabling Secure Boot in BIOS if your Linux doesn’t boot properly after installation.

### Further Reading

- **[The Linux Desktop Guide](https://thelinuxbook.com)** by Chris Titus.
  - A practical guide covering desktop Linux topics that most installation guides don’t touch: terminal usage, networking, audio, bluetooth, drives, gaming, environment setup, and more. Not required reading, but a great resource if you want to go beyond installation and really learn how to use your Linux desktop day-to-day.
  - Not overly detailed; the [Arch Wiki](https://wiki.archlinux.org/) remains your best friend for in-depth references.
