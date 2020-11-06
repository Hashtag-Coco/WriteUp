![Pic1](../img/teacher1.PNG?raw=true) </br>

# Introduction
Teacher est une machine Linux qui a pour adresse IP 10.10.10.153.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
</br>

# Enumération initiale
Nous commençons comme d'habitude avec nmap pour énumémer les ports d'ouverts et les services exposés de la box :
```bash
$ nmap -T4 -A 10.10.10.153
$ nmap -T4 -p1-65535 10.10.10.153
```
![Pic2](../img/teacher2.PNG?raw=true) </br>
Seul le port 80 est ouvert, nous allons donc faire une énumération des dossiers et fichiers du site avec dirsearch :
```bash
$ dirsearch.py -w wordlist -e 'php,txt' -f -t 100 -x 400,404,403,500 -u http://10.10.10.153/
```
![Pic3](../img/teacher3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
Sur la page /moodle/, nous avons une interface concernant la plateforme d'apprentissage Moodle :</br>
![Pic4](../img/teacher4.PNG?raw=true) </br>
Sauf qu'apparament, il faut être identifier pour accès à une ressource du site, néanmoins, nous pouvons nous logger en tant que Guest :</br>
![Pic5](../img/teacher5.PNG?raw=true) </br>
Sauf qu'en tant que Guest, rien n'est possible de faire. Nous avons de sûr un utilisateur : Giovanni Chhatta. Qui est un enseignant qui propose
le cours en ligne Algebra.</br>
Après avoir passé ***beaucoup*** de temps à chercher, l'image numéro 5 du site ne s'affiche pas. Elle contient en réalité des données :
```bash
$ wget http://10.10.10.153/images/5.png
$ cat 5.png
```
![Pic6](../img/teacher6.PNG?raw=true) </br>
Nous avons a deviner le dernier caractère qu'il reste, après un certain temps, le couple Giovanni:Th4C00lTheacha# fonctionne pour s'authentifier.</br>
Une fois connecté, nous pouvons suivre un POC pour obtenir une RCE :</br>
https://blog.ripstech.com/2018/moodle-remote-code-execution/</br>
