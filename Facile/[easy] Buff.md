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