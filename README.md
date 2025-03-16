# SAÉ303 - Concevoir un réseau informatique adapté au multimédia


Rédigé par Tristan BRINGUIER et Jack CORRÊA DO CARMO. Cette SAÉ a été réalisée dans le cadre de notre deuxième année de BUT Réseaux et Télécommunications parcours Réseaux Opérateurs Multimédia à l’IUT de Villetaneuse. Le sujet initial de cette SAÉ proviens de nos professeurs Mr. Mohamed Amine Ouamri et Mme. Yamina Amzal.

## Objectif

L’objectif de cette SAÉ est 

## 0 - Pré-requis

> [!IMPORTANT]  
> Pour assurer le bon fonctionnement de cette SAE, il est nécessaire d’installer et configurer des logiciel de manière spécifique.


- **VMWare Workstation Pro** : Pour créer et gérer les machines virtuelles. VMWare est désormais gratuit pour les particuliers et étudiants. Il intègre automatiquement les outils invités permettant le redimensionnement automatique de l'écran et la gestion simplifiée du presse-papier.

### Installer VMWare Workstation Pro sous Linux (Debian)

* Installer les paquets suivants (ce sont des dépendances à VMWare)

```shell
sudo apt update
sudo apt install build-essential linux-headers-$(uname -r) -y
```

