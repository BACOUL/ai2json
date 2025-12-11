# AI2JSON – Plan de création v0.1 (Build Plan)

Ce fichier décrit **tout** ce qu’il faut pour rendre AI2JSON crédible, opérationnel et montrable (sans monétisation avancée).  
À cocher au fur et à mesure.

---

## 0. Fondations du projet

- [ ] Créer le repo GitHub `ai2json`
  - [ ] Fichier `README.md` (sera rempli plus tard)
  - [ ] Dossier `api/` (Worker Cloudflare)
  - [ ] Dossier `site/` (landing Vercel)
  - [ ] Dossier `spec/` (spécification du format)

- [ ] Créer un compte / config Cloudflare Workers (si pas déjà fait)
- [ ] Créer un projet Vercel pour le site (url temporaire)

> Le domaine `ai2json.com` pourra être acheté **à la fin** quand tout est validé.

---

## 1. Spécification du format `webpage.ai.v0.1`

**Objectif :** figer le contrat JSON que l’API doit respecter.

- [ ] Créer `spec/webpage-ai-v0.1.md`
- [ ] Définir les champs obligatoires :
  - [ ] `spec` (string, ex: `"webpage.ai.v0.1"`)
  - [ ] `url`
  - [ ] `canonical_url`
  - [ ] `last_fetched` (ISO 8601, UTC)
  - [ ] `language` (code BCP47, ex: `fr`, `en`)
  - [ ] `content_hash` (`sha256:<hex>`)
  - [ ] `title`
  - [ ] `summary` (3–5 phrases max, factuel)
  - [ ] `type` (`"article" | "homepage" | "unknown"`)
  - [ ] `sections[]` :
    - [ ] `heading`
    - [ ] `level` (1/2/3/0)
    - [ ] `id` (slug ou identifiant stable)
    - [ ] `text`
  - [ ] `links[]` :
    - [ ] `text`
    - [ ] `href` (URL absolue)
    - [ ] `rel` (`"reference" | "internal" | "external" | "unknown"`)
  - [ ] `meta` (objet best-effort)
  - [ ] `proof` (optionnel pour plus tard, peut rester vide au début)

- [ ] Ajouter **un exemple JSON complet** (copier/coller un exemple réaliste)
- [ ] Préciser :
  - [ ] Ce qui se passe si une info est inconnue (champ absent ou vide)
  - [ ] Que la spec est **stable pour v0.1** (pas de breaking change)

---

## 2. API AI2JSON – Worker `/v1/transform` (MVP)

### 2.1 Squelette & routing

- [ ] Créer `api/worker.js` (ou `index.js/ts` selon stack)
- [ ] Définir la route `GET /v1/transform`
- [ ] Lire le paramètre `url` dans la query string
- [ ] Si `url` manquante → `400` + JSON `{ "error": "missing_url" }`
- [ ] Retourner provisoirement un JSON statique de test pour valider le pipeline

### 2.2 Auth par API Key (protection dès le début)

- [ ] Ajouter une variable d’environnement `AI2JSON_API_KEY`
- [ ] Vérifier le header `Authorization: Bearer <API_KEY>`
- [ ] Si clé absente / invalide → `401` + `{ "error": "unauthorized" }`
- [ ] Prévoir **une seule clé globale** pour v0.1 (simple)

### 2.3 Fetch de la page cible

- [ ] `fetch(url)` avec :
  - [ ] Timeout raisonnable
  - [ ] Suivi des redirections
- [ ] Si erreur réseau → `502` + `{ "error": "upstream_error" }`
- [ ] Si statut HTTP 4xx/5xx → renvoyer `502` ou `422` avec message clair
- [ ] Log minimal (console.log) pour debug (sans données sensibles)

### 2.4 Extraction HTML → texte principal

- [ ] Récupérer le `text/html` complet
- [ ] Extraire :
  - [ ] `<title>`
  - [ ] Canonical (`<link rel="canonical">`) si présent
- [ ] Détecter un bloc principal de contenu :
  - [ ] Essayer `<main>`, `<article>`, ou un container central
  - [ ] Fallback sur `<body>` si aucune structure claire
- [ ] Supprimer :
  - [ ] `<script>`, `<style>`, `<nav>`, `<footer>`, `<header>` autant que possible
- [ ] Nettoyer le texte :
  - [ ] Retirer les multiples espaces
  - [ ] Normaliser les sauts de ligne

### 2.5 Structuration en sections

- [ ] Identifier les headings (`<h1>`, `<h2>`, `<h3>`) dans le bloc principal
- [ ] Construire `sections[]` :
  - [ ] `heading` = texte du heading (ou `null`)
  - [ ] `level` = 1, 2, 3, ou 0 si pas de heading
  - [ ] `id` = slug du heading ou `section-1`, `section-2`, etc.
  - [ ] `text` = paragraphe(s) qui suivent jusqu’au heading suivant
- [ ] Cas fallback :
  - [ ] Si aucun heading exploitable → une seule section `id: "body"`

### 2.6 Métadonnées basiques

- [ ] `language` :
  - [ ] Détection très simple (FR/EN) ou via petite lib
- [ ] `type` :
  - [ ] `"article"` si présence de `<article>` ou longueur suffisante
  - [ ] `"homepage"` si URL racine + structure légère
  - [ ] `"unknown"` sinon
- [ ] `meta.site_name` (balises OpenGraph, `<meta property="og:site_name">` ou host)
- [ ] `meta.author` (si trouvable)
- [ ] `meta.published_at` / `meta.updated_at` (si trouvables dans `<meta>`)

