# 004 - Configuration Nginx pour Asset Mapper

**Date:** 27 janvier 2026
**Version:** 1.0
**Auteur:** Claude Code

---

## Problème Rencontré

Après l'installation de Bootstrap 5.3 via Asset Mapper, tous les fichiers CSS et JS retournaient des erreurs 404 :

```
bootstrap.min-z0zQt-h.css       404
app-xkacMA9.css                 404
app-_ycYZv3.js                  404
stimulus_bootstrap-xCO4u8H.js   404
bootstrap.index-HgGqGv8.js      404
loader-V1GtHuK.js               404
...
```

---

## Cause du Problème

En mode développement, Symfony Asset Mapper génère les fichiers CSS/JS **dynamiquement via PHP**. Ces fichiers n'existent pas physiquement dans le dossier `public/assets/`.

La configuration Nginx originale tentait de servir les fichiers statiques (`.css`, `.js`, etc.) directement depuis le système de fichiers, sans passer par PHP :

```nginx
# Configuration originale (problématique)
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

Quand Nginx ne trouvait pas le fichier physique, il retournait une erreur 404 au lieu de transmettre la requête à PHP.

---

## Solution Appliquée

Ajout de `try_files` avec fallback vers `index.php` pour les fichiers statiques :

```nginx
# Configuration corrigée
location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
    try_files $uri /index.php$is_args$args;
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

### Explication

- `try_files $uri` : Nginx tente d'abord de servir le fichier s'il existe physiquement
- `/index.php$is_args$args` : Si le fichier n'existe pas, la requête est transmise à PHP (Symfony)
- En mode dev : Symfony génère les assets dynamiquement
- En mode prod : Les assets compilés sont servis directement par Nginx

---

## Fichier Modifié

**Fichier :** `docker/nginx/default.conf`

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;
    index index.php;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";

    # Max upload size
    client_max_body_size 20M;

    # Logs
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location / {
        # Try to serve file directly, fallback to index.php
        try_files $uri /index.php$is_args$args;
    }

    # Handle PHP files
    location ~ ^/index\.php(/|$) {
        fastcgi_pass php:9000;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;

        # Buffer settings
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;

        # Timeout settings
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;

        # Prevents URIs that include the front controller
        internal;
    }

    # Block access to other PHP files
    location ~ \.php$ {
        return 404;
    }

    # Static files caching (with fallback to PHP for Asset Mapper in dev mode)
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|svg|woff|woff2|ttf|eot)$ {
        try_files $uri /index.php$is_args$args;
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # Block access to hidden files
    location ~ /\. {
        deny all;
    }

    # Block access to sensitive files
    location ~ /(composer\.(json|lock)|\.env|\.git) {
        deny all;
    }
}
```

---

## Commandes pour Appliquer les Changements

Après modification de `docker/nginx/default.conf`, il faut **reconstruire l'image Nginx** :

```bash
# Reconstruire et redémarrer le conteneur Nginx
docker compose up -d --build nginx
```

> **Note :** Un simple `docker compose restart nginx` ne suffit pas car la configuration est copiée dans l'image Docker lors du build.

---

## Vérification

### Tester que les assets sont servis

```bash
# Tester un fichier JS
curl -s http://localhost:8080/assets/app-_ycYZv3.js | head -5

# Résultat attendu :
# import './stimulus_bootstrap.js';
# /*
#  * Welcome to your app's main JavaScript file!
# ...
```

### Vérifier les logs Nginx

```bash
docker compose logs nginx --tail=20
```

Les erreurs "No such file or directory" ne devraient plus apparaître pour les fichiers `/assets/`.

---

## Mode Production

En production, il est recommandé de **compiler les assets** pour de meilleures performances :

```bash
# Compiler les assets
docker compose exec php bin/console asset-map:compile

# Les fichiers sont créés dans public/assets/
# Nginx les servira directement sans passer par PHP
```

Pour revenir en mode développement (assets dynamiques) :

```bash
# Supprimer les assets compilés
rm -rf public/assets

# Vider le cache
docker compose exec php bin/console cache:clear
```

---

## Résumé

| Aspect | Avant | Après |
|--------|-------|-------|
| Fichiers CSS/JS | 404 Not Found | Servis correctement |
| Mode dev | Non fonctionnel | Assets générés via PHP |
| Mode prod | Fonctionnel | Assets servis par Nginx |
| Configuration | `try_files` manquant | `try_files $uri /index.php...` |

---

## Leçon Apprise

Avec Symfony Asset Mapper, la configuration Nginx doit toujours prévoir un **fallback vers PHP** pour les fichiers statiques, car en mode développement, ces fichiers sont générés dynamiquement et n'existent pas sur le disque.

---

*Document créé suite à la résolution du problème des assets 404.*
