# SAÉ303 - Concevoir un réseau informatique adapté au multimédia

Cette SAÉ a été réalisée dans le cadre de notre deuxième année de BUT Réseaux et Télécommunications, parcours Réseaux Opérateurs Multimédia, au sein de l’IUT de Villetaneuse.

## Introduction

Rédiger une introduction

## Méthodologie de la SAÉ

Pour faire des machines virtuelles nous utiliserons **[VMWare Workstation Pro](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)** qui est désormais **gratuit** pour les **particuliers** **et** **étudiants**. Celui-ci **installe** **automatiquement** les **outils additionnels invités** sur les machines virtuelles, **simplifiant** l’usage du **presse-papier** ou bien encore le **redimensionnement automatique** de l’**écran**. Il vous est possible de **télécharger les installateurs**, **Linux** ou **Windows** ici -> [sur le CDN de VMWare](https://softwareupdate.vmware.com/cds/vmw-desktop/ws/).

Pour **passer des appels téléphoniques**, nous utiliserons le **softphone [Linphone](https://www.linphone.org/home/)**. Il existe également le client [MicroSIP](https://www.microsip.org/) exclusif à Windows.

**Pour relier les téléphones physiques** Yealink T42U, il est **nécessaire** de **bien configurer** les **interfaces réseau** de nos **machines virtuelles** sur VMWare.
Voici un exemple de réseau recommandé :

![vmware_XgMoz08DAt](img/vmware_XgMoz08DAt.png)

Les **machines virtuelles** **doivent avoir 2 interfaces réseau**, **une interface WAN** **gérée automatiquement par VMWare** qui leur fourniras un **accès** constant **à internet**, **et une interface LAN**, **reliée** à un **port Ethernet physique** de l’ordinateur, **permettant** de **connecter** **un switch** puis des **téléphones** à notre réseau virtuel.

**Sur VMWare**, voici la **configuration d’une machine virtuelle** :

![image-20250315115222963](img/image-20250315115222963.png)

La **première interface doit être** celle **NAT**, **la seconde doit être Bridged**, **dans debian** elles **apparaîtront** respectivement **comme** **ens33** et **ens36**.

Pour être certain que le Bridged renvoie vers le bon port Ethernet physique, il faut aller dans le Virtual Network Editor

![image-20250315115954513](img/image-20250315115954513.png)

et sélectionner la bonne interface

![image-20250315120034289](img/image-20250315120034289.png)

Une fois ceci fait nous avons un environnement prêt pour la SAE.
