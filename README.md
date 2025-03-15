# SAÉ303 - Concevoir un réseau informatique adapté au multimédia

Cette SAÉ a été réalisée dans le cadre de notre deuxième année de BUT Réseaux et Télécommunications, parcours Réseaux Opérateurs Multimédia, au sein de l’IUT de Villetaneuse.

## Introduction

Rédiger une introduction

## 0 - Pré-requis

### Logiciels nécessaires

- **VMWare Workstation Pro** : Pour créer et gérer les machines virtuelles. VMWare est désormais gratuit pour les particuliers et étudiants. Il intègre automatiquement les outils invités permettant le redimensionnement automatique de l'écran et la gestion simplifiée du presse-papier. Les installateurs pour Linux ou Windows sont disponibles sur le [CDN de VMWare](https://softwareupdate.vmware.com/cds/vmw-desktop/ws/).
- **Linphone** : Logiciel utilisé pour passer des appels téléphoniques via le réseau virtuel. Linphone est multiplateforme. Alternativement, **MicroSIP** est une option exclusivement pour Windows.

### Configuration réseau des machines virtuelles

Pour connecter correctement les téléphones physiques Yealink T42U à notre réseau virtuel, il est impératif de configurer soigneusement les interfaces réseau de nos machines virtuelles sous VMWare.

Chaque machine virtuelle doit disposer de deux interfaces réseau :

- Une interface WAN (NAT) gérée automatiquement par VMWare, permettant un accès constant à Internet.
- Une interface LAN (Bridged) reliée directement à un port Ethernet physique de l'ordinateur. Cette dernière interface permet la connexion d'un switch réseau, auquel seront reliés les téléphones physiques.

![vmware_XgMoz08DAt](img/vmware_XgMoz08DAt.png)

Configuration dans VMWare :

![image-20250315115222963](img/image-20250315115222963.png)

- Interface réseau **1** : NAT
- Interface réseau **2** : Bridged

Sous Debian, ces interfaces apparaîtront respectivement comme **ens33** et **ens34**.

Pour garantir que l'interface Bridged pointe vers le bon port Ethernet physique, accéder à Virtual Network Editor :

![image-20250315115954513](img/image-20250315115954513.png)

et sélectionner la bonne interface réseau physique dans la configuration Bridged :

![image-20250315120034289](img/image-20250315120034289.png)

## I - FreePBX

#### Étape 1 : Création de la machine virtuelle

