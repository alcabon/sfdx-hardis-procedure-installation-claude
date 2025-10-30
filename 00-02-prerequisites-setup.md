Merci beaucoup ! 🙏 Ça me fait vraiment plaisir que cette synthèse vous soit utile ! Je vais créer **les deux** : des scripts d'installation automatisée ET un guide détaillé étape par étape ultra-complet.

# 🚀 INSTALLATION AUTOMATISÉE + GUIDE COMPLET

## Structure de ce qui suit

1. **Scripts d'installation automatisée** (shell scripts)
2. **Guide étape par étape détaillé** (avec screenshots textuels)
3. **Scripts de validation**
4. **Troubleshooting guide**

---

# 1️⃣ SCRIPTS D'INSTALLATION AUTOMATISÉE

## 📁 Structure des scripts

```
scripts/
├── setup/
│   ├── 00-prerequisites.sh          # Vérification prérequis
│   ├── 01-create-connected-apps.sh  # Guide création Connected Apps
│   ├── 02-setup-github-secrets.sh   # Configuration secrets GitHub
│   ├── 03-create-config-files.sh    # Génération fichiers config
│   ├── 04-init-technical-org.sh     # Initialisation Technical Org
│   └── 05-validate-setup.sh         # Validation complète
├── utils/
│   ├── colors.sh                     # Couleurs terminal
│   ├── logger.sh                     # Logging
│   └── helpers.sh                    # Fonctions utilitaires
└── install.sh                        # Script principal
```

---

## 📄 `scripts/install.sh` - Script principal

