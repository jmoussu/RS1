# RS1

J'utilise une VM Ubuntu server !

## Attribuer une IP fixe à un serveur

`sudo vim /etc/network/interfaces`

/ # The primary network interface
/ auto enp0s3

/ #DHCP :
/ #iface enp0s3 inet dhcp

/ #STATIC :
/ iface enp0s3 inet static
/ #	ip a pour l'adresse ou ifconfig
/ 	address 10.11.200.173
/ #	/30
/ 	netmask 255.255.255.252
/ #	ip root
/ 	gateway 10.13.254.254
/ #	dns
/ 	dns-nameservers 8.8.8.8
/ 	dns-nameservers 8.8.4.4
/ 	dns-search something.network.com


`sudo /etc/init.d/networking restart`

## Vous devez changer le port par defaut du service SSH par celui de votre choix.
Tuto suivi : https://www.aidoweb.com/tutoriaux/changer-port-serveur-ssh-645
vim /etc/ssh/sshd_config
port 22 to 1666

relancer le service avec 
`service sshd restart`
ou
`/etc/init.d/ssh restart` (status, stop, start, restart)

Ne pas oublier le flag -p
`ssh jmoussu@10.11.x.x -p 1666`

## L’accès SSH DOIT se faire avec des publickeys. L’utilisateur root ne doit pas pouvoir se connecter en SSH.
Tuto suivi : https://doc.fedora-fr.org/wiki/SSH_:_Authentification_par_cl%C3%A9
Sur la machine d'ou l'on veut se conecter (Le Mac de l'ecole)
La création de la paire de clé se fait avec `ssh-keygen`.
Deux fichiers ont été créés (dans le dossier ~/.ssh/) :

id_rsa (ou id_dsa dans le cas d'une clé DSA) : contient la clé privée et ne doit pas être dévoilé ou mis à disposition ;
id_rsa.pub (ou id_dsa.pub dans le cas d'une clé DSA) : contient la clé publique, c'est elle qui sera mise sur les serveurs dont l'accès est voulu.


Copier la clef dans `~/.ssh/authorized_keys` de la VM
`ssh-copy-id -i ~/.ssh/id_rsa.pub user@ip_machine`
l'aces ce fait sans mot de passe!

## Vous devez mettre en place des règles de pare-feu (firewall) sur le serveur avec uniquement les services utilisés accessible en dehors de la VM.
J'utilise UFW Uncomlicated Fire Wall avec ce tuto : https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw/

`sudo ufw default allow outgoing
sudo ufw default deny incoming`

`sudo ufw allow 1666` pour mon port ssh
`sudo ufw allow 80/tcp` http
`sudo ufw allow 443` https

`sudo ufw status`
donne :
/ État : actif
/ Vers                       Action      De
/ ----                       ------      --
/ 1666                       ALLOW       Anywhere
/ 80/tcp                     ALLOW       Anywhere
/ 443                        ALLOW       Anywhere
/ 1666 (v6)                  ALLOW       Anywhere (v6)
/ 80/tcp (v6)                ALLOW       Anywhere (v6)
/ 443 (v6)                   ALLOW       Anywhere (v6)

## Vous devez mettre en place une protection contre les DOS (Denial Of Service Attack) sur les ports ouverts de votre VM.
J'ai utilise fail2ban
avec ce tuto https://blog.rapid7.com/2017/02/13/how-to-protect-ssh-and-apache-using-fail2ban-on-ubuntu-linux/

Cmd utile :
Jail list :
`sudo fail2ban-client status`
Ip ban list :
`sudo fail2ban-client status ssh` ou apach http-get-dos...
Unban ip.
`sudo fail2ban-client set ssh unbanip 10.11.x.x`

## Vous devez mettre en place une protection contre les scans sur les ports ouverts de votre VM.
tuto suivi : https://fr-wiki.ikoula.com/fr/Se_prot%C3%A9ger_contre_le_scan_de_ports_avec_portsentry

Une foi la config terminer pour tester le scan avec nmap
install nmap
`brew install nmap` Tres long !
Scan :
`nmap -sV -p 1-65535 10.11.x.x/30`
l'ip du scan sera alors bani
l'acces SSH va être fermé et bloqué !
pour voir l'ip ban :
`iptables -L -n -v`
et
`sudo cat /etc/hosts.deny`

Pour unban :
`iptables -D INPUT -s 178.170.xxx.xxx -j DROP`
et
`sudo vim /etc/hosts.deny` pour retirer l'adresse ip ban

`sudo reboot`

##  Arretez les services dont vous n’avez pas besoin pour ce projet.

 Lister les service :
 Absolument tout !
 `systemctl list-unit-files`
 Seulement les serice non statique (plus utile) :
 `sudo systemctl stop monservice`

 Ayant une VM déjà faite pour les serveur web je n'ai pas beaucoup de service inutile.

-Acpid : Gestion d'allimentation (utile surtout pour les laptop)
- Virtualbox-guest-utils : Pasfonctionelle de tout façon sur ma VM !

Commandes utile pour arreter ou desactiver un service :
`sudo systemctl stop monservice`
`sudo systemctl disable <services inutiles>`
`sudo update-rc.d monservice disable`

## Réalisez un script qui met à jour l’ensemble des sources de package, puis de vos Packages et qui log l’ensemble dans un fichier nommé /var/log/update_script.log.
Le-dit script est placer dans /root/scripts/update_script.sh pour qu'il soit proteger et `chmod 700 /root/scripts/update_script.sh`	

`#! /bin/bash
LOG_FILE=/var/log/update_script.log
echo "WAIT PLEASE..."
echo "

//// NEW LOG ////" 1>>$LOG_FILE
echo "Date du jour :" 1>>$LOG_FILE
date 1>>$LOG_FILE
apt-get update 1>>$LOG_FILE
apt-get upgrade 1>>$LOG_FILE
echo "This script RUN:"
echo "apt-get update 1>>$LOG_FILE AND apt-get upgrade 1>>$LOG_FILE"
echo "Log are in /var/log/update_script.log"`

On trouve dans var/log/update_script.log
`

//// NEW LOG ////
Date du jour :
vendredi 1 mars 2019, 17:55:30 (UTC+0100)
Atteint:1 http://fr.archive.ubuntu.com/ubuntu xenial InRelease
Réception de:2 http://fr.archive.ubuntu.com/ubuntu xenial-updates InRelease [109 kB]
Réception de:3 http://fr.archive.ubuntu.com/ubuntu xenial-backports InRelease [107 kB]
Réception de:4 http://security.ubuntu.com/ubuntu xenial-security InRelease [109 kB]
325 ko réceptionnés en 0s (1 042 ko/s)
Lecture des listes de paquets…
Lecture des listes de paquets…
Construction de l'arbre des dépendances…
Lecture des informations d'état…
Calcul de la mise à jour…
Les paquets suivants ont été conservés :
  linux-generic linux-headers-generic linux-image-generic ubuntu-minimal
0 mis à jour, 0 nouvellement installés, 0 à enlever et 4 non mis à jour.`

Pour eviter que mon fichier explose en taille j'ai ajouter une regle dans logrotate `/etc/logrotate.d`

`/var/log/update_script.log {
    rotate 1
    size 5k
    missingok
    notifempty
    create 440 root root
}`

rotate est le nombre d’indice de fichiers de logs à atteindre avant la suppression du fichier le plus ancien. 
size force la rotation du fichier de logs si celui atteint 5Ko. Cette indication écrase donc daily.
missingok force Logrotate à créer un nouveau fichier de logs si celui étant ouvert devient introuvable.
notifempty empèche une rotation de fichier de logs si celui est vide.
create permet de définir les droits sur un fichier de logs ainsi que son appartenance.

Cela va faire un fichier de log de rotation si le fichier depasse de 5Ko 
Lors d'une autre rotation le fichier de rotation sera supprimer.
Logrotate a une configuration par defaut qui lui permet de ce lancer tout les jours.
On peut simuler sont lancement avec ; `sudo logrotate /etc/logrotate.conf --debug`
Ou le lancer avec `sudo logrotate /etc/logrotate.conf`


##Créez une tache planifiée pour ce script une fois par semaine à 4h00 du matin et à chaque reboot de la machine.

J'ai ajouter les ligne 
`0 4     * * 1   root    /root/scripts/update_script.sh  >> /var/log/update_script.log
@reboot         root    /root/scripts/update_script.sh  >> /var/log/update_script.log`

Dans `/etc/contab`
Apres un 'sudo reboot' Cela fonctionne bien le fichier de log affiche un nouveau log.

##Réalisez un script qui permet de surveiller les modifications du fichier /etc/crontab et envoie un mail à root si celui-ci a été modifié.

Installation de mail en local : `sudo apt-get install mailutils`

Envoie du mail : `echo "missive" | mail -s "subject" root`
Lecture du mail : `sudo cat /var/mail/root`

Scripte dans `/root/scripts/cron_watchdog.sh`:
`
#!/bin/bash
FILECHECKSUM="/root/scripts/checksum_de_etc_contab"
FILE_TO_WATCH="/etc/crontab"
CHECKSUM=$(md5sum $FILE_TO_WATCH)
if [ ! -f $FILECHECKSUM ]
then
	echo "$CHECKSUM" > $FILECHECKSUM
	echo "1er lancement du script mise en place du Checksum de $FILE_TO_WATCH dans $FILECHECKSUM"
	exit 0;
fi;
if [ "$CHECKSUM" != "$(cat $FILECHECKSUM)" ];
then
	echo "$CHECKSUM" > $FILECHECKSUM
	echo "$FILE_TO_WATCH has been modified !" | mail -s "$FILE_TO_WATCH modified !" root
	echo "$FILE_TO_WATCH has beed modified ! Mail send to root, check /var/mail/root"
else
	echo "$FILE_TO_WATCH hasn't beed modified."
fi;
`
On utilise md5sum pour crée un CHECKSUM du fichier une clef qui lui est propre qui change quand le fichier est modifier.

## Créez une tache plannifiée pour script tous les jours à minuit.

Vu que notre script cherche les modification de `/etc/crontab` il serait peu sensé ce le placer dedans pour le lancer.
Il sufirait de modifier crontab eb suprimant le ligne pour qu'il ne se lance plus !
On va donc le lancer depuis le crontab de root : `/var/spool/cron/crontabs/root` ou `en su : crontab -e`
La ligne : `0 0 * * * root /root/scripts/cron_watchdog.sh`

##Partie optionnelle WEB

## Vous devez mettre en place du SSL auto-signé sur l’ensemble de vos services.
Activation du module ssl de apache2
`sudo a2enmod ssl`
`sudo service apache2 restart`

On ajoute un docier ssl `sudo mkdir /etc/apache2/ssl` pour générer le certificat ssl
`sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt`
repondre aux questions pour le certificat

On fait notre propre config ssl pour Roger Skyline : `rsync /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/ssl-roger.conf`
(rsync ou cp)
sudo vim /etc/apache2/sites-available/ssl-roger.conf

on modifie les ligne :
`SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key`

Pour activer la coonfig : `a2ensite ssl-roger.conf`

Ajout de `ServerName localhost` dans /etc/apache2/apache2.conf

`sudo service apache2 restart`

On test notre config avec les commande suivante : 
`sudo apachectl configtest`
`sudo a2enmod ssl`
`sudo a2ensite ssl-roger.conf`

Si il y a aucun message d'erreur la config est corect et fini ! 

## Vous devez mettre en place un serveur web qui DOIT être disponible sur l’IP de laVM ou un host

On peut a présent proposer une pageweb html sur l'ip de nottre machine.
`/var/www/html`
