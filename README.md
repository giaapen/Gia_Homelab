Homelab Infrastructure & Sécurité (VMware Workstation)

Homelab réalisé pour pratiquer l'administration systèmes et réseaux et les bases de la sécurité : un domaine Active Directory, des postes clients Windows et Linux et une machine d'analyse, sur un réseau virtuel isolé.
## Schéma


<img width="2720" height="1360" alt="Image" src="https://github.com/user-attachments/assets/cba25f30-2082-43e8-9ba6-28dd2c5e23e0" />

## Machines

| Machine | Système | Rôle | Adresse IP |
|---|---|---|---|
| SRV-GIA01 | Windows Server 2022 | Contrôleur de domaine (AD DS, DNS) | 192.168.174.142 |
| PC-CLIENTLAB01 | Windows 11 Enterprise (évaluation) | Poste client joint au domaine | 192.168.174.200 |
| Kali | Kali Linux | Scans et analyse réseau | 192.168.174.129 |
| Debian | Debian | Machine Linux du lab | 192.168.174.137 |

Réseau : VMnet2 (Host-only), 192.168.174.0/24, sans passerelle ni accès à Internet.
Domaine : `labgia.lan`

Réseau : VMnet2 (Host-only), 192.168.174.0/24, sans passerelle ni accès à Internet. Domaine : labgia.lan

Ce que j'ai réalisé
1. Réseau virtuel isolé
Création d'un réseau Host-only (VMnet2) sous VMware Workstation.
Adressage IP fixe pour les serveurs et les machines d'analyse, tests de connectivité avec ping.

### Tests de connectivité (ping)

| Depuis \ Vers | SRV-GIA01 (192.168.174.142) | PC-CLIENTLAB01 (192.168.174.200) | Kali (192.168.174.129) | Debian (192.168.174.130) |
|---|:---:|:---:|:---:|:---:|
| **SRV-GIA01** | n/a | ✅ | ✅ | ✅ |
| **PC-CLIENTLAB01** | ✅ | n/a | ✅ | ✅ |
| **Kali** | ✅ | ✅ | n/a | ✅ |
| **Debian** | ✅ | ✅ | ✅ | n/a |

<img width="557" height="155" alt="Image" src="https://github.com/user-attachments/assets/92f2bf34-09ad-4002-af94-8a8c07c707ee" />

2. Active Directory
Installation du rôle AD DS et promotion en contrôleur de domaine (labgia.lan), avec DNS intégré.

<img width="945" height="787" alt="Image" src="https://github.com/user-attachments/assets/99f07cae-1904-4a88-a4a2-5fbe4268cbea" />


- Création d'unités d'organisation, d'utilisateurs et d'un groupe.

<img width="945" height="717" alt="Image" src="https://github.com/user-attachments/assets/753b9c1f-050e-4ecb-a063-224d86f22a7d" />



3. Jonction d'un poste client au domaine
Configuration du DNS du client sur le contrôleur de domaine.
Jonction du poste Windows 11 au domaine via sysdm.cpl, connexion avec un compte du domaine.

<img width="945" height="727" alt="Image" src="https://github.com/user-attachments/assets/c6c896ce-f582-4d85-821f-6eb2e5c2ff87" />

4. Analyse du trafic d'authentification avec Wireshark
Capture de l'ouverture de session d'un utilisateur du domaine, réalisée sur le contrôleur de domaine.
Analyse des échanges :

DNS : le client cherche le contrôleur de domaine (requêtes SRV).

<img width="1132" height="182" alt="Image" src="https://github.com/user-attachments/assets/e3b1a55f-ea18-43a4-8da4-96ebc15c4ac1" />


LDAP : consultation de l'annuaire Active Directory par le client.


<img width="1275" height="377" alt="Image" src="https://github.com/user-attachments/assets/453db7a3-8e4f-438f-bcdd-ee7ee54616d5" />

Kerberos : authentification de l'utilisateur et obtention des tickets (AS-REQ / AS-REP, TGS-REQ / TGS-REP).

<img width="1142" height="412" alt="Image" src="https://github.com/user-attachments/assets/f946bc94-8d83-4ad4-a89c-81a60ecf3005" />
5. Scan du réseau depuis kali avec nmap

Scan du contrôleur de domaine pour identifier les ports ouverts et les services.
Ports caractéristiques d'un contrôleur de domaine : 53 (DNS), 88 (Kerberos), 389 (LDAP), 445 (SMB).



Le scan révèle les ports typiques d'un contrôleur de domaine Active Directory  
**53** (DNS) : localisation du contrôleur de domaine par les postes
//**88** (Kerberos) : authentification des utilisateurs
//**389** (LDAP) : consultation de l'annuaire
//**445** (SMB) : accès aux partages SYSVOL et NETLOGON (GPO, scripts)

<img width="891" height="431" alt="Image" src="https://github.com/user-attachments/assets/170c6575-165f-4c2d-b3a7-bba1db3801c8" />




6. Gestion des accès et GPO
Groupes de sécurité par service (Support, RH, Compta, Direction) et un dossier partagé par service, avec droits de partage et NTFS donnés uniquement au groupe concerné.
GPO de préférences : mappage automatique d'un lecteur réseau par service, limité par ciblage au groupe de sécurité.
filtrage des dossiers partagé : pas de fichier .exe ou de fichier audio autorisés

<img width="951" height="777" alt="image" src="https://github.com/user-attachments/assets/acd83327-5555-4602-9a5c-b49ca53cb084" />


<img width="961" height="652" alt="image" src="https://github.com/user-attachments/assets/569f7585-bdd8-41a0-afc6-2829510933ce" />


<img width="777" height="510" alt="image" src="https://github.com/user-attachments/assets/f38eaa4d-718a-4443-ab5e-8865dde20052" />


