# Linux Install Guide - EN

### Table of Contents
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
- [Installation of VirtualBox Guest Additions (Only on VM)](#installation-of-virtualbox-guest-additions-only-on-vm)
- [Common Errors](#common-errors)
	- [No Mirrors Available](#no-mirrors-available)
	- [Wrong Shell (sh instead of bash)](#wrong-shell-sh-instead-of-bash)
	- [VirtualBox Dependencies (Visual C++)](#virtualbox-dependencies-visual-c)
	- [Wrong Username](#wrong-username)
	- [Troubleshooting Boot Issues (Secure Boot)](#troubleshooting-boot-issues-secure-boot)
### Prep
#### VM
##### Windows
- **Enable virtualization in your BIOS**:
	- Look for and enable the option **Virtualization Technology** (Intel VT-x, AMD-V, or similar).
- **Enable virtualization in the operating system (Windows)**.
See this [Microsoft](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1) article for more details.
##### Mac
Recent Macs support virtualization by default.
- For Intel Macs : La virtualisation est activée automatiquement.
- For Apple Silicon Macs :
   - You must download the **ARM** image of your preferred Linux distribution instead of the standard **x86 (amd64)** version.
   - [Debian ARM Image](https://cdimage.debian.org/debian-cd/current/arm64/iso-cd/)
- See this video guide: [Installation d’une MV pour les Mac Apple Silicon](https://ena01.uqam.ca/pluginfile.php/5473202/course/section/908922/InstallVMUbuntuAvecUTM.mp4), by Mélanie Lord (Fall 2024).
- Instead of using VirtualBox, use [UTM](https://mac.getutm.app/).
#### Dual/Single Boot
- Instead of installing Linux in a virtual machine, it is generally better to install it directly on your system (better performance overall). However, this is a bit more complex and carries a higher risk of data loss.  If you're not comfortable following installation steps on your own, it's recommended to use a virtual machine for now and/or attend the [Install Party](https://info.uqam.ca/linux/) organized by the computer science department for a guided setup.
1. Préparation :
	- Make sure your hard drive/SSD has enough free space (at least 20–30 GB recommended for Linux).
	-  Perform a full backup of your important files to avoid any accidental data loss.
2. Distro Choice:
   - **Debian 12** is recommended for *INF1070*.
   - However, if your machine requires more recent drivers (e.g., gaming laptops or newer hardware), you may want to use a **rolling-release** (more recent but potentially less stable) or **semi-rolling** distribution such as: [_Debian Unstable_](https://wiki.debian.org/fr/DebianUnstable), [Arch Linux](https://archlinux.org/), [Manjaro](https://manjaro.org/), [Fedora](https://fedoraproject.org/), etc.
1. **Create a bootable USB drive** using [Rufus](https://rufus.ie/en/) or a similar tool (e.g., Etcher for macOS/Linux).
### Intel RST
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
3. If step 2 doesn't work, re-configure windows to use AHCI *(If AHCI isn't supported by your BIOS, nothing can be done)*
	1. Open Regedit
	2. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV\`
	3. Change value of Start to 0
	4. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV\StartOverride`
	5. Change value of 0 to 0
	6. Navigate to `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\storahci\` and repeat steps 3-5
	7. Reboot to BIOS and change to AHCI
	8. If blue screen, navigate to Windows Recovery, CMD
		1. Find Windows partition and boot partition and assign them a letter in diskpart
		2. `bcdedit /deletevalue {default} safeboot`
			1. If this doesn't work `bcdedit /deletevalue {current} safeboot` or `bcdedit /deletevalue safeboot`
		3. Try rebooting. if it doesn't work, proceed to step 4
		4. Recreate bcd file with `Bcdboot`
			1. For UEFI
				1. `cd \d A:\EFI\Microsoft\Boot`
					-  A is letter I assigned to EFI partition (fat32)
					-  this can vary BCD file could also be in `A:\Boot`
				2. Backup BCD: `ren BCD BCD.bak`
				3. `bcdboot c:\windows /l en-us /s a: /f ALL`
					- Change lang with corresponding language. e.g, `fr-ca`
					- use drive letters assigned in diskpart (c ->main windows partition and a-> efi - fat32 partition)
4. Proceed with Ubuntu Install
### BitLocker
1. Get Recovery Key from [Microsoft](https://account.microsoft.com/devices/recoverykey) - **IMPORTANT**
2. Disable Bitlocker *(Decryption may take hours depending on drive size - best done ahead of time)*
	- Run in Powershell as Admin: `Get-BitLockerVolume | foreach { Disable-BitLocker -MountPoint $_.MountPoint }`
### Internet
- If no ethernet connection is available, use Debian DVD/usb full image or Ubuntu 
	- For Debian, may need to fix *sources.list*
- Wifi
	- PEAP
	- NO CA Certificate
	- user: `codems@ens.uqam.ca`
	- pass: `motdepasse`
### Debian Installation
1. Select "Graphical install" at the boot menu.
2. Install Debian in **French** to make future course work easier.
3. You can leave "debian" as the machine name and leave the domain name blank.
4. **Do not set a password for the root user**.  This ensures the first user created will have `sudo` privileges.
5. (**For INF1070**) When prompted for full name, enter your actual name.  For the username, **you must** use your UQAM [permanent code](https://etudier.uqam.ca/code-permanent-uqam) in lowercase.  **Do not forget your password.**
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
- Resize your Windows partitions from Windows before installing Linux.
- In a dual boot setup, it’s recommended to install Linux on a separate disk. 
	- If you do so, you must create a new EFI partition on that disk.
	- If using the same disk as Windows, **resize the existing EFI partition to 1 GB** and **mount it as `/boot/efi`** during installation.
##### Recommended Guides
- [Debian Dual Boot Guide](https://wiki.debian.org/DualBoot/Windows) (English)
- [Arch Dual Boot Guide](https://wiki.archlinux.org/title/Dual_boot_with_Windows) (Available in English and [French](https://wiki.archlinux.org/title/Dual_boot_with_Windows_(Fran%C3%A7ais)))  
	- Very detailed and relevant for all Linux distributions.
### Graphics Drivers
- See the [Nvidia Debian Wiki](https://wiki.debian.org/fr/NvidiaGraphicsDrivers#Debian_12_.2BAKs_Bookworm_.2BALs-) for installation instructions.
### Printers (UQAM Setup)
- Follow [Ryan Kavanagh's guide](https://rak.ac/blog/2024-01-17-imprimer-sous-linux-uqam-informatique/) for printing on Linux at UQAM.  
- Additional info for UQAM
	- Use `esquisse.ens.uqam.ca` instead of `Fresque.adm.gst.uqam.ca`.
	- Install `cups` and `smbclient` or `samba`.
	- Use the [latest Kyocera Linux drivers](https://www.kyoceradocumentsolutions.us/content/download-center-americas/us/drivers/drivers/KyoceraLinuxPackages_20240521_tar_gz.download.gz).
	- Example Printer URL `smb://CODEMS%40ens.uqam.ca:MOT_DE_PASSE_MS@esquisse.ens.uqam.ca/Impression_Mono_Kyocera`
		- Change `Mono` to `Couleur` if you want color printing
### Debian Root & Sudo
- If you did set a password for `root`, your regular user won’t have sudo rights by default.
```sh
su -l # Log in as root

apt update
apt install sudo # May already be installed

usermod -aG sudo nomUtilisateur
```
### Installation of Virtualbox Guest Additions (only on VM)
To enhance VM integration (better resolution, mouse integration, etc.), follow [Debian’s guide](https://wiki.debian.org/VirtualBox#Debian_10_.22Buster.22.2C_Debian_11_.22Bullseye.22.2C_and_Debian_12_.22Bookworm.22-1):

```sh
# Add Debian Fast-Track
sudo apt install lsb-release
echo "deb http://deb.debian.org/debian $(lsb_release -cs)-backports main contrib" |
sudo tee /etc/apt/sources.list.d/backports.list
sudo apt install fasttrack-archive-keyring
echo "deb http://fasttrack.debian.net/debian-fasttrack/ $(lsb_release -cs)-fasttrack main contrib" |
sudo tee /etc/apt/sources.list.d/fasttrack.list
echo "deb http://fasttrack.debian.net/debian-fasttrack/ $(lsb_release -cs)-backports-staging main contrib" |
sudo tee -a /etc/apt/sources.list.d/fasttrack.list
sudo apt update

# Install Guest Additions
sudo apt install virtualbox-guest-x11 virtualbox-guest-utils
```
### Common Errors
#### No Mirrors Available
1. If you used the DVD image or didn’t select a mirror like deb.debian.org: .
   - You may see an error like:
      - `The repository 'cdrom://[Debian GNU/Linux 12.1.0 ...] bookworm Release' does not have a Release file.`
2. Run `cat /etc/apt/sources.list`. If you see `"cdrom://[Debian GNU/...`, :
3. Access `sources.list` with nano *(or other text editor)* using  sudo or root. 
	   - `sudo nano /etc/apt/sources.list` or 
	   - `su -l` and then `nano /etc/apt/sources.list`
4. Replace the entire file with: :

   ```sh
   deb https://deb.debian.org/debian bookworm main non-free-firmware
   deb-src https://deb.debian.org/debian bookworm main non-free-firmware

   deb https://security.debian.org/debian-security bookworm-security main non-free-firmware
   deb-src https://security.debian.org/debian-security bookworm-security main non-free-firmware

   deb https://deb.debian.org/debian bookworm-updates main non-free-firmware
   deb-src https://deb.debian.org/debian bookworm-updates main non-free-firmware
   ```
#### Old usermod command
-  If `su -c "usermod -aG sudo $USER"` returns an error:
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
- **Stuck at boot after install**:  Try disabling Secure Boot in BIOS if your Linux doesn’t boot properly after installation.