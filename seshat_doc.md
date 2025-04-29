Rédige un mémo synthétique et professionnel pour le projet Seshat v1.0. Ce mémo doit servir de résumé exécutif du projet.

N'inclus que le mémo final dans ta réponse, **sans** répéter cette instruction ni la documentation fournie ci-dessous.

Le mémo doit inclure les points suivants :
1.  **En-tête :** MÉMO PROJET - Seshat v1.0
2.  **Date :** Mai 2024
3.  **De :** de SOUZA Schadrac Félicio C. (Développeur Web)
4.  **Objet :** Présentation de la Version 1.0 de l'Assistant IA Seshat
5.  **Introduction :** Objectif principal du projet Seshat (1-2 phrases).
6.  **Fonctionnalités Clés Livrées (v1.0) :** Liste à puces résumant les fonctionnalités majeures implémentées (Auth, Gestion Profil/Compte, Gestion Conversations, Interface Chat, Intégration IA, Rendu Markdown).
7.  **Stack Technique Principale :** Résumé des technologies clés (Next.js, React, Supabase, Google Gemini, Vercel).
8.  **Statut et Conclusion :** Bref état du projet (MVP fonctionnel) et mention du potentiel d'évolution.

Base-toi **exclusivement** sur la documentation technique détaillée fournie ci-dessous pour extraire les informations nécessaires à la rédaction de ce mémo.

--- DEBUT DE LA DOCUMENTATION TECHNIQUE (CONTEXTE) ---

# Documentation Projet - Seshat AI (v1.0)

**Version :** 1.0
**Date :** Mai 2024
**Auteur :** de SOUZA Schadrac Félicio C. (Développeur Web)

---

## 1. Introduction et Objectifs

### 1.1. Contexte et Problématique

La création de contenu textuel (scénarios, articles, récits) représente un défi constant impliquant inspiration, structuration, cohérence et style. Seshat a été conçu comme un **assistant IA spécialisé** pour adresser ces défis, en servant de partenaire créatif plutôt que de simple chatbot. Il vise à s'intégrer au flux de travail des créateurs en fournissant une aide contextuelle via une interface conversationnelle.

### 1.2. Objectifs de la Version 1.0

Cette version initiale vise à établir un **Produit Minimum Viable (MVP)** robuste et fonctionnel :

1.  **Plateforme IA Fiable :** Intégration stable de Google Gemini (`gemini-1.5-flash-latest`) pour des réponses pertinentes.
2.  **Gestion Conversation Intuitive :** Création, visualisation de l'historique, interaction fluide.
3.  **Persistance & Organisation :** Sauvegarde automatique (Supabase) et génération de titres de conversation.
4.  **Sécurité & Confidentialité :** Authentification (Supabase Auth) et contrôle d'accès (RLS).
5.  **Gestion de Compte Essentielle :** Modification du nom d'utilisateur (avec délai), suppression sécurisée.
6.  **Architecture Moderne :** Utilisation de Next.js (App Router), Supabase, et Vercel pour la scalabilité et la maintenabilité.

---

## 2. Fonctionnalités Clés (v1.0)

### 2.1. Gestion des Utilisateurs et Authentification

*   **Inscription :** Email, mot de passe sécurisé, nom d'utilisateur unique. Validation email optionnelle (via Supabase).
*   **Connexion / Déconnexion :** Authentification standard et gestion de session JWT.
*   **Profil Automatique :** Trigger SQL (`handle_new_user`) pour créer une entrée `profils` à l'inscription.

### 2.2. Gestion du Profil et du Compte

*   **Consultation :** Affichage du nom d'utilisateur et email dans les paramètres.
*   **Modification Nom d'Utilisateur :**
    *   Interface de mise à jour dans les paramètres.
    *   Contrainte : Changement possible seulement **tous les 7 jours** (basé sur `profils.last_username_change`).
    *   L'UI indique la disponibilité ou le délai restant.
    *   Synchronisation avec `auth.users.raw_user_meta_data` et `profils.username`.
*   **Avatar :** Affichage de l'initiale de l'utilisateur (basé sur `username` ou `email`) dans le chat.
*   **Suppression Compte :** Processus sécurisé (via Fonction Edge recommandée) avec confirmation utilisateur, supprimant toutes les données associées (Auth, profils, conversations, messages via `ON DELETE CASCADE`).

### 2.3. Gestion des Conversations

*   **Création :** Au premier message envoyé par l'utilisateur (depuis une interface dédiée), une nouvelle conversation est créée dans la table `conversations` (liée à `user_id`).
*   **Génération Titre :** Appel asynchrone (non bloquant) à l'API `/api/generate-title` qui utilise Gemini pour générer un titre basé sur le premier message et met à jour la DB.
*   **Accès :** Redirection automatique vers `/chat/[conversationId]` après création.

### 2.4. Interface de Chat

