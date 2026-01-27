# 005 - Création de l'Entité User

**Date:** 27 janvier 2026
**Version:** 1.0
**Auteur:** Claude Code

---

## Résumé

Création de l'entité User avec gestion des rôles, upload d'avatar et CV, système d'activation par token, et configuration de la sécurité Symfony.

---

## 1. Structure de l'Entité User

### Fichier créé : `src/Entity/User.php`

### Champs de l'entité

| Champ | Type | Description |
|-------|------|-------------|
| `id` | int | Identifiant auto-incrémenté |
| `email` | string(180) | Email unique (identifiant de connexion) |
| `roles` | json | Tableau des rôles |
| `password` | string | Mot de passe hashé |
| `firstName` | string(100) | Prénom |
| `lastName` | string(100) | Nom |
| `phone` | string(20) | Téléphone (optionnel) |
| `avatarFile` | File | Fichier avatar (VichUploader) |
| `avatarName` | string(255) | Nom du fichier avatar |
| `cvFile` | File | Fichier CV (VichUploader) |
| `cvName` | string(255) | Nom du fichier CV |
| `isActive` | bool | Compte activé (défaut: false) |
| `activationToken` | string(255) | Token d'activation |
| `activationTokenExpiresAt` | DateTimeImmutable | Expiration du token (48h) |
| `lastLoginAt` | DateTimeImmutable | Dernière connexion |
| `createdAt` | DateTimeImmutable | Date de création (Gedmo) |
| `updatedAt` | DateTimeImmutable | Date de mise à jour (Gedmo) |

### Interfaces implémentées

- `UserInterface` : Interface Symfony pour l'authentification
- `PasswordAuthenticatedUserInterface` : Interface pour le hashage de mot de passe

### Annotations utilisées

- `#[Vich\Uploadable]` : Entité uploadable pour VichUploader
- `#[Gedmo\Timestampable]` : Timestamps automatiques
- `#[UniqueEntity]` : Contrainte d'unicité sur l'email

---

## 2. Système de Rôles

### Constantes définies

```php
public const ROLE_USER = 'ROLE_USER';
public const ROLE_RESPONSABLE = 'ROLE_RESPONSABLE';
public const ROLE_ADMIN = 'ROLE_ADMIN';
```

### Hiérarchie des rôles

```yaml
role_hierarchy:
    ROLE_RESPONSABLE: ROLE_USER
    ROLE_ADMIN: [ROLE_RESPONSABLE, ROLE_USER]
```

- `ROLE_USER` : Utilisateur standard (accès au profil, commentaires)
- `ROLE_RESPONSABLE` : Responsable de formation (hérite de ROLE_USER)
- `ROLE_ADMIN` : Administrateur (hérite de tous les rôles)

### Méthodes utilitaires

```php
// Vérifier si l'utilisateur a un rôle
$user->hasRole('ROLE_ADMIN');

// Raccourcis
$user->isAdmin();
$user->isResponsable();
```

---

## 3. Système d'Activation

### Workflow d'activation

1. L'administrateur crée un compte utilisateur
2. Un token d'activation est généré (valide 48h)
3. Un email avec le lien d'activation est envoyé
4. L'utilisateur clique sur le lien et définit son mot de passe
5. Le compte est activé

### Méthodes

```php
// Générer un token d'activation
$user->generateActivationToken();

// Vérifier si le token est valide
$user->isActivationTokenValid();

// Effacer le token après activation
$user->clearActivationToken();
```

---

## 4. Upload de Fichiers (VichUploader)

### Avatar

- **Mapping :** `user_avatar`
- **Destination :** `public/uploads/users/`
- **Types acceptés :** JPEG, PNG, WebP
- **Taille max :** 2 Mo

### CV

- **Mapping :** `user_cv`
- **Destination :** `public/uploads/cv/`
- **Types acceptés :** PDF
- **Taille max :** 5 Mo

### Utilisation

```php
// Upload d'avatar
$user->setAvatarFile($uploadedFile);

// Récupérer le nom du fichier
$avatarName = $user->getAvatarName();
```

---

## 5. Repository (UserRepository)

### Fichier créé : `src/Repository/UserRepository.php`

### Méthodes disponibles

| Méthode | Description |
|---------|-------------|
| `upgradePassword()` | Met à jour le mot de passe hashé |
| `findByActivationToken()` | Trouve un utilisateur par token |
| `findAllActive()` | Liste tous les utilisateurs actifs |
| `findByRole()` | Liste les utilisateurs par rôle |
| `findAllResponsables()` | Liste tous les responsables |
| `findAllAdmins()` | Liste tous les administrateurs |
| `findWithExpiredActivationTokens()` | Tokens expirés |
| `search()` | Recherche par nom/email |

---

