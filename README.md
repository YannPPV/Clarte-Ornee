# Clarté Ornée

Site e-commerce de luminaires développé en PHP natif, avec une base de données MySQL/MariaDB locale.
Il présente un catalogue de lampes classées par catégorie, et chaque lampe a sa fiche détaillée. Le site gère aussi les comptes clients, un panier, une page de paiement (simulée) et un formulaire de service après-vente (SAV).

---

## Sommaire

- [Fonctionnalités](#fonctionnalités)
- [Technologies](#technologies)
- [Prérequis](#prérequis)
- [Installation](#installation)
- [Configuration](#configuration)
- [Architecture du projet](#architecture-du-projet)
- [Routage](#routage)
- [Base de données](#base-de-données)
- [Sécurité](#sécurité)
- [Auteurs](#auteurs)

---

## Fonctionnalités

| Fonctionnalité | Page | Connexion requise |
|---|---|---|
| Catalogue des lampes, regroupées par type (plafond, murale, sol, chevet, extérieur) | `home` | Non |
| Fiche produit : prix, puissance, couleur, style, matériau, dimensions, description | `lamp-details` | Non |
| Ajout d'une lampe au panier | `lamp-details` | Oui |
| Panier : liste des articles, suppression, total | `basket` | Oui |
| Paiement simulé : adresse de livraison et carte bancaire, puis vidage du panier | `achat` | Oui |
| Création de compte | `inscription` | Non |
| Connexion / déconnexion | `login` / `logout` | Non |
| Demande au service client (SAV), avec numéro de commande facultatif | `customers-services` | Oui |
| Pages d'information : FAQ, contact, à propos, mentions légales | `FAQ`, `contact`, `about`, `legal_disclaimer` | Non |

> Le paiement est **fictif** : aucune donnée bancaire n'est enregistrée ni transmise. Valider le formulaire vide simplement le panier du client.

---

## Technologies

- **Back-end :** PHP 8 (sans framework), PDO
- **Base de données :** MySQL / MariaDB
- **Front-end :** HTML5 et CSS3 (une feuille de style par page)
- **Serveur local :** WampServer (Apache + MySQL + PHP)

---

## Prérequis

- Windows 10 ou supérieur, avec les droits administrateur
- [WampServer](https://www.wampserver.com/) en 64 bits (recommandé), avec les *Microsoft Visual C++ Redistributables*
- PHP **8.0 ou plus**, avec l'extension `pdo_mysql` activée (elle l'est par défaut sous WAMP)
- MySQL ou MariaDB
- Git
- Un navigateur web

---

## Installation

### 1. Démarrer WampServer

1. Lancer WampServer.
2. Attendre que l'icône de la barre des tâches passe au **vert**.
3. Vérifier que http://localhost s'affiche.

### 2. Cloner le projet

Ouvrir un terminal dans `C:\wamp64\www`, puis :

```bash
git clone https://github.com/YannPPV/Projet-transversale-web.git
```

### 3. Créer la base de données

1. Ouvrir phpMyAdmin : http://localhost/phpmyadmin
2. Se connecter avec l'utilisateur `root`, sans mot de passe, sur le serveur **MySQL**.
3. Ouvrir l'onglet **SQL** et exécuter, **dans cet ordre** :
   1. le contenu de [base_de_donnees/lampes.sql](base_de_donnees/lampes.sql), qui crée la base `clarte_Ornee` et ses tables ;
   2. le contenu de [base_de_donnees/bddtest.sql](base_de_donnees/bddtest.sql), qui ajoute le catalogue de 50 lampes (10 par catégorie).

> ⚠️ `lampes.sql` commence par `DROP DATABASE IF EXISTS clarte_Ornee;`. Le relancer **efface toutes les données existantes**, y compris les comptes clients, les paniers et les demandes SAV.

### 4. Lancer le site

Ouvrir dans le navigateur :

```
http://localhost/Projet-transversale-web/index.php
```

---

## Configuration

Les paramètres de connexion à la base se trouvent dans [dbConnect.php](dbConnect.php) :

```php
$dbConfig = [
    'db_sgbd'     => 'mysql',        // SGBD utilisé
    'db_host'     => 'localhost',    // Adresse du serveur de base de données
    'db_name'     => 'Clarte_Ornee', // Nom de la base
    'db_username' => 'root',         // Utilisateur MySQL
    'db_password' => '',             // Mot de passe (vide par défaut sous WAMP)
];
```

La connexion PDO est configurée en UTF-8, avec le mode d'erreur `ERRMODE_EXCEPTION` et la récupération par défaut `FETCH_ASSOC`.

> Ces identifiants ne conviennent qu'à un environnement local. Il faut les changer avant tout déploiement.

---

## Architecture du projet

```
Projet-transversale-web/
├── index.php                  # Point d'entrée unique (routeur)
├── dbConnect.php              # Connexion PDO à la base
├── assets/
│   ├── css/                   # Une feuille de style par page + navbar/footer
│   └── images/                # Photos des lampes, logos, icônes de paiement
├── base_de_donnees/
│   ├── lampes.sql             # Création de la base et des tables
│   └── bddtest.sql            # Jeu de données : catalogue de lampes
├── models/                    # Accès aux données et traitements
│   ├── lampes.php             # Chargement des lampes et des catégories
│   ├── select-details.php     # Chargement d'une lampe par son id
│   ├── basket.php             # Lecture du panier et suppression d'un article
│   └── add.php                # Inscription d'un client
└── views/
    ├── partials/
    │   ├── _navbar.php        # En-tête et menu (affiche Connexion ou Déconnexion)
    │   └── _footer.php        # Pied de page et liens d'information
    └── pages/                 # Une vue par page (home, login, basket, achat…)
```

---

## Routage

Toutes les pages passent par [index.php](index.php), qui lit le paramètre `page` dans l'URL :

```
index.php?page=<nom>
```

- Sans paramètre, la page `home` est affichée.
- `index.php` charge `assets/css/<nom>.css`, puis inclut la navbar, `views/pages/<nom>.php` et le footer.
- La fiche produit reçoit aussi l'identifiant de la lampe : `index.php?page=lamp-details&id=12`.

L'utilisateur connecté est suivi dans la session PHP : `$_SESSION['authenticated']` et `$_SESSION['pseudo']`, qui contient l'adresse e-mail du client.

---

## Base de données

Base : `clarte_Ornee` (encodage `utf8mb4`, moteur InnoDB).

| Table | Colonnes | Rôle |
|---|---|---|
| **Lamps** | `idLamp`, `name`, `TypeLamp`, `Power`, `Color`, `Price`, `Style`, `Material`, `Dimensions`, `Description`, `Quantity`, `PathPicture` | Catalogue des luminaires |
| **Customers** | `idCustomer`, `Name`, `Lastname`, `Email`, `Street`, `City`, `Country`, `PostCode`, `NumberPhone`, `password` | Comptes clients (mot de passe haché) |
| **Baskets** | `idBasket`, `idLamp` → Lamps, `idCustomer` → Customers | Articles du panier (une ligne par article ajouté) |
| **CustomerService** | `idCustomerService`, `idCustomer` → Customers, `Request`, `OrderNumber` | Demandes adressées au SAV |

```
Customers 1 ──── n Baskets n ──── 1 Lamps
Customers 1 ──── n CustomerService
```

---

## Sécurité

- Les requêtes SQL sont préparées avec PDO (paramètres nommés).
- Les mots de passe sont hachés avec `password_hash()` et vérifiés avec `password_verify()`.
- Les données affichées sont échappées avec `htmlspecialchars()`.
- Les identifiants reçus (`id`, `idLamp`) sont validés avec `filter_input(..., FILTER_VALIDATE_INT)`.
- Les formulaires sont validés côté client (`required`, `pattern`, `maxlength`).
- L'e-mail est vérifié à l'inscription : impossible de créer deux comptes avec la même adresse.
- Le panier, le paiement et le SAV exigent d'être connecté. Sinon, l'utilisateur est redirigé vers `login`.

---

## Auteurs

Projet transversal réalisé par :

- Edely Lola
- Popov Yann
- Lefranc Nathan
- Bourt Théo
