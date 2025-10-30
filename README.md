# sfdx-hardis-procedure-installation-claude.md
procédure d'installation créée par Claude Sonnet 4.5

# 📋 SYNTHÈSE COMPLÈTE : Architecture sfdx-hardis INT/RCT/MAIN

Guide de référence complet pour l'implémentation de votre projet Salesforce DevOps.

---

## 🏢 ARCHITECTURE GLOBALE

```
┌─────────────────────────────────────────────────────────────────┐
│                    REPOSITORIES GITHUB                          │
├─────────────────────────────────────────────────────────────────┤
│  📦 REPO 1: SOURCE OF TRUTH                                    │
│  📊 REPO 2: MONITORING                                         │
└─────────────────────────────────────────────────────────────────┘
                            ⬇️
┌─────────────────────────────────────────────────────────────────┐
│                    SALESFORCE ORGS (4)                          │
├─────────────────────────────────────────────────────────────────┤
│  🔧 ORG TECHNICAL (Sandbox) - Monitoring & Flow Diff           │
│  🟦 ORG INT (Sandbox) - Intégration/Développement              │
│  🟨 ORG RCT (Sandbox) - Recette/UAT                            │
│  🟥 ORG MAIN (Production ou PreProd)                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 1️⃣ ORGS SALESFORCE

### Liste des 4 Orgs

| Org | Type | Usage | Alias | Branche Git |
|-----|------|-------|-------|-------------|
| **Technical Org** | Sandbox | CI/CD operations, Flow Diff, Documentation | `TechnicalOrg` | N/A |
| **INT** | Sandbox | Développement, Intégration | `INT` | `int` |
| **RCT** | Sandbox | Recette, UAT, Tests | `RCT` | `rct` |
| **MAIN** | Production | Production | `MAIN` | `main` |

### Rôle de chaque Org

#### 🔧 **Technical Org**
- **Source**: Copie synchronisée depuis MAIN (production)
- **Usage**: 
  - Génération documentation Flow (avec historique)
  - Opérations sfdx-hardis (monitoring, backup)
  - Référence "golden master"
  - Comparaisons de métadonnées
- **Ne contient PAS**: Données métier
- **Volume**: ~20,000 artefacts (copie de MAIN)

#### 🟦 **ORG INT**
- **Type**: Sandbox d'intégration
- **Usage**: Développement, intégration continue
- **Tests**: RunLocalTests
- **Coverage**: ≥75%
- **Timeout**: 60 minutes

#### 🟨 **ORG RCT**
- **Type**: Sandbox de recette
- **Usage**: Tests UAT, validation pre-prod
- **Tests**: RunLocalTests
- **Coverage**: ≥80%
- **Timeout**: 90 minutes

#### 🟥 **ORG MAIN**
- **Type**: Production
- **Usage**: Production finale
- **Tests**: RunLocalTests
- **Coverage**: ≥85%
- **Timeout**: 120 minutes
- **Source pour**: Technical Org, Retrofit

---

## 2️⃣ FICHIERS DE CONFIGURATION (.yml)

### 📁 Structure complète des fichiers

```
your-salesforce-project/
├── .github/
│   └── workflows/
│       ├── check-deploy.yml                 ← PR validation
│       ├── process-deploy.yml               ← Déploiement réel
│       ├── megalinter.yml                   ← Qualité code
│       ├── init-technical-org.yml           ← Init org technique
│       ├── sync-technical-org.yml           ← Sync org technique
│       ├── retrofit-production.yml          ← Retrofit MAIN
│       ├── retrofit-rct.yml                 ← Retrofit RCT
│       └── auto-clean-commit.yml            ← Auto-clean auto
│
├── config/
│   ├── .sfdx-hardis.yml                     ← CONFIG PRINCIPALE
│   └── branches/
│       ├── .sfdx-hardis-int.yml             ← Overrides INT
│       ├── .sfdx-hardis-rct.yml             ← Overrides RCT
│       └── .sfdx-hardis-main.yml            ← Overrides MAIN
│
├── manifest/
│   ├── package.xml
│   └── package-skip-items.xml               ← Exclusions backup
│
├── scripts/
│   └── apex/
│       └── activate-feature-flags.apex      ← Scripts Apex
│
├── .gitignore
├── .forceignore
├── sfdx-project.json
└── README.md
```

---

## 3️⃣ FICHIERS YAML À CRÉER

### A. REPO SOURCE OF TRUTH - GitHub Actions (7 fichiers)

#### 1. `.github/workflows/check-deploy.yml`
```yaml
Déclencheur: Pull Request vers int/rct/main
Fonction: Simule le déploiement (validation)
Commande: sf hardis:project:deploy:smart --check
Secrets utilisés:
  - SFDX_CLIENT_ID_{BRANCH}
  - SFDX_CLIENT_KEY_{BRANCH}
  - SFDX_AUTH_URL_TECHNICAL_ORG
  - GITHUB_TOKEN
  - SLACK_TOKEN (optionnel)
