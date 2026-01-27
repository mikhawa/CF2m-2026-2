# CLAUDE.md
Ce fichier fournit des directives à Claude Code (claude.ai/code) lors du travail sur le code de ce dépôt.

## Aperçu du Projet
Ce dépôt contient les spécifications du site web du CF2M (Centre de Formation 2 Mille) - un centre de formation professionnelle à Bruxelles spécialisé dans les métiers du numérique pour les demandeurs d'emploi. Le projet est actuellement en phase de planification et de documentation.

## Stack Technique (Prévue)

- chemin de développement au travail : 
    - \\wsl.localhost\Ubuntu2\home\mikhawa\CF2m-2026-2
    ou à la maison :
    - \\wsl.localhost\Ubuntu\home\mikhawa\CF2m-2026-2
- Backend : Symfony 7.4 avec l'option --webapp
- PHP : 8.4
- Base de données : MariaDB 11.4 LTS
- Frontend : Symfony UX (Asset Mapper) + Bootstrap 5.3
- Administration : EasyAdmin 4 avec TinyMCE (via Stimulus)
- Éditeur WYSIWYG dans l'administration : TinyMCE avec upload d'images et fichiers via VichUploaderBundle
- Gestion des fichiers : VichUploaderBundle + LiipImagineBundle pour le redimensionnement d'images
- Messagerie asynchrone : Symfony Messenger avec transport Doctrine pour la file d'attente des emails
- Conteneurisation : Docker + Docker Compose pour le développement local
- Environemment de développement : WSL2 (Ubuntu) sur Windows 11
- Environement de production prévu : Serveur Linux avec Apache ou Nginx et MariaDB

## Fichiers de Documentation Clés
cahier-des-charges-cf2m-v2.md - Document de spécifications complet comprenant :

Modèle de données (12 entités : User, Formation, Registration, Page, Article, Comment, Portfolio, Testimonial, Partner, ContactMessage, NotificationRecipient, SiteConfig)

Besoins CRUD backend pour EasyAdmin

Spécifications des pages publiques frontend

Besoins sécurité/authentification (Form Login, 3 rôles : ROLE_ADMIN, ROLE_RESPONSABLE, ROLE_USER)

Spécifications du système d'email (Symfony Mailer avec file d'attente asynchrone Messenger)

recapitulatif-modifications.md - Journal des modifications (changelog) de la version 2.0 détaillant :

Système d'inscription basé sur des fenêtres modales

Champs de gestion de session pour les formations

Nouvelles entités (Article, Comment, Portfolio, NotificationRecipient)

Système de notification multi-destinataires

Workflow d'activation utilisateur (pas d'auto-inscription)

Notes d'Architecture
Relations entre les Entités
User ↔ Formation : ManyToMany (responsables)

Formation → Registration, NotificationRecipient, Testimonial : OneToMany

Article → Comment : OneToMany

L'entité User est liée en tant qu'auteur (author), responsable du traitement (processedBy) ou de la mise à jour (updatedBy) à travers plusieurs entités.

Accès Basé sur les Rôles
ROLE_ADMIN : Accès complet au backend.

ROLE_RESPONSABLE : Accès aux formations assignées + inscriptions + articles + commentaires + portfolio.

ROLE_USER : Accès au profil + commentaires d'articles uniquement.

Structure des Téléchargements (Prévue)
/public/uploads/
├── formations/
├── testimonials/
├── partners/
├── articles/
├── portfolio/
├── users/
└── cv/
Bundles Symfony Requis
Bash
composer require easycorp/easyadmin-bundle
composer require vich/uploader-bundle
composer require liip/imagine-bundle
composer require symfony/messenger
composer require stof/doctrine-extensions-bundle
# TinyMCE est installé via importmap (pas de bundle composer)
Notes de Développement
Tout le contenu est en français.

Langue du site : Français uniquement (le multilingue est une question ouverte).

Pas d'inscription publique : les comptes sont créés uniquement par les administrateurs avec un lien d'activation envoyé par email (validité de 48h).

Les commentaires nécessitent une modération avant publication.

La pré-inscription utilise une modale AJAX avec fermeture automatique après 3 secondes en cas de succès.