### 2.7 Résumé v0.1 (sans LLM au début)

- [ ] Construire `summary` en prenant :
  - [ ] Les 1–2 premiers paragraphes significatifs
  - [ ] Limiter à une longueur raisonnable (ex: ~600–800 caractères)
- [ ] Garder le résumé **factuel**, sans extrapolation

### 2.8 Hash & timestamps

- [ ] `last_fetched` = `new Date().toISOString()` (UTC)
- [ ] `content_hash` = SHA-256 de `sections[].text` concaténé (ordre stable)
- [ ] Vérifier que le hash a le format `sha256:<hex>`

### 2.9 Assemblage de la réponse JSON

- [ ] Construire l’objet selon `webpage.ai.v0.1` :
  - [ ] `spec`
  - [ ] `url`
  - [ ] `canonical_url`
  - [ ] `last_fetched`
  - [ ] `language`
  - [ ] `content_hash`
  - [ ] `title`
  - [ ] `summary`
  - [ ] `type`
  - [ ] `sections[]`
  - [ ] `links[]` (au début : vide ou liens simples)
  - [ ] `meta`
  - [ ] `proof` (pour v0.1 : `{}` ou absent)

- [ ] Vérifier que la réponse :
  - [ ] Est bien du JSON valide
  - [ ] Suit exactement la spec (noms de champs, types)

### 2.10 Rate limiting minimal

- [ ] Mettre en place un rate-limit **simple** :
  - [ ] Ex: max 60 requêtes / minute par `API_KEY`
- [ ] Logguer les dépassements pour identifier les abus

---

## 3. Site AI2JSON – Landing + Docs (Vercel)

### 3.1 Structure de base

- [ ] Créer `site/index.html`
- [ ] Sections obligatoires :
  - [ ] **Hero** : titre + sous-titre + CTA
    - Ex: “AI2JSON — Turn any webpage into clean AI-ready JSON.”
  - [ ] **Example request** :
    - [ ] Bloc `curl` vers `https://api.ai2json.com/v1/transform?url=...`
  - [ ] **Example response** :
    - [ ] Extrait de JSON `webpage.ai.v0.1`
  - [ ] **How it works** (4 étapes)
  - [ ] **webpage.ai.v0.1** (liste des champs principaux)
  - [ ] **Use cases** (agents, RAG, outils dev)
  - [ ] **Pricing** (Free / Pro / Enterprise – même théorique au début)
  - [ ] **Footer** : lien GitHub, contact (email)

### 3.2 Déploiement initial

- [ ] Déployer `site/` sur Vercel (URL provisoire)
- [ ] Vérifier :
  - [ ] Responsive
  - [ ] Liens internes fonctionnels
  - [ ] Exemples cohérents avec l’API réelle

---

## 4. GitHub & documentation

### 4.1 README.md

- [ ] Décrire clairement ce qu’est AI2JSON :
  - [ ] “Turn any public webpage into clean, AI-ready JSON.”
- [ ] Ajouter :
  - [ ] Exemple de requête `curl`
  - [ ] Exemple de réponse JSON complet
  - [ ] Lien vers `spec/webpage-ai-v0.1.md`
  - [ ] Minimal “How it works”
  - [ ] Roadmap v0.1 → v0.2

### 4.2 Spécification

- [ ] Pousser `spec/webpage-ai-v0.1.md` dans GitHub
- [ ] Ajouter lien vers cette spec dans le README et sur le site

---

## 5. Protection & Free plan (v0.1)

### 5.1 Pas d’accès anonyme

- [ ] Exiger **toujours** une API key
- [ ] Pas d’endpoint public sans auth (même pour tests)

### 5.2 Free limité

- [ ] Définir une clé Free (utilisée éventuellement pour démo interne)
- [ ] Limiter :
  - [ ] N requêtes / jour (ex: 50 ou 100)
  - [ ] N requêtes / minute
- [ ] Indiquer dans la doc : “Contact us for higher limits / Pro plan”

---

## 6. Tests v0.1 – Ready to show

**Considérer AI2JSON v0.1 comme prêt à montrer lorsque :**

- [ ] L’API `/v1/transform` fonctionne correctement sur au moins :
  - [ ] 1 article de blog
  - [ ] 1 doc technique
  - [ ] 1 page d’accueil
  - [ ] 1 page très longue
  - [ ] 1 page très simple
- [ ] Les erreurs classiques sont bien gérées :
  - [ ] URL manquante
  - [ ] URL invalide
  - [ ] Timeout
  - [ ] 4xx/5xx
  - [ ] API key manquante / invalide
- [ ] Le site :
  - [ ] Explique clairement le produit
  - [ ] Montre un exemple réaliste
  - [ ] Contient la doc de l’endpoint
- [ ] GitHub :
  - [ ] README propre
  - [ ] Spec `webpage.ai.v0.1` en place
- [ ] Tu es capable de :
  - [ ] Copier/coller l’exemple `curl` depuis le site
  - [ ] Obtenir une réponse correcte en temps réel

---

## 7. Après v0.1 (v0.2 et +, à ne faire qu’une fois v0.1 stable)

- [ ] Ajouter un vrai résumé via LLM (summary plus intelligent)
- [ ] Gérer JS rendering (Playwright / headless) pour pages dynamiques
- [ ] Ajouter un endpoint batch (`POST /v1/transform/batch`)
- [ ] Ajouter un système de création de clés Free / Pro via une petite interface
- [ ] Ajouter un début de dashboard (Stats, quotas)
- [ ] Intégrer éventuellement TimeProofs dans `proof` (optionnel)

---
