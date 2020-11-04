![Pic1](../img/haystack1.PNG?raw=true) </br>

# Introduction
Haystack est une machine Linux dont l'adresse IP est 10.10.10.115.

Compétences mises en oeuvre :
* Enumération des ports et services d'ouverts sur une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* Recherche sur Elasticsearch pour trouver un tool.
</br>

# Enumération initiale
Nous commencons par l'énumération des ports et services avec nmap :
```bash
$ nmap -T4 -A 10.10.10.115
```
![Pic2](../img/haystack2.PNG?raw=true) </br>
Plusieurs ports d'ouverts :
* 22 pour un serveur openssh en version 7.4.
* 80 pour un serveur web nginx version 1.12.2
* 9200 pour un autre serveur web nginx version 1.12.2
</br>
Nous allons faire une énumération des fichiers/dossiers présents sur les sites web avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 500,403,404,501 -u http://10.10.10.115:9200/
```
![Pic3](../img/haystack3.PNG?raw=true) </br>
Sur le port 80 rien à part une grosse image et plusieurs pages sur le port 9200.

# Obtenir un accès utilisateur
Sur le port 80, l'image contient du texte encodé en base64 :
```bash
$ wget http://10.10.10.115/needle.jpg
$ strings needle.jpg
$ echo "strings" | base64 -d
```
![Pic5](../img/haystack5.PNG?raw=true) </br>
Nous avons donc un indice en espagnol : la aguja en el pajar es "clave". Clave est surement un mot clé.</br>
Sur le port 9200, les deux pages sont en réalité du json avec du contenu, exemple :</br>
![Pic4](../img/haystack4.PNG?raw=true) </br>
Sur le json, nous pouvons lire elasticsearch, qui est un moteur de recherche et d'analyse de données. Un tool nommé elasticdump nous permet de récupérer du contenu s'il y en a :
```bash
$ elasticdump --input=http://10.10.10.115:9200/quotes --output=quotes.json --type=data
```
![Pic6](../img/haystack6.PNG?raw=true) </br>
Toutes les données récupérées sont des citations en espagnol. Nous allons faire un grep avec le mot "clave" :
```bash
$ grep "clave" quotes.json
```
![Pic7](../img/haystack7.PNG?raw=true) </br>
Nous décodons alors les deux passages en base64 :</br>
![Pic8](../img/haystack8.PNG?raw=true) </br>
Nous avons un identifiant et un mot de passe, nous pouvons nous connecter en ssh et récupérer le user.txt :</br>
![Pic9](../img/haystack9.PNG?raw=true) </br>