```bash
#!/bin/bash

# ============================================================================
# SFDX-HARDIS COMPLETE SETUP - Main Installation Script
# ============================================================================
# Description: Automated setup for sfdx-hardis CI/CD pipeline
# Author: DevOps Team
# Version: 1.0.0
# ============================================================================

set -e  # Exit on error

# Colors and formatting
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
MAGENTA='\033[0;35m'
CYAN='\033[0;36m'
NC='\033[0m' # No Color
BOLD='\033[1m'

# Script directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
PROJECT_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"

# Load utilities
source "$SCRIPT_DIR/utils/colors.sh" 2>/dev/null || true
source "$SCRIPT_DIR/utils/logger.sh" 2>/dev/null || true

# ============================================================================
# FUNCTIONS
# ============================================================================

print_banner() {
    echo ""
    echo -e "${CYAN}${BOLD}╔════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${CYAN}${BOLD}║                                                                ║${NC}"
    echo -e "${CYAN}${BOLD}║          🚀 SFDX-HARDIS CI/CD COMPLETE SETUP 🚀               ║${NC}"
    echo -e "${CYAN}${BOLD}║                                                                ║${NC}"
    echo -e "${CYAN}${BOLD}║              Architecture: INT → RCT → MAIN                    ║${NC}"
    echo -e "${CYAN}${BOLD}║                                                                ║${NC}"
    echo -e "${CYAN}${BOLD}╚════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
}

print_section() {
    echo ""
    echo -e "${BLUE}${BOLD}═══════════════════════════════════════════════════════════════${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}═══════════════════════════════════════════════════════════════${NC}"
    echo ""
}

print_step() {
    echo -e "${GREEN}${BOLD}▶${NC} $1"
}

print_success() {
    echo -e "${GREEN}${BOLD}✓${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}${BOLD}⚠${NC} $1"
}

print_error() {
    echo -e "${RED}${BOLD}✗${NC} $1"
}

print_info() {
    echo -e "${CYAN}ℹ${NC} $1"
}

ask_yes_no() {
    local question="$1"
    local default="${2:-n}"
    
    if [ "$default" = "y" ]; then
        local prompt="[Y/n]"
    else
        local prompt="[y/N]"
    fi
    
    while true; do
        read -p "$(echo -e ${YELLOW}$question $prompt:${NC} )" answer
        answer=${answer:-$default}
        case $answer in
            [Yy]* ) return 0;;
            [Nn]* ) return 1;;
            * ) echo "Please answer yes or no.";;
        esac
    done
}

# ============================================================================
# MAIN INSTALLATION FLOW
# ============================================================================

main() {
    print_banner
    
    echo -e "${BOLD}This script will guide you through the complete setup of:${NC}"
    echo "  • Salesforce Connected Apps (4 orgs)"
    echo "  • GitHub Secrets configuration"
    echo "  • SFDX-Hardis configuration files"
    echo "  • Technical Org initialization"
    echo "  • CI/CD workflows setup"
    echo ""
    
    if ! ask_yes_no "Ready to start the installation?" "y"; then
        echo "Installation cancelled."
        exit 0
    fi
    
    # Step 0: Prerequisites check
    print_section "📋 STEP 0: Checking Prerequisites"
    bash "$SCRIPT_DIR/setup/00-prerequisites.sh"
    
    # Step 1: Connected Apps creation guide
    print_section "🔐 STEP 1: Salesforce Connected Apps Setup"
    bash "$SCRIPT_DIR/setup/01-create-connected-apps.sh"
    
    # Step 2: GitHub Secrets configuration
    print_section "🔑 STEP 2: GitHub Secrets Configuration"
    bash "$SCRIPT_DIR/setup/02-setup-github-secrets.sh"
    
    # Step 3: Configuration files creation
    print_section "📝 STEP 3: Configuration Files Generation"
    bash "$SCRIPT_DIR/setup/03-create-config-files.sh"
    
    # Step 4: Technical Org initialization
    print_section "🔧 STEP 4: Technical Org Initialization"
    if ask_yes_no "Initialize Technical Org now?" "y"; then
        bash "$SCRIPT_DIR/setup/04-init-technical-org.sh"
    else
        print_warning "Technical Org initialization skipped. You can run it later with:"
        echo "  bash scripts/setup/04-init-technical-org.sh"
    fi
    
    # Step 5: Validation
    print_section "✅ STEP 5: Setup Validation"
    bash "$SCRIPT_DIR/setup/05-validate-setup.sh"
    
    # Success!
    print_section "🎉 INSTALLATION COMPLETE!"
    echo ""
    echo -e "${GREEN}${BOLD}✓ All steps completed successfully!${NC}"
    echo ""
    echo -e "${BOLD}Next steps:${NC}"
    echo "  1. Review generated configuration files"
    echo "  2. Commit and push to GitHub"
    echo "  3. Test your first deployment"
    echo ""
    echo -e "${CYAN}Documentation: https://sfdx-hardis.cloudity.com/${NC}"
    echo ""
}

# Run main function
main "$@"
```

---

## 📄 `scripts/setup/00-prerequisites.sh`

