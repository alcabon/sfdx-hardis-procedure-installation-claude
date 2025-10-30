# Scripts restants (03, 04, 05) - Suite complète

---

## 📄 `scripts/setup/03-create-config-files.sh`

```bash
#!/bin/bash

# ============================================================================
# CONFIGURATION FILES GENERATION
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m'
BOLD='\033[1m'

PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/../.." && pwd)"

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

print_success() {
    echo -e "${GREEN}✓${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}⚠${NC} $1"
}

ask_value() {
    local prompt="$1"
    local default="$2"
    local value
    
    if [ -n "$default" ]; then
        read -p "$(echo -e ${CYAN}$prompt [${default}]:${NC} )" value
        value=${value:-$default}
    else
        read -p "$(echo -e ${CYAN}$prompt:${NC} )" value
    fi
    
    echo "$value"
}

# ============================================================================
# COLLECT PROJECT INFORMATION
# ============================================================================

print_header "📝 PROJECT CONFIGURATION"

echo "Let's gather some information about your project:"
echo ""

PROJECT_NAME=$(ask_value "Project name" "MyProject")
DEV_HUB_ALIAS=$(ask_value "DevHub alias" "DevHub_MyClient")
DEV_HUB_USERNAME=$(ask_value "DevHub username" "devhub.admin@myclient.com")
COMPANY_DOMAIN=$(ask_value "Company domain" "myclient.com")

echo ""
echo "Which managed packages do you use? (leave empty if none)"
USE_MARKETING_CLOUD=$(ask_value "Use Marketing Cloud? (y/n)" "n")
USE_DLRS=$(ask_value "Use Declarative Lookup Rollup Summaries? (y/n)" "y")

echo ""
echo "Configuration preferences:"
MIN_COVERAGE_INT=$(ask_value "Minimum test coverage for INT (%)" "75")
MIN_COVERAGE_RCT=$(ask_value "Minimum test coverage for RCT (%)" "80")
MIN_COVERAGE_MAIN=$(ask_value "Minimum test coverage for MAIN (%)" "85")

# ============================================================================
# CREATE DIRECTORY STRUCTURE
# ============================================================================

print_header "📁 Creating Directory Structure"

mkdir -p "$PROJECT_ROOT/.github/workflows"
mkdir -p "$PROJECT_ROOT/config/branches"
mkdir -p "$PROJECT_ROOT/manifest"
mkdir -p "$PROJECT_ROOT/scripts/apex"
mkdir -p "$PROJECT_ROOT/force-app/main/default"

print_success "Directories created"

# ============================================================================
# GENERATE .sfdx-hardis.yml (MAIN CONFIG)
# ============================================================================

print_header "📄 Generating config/.sfdx-hardis.yml"

cat > "$PROJECT_ROOT/config/.sfdx-hardis.yml" << 'EOFMAIN'
# ============================================================================
# SFDX-HARDIS Configuration File
# ============================================================================
# Project: PROJECT_NAME_PLACEHOLDER
# Architecture: 3 major branches (int, rct, main) + Technical Org
# Generated: GENERATION_DATE_PLACEHOLDER
# ============================================================================

# ============================================================================
# PROJECT IDENTIFICATION
# ============================================================================
projectName: PROJECT_NAME_PLACEHOLDER
description: Salesforce DevOps project with sfdx-hardis

# ============================================================================
# ORG CONFIGURATION
# ============================================================================

# DevHub configuration
devHubAlias: DEVHUB_ALIAS_PLACEHOLDER
devHubUsername: DEVHUB_USERNAME_PLACEHOLDER

# Technical Org (used for monitoring, Flow diff, documentation)
technicalOrgAlias: TechnicalOrg

# Development branch (primary integration branch)
developmentBranch: int

# Available target branches for deployments
availableTargetBranches:
  - int       # Integration/Development environment
  - rct       # Recette/UAT environment  
  - main      # Production environment

# Allowed org types for deployments
allowedOrgTypes:
  - sandbox
  - production

# ============================================================================
# CI/CD & DEPLOYMENT SETTINGS
# ============================================================================

# Install packages during check deployments (validation)
installPackagesDuringCheckDeploy: true

# Auto-retrieve metadata when pulling from version control
autoRetrieveWhenPull:
  - CustomApplication
  - CustomMetadata
  - PermissionSet

# ============================================================================
# MANAGED PACKAGES
# ============================================================================
MANAGED_PACKAGES_PLACEHOLDER

# ============================================================================
# METADATA FILTERING & CLEANUP
# ============================================================================

# Auto-clean types during retrieve/backup operations
autoCleanTypes:
  - destructivechanges    # Remove destructive changes files
  - datadotcom            # Remove data.com related metadata
  - minimizeProfiles      # Minimize profile metadata
  - listViewsMine         # Convert "Mine" list views to user-specific
  - dashboards            # Clean dashboards (remove hardcoded user IDs)
  - systemDebug           # Comment System.debug statements
  - flexipages            # Clean FlexiPages
  - standardObjects       # Clean standard objects

# Automatically remove these user permissions from profiles
autoRemoveUserPermissions:
  - EnableCommunityAppLauncher
  - FieldServiceAccess
  - OmnichannelInventorySync
  - SendExternalEmailAvailable
  - UseOmnichannelInventoryAPIs
  - ViewDataLeakageEvents
  - ViewMLModels
  - ViewPlatformEvents
  - WorkCalibrationUser

# List views to automatically set to "Mine" visibility
listViewsToSetToMine:
  - force-app/main/default/objects/Account/listViews/MyAccounts.listView-meta.xml
  - force-app/main/default/objects/Opportunity/listViews/MyOpportunities.listView-meta.xml
  - force-app/main/default/objects/Case/listViews/MyCases.listView-meta.xml

# Metadata types to skip during monitoring backup
monitoringBackupSkipMetadataTypes:
  - Translation
  - Report
  - Dashboard
  - Document

# ============================================================================
# PERMISSIONS & INITIALIZATION
# ============================================================================

# Permission sets to assign after scratch org creation or deployment
initPermissionSets:
  - AdminDefault
  - ApiUserPS

# ============================================================================
# PRE/POST DEPLOYMENT COMMANDS
# ============================================================================

# Commands to run BEFORE deployment
commandsPreDeploy: []

# Commands to run AFTER deployment
commandsPostDeploy:
  - id: refreshCache
    label: Refresh cache after deployment
    command: sf data query --query "SELECT COUNT() FROM Account" --json
    skipIfError: true
    context: process-deployment-only

# ============================================================================
# CODE QUALITY & TESTING
# ============================================================================

# Apex test configuration
apexTests:
  minCoverageRequired: MIN_COVERAGE_INT_PLACEHOLDER
  testLevel: RunLocalTests

# ============================================================================
# DOCUMENTATION GENERATION
# ============================================================================

# Flow documentation settings
flowDocumentation:
  enabled: true
  includeHistory: true
  languages:
    - en

# Deploy documentation to Salesforce org as static resource
deployDocToOrg: false

# Deploy documentation to Cloudflare Pages
deployDocToCloudflare: false

# ============================================================================
# NOTIFICATIONS
# ============================================================================

notificationSettings:
  enabled: true
  notifyOnSuccess: true
  notifyOnFailure: true
  includeMetrics: true

# ============================================================================
# JIRA INTEGRATION
# ============================================================================

jiraIntegration:
  enabled: false

# ============================================================================
# MONITORING CONFIGURATION
# ============================================================================

monitoring:
  enabled: true
  backupMode: filtered
  maxBackupChunkSize: 1000
  excludeNamespaces: true
  applyFiltersInFullMode: true

# ============================================================================
# RETROFIT CONFIGURATION
# ============================================================================

# Retrofit: recover changes made directly in production
productionBranch: main
retrofitBranch: int

# Types of metadata to retrieve during retrofit
sourcesToRetrofit:
  - CustomObject
  - CustomField
  - ValidationRule
  - Flow
  - PermissionSet
  - CustomMetadata
  - Layout
  - FlexiPage
  - RecordType

# Files to ignore during retrofit
retrofitIgnoredFiles:
  - force-app/main/default/profiles/*.profile-meta.xml

# ============================================================================
# CUSTOM METADATA HANDLING
# ============================================================================

customMetadataDeployment:
  deployAll: true

# ============================================================================
# ADVANCED SETTINGS
# ============================================================================

# Performance optimization
performance:
  disableFlowDiff: false
  ignoreSplitPackages: true

# Debug settings
debug:
  enabled: false
  deploymentDebug: false
EOFMAIN

# Replace placeholders
sed -i.bak "s/PROJECT_NAME_PLACEHOLDER/$PROJECT_NAME/g" "$PROJECT_ROOT/config/.sfdx-hardis.yml"
sed -i.bak "s/DEVHUB_ALIAS_PLACEHOLDER/$DEV_HUB_ALIAS/g" "$PROJECT_ROOT/config/.sfdx-hardis.yml"
sed -i.bak "s/DEVHUB_USERNAME_PLACEHOLDER/$DEV_HUB_USERNAME/g" "$PROJECT_ROOT/config/.sfdx-hardis.yml"
sed -i.bak "s/MIN_COVERAGE_INT_PLACEHOLDER/$MIN_COVERAGE_INT/g" "$PROJECT_ROOT/config/.sfdx-hardis.yml"
sed -i.bak "s/GENERATION_DATE_PLACEHOLDER/$(date)/g" "$PROJECT_ROOT/config/.sfdx-hardis.yml"

# Add managed packages if needed
PACKAGES_SECTION=""
if [ "$USE_MARKETING_CLOUD" = "y" ]; then
    PACKAGES_SECTION+="installedPackages:\n"
    PACKAGES_SECTION+="  - Id: 0A37Z000000AtDYSA0\n"
    PACKAGES_SECTION+="    SubscriberPackageId: 033i0000000LVMYAA4\n"
    PACKAGES_SECTION+="    SubscriberPackageName: Marketing Cloud\n"
    PACKAGES_SECTION+="    SubscriberPackageNamespace: et4ae5\n"
    PACKAGES_SECTION+="    installOnScratchOrgs: true\n"
    PACKAGES_SECTION+="    installDuringDeployments: true\n"
fi

if [ "$USE_DLRS" = "y" ]; then
    [ -z "$PACKAGES_SECTION" ] && PACKAGES_SECTION="installedPackages:\n"
    PACKAGES_SECTION+="  - Id: 0A35r0000009F9CCAU\n"
    PACKAGES_SECTION+="    SubscriberPackageId: 033b0000000Pf2AAAS\n"
    PACKAGES_SECTION+="    SubscriberPackageName: Declarative Lookup Rollup Summaries Tool\n"
    PACKAGES_SECTION+="    SubscriberPackageNamespace: dlrs\n"
    PACKAGES_SECTION+="    installOnScratchOrgs: true\n"
    PACKAGES_SECTION+="    installDuringDeployments: true\n"
fi

[ -z "$PACKAGES_SECTION" ] && PACKAGES_SECTION="installedPackages: []"

sed -i.bak "s/MANAGED_PACKAGES_PLACEHOLDER/$PACKAGES_SECTION/g" "$PROJECT_ROOT/config/.sfdx-hardis.yml"

rm -f "$PROJECT_ROOT/config/.sfdx-hardis.yml.bak"

print_success "config/.sfdx-hardis.yml created"

# ============================================================================
# GENERATE BRANCH-SPECIFIC CONFIGS
# ============================================================================

print_header "📄 Generating Branch-Specific Configs"

# INT config
cat > "$PROJECT_ROOT/config/branches/.sfdx-hardis-int.yml" << EOF
# INT environment specific overrides
projectName: $PROJECT_NAME - INT

# INT-specific settings
apexTests:
  minCoverageRequired: $MIN_COVERAGE_INT

# Shorter timeouts for INT
deploymentWaitMinutes: 60
testWaitMinutes: 60
EOF

print_success "config/branches/.sfdx-hardis-int.yml created"

# RCT config
cat > "$PROJECT_ROOT/config/branches/.sfdx-hardis-rct.yml" << EOF
# RCT environment specific overrides
projectName: $PROJECT_NAME - RCT

# RCT-specific settings
apexTests:
  minCoverageRequired: $MIN_COVERAGE_RCT
  testLevel: RunLocalTests

deploymentWaitMinutes: 90
testWaitMinutes: 90
EOF

print_success "config/branches/.sfdx-hardis-rct.yml created"

# MAIN config
cat > "$PROJECT_ROOT/config/branches/.sfdx-hardis-main.yml" << EOF
# MAIN (Production) environment specific overrides
projectName: $PROJECT_NAME - PRODUCTION

# Production-specific settings
apexTests:
  minCoverageRequired: $MIN_COVERAGE_MAIN
  testLevel: RunLocalTests

deploymentWaitMinutes: 120
testWaitMinutes: 120

# Production-specific post-deployment
commandsPostDeploy:
  - id: warmCache
    label: Warm up production cache
    command: sf data query --query "SELECT COUNT() FROM Account" --json
    skipIfError: true
    context: process-deployment-only
EOF

print_success "config/branches/.sfdx-hardis-main.yml created"

# ============================================================================
# GENERATE GITHUB WORKFLOWS
# ============================================================================

print_header "📄 Generating GitHub Actions Workflows"

# check-deploy.yml
cat > "$PROJECT_ROOT/.github/workflows/check-deploy.yml" << 'EOFCHECK'
---
name: Check Deployment (sfdx-hardis)

on:
  pull_request:
    branches:
      - int
      - rct
      - main

permissions: read-all

concurrency:
  group: ${{ github.ref }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  check_deployment:
    runs-on: ubuntu-latest
    name: Simulate Deployment to Major Org
    permissions:
      pull-requests: write
      contents: write
      issues: write
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          persist-credentials: true
          token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}
          fetch-depth: 0
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      
      - name: Install SFDX and plugins
        run: |
          npm install --no-cache @salesforce/cli --global
          sf plugins install @salesforce/plugin-packaging
          echo 'y' | sf plugins install sfdx-hardis
          echo 'y' | sf plugins install sfdx-git-delta
          sf version --verbose --json
      
      - name: Login & Simulate deployment
        env:
          SFDX_CLIENT_ID_INT: ${{ secrets.SFDX_CLIENT_ID_INT}}
          SFDX_CLIENT_KEY_INT: ${{ secrets.SFDX_CLIENT_KEY_INT}}
          SFDX_CLIENT_ID_RCT: ${{ secrets.SFDX_CLIENT_ID_RCT}}
          SFDX_CLIENT_KEY_RCT: ${{ secrets.SFDX_CLIENT_KEY_RCT}}
          SFDX_CLIENT_ID_MAIN: ${{ secrets.SFDX_CLIENT_ID_MAIN}}
          SFDX_CLIENT_KEY_MAIN: ${{ secrets.SFDX_CLIENT_KEY_MAIN}}
          SFDX_AUTH_URL_TECHNICAL_ORG: ${{ secrets.SFDX_AUTH_URL_TECHNICAL_ORG }}
          SFDX_DEPLOY_WAIT_MINUTES: ${{ vars.SFDX_DEPLOY_WAIT_MINUTES || '120' }}
          SFDX_TEST_WAIT_MINUTES: ${{ vars.SFDX_TEST_WAIT_MINUTES || '120' }}
          CI_COMMIT_REF_NAME: ${{ github.event.pull_request.base.ref }}
          ORG_ALIAS: ${{ github.event.pull_request.base.ref }}
          CONFIG_BRANCH: ${{ github.event.pull_request.base.ref }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_TOKEN: ${{ secrets.SLACK_TOKEN }}
          SLACK_CHANNEL_ID: ${{ secrets.SLACK_CHANNEL_ID }}
          SLACK_CHANNEL_ID_INT: ${{ secrets.SLACK_CHANNEL_ID_INT }}
          SLACK_CHANNEL_ID_RCT: ${{ secrets.SLACK_CHANNEL_ID_RCT }}
          SLACK_CHANNEL_ID_MAIN: ${{ secrets.SLACK_CHANNEL_ID_MAIN }}
          FORCE_COLOR: "1"
          SFDX_DISABLE_FLOW_DIFF: false
        run: |
          echo "Simulate SFDX deployment using Hardis against \"$CONFIG_BRANCH\""
          sf hardis:auth:login
          sf hardis:project:deploy:smart --check
EOFCHECK

print_success ".github/workflows/check-deploy.yml created"

# process-deploy.yml
cat > "$PROJECT_ROOT/.github/workflows/process-deploy.yml" << 'EOFPROCESS'
---
name: Process Deployment (sfdx-hardis)

on:
  push:
    branches:
      - int
      - rct
      - main

permissions: read-all

concurrency:
  group: ${{ github.ref }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  process_deployment:
    name: Process Deployment to Major Org
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          persist-credentials: true
          token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}
          fetch-depth: 0
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      
      - name: Install SFDX and plugins
        run: |
          npm install @salesforce/cli --global
          sf plugins install @salesforce/plugin-packaging
          echo 'y' | sf plugins install sfdx-hardis
          echo 'y' | sf plugins install sfdx-git-delta
          sf version --verbose --json
      
      - name: Set env.BRANCH
        run: echo "BRANCH=$(echo "$GITHUB_REF" | cut -d'/' -f 3)" >> "$GITHUB_ENV"
      
      - name: Login & Process Deployment
        env:
          SFDX_CLIENT_ID_INT: ${{ secrets.SFDX_CLIENT_ID_INT}}
          SFDX_CLIENT_KEY_INT: ${{ secrets.SFDX_CLIENT_KEY_INT}}
          SFDX_CLIENT_ID_RCT: ${{ secrets.SFDX_CLIENT_ID_RCT}}
          SFDX_CLIENT_KEY_RCT: ${{ secrets.SFDX_CLIENT_KEY_RCT}}
          SFDX_CLIENT_ID_MAIN: ${{ secrets.SFDX_CLIENT_ID_MAIN}}
          SFDX_CLIENT_KEY_MAIN: ${{ secrets.SFDX_CLIENT_KEY_MAIN}}
          SFDX_AUTH_URL_TECHNICAL_ORG: ${{ secrets.SFDX_AUTH_URL_TECHNICAL_ORG }}
          SFDX_DEPLOY_WAIT_MINUTES: ${{ vars.SFDX_DEPLOY_WAIT_MINUTES || '120' }}
          SFDX_TEST_WAIT_MINUTES: ${{ vars.SFDX_TEST_WAIT_MINUTES || '120' }}
          CI_COMMIT_REF_NAME: ${{ env.BRANCH }}
          ORG_ALIAS: ${{ env.BRANCH }}
          CONFIG_BRANCH: ${{ env.BRANCH }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SLACK_TOKEN: ${{ secrets.SLACK_TOKEN }}
          SLACK_CHANNEL_ID: ${{ secrets.SLACK_CHANNEL_ID }}
          SLACK_CHANNEL_ID_INT: ${{ secrets.SLACK_CHANNEL_ID_INT }}
          SLACK_CHANNEL_ID_RCT: ${{ secrets.SLACK_CHANNEL_ID_RCT }}
          SLACK_CHANNEL_ID_MAIN: ${{ secrets.SLACK_CHANNEL_ID_MAIN }}
          FORCE_COLOR: "1"
        run: |
          echo "Process SFDX deployment using Hardis against \"$CONFIG_BRANCH\""
          sf hardis:auth:login
          sf hardis:project:deploy:smart
EOFPROCESS

print_success ".github/workflows/process-deploy.yml created"

# megalinter.yml
cat > "$PROJECT_ROOT/.github/workflows/megalinter.yml" << 'EOFLINTER'
---
name: Mega-Linter

on:
  push:
  pull_request:
    branches: [int, rct, main]

env:
  APPLY_FIXES: all
  APPLY_FIXES_EVENT: pull_request
  APPLY_FIXES_MODE: commit

permissions: read-all

concurrency:
  group: ${{ github.ref }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  build:
    name: Mega-Linter
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
    
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}
          fetch-depth: 0

      - name: Mega-Linter
        id: ml
        uses: oxsecurity/megalinter/flavors/salesforce@latest
        env:
          VALIDATE_ALL_CODEBASE: true
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Archive production artifacts
        if: success() || failure()
        uses: actions/upload-artifact@v4
        with:
          name: Mega-Linter reports
          include-hidden-files: "true"
          path: |
            megalinter-reports
            mega-linter.log
EOFLINTER

print_success ".github/workflows/megalinter.yml created"

# init-technical-org.yml
cat > "$PROJECT_ROOT/.github/workflows/init-technical-org.yml" << 'EOFINIT'
---
name: Initialize Technical Org

on:
  workflow_dispatch:
    inputs:
      source_org:
        description: 'Source org to retrieve metadata from'
        required: true
        default: 'main'
        type: choice
        options:
          - int
          - rct
          - main
      backup_mode:
        description: 'Backup mode'
        required: true
        default: 'filtered'
        type: choice
        options:
          - filtered
          - full
      chunk_size:
        description: 'Chunk size for full mode'
        required: false
        default: '1000'

permissions: read-all

jobs:
  initialize_technical_org:
    runs-on: ubuntu-latest
    name: Initialize Technical Org
    timeout-minutes: 120
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}
          fetch-depth: 0
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      
      - name: Install SFDX and plugins
        run: |
          npm install --no-cache @salesforce/cli --global
          sf plugins install @salesforce/plugin-packaging
          echo 'y' | sf plugins install sfdx-hardis
          echo 'y' | sf plugins install sfdx-git-delta
          sf version --verbose --json
      
      - name: Authenticate to Source Org
        env:
          SFDX_CLIENT_ID: ${{ secrets[format('SFDX_CLIENT_ID_{0}', upper(github.event.inputs.source_org))] }}
          SFDX_CLIENT_KEY: ${{ secrets[format('SFDX_CLIENT_KEY_{0}', upper(github.event.inputs.source_org))] }}
        run: |
          echo "$SFDX_CLIENT_KEY" > ./server.key
          sf org login jwt \
            --client-id "$SFDX_CLIENT_ID" \
            --jwt-key-file ./server.key \
            --username "auto-discover" \
            --alias SOURCE_ORG
          rm -f ./server.key
      
      - name: Authenticate to Technical Org
        env:
          SFDX_AUTH_URL_TECHNICAL_ORG: ${{ secrets.SFDX_AUTH_URL_TECHNICAL_ORG }}
        run: |
          echo "$SFDX_AUTH_URL_TECHNICAL_ORG" > ./authfile_tech.txt
          sf org login sfdx-url \
            --sfdx-url-file ./authfile_tech.txt \
            --alias TECH
          rm -f ./authfile_tech.txt
      
      - name: Backup metadata from Source Org
        env:
          MODE: ${{ github.event.inputs.backup_mode }}
          CHUNK_SIZE: ${{ github.event.inputs.chunk_size }}
          MONITORING_BACKUP_SKIP_METADATA_TYPES: Translation,Report,Dashboard,Document
        run: |
          if [ "$MODE" = "filtered" ]; then
            sf hardis:org:monitor:backup --target-org SOURCE_ORG --skip-doc
          else
            sf hardis:org:monitor:backup \
              --target-org SOURCE_ORG \
              --full \
              --exclude-namespaces \
              --full-apply-filters \
              --max-by-chunk "$CHUNK_SIZE" \
              --skip-doc
          fi
      
      - name: Deploy metadata to Technical Org
        run: |
          sf hardis:project:deploy:smart --target-org TECH
EOFINIT

print_success ".github/workflows/init-technical-org.yml created"

# sync-technical-org.yml
cat > "$PROJECT_ROOT/.github/workflows/sync-technical-org.yml" << 'EOFSYNC'
---
name: Sync Technical Org

on:
  workflow_run:
    workflows: ["Process Deployment (sfdx-hardis)"]
    branches: [main]
    types: [completed]
  
  schedule:
    - cron: '0 2 * * 0'
  
  workflow_dispatch:

permissions: read-all

jobs:
  sync_technical_org:
    runs-on: ubuntu-latest
    name: Sync Technical Org with MAIN
    timeout-minutes: 90
    
    if: ${{ github.event.workflow_run.conclusion == 'success' || github.event_name != 'workflow_run' }}
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}
          fetch-depth: 0
          ref: main
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      
      - name: Install SFDX and plugins
        run: |
          npm install --no-cache @salesforce/cli --global
          echo 'y' | sf plugins install sfdx-hardis
          echo 'y' | sf plugins install sfdx-git-delta
      
      - name: Authenticate to MAIN org
        env:
          SFDX_CLIENT_ID_MAIN: ${{ secrets.SFDX_CLIENT_ID_MAIN }}
          SFDX_CLIENT_KEY_MAIN: ${{ secrets.SFDX_CLIENT_KEY_MAIN }}
        run: |
          echo "$SFDX_CLIENT_KEY_MAIN" > ./server.key
          sf org login jwt \
            --client-id "$SFDX_CLIENT_ID_MAIN" \
            --jwt-key-file ./server.key \
            --username "auto-discover" \
            --alias MAIN
          rm -f ./server.key
      
      - name: Authenticate to Technical Org
        env:
          SFDX_AUTH_URL_TECHNICAL_ORG: ${{ secrets.SFDX_AUTH_URL_TECHNICAL_ORG }}
        run: |
          echo "$SFDX_AUTH_URL_TECHNICAL_ORG" > ./authfile_tech.txt
          sf org login sfdx-url \
            --sfdx-url-file ./authfile_tech.txt \
            --alias TECH
          rm -f ./authfile_tech.txt
      
      - name: Sync metadata
        env:
          MONITORING_BACKUP_SKIP_METADATA_TYPES: Translation,Report,Dashboard
        run: |
          sf hardis:org:monitor:backup --target-org MAIN --skip-doc
          sf hardis:project:deploy:smart --target-org TECH
EOFSYNC

print_success ".github/workflows/sync-technical-org.yml created"

# retrofit-production.yml
cat > "$PROJECT_ROOT/.github/workflows/retrofit-production.yml" << 'EOFRETRO'
---
name: Retrofit Production Changes

on:
  schedule:
    - cron: '0 6 * * *'
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  retrofit:
    runs-on: ubuntu-latest
    name: Retrofit from MAIN
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.PAT || secrets.GITHUB_TOKEN }}
          fetch-depth: 0
          ref: int
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      
      - name: Install SFDX and plugins
        run: |
          npm install --no-cache @salesforce/cli --global
          echo 'y' | sf plugins install sfdx-hardis
      
      - name: Configure Git
        run: |
          git config user.name "Retrofit Bot"
          git config user.email "retrofit-bot@company.com"
      
      - name: Authenticate to MAIN
        env:
          SFDX_CLIENT_ID_MAIN: ${{ secrets.SFDX_CLIENT_ID_MAIN }}
          SFDX_CLIENT_KEY_MAIN: ${{ secrets.SFDX_CLIENT_KEY_MAIN }}
        run: |
          echo "$SFDX_CLIENT_KEY_MAIN" > ./server.key
          sf org login jwt \
            --client-id "$SFDX_CLIENT_ID_MAIN" \
            --jwt-key-file ./server.key \
            --username "auto-discover" \
            --alias MAIN \
            --set-default
          rm -f ./server.key
      
      - name: Run retrofit
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          sf hardis:org:retrieve:sources:retrofit \
            --productionbranch main \
            --retrofitbranch int \
            --commit \
            --commitmode updated \
            --push \
            --pushmode mergerequest \
            --target-org MAIN
EOFRETRO

print_success ".github/workflows/retrofit-production.yml created"

# ============================================================================
# GENERATE MANIFEST FILES
# ============================================================================

print_header "📄 Generating Manifest Files"

# package-skip-items.xml
cat > "$PROJECT_ROOT/manifest/package-skip-items.xml" << 'EOFSKIP'
<?xml version="1.0" encoding="UTF-8"?>
<Package xmlns="http://soap.sforce.com/2006/04/metadata">
    <!-- Skip translations -->
    <types>
        <members>*</members>
        <name>Translation</name>
    </types>
    
    <!-- Skip reports -->
    <types>
        <members>*</members>
        <name>Report</name>
    </types>
    
    <!-- Skip dashboards -->
    <types>
        <members>*</members>
        <name>Dashboard</name>
    </types>
    
    <!-- Skip documents -->
    <types>
        <members>*</members>
        <name>Document</name>
    </types>
</Package>
EOFSKIP

print_success "manifest/package-skip-items.xml created"

# ============================================================================
# GENERATE UTILITY FILES
# ============================================================================

print_header "📄 Generating Utility Files"

# .gitignore additions
if [ -f "$PROJECT_ROOT/.gitignore" ]; then
    cat >> "$PROJECT_ROOT/.gitignore" << 'EOFGITIGNORE'

# sfdx-hardis
.sfdx-hardis/
**/*.dup

# Certificates (NEVER commit)
*.key
*.crt
*.pem
/certs/

# Auth files
authfile*.txt
server.key

# Logs
*.log
.apex-logs/
EOFGITIGNORE
    print_success ".gitignore updated"
else
    print_warning ".gitignore not found, skipping"
fi

# README.md
cat > "$PROJECT_ROOT/README.md" << EOFREADME
# $PROJECT_NAME - Salesforce DevOps

This project uses **sfdx-hardis** for CI/CD automation.

## Architecture

- **INT** (Integration): Development environment
- **RCT** (Recette): UAT/Testing environment  
- **MAIN** (Production): Production environment
- **Technical Org**: CI/CD operations, documentation

## Workflows

### Deployment Flow
\`\`\`
feature branch → PR to int → check-deploy → merge → deploy to INT
int → PR to rct → check-deploy → merge → deploy to RCT
rct → PR to main → check-deploy → merge → deploy to MAIN
\`\`\`

### Automated Jobs
- **Daily (6am)**: Retrofit production changes
- **Weekly (Sunday 2am)**: Sync Technical Org
- **Every 15min**: Collect deployment metrics

## Commands

### Deploy
\`\`\`bash
# Check deployment (validation)
sf hardis:project:deploy:smart --check --target-org INT

# Real deployment
sf hardis:project:deploy:smart --target-org RCT
\`\`\`

### Retrofit
\`\`\`bash
# Detect production changes
sf hardis:org:retrieve:sources:retrofit \\
  --productionbranch main \\
  --retrofitbranch int \\
  --commit --push
\`\`\`

### Auto-clean
\`\`\`bash
# Clean metadata
sf hardis:work:save --auto-clean
\`\`\`

## Documentation

- [sfdx-hardis docs](https://sfdx-hardis.cloudity.com/)
- [Setup guide](./docs/SETUP.md)

## Support

Contact: devops@${COMPANY_DOMAIN}
EOFREADME

print_success "README.md created"

# ============================================================================
# SUMMARY
# ============================================================================

print_header "✅ Configuration Files Generated"

echo "Summary of created files:"
echo ""
echo "Configuration:"
echo "  • config/.sfdx-hardis.yml"
echo "  • config/branches/.sfdx-hardis-int.yml"
echo "  • config/branches/.sfdx-hardis-rct.yml"
echo "  • config/branches/.sfdx-hardis-main.yml"
echo ""
echo "GitHub Workflows:"
echo "  • .github/workflows/check-deploy.yml"
echo "  • .github/workflows/process-deploy.yml"
echo "  • .github/workflows/megalinter.yml"
echo "  • .github/workflows/init-technical-org.yml"
echo "  • .github/workflows/sync-technical-org.yml"
echo "  • .github/workflows/retrofit-production.yml"
echo ""
echo "Manifest:"
echo "  • manifest/package-skip-items.xml"
echo ""
echo "Documentation:"
echo "  • README.md"
echo ""
echo -e "${GREEN}${BOLD}✓ All configuration files created successfully!${NC}"
echo ""
```

