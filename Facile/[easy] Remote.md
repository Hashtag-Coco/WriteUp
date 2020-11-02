![Pic1](../img/remote1.PNG?raw=true) </br>

# Introduction
Remote est une machine Windows dont l'adresse IP est 10.10.10.180.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
</br>

# Enumération initiale
Nous commençons l'énumération par les ports et services de la machine distante avec nmap :
```bash
$ nmap -T4 -A 10.10.10.180
```
![Pic2](../img/remote2.PNG?raw=true) </br>
![Pic3](../img/remote3.PNG?raw=true) </br>
Beaucoup de ports sont ouverts, nous pouvons reconnaitre les ports habituels d'un serveur Windows :
*21 pour un serveur ftp, dont l'utilisateur Anonymous est autorisé.
*80 pour un serveur web.
*111 pour du rpc.
*135 pour du msrpc, il permet des appels de procédures RPC.
*139 pour du netbios-ssn, il permet la découverte et la connexion des voisins dans le réseau.
*445 pour un service SMB.
*2049 pour un démon de montage (rpc.mount).
</br>

Nous allons faire une énumération des fichiers/dossiers du site web avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,html,aspx" -f -t 100 -x 400,403,404,501 -u http://10.10.10.180
```
![Pic4](../img/remote4.PNG?raw=true) </br>
Plein de dossier, l'énumération n'est pas exhaustive, umbraco est intéréssant car c'est un cms.

# Obtenir un accès utilisateur
Pour l'instant, nous avons rien à faire avec Umbraco, nous allons voir les répertoires partagés disponibles :
```bash
$ showmount -e 10.10.10.180
```
![Pic5](../img/remote5.PNG?raw=true) </br>
Nous allons monter ce répertoire chez nous et l'explorer :
```bash
$ 
```