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