---

## 📄 `scripts/setup/04-init-technical-org.sh`

```bash
#!/bin/bash

# ============================================================================
# TECHNICAL ORG INITIALIZATION
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
RED='\033[0;31m'
NC='\033[0m'
BOLD='\033[1m'

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

print_step() {
    echo -e "${CYAN}▶${NC} $1"
}

print_success() {
    echo -e "${GREEN}✓${NC} $1"
}

print_error() {
    echo -e "${RED}✗${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}⚠${NC} $1"
}

# ============================================================================
# MAIN INITIALIZATION
# ============================================================================

print_header "🔧 TECHNICAL ORG INITIALIZATION"

echo "This script will initialize your Technical Org by:"
echo "  1. Backing up metadata from MAIN (production)"
echo "  2. Deploying to Technical Org"
echo "  3. Setting up monitoring"
echo ""
echo "⏱️  Estimated time: 30-60 minutes (for ~20,000 artifacts)"
echo ""

# Ask for source org
echo "Which org should be used as the source?"
echo "  1) MAIN (production) - Recommended"
echo "  2) RCT (recette)"
echo "  3) INT (integration)"
echo ""
read -p "Choice [1]: " SOURCE_CHOICE
SOURCE_CHOICE=${SOURCE_CHOICE:-1}

case $SOURCE_CHOICE in
    1) SOURCE_ORG="MAIN" ;;
    2) SOURCE_ORG="RCT" ;;
    3) SOURCE_ORG="INT" ;;
    *) 
        print_error "Invalid choice"
        exit 1
        ;;
esac

echo ""
echo "Selected source: ${CYAN}${BOLD}$SOURCE_ORG${NC}"
echo ""

# Ask for backup mode
echo "Which backup mode?"
echo "  1) Filtered (recommended, faster) - Excludes namespaces automatically"
echo "  2) Full with chunks (slower, comprehensive)"
echo ""
read -p "Choice [1]: " MODE_CHOICE
MODE_CHOICE=${MODE_CHOICE:-1}

if [ "$MODE_CHOICE" = "1" ]; then
    BACKUP_MODE="filtered"
    CHUNK_SIZE=""
else
    BACKUP_MODE="full"
    read -p "Chunk size [1000]: " CHUNK_SIZE
    CHUNK_SIZE=${CHUNK_SIZE:-1000}
fi

echo ""
echo -e "${YELLOW}${BOLD}⚠️  IMPORTANT:${NC}"
echo "  • This will take 30-60 minutes"
echo "  • Do not interrupt the process"
echo "  • Make sure you have a stable internet connection"
echo ""

read -p "Continue? [y/N]: " CONFIRM
if [ "$CONFIRM" != "y" ] && [ "$CONFIRM" != "Y" ]; then
    echo "Initialization cancelled."
    exit 0
fi

# ============================================================================
# STEP 1: AUTHENTICATE TO SOURCE ORG
# ============================================================================

print_header "Step 1: Authenticate to Source Org ($SOURCE_ORG)"

print_step "Logging in to $SOURCE_ORG..."

# Check if already authenticated
if sf org list --json | jq -e ".result.nonScratchOrgs[] | select(.alias == \"$SOURCE_ORG\")" > /dev/null 2>&1; then
    print_success "Already authenticated to $SOURCE_ORG"
else
    echo "Opening browser for authentication..."
    
    if [ "$SOURCE_ORG" = "MAIN" ]; then
        sf org login web --alias $SOURCE_ORG --instance-url https://login.salesforce.com
    else
        sf org login web --alias $SOURCE_ORG --instance-url https://test.salesforce.com
    fi
    
    if [ $? -eq 0 ]; then
        print_success "Successfully authenticated to $SOURCE_ORG"
    else
        print_error "Authentication failed"
        exit 1
    fi
fi

# Display org info
print_step "Org information:"
sf org display --target-org $SOURCE_ORG

# ============================================================================
# STEP 2: AUTHENTICATE TO TECHNICAL ORG
# ============================================================================

print_header "Step 2: Authenticate to Technical Org"

print_step "Logging in to Technical Org..."

if sf org list --json | jq -e '.result.nonScratchOrgs[] | select(.alias == "TechnicalOrg")' > /dev/null 2>&1; then
    print_success "Already authenticated to TechnicalOrg"
else
    echo "Opening browser for authentication..."
    
    sf org login web --alias TechnicalOrg --instance-url https://test.salesforce.com
    
    if [ $? -eq 0 ]; then
        print_success "Successfully authenticated to TechnicalOrg"
    else
        print_error "Authentication failed"
        exit 1
    fi
fi

# Display org info
print_step "Technical Org information:"
sf org display --target-org TechnicalOrg

# ============================================================================
# STEP 3: COUNT METADATA IN SOURCE ORG
# ============================================================================

print_header "Step 3: Analyzing Source Org"

print_step "Counting metadata items..."

# Count Apex Classes
APEX_COUNT=$(sf data query --query "SELECT COUNT() FROM ApexClass" --target-org $SOURCE_ORG --json 2>/dev/null | jq -r '.result.totalSize // 0')
echo "  Apex Classes: $APEX_COUNT"

# Count Flows
FLOW_COUNT=$(sf data query --query "SELECT COUNT() FROM FlowDefinitionView" --target-org $SOURCE_ORG --json 2>/dev/null | jq -r '.result.totalSize // 0')
echo "  Flows: $FLOW_COUNT"

# Count Custom Objects
OBJ_COUNT=$(sf data query --query "SELECT COUNT() FROM CustomObject WHERE NamespacePrefix = null" --target-org $SOURCE_ORG --json 2>/dev/null | jq -r '.result.totalSize // 0')
echo "  Custom Objects (without namespace): $OBJ_COUNT"

TOTAL_ESTIMATED=$((APEX_COUNT + FLOW_COUNT + OBJ_COUNT * 5))
echo ""
echo "  Estimated metadata items: ~${TOTAL_ESTIMATED}"
echo ""

if [ $TOTAL_ESTIMATED -gt 15000 ]; then
    print_warning "Large org detected (>15,000 items)"
    echo "  • Backup may take 45-60 minutes"
    echo "  • Using chunk size: ${CHUNK_SIZE:-auto}"
fi

# ============================================================================
# STEP 4: BACKUP METADATA FROM SOURCE ORG
# ============================================================================

print_header "Step 4: Backup Metadata from $SOURCE_ORG"

print_step "Starting backup (mode: $BACKUP_MODE)..."
echo ""

START_TIME=$(date +%s)

if [ "$BACKUP_MODE" = "filtered" ]; then
    # Filtered mode (recommended)
    sf hardis:org:monitor:backup \
        --target-org $SOURCE_ORG \
        --skip-doc \
        || {
            print_error "Backup failed"
            exit 1
        }
else
    # Full mode with chunking
    sf hardis:org:monitor:backup \
        --target-org $SOURCE_ORG \
        --full \
        --exclude-namespaces \
        --full-apply-filters \
        --max-by-chunk $CHUNK_SIZE \
        --skip-doc \
        || {
            print_error "Backup failed"
            exit 1
        }
fi

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
MINUTES=$((DURATION / 60))
SECONDS=$((DURATION % 60))

echo ""
print_success "Backup completed in ${MINUTES}m ${SECONDS}s"

# ============================================================================
# STEP 5: INSTALL PACKAGES IN TECHNICAL ORG
# ============================================================================

print_header "Step 5: Install Managed Packages"

if [ -f "config/.sfdx-hardis.yml" ] && grep -q "installedPackages:" "config/.sfdx-hardis.yml"; then
    print_step "Installing managed packages..."
    
    sf hardis:package:install \
        --target-org TechnicalOrg \
        || {
            print_warning "Some packages may have failed to install (might already be installed)"
        }
    
    print_success "Package installation completed"
else
    print_step "No managed packages configured, skipping..."
fi

# ============================================================================
# STEP 6: DEPLOY TO TECHNICAL ORG
# ============================================================================

print_header "Step 6: Deploy Metadata to Technical Org"

print_step "Starting deployment..."
echo ""

START_TIME=$(date +%s)

sf hardis:project:deploy:smart \
    --target-org TechnicalOrg \
    || {
        print_error "Deployment failed"
        echo ""
        echo "Common issues:"
        echo "  • Missing dependencies"
        echo "  • Package installation failures"
        echo "  • Org limits reached"
        echo ""
        echo "Check the deployment logs above for details."
        exit 1
    }

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
MINUTES=$((DURATION / 60))
SECONDS=$((DURATION % 60))

echo ""
print_success "Deployment completed in ${MINUTES}m ${SECONDS}s"

# ============================================================================
# STEP 7: VERIFY DEPLOYMENT
# ============================================================================

print_header "Step 7: Verify Deployment"

print_step "Verifying metadata counts..."

# Count in Technical Org
TECH_APEX=$(sf data query --query "SELECT COUNT() FROM ApexClass" --target-org TechnicalOrg --json 2>/dev/null | jq -r '.result.totalSize // 0')
TECH_FLOW=$(sf data query --query "SELECT COUNT() FROM FlowDefinitionView" --target-org TechnicalOrg --json 2>/dev/null | jq -r '.result.totalSize // 0')

echo ""
echo "Comparison:"
echo "  Apex Classes:"
echo "    Source ($SOURCE_ORG): $APEX_COUNT"
echo "    Technical: $TECH_APEX"
echo ""
echo "  Flows:"
echo "    Source ($SOURCE_ORG): $FLOW_COUNT"
echo "    Technical: $TECH_FLOW"
echo ""

# Check if counts are similar (allow 10% difference)
APEX_DIFF=$((APEX_COUNT - TECH_APEX))
APEX_DIFF=${APEX_DIFF#-}  # Absolute value
APEX_PERCENT=$((APEX_DIFF * 100 / APEX_COUNT))

if [ $APEX_PERCENT -gt 10 ]; then
    print_warning "Significant difference in Apex Classes count (${APEX_PERCENT}%)"
    echo "  • This might be expected if some classes are namespaced"
else
    print_success "Apex Classes count is similar"
fi

FLOW_DIFF=$((FLOW_COUNT - TECH_FLOW))
FLOW_DIFF=${FLOW_DIFF#-}
FLOW_PERCENT=$((FLOW_DIFF * 100 / FLOW_COUNT))

if [ $FLOW_PERCENT -gt 10 ]; then
    print_warning "Significant difference in Flows count (${FLOW_PERCENT}%)"
else
    print_success "Flows count is similar"
fi

# ============================================================================
# STEP 8: GENERATE SFDX AUTH URL
# ============================================================================

print_header "Step 8: Generate SFDX Auth URL for GitHub"

print_step "Generating Auth URL..."

AUTH_URL=$(sf org display --target-org TechnicalOrg --verbose --json 2>/dev/null | jq -r '.result.sfdxAuthUrl' 2>/dev/null)

if [ -n "$AUTH_URL" ] && [ "$AUTH_URL" != "null" ]; then
    echo ""
    echo -e "${YELLOW}${BOLD}⚠️  SAVE THIS VALUE AS GITHUB SECRET:${NC}"
    echo ""
    echo "Secret name: ${CYAN}SFDX_AUTH_URL_TECHNICAL_ORG${NC}"
    echo "Secret value:"
    echo -e "${GREEN}$AUTH_URL${NC}"
    echo ""
    
    # Save to file
    AUTH_FILE="$HOME/.sfdx-hardis/technical-org-auth-url.txt"
    echo "$AUTH_URL" > "$AUTH_FILE"
    print_success "Auth URL saved to: $AUTH_FILE"
else
    print_error "Could not generate Auth URL"
    echo ""
    echo "Please run manually:"
    echo "  sf org display --target-org TechnicalOrg --verbose --json | jq -r '.result.sfdxAuthUrl'"
fi

# ============================================================================
# SUMMARY
# ============================================================================

print_header "✅ TECHNICAL ORG INITIALIZATION COMPLETE"

echo ""
echo -e "${GREEN}${BOLD}Success! Technical Org is ready.${NC}"
echo ""
echo "Summary:"
echo "  • Source: $SOURCE_ORG"
echo "  • Backup mode: $BACKUP_MODE"
echo "  • Apex Classes deployed: $TECH_APEX"
echo "  • Flows deployed: $TECH_FLOW"
echo ""
echo "Next steps:"
echo "  1. Save the Auth URL to GitHub Secrets"
echo "  2. Test a deployment workflow"
echo "  3. Set up scheduled sync (runs automatically)"
echo ""
echo "The Technical Org will be automatically synced:"
echo "  • After each deployment to MAIN"
echo "  • Every Sunday at 2am (scheduled)"
echo ""
```