*   **Historique :** Chargement et affichage des messages (`user` et `assistant`) depuis Supabase, triés chronologiquement.
*   **Saisie (`ChatInput`) :** Zone de texte pour les prompts utilisateur.
*   **Envoi Message Utilisateur :** Affichage immédiat (optimistic update), sauvegarde (`sender='user'`) dans Supabase, puis envoi du prompt et de l'historique formaté à l'API `/api/chat`.
*   **Réponse IA :** Indicateur de chargement (`SmartTypingAnimation`). Réception de la réponse depuis `/api/chat`, affichage, puis sauvegarde (`sender='model'`) dans Supabase.
*   **Rendu Markdown (`MarkdownRenderer`) :** Interprétation du Markdown de l'IA avec `react-markdown`, `remark-gfm`, `rehype-raw`, `rehype-sanitize`. Blocs de code rendus avec `react-syntax-highlighter` (style `vscDarkPlus`). Correction des problèmes d'imbrication HTML invalide.
*   **Actions :** Copie du contenu des messages.

### 2.5. Intégration de l'IA (Google Gemini)

*   **Modèle Chat :** `gemini-1.5-flash-latest`.
*   **Modèle Titre :** `gemini-1.5-flash-latest` (avec config spécifique : temp basse, tokens courts).
*   **Instructions Système :** Fichier `app/ai-system/instructions.md` (ou défaut) définissant le rôle (Seshat), le format Markdown et le style.
*   **Configuration Génération :** Paramètres ajustés (`temperature`, `topP`, `topK`, `maxOutputTokens`) pour la créativité.
*   **Configuration Sécurité :** Seuils `HarmBlockThreshold.BLOCK_MEDIUM_AND_ABOVE` pour les catégories de contenu sensible.
*   **"Deep Thinking" :** Flag booléen envoyé à l'API pour potentiellement modifier les `generationConfig` (e.g., `maxOutputTokens`).

---

## 3. Architecture Technique

### 3.1. Vue d'Ensemble

Seshat utilise une architecture web moderne, découplée et serverless :

*   **Frontend :** Next.js 13+ (App Router) / React / Tailwind CSS (supposé)
*   **Backend / API :** Next.js API Routes (Route Handlers) déployées comme fonctions serverless (Vercel)
*   **Base de Données & Auth (BaaS) :** Supabase (PostgreSQL, Auth, Storage, potentiellement Edge Functions)
*   **Service IA :** Google Gemini API
*   **Hébergement :** Vercel

### 3.2. Composants Détaillés

*   **Frontend :** Gère l'UI/UX, les interactions client, la gestion d'état locale et globale (`AuthContext`), les appels client Supabase (auth, requêtes simples RLS) et les appels `fetch` vers les API Routes Next.js.
*   **API Routes (Backend) :** Intégrées au projet Next.js (`app/api/.../route.ts`). Point d'entrée sécurisé pour la logique serveur. Elles seules détiennent les clés API secrètes (Gemini, Supabase Service Role). Gèrent la communication avec Gemini et les opérations DB nécessitant des droits élevés.
*   **Supabase :** Fournit l'authentification, la base de données relationnelle avec RLS, le stockage d'objets, et l'environnement pour les fonctions Edge (recommandé pour la suppression de compte).
*   **Google Gemini :** Fournit le modèle LLM. Contacté uniquement via les API Routes.

### 3.3. Flux de Données (Exemple: Envoi Message)

1.  **Client (UI) -> Client (Logic)**: `ChatInput` -> `handleSubmit` dans `ChatInterface`.
2.  **Client (Logic)**: Update état local UI, `saveMessageToDB('user')`, call `getAIResponse`.
3.  **Client (Logic) -> Serveur (API Route)**: `getAIResponse` prépare `FormData`, `fetch POST '/api/chat'`.
4.  **Serveur (API Route `/api/chat`)**: Reçoit requête, initialise services, parse `FormData`, appelle `getAIResponseFromGemini`.
5.  **Serveur (API Route) -> Gemini**: `getAIResponseFromGemini` appelle l'API Gemini.
6.  **Gemini -> Serveur (API Route)**: Gemini renvoie la réponse textuelle.
7.  **Serveur (API Route) -> Client (Logic)**: L'API Route renvoie `NextResponse.json({ response: ... })`.
8.  **Client (Logic)**: `getAIResponse` reçoit la réponse, update état local UI, `saveMessageToDB('model')`.

### 3.4. Sécurité

*   **AuthN/AuthZ :** Supabase Auth (JWT) + RLS sur toutes les tables.
*   **Secrets :** Clés API gérées comme variables d'environnement serveur (Vercel), jamais exposées au client.
*   **Validation :** Validation basique des entrées dans les API Routes.
*   **XSS :** `rehype-sanitize` sur le Markdown rendu.
*   **Opérations Sensibles :** Fonctions Edge Supabase recommandées pour les actions nécessitant `SERVICE_ROLE_KEY` (ex: suppression compte).

---

## 4. Schéma de la Base de Données

Base de données PostgreSQL gérée par Supabase. RLS activée partout.

### 4.1. Table `profils`