- Télécharger l’ISO de Debian 12 sur le [site officiel Debian](https://www.debian.org/)
- Créer une nouvelle machine virtuelle sur VMWare avec les caractéristiques suivantes :
  - CPU : 2 cœur
  - Mémoire vive : 2 Go
  - Disque dur : 16 Go

![image-20250315134613384](img/image-20250315134613384.png)

![image-20250315134622907](img/image-20250315134622907.png)

![image-20250315134632755](img/image-20250315134632755.png)

![image-20250315134640002](img/image-20250315134640002.png)

![image-20250315134648226](img/image-20250315134648226.png)

![image-20250315134709773](img/image-20250315134709773.png)

![image-20250315134814311](img/image-20250315134814311.png)

![image-20250315134905723](img/image-20250315134905723.png)

#### Étape 2 : Installation de Debian 12

* Démarrer la machine virtuelle et installer Debian
* Sélectionner **ens33 (NAT)** comme interface principale.

![image-20250315135052350](img/image-20250315135052350.png)

* Attribuer un nom d'hôte clair pour identifier facilement la machine (ex : **sae-freepbx**)

![image-20250315135144491](img/image-20250315135144491.png)

* Ne pas définir de mot de passe root ; créer un utilisateur disposant des permissions **sudo**.

![image-20250315135239614](img/image-20250315135239614.png)

* Sélectionner **XFCE** comme environnement de bureau (plus léger) et activer le **serveur SSH** pour faciliter l'administration à distance.

![image-20250315135552894](img/image-20250315135552894.png)

#### Étape 3 : Configuration des interfaces réseau

![image-20250315151940455](img/image-20250315151940455.png)

Une fois Debian installé, les interfaces réseau ne possèdent pas encore d’adresses IP définies. Pour corriger cela, modifier le fichier de configuration réseau :

```bash
sudo nano /etc/network/interfaces
```

Configurer comme suit :

```bash
auto ens33
iface ens33 inet dhcp

auto ens34
iface ens34 inet static
        address 192.168.1.1/24
```

* Redémarrer la machine virtuelle pour appliquer les changements.
* Vérifier avec la commande `ip a` que les adresses IP sont correctement attribuées.
* Confirmer l'accès à Internet en exécutant un test avec la commande `ping 1.1.1.1`

![image-20250315140922264](img/image-20250315140922264.png)

#### Étape 4 : Installation de FreePBX

L’installation de FreePBX sur Debian s’effectue à l’aide du script officiel disponible sur le [GitHub de FreePBX](https://github.com/FreePBX/sng_freepbx_debian_install).

Dans un terminal, exécuter les commandes suivantes en tant qu'utilisateur :

```
sudo su -
wget https://github.com/FreePBX/sng_freepbx_debian_install/raw/master/sng_freepbx_debian_install.sh -O /tmp/sng_freepbx_debian_install.sh
bash /tmp/sng_freepbx_debian_install.sh
```

Ce script lance automatiquement l'installation complète de FreePBX avec tous ses modules nécessaires. Cette étape peut prendre du temps, il faut attendre la fin de l'installation sans interruption.

![image-20250315141143324](img/image-20250315141143324.png)

#### Étape 5 : Configuration de FreePBX

![image-20250315143803476](img/image-20250315143803476.png)

* Ouvrir un navigateur web et aller sur http://192.168.1.1/
* Définir un nom d’utilisateur et mot de passe administrateur

![image-20250315144016918](img/image-20250315144016918.png)

* Se connecter sur FreePBX Administration

![image-20250315144135833](img/image-20250315144135833.png)

* Ignorer les offres commerciales

![image-20250315144229769](img/image-20250315144229769.png)

![image-20250315144252084](img/image-20250315144252084.png)

![image-20250315144311796](img/image-20250315144311796.png)

![image-20250315144332250](img/image-20250315144332250.png)

![image-20250315144350680](img/image-20250315144350680.png)

![image-20250315144415601](img/image-20250315144415601.png)

* Définir la langue des messages audios et d’affichage

![image-20250315144530686](img/image-20250315144530686.png)

* Laisser les paramètres par défaut du Sangoma Smart Firewall

![image-20250315144624283](img/image-20250315144624283.png)

![image-20250315144655398](img/image-20250315144655398.png)

![image-20250315144718684](img/image-20250315144718684.png)

![image-20250315144755138](img/image-20250315144755138.png)

![image-20250315144817525](img/image-20250315144817525.png)

![image-20250315144851212](img/image-20250315144851212.png)

![image-20250315144911512](img/image-20250315144911512.png)

* Cliquer, si proposé, sur Appliquer la Configuration. FreePBX est maintenant prêt à l’emploi !

![image-20250315145408133](img/image-20250315145408133.png)

#### Étape 5 : Création des comptes SIP

* Aller dans “Connectivité > Postes”

![image-20250315152623215](img/image-20250315152623215.png)

* Cliquer sur “Ajouter un poste” puis choisir “Ajout nouveau poste SIP”

![image-20250315153348110](img/image-20250315153348110.png)

* Créer un poste SIP : Choisir une extension utilisateur (numéro) et définir le secret (mot de passe)

![image-20250315155008665](img/image-20250315155008665.png)

* Créer un second poste SIP
* Appliquer la configuration

![image-20250315155456600](img/image-20250315155456600.png)

#### Étape 6.1 : Connecter un client SIP Softphone

* Installer une seconde machine Debian avec les mêmes étapes que précédemment (configuration des cartes réseaux)

```
sudo nano /etc/network/interfaces
```

Configurer comme suit :

```
auto ens33
iface ens33 inet dhcp

auto ens34
iface ens34 inet static
        address 192.168.1.10/24
```

* Installer Liphone

```
sudo apt update
sudo apt install linphone -y
```

* Démarrer Linphone

![image-20250315163004388](img/image-20250315163004388.png)

* Aller sur “Utiliser un compte SIP”

![image-20250315163059002](img/image-20250315163059002.png)

![image-20250315163121066](img/image-20250315163121066.png)

* Renseigner les informations du compte SIP créé précédemment

![image-20250315163216375](img/image-20250315163216375.png)

* Une fois Linphone connecté, l’icône devrait passer au vert

![image-20250315163711564](img/image-20250315163711564.png)

* Appeler ```*97``` et vérifier si le son fonctionne

![image-20250315163850005](img/image-20250315163850005.png)