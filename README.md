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
