# Architecture du projet

Le projet est composé de trois parties principales :

- **Site Web** : interface utilisée par les clients et les administrateurs.
- **API ASP.NET Core** : fait le lien entre le site Web et la base de données.
- **MariaDB** : contient les produits, clients, commandes et autres données.

```mermaid
graph LR
    A["Site Web"] -->|"HTTP / JSON"| B["API ASP.NET Core"]
    B -->|"Entity Framework Core"| C[("MariaDB")]
```

---

# Base de données

La base de données MariaDB contient les informations nécessaires au fonctionnement de la boutique.

```mermaid
erDiagram

    ADMIN {
        INT id PK
        VARCHAR nom
        VARCHAR password_hash
    }

    ARTICLE {
        INT id PK
        VARCHAR nom
        INT id_categorie FK
        DECIMAL prix_actuel
        VARCHAR description
        VARCHAR image_url
        BOOLEAN est_actif
        INT quantite
    }

    CATEGORIE_PRODUIT {
        INT id PK
        VARCHAR nom
    }

    LIGNE_COMMANDE {
        INT id PK
        INT id_commande FK
        INT id_article FK
        DECIMAL prix_unitaire
        INT quantite
    }

    COMMANDE {
        INT id PK
        DATETIME date_creation
        DATETIME date_terminee
        DECIMAL total
        INT id_client FK
        INT id_statut FK
    }

    STATUT_COMMANDE {
        INT id PK
        VARCHAR nom
    }

    CLIENT {
        INT id PK
        VARCHAR nom
        VARCHAR prenom
        VARCHAR courriel
        VARCHAR telephone
    }

    CATEGORIE_PRODUIT ||--o{ ARTICLE : contient
    ARTICLE ||--o{ LIGNE_COMMANDE : concerne
    COMMANDE ||--|{ LIGNE_COMMANDE : contient
    CLIENT ||--o{ COMMANDE : passe
    STATUT_COMMANDE ||--o{ COMMANDE : possede
```

---

# API

L'API ASP.NET Core permet au site Web de communiquer avec la base de données.

Les endpoints sont séparés selon leur niveau de sécurité :

- **PUBLIC** : accessible sans authentification.
- **ADMIN** : nécessite un JWT administrateur.
- **ProtecClients** : accès protégé aux informations d'une commande client.

## Authentification

```mermaid
flowchart LR

    AUTH["Authentification Admin"]

    AUTH --> LOGIN["POST /api/auth/login
    PUBLIC
    Connexion employe/admin
    Retourne JWT"]
```

## Articles

```mermaid
flowchart TB

    ARTICLES["Articles"]

    ARTICLES --> GET_ARTICLES["GET /api/articles
    PUBLIC
    Afficher les produits disponibles"]

    ARTICLES --> GET_ARTICLE["GET /api/articles/{id}
    PUBLIC
    Afficher un produit"]

    ARTICLES --> CREATE_ARTICLE["POST /api/articles
    ADMIN
    Ajouter un produit"]

    ARTICLES --> UPDATE_ARTICLE["PUT /api/articles/{id}
    ADMIN
    Modifier produit / prix / stock"]

    ARTICLES --> DISABLE_ARTICLE["PATCH /api/articles/{id}/actif
    ADMIN
    Activer / désactiver un produit"]
```

## Commandes

Les clients peuvent passer une commande et ensuite consulter **leur propre commande** pour voir son statut.

Les administrateurs peuvent consulter toutes les commandes et modifier leur statut.

```mermaid
flowchart TB

    COMMANDES["Commandes"]

    COMMANDES --> CREATE_COMMANDE["POST /api/commandes
    PUBLIC
    Passer une commande"]

    COMMANDES --> CLIENT_COMMANDE["GET /api/commandes/{id}/suivi
    ProtecClients
    Voir sa propre commande
    et son statut"]

    COMMANDES --> GET_COMMANDE["GET /api/commandes/{id}
    ADMIN
    Voir une commande complète"]

    COMMANDES --> GET_COMMANDES["GET /api/commandes
    ADMIN
    Voir toutes les commandes"]

    COMMANDES --> STATUT_COMMANDE["PATCH /api/commandes/{id}/statut
    ADMIN
    Modifier le statut"]
```

> **ProtecClients** doit vérifier que le client a le droit de consulter cette commande. Le simple fait de connaître l'ID d'une commande ne doit pas permettre de voir les informations d'un autre client.

## Catégories

```mermaid
flowchart TB

    CATEGORIES["Catégories"]

    CATEGORIES --> GET_CATEGORIES["GET /api/categories
    PUBLIC
    Afficher les catégories"]

    CATEGORIES --> CREATE_CATEGORIE["POST /api/categories
    ADMIN
    Ajouter une catégorie"]

    CATEGORIES --> UPDATE_CATEGORIE["PUT /api/categories/{id}
    ADMIN
    Modifier une catégorie"]
```

---

# Navigation du site Web

Le site Web possède une partie publique pour commander et suivre une commande ainsi qu'une partie réservée aux administrateurs.

```mermaid
flowchart TB

    ACCUEIL["Accueil"]

    PRODUITS["Boutique / Produits"]
    DETAIL["Détail produit"]
    PANIER["Panier"]
    CHECKOUT["Informations client"]
    CONFIRMATION["Confirmation commande"]

    SUIVI["Suivre ma commande"]
    COMMANDE_CLIENT["Ma commande
    Statut
    Articles
    Total"]

    ADMIN_LOGIN["Connexion Admin"]
    ADMIN["Dashboard Admin"]
    ADMIN_COMMANDES["Gestion commandes"]
    ADMIN_PRODUITS["Gestion produits"]
    ADMIN_CATEGORIES["Gestion catégories"]

    ACCUEIL --> PRODUITS
    ACCUEIL --> SUIVI

    PRODUITS --> DETAIL
    PRODUITS --> PANIER
    DETAIL --> PANIER

    PANIER --> CHECKOUT
    CHECKOUT --> CONFIRMATION

    CONFIRMATION --> COMMANDE_CLIENT
    SUIVI --> COMMANDE_CLIENT

    ACCUEIL --> ADMIN_LOGIN
    ADMIN_LOGIN --> ADMIN

    ADMIN --> ADMIN_COMMANDES
    ADMIN --> ADMIN_PRODUITS
    ADMIN --> ADMIN_CATEGORIES
```

---

# Structure générale

```text
Client
  │
  ▼
Site Web
  │
  │ HTTP / JSON
  ▼
API ASP.NET Core
  │
  │ Entity Framework Core
  │ Database First / Scaffold
  ▼
MariaDB
```

Le client peut commander sans accéder à l'administration. Après sa commande, il peut utiliser la fonction **Suivre ma commande** afin de consulter uniquement sa propre commande.

L'administrateur doit s'authentifier avec un **JWT** pour gérer les produits, les catégories et les commandes.

_____________
