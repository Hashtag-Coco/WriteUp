![Pic1](../img/nest1.PNG?raw=true) </br>

# Introduction

Nest est une machine windows dont l'adresse IP est 10.10.10.178.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des dossiers partagés d'une machine distante.
</br>

# Enumération initiale
Nous commencons l'enumération par un scan des ports et services exposés de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.178
```
![Pic2](../img/nest2.PNG?raw=true) </br>
Deux ports d'ouverts, nous allons faire un scan sur le port 445 pour voir les dossiers partagés :
```bash
$ smbclient -L 10.10.10.178
```
![Pic3](../img/nest3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
