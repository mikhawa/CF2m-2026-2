# 001 - Analyse Initiale du Projet CF2M

**Date:** 27 janvier 2026
**Version:** 1.0
**Auteur:** Claude Code

---

## Table des Matières

1. [Résumé Exécutif](#1-résumé-exécutif)
2. [Structure du Projet](#2-structure-du-projet)
3. [Dépendances Installées](#3-dépendances-installées)
4. [Configuration Docker](#4-configuration-docker)
5. [Configuration Symfony](#5-configuration-symfony)
6. [État du Code Source](#6-état-du-code-source)
7. [Éléments Manquants](#7-éléments-manquants)
8. [Checklist de Développement](#8-checklist-de-développement)
9. [Commandes Utiles](#9-commandes-utiles)

---

## 1. Résumé Exécutif

Le projet CF2M est actuellement en phase de **mise en place initiale** du framework Symfony 7.4 LTS. L'infrastructure Docker est complètement configurée et fonctionnelle, mais **aucune fonctionnalité métier n'a encore été développée**. Le dépôt contient uniquement le squelette Symfony standard avec les configurations de base.

### État Global

| Composant | Statut |
|-----------|--------|
| Infrastructure Docker | Complète |
| Configuration Symfony | Base uniquement |
| Entités Doctrine | Non créées |
| Contrôleurs | Non créés |
| Templates | Minimal |
| Bundles métier | Non installés |

---

## 2. Structure du Projet

```
CF2m-2026-2/
├── .env                      # Configuration environnement dev
├── .env.dev                  # Secret APP généré
├── .env.test                 # Config test
├── CLAUDE.md                 # Instructions pour Claude Code
├── README.md                 # Documentation sommaire
├── composer.json             # Dépendances PHP
├── composer.lock             # Verrouillage des versions
├── symfony.lock              # Configuration recettes Symfony
├── phpunit.dist.xml          # Config tests PHPUnit
├── importmap.php             # Importmap Asset Mapper
├── docker-compose.yml        # Orchestration Docker
├── docker/                   # Fichiers Docker
│   ├── php/                  # Image PHP 8.4-FPM
│   │   ├── Dockerfile
│   │   └── php.ini           # Config PHP personnalisée
│   ├── nginx/                # Image Nginx Alpine
│   │   ├── Dockerfile
│   │   └── default.conf      # Configuration Nginx
│   └── mariadb/              # Image MariaDB 11.4
│       └── Dockerfile
├── config/                   # Configuration Symfony
│   ├── bundles.php           # Enregistrement des bundles
│   ├── services.yaml         # Configuration services DI
│   ├── routes.yaml           # Routage racine
│   └── packages/             # Configuration des bundles
├── src/                      # Code source PHP
│   └── Kernel.php            # Kernel Symfony (minimal)
├── templates/                # Vues Twig
│   └── base.html.twig        # Template de base
├── assets/                   # Ressources frontend
│   ├── app.js                # Entrypoint JavaScript
│   ├── styles/app.css        # Styles
│   └── controllers/          # Contrôleurs Stimulus
├── public/                   # Point d'entrée web
│   └── index.php
├── var/                      # Fichiers runtime (cache, logs)
├── migrations/               # Migrations Doctrine (vide)
├── tests/                    # Tests PHPUnit
├── translations/             # Fichiers i18n
├── docs/                     # Documentation
└── vendor/                   # Dépendances Composer
```

---

## 3. Dépendances Installées

### 3.1 Versions Cibles

| Composant | Version |
|-----------|---------|
| PHP | 8.4 |
| Symfony | 7.4.* LTS |
| MariaDB | 11.4 LTS |
| Node/Assets | Asset Mapper (natif) |

### 3.2 Bundles Symfony Core (14)

- `symfony/framework-bundle`
- `symfony/twig-bundle`
- `symfony/security-bundle`
- `symfony/mailer`
- `symfony/notifier`
- `symfony/validator`
- `symfony/form`
- `symfony/serializer`
- `symfony/asset`
- `symfony/asset-mapper`
- `symfony/string`
- `symfony/translation`
- `symfony/intl`
- `symfony/yaml`

### 3.3 Doctrine (3)

- `doctrine/doctrine-bundle` (^3.2.2)
- `doctrine/doctrine-migrations-bundle` (^4.0)
- `doctrine/orm` (^3.6.1)

### 3.4 Symfony UX (2)

- `symfony/stimulus-bundle` (^2.32)
- `symfony/ux-turbo` (^2.32)

### 3.5 Autres Packages

- `symfony/monolog-bundle` - Logging
- `symfony/doctrine-messenger` - Files d'attente
- `twig/extra-bundle` - Extensions Twig
- `symfony/http-client` - Requêtes HTTP
- `symfony/process` - Exécution processus
- `symfony/expression-language` - Expressions

### 3.6 Dépendances Manquantes (Critiques)

| Bundle | Statut | Utilité |
|--------|--------|---------|
| `easycorp/easyadmin-bundle` | MANQUANT | Backend administration |
| `vich/uploader-bundle` | MANQUANT | Gestion des fichiers uploadés |
| `liip/imagine-bundle` | MANQUANT | Redimensionnement images |
| `stof/doctrine-extensions-bundle` | MANQUANT | Slugs, timestamps automatiques |
| TinyMCE (via importmap) | MANQUANT | Éditeur WYSIWYG |

---

## 4. Configuration Docker

### 4.1 Services Disponibles

| Service | Image | Port Local | Description |
|---------|-------|------------|-------------|
| **php** | PHP 8.4-FPM (custom) | 9000 | Serveur PHP-FPM |
| **nginx** | nginx:alpine | 8080 | Serveur web |
| **mariadb** | mariadb:11.4 | 3307 | Base de données |
| **mailhog** | mailhog/mailhog | 8025 | Capture emails dev |
| **phpmyadmin** | phpmyadmin:latest | 8081 | Interface BDD |

### 4.2 Configuration PHP (php.ini)

- Memory limit: 256M
- Upload max filesize: 20M
- Timezone: Europe/Brussels
- Xdebug: mode trigger, port 9003
- APCu: 64M SHM
- OPcache: activé

### 4.3 Configuration Nginx

- Root: `/var/www/html/public`
- Security headers configurés
- Cache 1 an pour assets statiques
- Blocage: .git, composer.*, .env

### 4.4 Credentials Base de Données

```
Host: mariadb (interne) / localhost:3307 (externe)
Database: cf2m
User: cf2m
Password: cf2m
Root password: root
```

---

## 5. Configuration Symfony

### 5.1 Doctrine (config/packages/doctrine.yaml)

- ORM configuré avec mapping par attributs PHP 8
- URL de connexion MariaDB configurée
- Migrations activées
- Cache query et result pour production

### 5.2 Sécurité (config/packages/security.yaml)

**État actuel:**
- Password hashers: auto
- Provider: users_in_memory (temporaire)

**À configurer:**
- User provider Doctrine
- Form Login
- Rôles: ROLE_ADMIN, ROLE_RESPONSABLE, ROLE_USER
- Hiérarchie des rôles
- Access control

### 5.3 Messenger (config/packages/messenger.yaml)

- Transport "async" avec Doctrine
- SendEmailMessage routé vers async
- Retry: 3 tentatives
- Transport "failed" configuré

### 5.4 Mailer (config/packages/mailer.yaml)

- DSN: smtp://mailhog:1025
- Intégration Messenger active

### 5.5 Translation

- Locale par défaut: EN (à changer en FR)
- Dossier translations/ existant

---

## 6. État du Code Source

### 6.1 Entités Doctrine

**Statut:** Aucune entité créée

**Entités à créer selon le cahier des charges:**

1. **User** - Utilisateurs avec rôles
2. **Formation** - Formations proposées
3. **Registration** - Inscriptions aux formations
4. **Page** - Pages statiques du site
5. **Article** - Articles de blog
6. **Comment** - Commentaires sur articles
7. **Portfolio** - Portfolio utilisateurs
8. **Testimonial** - Témoignages
9. **Partner** - Partenaires
10. **ContactMessage** - Messages de contact
11. **NotificationRecipient** - Destinataires notifications
12. **SiteConfig** - Configuration du site

### 6.2 Contrôleurs

**Statut:** Aucun contrôleur créé

**Contrôleurs à créer:**
- HomeController
- FormationController
- RegistrationController
- ContactController
- ProfileController
- ArticleController
- CommentController

### 6.3 Templates Twig

**État actuel:** Template de base minimal uniquement

```twig
<!DOCTYPE html>
<html>
    <head>
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
        {% block javascripts %}
            {% block importmap %}{{ importmap('app') }}{% endblock %}
        {% endblock %}
    </head>
    <body>
        {% block body %}{% endblock %}
    </body>
</html>
```

### 6.4 Assets Frontend

- app.js configuré avec Stimulus et Turbo
- CSS minimal
- Bootstrap 5.3 non installé

---

## 7. Éléments Manquants

### 7.1 Priorité Critique

| Élément | Chemin | Notes |
|---------|--------|-------|
| Entités Doctrine | src/Entity/ | 12 entités à créer |
| Contrôleurs | src/Controller/ | 7+ contrôleurs |
| EasyAdmin | composer require | Backend admin |
| VichUploader | composer require | Uploads fichiers |
| LiipImagine | composer require | Images |

### 7.2 Priorité Haute

| Élément | Chemin | Notes |
|---------|--------|-------|
| Templates site | templates/ | Layout + pages |
| Migrations | migrations/ | Après entités |
| Bootstrap 5.3 | importmap | CSS framework |
| Configuration sécurité | config/packages/security.yaml | Auth complète |

### 7.3 Priorité Moyenne

| Élément | Chemin | Notes |
|---------|--------|-------|
| FormTypes | src/Form/ | Formulaires |
| DataFixtures | src/DataFixtures/ | Données test |
| Templates emails | templates/emails/ | Notifications |
| TinyMCE | importmap | WYSIWYG admin |

---

## 8. Checklist de Développement

### Phase 1: Infrastructure (COMPLÈTE)

- [x] Symfony 7.4 LTS créé
- [x] Docker Compose configuré
- [x] MariaDB 11.4 prête
- [x] Mailhog intégré
- [x] PhpMyAdmin disponible
- [x] Asset Mapper configuré
- [x] Stimulus & Turbo importés
- [x] Doctrine ORM ready
- [x] Messenger ready

### Phase 2: Fondations Backend (À FAIRE)

- [ ] Installer EasyAdmin 4
- [ ] Installer VichUploaderBundle
- [ ] Installer LiipImagineBundle
- [ ] Installer StofDoctrineExtensionsBundle
- [ ] Créer entité User
- [ ] Créer entité Formation
- [ ] Créer entité Registration
- [ ] Créer entité Page
- [ ] Créer entité Article
- [ ] Créer entité Comment
- [ ] Créer entité Portfolio
- [ ] Créer entité Testimonial
- [ ] Créer entité Partner
- [ ] Créer entité ContactMessage
- [ ] Créer entité NotificationRecipient
- [ ] Créer entité SiteConfig
- [ ] Générer migrations
- [ ] Exécuter migrations
- [ ] Configurer sécurité complète

### Phase 3: Frontend (À FAIRE)

- [X] Ajouter Bootstrap 5.3
- [ ] Créer layout principal
- [ ] Créer page d'accueil
- [ ] Créer page formations
- [ ] Créer page détail formation
- [ ] Créer formulaire inscription
- [ ] Créer page contact
- [ ] Créer templates admin

### Phase 4: Fonctionnalités (À FAIRE)

- [ ] Authentification Form Login
- [ ] Gestion formations (CRUD admin)
- [ ] Inscription aux formations
- [ ] Gestion utilisateurs
- [ ] Gestion contenu (articles/pages)
- [ ] Système commentaires
- [ ] Portfolio
- [ ] Notifications email

### Phase 5: Tests & Documentation (À FAIRE)

- [ ] Tests unitaires
- [ ] Tests fonctionnels
- [ ] Documentation technique
- [ ] Documentation utilisateur

---

## 9. Commandes Utiles

### Docker

```bash
# Démarrer les conteneurs
docker-compose up -d

# Arrêter les conteneurs
docker-compose down

# Accéder au conteneur PHP
docker compose exec -it php bash

# Voir les logs
docker-compose logs -f
```

### Symfony (dans le conteneur PHP)

```bash
# Créer une entité
bin/console make:entity NomEntite

# Créer un contrôleur
bin/console make:controller NomController

# Créer un formulaire
bin/console make:form NomFormType

# Générer une migration
bin/console make:migration

# Exécuter les migrations
bin/console doctrine:migrations:migrate

# Lister les routes
bin/console debug:router

# Vider le cache
bin/console cache:clear
```

### Composer (dans le conteneur PHP)

```bash
# Installer EasyAdmin
composer require easycorp/easyadmin-bundle

# Installer VichUploader
composer require vich/uploader-bundle

# Installer LiipImagine
composer require liip/imagine-bundle

# Installer Doctrine Extensions
composer require stof/doctrine-extensions-bundle
```

### Asset Mapper

```bash
# Ajouter Bootstrap
bin/console importmap:require bootstrap

# Lister les assets
bin/console debug:asset-map
```

---

## URLs de Développement

| Service | URL | Identifiants |
|---------|-----|--------------|
| Application Symfony | http://localhost:8080 | - |
| PhpMyAdmin | http://localhost:8081 | cf2m / cf2m |
| Mailhog | http://localhost:8025 | - |

---

## Prochaines Étapes Recommandées

1. **Installer les bundles manquants** (EasyAdmin, VichUploader, LiipImagine, StofDoctrineExtensions)
2. **Créer l'entité User** avec les 3 rôles définis
3. **Configurer la sécurité** avec Form Login
4. **Créer les autres entités** selon le cahier des charges
5. **Générer et exécuter les migrations**
6. **Installer Bootstrap 5.3** via importmap
7. **Créer le layout principal** avec navbar et footer
8. **Développer les contrôleurs et templates** page par page

---

*Document généré automatiquement lors de l'analyse initiale du projet.*
