![Pic1](../img/traverxec1.PNG?raw=true) </br>

# Présentation
Traverxec est une box Linux dont l'adresse IP est 10.10.10.165.</br>

Compétences mises en oeuvre :</br>
* Enumération des ports et services d'une machine distante.


</br>

# Enumération initiale

Nous commençons l'énumération par les ports et services disponibles avec nmap :
```bash
$ nmap -T4 -A 10.10.10.165
```
![Pic2](../img/traverxec2.PNG?raw=true) </br>
Deux ports sont ouverts :</br>
* 22 pour un OpenSSH en version 7.9
* 80 pour un serveur web nostromo en version 1.9.6
</br>
Nous allons faire une énumération des fichiers et dossiers présents sur le site web avec dirsearch :</br>
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 400,403,404,501 -u http://10.10.10.165/
```
![Pic3](../img/traverxec3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
Nous avons vu que le serveur web en écoute était un nostromo 1.9.6, ce qui n'est pas commun, nous allons voir s'il existe un exploit pour 
cette version avec searchsploit :
```bash
$ searchsploit nostromo
```
![Pic4](../img/traverxec4.PNG?raw=true) </br>