```

#### 2. `.github/workflows/process-deploy.yml`
```yaml
Déclencheur: Push sur int/rct/main
Fonction: Déploiement réel
Commande: sf hardis:project:deploy:smart
Secrets utilisés: (identiques à check-deploy.yml)
```

#### 3. `.github/workflows/megalinter.yml`
```yaml
Déclencheur: Push + PR
Fonction: Qualité du code (PMD, ESLint, etc.)
Secrets utilisés:
  - PAT ou GITHUB_TOKEN
```

#### 4. `.github/workflows/init-technical-org.yml`
```yaml
Déclencheur: Manuel (workflow_dispatch)
Fonction: Initialisation Technical Org depuis MAIN/RCT/INT
Commandes:
  - sf hardis:org:monitor:backup (avec chunking)
  - sf hardis:project:deploy:smart
Secrets utilisés:
  - SFDX_CLIENT_ID_{SOURCE_ORG}
  - SFDX_CLIENT_KEY_{SOURCE_ORG}
  - SFDX_AUTH_URL_TECHNICAL_ORG
```

#### 5. `.github/workflows/sync-technical-org.yml`
```yaml
Déclencheur: 
  - Après déploiement MAIN
  - Cron hebdomadaire
  - Manuel
Fonction: Synchronisation Technical Org avec MAIN
Commandes:
  - sf hardis:org:monitor:backup
  - sf hardis:project:deploy:smart
```

#### 6. `.github/workflows/retrofit-production.yml`
```yaml
Déclencheur: 
  - Cron quotidien (6h)
  - Manuel
Fonction: Détecte changements MAIN → Crée PR vers INT
Commande: sf hardis:org:retrieve:sources:retrofit
Secrets utilisés:
  - SFDX_CLIENT_ID_MAIN
  - SFDX_CLIENT_KEY_MAIN
  - PAT (pour créer PR)
```

#### 7. `.github/workflows/auto-clean-commit.yml`
```yaml
Déclencheur: 
  - Push sur feature branches
  - Manuel
Fonction: Auto-clean metadata + commit
Commande: sf hardis:work:save --auto-clean
```

### B. REPO SOURCE OF TRUTH - Configuration (4 fichiers)

#### 1. `config/.sfdx-hardis.yml` (PRINCIPAL - ~400 lignes)
```yaml
Sections:
  - Project identification
  - Org configuration (devHub, technical, branches)
  - CI/CD settings
  - Managed packages
  - Metadata filtering & cleanup
  - Permissions & initialization
  - Pre/Post deployment commands
  - Code quality & testing
  - Documentation generation
  - Notifications
  - JIRA integration
  - Scratch org configuration
  - Monitoring configuration
  - Custom Metadata handling
  - Auto-clean configuration (9 types)
  - Retrofit configuration
  - Advanced settings
```

#### 2. `config/branches/.sfdx-hardis-int.yml`
```yaml
Overrides pour INT:
  - deploymentWaitMinutes: 60
  - testWaitMinutes: 60
  - Commandes post-deploy spécifiques INT
```

#### 3. `config/branches/.sfdx-hardis-rct.yml`
```yaml
Overrides pour RCT:
  - deploymentWaitMinutes: 90
  - minCoverageRequired: 80
```

#### 4. `config/branches/.sfdx-hardis-main.yml`
```yaml
Overrides pour MAIN:
  - deploymentWaitMinutes: 120
  - minCoverageRequired: 85
  - Commandes critiques production
