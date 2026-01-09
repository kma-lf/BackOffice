# Questionnaire Produit - Système de Gestion Documentaire CG/IPID

## 1) Périmètre & objectifs

**Q: Objectif cible de ce sprint : POC uniquement (qualif), ou premiers incréments utilisables en prod ?**
R: Premiers incréments utilisables en prod avec rollout progressif. Phase 1: environnement qualif + 2 partenaires pilotes, Phase 2: production complète.

**Q: Produits concernés au départ (Santé, MRH, Auto…?) et partenaires pilotes ?**
R: Démarrage par Santé (moins de variabilité réglementaire), puis MRH et Auto. 2 partenaires pilotes : 1 gros volume + 1 partenaire technique avancé.

**Q: Utilisateurs visés par l'interface d'upload : TAM/KAM internes uniquement au début, ou ouverture partenaire à court terme ?**
R: Phase 1: TAM/KAM internes uniquement. Phase 2 (T+6 mois): ouverture partenaires avec workflow d'approbation obligatoire.

## 2) Stockage GCP & structure des objets

**Q: Buckets par environnement (qualif / préprod / prod) : quelles références exactes ?**
R: 3 buckets séparés : `lf-documents-qualif`, `lf-documents-preprod`, `lf-documents-prod`. Conservation de la chaîne CI existante avec modification du job `upload-statics`.

**Q: Organisation des répertoires : conditionsGenerales/ et ipid/ restent-ils les préfixes standards ?**
R: Oui, conservation des préfixes existants pour compatibilité. Structure : `/{type}/{partenaire}/{produit}/{version}/`

**Q: Convention de nommage des fichiers ?**
R: `{PARTENAIRE}_{PRODUIT}_{FORMULE}_{YYYYMMDD}_{LANG}_{MARQUE}.pdf`
Exemple: `ALLIANZ_SANTE_PREMIUM_20241201_FR_LF.pdf`

**Q: Versioning & lifecycle : active-t-on la gestion de versions GCS ?**
R: Oui, versioning GCS activé avec rétention 3 ans (vs 10 ans actuels). Archivage automatique après 1 an vers classe Nearline.

**Q: Métadonnées à stocker ?**
R: JSON metadata stockées : partenaire, produit, formule, dateEffet, dateExpiration, version, statut, type (static/dynamic), langue, marque, utilisateurUpload, dateUpload, checksumMD5.

## 3) Publication & URLs (PPS, Product Catalog, BO)

**Q: On conserve le schéma d'URL public ?**
R: Oui, conservation du schéma `https://static.lesfurets.com/conditionsGenerales/<fichier>.pdf` avec service Java de résolution d'URLs pour compatibilité.

**Q: Le front B2C/BO continue d'assembler l'URL via staticSansSha1Url + CG_PREFIX ?**
R: Oui, pas de changement côté front. Le service Java `DocumentUrlResolver` fait la traduction vers GCS en arrière-plan.

**Q: Fallback attendu si un document est manquant ?**
R: Retour HTTP 404 avec JSON d'erreur structuré. Pour PPS/PC : affichage message "Document temporairement indisponible" + notification automatique équipe.

**Q: Gestion des CG dynamiques ?**
R: Service Java `DynamicDocumentGenerator` avec templates + API de génération à la volée. Cache Redis 1h pour éviter regénération excessive.

## 4) Back-office d'upload (MVP)

**Q: Parcours d'upload ?**
R: Workflow en 2 temps : 1) Upload + métadonnées + preview → statut "draft", 2) Validation TAM → statut "published". Interface Spring Boot + React avec drag-and-drop.

**Q: Contrôles à l'upload ?**
R: PDF uniquement, max 50MB, scan antivirus ClamAV, validation structure PDF, métadonnées obligatoires, blacklist noms de fichiers.

**Q: Rôles & droits ?**
R: 3 rôles : UPLOADER (TAM/KAM), VALIDATOR (lead TAM), ADMIN (IT). Audit trail complet en base PostgreSQL.

**Q: Gestion du remplacement ?**
R: Remplacement par nouvelle version avec conservation historique. Rollback possible via interface admin sous 48h.

**Q: Bulk upload ?**
R: Oui, upload zip avec CSV de métadonnées. Auto-parsing des noms de fichiers selon convention. Preview batch avant validation.

**Q: Scope MVP ?**
R: Qualif + 2 partenaires pilotes en prod sous feature flag. Rollout complet après validation 1 mois.

## 5) CI/CD & jobs techniques

**Q: Confirmer la modif de upload-statics ?**
R: Oui, exclusion CG/IPID du binaire. Modification Jenkins job existant avec paramètre `--exclude-documents`.

**Q: Nouveau job "bucket-validator" ?**
R: Job Jenkins quotidien avec :
- Vérification présence fichiers référencés
- Détection orphelins (présents mais non référencés)  
- Contrôle format PDF et métadonnées
- Report Slack en cas d'anomalie

**Q: Déclenchement ?**
R: Label CI `DOCUMENTS_CHECK` + planification quotidienne 2h du matin. Notifications Slack canal #documents + email leads.

