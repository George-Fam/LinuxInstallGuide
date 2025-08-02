# Guide d'Installation Linux - FR
### Table des matières
<!-- START doctoc -->
<!-- END doctoc -->
### Préparation
#### Machine virtuelle (VM)
##### Windows
- **Activer la virtualisation dans le BIOS** :
    - Recherchez et activez l'option **Virtualization Technology** (Intel VT-x, AMD-V ou similaire).
- **Activer la virtualisation dans Windows**.  
Consultez cet article de [Microsoft](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1) pour plus de détails.
##### Mac
Les Mac récents prennent en charge la virtualisation par défaut.
- Pour les Mac Intel : la virtualisation est activée automatiquement.
- Pour les Mac Apple Silicon :
    - Vous devez télécharger l'image **ARM** de votre distribution Linux au lieu de l'image **x86 (amd64)** standard.
    - [Image ARM de Debian](https://cdimage.debian.org/debian-cd/current/arm64/iso-cd/)
- Voir la vidéo : [Installation d’une MV pour les Mac Apple Silicon](https://ena01.uqam.ca/pluginfile.php/5473202/course/section/908922/InstallVMUbuntuAvecUTM.mp4) (Mélanie Lord, automne 2024).
- Utilisez [UTM](https://mac.getutm.app/) au lieu de VirtualBox.
#### Dual Boot ou installation unique

- Plutôt que d’installer Linux dans une machine virtuelle, il est généralement préférable de l’installer directement sur votre machine (meilleures performances). Cela dit, l’installation directe est plus complexe et comporte des risques de perte de données. Si vous n’êtes pas à l’aise de suivre les étapes seul, utilisez une machine virtuelle pour l’instant ou assistez à la [Install Party](https://info.uqam.ca/linux/) organisée par le département d'informatique.
1. **Préparation :**
    - Assurez-vous d'avoir au moins 20–30 Go d'espace libre sur votre disque (HDD/SSD).
    - Faites une sauvegarde complète de vos fichiers importants.
2. **Choix de distribution :**
    - **Debian 12** est recommandée pour le cours _INF1070_.
    - Si votre matériel est très récent (ex. portables de jeu), envisagez une distribution **rolling-release** ou **semi-rolling** comme [_Debian Unstable_](https://wiki.debian.org/fr/DebianUnstable), [Arch Linux](https://archlinux.org/), [Manjaro](https://manjaro.org/), [Fedora](https://fedoraproject.org/), etc.
3. **Créer une clé USB bootable** avec [Rufus](https://rufus.ie/en/) ou un outil similaire (ex. Etcher pour macOS/Linux).
    - Guides pour le dual-boot :
        - [Guide Debian (anglais)](https://wiki.debian.org/DualBoot/Windows)
        - [Guide Arch (anglais/français)](https://wiki.archlinux.org/title/Dual_boot_with_Windows_\(Fran%C3%A7ais\))
### Intel RST

- Intel RST (Rapid Storage Technology) est une technologie RAID/cache SSD propriétaire qui pose de **sérieux problèmes de compatibilité avec Linux**. Elle n'est **pas supportée** par les développeurs du noyau et **doit être désactivée**.
- Non supporté à cause de
	- Pas de gestion d'énergie NVMe
	- Pas de support de réinitialisation NVMe
	- Pas de gestion des exceptions NVMe selon l'identifiant PCI
	- Pas de support des fonctions virtuelles SR-IOV
	- Moins bonnes performances (interruptions héritées et partagées)
	- [Source : linux-pci mailing list](https://lore.kernel.org/linux-pci/20190620061038.GA20564@lst.de/T/)
#### Désactivation de RST
1. Désactivez RST dans Intel Optane s'il est installé.
2. Essayez d'installer Ubuntu.
3. Si cela échoue, réactivez AHCI sous Windows (si pris en charge par votre BIOS) :
    1. Ouvrez **Regedit**
    2. Allez dans :  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV\`
    3. Changez `Start` à `0`
    4. Puis :  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\iaStorV\StartOverride`
    5. Changez `0` à `0`
    6. Répétez pour :  `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\storahci\`
    7. Redémarrez et activez **AHCI** dans le BIOS
    8. Si un écran bleu apparaît :
        1. Accédez à la récupération Windows (CMD)
        2. Utilisez `diskpart` pour assigner des lettres à vos partitions
        3. `bcdedit /deletevalue {default} safeboot`
	        - Essayez `{current}` ou sans identifiant si ça échoue)
	    4. Si nécessaire, recréez le BCD avec : (Commandes pour UEFI)
		    1. `cd \d A:\EFI\Microsoft\Boot`
			    - où A est la partition EFI
		    2. Faites un sauvegarde du BCD: `ren BCD BCD.bak`
		    3. `bcdboot C:\Windows /l fr-CA /s A: /f ALL`
			    - Utilized les lettres assignez dans diskpart
4. Relancez l'installation de Linux (Ubuntu ou autre).
### BitLocker
1. Récupérez votre clé de récupération sur [Microsoft](https://account.microsoft.com/devices/recoverykey) – **IMPORTANT**
2. Désactivez BitLocker *(Le déchiffrement peut prendre plusieurs heures selon la taille du disque - à faire à l'avance)*
	- Exécutez dans Powershell comme Admin
		- `Get-BitLockerVolume | foreach { Disable-BitLocker -MountPoint $_.MountPoint }`
### Internet
- Si vous n'avez pas de connexion Ethernet, utilisez l'image complète de Debian DVD/USB ou Ubuntu.
    - Pour Debian, il peut être nécessaire de corriger le fichier *sources.list*
- Wifi
    - PEAP  
    - PAS de certificat CA
    - utilisateur : `codems@ens.uqam.ca`
    - mot de passe : `motdepasse`
### Installation de Debian
1. Sélectionnez "Graphical install" dans le menu de démarrage.
2. Installez Debian en **français** pour faciliter les travaux pratiques.
3. Vous pouvez laisser "debian" comme nom de machine et laisser le domaine vide.
4. **Ne définissez pas de mot de passe root**. Cela garantit que le premier utilisateur aura les droits `sudo`.
5. (**Pour INF1070**) Quand on vous demande le nom complet, entrez votre vrai nom. Comme nom d'utilisateur, vous **devez** utiliser votre [code permanent UQAM](https://etudier.uqam.ca/code-permanent-uqam) en minuscules. **N'oubliez pas votre mot de passe.**
6. Choisissez d'utiliser tout le disque avec une seule partition.
	- Si vous configurez un **dual boot** ou si vous souhaitez un meilleur contrôle sur vos partitions, consultez la section [Partitionnement personnalisé](#partitionnement-personnalis%C3%A9-optionnel--dual-boot) ci-dessous.
7. Lors de la sélection des logiciels, choisissez l'environnement de bureau souhaité (GNOME, KDE Plasma ou Xfce recommandés) ainsi que les utilitaires de base.
### Partitionnement personnalisé (optionnel / dual boot)
Si vous ne souhaitez pas utiliser tout le disque ou si vous installez Debian aux côtés d’un autre système d’exploitation (dual boot), vous pouvez opter pour un partitionnement manuel.
#### Schéma de partition

| Mount Point  | Suggested Size   | File System   | Notes                                           |
| ------------ | ---------------- | ------------- | ----------------------------------------------- |
| `/boot/efi`  | 512MB - 1 GB     | FAT32         |                                                 |
| `swap`       | 1-2 x RAM        | swap          | Utile pour l’hibernation                        |
| `/` (racine) | 20-30 GB Minimum | ext4 or btrfs | Contient tout si /home est omis                 |
| `/home`      | Reste du disque  | ext4 or btrfs | Optionnel, utile pour séparer les données perso |
- Vous pouvez tout mettre dans la partition `/` si vous préférez une configuration plus simple. 
	- Avoir un `/home` séparé est pratique mais **pas obligatoire**. Cependant, cela peut créer des problèmes si la partition `/` vient à manquer d’espace. 
#### Dual Boot
- Redimensionnez vos partitions Windows **depuis Windows** avant d’installer Linux.
- En dual boot, il est **recommandé d’installer Linux sur un disque séparé**.
	- Dans ce cas, vous devez créer une **nouvelle partition EFI** sur ce disque.
	- Si vous utilisez le même disque que Windows **redimensionnez la partition EFI existante à 1 Go** et **montez-la comme `/boot/efi`** lors de l’installation.
##### Guides recommandés
- [Guide Debian pour le dual boot](https://wiki.debian.org/DualBoot/Windows) (anglais)
- [Guide Arch pour le dual boot](https://wiki.archlinux.org/title/Dual_boot_with_Windows_(Fran%C3%A7ais)) (disponible en [anglais](https://wiki.archlinux.org/title/Dual_boot_with_Windows) et en français)
	- Très détaillé et pertinent pour toutes les distributions Linux.
### Pilotes graphiques
- Suivez le [Wiki Debian Nvidia](https://wiki.debian.org/fr/NvidiaGraphicsDrivers#Debian_12_.2BAKs_Bookworm_.2BALs-) pour les instructions d'installation.
### Imprimantes (configuration UQAM)
- Suivez le [guide de Ryan Kavanagh](https://rak.ac/blog/2024-01-17-imprimer-sous-linux-uqam-informatique/) pour imprimer à l'UQAM.
- Infos supplémentaires pour UQAM :
    - Utilisez `esquisse.ens.uqam.ca` au lieu de `Fresque.adm.gst.uqam.ca`
    - Installez `cups` et `smbclient` ou `samba`
    - Téléchargez les [derniers pilotes Kyocera pour Linux](https://www.kyoceradocumentsolutions.us/content/download-center-americas/us/drivers/drivers/KyoceraLinuxPackages_20240521_tar_gz.download.gz)
    - Exemple d'URL d'imprimante : `smb://CODEMS%40ens.uqam.ca:MOT_DE_PASSE_MS@esquisse.ens.uqam.ca/Impression_Mono_Kyocera`
        - Remplacez `Mono` par `Couleur` si vous souhaitez imprimer en couleur    
### Debian root & sudo
- Si vous avez défini un mot de passe pour `root`, votre utilisateur n’aura pas les droits sudo par défaut.
```sh
su -l # Connexion en tant que root

apt update
apt install sudo # Peut être déjà installé

usermod -aG sudo nomUtilisateur
```
### Installation des Additions invité VirtualBox (VM uniquement)
Pour une meilleure intégration VM (résolution, souris, etc.), suivez le [guide Debian](https://wiki.debian.org/VirtualBox#Debian_10_.22Buster.22.2C_Debian_11_.22Bullseye.22.2C_and_Debian_12_.22Bookworm.22-1) :

```sh
# Ajouter Debian Fast-Track
sudo apt install lsb-release
echo "deb http://deb.debian.org/debian $(lsb_release -cs)-backports main contrib" |
sudo tee /etc/apt/sources.list.d/backports.list
sudo apt install fasttrack-archive-keyring
echo "deb http://fasttrack.debian.net/debian-fasttrack/ $(lsb_release -cs)-fasttrack main contrib" |
sudo tee /etc/apt/sources.list.d/fasttrack.list
echo "deb http://fasttrack.debian.net/debian-fasttrack/ $(lsb_release -cs)-backports-staging main contrib" |
sudo tee -a /etc/apt/sources.list.d/fasttrack.list
sudo apt update

# Installer les Additions invité
sudo apt install virtualbox-guest-x11 virtualbox-guest-utils
```
### Erreurs fréquentes
#### Aucun miroir disponible
1. Si vous avez utilisé l'image DVD ou que vous n'avez pas sélectionné de miroir (ex. deb.debian.org), 
	- Vous verrez peut-être une erreur :
	    - `The repository 'cdrom://[Debian GNU/Linux 12.1.0 ...] bookworm Release' does not have a Release file.`    
2. Exécutez `cat /etc/apt/sources.list` — si vous voyez une ligne avec `cdrom://`, modifiez le fichier :
3. Access `sources.list` with nano *(or other text editor)* using  sudo or root. 
	- `sudo nano /etc/apt/sources.list` or 
	- `su -l` and then `nano /etc/apt/sources.list`
4. Remplacez tout le contenu par :

```sh
deb https://deb.debian.org/debian bookworm main non-free-firmware
deb-src https://deb.debian.org/debian bookworm main non-free-firmware

deb https://security.debian.org/debian-security bookworm-security main non-free-firmware
deb-src https://security.debian.org/debian-security bookworm-security main non-free-firmware

deb https://deb.debian.org/debian bookworm-updates main non-free-firmware
deb-src https://deb.debian.org/debian bookworm-updates main non-free-firmware
```

#### Ancienne commande usermod 
- Si `su -c "usermod -aG sudo $USER"` échoue : 
	- utilisez `su -l` puis `usermod -aG sudo nomUtilisateur`
#### Mauvais shell (sh au lieu de bash)
- Si l'invite affiche juste `$` (et pas `nom@machine:~$`), vous êtes dans `sh`.
    - Exécutez : `sudo chsh -s /bin/bash`
    - Si `sudo` ne fonctionne pas : `su -l` puis `chsh -s /bin/bash`    
#### Dépendances Visual C++ VirtualBox
- Si l'installation de VirtualBox sur Windows échoue avec une erreur VC++, installez :
    - `https://aka.ms/vs/17/release/vc_redist.x64.exe`
    - `https://aka.ms/vs/17/release/vc_redist.x86.exe`
#### Mauvais nom d'utilisateur
- Créez un nouvel utilisateur avec GUI ou utilisez :

```sh
sudo useradd -m nomUtilisateur -s /bin/bash
sudo passwd nomUtilisateur
sudo usermod -aG sudo nomUtilisateur
```
#### Problèmes de démarrage (Secure Boot)
- **Bloqué au démarrage après l'installation** : désactivez Secure Boot dans le BIOS si Linux ne démarre pas correctement.