```

### C. REPO SOURCE OF TRUTH - Manifest

#### `manifest/package-skip-items.xml`
```xml
Exclusions pour backup monitoring:
  - Translation
  - Report
  - Dashboard
  - Document
  - Namespaced items (et4ae5__*, pi__*, dlrs__*)
```

### D. REPO MONITORING - GitHub Actions (4 fichiers)

#### 1. `.github/workflows/collect-deployment-metrics.yml`
```yaml
Déclencheur: Cron (toutes les 15 min)
Fonction: Collecte métriques de tous les environnements
```

#### 2. `.github/workflows/generate-reports.yml`
```yaml
Déclencheur: Cron (hebdomadaire)
Fonction: Génération rapports et dashboards
```

#### 3. `.github/workflows/health-check.yml`
```yaml
Déclencheur: Cron (toutes les 4h)
Fonction: Health check des 3 orgs (INT/RCT/MAIN)
```

#### 4. `.github/workflows/analyze-code-quality.yml`
```yaml
Déclencheur: Cron (quotidien)
Fonction: Analyse qualité code (PMD, ESLint, coverage)
```

---

## 4️⃣ SECRETS GITHUB

### REPO SOURCE OF TRUTH - Secrets (Settings > Secrets and variables > Actions)

#### A. Credentials Salesforce (Connected Apps)

```bash
# INT Environment
SFDX_CLIENT_ID_INT          # Connected App Client ID pour org INT
SFDX_CLIENT_KEY_INT         # Private Key (certificat) pour org INT

# RCT Environment
SFDX_CLIENT_ID_RCT          # Connected App Client ID pour org RCT
SFDX_CLIENT_KEY_RCT         # Private Key (certificat) pour org RCT

# MAIN (Production) Environment
SFDX_CLIENT_ID_MAIN         # Connected App Client ID pour org MAIN
SFDX_CLIENT_KEY_MAIN        # Private Key (certificat) pour org MAIN

# Technical Org
SFDX_AUTH_URL_TECHNICAL_ORG # Auth URL complète de l'org technique

# DevHub (si scratch orgs)
SFDX_AUTH_URL_DEV_HUB       # Auth URL du DevHub (optionnel)
```

#### B. GitHub

```bash
PAT                          # Personal Access Token (repo access)
GITHUB_TOKEN                 # Fourni automatiquement par GitHub Actions
```

#### C. Notifications

```bash
# Slack
SLACK_TOKEN                  # Bot User OAuth Token
SLACK_CHANNEL_ID             # Channel par défaut
SLACK_CHANNEL_ID_INT         # Channel spécifique INT
SLACK_CHANNEL_ID_RCT         # Channel spécifique RCT
SLACK_CHANNEL_ID_MAIN        # Channel spécifique MAIN

# Email
NOTIF_EMAIL_ADDRESS          # Email par défaut
NOTIF_EMAIL_ADDRESS_INT      # Email spécifique INT
NOTIF_EMAIL_ADDRESS_RCT      # Email spécifique RCT
NOTIF_EMAIL_ADDRESS_MAIN     # Email spécifique MAIN

# MS Teams (optionnel)
MS_TEAMS_WEBHOOK_URL         # Webhook MS Teams
```

#### D. JIRA Integration

```bash
JIRA_HOST                    # https://mycompany.atlassian.net
JIRA_EMAIL                   # admin@mycompany.com
JIRA_TOKEN                   # API Token JIRA
JIRA_PAT                     # Personal Access Token (alternative)
JIRA_TICKET_REGEX            # Regex pour détecter tickets (optionnel)
```

#### E. AI Integration (optionnel)

```bash
OPENAI_API_KEY               # Pour analyse erreurs déploiement
```

### REPO SOURCE OF TRUTH - Variables (Settings > Secrets and variables > Actions > Variables)

```bash
SFDX_DEPLOY_WAIT_MINUTES=120
SFDX_TEST_WAIT_MINUTES=120
INSTALL_PACKAGES_TECHNICAL_ORG=true
CI_SOURCES_TO_RETROFIT=CustomObject,CustomField,ValidationRule,PermissionSet,Flow
```

### REPO MONITORING - Secrets

```bash
# Accès aux orgs (lecture seule)
SFDX_AUTH_URL_INT
SFDX_AUTH_URL_RCT
SFDX_AUTH_URL_MAIN
SFDX_AUTH_URL_TECHNICAL_ORG

