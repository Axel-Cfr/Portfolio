# Portfolio d'Axel CHABOT

Je suis un étudiant en deuxième année de cycle préparatoire intégré à 3iL Ingénieurs.  
Je m'intêresse à tout ce qui touche à l'informatique et plus particulièrement à la cybersécurité.

Je recherche dans le domaine de la cybersécurité :
- un stage de 6 semaines à 3 mois à partir du 18 mai 2027
- une alternance à partir de septembre 2027 couvrant mes 3 années de cycle ingénieur
Le stage peut bien sûr servir de "période d'essai" pour une alternance !

J’explore plusieurs facettes de la cybersécurité en autodidacte comme le SOC, le pentest basique et le réseau via TryHackMe mais j’aimerais aussi m’initier à la sécurité de l’IA ainsi qu’à la conception et au maintien d’un SI sécurisé. J'ai complété le parcours Cyber Security 101 (56 salles) et je suis en train de suivre le parcours SOC Level 1.

J'ai aussi réalisé, avec deux camarades, un gestionnaire de mots de passe en Java utilisant AES-256-GCM et Argon2d/id et fonctionnant dans le terminal. L'authentification se fait sans stockage du mot de passe maître ni de sa clé dérivée dans la BDD SQLite.

## Profils

[Linkedin](www.linkedin.com/in/axelchabot)  
[TryHackMe](https://tryhackme.com/p/AxelCfr)

## Formations

### 2025 :

#### Introduction to Cybersecurity - Cisco Networking Academy :
- Valeur des données personnelles et organisationnelles
- Types de vulnérabilités, d'attaquants, de malwares, d'attaques
- Protections des données et bonnes pratiques
- Équipement de sécurité en entreprise
- Schémas d'attaque et de défense
- Cadres de référence en cybersécurité (normes, organismes, principes)

### 2026 :

#### Parcours Cyber Security 101 - TryHackMe (56 salles): 
- Fondamentaux Linux et Windows
- Fondamentaux d'Active Directory et PowerShell
- Concepts et protocoles réseaux + WireShark, Tcpdump et Nmap
- Introduction à la cryptographie et au hashing + John the Ripper
- Exploitation de failles et post-exploitation avec Metasploit
- Introduction au Web Hacking : Console, JavaScript, Injection SQL et BurpSuite
- Introduction aux outils offensifs : Hydra, Gobuster, SQLMap
- Fondamentaux de la sécurité défensive : SOC, Forensic avec la méthode NIST, Réponses à incidents et Logs
- Fondamentaux des solutions de sécurité : Firewall, SIEM, IDS et Scan de vulnérabilités
- Introduction aux outils de sécurité défensive : CyberChef, CAPA, REMnux et FlareVM
- OWASP TOP 10
- Questions sur les notions abordées
- Exercices pratiques

#### Introduction à la méthode EBIOS Risk Manager : 
- Définition et identification des concepts :
    - Critères DIC(T)
    - Valeur Métier
    - Bien support
    - Evenement redouté
    - Source de risque et objectif visé
    - Partie prenante
    - Risque résiduel
    - Scénario stratégique et opérationnel
- Mise en place d'échelles de gravité, de vraisemblance et de risque (vraisemblance/gravité)

#### Parcours SOC Level 1 - TryHackMe (En cours)

## Projets

### [JavaPass](https://github.com/Axel-Cfr/JavaPass) :

Gestionnaire de mots de passe multicompte en ligne de commande (CLI) local et open source entièrement développé en Java. JavaPass fonctionne dans le terminal et stocke vos mots de passe chiffrés dans une base de données SQLite. 
- Stockage sécurisé des identifiants et mots de passe avec l'algorithme de chiffrement **AES-256-GCM**
- Authentification et accès aux identifiants **sans stocker le mot de passe maître, ni la clé dérivée**.
- Accès aux identifiants via un mot de passe maître dérivé avec **Argon2d/id**
- Génération de mots de passes sûrs
- Analyse de la robustesse d'un mot de passe
- Renforcement d'un mot de passe existant
- Recherche rapide des identifiants
- Modification des mots de passes stockés et du mot de passe maître
- Fermeture automatique du gestionnaire au bout de **5 min** d'inactivité
- Interface soignée avec de l'ASCII et des codes couleurs ANSI
- Documentation complète et précise

## WriteUps

#### [Salle Blue - TryHackMe](WriteUps/WriteUp_Blue.md)
- Scan d'une machine vulnérable (Nmap)
- Utilisation de l'exploit Eternal Blue (Metasploit)
- Élevation de privilèges
- Récupération des hashs de mots de passe et décryptage (hashes.com)
- Récupérations de flags

#### [Salle ToolsRus - TryHackMe](WriteUps/WriteUp_ToolsRus.md)
- Enumération de répertoire d'un serveur web (GoBuster)
- Recherche d'informations pouvant aider à une compromission
- Scan de ports (Nmap)
- Scan de l'arborescence des fichiers du serveur web (Nikto)
- Bruteforce de mot de passe (Hydra)
- Utilisation de l'exploit tomcat_mgr_deploy (Metasploit)
- Récupérations de flags