## 6) Tests : à conserver / à migrer

**Q: Confirmer les tests à garder ?**
R: Conservation :
- `verify_conditions_generales_external_format` 
- Tests construction URLs `getFullPath()`
- Tests IPID hébergement interne

**Q: Confirmer les tests à migrer ?**
R: Migration vers job Jenkins :
- Présence et validité PDFs
- Nettoyage fichiers obsolètes
- Extension .pdf obligatoire
- Cohérence métadonnées/fichiers

**Q: Fonctionnement local ?**
R: Oui, même URLs statics en local avec fallback sur environnement qualif via property `documents.fallback.enabled=true`.

## 7) Consommation Legacy vs Split

**Q: Split : confirmé qu'on ne teste pas CG/IPID côté Split ?**
R: Confirmé, tout reste côté Legacy pour l'instant. Ticket séparé "DOC-SPLIT-MIGRATION" à créer après stabilisation Legacy.

**Q: Points de code Legacy impactés ?**
R: 3 modules : PPS (service `DocumentUrlService`), Product Catalog (controller `CatalogController`), BO (écrans partenaires). MEP par module avec feature flags.

## 8) Sécurité, conformité & gouvernance

**Q: AuthN/Z BO ?**
R: SSO LDAP interne, rôles Spring Security, audit trail PostgreSQL avec rétention 7 ans. Logs applicatifs conservés 3 ans.

**Q: Confidentialité ?**
R: CG/IPID publics donc pas d'enjeu RGPD sur contenu. Métadonnées upload (utilisateur, date) = données personnelles → consentement explicite + droit effacement après 3 ans.

**Q: RPO/RTO ?**
R: GCS multi-régional (europe-west1 + europe-west4). RPO 1h, RTO 4h. Backup daily vers bucket séparé.

**Q: Traçabilité métier ?**
R: RACI défini : TAM=Responsible, Lead TAM=Accountable, Legal=Consulted, IT=Informed. Workflow d'approbation dans l'interface.

## 9) Observabilité & qualité de service

**Q: Monitoring ?**
R: Métriques (Prometheus + Grafana) : latence p95, taux 404/403, volumétrie, coût GCS mensuel. Dashboard dédié équipe documents.

**Q: Alerting ?**
R: Alertes (PagerDuty + Slack) : pic 404 >5%, latence >2s, coût mensuel +20%, virus détecté, TTL expiration <30j.

**Q: Link checker périodique ?**
R: Job Jenkins hebdomadaire crawling PPS/PC production. Validation tous liens CG/IPID + report anomalies.

## 10) Performance, cache & SEO

**Q: CDN/Cache ?**
R: CloudFlare devant GCS. Cache-Control 24h, purge automatique sur remplacement via webhook. `noindex` sur PDFs via `X-Robots-Tag: noindex`.

**Q: Hash vs nom stable ?**
R: Nom stable pour URLs publiques (SEO + bookmarks). Hash interne GCS pour déduplication + versionning.

## 11) Migration & nettoyage

**Q: Plan de retrait des fichiers CG/IPID du codebase ?**
R: 3 phases : 1) Double écriture (code + GCS), 2) Tests non-régression 1 mois, 3) Suppression code + cleanup /static. Tests PPS/PC automatisés avant chaque étape.

**Q: Backfill initial ?**
R: Script migration depuis `/static` + base produits existante. Contrôle croisé via checksum MD5. Validation manuelle échantillon 10% par équipe métier.

**Q: Communication interne ?**
R: Formation 2h TAM/KAM, runbook ops, FAQ, support Slack dédié 1 mois. Documentation Confluence + vidéos.

## 12) Modèle de données & intégration au Product Catalog

**Q: Lien document ↔ offre/formule dans le catalogue ?**
R: Clé composite : `codePartenaire + codeProduit + codeFormule + dateEffet`. Table PostgreSQL `document_catalog_mapping`.

**Q: Vue "catalogue" dans BO ?**
R: Oui, écran dédié par partenaire/produit avec statuts (actif/draft/expiré/obsolète). Filtrages par date effet et statut.

**Q: Export pour contrôles & audit ?**
R: Export CSV/Excel avec tous métadonnées + URLs. Job mensuel automatique + export à la demande. API REST pour intégrations tierces.

## 13) Spécificités métier

**Q: Effet différé : prise d'effet d'un nouveau PDF à une date/heure donnée ?**
R: Oui, champ `dateEffet` en base + scheduler Quartz pour activation automatique. Interface planning dans BO.

**Q: Multi-marques / affiliations ?**
R: Template unique + variables (logo, couleurs, mentions légales). Service `DocumentCustomizer` pour génération variantes. Stockage optimisé avec références partagées.

**Q: Langues ?**
R: FR uniquement pour MVP. Architecture prête multi-langues via suffixe fichier + champ `langue` en base pour évolutions futures.

---

**Budget estimé :** 180k€ (6 mois, équipe 4 personnes)
**Timeline :** Q1 2025 - Q2 2025
**Risques principaux :** Migration legacy, formation utilisateurs, performance GCS