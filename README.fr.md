# Guide d'installation Linux à l'UQAM

[![en](https://img.shields.io/badge/lang-en-blue.svg)](https://github.com/George-Fam/LinuxInstallGuide/blob/master/README.md) [![fr](https://img.shields.io/badge/lang-fr-brightgreen.svg)](https://github.com/George-Fam/LinuxInstallGuide/blob/master/README.fr.md)

Ce guide a été créé pour soutenir le **Party d’installation Linux**, organisé chaque session afin d’aider les nouveaux étudiants à installer Linux - que ce soit dans une machine virtuelle ou en dual boot.
## Contenu

- Instructions détaillées pour l'installation de Linux
- Configuration de machines virtuelles (Windows/macOS)
- Instructions pour le dual boot et le partitionnement
- Dépannage des erreurs d'installation fréquentes
- Configuration de Secure Boot, BitLocker et Intel RST
- Connexion au réseau (Wi-Fi UQAM)
- Configuration des imprimantes pour les systèmes UQAM
- Recommandations spécifiques aux étudiants (ex. : Debian 12 pour INF1070)
- Configuration avancée (Additions invité VirtualBox, correctifs shell, etc.)

## Guides

| Language | Link                              |
| -------- | --------------------------------- |
| Anglais  | [`Guide (EN)`](./Guide%20(EN).md) |
| Français | [`Guide (FR)`](./Guide%20(FR).md) |

## Avant de commencer

Assurez-vous de :
- Sauvegarder vos données importantes
- Avoir au moins **20 à 30 Go d’espace libre**
- Apporter votre **code permanent UQAM** et vos **identifiants Wi-Fi** si vous êtes sur le campus
- **Désactiver BitLocker** sur Windows si activé — c’est nécessaire pour les installations en dual boot.  *(Voir les instructions dans le [Guide](./Guide%20(FR).md#bitlocker).)*
  
En cas de doute, participez au [Party d’installation Linux](https://info.uqam.ca/linux/) pour obtenir de l’aide sur place!

## Contributions & Remerciements

Merci particulièrement à :
- Mélanie Lord pour la vidéo sur la virtualisation avec Mac
- [Ryan Kavanagh](https://rak.ac) pour le guide de configuration des imprimantes UQAM et les étapes d'installation de Debian