# Accès au repo source
MONITORING_PAT               # PAT pour lire repo source

# Notifications
SLACK_WEBHOOK_URL            # Pour alertes monitoring
```

### REPO MONITORING - Variables

```bash
SOURCE_REPO_OWNER=mycompany
SOURCE_REPO_NAME=salesforce-source
```

---

## 5️⃣ VARIABLES D'ENVIRONNEMENT PRINCIPALES

### Variables utilisées dans les workflows

| Variable | Usage | Valeur type | Où définie |
|----------|-------|-------------|------------|
| `CI_COMMIT_REF_NAME` | Nom de branche cible | `int`, `rct`, `main` | GitHub Actions (auto) |
| `ORG_ALIAS` | Alias de l'org cible | `INT`, `RCT`, `MAIN` | GitHub Actions (dérivée) |
| `CONFIG_BRANCH` | Branche de config | `int`, `rct`, `main` | GitHub Actions (dérivée) |
| `SFDX_DEPLOY_WAIT_MINUTES` | Timeout déploiement | `120` | Variables GitHub |
| `SFDX_TEST_WAIT_MINUTES` | Timeout tests | `120` | Variables GitHub |
| `SFDX_DISABLE_FLOW_DIFF` | Désactiver Flow Diff | `false` | .sfdx-hardis.yml |
| `FORCE_COLOR` | Couleurs dans logs | `1` | GitHub Actions |
| `MONITORING_BACKUP_SKIP_METADATA_TYPES` | Types à ignorer | `Translation,Report` | .sfdx-hardis.yml |

### Variables sfdx-hardis internes (auto-détectées)

```bash
# Résolution automatique par sfdx-hardis
SFDX_CLIENT_ID_{BRANCH}      # Ex: SFDX_CLIENT_ID_INT
SFDX_CLIENT_KEY_{BRANCH}     # Ex: SFDX_CLIENT_KEY_RCT
SLACK_CHANNEL_ID_{BRANCH}    # Ex: SLACK_CHANNEL_ID_MAIN
```

---

## 6️⃣ PROCÉDURES PRINCIPALES

### A. INITIALISATION INITIALE (One-time setup)

#### 1. Créer les Connected Apps Salesforce (4 orgs)

```bash
Pour chaque org (INT, RCT, MAIN, Technical):

1. Setup → App Manager → New Connected App
2. Nom: "GitHub Actions CI/CD - {ENV}"
3. Enable OAuth Settings
4. Callback URL: http://localhost:1717/OauthRedirect
5. Selected OAuth Scopes:
   - Full access (full)
   - Perform requests at any time (refresh_token, offline_access)
6. Enable "Use digital signatures"
7. Upload certificate (générer avec: openssl req -x509 -newkey rsa:4096)
8. Sauvegarder → Récupérer Consumer Key
9. Stocker dans GitHub Secrets:
   - Consumer Key → SFDX_CLIENT_ID_{ENV}
   - Private Key → SFDX_CLIENT_KEY_{ENV}
```

#### 2. Générer SFDX Auth URL pour Technical Org

```bash
# Se connecter à Technical Org
sf org login web --alias TechnicalOrg

# Générer Auth URL
sf org display --target-org TechnicalOrg --verbose --json | jq -r '.result.sfdxAuthUrl'

# Stocker dans GitHub Secret
# → SFDX_AUTH_URL_TECHNICAL_ORG
```

#### 3. Créer les repositories GitHub

```bash
# Repo 1: Source of Truth
- Créer repo: mycompany/salesforce-source
- Protéger branches: int, rct, main
- Ajouter tous les secrets

# Repo 2: Monitoring
- Créer repo: mycompany/salesforce-monitoring
- Ajouter secrets monitoring
```

#### 4. Initialiser Technical Org

```bash
# Via GitHub Actions
gh workflow run init-technical-org.yml \
  -f source_org=main \
  -f backup_mode=filtered \
  -f chunk_size=1000

# Durée: 30-60 minutes (20k artifacts)
```

### B. WORKFLOW QUOTIDIEN

#### Flow de développement complet

```
1. Développeur crée feature branch depuis int
   ↓
2. Développeur fait changements + commit + push
   ↓
3. Crée Pull Request vers int
   ↓
4. check-deploy.yml s'exécute (validation)
   ↓
