# 🚀 LF Partner Hub — Plateforme d'Onboarding Partenaires LesFurets

> **Back-office multi-tenant intelligent** pour la gestion des intégrations partenaires d'assurance, remplaçant le CDC Excel par un système moderne de formulaires versionnés, mapping no-code, et règles d'exclusion configurables.

![Version](https://img.shields.io/badge/version-2.0.0-blue)
![React](https://img.shields.io/badge/React-18.3.1-61dafb)
![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178c6)
![Supabase](https://img.shields.io/badge/Backend-Supabase-3ecf8e)

---

## 📋 Table des matières

- [🎯 Vue d'ensemble](#-vue-densemble)
- [✨ Fonctionnalités principales](#-fonctionnalités-principales)
- [🏗️ Architecture technique](#️-architecture-technique)
- [📊 Modèle de données](#-modèle-de-données)
- [🔐 Système d'authentification](#-système-dauthentification)
- [🧩 Composants clés](#-composants-clés)
- [📱 Structure des pages](#-structure-des-pages)
- [⚡ Installation](#-installation)
- [🚀 Déploiement](#-déploiement)
- [🔧 Configuration](#-configuration)
- [📚 Guide d'utilisation](#-guide-dutilisation)
- [🤝 Contribution](#-contribution)

---

## 🎯 Vue d'ensemble

### Contexte

La **LF Partner Hub** est une plateforme web full-stack conçue pour **industrialiser l'onboarding des partenaires assureurs** chez LesFurets. Elle remplace le processus manuel basé sur des fichiers Excel (Cahier des Charges) par un système moderne offrant :

- **Formulaires dynamiques versionnés** pour le cadrage technique
- **Mapping no-code** entre les champs LesFurets et les APIs partenaires
- **Règles d'exclusion configurables** avec support des formules produit
- **Simulation en temps réel** des appels API
- **Gestion documentaire** avec versioning et audit trail
- **Multi-tenancy** avec isolation des données par organisation

### Objectifs

| Objectif | Solution |
|----------|----------|
| Remplacer le CDC Excel | Formulaires dynamiques par ligne produit |
| Simplifier le mapping | Designer visuel avec suggestions IA |
| Gérer les exclusions | Moteur de règles DSL avec conditions combinatoires |
| Assurer la traçabilité | Versioning, audit log, historique complet |
| Sécuriser l'accès | RBAC + RLS (Row Level Security) |

---

## ✨ Fonctionnalités principales

### 🔄 Mapping Manager (No-Code)

Le cœur du système permettant de configurer la transformation des données entre LesFurets et les partenaires.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        MAPPING COMBINATOIRE                                 │
├─────────────────────┬───────────────────────────┬──────────────────────────┤
│  Champs LesFurets   │    Configuration Règle    │     Règles créées        │
│  ─────────────────  │   ─────────────────────   │   ─────────────────      │
│  □ subscriber.name  │   Type: Concat (N→1)      │   ✓ Nom complet → name   │
│  □ subscriber.email │   Transform: uppercase     │   ✓ Email → contact      │
│  □ vehicle.brand    │   Défaut: "N/A"           │   ✓ Véhicule → car_info  │
│  □ vehicle.model    │   Obligatoire: ✓          │                          │
└─────────────────────┴───────────────────────────┴──────────────────────────┘
```

**Types de combinaison supportés :**

| Type | Description | Exemple |
|------|-------------|---------|
| `direct` | 1 champ LF → 1 champ partenaire | `email` → `contact_email` |
| `concat` | N champs LF → 1 champ partenaire | `first_name + last_name` → `fullName` |
| `split` | 1 champ LF → N champs partenaire | `address` → `street, city, zip` |
| `lookup` | Table de correspondance | `M` → `1`, `MME` → `2` |
| `formula` | Expression calculée | `age * 12` → `months` |
| `conditional` | Logique si/sinon | `if premium then 'A' else 'B'` |

**Transformations disponibles :**
- `uppercase`, `lowercase`, `trim`
- `toDate`, `toNumber`, `toBoolean`
- `format_date`, `round`, `abs`

### 🚫 Exclusion Rules Engine

Système de règles pour gérer les critères de refus ou d'avertissement.

```
┌───────────────────────────────────────────────────────────────────────────┐
│                     RÈGLE D'EXCLUSION                                     │
├───────────────────────────────────────────────────────────────────────────┤
│  Nom: Âge conducteur limite                                               │
│  ─────────────────────────────────────────────────────────────────────── │
│  Conditions (ET):                                                         │
│    • subscriber.birth_date  >  75 ans                                     │
│    • vehicle.power          >  150 CV                                     │
│  ─────────────────────────────────────────────────────────────────────── │
│  Action: EXCLUSION          Catégorie: Risque          Priorité: 1       │
└───────────────────────────────────────────────────────────────────────────┘
```

**Comparateurs disponibles :**
- Égalité : `==`, `!=`
- Comparaison : `<`, `<=`, `>`, `>=`
- Listes : `IN`, `NOT_IN`
- Texte : `STARTS_WITH`, `ENDS_WITH`, `CONTAINS`
- Nullité : `IS_NULL`, `IS_NOT_NULL`
- Plage : `BETWEEN`

**Actions possibles :**
| Action | Description |
|--------|-------------|
| `EXCLUDE` | Blocage total de la souscription |
| `WARNING` | Avertissement affiché à l'utilisateur |
| `REQUIRE_VALIDATION` | Validation manuelle requise |

### 📦 Gestion des Formules

Les règles de mapping et d'exclusion peuvent être liées à des **formules produit** spécifiques :

```typescript
interface Formula {
  id: string;
  code: string;      // Ex: "F1_BASE", "F2_CONFORT", "F3_PREMIUM"
  name: string;      // Ex: "Formule Essentielle"
  description?: string;
}
```

**Fonctionnalités :**
- Sélection de formule avec filtrage des règles
- Duplication de règles entre formules
- Vue "Toutes les formules" pour règles globales
- Compteur de règles par formule

### 📄 Gestion Documentaire

Système complet de gestion des documents contractuels :

- **Types supportés** : CG (Conditions Générales), IPID, Logo, Swagger, Samples
- **Workflow** : Upload → Draft → Validation → Publication
- **Versioning** : Historique complet avec rollback possible
- **Expiration automatique** : Drafts expirés après 7 jours
- **Audit trail** : Traçabilité complète des actions

### 🧪 Live Simulator

Testez vos configurations en temps réel :

```
┌─────────────────────────────────────────────────────────────────────┐
│  SIMULATION TARIFICATION                                            │
├─────────────────────────────────────────────────────────────────────┤
│  Profil: Conducteur standard 35 ans                                 │
│  Environnement: TEST                                                │
├─────────────────────────────────────────────────────────────────────┤
│  Requête envoyée:                    │  Réponse reçue:              │
│  {                                   │  {                           │
│    "subscriber": {                   │    "status": "OK",           │
│      "birth_date": "1990-01-15",     │    "premium": 45.99,         │
│      "email": "test@example.com"     │    "guarantees": [...]       │
│    }                                 │  }                           │
│  }                                   │                              │
├─────────────────────────────────────────────────────────────────────┤
│  ✓ Durée: 245ms    ✓ Status: SUCCESS    ✓ Mapping: 100%            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Architecture technique

### Stack technologique

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FRONTEND                                       │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────────────────┐│
│  │   React 18  │ │  TypeScript │ │ Tailwind CSS│ │      shadcn/ui         ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────────────────┘│
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌────────────────────────┐│
│  │TanStack Query│ │React Router │ │React H. Form│ │         Zod            ││
│  └─────────────┘ └─────────────┘ └─────────────┘ └────────────────────────┘│
├─────────────────────────────────────────────────────────────────────────────┤
│                              BACKEND                                        │
│  ┌───────────────────────────────────────────────────────────────────────┐ │
│  │                         SUPABASE                                      │ │
│  │  ┌──────────────┐ ┌───────────────┐ ┌─────────────┐ ┌───────────────┐ │ │
│  │  │  PostgreSQL  │ │     Auth      │ │   Storage   │ │Edge Functions │ │ │
│  │  │  + RLS       │ │  Email/OAuth  │ │   Buckets   │ │     Deno      │ │ │
│  │  └──────────────┘ └───────────────┘ └─────────────┘ └───────────────┘ │ │
│  └───────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```
•

### Edge Functions

| Fonction | Description | Endpoint |
|----------|-------------|----------|
| `partners-api` | API REST CRUD partenaires | `/rest/v1/partners-api` |
| `ai-mapping-suggest` | Suggestions IA de mapping | `/rest/v1/ai-mapping-suggest` |
| `ai-exclusion-suggest` | Suggestions IA d'exclusions | `/rest/v1/ai-exclusion-suggest` |
| `dev-seed` | Seed de données de développement | `/rest/v1/dev-seed` |

### Hooks React personnalisés

```typescript
// Authentification et profil utilisateur
useAuth()              → { user, authUser, signIn, signUp, signOut, loading }

// Données Supabase avec cache TanStack Query
useSupabaseData()      → { partners, products, offers, loading, error }

// API partenaires avec mutations
usePartnersAPI()       → { partners, createPartner, updatePartner, deletePartner }

// Règles de mapping persistées
useMappingRules()      → { rules, saveRule, deleteRule, loading }

// Champs partenaires importés
usePartnerFields()     → { fields, importFromSwagger, deleteField }

// Suggestions IA
useAIMappingSuggestions()    → { getSuggestions, suggestions, loading }
useAIExclusionSuggestions()  → { getSuggestions, suggestions, loading }

// Notifications temps réel
useNotifications()     → { notifications, markAsRead, unreadCount }

// Simulation de rôles (dev)
useRoleSimulator()     → { simulatedRole, setSimulatedRole }
```

---

## 📊 Modèle de données

### Schéma relationnel

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  organizations  │────<│     users       │────<│   memberships   │
│─────────────────│     │─────────────────│     │─────────────────│
│ id              │     │ id              │     │ user_id         │
│ name            │     │ email           │     │ org_id          │
│ type (INTERNAL/ │     │ name            │     │ role            │
│       PARTNER)  │     │ org_id (FK)     │     │ created_at      │
│ domain_whitelist│     │ is_active       │     └─────────────────┘
└─────────────────┘     │ last_login      │
         │              └─────────────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│    partners     │────<│    products     │────<│     offers      │
│─────────────────│     │─────────────────│     │─────────────────│
│ id              │     │ id              │     │ id              │
│ org_id (FK)     │     │ partner_id (FK) │     │ product_id (FK) │
│ name            │     │ line (AUTO/MRH/ │     │ label           │
│ status          │     │      SANTE...)  │     │ status (DRAFT/  │
│ created_at      │     │ name            │     │        TEST/PROD│
│ updated_at      │     │ created_at      │     │ effective_from  │
└─────────────────┘     │ updated_at      │     │ effective_to    │
                        └─────────────────┘     └─────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         ▼                       ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  mapping_rules  │     │ partner_fields  │     │environment_conf │
│─────────────────│     │─────────────────│     │─────────────────│
│ id              │     │ id              │     │ id              │
│ product_id (FK) │     │ product_id (FK) │     │ product_id (FK) │
│ lf_field_keys[] │     │ path            │     │ env (TEST/PROD) │
│ partner_paths[] │     │ type            │     │ base_url        │
│ combination_type│     │ required        │     │ auth_kind       │
│ transform_expr  │     │ description     │     │ credentials_ref │
│ is_required     │     │ example         │     │ timeout_ms      │
│ ai_confidence   │     │ source          │     └─────────────────┘
│ execution_order │     └─────────────────┘
└─────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ exclusion_rules │     │  published_docs │     │    doc_drafts   │
│─────────────────│     │─────────────────│     │─────────────────│
│ id              │     │ id              │     │ id              │
│ offer_id (FK)   │     │ type (cg/ipid)  │     │ type            │
│ rule_json       │     │ partenaire      │     │ partenaire      │
│ scope           │     │ produit         │     │ produit         │
│ enabled         │     │ formules[]      │     │ formules[]      │
│ created_at      │     │ version_number  │     │ storage_path    │
└─────────────────┘     │ current_url     │     │ expires_at      │
                        │ published_by    │     │ status          │
                        └─────────────────┘     └─────────────────┘
```

### Types énumérés (Enums)

```sql
-- Rôles utilisateur
CREATE TYPE user_role AS ENUM (
  'SUPER_ADMIN',      -- Accès total, gestion multi-org
  'LF_ADMIN',         -- Admin LesFurets
  'LF_DEV',           -- Développeur LesFurets
  'LF_TAM',           -- Technical Account Manager
  'PARTNER_ADMIN',    -- Admin partenaire
  'PARTNER_EDITOR',   -- Éditeur partenaire
  'PARTNER_VIEWER'    -- Lecture seule partenaire
);

-- Lignes d'assurance
CREATE TYPE insurance_line AS ENUM (
  'AUTO', 'MOTO', 'SANTE', 'MRH', 'EMPRUNTEUR', 'ENERGIE'
);

-- Statuts partenaire
CREATE TYPE partner_status AS ENUM ('draft', 'active', 'inactive');

-- Statuts offre
CREATE TYPE offer_status AS ENUM ('DRAFT', 'TEST', 'PROD');

-- Types de documents
CREATE TYPE doc_type AS ENUM ('cg', 'ipid');
CREATE TYPE document_type AS ENUM (
  'LOGO', 'IPID', 'CG', 'SWAGGER', 'SAMPLES_TARIF', 'SAMPLES_MER'
);
```

---

## 🔐 Système d'authentification

### Hiérarchie des rôles

```
SUPER_ADMIN
    │
    ├── LF_ADMIN
    │       │
    │       ├── LF_DEV
    │       └── LF_TAM
    │
    └── PARTNER_ADMIN
            │
            ├── PARTNER_EDITOR
            └── PARTNER_VIEWER
```

### Matrice des permissions

| Action | SUPER_ADMIN | LF_ADMIN | LF_DEV | LF_TAM | PARTNER_ADMIN | PARTNER_EDITOR | PARTNER_VIEWER |
|--------|:-----------:|:--------:|:------:|:------:|:-------------:|:--------------:|:--------------:|
| Gérer organisations | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Gérer utilisateurs internes | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Créer partenaires | ✅ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ |
| Configurer mapping | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Configurer exclusions | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Upload documents | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ |
| Publier en production | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Voir données partenaire | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Exécuter simulations | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

### Row Level Security (RLS)

Toutes les tables sont protégées par RLS. Exemple de politique :

```sql
-- Les utilisateurs ne voient que les partenaires de leur organisation
CREATE POLICY "Users see own org partners" ON partners
  FOR SELECT
  USING (
    org_id IN (
      SELECT org_id FROM users WHERE id = auth.uid()
    )
    OR
    EXISTS (
      SELECT 1 FROM memberships 
      WHERE user_id = auth.uid() 
      AND role IN ('SUPER_ADMIN', 'LF_ADMIN', 'LF_DEV', 'LF_TAM')
    )
  );
```

---

## 🧩 Composants clés

### CombinatorialMappingDesigner

Designer visuel 3 colonnes pour créer des règles de mapping.

```tsx
<CombinatorialMappingDesigner
  productId="uuid-product"
  productLine="AUTO"
  formulas={formulas}
  partnerFields={partnerFields}
  existingRules={rules}
  onRulesChange={handleRulesChange}
/>
```

**Caractéristiques :**
- Sélection multiple de champs LesFurets (colonne gauche)
- Configuration de règle avec type, transformation, valeur par défaut (centre)
- Liste des règles avec expansion/contraction (colonne droite)
- Suggestions IA intégrées
- Sauvegarde vers Supabase

### CombinatorialExclusionDesigner

Designer de règles d'exclusion avec conditions combinatoires.

```tsx
<CombinatorialExclusionDesigner
  productId="uuid-product"
  productLine="SANTE"
  formulas={formulas}
  existingRules={exclusionRules}
  onRulesChange={handleExclusionsChange}
/>
```

**Caractéristiques :**
- Sélection de champs pour conditions
- Opérateurs logiques AND/OR
- Comparateurs typés selon le type de champ
- Actions configurables (exclusion, warning, validation)
- Catégorisation et priorité

### FormulaRuleManager

Gestionnaire de formules pour filtrer et dupliquer les règles.

```tsx
<FormulaRuleManager
  formulas={formulas}
  selectedFormula={selectedFormula}
  onFormulaChange={setSelectedFormula}
  rulesCountByFormula={rulesCount}
  onDuplicateRules={handleDuplicate}
/>
```

### DocumentManager

Gestionnaire de documents avec workflow complet.

```tsx
<DocumentManager
  drafts={drafts}
  publishedDocs={publishedDocs}
  onUpload={handleUpload}
  onPublish={handlePublish}
  onDelete={handleDelete}
/>
```

---

## 📱 Structure des pages

```
/                               # Accueil avec sélection rapide
├── /auth                       # Connexion / Inscription
├── /dashboard                  # Tableau de bord global
│
├── /partners                   # Liste des partenaires
│   └── /:id                    # Fiche partenaire
│       ├── /overview           # KPIs et vue d'ensemble
│       ├── /products           # Gestion des produits
│       ├── /catalog            # Catalogue (formules, garanties)
│       ├── /endpoints          # Configuration des endpoints API
│       ├── /mapping            # 🎯 Mapping Manager
│       ├── /exclusions         # 🚫 Règles d'exclusion
│       ├── /documents          # Documents contractuels
│       ├── /cadrage            # Formulaires CDC dynamiques
│       ├── /tests              # Test Runner
│       ├── /codegen            # Génération de code
│       └── /export             # Export des configurations
│
├── /documents                  # Gestion documentaire globale
│   ├── /upload                 # Upload de documents
│   ├── /drafts                 # Brouillons en attente
│   └── /history                # Historique des publications
│
├── /fields                     # Dictionnaire des champs
├── /import-wizard              # Assistant d'import CSV/Excel
│
├── /users                      # Gestion des utilisateurs (Admin)
├── /admin/orgs                 # Gestion des organisations (Super Admin)
├── /settings                   # Paramètres application
└── /profile                    # Profil utilisateur
```

---

## ⚡ Installation

### Prérequis

- **Node.js** 18+ et npm/pnpm/bun
- Compte **Supabase** (ou accès au projet existant)

### Installation locale

```bash
# Cloner le repository
git clone <repository-url>
cd lf-partner-hub

# Installer les dépendances
npm install

# Configurer l'environnement
cp .env.example .env
# Éditer .env avec les credentials Supabase

# Lancer le serveur de développement
npm run dev
```

L'application sera accessible sur `http://localhost:5173`

### Variables d'environnement

```env
# Supabase
VITE_SUPABASE_URL=https://mzbpynlbhtjzgfopxfvf.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 🚀 Déploiement

### Via Lovable (Recommandé)

1. Ouvrir le projet sur [Lovable](https://lovable.dev/projects/8efd7081-258f-487e-af10-a0f19a57a2bd)
2. Cliquer sur **Share → Publish**
3. Configurer le domaine personnalisé si nécessaire

### Build manuel

```bash
# Build de production
npm run build

# Preview local du build
npm run preview
```

Les fichiers statiques sont générés dans le dossier `dist/`.

---

## 🔧 Configuration

### Dictionnaire des champs

Le fichier `src/data/insuranceFieldsDictionary.ts` contient tous les champs standards LesFurets :

```typescript
// Structure d'un champ
interface InsuranceField {
  id: string;
  key: string;           // Ex: "subscriber.birth_date"
  label: string;         // Ex: "Date de naissance"
  type: FieldType;       // 'string' | 'number' | 'date' | 'boolean' | 'enum'
  required: boolean;
  description: string;
  category: string;      // Ex: "Souscripteur", "Véhicule"
  productLines: ProductLine[];  // ['AUTO', 'MRH', ...]
  enumValues?: string[];
  example?: string;
  validation?: string;
}
```

**Catégories disponibles :**
- Commun : Dates, Souscripteur, Adresse, Paiement, Antécédents
- Auto : Véhicule, Conducteur, Permis, Usage, Stationnement
- MRH : Logement, Occupation, Dépendances
- Santé : Bénéficiaires, Régime, Besoins
- Emprunteur : Prêt, Quotité, Garanties

### Schémas CDC

Le fichier `src/data/cdcSchema.ts` définit les formulaires de cadrage par produit :

```typescript
interface CDCSchema {
  version: string;
  product: 'SANTE' | 'MRH' | 'EMPRUNTEUR' | 'ENERGIE';
  sections: CDCSection[];
  metadata: { created_at, updated_at, author };
}

interface CDCSection {
  id: string;
  label: string;
  description?: string;
  icon?: string;
  fields: CDCField[];
  order: number;
}
```

---

## 📚 Guide d'utilisation

### 1. Onboarding d'un nouveau partenaire

```
1. Créer le partenaire (Dashboard → Nouveau partenaire)
2. Ajouter un produit (Ex: SANTE)
3. Remplir le formulaire CDC (Cadrage)
4. Importer le Swagger de l'API partenaire
5. Configurer le mapping des champs
6. Définir les règles d'exclusion
7. Tester avec le simulateur
8. Publier en production
```

### 2. Configuration du mapping

```
1. Aller sur Mapping Manager du partenaire
2. Sélectionner la formule (ou "Toutes les formules")
3. Cliquer sur les champs LesFurets à mapper
4. Choisir le type de combinaison
5. Sélectionner le(s) champ(s) partenaire cible
6. Configurer la transformation si nécessaire
7. Ajouter la règle → Elle apparaît dans la liste
8. Sauvegarder
```

### 3. Configuration des exclusions

```
1. Aller sur Exclusions du partenaire
2. Sélectionner la formule concernée
3. Créer une nouvelle règle
4. Définir les conditions (champ, opérateur, valeur)
5. Choisir le type logique (AND/OR)
6. Sélectionner l'action (Exclusion/Warning/Validation)
7. Catégoriser et prioriser
8. Sauvegarder
```

### 4. Duplication de règles

```
1. Dans le panneau Formules, cliquer "Dupliquer"
2. Sélectionner la formule source
3. Cocher les formules cibles
4. Confirmer la duplication
```

---

## 🤝 Contribution

### Workflow Git

```bash
# Créer une branche feature
git checkout -b feature/nouvelle-fonctionnalite

# Développer et committer
git commit -m "feat: description de la fonctionnalité"

# Push et créer une PR
git push origin feature/nouvelle-fonctionnalite
```

### Conventions de code

- **TypeScript strict** obligatoire
- **ESLint** pour la qualité du code
- Composants **fonctionnels** avec hooks
- **Design tokens** Tailwind (pas de couleurs en dur)
- Fichiers **< 400 lignes** (refactoriser si nécessaire)

### Structure des commits

```
feat: nouvelle fonctionnalité
fix: correction de bug
docs: documentation
style: formatage
refactor: refactorisation
test: ajout de tests
chore: maintenance
```

---

## 📄 Licence

Propriétaire — **LesFurets.com** © 2024-2025

---

## 🔗 Liens utiles

| Ressource | URL |
|-----------|-----|
| Projet Lovable | [lovable.dev/projects/8efd7081-258f-487e-af10-a0f19a57a2bd](https://lovable.dev/projects/8efd7081-258f-487e-af10-a0f19a57a2bd) |
| Dashboard Supabase | [supabase.com/dashboard](https://supabase.com/dashboard/project/mzbpynlbhtjzgfopxfvf) |
| Documentation technique | [docs/](./docs/) |

---

<p align="center">
  <strong>LF Partner Hub</strong> — Industrialiser l'onboarding partenaires
  <br/>
  <sub>Version 2.0.0 • React 18 • TypeScript • Supabase</sub>
</p>
