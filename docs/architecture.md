# Architecture de SHELLGATE

## Composants de l'architecture

SHELLGATE repose actuellement sur plusieurs composants :

* Docker
* Python
* FastAPI
* PostgreSQL
* OpenSSH

### Docker

Docker permet de conteneuriser les différents composants de SHELLGATE afin de faciliter leur déploiement, leur isolation et leur environnement d'exécution.

Pour le développement, Docker Compose sera utilisé afin de gérer les différents conteneurs et leur réseau.

### Python

Python est le langage principal utilisé pour développer SHELLGATE.

Il sera utilisé pour développer la logique métier de l'application ainsi que les différents services du backend.

### FastAPI

FastAPI constitue le point d'entrée du backend de SHELLGATE.

Il reçoit les requêtes HTTP, les valide et les route vers les fonctionnalités correspondantes de l'application.

FastAPI sera notamment utilisé pour exposer l'API permettant de :

* gérer les utilisateurs ;
* gérer les serveurs ;
* gérer les autorisations ;
* gérer les sessions ;
* consulter les journaux d'audit.

### PostgreSQL

PostgreSQL est utilisé comme système de gestion de base de données.

Il permettra notamment de stocker les informations relatives :

* aux utilisateurs ;
* aux rôles ;
* aux serveurs ;
* aux autorisations ;
* aux sessions ;
* aux journaux d'audit.

Les données sensibles ne devront pas être stockées en clair.

### OpenSSH

OpenSSH sera utilisé pour établir les connexions SSH entre SHELLGATE et les machines administrées.

Le principe de fonctionnement sera le suivant :

```text
Administrateur
      │
      │
      ▼
  SHELLGATE
      │
      │ SSH
      ▼
Machine administrée
```

L'administrateur ne doit pas avoir besoin d'établir directement une connexion SSH vers la machine cible. SHELLGATE constitue le point intermédiaire permettant de contrôler et de journaliser l'accès.

## Architecture simplifiée

```text
                    Administrateur
                           │
                           │ HTTP/HTTPS
                           ▼
                  ┌─────────────────┐
                  │    SHELLGATE    │
                  │                 │
                  │     FastAPI     │
                  │        │        │
                  │     Python      │
                  └───────┬─┴───────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
        ┌───────────┐           ┌───────────┐
        │ PostgreSQL│           │  OpenSSH  │
        │           │           │           │
        └───────────┘           └─────┬─────┘
                                      │
                                      │ SSH
                                      ▼
                              Machine administrée

              Docker / Docker Compose
              gère l'environnement
              d'exécution
```

## Principe général

SHELLGATE sera conçu comme un backend monolithique dans un premier temps.

L'objectif est de conserver une architecture simple afin de faciliter le développement, les tests et l'apprentissage.

Les différents composants seront séparés par responsabilité :

* **FastAPI** : gestion des requêtes HTTP et exposition de l'API ;
* **Python** : logique applicative ;
* **PostgreSQL** : persistance des données ;
* **OpenSSH** : communication SSH avec les machines administrées ;
* **Docker** : conteneurisation et environnement d'exécution.

Cette architecture pourra évoluer au fur et à mesure que de nouvelles fonctionnalités seront ajoutées.