5. Review + Approbation
   ↓
6. Merge dans int
   ↓
7. process-deploy.yml s'exécute → Déploie vers ORG INT
   ↓
8. Tests OK → PR de int vers rct
   ↓
9. check-deploy.yml valide vers RCT
   ↓
10. Merge dans rct → Déploie vers ORG RCT
    ↓
11. UAT OK → PR de rct vers main
    ↓
12. check-deploy.yml valide vers MAIN (strict)
    ↓
13. Approbation manager + Merge
    ↓
14. Déploiement PRODUCTION
    ↓
15. sync-technical-org.yml se déclenche automatiquement
```

### C. OPÉRATIONS PLANIFIÉES

#### Quotidiennes

```yaml
06h00: retrofit-production.yml
       → Détecte changements MAIN non dans Git
       → Crée PR si changements

06h30: collect-deployment-metrics.yml (monitoring)
       → Collecte métriques toutes les 15min en réalité
```

#### Hebdomadaires

```yaml
Dimanche 02h00: sync-technical-org.yml
                → Synchronisation complète Technical Org

Lundi 08h00: generate-reports.yml (monitoring)
             → Génération rapports hebdomadaires

Vendredi 06h00: retrofit-rct.yml
                → Retrofit des changements RCT
```

#### Toutes les 4 heures

```yaml
health-check.yml (monitoring)
→ Vérifie santé des 3 orgs
```

### D. COMMANDES MANUELLES UTILES

#### Déploiement

```bash
# Check deploy local
sf hardis:project:deploy:smart --check --target-org INT

# Deploy réel local
sf hardis:project:deploy:smart --target-org RCT

# Deploy rapide (quick deploy après validation)
sf hardis:deploy:quick --job-id 0Af... --target-org MAIN
```

#### Monitoring & Backup

```bash
# Backup org avec chunking (20k+ artifacts)
sf hardis:org:monitor:backup \
  --target-org MAIN \
  --full \
  --exclude-namespaces \
  --full-apply-filters \
  --max-by-chunk 1000

# Backup mode filtered (recommandé)
sf hardis:org:monitor:backup --target-org MAIN
```

#### Retrofit

```bash
# Retrofit depuis MAIN vers INT
sf hardis:org:retrieve:sources:retrofit \
  --productionbranch main \
  --retrofitbranch int \
  --commit \
  --commitmode updated \
  --push \
  --pushmode mergerequest \
  --target-org MAIN
```

#### Auto-clean

```bash
# Auto-clean complet
sf hardis:work:save --auto-clean

# Auto-clean types spécifiques
sf hardis:work:save --auto-clean-types "minimizeProfiles,listViewsMine,systemDebug"

# Juste le nettoyage sans commit
sf hardis:project:clean
```

#### Custom Metadata

```bash
# Récupérer tous les Custom Metadata
sf hardis:org:retrieve:sources:metadata \
  --metadata-types CustomMetadata \
  --target-org MAIN

# Déployer Custom Metadata spécifiques
sf project deploy start \
  --metadata "CustomMetadata:MyApp__*INT*" \
  --target-org INT
