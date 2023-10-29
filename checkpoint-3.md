# Checkpoint 3

<br>

## Partie 1 : Gestion des utilisateurs

### 1.1 : Lister les comptes utilisateurs

```
cat /etc/passwd
```

Les comptes utilisateurs ayant la possibilité de se connecter (d'ouvrir un shell) sur le serveur sont : Wilder et Root

![root](https://github.com/Fairskip/Checkpoint-3/blob/main/Connexion%20Root.png)
![wilder](https://github.com/Fairskip/Checkpoint-3/blob/main/Connexion%20Wilder.png)

<br>

### 1.2 : Création un compte utilisateur

```
root@cp3:/etc# adduser j****
```


Ajout de l'utilisateur « j**** » ...  

Ajout du nouveau groupe « j**** » (1001) ...  

Ajout du nouvel utilisateur « j**** » (1001) avec le groupe « j**** » ...  

Création du répertoire personnel « /home/j**** »...  

Copie des fichiers depuis « /etc/skel »...  

Nouveau mot de passe :  

Retapez le nouveau mot de passe :  

passwd: password updated successfully  

Changing the user information for j****  

Enter the new value, or press ENTER for the default  

        Full Name []: j****  
        
        Room Number []: 7  
        
        Work Phone []:  
        
        Home Phone []:  
        
        Other []:  
        
Cette information est-elle correcte ? [O/n] o  


<br>

### 1.3 : Recommandations sur le compte wilder
Qu'il aurait du donner plus de renseignement lors de son ajout à la liste des users du pc ?

<br>


## Partie 2 : Le stockage

### 2.1 : Analyse des disques

```
lsblk
```

![lsblk](https://github.com/Fairskip/Checkpoint-3/blob/main/2.1%20lsblk.png)

Pour voir l'état des disques durs, on peut aussi utiliser  :

```
cat /proc/mdstat
```

On y trouve un Raid1 de 8Go active formé de 2 physical volume dont seulement un est up (sda1) et l'autre enlevé. Son state est clean, dégradé.  

Le Raid possède 2 volume groupe : root et swap_1

<br>


### 2.2 : Gestion du RAID

* Formatage du disque dure sdb
![formatage](https://github.com/Fairskip/Checkpoint-3/blob/main/2.1%20formatage%20du%20dd%20sdb.png)

* Ajouter la partition sbd1 au Md0

```
 mdadm --manage /dev/md0 --add dev/sdb1
```

* Verifier que le raid est reconstruit

![raid opérationnel](https://github.com/Fairskip/Checkpoint-3/blob/main/2.1%20detail.png)

<br>

### 2.3 : Gestion de LVM
- Ajoute un nouveau volume logique LVM de 2 Gio pour héberger les sauvegardes bareos. Ce volume doit être monté automatiquement à chaque démarrage dans l'emplacement par défaut : `/var/lib/bareos/storage`.
- Combien d'espace disponible reste-t-il dans le groupe de volume ?

<br>

## Partie 3 : SSH

### 3.1 : Configuration de SSH

Commentaire | Action
:---:|:---:
ouverture du fichier sshd_config | root@cp3:/etc/ssh# nano sshd_config
decommenter permitRootLogin et remplacer le truc par no | PermitRootLogin no
Decommenter Match User et ajouter son ID | Match User j****
relancer le service ssh | root@cp3:/etc/ssh# service ssh restart


<br>

### 3.2 : Authentification cryptographique

Commentaire | Action
:---:|:---:
Ouvrir le fichier sshd_config pour modifier son contenu| nano sshd_config
Decommenter PasswordAuthentification et mettre no | PasswordAuthentication no
Decommenter PubeyAuthentification et mettre Yes | PubkeyAuthentication yes
Decommenter ListenAddress Ipv4 | ListenAddress 0.0.0.0
Decommenter Listenddress IPv6 | ListenAddress ::
Sauvegarder et quitter | ctrl X
Créer un dossier .ssh dans home avec le compte de l'user j**** | mkdir .ssh
Entrer dans le dossier .ssh | cd .ssh
Créer un fichier Authorized_keys avec le compte de l'user j**** | Touch Authorized_keys
Ouvrir le fichier Authorized_keys | nano authorized_keys
Coller la clé public rsa généré mobaxtern qu'on peut retrouver dans le fichier .ssh/id_rsa.pub | [cliquer ici](https://github.com/Fairskip/Checkpoint-3/blob/main/Authorized%20keys.png)
Sauvegarder et quitter | ctrl X

![ssh user](https://github.com/Fairskip/Checkpoint-3/blob/main/Connexion%20j.jpg)

<br>

## Partie 4 : Filtrage et analyse réseau

### 4.1 : Analyse de règles de filtrage

![nft](https://github.com/Fairskip/Checkpoint-3/blob/main/nft%20list%20ruleset.png)

Quelles sont actuellement les règles appliquées sur Netfilter ?  
Il y a une seule regle. La règle inet_filter_table de la famille de Inet. Ca filtre les paquets venant d'expérieur et arrivant au port 22.

Quels types de communications sont autorisées ?  

le ping, c'est à dire le protocol icmp 

Et quels types sont interdites ?
tout ce qui n'est pas un ipv6

<br>

### ### 4.2 : Paramétrage avec **nftables**

Ajoute les règles nécessaires pour autoriser bareos actuellement installé sur le serveur à communiquer avec les clients bareos potentiellement présents sur l'ensemble des machines du réseau local sur lequel se trouve le serveur.

<br>

## Partie 5 : Sauvegardes


<br>

## Partie 6 : Analyse de Logs
### 6.1 : Les échecs de connexion


```
root@cp3:/etc# tail /var/log/auth.log
```
<br>


> Oct 27 09:19:47 cp3 login[742]: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=/dev/tty1 ruser= rhost=  user=root

Date et heure de la tentative : 27 octobre à 09:19:47
Adresse IP de la machine ayant fait la tentative : Pas d'Ip car ça s'est fait sur la console de la VM

<br>

> Oct 27 09:19:50 cp3 login[742]: FAILED LOGIN (1) on '/dev/tty1' FOR 'root', Authentication failure

Date et heure de la tentative : 27 octobre à 09:19:50
Adresse IP de la machine ayant fait la tentative : Pas d'Ip car ça s'est fait sur la console de la VM

<br>

> Oct 27 09:20:34 cp3 login[1117]: pam_unix(login:auth): check pass; user unknown
> Oct 27 09:20:34 cp3 login[1117]: pam_unix(login:auth): authentication failure; logname=LOGIN uid=0 euid=0 tty=/dev/tty1 ruser= rhost=

Date et heure de la tentative : 27 octobre à 09:20:34
Adresse IP de la machine ayant fait la tentative : Pas d'Ip car ça s'est fait sur la console de la VM

<br>

> Oct 27 11:48:38 cp3 sshd[843]: Connection closed by authenticating user root 192.168.43.45 port 6925 [preauth]

Date et heure de la tentative : 27 octobre à 11:48:38
Adresse IP de la machine ayant fait la tentative : 192.168.43.45

<br>

> Oct 27 11:49:05 cp3 sshd[845]: Connection closed by authenticating user jfong 192.168.43.45 port 6931 [preauth]

Date et heure de la tentative : 27 octobre à 11:49:05
Adresse IP de la machine ayant fait la tentative : 192.168.43.45


<br>

> Oct 27 12:20:22 cp3 sshd[991]: Connection closed by authenticating user jfong 192.168.43.45 port 7436 [preauth]

Date et heure de la tentative : 27 octobre à 12:20:22
Adresse IP de la machine ayant fait la tentative : 192.168.43.45

<br>

> Oct 27 12:28:30 cp3 sshd[1033]: Connection closed by authenticating user root 192.168.43.45 port 7516 [preauth]

Date et heure de la tentative : 27 octobre à 12:28:30
Adresse IP de la machine ayant fait la tentative : 192.168.43.45

<br>

> Oct 27 13:15:34 cp3 sshd[1298]: Connection closed by authenticating user root 192.168.43.45 port 8605 [preauth]

Date et heure de la tentative : 27 octobre à 13:15:34
Adresse IP de la machine ayant fait la tentative : 192.168.43.45

<br>

> Oct 27 13:15:41 cp3 sshd[1300]: Connection closed by authenticating user jfong 192.168.43.45 port 8606 [preauth]

Date et heure de la tentative : 27 octobre à 13:15:41
Adresse IP de la machine ayant fait la tentative : 192.168.43.45

<br>

> Oct 27 13:18:42 cp3 sshd[1353]: Connection closed by authenticating user jfong 192.168.43.45 port 8617 [preauth]

Date et heure de la tentative : 27 octobre à 13:18:42
Adresse IP de la machine ayant fait la tentative : 192.168.43.45