*   Stocke les informations utilisateur étendues et paramètres.
*   `id` (UUID, PK, FK -> auth.users.id ON DELETE CASCADE)
*   `email` (TEXT, UNIQUE, NOT NULL)
*   `username` (TEXT, UNIQUE)
*   `mode` (TEXT, DEFAULT 'standard')
*   `deep_think_enabled` (BOOLEAN, DEFAULT FALSE)
*   `plan` (TEXT, DEFAULT 'free')
*   `subscription_status` (TEXT, DEFAULT 'free')
*   `last_username_change` (TIMESTAMPTZ, NULLable)
*   `created_at` / `updated_at` (TIMESTAMPTZ)
*   *RLS :* Select/Update par propriétaire (`auth.uid() = id`). Insert/Delete via trigger/cascade.

### 4.2. Table `conversations`

*   Représente une session de chat.
*   `id` (UUID, PK)
*   `user_id` (UUID, FK -> auth.users.id ON DELETE CASCADE)
*   `anonymous_session_id` (TEXT, UNIQUE, NULLable) - Si utilisé
*   `title` (TEXT, NULLable) - Généré par IA
*   `created_at` / `updated_at` (TIMESTAMPTZ)
*   *RLS :* CRUD par propriétaire (`auth.uid() = user_id`). Politiques spécifiques pour anonyme si nécessaire.

### 4.3. Table `messages`

*   Stocke chaque échange.
*   `id` (UUID, PK)
*   `conversation_id` (UUID, FK -> conversations.id ON DELETE CASCADE, NOT NULL)
*   `sender` (TEXT, NOT NULL, CHECK ('user', 'model'))
*   `content` (TEXT, NOT NULL)
*   `metadata` (JSONB, NULLable)
*   `created_at` (TIMESTAMPTZ)
*   *RLS :* Select/Insert si propriétaire de la conversation parente. Update/Delete généralement interdit.

### 4.4. Triggers & Fonctions

*   `handle_new_user()` (Fonction SQL) & Trigger `on_auth_user_created`: Crée automatiquement un profil `profils` à l'inscription.
*   `update_updated_at_column()` (Fonction SQL) & Triggers associés : Met à jour `updated_at` sur `profils` et `conversations`.

---

## 5. Configuration et Déploiement

### 5.1. Prérequis

*   Node.js (LTS) & npm/yarn/pnpm
*   Compte Supabase
*   Clé API Google Gemini
*   Compte Vercel & Vercel CLI (optionnel)
*   Git

### 5.2. Configuration Locale

1.  Cloner le dépôt.
2.  `npm install`
3.  Créer `.env.local` à la racine.
4.  Ajouter les variables d'environnement locales :
    ```env
    NEXT_PUBLIC_SUPABASE_URL=VOTRE_URL_SUPABASE
    NEXT_PUBLIC_SUPABASE_ANON_KEY=VOTRE_CLE_ANON_SUPABASE
    GEMINI_API_KEY=VOTRE_CLE_API_GEMINI
    # SUPABASE_SERVICE_ROLE_KEY=VOTRE_CLE_SERVICE_SUPABASE (Attention!)
    ```
5.  Appliquer le schéma SQL à votre instance Supabase.
6.  `npm run dev` (accès via `http://localhost:3000`).

### 5.3. Déploiement Vercel

1.  Pousser le code sur Git (GitHub/GitLab/Bitbucket).
2.  Importer le projet Git sur Vercel.
3.  Configurer les variables d'environnement **sur Vercel** (Settings -> Environment Variables), en marquant les clés API comme secrètes :
    *   `NEXT_PUBLIC_SUPABASE_URL`
    *   `NEXT_PUBLIC_SUPABASE_ANON_KEY`
    *   `GEMINI_API_KEY` (Secret)
    *   `SUPABASE_SERVICE_ROLE_KEY` (Secret)
4.  Déployer (automatique au push sur la branche principale par défaut).

---

## 6. Développement et Défis (Synthèse)

*   **Approche :** Développement itératif par blocs fonctionnels (Auth -> Chat UI -> IA -> Persistance -> Gestion Profil -> Déploiement).
*   **Défis Majeurs :** Gestion de l'asynchronisme (React State & API calls), configuration RLS et triggers Supabase, intégration API Gemini (historique, sécurité), rendu Markdown complexe (éviter erreurs hydratation), adaptation à l'architecture serverless Vercel, gestion d'erreurs full-stack.
*   **Apprentissages :** Importance d'une archi découplée, compréhension fine de Supabase RLS/Auth, spécificités serverless, débogage multi-niveaux.

---

## 7. Conclusion et Perspectives

### 7.1. Bilan v1.0

Seshat v1.0 est une base fonctionnelle solide, intégrant avec succès Next.js, Supabase et Gemini pour créer un assistant IA spécialisé. L'architecture est moderne, sécurisée et prête pour l'évolution.

### 7.2. Pistes d'Amélioration

*   Organisation des conversations (dossiers, recherche).
*   Fonctions d'édition / prise de notes.
*   Personnalisation de l'IA (modèles, prompts).
*   Gestion de fichiers (contexte).
*   Collaboration / Partage.
*   Monétisation (Plans Pro via Stripe).
*   Améliorations UX/UI (Thèmes, accessibilité).
*   Exportation de données.

--- FIN DE LA DOCUMENTATION TECHNIQUE ---