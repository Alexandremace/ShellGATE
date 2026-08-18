Modélisation des données

Cette première modélisation définit les principales entités utilisées par SHELLGATE dans le cadre de la V0.1.

Le modèle est volontairement simple et pourra évoluer avec l'ajout de nouvelles fonctionnalités.

User

Représente un utilisateur de SHELLGATE.

Un utilisateur peut appartenir à un ou plusieurs groupes.

Informations
id : identifiant unique
username : nom d'utilisateur
password_hash : hash du mot de passe
is_active : indique si le compte est actif
created_at : date de création du compte
Relations

Un utilisateur peut appartenir à plusieurs groupes.

User
  │
  └── N:N ── Group
Group

Représente un groupe d'utilisateurs.

Les groupes permettent de regrouper les utilisateurs selon leur fonction ou leurs responsabilités.

Exemples :

admin
tech
auditor

Un groupe peut être associé à plusieurs jeux d'accès.

Informations
id : identifiant unique
name : nom du groupe
description : description du groupe
created_at : date de création
Relations
Group
  │
  ├── N:N ── User
  │
  └── N:N ── AccessSet
AccessSet

Représente un jeu d'accès.

Un jeu d'accès regroupe un ensemble de serveurs auxquels un groupe peut accéder.

Exemples :

AccessSet: Linux
    ├── srv-linux-01
    └── srv-linux-02

AccessSet: Production
    ├── srv-linux-01
    ├── srv-linux-02
    └── srv-windows-01

Un jeu d'accès peut être associé à plusieurs groupes et un serveur peut appartenir à plusieurs jeux d'accès.

Informations
id : identifiant unique
name : nom du jeu d'accès
description : description du jeu d'accès
created_at : date de création
Relations
AccessSet
  │
  ├── N:N ── Group
  │
  └── N:N ── Server
Server

Représente une machine administrée par SHELLGATE.

Pour la V0.1, le protocole principal est SSH.

Le support RDP sera étudié ultérieurement.

Informations
id : identifiant unique
name : nom du serveur
hostname : hostname ou nom DNS
ip_address : adresse IP
ssh_port : port SSH
ssh_username : utilisateur utilisé pour la connexion SSH
is_active : indique si le serveur peut être utilisé
created_at : date d'ajout du serveur
Relations

Un serveur peut appartenir à plusieurs jeux d'accès.

Server
  │
  └── N:N ── AccessSet
Tables de liaison

Les relations plusieurs-à-plusieurs seront représentées par des tables de liaison.

UserGroup

Associe les utilisateurs aux groupes.

User
  │
  └── UserGroup ── Group
Informations
user_id : utilisateur
group_id : groupe
GroupAccessSet

Associe les groupes aux jeux d'accès.

Group
  │
  └── GroupAccessSet ── AccessSet
Informations
group_id : groupe
access_set_id : jeu d'accès
AccessSetServer

Associe les jeux d'accès aux serveurs.

AccessSet
  │
  └── AccessSetServer ── Server
Informations
access_set_id : jeu d'accès
server_id : serveur
Session

Représente une session d'administration ouverte par un utilisateur.

Une session est associée à un utilisateur et à un serveur cible.

Informations
id : identifiant unique
user_id : utilisateur ayant ouvert la session
server_id : serveur cible
type : type de session
started_at : date et heure de début
ended_at : date et heure de fin
source_ip : adresse IP de l'administrateur
status : état de la session
Types de session

Pour la V0.1 :

SSH

D'autres types pourront être ajoutés ultérieurement :

RDP
HTTPS
SFTP
AuditLog

Représente une entrée dans le journal d'audit de SHELLGATE.

L'objectif est de pouvoir déterminer :

qui a effectué une action ;
quand l'action a été effectuée ;
sur quelle ressource ;
quelle action a été effectuée ;
quel a été le résultat.
Informations
id : identifiant unique
user_id : utilisateur ayant effectué l'action
session_id : session associée, si applicable
action : action effectuée
target : ressource concernée
timestamp : date et heure
result : résultat de l'action
Exemples d'actions
LOGIN
LOGOUT
CONNECT
DISCONNECT
CREATE_USER
DELETE_USER
CREATE_SERVER
UPDATE_SERVER
UPDATE_ACCESS

Le système d'audit devra être conçu de manière à empêcher la modification arbitraire des traces par un utilisateur non autorisé.

Modèle global

Le fonctionnement général peut être résumé ainsi :

                         ┌──────────┐
                         │   User   │
                         └────┬─────┘
                              │
                              │ N:N
                              ▼
                         ┌──────────┐
                         │  Group   │
                         └────┬─────┘
                              │
                              │ N:N
                              ▼
                       ┌─────────────┐
                       │  AccessSet  │
                       └──────┬──────┘
                              │
                              │ N:N
                              ▼
                         ┌──────────┐
                         │  Server  │
                         └──────────┘


        User ──────────────── Session ──────────────── Server
          │
          │
          ▼
      AuditLog
Exemple concret

Un utilisateur peut appartenir à plusieurs groupes :

Alice
 ├── admin
 └── tech

Le groupe admin peut avoir accès au jeu d'accès Production :

admin
 └── Production
      ├── srv-linux-01
      ├── srv-linux-02
      └── srv-windows-01

Le groupe tech peut avoir accès au jeu d'accès Linux :

tech
 └── Linux
      ├── srv-linux-01
      └── srv-linux-02

SHELLGATE peut alors déterminer automatiquement les serveurs accessibles à Alice en parcourant ses groupes puis leurs jeux d'accès.

Alice
 │
 ├── admin
 │    └── Production
 │         ├── srv-linux-01
 │         ├── srv-linux-02
 │         └── srv-windows-01
 │
 └── tech
      └── Linux
           ├── srv-linux-01
           └── srv-linux-02

Alice possède donc un accès aux trois serveurs.

Principes de conception

Le modèle repose sur plusieurs principes :

Un utilisateur peut appartenir à plusieurs groupes.
Un groupe peut être associé à plusieurs jeux d'accès.
Un jeu d'accès peut être associé à plusieurs serveurs.
Un serveur peut appartenir à plusieurs jeux d'accès.
Les accès sont donc attribués indirectement via les groupes et les jeux d'accès.
Les mots de passe ne sont jamais stockés en clair.
Les relations entre les entités seront représentées par des clés étrangères dans PostgreSQL.
Les fonctionnalités non nécessaires au MVP seront ajoutées ultérieurement.
Évolution future

Le modèle pourra être enrichi pour prendre en charge :

différents niveaux de privilèges ;
permissions SSH spécifiques ;
accès en lecture seule ;
accès temporaires ;
expiration d'une autorisation ;
MFA ;
LDAP / Active Directory ;
RDP ;
SFTP ;
gestion des clés SSH ;
gestion des secrets ;
approbation d'une demande d'accès.

Ces fonctionnalités ne font pas partie du modèle initial et seront ajoutées lorsque leur besoin sera clairement défini. 

Tables:

User:
- id  entier
- username  texte
- password_hash  texte
- is_active. booleen
- created_at. date/heure


Server:
- id entier
- name texte
- ip_address texte
- ssh_port. entier
- ssh_username texte
- is_active. booleen
- created_at. date/heure


User_groups:
- id. entier
- name  texte
- description. texte
- created_at. date/heure


Server_groups:
- id  entier
- name texte
- description texte
- created_at. date/heure


Access_sets:
- id. entier
- name texte
- user_group_id entier
- server_group_id entier
- created_at date/heure

user_group_members:
- user_id entier
- user_group_id entier


server_group_members:
- server_id entier 
- server_group_id entier
