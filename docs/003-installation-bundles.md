# 003 - Installation des Bundles Symfony

**Date:** 27 janvier 2026
**Version:** 1.0
**Auteur:** Claude Code

---

## Résumé

Installation et configuration des bundles Symfony requis pour le projet CF2M :
- EasyAdmin 4 (administration)
- VichUploaderBundle (gestion des fichiers)
- LiipImagineBundle (redimensionnement d'images)
- StofDoctrineExtensionsBundle (slugs, timestamps)

---

## 1. EasyAdmin 4

### Installation

```bash
docker compose exec php composer require easycorp/easyadmin-bundle
```

### Version installée

- `easycorp/easyadmin-bundle` : v4.27.8

### Dépendances ajoutées

- `symfony/ux-twig-component` : v2.32.0
- `symfony/uid` : v7.4.4
- `twig/html-extra` : v3.23.0

### Configuration

Le bundle a été configuré automatiquement par la recette Symfony Flex.

**Fichier créé :** `config/packages/twig_component.yaml`

### Prochaines étapes

1. Créer le Dashboard : `src/Controller/Admin/DashboardController.php`
2. Créer les CRUD Controllers pour chaque entité

---

## 2. VichUploaderBundle

### Installation

```bash
docker compose exec php composer require vich/uploader-bundle
```

### Version installée

- `vich/uploader-bundle` : v2.9.1

### Configuration manuelle

**Fichier créé :** `config/packages/vich_uploader.yaml`

```yaml
vich_uploader:
    db_driver: orm
    metadata:
        type: attribute

    mappings:
        formation_image:
            uri_prefix: /uploads/formations
            upload_destination: '%kernel.project_dir%/public/uploads/formations'
            namer: Vich\UploaderBundle\Naming\SmartUniqueNamer

        testimonial_image:
            uri_prefix: /uploads/testimonials
            upload_destination: '%kernel.project_dir%/public/uploads/testimonials'

        partner_logo:
            uri_prefix: /uploads/partners
            upload_destination: '%kernel.project_dir%/public/uploads/partners'

        article_image:
            uri_prefix: /uploads/articles
            upload_destination: '%kernel.project_dir%/public/uploads/articles'

        portfolio_image:
            uri_prefix: /uploads/portfolio
            upload_destination: '%kernel.project_dir%/public/uploads/portfolio'

        user_avatar:
            uri_prefix: /uploads/users
            upload_destination: '%kernel.project_dir%/public/uploads/users'

        user_cv:
            uri_prefix: /uploads/cv
            upload_destination: '%kernel.project_dir%/public/uploads/cv'
```

### Dossiers d'upload créés

```
public/uploads/
├── formations/
├── testimonials/
├── partners/
├── articles/
├── portfolio/
├── users/
└── cv/
```

### Utilisation dans les entités

```php
use Vich\UploaderBundle\Mapping\Annotation as Vich;

#[Vich\Uploadable]
class Formation
{
    #[Vich\UploadableField(mapping: 'formation_image', fileNameProperty: 'imageName')]
    private ?File $imageFile = null;

    #[ORM\Column(nullable: true)]
    private ?string $imageName = null;
}
```

---

## 3. LiipImagineBundle

### Installation

```bash
docker compose exec php composer require liip/imagine-bundle
```

### Version installée

- `liip/imagine-bundle` : v2.17.1
- `imagine/imagine` : v1.5.2

### Configuration manuelle

**Fichier créé :** `config/packages/liip_imagine.yaml`

```yaml
liip_imagine:
    driver: gd

    resolvers:
        default:
            web_path:
                web_root: '%kernel.project_dir%/public'
                cache_prefix: 'media/cache'

    filter_sets:
        thumb_small:
            quality: 85
            filters:
                thumbnail:
                    size: [100, 100]
                    mode: outbound

        thumb_medium:
            quality: 85
            filters:
                thumbnail:
                    size: [250, 250]
                    mode: outbound

        list_image:
            quality: 85
            filters:
                thumbnail:
                    size: [400, 300]
                    mode: outbound

        detail_image:
            quality: 90
            filters:
                thumbnail:
                    size: [800, 600]
                    mode: inset

        partner_logo:
            quality: 90
            filters:
                thumbnail:
                    size: [200, 100]
                    mode: inset

        avatar:
            quality: 85
            filters:
                thumbnail:
                    size: [150, 150]
                    mode: outbound

        portfolio_thumb:
            quality: 85
            filters:
                thumbnail:
                    size: [350, 250]
                    mode: outbound

        portfolio_full:
            quality: 90
            filters:
                thumbnail:
                    size: [1200, 800]
                    mode: inset
```

**Fichier créé :** `config/routes/liip_imagine.yaml`

```yaml
_liip_imagine:
    resource: "@LiipImagineBundle/Resources/config/routing.yaml"
```

### Utilisation dans Twig

```twig
{# Appliquer un filtre à une image #}
<img src="{{ asset('uploads/formations/' ~ formation.imageName) | imagine_filter('list_image') }}" alt="">

{# Avec VichUploader #}
<img src="{{ vich_uploader_asset(formation, 'imageFile') | imagine_filter('thumb_medium') }}" alt="">
```

### Dossier de cache

```
public/media/cache/
```

---

## 4. StofDoctrineExtensionsBundle

### Installation

```bash
docker compose exec php composer require stof/doctrine-extensions-bundle
```

### Version installée

- `stof/doctrine-extensions-bundle` : v1.15.3
- `gedmo/doctrine-extensions` : v3.22.0

### Configuration manuelle

**Fichier créé :** `config/packages/stof_doctrine_extensions.yaml`

```yaml
stof_doctrine_extensions:
    default_locale: fr_FR
    orm:
        default:
            sluggable: true
            timestampable: true
            softdeleteable: false
            sortable: false
            tree: false
            translatable: false
            loggable: false
            blameable: true
```

### Extensions activées

| Extension | Activée | Description |
|-----------|---------|-------------|
| Sluggable | Oui | Génération automatique des slugs |
| Timestampable | Oui | createdAt/updatedAt automatiques |
| Blameable | Oui | Auteur des créations/modifications |
| SoftDeleteable | Non | Suppression logique |
| Sortable | Non | Tri automatique |
| Tree | Non | Arborescence |
| Translatable | Non | Traductions |
| Loggable | Non | Historique des modifications |

### Utilisation dans les entités

```php
use Gedmo\Mapping\Annotation as Gedmo;

class Formation
{
    #[Gedmo\Slug(fields: ['title'])]
    #[ORM\Column(length: 255, unique: true)]
    private ?string $slug = null;

    #[Gedmo\Timestampable(on: 'create')]
    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    #[Gedmo\Timestampable(on: 'update')]
    #[ORM\Column]
    private ?\DateTimeImmutable $updatedAt = null;

    #[Gedmo\Blameable(on: 'create')]
    #[ORM\ManyToOne(targetEntity: User::class)]
    private ?User $createdBy = null;
}
```

---

## 5. Enregistrement des Bundles

**Fichier modifié :** `config/bundles.php`

```php
<?php

return [
    // ... autres bundles ...
    EasyCorp\Bundle\EasyAdminBundle\EasyAdminBundle::class => ['all' => true],
    Vich\UploaderBundle\VichUploaderBundle::class => ['all' => true],
    Liip\ImagineBundle\LiipImagineBundle::class => ['all' => true],
    Stof\DoctrineExtensionsBundle\StofDoctrineExtensionsBundle::class => ['all' => true],
];
```

---

## 6. Récapitulatif des Fichiers Créés/Modifiés

| Fichier | Action | Description |
|---------|--------|-------------|
| `config/bundles.php` | Modifié | Ajout des 3 bundles manuels |
| `config/packages/twig_component.yaml` | Créé (auto) | Config UX Twig Component |
| `config/packages/vich_uploader.yaml` | Créé | Config VichUploader |
| `config/packages/liip_imagine.yaml` | Créé | Config LiipImagine |
| `config/packages/stof_doctrine_extensions.yaml` | Créé | Config Doctrine Extensions |
| `config/routes/liip_imagine.yaml` | Créé | Routes LiipImagine |
| `public/uploads/` | Créé | Dossiers d'upload |
| `public/uploads/.gitignore` | Créé | Ignore fichiers uploadés |
| `public/media/cache/` | Créé | Cache images LiipImagine |

---

## 7. Vérification

Pour vérifier que les bundles sont correctement configurés :

```bash
# Vérifier VichUploader
docker compose exec php bin/console debug:config vich_uploader

# Vérifier LiipImagine
docker compose exec php bin/console debug:config liip_imagine

# Vérifier Doctrine Extensions
docker compose exec php bin/console debug:config stof_doctrine_extensions

# Lister les routes LiipImagine
docker compose exec php bin/console debug:router | grep liip
```

---

## 8. Checklist Mise à Jour

Les tâches suivantes sont maintenant complètes :

- [x] Installer EasyAdmin 4
- [x] Installer VichUploaderBundle
- [x] Installer LiipImagineBundle
- [x] Installer StofDoctrineExtensionsBundle

---

## Prochaines Étapes

1. **Créer l'entité User** avec les attributs Gedmo (timestampable, blameable)
2. **Créer les autres entités** avec VichUploader pour les images
3. **Configurer EasyAdmin** avec le Dashboard et les CRUD controllers
4. **Configurer la sécurité** Symfony

---

*Document créé lors de l'installation des bundles Symfony.*
