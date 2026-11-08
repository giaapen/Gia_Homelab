# Gia_Homelab
Homelab Infrastructure &amp; Sécurité (VMware Workstation) Active Directory / Windows Server 2022 (AD DS, DNS)


# Homelab Infrastructure & Sécurité (VMware Workstation)

Homelab réalisé pour pratiquer l'administration systèmes et réseaux et les bases de la sécurité : un domaine Active Directory, un poste client et une machine d'analyse, sur un réseau virtuel isolé.

## Schéma


<img width="2720" height="1360" alt="Image" src="https://github.com/user-attachments/assets/cba25f30-2082-43e8-9ba6-28dd2c5e23e0" />

## Machines

| Machine | Système | Rôle | Adresse IP |
|---|---|---|---|
| SRV-GIA01 | Windows Server 2022 | Contrôleur de domaine (AD DS, DNS) | 192.168.174.142 |
| PC-CLIENTLAB01 | Windows 11 Enterprise (évaluation) | Poste client joint au domaine | 192.168.174.150 |
| Kali | Kali Linux | Scans et analyse réseau | 192.168.174.129 |
| Debian | Debian | Machine Linux du lab | 192.168.174.130 |

Réseau : VMnet2 (Host-only), 192.168.174.0/24, sans passerelle ni accès à Internet.
Domaine : `labgia.lan`

## Ce que j'ai réalisé

### 1. Réseau virtuel isolé
- Création d'un réseau Host-only (VMnet2) sous VMware Workstation.
- Adressage IP fixe pour chaque machine, tests de connectivité avec `ping`.

### Tests de connectivité (ping)

| Depuis \ Vers | SRV-GIA01 (192.168.174.142) | PC-CLIENTLAB01 (192.168.174.150) | Kali (192.168.174.129) | Debian (192.168.174.130) |
|---|:---:|:---:|:---:|:---:|
| **SRV-GIA01** | n/a | ✅ | ✅ | ✅ |
| **PC-CLIENTLAB01** | ✅ | n/a | ✅ | ✅ |
| **Kali** | ✅ | ✅ | n/a | ✅ |
| **Debian** | ✅ | ✅ | ✅ | n/a |

<img width="557" height="155" alt="Image" src="https://github.com/user-attachments/assets/92f2bf34-09ad-4002-af94-8a8c07c707ee" />

### 2. Active Directory
- Installation du rôle AD DS et promotion en contrôleur de domaine (`labgia.lan`), avec DNS intégré.

<img width="945" height="787" alt="Image" src="https://github.com/user-attachments/assets/99f07cae-1904-4a88-a4a2-5fbe4268cbea" />


- Création d'unités d'organisation, d'utilisateurs et d'un groupe.

<img width="945" height="717" alt="Image" src="https://github.com/user-attachments/assets/753b9c1f-050e-4ecb-a063-224d86f22a7d" />



### 3. Jonction d'un poste client au domaine
- Configuration du DNS du client sur le contrôleur de domaine.
- Jonction du poste Windows 11 au domaine, connexion avec un compte du domaine. (sysdm.cpl)

<img width="945" height="727" alt="Image" src="https://github.com/user-attachments/assets/c6c896ce-f582-4d85-821f-6eb2e5c2ff87" />

### 4. Analyse du trafic d'authentification avec Wireshark
- Capture de l'ouverture de session d'un utilisateur du domaine.
- Analyse des échanges : DNS (recherche du contrôleur), Kerberos (authentification), LDAP (consultation de l'annuaire).




**DNS** : le client interroge le DNS pour localiser le contrôleur de domaine (requêtes SRV).
<img width="1132" height="182" alt="Image" src="https://github.com/user-attachments/assets/e3b1a55f-ea18-43a4-8da4-96ebc15c4ac1" />


**LDAP** : consultation de l'annuaire Active Directory par le client.

<img width="1275" height="377" alt="Image" src="https://github.com/user-attachments/assets/453db7a3-8e4f-438f-bcdd-ee7ee54616d5" />

**Kerberos** : authentification de l'utilisateur et obtention des tickets (AS-REQ / AS-REP, TGS-REQ / TGS-REP).

<img width="1142" height="412" alt="Image" src="https://github.com/user-attachments/assets/f946bc94-8d83-4ad4-a89c-81a60ecf3005" />

### 5. Scan réseau depuis Kali avec Nmap
- Scan du contrôleur de domaine pour identifier les ports ouverts et les services.

<img width="891" height="431" alt="Image" src="https://github.com/user-attachments/assets/170c6575-165f-4c2d-b3a7-bba1db3801c8" />
Le scan révèle les ports typiques d'un contrôleur de domaine Active Directory :
- **53** (DNS) : localisation du contrôleur de domaine par les postes
- **88** (Kerberos) : authentification des utilisateurs
- **389** (LDAP) : consultation de l'annuaire
- **445** (SMB) : accès aux partages SYSVOL et NETLOGON (GPO, scripts)



### 6. Gestion des accès par groupes et GPO
- Création de groupes de sécurité par service (Support, RH, Compta, Direction) et d'un dossier partagé par service.
- Droits NTFS et de partage donnés uniquement au groupe concerné.
- GPO de préférences : mappage automatique d'un lecteur réseau par service, limité par ciblage au groupe de sécurité.

<img width="951" height="777" alt="image" src="https://github.com/user-attachments/assets/acd83327-5555-4602-9a5c-b49ca53cb084" />

filtrage des dossiers partagés
<img width="961" height="652" alt="image" src="https://github.com/user-attachments/assets/569f7585-bdd8-41a0-afc6-2829510933ce" />

mappages de lecteurs 
<img width="916" height="522" alt="image" src="https://github.com/user-attachments/assets/a3a37d96-fb65-4f77-b387-ebc7264c48f7" />

| Groupe | Lecteur | Partage |
|---|---|---|
| Support | S: | \\SRV-GIA01\Support |
| RH | R: | \\SRV-GIA01\RH |
| Compta | T: | \\SRV-GIA01\Compta |
| Direction | G: | \\SRV-GIA01\Direction |

## Ce que j'ai appris

- Kerberos prouve l'identité de l'utilisateur sans envoyer le mot de passe, alors que LDAP permet de consulter l'annuaire.
- Le sous-réseau d'un réseau virtuel VMware doit être choisi pour ne pas entrer en conflit avec le réseau NAT de l'hôte.

## Prochaines étapes

- [ ] Ajouter le service DHCP
- [ ] Créer des GPO
- [ ] Ajouter un pare-feu (pfSense)

## Note

Lab réalisé à des fins d'apprentissage, sur un réseau isolé. Les captures ne contiennent ni mot de passe ni donnée personnelle.