```bash
#!/bin/bash

# ============================================================================
# PREREQUISITES CHECK
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m'

print_check() {
    echo -e "${GREEN}✓${NC} $1"
}

print_fail() {
    echo -e "${RED}✗${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}⚠${NC} $1"
}

ERRORS=0

echo "Checking prerequisites..."
echo ""

# Check Node.js
echo -n "Checking Node.js... "
if command -v node &> /dev/null; then
    NODE_VERSION=$(node --version)
    print_check "Node.js $NODE_VERSION installed"
else
    print_fail "Node.js is not installed"
    echo "   Install from: https://nodejs.org/"
    ERRORS=$((ERRORS + 1))
fi

# Check npm
echo -n "Checking npm... "
if command -v npm &> /dev/null; then
    NPM_VERSION=$(npm --version)
    print_check "npm $NPM_VERSION installed"
else
    print_fail "npm is not installed"
    ERRORS=$((ERRORS + 1))
fi

# Check Salesforce CLI
echo -n "Checking Salesforce CLI... "
if command -v sf &> /dev/null; then
    SF_VERSION=$(sf version --json | jq -r '.cliVersion' 2>/dev/null || sf version)
    print_check "Salesforce CLI installed"
else
    print_fail "Salesforce CLI is not installed"
    echo "   Install with: npm install -g @salesforce/cli"
    ERRORS=$((ERRORS + 1))
fi

# Check sfdx-hardis plugin
echo -n "Checking sfdx-hardis plugin... "
if sf plugins | grep -q "sfdx-hardis"; then
    print_check "sfdx-hardis plugin installed"
else
    print_warning "sfdx-hardis plugin is not installed"
    echo "   Install with: sf plugins install sfdx-hardis"
    if ask_yes_no "Install now?" "y"; then
        echo 'y' | sf plugins install sfdx-hardis
        print_check "sfdx-hardis plugin installed"
    else
        ERRORS=$((ERRORS + 1))
    fi
fi

# Check Git
echo -n "Checking Git... "
if command -v git &> /dev/null; then
    GIT_VERSION=$(git --version | cut -d' ' -f3)
    print_check "Git $GIT_VERSION installed"
else
    print_fail "Git is not installed"
    ERRORS=$((ERRORS + 1))
fi

# Check GitHub CLI (optional but recommended)
echo -n "Checking GitHub CLI... "
if command -v gh &> /dev/null; then
    GH_VERSION=$(gh --version | head -n1 | cut -d' ' -f3)
    print_check "GitHub CLI $GH_VERSION installed"
else
    print_warning "GitHub CLI is not installed (optional but recommended)"
    echo "   Install from: https://cli.github.com/"
fi

# Check jq (for JSON parsing)
echo -n "Checking jq... "
if command -v jq &> /dev/null; then
    JQ_VERSION=$(jq --version)
    print_check "jq $JQ_VERSION installed"
else
    print_warning "jq is not installed (recommended for scripting)"
    echo "   Install with: brew install jq (macOS) or apt-get install jq (Linux)"
fi

# Check openssl
echo -n "Checking openssl... "
if command -v openssl &> /dev/null; then
    print_check "openssl installed"
else
    print_fail "openssl is not installed"
    echo "   Required for generating certificates"
    ERRORS=$((ERRORS + 1))
fi

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
if [ $ERRORS -eq 0 ]; then
    echo -e "${GREEN}✓ All prerequisites met!${NC}"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    exit 0
else
    echo -e "${RED}✗ $ERRORS prerequisite(s) missing${NC}"
    echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    echo "Please install missing dependencies and run again."
    exit 1
fi
```

---

## 📄 `scripts/setup/01-create-connected-apps.sh`