```

---

## 7️⃣ CHECKLIST DE DÉPLOIEMENT

### Phase 1: Setup initial (1 jour)

```markdown
□ Créer Connected Apps (INT, RCT, MAIN, Technical)
□ Générer certificats et private keys
□ Configurer GitHub Secrets (tous)
□ Créer repositories GitHub (source + monitoring)
□ Protéger branches (int, rct, main)
□ Créer fichiers .sfdx-hardis.yml (4 fichiers)
□ Créer GitHub Actions workflows (11 fichiers)
□ Créer manifest/package-skip-items.xml
```

### Phase 2: Initialisation Technical Org (2h)

```markdown
□ Exécuter init-technical-org.yml avec source=main
□ Vérifier déploiement Technical Org
□ Tester génération documentation Flow
□ Valider backup monitoring (chunking)
```

### Phase 3: Tests CI/CD (1 jour)

```markdown
□ Créer feature branch test
□ Faire changement simple (ex: CustomLabel)
□ Push + créer PR vers int
□ Vérifier check-deploy.yml réussit
□ Merger → vérifier process-deploy.yml déploie
□ Tester PR int → rct
□ Tester PR rct → main
□ Vérifier sync-technical-org après deploy main
```

### Phase 4: Tests Retrofit (1h)

```markdown
□ Faire changement manuel en MAIN (ex: créer champ)
□ Attendre cron ou déclencher retrofit-production.yml
□ Vérifier PR créée automatiquement
□ Merger PR retrofit
```

### Phase 5: Tests Auto-clean (1h)

```markdown
□ Récupérer metadata avec Profiles non minimizés
□ Exécuter auto-clean
□ Vérifier Profiles nettoyés
□ Vérifier System.debug commentés
□ Vérifier ListView Mine converties
```

### Phase 6: Configuration Monitoring (2h)

```markdown
□ Créer workflows monitoring (4 fichiers)
□ Tester collect-deployment-metrics.yml
□ Vérifier génération rapports
□ Configurer dashboards
□ Tester health-check.yml
```

### Phase 7: Documentation & Formation (1 jour)

```markdown
□ Documenter architecture
□ Créer runbook pour équipe
□ Former développeurs sur workflow
□ Documenter procédures d'urgence
□ Tester rollback
```

---

## 8️⃣ RÉSUMÉ DES COMMANDES PRINCIPALES

### Commandes sfdx-hardis essentielles

```bash
# Authentification
sf hardis:auth:login

# Déploiement
sf hardis:project:deploy:smart --check        # Validation
sf hardis:project:deploy:smart                # Déploiement réel

# Monitoring
sf hardis:org:monitor:backup                  # Backup filtered
sf hardis:org:monitor:backup --full           # Backup complet

# Retrofit
sf hardis:org:retrieve:sources:retrofit       # Récup changements prod

# Auto-clean
sf hardis:work:save --auto-clean              # Nettoyage + commit

# Documentation
sf hardis:doc:project:generate                # Génération doc

# Configuration
sf hardis:project:configure                   # Update config files
```

---

## 9️⃣ MÉTRIQUES & KPIs

### À surveiller (dashboards monitoring)

```yaml
Déploiements:
  - Success Rate: >90%
  - Average Duration: <60min
  - Failed Deployments: <3/semaine
  - Deployment Frequency: tracking

Code Quality:
  - Test Coverage: >75% (INT), >80% (RCT), >85% (MAIN)
  - PMD Score: >3/5
  - ESLint Errors: 0

Retrofit:
  - Production Changes: tracking
  - Retrofit PRs: tracking
  - Time to Merge Retrofit: <48h

Technical Org:
  - Sync Frequency: quotidien
  - Sync Success Rate: >95%
  - Metadata Count: ~20,000
```

---

## 🎯 PROCHAINES ÉTAPES

### Actions immédiates

1. **Créer Connected Apps** (2h)
2. **Configurer GitHub Secrets** (30min)
3. **Créer config/.sfdx-hardis.yml** (1h)
4. **Créer workflows GitHub Actions** (2h)
5. **Initialiser Technical Org** (1h)
6. **Tests CI/CD complets** (4h)

### Semaine 1

- Setup complet infrastructure
- Tests validation complète
- Documentation

### Semaine 2

- Formation équipe
- Migration premiers projets
- Monitoring actif

---

## 📚 RESSOURCES

### Documentation sfdx-hardis

- https://sfdx-hardis.cloudity.com/
- https://github.com/hardisgroupcom/sfdx-hardis

### Scripts utiles

```bash
# Script validation complète
./scripts/validate-setup.sh

# Script génération secrets
./scripts/generate-secrets.sh

# Script test CI/CD
./scripts/test-cicd.sh
```

---

## 🎉 RÉCAPITULATIF FINAL

```
✅ 4 Orgs Salesforce (Technical + INT + RCT + MAIN)
✅ 2 Repositories GitHub (Source + Monitoring)
✅ 15 fichiers YAML (11 workflows + 4 configs)
✅ 25+ Secrets GitHub
✅ 10+ Variables GitHub
✅ 9 types d'auto-clean
✅ 20+ commandes sfdx-hardis
✅ Architecture production-ready
```

**Temps estimé setup complet**: 3-5 jours

**Résultat**: Pipeline CI/CD professionnel, scalable, avec monitoring complet ! 🚀

---

Voulez-vous que je génère des **scripts d'installation automatisée** ou un **guide détaillé étape par étape** pour une section spécifique ?