* Aller sur [le CDN de VMWare](https://softwareupdate.vmware.com/cds/vmw-desktop/ws/) et sélectionner la dernière version disponible, puis linux, puis core, et télécharger l’archive .tar
* Décompresser l’archive .tar (remplacer le nom du fichier par celui téléchargé)

```shell
tar --extract -f VMware-Workstation-17.6.3-24583834.x86_64.bundle.tar
```

* Exécuter en tant qu’administrateur l’installateur présent dans l’archive

```shell
chmod +x VMware-Workstation-17.6.3-24583834.x86_64.bundle
sudo ./VMware-Workstation-17.6.3-24583834.x86_64.bundle
```

* Démarrer VMWare en ligne de commandes

```shell
vmware
```

![image-20250316014946420](img/image-20250316014946420.png)
![image-20250316015008802](img/image-20250316015008802.png)
![image-20250316015028985](img/image-20250316015028985.png)
![image-20250316015045048](img/image-20250316015045048.png)

* Fermer VMWare et redémarrer le linux pour finaliser l’installation

```shell
sudo reboot
```

> [!NOTE]  
> Après redémarrage du linux, VMWare Workstation est installé et prêt à l’emploi !

### Installer VMWare Workstation Pro sous Windows

* Aller sur [le CDN de VMWare](https://softwareupdate.vmware.com/cds/vmw-desktop/ws/) et sélectionner la dernière version disponible, puis windows, puis core, et télécharger l’archive .tar
* Décompresser l’archive .tar et installer VMWare via l’installateur présent dans l’archive

> [!NOTE]  
> VMWare Workstation est installé et prêt à l’emploi !

### Configurer le réseau virtuel VMWare

Pour interconnecter avec aisance les téléphones physiques Yealink au réseau virtualisé, il est impératif de configurer soigneusement les interfaces réseau des machines virtuelles sous VMWare.

Chaque machine virtuelle doit disposer de deux interfaces réseau :

* Une interface “WAN” (NAT) gérée automatiquement par VMWare. Cette interface permet un accès simple à et constant à Internet
* Une interface “LAN” (Bridgée) reliée à un port Ethernet physique de l’ordinateur. Cette interface permet un accès physique aux équipements branchés en Ethernet à l’ordinateur. (ex : Switch Cisco, Téléphone Yealink, etc.)

ILLUSTRATION RÉSEAU GNS3

> [!IMPORTANT]  
> Il est primordial de vérifier que l’interface Bridgée renvoie vers la bonne interface physique !

* Ouvrir le Virtual Network Editor
![image-20250316020245859](img/image-20250316020245859.png)

- Sélectionner vmnet0 (qui a pour type bridged) et choisir la bonne interface Ethernet
![image-20250316020321262](img/image-20250316020321262.png)

Les machines virtuelles devront être connectées de la sorte :
![image-20250315115222963](img/image-20250315115222963.png)

- Network Adapter (Interface réseau 1) : NAT
- Network Adapter 2 (Interface réseau 2) : Bridgée

Sous Debian, les interfaces réseaux apparaîtront respectivement comme ens33 et ens34.

## 1 - VoIP / SIP avec FreePBX (Asterisk)

FreePBX fournis une interface d’administration web simple et intuitive, tout en reposant sur la robustesse du moteur de téléphonie Asterisk. FreePBX est né en 2004 sous le nom d’Asterisk Management Portal, dans le but de simplifier l’administration d’Asterisk. Au fil des années, rebaptisé en FreePBX, il est devenu l’une des solutions de référence pour la gestion de la téléphonie IP dans l’écosystème open source.

En plus de reposer sur une large communauté et de proposer de nombreuses fonctionnalités avancées (gestion des extensions, lignes SIP, files d’attente d’appels, conférences, etc.), FreePBX a longtemps été distribué sous forme d’une distribution Linux indépendante basée sur RHEL (Red Hat Enterprise Linux) — une solution plus robuste pour un usage d’entreprise mais plus complexe à installer. Depuis la version 17, FreePBX est désormais porté sur Debian ce qui facilite grandement l’installation et la maintenance.

#### Étape 1 : Création et installation de la machine virtuelle

- Télécharger l’ISO de Debian 12 sur le [site officiel Debian](https://www.debian.org/)
- Créer une nouvelle machine virtuelle sur VMWare avec les caractéristiques suivantes :
  - CPU : 4 cœurs
  - RAM : 4 Go
  - Stockage : 32 Go

* Configurer les interfaces réseau en suivant la procédure dans les pré-requis
* Démarrer la machine virtuelle et installer Debian
* Sélectionner **ens33 (NAT)** comme interface principale
![image-20250315135052350](img/image-20250315135052350.png)
* Ne pas définir de mot de passe root, le compte créé sera membre du groupe sudoers
![image-20250315135239614](img/image-20250315135239614.png)
* Sélectionner **XFCE** comme environnement de bureau (plus léger)
![image-20250315135552894](img/image-20250315135552894.png)

Debian est maintenant installé ! Nous allons maintenant configurer les interfaces réseaux.

#### Étape 2 : Configuration des interfaces réseau

La configuration des adresses IP sous Debian repose par défaut en DHCP via NetworkManager. Notre configuration réseau étant particulière, il est nécessaire de configurer manuellement les interfaces de manière permanante grâce au fichier /etc/network/interfaces.

- Modifier le fichier de configuration réseau :

```bash
sudo nano /etc/network/interfaces
```

```bash
auto ens33 # Initialisation de l'interface ens33
iface ens33 inet dhcp # Configuration via DHCP de l'interface ens33

auto ens34 # Initialisation de l'interface ens34
iface ens34 inet static # Configuration de manière statique de l'interface ens34
        address 192.168.1.1/24 # L'interface ens34 aura pour addresse 192.168.1.1 avec pour masque de sous-réseau un /24 (255.255.255.0)
```

* Redémarrer la machine virtuelle pour appliquer les changements
* Vérifier avec la commande `ip a` que les adresses IP sont correctement attribuées (ens33 doit avoir une IP d’un sous réseau obtenu via un DHCP et ens34 avoir l’IP fixe 192.168.1.1/24)
* Confirmer l'accès à Internet en exécutant un ```ping 1.1.1.1```
![image-20250315140922264](img/image-20250315140922264.png)

Le réseau de notre Debian est maintenant parfaitement configuré ! Nous allons maintenant installer FreePBX.

#### Étape 3 : Installation de FreePBX

Nous allons suivre les instructions du guide officiel présent sur le [GitHub de FreePBX](https://github.com/FreePBX/sng_freepbx_debian_install).

- Exécuter les commandes suivantes  :

```shell
sudo su -
wget https://github.com/FreePBX/sng_freepbx_debian_install/raw/master/sng_freepbx_debian_install.sh -O /tmp/sng_freepbx_debian_install.sh
bash /tmp/sng_freepbx_debian_install.sh
```

Le script lance automatiquement l'installation complète de FreePBX avec tous ses modules nécessaires. Cette étape peut prendre du temps, il faut attendre la fin de l'installation sans interruption.

![image-20250315141143324](img/image-20250315141143324.png)

A la fin de son exécution, le script affichera des informations telle que l’adresse IP du FreePBX en vert. Nous allons maintenant le configurer avec l’aide de son interface graphique.

#### Étape 5 : Configuration de FreePBX

* Ouvrir un navigateur web et aller sur http://192.168.1.1/
* Définir le compte utilisateur administrateur de l’instance FreePBX, définir un mail quelconque pour les notifications du système, définir le nom du serveur FreePBX. Puis cliquer sur “Setup System”

![image-20250315144016918](img/image-20250315144016918.png)

* Accéder à l’interface d’administration en allant dans l’onglet “FreePBX Administration”. Se connecter avec les identifiants définis précédemment

![image-20250315144135833](img/image-20250315144135833.png)

* Ignorer toutes les propositions commerciales

* Définir la langue du système sur Français

![image-20250315144530686](img/image-20250315144530686.png)

* Laisser les paramètres par défaut du Sangoma Smart Firewall

![image-20250315144624283](img/image-20250315144624283.png)

![image-20250315144655398](img/image-20250315144655398.png)

![image-20250315144718684](img/image-20250315144718684.png)

![image-20250315144755138](img/image-20250315144755138.png)

![image-20250315144817525](img/image-20250315144817525.png)

![image-20250315144851212](img/image-20250315144851212.png)

- Ignorer toutes les propositions commerciales

* Pour finaliser l’installation, cliquer sur “Appliquer la configuration” ou “Apply Config” en haut à droite si proposé

![image-20250315145408133](img/image-20250315145408133.png)

L’instance FreePBX est désormais installée ! Nous allons maintenant déployer des postes SIP pour connecter nos téléphones physiques et virtuels.

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

#### Étape 6.2 : Connecter un Yealink T42U

Cette partie est extraite du sujet de TP de Sami Evangelista.

Nous allons redémarrer le téléphone en mode usine pour revenir à une configuration vierge.

* Mettre le téléphone sous tension
* Redémarrer le téléphone en mode usine : laisser le bouton OK appuyé pendant quelques secondes puis confirmer le redémarrage.

Une fois le téléphone redémarré, nous allons d’abord configurer statiquement ses paramètres IP.

* Aller dans le menu “3 Settings” -> “2 Advanced Settings” (le mot de passe est admin) -> “2 Network” -> “1 WAN Port” -> “2 IPv4” -> “2 Static IPv4 Client”
* Définir une IP du réseau 192.168.1.0/24 (ex : 192.168.1.11), et choisir comme passerelle l’IP du FreePBX (192.168.1.1)
* Sauvegarder les paramètres : “Save”

Une fois la configuration IP appliquée, ping le téléphone depuis le serveur FreePBX.
Une fois que les pings passent, configurer la ligne SIP.

* Aller dans le menu “3 Settings” -> “2 Advanced Settings” -> “1 Accounts” -> “1.”
* Passer “Active line” à “Enabled”
* Saisir les paramètres suivants :
  * Display Name : Le nom qui s’affichera quand le correspondant recevra un appel
  * Register Name : Numéro du compte SIP
  * User Name : Numéro du compte SIP
  * Password : Mot de passe du compte SIP
  * SIP Server 1 : IP du FreePBX
* Sauvegarder les paramètres : “Save”

Une fois les paramètres saisis, le Display Name devrait s’afficher sur l’écran du téléphone.

* Vérifier la connectivité en appelant le ```*97```
* Effectuer un appel entre le Yealink et le Softphone

FreePBX est désormais installé et correctement configuré !

#### Étape 7 (Bonus) : Auto-Provisioning des Yealink

Il est nécessaire de configurer manuellement les adresses IP et numéros SIP sur les téléphones Yealink, ce qui peut-être fort contraignant en entreprise quand on a un parc de téléphones énorme à gérer.

Cette partie est extraite du sujet de TP de Sami Evangelista.

- Installer isc-dhcp-server sur le serveur FreePBX

```
sudo apt update
sudo apt install isc-dhcp-server -y
```

- Configurer l’interface d’écoute du serveur DHCP

```
sudo nano /etc/default/isc-dhcp-server
```

```
INTERFACESv4="ens34"
```

- Créer la configuration du serveur DHCP

```
sudo nano /etc/dhcp/dhcpd.conf
```

```
# Définition du sous-réseau 192.168.1.0/24
subnet 192.168.1.0 netmask 255.255.255.0 {
    # Plage d'adresses DHCP
    range 192.168.1.10 192.168.1.100;
    # Passerelle (gateway)
    option routers 192.168.1.1;
    # Masque de sous-réseau
    option subnet-mask 255.255.255.0;
    # Option TFTP (pour le provisionning des tels)
    option tftp-server-name "tftp://192.168.1.1";
}
```

- Récupérer l’adresse MAC d’un téléphone Yealink et créer le fichier de configuration associé

```
sudo nano /tftpboot/addressemacdutelephone.cfg
```

```
#!version:1.0.0.1
account.1.enable = 1
account.1.label = Numéro de téléphone SIP
account.1.display_name = Nom de l'appelant
account.1.auth_name = Numéro de Téléphone SIP
account.1.user_name = Numéro de Téléphone SIP
account.1.pasword = Mot de passe du numéro SIP
account.1.sip_server.1.address = IP du serveur FreePBX
lang.gui = French
```

- Redémarrer le serveur DHCP et le serveur TFTP

```
sudo systemctl restart isc-dhcp-server tftpd-hpa
```

- Redémarrer le téléphone Yealink en mode usine et vérifier si le téléphone récupère le compte SIP

- Vérifier le bon fonctionnement du téléphone en appelant le ```*97```

## 2 - Jitsi Meet

#### Étape 0 : Préparer FreePBX

- Créer un poste SIP pour Jitsi avec pour numéro 1000

![image-20250315220329769](img/image-20250315220329769.png)

#### Étape 1 : Installer le Docker Engine

```
sudo curl -sSL https://get.docker.com/ | bash
```

#### Étape 2 : Télécharger les fichiers Jitsi

```
wget $(curl -s https://api.github.com/repos/jitsi/docker-jitsi-meet/releases/latest | grep 'zip' | cut -d\" -f4)
unzip stable*
cd jitsi*
cp env.example .env
./gen-passwords.sh
mkdir -p ~/.jitsi-meet-cfg/{web,transcripts,prosody/config,prosody/prosody-plugins-custom,jicofo,jvb,jigasi,jibri}

cat <<EOF >> .env
PUBLIC_URL=https://192.168.1.1:8443
JVB_ADVERTISE_IPS=192.168.1.1
JIGASI_SIP_URI=1000@192.168.1.1
JIGASI_SIP_PASSWORD=jitsi
JIGASI_SIP_SERVER=192.168.1.1
JIGASI_SIP_PORT=5060
JIGASI_SIP_TRANSPORT=UDP
ENABLE_LETS_ENCRYPT=0
JVB_DISABLE_STUN=true
TZ=Europe/Paris
ENABLE_AUTH=0
ENABLE_GUESTS=1
EOF

mkdir -p ./config/web/certs
if [ ! -f "./config/web/certs/cert.crt" ] || [ ! -f "./config/web/certs/cert.key" ]; then
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
        -keyout "./config/web/certs/cert.key" \
        -out "./config/web/certs/cert.crt" \
        -subj "/C=FR/ST=Ile-de-France/L=Villetaneuse/O=Universite Sorbonne Paris Nord/OU=IUT de Villetaneuse/CN=sae.iutv.univ-paris13.fr"
fi
sed -i '/- ${CONFIG}\/web:\/config:Z/ a\
            - .\/config\/web\/certs\/cert.crt:\/config\/keys\/cert.crt:Z\
            - .\/config\/web\/certs\/cert.key:\/config\/keys\/cert.key:Z' docker-compose.yml
```


```
docker compose -f docker-compose.yml -f jigasi.yml up -d
```

https://192.168.1.1:8443

#### Étape 3 : Visioconférence + SIP

* Passer un appel entre deux ordinateurs en accédant à https://192.168.1.1:8443/ depuis deux ordinateurs du même réseau local
* Appeler un téléphone SIP via le bouton d’invitation

### 3 - Nextcloud Hub

#### Étape 1 : Forger le compose.yaml

```
mkdir nextcloud
nano compose.yaml
```

```yaml
services:
  nextcloud:
    image: ghcr.io/linuxserver/nextcloud:latest
    container_name: nextcloud
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Paris
    volumes:
      - ./nextcloud_config/:/config
      - ./nextcloud_data:/data
    ports:
      - 7443:443
    restart: unless-stopped
  mariadb:
    image: ghcr.io/linuxserver/mariadb:latest
    container_name: mariadb
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Paris
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=nextcloud
    volumes:
      - ./mariadb_config/:/config
    restart: unless-stopped
```

![image-20250316003659370](img/image-20250316003659370.png)

![image-20250316003800348](img/image-20250316003800348.png)

![image-20250316003929728](img/image-20250316003929728.png)

![image-20250316004014530](img/image-20250316004014530.png)
