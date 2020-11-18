![Pic1](../img/friendzone1.PNG?raw=true) </br>

# Introduction
Friendzone est une machine linux dont l'adresse IP est 10.10.10.123.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
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
