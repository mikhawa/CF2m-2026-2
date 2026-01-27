# CLAUDE.md
Ce fichier fournit des directives à Claude Code (claude.ai/code) lors du travail sur le code de ce dépôt.

## Aperçu du Projet
Ce dépôt contient les spécifications du site web du CF2M (Centre de Formation 2 Mille) - un centre de formation professionnelle à Bruxelles spécialisé dans les métiers du numérique pour les demandeurs d'emploi. Le projet est actuellement en phase de planification et de documentation.

## 1. Présentation du Projet

### 1.1 Contexte
Refonte du site web du Centre de Formation 2 Mille (CF2M), organisme de formation professionnelle situé à Saint-Gilles, Bruxelles, spécialisé dans les métiers du numérique pour chercheurs et chercheuses d'emploi.

### 1.2 Objectifs
- Présenter le centre de formation et ses valeurs
- Afficher les formations disponibles avec leurs détails
- Permettre l'inscription en ligne aux formations
- Gérer le contenu via une interface d'administration intuitive
- Assurer une navigation responsive et moderne
- Faciliter le contact avec le centre

### 1.3 Public Cible
- Chercheurs d'emploi intéressés par les formations en informatique
- Partenaires et entreprises
- Personnel administratif du CF2M

## 2. Stack Technique (Prévue)

- chemin de développement au travail : 
    - \\wsl.localhost\Ubuntu2\home\mikhawa\CF2m-2026-2
    ou à la maison :
    - \\wsl.localhost\Ubuntu\home\mikhawa\CF2m-2026-2
- Backend : Symfony 7.4 LTS avec l'option --webapp
- PHP : 8.4
- Base de données : MariaDB 11.4 LTS
- Frontend : Symfony UX (Asset Mapper) + Bootstrap 5.3
- Administration : EasyAdmin 4 avec TinyMCE (via Stimulus)
- Éditeur WYSIWYG dans l'administration : TinyMCE avec upload d'images et fichiers via VichUploaderBundle
- Gestion des fichiers : VichUploaderBundle + LiipImagineBundle pour le redimensionnement d'images
- Messagerie asynchrone : Symfony Messenger avec transport Doctrine pour la file d'attente des emails
- Conteneurisation : Docker + Docker Compose pour le développement local
- Environemment de développement : WSL2 (Ubuntu) sur Windows 11
- Environement de production prévu : Serveur Linux avec Apache et Nginx et MariaDB

