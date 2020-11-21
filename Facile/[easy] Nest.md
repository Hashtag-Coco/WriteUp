![Pic1](../img/nest1.PNG?raw=true) </br>

# Introduction

Nest est une machine windows dont l'adresse IP est 10.10.10.178.</br>
Compétences mises en oeuvre :
* Enumération des ports et services d'une machine distante.
* Enumération des dossiers partagés d'une machine distante.
</br>

# Enumération initiale
Nous commencons l'enumération par un scan des ports et services exposés de la box avec nmap :
```bash
$ nmap -T4 -A 10.10.10.178
```
![Pic2](../img/nest2.PNG?raw=true) </br>
Deux ports d'ouverts, nous allons faire un scan sur le port 445 pour voir les dossiers partagés :
```bash
$ smbclient -L 10.10.10.178
```
![Pic3](../img/nest3.PNG?raw=true) </br>

# Obtenir un accès utilisateur
Nous avons accès à deux dossiers partagés : **Data** et **Users**. Le dossier Data est le plus intéréssant, il contient plusieurs dossiers :
```bash
$ smbclient \\\\10.10.10.178\\Data
smb > ls
```
![Pic4](../img/nest4.PNG?raw=true) </br>
Le sous dossier Shared contient d'autres sous-dossiers pour qui l'un deux, contient un fichier **Welcome Email.txt**
qui a le contenu suivant :</br>
![Pic5](../img/nest5.PNG?raw=true) </br>
Nous avons des creds, qui vont simplement nous permettre d'obtenir des droits sur certains dossiers partagés. C'est le cas de **Data**,
sur lequel nous pouvons maintenant voir beaucoup de fichiers xml à l'intérieur de \\Data\IT\Configs, mais l'un deux contient des info intéréssantes :</br>
![Pic6](../img/nest6.PNG?raw=true) </br>
Nous avons un mot de passe encodé en base 64, une fois décodé, le mot de passe ne fonctionne nul part.
Après un tour sur le forum d'aide, il faut nous aider de deux choses, d'une part un fichier config de **NotepadPlusPlus** afin de découvrir les fichiers récémment ouverts, qui se situe dans //10.10.10.178/Data/IT/Configs/NotepadPlusPlus/config.xml :</br>
![Pic7](../img/nest7.PNG?raw=true) </br>
C'est avec l'étape précédente que nous pouvons comprendre que nous avons des droits dessus, nous pouvons alors monter Secure$ :
```bash
$ mount.cifs //10.10.10.178/Secures$/IT/Carl ./Carl -o user=TempUser
```
</br>
Deuxième chose : le dossier partagé **Secure$**. Ce dernier contient deux fichiers très intéréssants : **\\Secure$/VB Projects/WIP/RU/RUScanner/Module1.vb** et **\\Secure$/VB Projects/WIP/RU/RUScanner/Utils.vb**, il s'agit de la méthode pour chiffrer les mot de passes :</br>
![Pic8](../img/nest8.PNG?raw=true) </br>
![Pic9](../img/nest9.PNG?raw=true) </br>
Dans le fichier Utils.vb, nous pouvons retrouver tous les élèments qui composent le mot de passe :
* L'IV est : 464R5DFA5DL6LE28.
* La passphrase : N3st22.
* Le salt : 88552299.
* Le nombre d'itération : 2.
* La clé : 256.
</br>
Nous pouvons utiliser Dotnetfiddle.net pour déchiffrer le mot de passe, nous pouvons coller entierement le code de utils.vb en ajoutant juste un write-line pour afficher le mot de passe :</br>
![Pic10](../img/nest10.PNG?raw=true) </br>
Nous pouvons alors récupérer et lire le flag user.txt :
```bash
$ mount.cifs //10.10.10.178/Users/C.Smith ./smith
$ cat smith/user.txt
```
![Pic11](../img/nest11.PNG?raw=true) </br>