```bash
#!/bin/bash

# ============================================================================
# CONNECTED APPS CREATION GUIDE
# ============================================================================

set -e

# Colors
BLUE='\033[0;34m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
NC='\033[0m'
BOLD='\033[1m'

CERT_DIR="$HOME/.sfdx-hardis/certs"
mkdir -p "$CERT_DIR"

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

generate_certificate() {
    local env=$1
    local cert_file="$CERT_DIR/server_${env}.key"
    local crt_file="$CERT_DIR/server_${env}.crt"
    
    echo -e "${CYAN}Generating certificate for ${env}...${NC}"
    
    openssl req -x509 -newkey rsa:4096 \
        -keyout "$cert_file" \
        -out "$crt_file" \
        -days 3650 \
        -nodes \
        -subj "/C=US/ST=State/L=City/O=Company/OU=IT/CN=sfdx-hardis-${env}"
    
    echo -e "${GREEN}✓ Certificate generated:${NC}"
    echo "  Private Key: $cert_file"
    echo "  Certificate: $crt_file"
    echo ""
    
    echo "$cert_file"  # Return path
}

print_header "🔐 SALESFORCE CONNECTED APPS SETUP"

echo "This script will guide you through creating 4 Connected Apps:"
echo "  • INT Environment"
echo "  • RCT Environment"
echo "  • MAIN Environment (Production)"
echo "  • Technical Org"
echo ""

ORGS=("INT" "RCT" "MAIN" "TECHNICAL")
declare -A CLIENT_IDS
declare -A CERT_PATHS

for ORG in "${ORGS[@]}"; do
    print_header "Creating Connected App for: $ORG"
    
    # Generate certificate
    CERT_PATH=$(generate_certificate "$ORG")
    CERT_PATHS[$ORG]=$CERT_PATH
    CRT_PATH="${CERT_PATH%.key}.crt"
    
    echo -e "${BOLD}📋 Follow these steps in Salesforce:${NC}"
    echo ""
    echo "1. Log in to your ${ORG} org:"
    
    if [ "$ORG" = "MAIN" ]; then
        echo "   ${CYAN}https://login.salesforce.com${NC}"
    else
        echo "   ${CYAN}https://test.salesforce.com${NC}"
    fi
    
    echo ""
    echo "2. Navigate to:"
    echo "   Setup → Apps → App Manager → New Connected App"
    echo ""
    echo "3. Fill in the form:"
    echo "   • Connected App Name: ${BOLD}GitHub Actions CI/CD - ${ORG}${NC}"
    echo "   • API Name: ${BOLD}GitHub_Actions_CICD_${ORG}${NC}"
    echo "   • Contact Email: your-email@company.com"
    echo ""
    echo "4. Enable OAuth Settings:"
    echo "   ☑ Enable OAuth Settings"
    echo "   • Callback URL: ${CYAN}http://localhost:1717/OauthRedirect${NC}"
    echo ""
    echo "5. Selected OAuth Scopes (add these):"
    echo "   ☑ Full access (full)"
    echo "   ☑ Perform requests at any time (refresh_token, offline_access)"
    echo ""
    echo "6. Enable Digital Signatures:"
    echo "   ☑ Use digital signatures"
    echo "   • Click 'Choose File'"
    echo "   • Upload: ${CYAN}$CRT_PATH${NC}"
    echo ""
    echo "7. Click Save"
    echo ""
    echo "8. After saving:"
    echo "   • Click 'Manage Consumer Details'"
    echo "   • Verify your identity"
    echo "   • Copy the ${BOLD}Consumer Key${NC}"
    echo ""
    
    read -p "Press Enter when you have copied the Consumer Key..."
    
    echo ""
    read -p "Paste the Consumer Key for ${ORG}: " CONSUMER_KEY
    CLIENT_IDS[$ORG]=$CONSUMER_KEY
    
    echo -e "${GREEN}✓ Consumer Key saved for ${ORG}${NC}"
    echo ""
    
    # Wait a bit before next org
    if [ "$ORG" != "TECHNICAL" ]; then
        echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
        echo ""
        sleep 2
    fi
done

# Generate Technical Org Auth URL
print_header "🔧 GENERATING TECHNICAL ORG AUTH URL"

echo "Now we need to generate the SFDX Auth URL for the Technical Org."
echo ""
echo "Run this command and follow the browser login:"
echo ""
echo -e "${CYAN}sf org login web --alias TechnicalOrg${NC}"
echo ""
read -p "Press Enter when you've logged in..."

echo ""
echo "Generating Auth URL..."
AUTH_URL=$(sf org display --target-org TechnicalOrg --verbose --json 2>/dev/null | jq -r '.result.sfdxAuthUrl' 2>/dev/null || echo "ERROR")

if [ "$AUTH_URL" = "ERROR" ] || [ -z "$AUTH_URL" ]; then
    echo -e "${YELLOW}⚠ Could not auto-generate Auth URL${NC}"
    echo ""
    echo "Please run this command manually:"
    echo -e "${CYAN}sf org display --target-org TechnicalOrg --verbose --json | jq -r '.result.sfdxAuthUrl'${NC}"
    echo ""
    read -p "Paste the Auth URL: " AUTH_URL
fi

# Save everything to a file
OUTPUT_FILE="$CERT_DIR/secrets.txt"

cat > "$OUTPUT_FILE" << EOF
# ============================================================================
# SALESFORCE CONNECTED APPS CREDENTIALS
# Generated: $(date)
# ============================================================================

# INT Environment
SFDX_CLIENT_ID_INT=${CLIENT_IDS[INT]}
SFDX_CLIENT_KEY_INT_PATH=${CERT_PATHS[INT]}

# RCT Environment
SFDX_CLIENT_ID_RCT=${CLIENT_IDS[RCT]}
SFDX_CLIENT_KEY_RCT_PATH=${CERT_PATHS[RCT]}

# MAIN Environment (Production)
SFDX_CLIENT_ID_MAIN=${CLIENT_IDS[MAIN]}
SFDX_CLIENT_KEY_MAIN_PATH=${CERT_PATHS[MAIN]}

# Technical Org
SFDX_CLIENT_ID_TECHNICAL=${CLIENT_IDS[TECHNICAL]}
SFDX_CLIENT_KEY_TECHNICAL_PATH=${CERT_PATHS[TECHNICAL]}
SFDX_AUTH_URL_TECHNICAL_ORG=${AUTH_URL}

# ============================================================================
# GITHUB SECRETS
# Copy these values to GitHub: Settings > Secrets and variables > Actions
# ============================================================================

# For each secret, you'll need to:
# 1. Copy the value
# 2. For *_KEY secrets, copy the ENTIRE content of the .key file
# 3. Paste in GitHub Secrets

# Example for SFDX_CLIENT_KEY_INT:
# cat ${CERT_PATHS[INT]}
# Copy all content (including BEGIN/END lines) → GitHub Secret

EOF

echo ""
echo -e "${GREEN}${BOLD}✓ All Connected Apps configured!${NC}"
echo ""
echo "Credentials saved to: ${CYAN}$OUTPUT_FILE${NC}"
echo ""
echo -e "${YELLOW}${BOLD}⚠ IMPORTANT SECURITY NOTE:${NC}"
echo "  • Keep this file secure"
echo "  • Add $CERT_DIR to .gitignore"
echo "  • Never commit certificates or secrets to Git"
echo ""
```

