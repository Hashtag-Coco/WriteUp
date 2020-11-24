![Pic1](../img/admirer1.PNG?raw=true) </br>

# Introduction

Admirer est une machine linux dont l'adresse IP est 10.10.10.187.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un serveur web.
</br>

# Enumération initiale
Nous commençons par l'énumération des services et ports de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.187
```
![Pic2](../img/admirer2.PNG?raw=true) </br>
Un port 80 étant ouvert, nous allons énumérer les fichiers/dossiers présents avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 400,401,404,500 -u http://10.10.10.187/
```
![Pic3](../img/admirer3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
Le scan web révèle un robots.txt qui contient :</Br>
![Pic4](../img/admirer4.PNG?raw=true) </br>
Le dossier /admin-dir est en 403, donc forbidden, mais nous testons une énumération avec dirsearch dessus :
```bash
$ dirsearch -w wordlist -e "php,txt" -f -t 100 -x 400,401,404,500 -u http://10.10.10.187/admin-dir/
``` 
![Pic5](../img/admirer5.PNG?raw=true) </br>
Voici ce que contient **contacts.txt** :</br>
![Pic6](../img/admirer6.PNG?raw=true) </br>
Et **credentials.txt** :</br>
![Pic7](../img/admirer7.PNG?raw=true) </br>
Nous pouvons déjà établir une liste de nom d'utilisateur avec le contacts.txt et une liste de mot de passe avec credentials.txt, et tester 
les différentes combinaisons en ssh :
```bash
$ hydra -L liste_user -P liste_pass 10.10.10.187 -t 10 ssh
```
![Pic8](../img/admirer8.PNG?raw=true) </br>
Aucune combinaison fonctionne, nous allons donc énumérer le ftp avec l'identifiant trouvé:
```bash
$
ftp > ls
```
![Pic9](../img/admirer9.PNG?raw=true) </br>
Nous avons accès à deux fichiers, un dump sql qui contient des informations sur une table et l'info sur la BDD : MariaDB. </br>
L'archive contient beaucoup de fichiers, dont des credentials :</br>
![Pic10](../img/admirer10.PNG?raw=true) </br>
Dans la capture, dans le TODO, nous savons que le developpeur n'a pas implémenter la DBB, il faut donc trouver une solution open
source alternative. Avec ces mots clé, nous tombons sur **Adminer**.</br>
Suite à cette étape, nous trouvons adminer.php :</br>
![Pic11](../img/admirer11.PNG?raw=true) </br>
Malheureusement aucun credentials ne fonctionnent ici. Nous recherchons alors une vuln pour adminer 4.6.2 :</br>
https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool</br>
Le principe est de faire notre BDD puis de la connecter sur celle à distance.