---

## 📄 `scripts/setup/05-validate-setup.sh`

```bash
#!/bin/bash

# ============================================================================
# SETUP VALIDATION
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'
BOLD='\033[1m'

PROJECT_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")/../.." && pwd)"

ERRORS=0
WARNINGS=0
SUCCESS=0

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

check_pass() {
    echo -e "${GREEN}✓${NC} $1"
    SUCCESS=$((SUCCESS + 1))
}

check_fail() {
    echo -e "${RED}✗${NC} $1"
    ERRORS=$((ERRORS + 1))
}

check_warn() {
    echo -e "${YELLOW}⚠${NC} $1"
    WARNINGS=$((WARNINGS + 1))
}

# ============================================================================
# VALIDATE CONFIGURATION FILES
# ============================================================================

print_header "📋 Validating Configuration Files"

# Main config
if [ -f "$PROJECT_ROOT/config/.sfdx-hardis.yml" ]; then
    check_pass "config/.sfdx-hardis.yml exists"
    
    # Check for required fields
    if grep -q "projectName:" "$PROJECT_ROOT/config/.sfdx-hardis.yml"; then
        check_pass "projectName is defined"
    else
        check_fail "projectName is missing"
    fi
    
    if grep -q "developmentBranch: int" "$PROJECT_ROOT/config/.sfdx-hardis.yml"; then
        check_pass "developmentBranch is set to 'int'"
    else
        check_warn "developmentBranch might not be set correctly"
    fi
else
    check_fail "config/.sfdx-hardis.yml not found"
fi

# Branch configs
for BRANCH in int rct main; do
    if [ -f "$PROJECT_ROOT/config/branches/.sfdx-hardis-${BRANCH}.yml" ]; then
        check_pass "config/branches/.sfdx-hardis-${BRANCH}.yml exists"
    else
        check_fail "config/branches/.sfdx-hardis-${BRANCH}.yml not found"
    fi
done

# ============================================================================
# VALIDATE GITHUB WORKFLOWS
# ============================================================================

print_header "📋 Validating GitHub Workflows"

REQUIRED_WORKFLOWS=(
    "check-deploy.yml"
    "process-deploy.yml"
    "megalinter.yml"
    "init-technical-org.yml"
    "sync-technical-org.yml"
    "retrofit-production.yml"
)

for WORKFLOW in "${REQUIRED_WORKFLOWS[@]}"; do
    if [ -f "$PROJECT_ROOT/.github/workflows/$WORKFLOW" ]; then
        check_pass ".github/workflows/$WORKFLOW exists"
        
        # Check if branches are correct
        if grep -q "branches:" "$PROJECT_ROOT/.github/workflows/$WORKFLOW"; then
            if grep -q "- int" "$PROJECT_ROOT/.github/workflows/$WORKFLOW" && \
               grep -q "- rct" "$PROJECT_ROOT/.github/workflows/$WORKFLOW" && \
               grep -q "- main" "$PROJECT_ROOT/.github/workflows/$WORKFLOW"; then
                check_pass "Branches (int, rct, main) configured in $WORKFLOW"
            else
                check_warn "Branch configuration might be incomplete in $WORKFLOW"
            fi
        fi
    else
        check_fail ".github/workflows/$WORKFLOW not found"
    fi
done

# ============================================================================
# VALIDATE SALESFORCE AUTHENTICATION
# ============================================================================

print_header "🔐 Validating Salesforce Authentication"

# Check if orgs are authenticated
ORGS=("INT" "RCT" "MAIN" "TechnicalOrg")

for ORG in "${ORGS[@]}"; do
    if sf org list --json 2>/dev/null | jq -e ".result.nonScratchOrgs[] | select(.alias == \"$ORG\")" > /dev/null 2>&1; then
        check_pass "$ORG org is authenticated"
    else
        check_warn "$ORG org is not authenticated locally"
        echo "     Run: sf org login web --alias $ORG"
    fi
done

# ============================================================================
# VALIDATE GITHUB CONFIGURATION
# ============================================================================

print_header "🔑 Validating GitHub Configuration"

if command -v gh &> /dev/null; then
    if gh auth status &> /dev/null; then
        check_pass "GitHub CLI is authenticated"
        
        # Try to get current repo
        CURRENT_REPO=$(git remote get-url origin 2>/dev/null | sed 's/.*github.com[:/]\(.*\)\.git/\1/')
        
        if [ -n "$CURRENT_REPO" ]; then
            check_pass "GitHub repository detected: $CURRENT_REPO"
            
            # Check secrets (this will only work if you have admin access)
            echo ""
            echo "Checking GitHub Secrets..."
            
            REQUIRED_SECRETS=(
                "SFDX_CLIENT_ID_INT"
                "SFDX_CLIENT_KEY_INT"
                "SFDX_CLIENT_ID_RCT"
                "SFDX_CLIENT_KEY_RCT"
                "SFDX_CLIENT_ID_MAIN"
                "SFDX_CLIENT_KEY_MAIN"
                "SFDX_AUTH_URL_TECHNICAL_ORG"
            )
            
            for SECRET in "${REQUIRED_SECRETS[@]}"; do
                if gh secret list -R "$CURRENT_REPO" 2>/dev/null | grep -q "^$SECRET"; then
                    check_pass "Secret $SECRET is configured"
                else
                    check_warn "Secret $SECRET might not be configured"
                fi
            done
        else
            check_warn "Could not detect GitHub repository"
        fi
    else
        check_warn "GitHub CLI is not authenticated"
        echo "     Run: gh auth login"
    fi
else
    check_warn "GitHub CLI not installed (optional)"
fi

# ============================================================================
# VALIDATE MANIFEST FILES
# ============================================================================

print_header "📋 Validating Manifest Files"

if [ -f "$PROJECT_ROOT/manifest/package-skip-items.xml" ]; then
    check_pass "manifest/package-skip-items.xml exists"
else
    check_fail "manifest/package-skip-items.xml not found"
fi

# ============================================================================
# VALIDATE GIT CONFIGURATION
# ============================================================================

print_header "📋 Validating Git Configuration"

if [ -d "$PROJECT_ROOT/.git" ]; then
    check_pass "Git repository initialized"
    
    # Check branches
    CURRENT_BRANCH=$(git branch --show-current 2>/dev/null)
    if [ -n "$CURRENT_BRANCH" ]; then
        check_pass "Current branch: $CURRENT_BRANCH"
    fi
    
    # Check if required branches exist (locally or remotely)
    for BRANCH in int rct main; do
        if git show-ref --verify --quiet "refs/heads/$BRANCH" 2>/dev/null || \
           git show-ref --verify --quiet "refs/remotes/origin/$BRANCH" 2>/dev/null; then
            check_pass "Branch '$BRANCH' exists"
        else
            check_warn "Branch '$BRANCH' does not exist yet"
            echo "     Create with: git branch $BRANCH"
        fi
    done
else
    check_fail "Not a Git repository"
fi

# Check .gitignore
if [ -f "$PROJECT_ROOT/.gitignore" ]; then
    check_pass ".gitignore exists"
    
    if grep -q "\.sfdx-hardis/" "$PROJECT_ROOT/.gitignore"; then
        check_pass ".gitignore includes sfdx-hardis patterns"
    else
        check_warn ".gitignore might be missing sfdx-hardis patterns"
    fi
    
    if grep -q "\.key" "$PROJECT_ROOT/.gitignore"; then
        check_pass ".gitignore protects certificate files"
    else
        check_fail ".gitignore does NOT protect certificate files"
        echo "     Add: *.key, *.crt, *.pem"
    fi
else
    check_fail ".gitignore not found"
fi

# ============================================================================
# VALIDATE SFDX PROJECT
# ============================================================================

print_header "📋 Validating SFDX Project"

if [ -f "$PROJECT_ROOT/sfdx-project.json" ]; then
    check_pass "sfdx-project.json exists"
    
    # Validate JSON
    if jq empty "$PROJECT_ROOT/sfdx-project.json" 2>/dev/null; then
        check_pass "sfdx-project.json is valid JSON"
    else
        check_fail "sfdx-project.json has JSON syntax errors"
    fi
else
    check_fail "sfdx-project.json not found"
fi

# ============================================================================
# VALIDATE PLUGINS
# ============================================================================

print_header "📋 Validating SFDX Plugins"

if sf plugins | grep -q "sfdx-hardis"; then
    VERSION=$(sf plugins | grep "sfdx-hardis" | awk '{print $2}')
    check_pass "sfdx-hardis plugin installed (version $VERSION)"
else
    check_fail "sfdx-hardis plugin not installed"
    echo "     Install with: sf plugins install sfdx-hardis"
fi

if sf plugins | grep -q "sfdx-git-delta"; then
    check_pass "sfdx-git-delta plugin installed"
else
    check_warn "sfdx-git-delta plugin not installed (optional but recommended)"
fi

# ============================================================================
# FINAL SUMMARY
# ============================================================================

print_header "📊 VALIDATION SUMMARY"

echo ""
echo "Results:"
echo -e "  ${GREEN}✓ Passed:${NC} $SUCCESS"
echo -e "  ${YELLOW}⚠ Warnings:${NC} $WARNINGS"
echo -e "  ${RED}✗ Errors:${NC} $ERRORS"
echo ""

if [ $ERRORS -eq 0 ] && [ $WARNINGS -eq 0 ]; then
    echo -e "${GREEN}${BOLD}🎉 Perfect! Your setup is complete and validated.${NC}"
    echo ""
    echo "You're ready to:"
    echo "  1. Commit and push your changes"
    echo "  2. Test your first deployment"
    echo "  3. Create a Pull Request"
    echo ""
    exit 0
elif [ $ERRORS -eq 0 ]; then
    echo -e "${YELLOW}${BOLD}⚠️  Setup is mostly complete with some warnings.${NC}"
    echo ""
    echo "Review the warnings above and fix if needed."
    echo "Most warnings are optional items."
    echo ""
    exit 0
else
    echo -e "${RED}${BOLD}❌ Setup has errors that need to be fixed.${NC}"
    echo ""
    echo "Please review the errors above and:"
    echo "  • Fix missing configuration files"
    echo "  • Complete required setup steps"
    echo "  • Run validation again"
    echo ""
    exit 1
fi
```

---

Et voilà ! 🎉 Vous avez maintenant les **3 scripts finaux** :

- **03-create-config-files.sh** : Génère tous les fichiers de configuration (yml, workflows, manifests)
- **04-init-technical-org.sh** : Initialise l'org technique depuis MAIN
- **05-validate-setup.sh** : Valide l'installation complète

Ces scripts forment un **système d'installation complet et automatisé** qui :
- ✅ Guide l'utilisateur pas à pas
- ✅ Génère tous les fichiers nécessaires
- ✅ Valide la configuration
- ✅ Gère les erreurs proprement
- ✅ Fournit des messages clairs et colorés

Voulez-vous maintenant que je crée :
1. **Le guide étape par étape détaillé** (avec captures d'écran textuelles)
2. **Scripts de troubleshooting**
3. **Scripts de test pour valider le CI/CD**

Ou peut-être souhaitez-vous que je package tout cela dans un **ZIP téléchargeable** avec une documentation complète ? 📦
