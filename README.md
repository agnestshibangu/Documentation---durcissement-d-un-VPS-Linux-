### Sécurisation d’un serveur vps
==================================================

mettre à jour le système : 
sudo apt update 
cette commande met à jour la liste des paquets et dépôts dans notre système mais pas les paquets eux mêmes.

sudo apt upgrade
avec cette commande on met à jour les dépôts en eux memes. Cette étape est importante pour les mises à jours de sécurité

créer un utilisateur dans le groupe sudo : 
sudo adduser <username>

puis l’ajouter au groupe sudo
sudo usermod -aG sudo <username>

pour verifier le contenu d’un groupe
	getent group <groupname>

	pour se logger au nouvel utilisateur
	sudo su - -login <username>

	ifconfig est obsolète. Maintenant on utilise ip link show
	ip est un binaire utilisateur. il envoie une requête netlink au noyau. Le noyau retourne la liste des interfaces réseaux connues. Chaque interface correspond à une structure net_device en mémoire noyau.

Sécuriser l’accès SSH : 
	modifier le port de connexion. Par défaut c’est le 22. 22 est le port par défaut du service ssh. interêt du changement : les scanners automatiques savent que 22 = SSH, donc le port est testé en priorité. Les attaques automatiques scannent le port 22, tentent root, admin, test combiné à des mots de passe faibles. Changer le port SSH permet de passer sous le radar de ces scripts automatisés → moins de tentatives, logs plus propres, moins de charge inutile sur sshd. Ca cache le service mais ne le protège pas. Un scan de type nmap -p- sur tout les ports découvrira le port non standard. Il faudra penser a changer le port sur toutes le configs.
	sudo nano /etc/ssh/sshd_config
	et changer la ligne qui correspond
puis redémarrer ssh avec la nouvelle config
	sudo systemctl restart ssh
maintenant pour se connecter il faut preciser le port
	ssh <username>@<ipaddr> -p <port ssh>
ensuite, on peut empêcher la connexion par mot de passe sur le compte root
pour cela on va de nouveau dans /etc/ssh/sshd_config et on change la ligne 
PermitRootLogin yes → PermitRootLogin prohibit-password
	

utiliser une clé SSH pour se connecter avec notre nouvel utilisateur
ssh-keygen -t ed25519
puis on envoie la clé publique au vps



sudo: cd: command not found
sudo: "cd" is a shell built-in command, it cannot be run directly.
sudo: the -s option may be used to run a privileged shell.
sudo: the -D option may be used to run a command in a specific directory.
→ sudo -s 

Installer le pare feu UFW


dpkg -L ufw

Voir les connexions entrantes actives avant UFW

voir les connexions déjà acceptées par le noyau
ss -tunap

ESTAB 0 0 192.0.2.10:22 198.51.100.4:53421 users:(("sshd",pid=1234,fd=3))
Interprétation ligne par ligne :

ESTAB : connexion TCP établie

192.0.2.10:22 : ton serveur, port SSH

198.51.100.4:53421 : client distant

sshd : processus qui a accepté la connexion

👉 Ça, c’est une connexion entrante réelle.

maintenant je vais activer les logs pour ufw

sudo ufw logging on      qui apparaitront ici sudo journalctl -f | grep UFW

et maintenant on va bloquer toutes les connexions entrantes avec la commande : 

 sudo ufw default deny incoming

il faut affiner la sécurité en bloquant / débloquant certains port sortants. Par défaut, UFW debloque tout

5) Fail2ban : 
un programme pour éviter les attaques par force brute. Fail2ban vérifie le journal système, les logs, etc. S’il constate qu’une adresse ip a tenté plusieurs fois et a échoué à se connecter, il la bloque automatiquement.

apt update

apt install fail2ban

config de fail2ban dans /etc/fail2ban/jail.local

  GNU nano 7.2                                                 /etc/fail2ban/jail.local *
[sshd]
enabled = true
port = 22
logpath = /var/log/syslog
maxretry = 5
bantime = 1200
findtime = 300
backend = systemd

et on l’active avec la commande suivante : 

sudo systemctl status fail2ban

et on garantit qu’il se lance à chaque démarrage du serveur 

sudo systemctl enable fail2ban

et si on change la config 

sudo systemctl restart fail2ban

et maintenant on audit avec lynis ! 

sudo apt update

sudo apt install lynis

lynis –version

lancer un audit 

sudo lynis audit system

on peut consulter les rapports suivants 



et j’ai ce warning 

 ! Can't find any security repository in /etc/apt/sources.list or sources.list.d directory [PKGS-7388]
      https://cisofy.com/lynis/controls/PKGS-7388/