| Groupe | Lecteur | Partage |
|---|---|---|
| Support | S: | \\SRV-GIA01\Support |
| RH | R: | \\SRV-GIA01\RH |
| Compta | Z: | \\SRV-GIA01\Compta |
| Direction | W: | \\SRV-GIA01\Direction |


7. Service DHCP
Rôle DHCP installé sur le contrôleur de domaine et autorisé dans Active Directory, étendue 192.168.174.200 – 192.168.174.250.
Options distribuées aux clients : serveur DNS 192.168.174.142 et domaine labgia.lan.
Test : le client reçoit une adresse de l'étendue, bail visible sur le serveur.

DHCP 
DORA

<img width="831" height="417" alt="image" src="https://github.com/user-attachments/assets/870c3af8-b7a4-47b4-a246-8dcc45b965e0" />

<img width="795" height="410" alt="image" src="https://github.com/user-attachments/assets/8ee8af5e-aab6-4700-8522-64795b9d162d" />

- Création d'une étendue `192.168.174.200` – `192.168.174.250` (masque `255.255.255.0`).
- Options distribuées aux clients : serveur DNS `192.168.174.142` et domaine `labgia.lan`, sans passerelle (réseau isolé).


- Test : le client Windows 11, passé en adressage automatique, reçoit une adresse de l'étendue 

8. Réservation DHCP
Adresse réservée pour le poste client, associée à son adresse MAC.
Test : après ipconfig /release puis /renew, le client reçoit toujours l'adresse réservée.
<img width="835" height="535" alt="image" src="https://github.com/user-attachments/assets/ee474be1-d6e2-4f9f-8b4a-dbc5a2616074" />

9. Politique de mots de passe et verrouillage de compte
GPO appliquée à la racine du domaine (Default Domain Policy) : longueur minimale, complexité, verrouillage du compte après 5 échecs.
Test : compte verrouillé après 5 mauvais mots de passe, événements 4625 (échec de connexion) et 4740 (verrouillage) relevés dans l'Observateur d'événements.

<img width="627" height="440" alt="image" src="https://github.com/user-attachments/assets/97f1a8b4-f9c6-4edc-964c-50c7fe9913e6" />
<img width="660" height="546" alt="image" src="https://github.com/user-attachments/assets/63b53f62-fc8e-4d59-a260-257c960afa8e" />
<img width="697" height="690" alt="image" src="https://github.com/user-attachments/assets/e4e2cf99-8b35-4e36-8977-bdc5915e2277" />

### 10. Intégration d'un poste Linux au domaine
- Jonction de la Debian au domaine `labgia.lan` avec `realmd`, `adcli` et `sssd` (Kerberos pour l'authentification, LDAP pour l'annuaire).
- Prérequis réglés : DNS sur le contrôleur de domaine (`192.168.174.142`), nom de machine `debian01.labgia.lan` et heure synchronisée avec le serveur.
- Vérification : `realm list` indique `kerberos-member`, `sssctl domain-status` affiche `Online`, et `id utilisateur@labgia.lan` renvoie l'identité et les groupes Active Directory.
- Test : ouverture de session avec un compte du domaine (`su - utilisateur@labgia.lan`), dossier personnel créé automatiquement.

<img width="511" height="190" alt="image" src="https://github.com/user-attachments/assets/a1b81546-d92a-4e1b-916e-51960d57d29f" />
<img width="566" height="452" alt="image" src="https://github.com/user-attachments/assets/b35d7dea-3480-4ca3-b065-435db7b1db51" />

## Ce que j'ai appris

- **DNS et Active Directory** : un poste trouve le contrôleur de domaine grâce au DNS. Une faute dans l'adresse DNS du client (`192.18.174.142` au lieu de `192.168.174.142`) l'empêchait de rejoindre le domaine, diagnostiquée avec `ipconfig /all` et `nslookup`.
- **Kerberos et LDAP** : Kerberos prouve l'identité sans envoyer le mot de passe, alors que LDAP permet de consulter l'annuaire.
- **Droits d'accès** : les droits d'un dossier partagé s'appliquent à deux niveaux (partage et NTFS), et le plus restrictif l'emporte.
- **GPO** : le ciblage au niveau de l'élément limite un lecteur à un groupe ; sans ciblage, il s'affiche pour tous.
- **Linux dans le domaine** : la jonction crée l'objet ordinateur, et c'est `sssd` qui rend les comptes du domaine utilisables.
- **Réseau virtuel** : le sous-réseau d'un réseau VMware doit éviter celui du réseau NAT de l'hôte.

## Difficultés rencontrées

| Problème | Cause | Résolution |
|---|---|---|
| Le client ne trouvait pas le domaine | Adresse DNS mal saisie sur le client | Correction dans les propriétés IPv4, vérifiée avec `nslookup` |
| Le lecteur réseau n'apparaissait pas | Ciblage de la GPO vide ou sur le mauvais groupe | Ciblage recréé avec le bon groupe de sécurité |
| Le dossier partagé était inaccessible | Droits NTFS supprimés par erreur en désactivant l'héritage | Restauration avec `takeown` et `icacls` |
| `id` ne reconnaissait pas les comptes du domaine sous Debian | `sssd` ne démarrait pas sans `/etc/sssd/sssd.conf` | Création du fichier (droits `600`) puis démarrage du service |

## Prochaines étapes

- [x] Service DHCP et réservation
- [x] GPO (lecteurs réseau, mots de passe)
- [x] Poste Linux joint au domaine
- [ ] SSH par clé sur la Debian
- [ ] Pare-feu (pfSense)

## Note

Lab réalisé à des fins d'apprentissage, sur un réseau isolé. Les captures ne contiennent ni mot de passe ni donnée personnelle.