---

## 📄 `scripts/setup/02-setup-github-secrets.sh`

```bash
#!/bin/bash

# ============================================================================
# GITHUB SECRETS CONFIGURATION
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'
BOLD='\033[1m'

CERT_DIR="$HOME/.sfdx-hardis/certs"
SECRETS_FILE="$CERT_DIR/secrets.txt"

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

print_header "🔑 GITHUB SECRETS CONFIGURATION"

# Check if GitHub CLI is installed
if ! command -v gh &> /dev/null; then
    echo -e "${YELLOW}⚠ GitHub CLI not found${NC}"
    echo ""
    echo "You have two options:"
    echo ""
    echo "1. Install GitHub CLI (recommended):"
    echo "   • macOS: brew install gh"
    echo "   • Linux: https://github.com/cli/cli/blob/trunk/docs/install_linux.md"
    echo "   • Windows: https://github.com/cli/cli/releases"
    echo ""
    echo "2. Manual setup (copy/paste secrets)"
    echo ""
    
    if ask_yes_no "Continue with manual setup?" "y"; then
        MANUAL_MODE=true
    else
        echo "Please install GitHub CLI and run again."
        exit 1
    fi
else
    MANUAL_MODE=false
    echo -e "${GREEN}✓ GitHub CLI found${NC}"
    echo ""
    
    # Check if authenticated
    if ! gh auth status &> /dev/null; then
        echo "Please authenticate with GitHub:"
        gh auth login
    fi
    
    # Get repository
    echo "Enter your GitHub repository (format: owner/repo):"
    read -p "Repository: " GITHUB_REPO
    
    echo ""
    echo "Using repository: ${CYAN}$GITHUB_REPO${NC}"
    echo ""
fi

# Load secrets from file
if [ ! -f "$SECRETS_FILE" ]; then
    echo -e "${YELLOW}⚠ Secrets file not found: $SECRETS_FILE${NC}"
    echo "Please run step 01 first."
    exit 1
fi

source "$SECRETS_FILE"

# Function to set secret
set_secret() {
    local secret_name=$1
    local secret_value=$2
    local is_file=${3:-false}
    
    if [ "$MANUAL_MODE" = true ]; then
        echo ""
        echo -e "${CYAN}${BOLD}Secret: $secret_name${NC}"
        if [ "$is_file" = true ]; then
            echo "Value (copy content of file):"
            echo -e "${YELLOW}$(cat "$secret_value")${NC}"
        else
            echo "Value:"
            echo -e "${YELLOW}$secret_value${NC}"
        fi
        echo ""
        echo "Go to: https://github.com/$GITHUB_REPO/settings/secrets/actions/new"
        echo "  • Name: $secret_name"
        echo "  • Value: (paste value above)"
        echo ""
        read -p "Press Enter when done..."
    else
        if [ "$is_file" = true ]; then
            gh secret set "$secret_name" < "$secret_value" -R "$GITHUB_REPO"
        else
            echo "$secret_value" | gh secret set "$secret_name" -R "$GITHUB_REPO"
        fi
        echo -e "${GREEN}✓ Set: $secret_name${NC}"
    fi
}

# Set all secrets
print_header "Setting Salesforce Credentials"

# INT
set_secret "SFDX_CLIENT_ID_INT" "$SFDX_CLIENT_ID_INT"
set_secret "SFDX_CLIENT_KEY_INT" "$SFDX_CLIENT_KEY_INT_PATH" true

# RCT
set_secret "SFDX_CLIENT_ID_RCT" "$SFDX_CLIENT_ID_RCT"
set_secret "SFDX_CLIENT_KEY_RCT" "$SFDX_CLIENT_KEY_RCT_PATH" true

# MAIN
set_secret "SFDX_CLIENT_ID_MAIN" "$SFDX_CLIENT_ID_MAIN"
set_secret "SFDX_CLIENT_KEY_MAIN" "$SFDX_CLIENT_KEY_MAIN_PATH" true

# Technical Org
set_secret "SFDX_AUTH_URL_TECHNICAL_ORG" "$SFDX_AUTH_URL_TECHNICAL_ORG"

print_header "Optional: Notification Secrets"

echo "Do you want to configure Slack notifications?"
if ask_yes_no "Configure Slack?" "n"; then
    read -p "Slack Bot Token: " SLACK_TOKEN
    read -p "Slack Channel ID (default): " SLACK_CHANNEL_ID
    read -p "Slack Channel ID (INT): " SLACK_CHANNEL_ID_INT
    read -p "Slack Channel ID (RCT): " SLACK_CHANNEL_ID_RCT
    read -p "Slack Channel ID (MAIN): " SLACK_CHANNEL_ID_MAIN
    
    set_secret "SLACK_TOKEN" "$SLACK_TOKEN"
    set_secret "SLACK_CHANNEL_ID" "$SLACK_CHANNEL_ID"
    [ -n "$SLACK_CHANNEL_ID_INT" ] && set_secret "SLACK_CHANNEL_ID_INT" "$SLACK_CHANNEL_ID_INT"
    [ -n "$SLACK_CHANNEL_ID_RCT" ] && set_secret "SLACK_CHANNEL_ID_RCT" "$SLACK_CHANNEL_ID_RCT"
    [ -n "$SLACK_CHANNEL_ID_MAIN" ] && set_secret "SLACK_CHANNEL_ID_MAIN" "$SLACK_CHANNEL_ID_MAIN"
fi

echo ""
echo "Do you want to configure JIRA integration?"
if ask_yes_no "Configure JIRA?" "n"; then
    read -p "JIRA Host (https://mycompany.atlassian.net): " JIRA_HOST
    read -p "JIRA Email: " JIRA_EMAIL
    read -p "JIRA API Token: " JIRA_TOKEN
    
    set_secret "JIRA_HOST" "$JIRA_HOST"
    set_secret "JIRA_EMAIL" "$JIRA_EMAIL"
    set_secret "JIRA_TOKEN" "$JIRA_TOKEN"
fi

echo ""
echo -e "${GREEN}${BOLD}✓ GitHub Secrets configured!${NC}"
```

Voulez-vous que je continue avec :
- Les scripts restants (03, 04, 05)
- Le guide étape par étape détaillé avec screenshots textuels
- Les scripts de validation et troubleshooting

Dites-moi ce qui vous intéresse le plus et je continue ! 🚀
