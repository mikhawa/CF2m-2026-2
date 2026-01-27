# 002 - Installation de Bootstrap 5.3

**Date:** 27 janvier 2026
**Version:** 1.0
**Auteur:** Claude Code

---

## Résumé

Installation de Bootstrap 5.3.8 via Symfony Asset Mapper (importmap) pour le projet CF2M.

---

## Packages Installés

| Package | Version | Type |
|---------|---------|------|
| `bootstrap` | 5.3.8 | JavaScript |
| `@popperjs/core` | 2.11.8 | JavaScript (dépendance) |
| `bootstrap/dist/css/bootstrap.min.css` | 5.3.8 | CSS |

---

## Commande Utilisée

```bash
docker compose exec php bin/console importmap:require bootstrap
```

---

## Fichiers Modifiés

### 1. importmap.php

Nouveaux entries ajoutés automatiquement :

```php
'bootstrap' => [
    'version' => '5.3.8',
],
'@popperjs/core' => [
    'version' => '2.11.8',
],
'bootstrap/dist/css/bootstrap.min.css' => [
    'version' => '5.3.8',
    'type' => 'css',
],
```

### 2. assets/app.js

Imports Bootstrap CSS et JS ajoutés :

```javascript
import './stimulus_bootstrap.js';

// Bootstrap CSS
import 'bootstrap/dist/css/bootstrap.min.css';

// Bootstrap JS (includes Popper)
import 'bootstrap';

// Custom styles (after Bootstrap to allow overrides)
import './styles/app.css';
```

### 3. templates/base.html.twig

Modifications pour la compatibilité Bootstrap :

- Ajout de `lang="fr"` sur la balise `<html>`
- Ajout de la meta viewport pour le responsive :
  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  ```
- Titre par défaut mis à jour : "CF2M - Centre de Formation 2 Mille"

### 4. assets/styles/app.css

Préparé pour les styles personnalisés CF2M :

```css
/*
 * Custom styles for CF2M
 * This file is loaded after Bootstrap to allow overrides
 */

/* Custom variables - CF2M brand colors */
:root {
    --cf2m-primary: #0d6efd;
    --cf2m-secondary: #6c757d;
}

/* Add custom styles below */
```

---

## Vérification

Pour vérifier que Bootstrap fonctionne correctement :

1. Accéder à http://localhost:8080
2. Ouvrir les DevTools du navigateur (F12)
3. Vérifier dans l'onglet Network que les fichiers Bootstrap sont chargés
4. Vérifier dans la Console qu'il n'y a pas d'erreurs

---

## Utilisation

### Classes CSS Bootstrap

Bootstrap 5.3 est maintenant disponible dans tous les templates Twig :

```twig
<div class="container">
    <div class="row">
        <div class="col-md-6">
            <button class="btn btn-primary">Bouton</button>
        </div>
    </div>
</div>
```

### Composants JavaScript

Les composants Bootstrap JS (modals, dropdowns, tooltips, etc.) sont disponibles :

```twig
<!-- Modal -->
<div class="modal fade" id="exampleModal" tabindex="-1">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title">Titre</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
            </div>
            <div class="modal-body">Contenu</div>
        </div>
    </div>
</div>
```

### Personnalisation

Pour personnaliser les styles Bootstrap, modifier `assets/styles/app.css` :

```css
:root {
    --cf2m-primary: #your-color;
}

.btn-primary {
    background-color: var(--cf2m-primary);
}
```

---

## Documentation Bootstrap

- Site officiel : https://getbootstrap.com/docs/5.3/
- Composants : https://getbootstrap.com/docs/5.3/components/
- Utilitaires : https://getbootstrap.com/docs/5.3/utilities/

---

## Checklist Mise à Jour

Dans `001-analyse-initiale-projet.md`, la tâche suivante est maintenant complète :

- [x] Ajouter Bootstrap 5.3

---

*Document créé lors de l'installation de Bootstrap 5.3 via importmap.*