## 6. Configuration Sécurité

### Fichier modifié : `config/packages/security.yaml`

### Provider

```yaml
providers:
    app_user_provider:
        entity:
            class: App\Entity\User
            property: email
```

### Form Login

```yaml
form_login:
    login_path: app_login
    check_path: app_login
    default_target_path: app_home
    enable_csrf: true
```

### Remember Me

```yaml
remember_me:
    secret: '%kernel.secret%'
    lifetime: 604800  # 1 semaine
```

### Access Control

```yaml
access_control:
    - { path: ^/admin, roles: ROLE_ADMIN }
    - { path: ^/gestion, roles: ROLE_RESPONSABLE }
    - { path: ^/profil, roles: ROLE_USER }
```

---

## 7. Migration

### Fichier créé : `migrations/Version20260127151710.php`

### Tables créées

**Table `user` :**

```sql
CREATE TABLE `user` (
    id INT AUTO_INCREMENT NOT NULL,
    email VARCHAR(180) NOT NULL,
    roles JSON NOT NULL,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) DEFAULT NULL,
    avatar_name VARCHAR(255) DEFAULT NULL,
    cv_name VARCHAR(255) DEFAULT NULL,
    is_active TINYINT NOT NULL,
    activation_token VARCHAR(255) DEFAULT NULL,
    activation_token_expires_at DATETIME DEFAULT NULL,
    last_login_at DATETIME DEFAULT NULL,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE INDEX UNIQ_8D93D649E7927C74 (email),
    PRIMARY KEY(id)
)
```

**Table `messenger_messages` :** (pour la file d'attente des emails)

```sql
CREATE TABLE messenger_messages (
    id BIGINT AUTO_INCREMENT NOT NULL,
    body LONGTEXT NOT NULL,
    headers LONGTEXT NOT NULL,
    queue_name VARCHAR(190) NOT NULL,
    created_at DATETIME NOT NULL,
    available_at DATETIME NOT NULL,
    delivered_at DATETIME DEFAULT NULL,
    INDEX IDX_... (queue_name, available_at, delivered_at, id),
    PRIMARY KEY(id)
)
```

---

## 8. Commandes Exécutées

```bash
# Créer les dossiers
mkdir -p src/Entity src/Repository

# Vider le cache
docker compose exec php bin/console cache:clear

# Générer la migration
docker compose exec php bin/console make:migration

# Exécuter la migration
docker compose exec php bin/console doctrine:migrations:migrate --no-interaction
```

---

## 9. Fichiers Créés/Modifiés

| Fichier | Action | Description |
|---------|--------|-------------|
| `src/Entity/User.php` | Créé | Entité User |
| `src/Repository/UserRepository.php` | Créé | Repository User |
| `config/packages/security.yaml` | Modifié | Configuration sécurité |
| `migrations/Version20260127151710.php` | Créé | Migration BDD |

---

## 10. Utilisation

### Créer un utilisateur (dans un contrôleur ou service)

```php
use App\Entity\User;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

$user = new User();
$user->setEmail('admin@cf2m.be');
$user->setFirstName('Admin');
$user->setLastName('CF2M');
$user->setPassword($passwordHasher->hashPassword($user, 'password123'));
$user->setRoles([User::ROLE_ADMIN]);
$user->setIsActive(true);

$entityManager->persist($user);
$entityManager->flush();
```

### Vérifier les rôles (dans Twig)

```twig
{% if is_granted('ROLE_ADMIN') %}
    <a href="{{ path('admin') }}">Administration</a>
{% endif %}

{% if is_granted('ROLE_RESPONSABLE') %}
    <a href="{{ path('gestion') }}">Gestion</a>
{% endif %}
```

### Afficher l'avatar

```twig
{% if user.avatarName %}
    <img src="{{ vich_uploader_asset(user, 'avatarFile') | imagine_filter('avatar') }}"
         alt="{{ user.fullName }}">
{% else %}
    <img src="{{ asset('images/default-avatar.png') }}" alt="Avatar par défaut">
{% endif %}
```

---

## 11. Prochaines Étapes

1. **Créer le contrôleur de sécurité** (SecurityController) avec les routes login/logout
2. **Créer le formulaire de connexion** (templates)
3. **Créer les DataFixtures** pour les utilisateurs de test
4. **Configurer EasyAdmin** pour la gestion des utilisateurs
5. **Créer les autres entités** (Formation, Registration, etc.)

---

## 12. Checklist Mise à Jour

Les tâches suivantes sont maintenant complètes :

- [x] Créer l'entité User
- [x] Configurer la sécurité (UserProvider, hiérarchie des rôles)
- [x] Configurer Form Login
- [x] Générer et exécuter la migration

---

*Document créé lors de la création de l'entité User.*
