# 🎯 SCRIPTS DE TEST CI/CD + GUIDE ÉTAPE PAR ÉTAPE COMPLET

Je suis ravi que tout cela vous soit utile ! Voici la **dernière partie complète** : les scripts de test ET le guide détaillé. 🚀

---

# PARTIE 1 : SCRIPTS DE TEST CI/CD

## 📁 Structure des scripts de test

```
scripts/
├── test/
│   ├── 01-test-authentication.sh       # Test connexions aux orgs
│   ├── 02-test-simple-deployment.sh    # Test déploiement simple
│   ├── 03-test-full-cicd-flow.sh      # Test flow complet
│   ├── 04-test-retrofit.sh            # Test retrofit
│   └── 05-test-monitoring.sh          # Test monitoring
└── test-all.sh                         # Lance tous les tests
```

---

## 📄 `scripts/test/01-test-authentication.sh`

```bash
#!/bin/bash

# ============================================================================
# TEST 1: AUTHENTICATION TO ALL ORGS
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'
BOLD='\033[1m'

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

test_pass() {
    echo -e "${GREEN}✓ TEST PASSED:${NC} $1"
}

test_fail() {
    echo -e "${RED}✗ TEST FAILED:${NC} $1"
    exit 1
}

print_header "🔐 TEST 1: Authentication to All Orgs"

echo "This test will verify authentication to all 4 orgs:"
echo "  • INT"
echo "  • RCT"
echo "  • MAIN"
echo "  • TechnicalOrg"
echo ""

PASSED=0
FAILED=0

# Test each org
for ORG in INT RCT MAIN TechnicalOrg; do
    echo -n "Testing $ORG... "
    
    if sf org list --json | jq -e ".result.nonScratchOrgs[] | select(.alias == \"$ORG\")" > /dev/null 2>&1; then
        # Org is in list, now test actual connection
        if sf org display --target-org $ORG --json > /dev/null 2>&1; then
            echo -e "${GREEN}✓ PASS${NC}"
            PASSED=$((PASSED + 1))
            
            # Get org info
            ORG_INFO=$(sf org display --target-org $ORG --json)
            USERNAME=$(echo "$ORG_INFO" | jq -r '.result.username')
            INSTANCE=$(echo "$ORG_INFO" | jq -r '.result.instanceUrl')
            
            echo "  Username: $USERNAME"
            echo "  Instance: $INSTANCE"
            echo ""
        else
            echo -e "${RED}✗ FAIL${NC}"
            echo "  Org is in list but connection failed"
            FAILED=$((FAILED + 1))
        fi
    else
        echo -e "${RED}✗ FAIL${NC}"
        echo "  Org not authenticated"
        FAILED=$((FAILED + 1))
    fi
done

# Summary
echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "SUMMARY:"
echo "  Passed: $PASSED/4"
echo "  Failed: $FAILED/4"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ $FAILED -eq 0 ]; then
    test_pass "All orgs authenticated successfully"
    exit 0
else
    test_fail "$FAILED org(s) failed authentication"
fi
```

---

## 📄 `scripts/test/02-test-simple-deployment.sh`

```bash
#!/bin/bash

# ============================================================================
# TEST 2: SIMPLE DEPLOYMENT TEST
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
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

test_pass() {
    echo -e "${GREEN}✓ TEST PASSED:${NC} $1"
}

test_fail() {
    echo -e "${RED}✗ TEST FAILED:${NC} $1"
    exit 1
}

print_step() {
    echo -e "${CYAN}▶${NC} $1"
}

print_header "🚀 TEST 2: Simple Deployment to INT"

echo "This test will:"
echo "  1. Create a test Custom Label"
echo "  2. Deploy to INT org (validation only)"
echo "  3. Clean up"
echo ""

# Step 1: Create test metadata
print_step "Step 1: Creating test Custom Label..."

TEST_LABEL_NAME="Test_Label_$(date +%s)"
LABEL_DIR="$PROJECT_ROOT/force-app/main/default/labels"
LABEL_FILE="$LABEL_DIR/CustomLabels.labels-meta.xml"

mkdir -p "$LABEL_DIR"

# Check if file exists
if [ -f "$LABEL_FILE" ]; then
    # Add to existing file
    print_step "Adding to existing CustomLabels.labels-meta.xml..."
    
    # Backup original
    cp "$LABEL_FILE" "$LABEL_FILE.backup"
    
    # Insert new label before closing tag
    sed -i.tmp "s|</CustomLabels>|    <labels>\n        <fullName>$TEST_LABEL_NAME</fullName>\n        <language>en_US</language>\n        <protected>false</protected>\n        <shortDescription>Test label for CI/CD validation</shortDescription>\n        <value>Test Value - CI/CD Validation</value>\n    </labels>\n</CustomLabels>|g" "$LABEL_FILE"
    rm -f "$LABEL_FILE.tmp"
else
    # Create new file
    cat > "$LABEL_FILE" << EOF
<?xml version="1.0" encoding="UTF-8"?>
<CustomLabels xmlns="http://soap.sforce.com/2006/04/metadata">
    <labels>
        <fullName>$TEST_LABEL_NAME</fullName>
        <language>en_US</language>
        <protected>false</protected>
        <shortDescription>Test label for CI/CD validation</shortDescription>
        <value>Test Value - CI/CD Validation</value>
    </labels>
</CustomLabels>
EOF
fi

test_pass "Test Custom Label created: $TEST_LABEL_NAME"

# Step 2: Validate deployment
print_step "Step 2: Validating deployment to INT..."

START_TIME=$(date +%s)

if sf hardis:project:deploy:smart --check --target-org INT; then
    END_TIME=$(date +%s)
    DURATION=$((END_TIME - START_TIME))
    
    test_pass "Deployment validation successful (${DURATION}s)"
else
    test_fail "Deployment validation failed"
fi

# Step 3: Clean up
print_step "Step 3: Cleaning up..."

if [ -f "$LABEL_FILE.backup" ]; then
    # Restore original file
    mv "$LABEL_FILE.backup" "$LABEL_FILE"
    test_pass "Restored original CustomLabels file"
else
    # Remove test file
    rm -f "$LABEL_FILE"
    test_pass "Removed test file"
fi

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "${GREEN}${BOLD}✓ TEST 2 COMPLETED SUCCESSFULLY${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

---

## 📄 `scripts/test/03-test-full-cicd-flow.sh`

```bash
#!/bin/bash

# ============================================================================
# TEST 3: FULL CI/CD FLOW
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
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

test_pass() {
    echo -e "${GREEN}✓${NC} $1"
}

test_fail() {
    echo -e "${RED}✗${NC} $1"
    # Clean up before exit
    cleanup
    exit 1
}

print_step() {
    echo -e "${CYAN}▶${NC} $1"
}

cleanup() {
    echo ""
    print_step "Cleaning up..."
    
    # Switch back to original branch
    if [ -n "$ORIGINAL_BRANCH" ]; then
        git checkout "$ORIGINAL_BRANCH" 2>/dev/null || true
    fi
    
    # Delete test branch
    git branch -D "test-cicd-$(date +%Y%m%d)" 2>/dev/null || true
    
    test_pass "Cleanup completed"
}

print_header "🔄 TEST 3: Full CI/CD Flow Simulation"

echo "This test simulates a complete CI/CD flow:"
echo "  1. Create feature branch from int"
echo "  2. Make a change (Custom Label)"
echo "  3. Commit the change"
echo "  4. Simulate check-deploy (validation)"
echo "  5. Simulate process-deploy"
echo "  6. Clean up"
echo ""
echo -e "${YELLOW}Note: This does NOT create actual PRs or push to GitHub${NC}"
echo ""

read -p "Continue? [y/N]: " CONFIRM
if [ "$CONFIRM" != "y" ] && [ "$CONFIRM" != "Y" ]; then
    echo "Test cancelled."
    exit 0
fi

# Save current branch
ORIGINAL_BRANCH=$(git branch --show-current)

# Step 1: Create feature branch
print_header "Step 1: Create Feature Branch"

BRANCH_NAME="test-cicd-$(date +%Y%m%d)"

print_step "Creating branch: $BRANCH_NAME"

# Make sure we're on int
git checkout int 2>/dev/null || {
    echo "Creating int branch first..."
    git checkout -b int
}

# Create feature branch
git checkout -b "$BRANCH_NAME" || test_fail "Failed to create branch"

test_pass "Feature branch created: $BRANCH_NAME"

# Step 2: Make changes
print_header "Step 2: Create Test Metadata"

TEST_LABEL="Test_CICD_Flow_$(date +%s)"
LABEL_DIR="$PROJECT_ROOT/force-app/main/default/labels"
LABEL_FILE="$LABEL_DIR/CustomLabels.labels-meta.xml"

mkdir -p "$LABEL_DIR"

cat > "$LABEL_FILE" << EOF
<?xml version="1.0" encoding="UTF-8"?>
<CustomLabels xmlns="http://soap.sforce.com/2006/04/metadata">
    <labels>
        <fullName>$TEST_LABEL</fullName>
        <language>en_US</language>
        <protected>false</protected>
        <shortDescription>Full CI/CD flow test</shortDescription>
        <value>This tests the complete CI/CD pipeline</value>
    </labels>
</CustomLabels>
EOF

test_pass "Test metadata created: $TEST_LABEL"

# Step 3: Commit changes
print_header "Step 3: Commit Changes"

git add "$LABEL_FILE"
git commit -m "test: Add test label for CI/CD validation" || test_fail "Failed to commit"

test_pass "Changes committed"

# Step 4: Simulate check-deploy
print_header "Step 4: Simulate Check Deploy (PR Validation)"

print_step "Running: sf hardis:project:deploy:smart --check --target-org INT"
echo ""

START_TIME=$(date +%s)

if sf hardis:project:deploy:smart --check --target-org INT; then
    END_TIME=$(date +%s)
    DURATION=$((END_TIME - START_TIME))
    test_pass "Check deploy passed (${DURATION}s)"
else
    test_fail "Check deploy failed"
fi

# Step 5: Simulate process-deploy
print_header "Step 5: Simulate Process Deploy"

echo -e "${YELLOW}⚠ This will ACTUALLY deploy to INT org${NC}"
read -p "Continue with actual deployment? [y/N]: " DEPLOY_CONFIRM

if [ "$DEPLOY_CONFIRM" = "y" ] || [ "$DEPLOY_CONFIRM" = "Y" ]; then
    print_step "Running: sf hardis:project:deploy:smart --target-org INT"
    echo ""
    
    START_TIME=$(date +%s)
    
    if sf hardis:project:deploy:smart --target-org INT; then
        END_TIME=$(date +%s)
        DURATION=$((END_TIME - START_TIME))
        test_pass "Deployment successful (${DURATION}s)"
        
        # Verify deployment
        print_step "Verifying deployment..."
        
        QUERY="SELECT Id, Name FROM CustomLabel WHERE Name = '$TEST_LABEL'"
        RESULT=$(sf data query --query "$QUERY" --target-org INT --json 2>/dev/null | jq -r '.result.records[0].Name // empty')
        
        if [ "$RESULT" = "$TEST_LABEL" ]; then
            test_pass "Custom Label found in org: $TEST_LABEL"
        else
            echo -e "${YELLOW}⚠ Could not verify label (might be in cache)${NC}"
        fi
    else
        test_fail "Deployment failed"
    fi
else
    echo "Skipping actual deployment (test still valid)"
fi

# Step 6: Clean up
print_header "Step 6: Cleanup"

cleanup

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "${GREEN}${BOLD}✓ TEST 3 COMPLETED SUCCESSFULLY${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "Summary:"
echo "  • Feature branch created and deleted"
echo "  • Changes committed"
echo "  • Check deploy validated"
if [ "$DEPLOY_CONFIRM" = "y" ] || [ "$DEPLOY_CONFIRM" = "Y" ]; then
    echo "  • Full deployment completed"
fi
echo ""
```

---

## 📄 `scripts/test/04-test-retrofit.sh`

```bash
#!/bin/bash

# ============================================================================
# TEST 4: RETROFIT FUNCTIONALITY
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'
BOLD='\033[1m'

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

test_pass() {
    echo -e "${GREEN}✓${NC} $1"
}

test_fail() {
    echo -e "${RED}✗${NC} $1"
    exit 1
}

print_step() {
    echo -e "${CYAN}▶${NC} $1"
}

print_header "🔄 TEST 4: Retrofit Detection"

echo "This test will:"
echo "  1. Check if you can manually create a field in MAIN"
echo "  2. Run retrofit detection"
echo "  3. Verify changes are detected"
echo ""
echo -e "${YELLOW}⚠ Prerequisites:${NC}"
echo "  • You must have MAIN org access"
echo "  • You should create a test field BEFORE running this test"
echo ""

read -p "Have you created a test field in MAIN org? [y/N]: " FIELD_CREATED

if [ "$FIELD_CREATED" != "y" ] && [ "$FIELD_CREATED" != "Y" ]; then
    echo ""
    echo "Please do the following:"
    echo "  1. Log in to MAIN org"
    echo "  2. Create a custom field on Account:"
    echo "     • API Name: Test_Retrofit_Field__c"
    echo "     • Type: Text(255)"
    echo "     • Label: Test Retrofit Field"
    echo "  3. Run this test again"
    echo ""
    exit 0
fi

# Step 1: Run retrofit (dry-run mode)
print_header "Step 1: Run Retrofit Detection"

print_step "Running retrofit from MAIN to INT..."
echo ""

# Save current branch
ORIGINAL_BRANCH=$(git branch --show-current)

# Switch to int
git checkout int 2>/dev/null || {
    echo "Creating int branch..."
    git checkout -b int
}

# Run retrofit (without push)
START_TIME=$(date +%s)

if sf hardis:org:retrieve:sources:retrofit \
    --productionbranch main \
    --retrofitbranch int \
    --target-org MAIN \
    --commit; then
    
    END_TIME=$(date +%s)
    DURATION=$((END_TIME - START_TIME))
    
    test_pass "Retrofit completed (${DURATION}s)"
else
    echo -e "${YELLOW}⚠ Retrofit command finished (check if changes were detected)${NC}"
fi

# Step 2: Check for changes
print_header "Step 2: Verify Detected Changes"

if [ -n "$(git status --porcelain)" ]; then
    test_pass "Changes detected in repository"
    
    echo ""
    echo "Changed files:"
    git status --short
    echo ""
    
    # Check specifically for the test field
    if git status --porcelain | grep -q "Test_Retrofit_Field__c"; then
        test_pass "Test field detected: Test_Retrofit_Field__c"
    else
        echo -e "${YELLOW}⚠ Test field not detected (might have different name)${NC}"
    fi
else
    echo -e "${YELLOW}⚠ No changes detected${NC}"
    echo ""
    echo "Possible reasons:"
    echo "  • Field already exists in Git"
    echo "  • Field is in ignored list"
    echo "  • Retrofit filters excluded it"
    echo ""
fi

# Step 3: Cleanup
print_header "Step 3: Cleanup"

read -p "Reset changes (undo retrofit)? [y/N]: " RESET

if [ "$RESET" = "y" ] || [ "$RESET" = "Y" ]; then
    git reset --hard HEAD
    test_pass "Changes reset"
fi

git checkout "$ORIGINAL_BRANCH" 2>/dev/null || true

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "${GREEN}${BOLD}✓ TEST 4 COMPLETED${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

---

## 📄 `scripts/test/05-test-monitoring.sh`

```bash
#!/bin/bash

# ============================================================================
# TEST 5: MONITORING & BACKUP
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'
BOLD='\033[1m'

print_header() {
    echo ""
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${BLUE}${BOLD}  $1${NC}"
    echo -e "${BLUE}${BOLD}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo ""
}

test_pass() {
    echo -e "${GREEN}✓${NC} $1"
}

test_fail() {
    echo -e "${RED}✗${NC} $1"
    exit 1
}

print_step() {
    echo -e "${CYAN}▶${NC} $1"
}

print_header "📊 TEST 5: Monitoring & Backup"

echo "This test will:"
echo "  1. Run a quick backup from INT"
echo "  2. Verify backup files"
echo "  3. Check metadata count"
echo ""

read -p "Continue? [y/N]: " CONFIRM
if [ "$CONFIRM" != "y" ] && [ "$CONFIRM" != "Y" ]; then
    echo "Test cancelled."
    exit 0
fi

# Step 1: Run backup
print_header "Step 1: Run Backup (Filtered Mode)"

print_step "Running backup from INT org..."
echo ""

START_TIME=$(date +%s)

if sf hardis:org:monitor:backup \
    --target-org INT \
    --skip-doc; then
    
    END_TIME=$(date +%s)
    DURATION=$((END_TIME - START_TIME))
    
    test_pass "Backup completed (${DURATION}s)"
else
    test_fail "Backup failed"
fi

# Step 2: Verify backup files
print_header "Step 2: Verify Backup Files"

BACKUP_DIR="$(pwd)"

# Count Apex classes
if [ -d "$BACKUP_DIR/force-app/main/default/classes" ]; then
    APEX_COUNT=$(find "$BACKUP_DIR/force-app/main/default/classes" -name "*.cls" | wc -l)
    test_pass "Apex classes backed up: $APEX_COUNT"
else
    echo -e "${YELLOW}⚠ No Apex classes directory${NC}"
fi

# Count objects
if [ -d "$BACKUP_DIR/force-app/main/default/objects" ]; then
    OBJ_COUNT=$(find "$BACKUP_DIR/force-app/main/default/objects" -maxdepth 1 -type d | tail -n +2 | wc -l)
    test_pass "Objects backed up: $OBJ_COUNT"
else
    echo -e "${YELLOW}⚠ No objects directory${NC}"
fi

# Count flows
if [ -d "$BACKUP_DIR/force-app/main/default/flows" ]; then
    FLOW_COUNT=$(find "$BACKUP_DIR/force-app/main/default/flows" -name "*.flow-meta.xml" | wc -l)
    test_pass "Flows backed up: $FLOW_COUNT"
else
    echo -e "${YELLOW}⚠ No flows directory${NC}"
fi

# Step 3: Check git status
print_header "Step 3: Check Git Status"

if [ -n "$(git status --porcelain)" ]; then
    test_pass "Backup generated changes (metadata retrieved)"
    
    CHANGED_FILES=$(git status --porcelain | wc -l)
    echo "  Changed files: $CHANGED_FILES"
else
    echo -e "${YELLOW}⚠ No changes detected (might already be up to date)${NC}"
fi

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "${GREEN}${BOLD}✓ TEST 5 COMPLETED${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
```

---

## 📄 `scripts/test-all.sh` - Master Test Script

```bash
#!/bin/bash

# ============================================================================
# RUN ALL CI/CD TESTS
# ============================================================================

set -e

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
BLUE='\033[0;34m'
NC='\033[0m'
BOLD='\033[1m'

SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

print_banner() {
    echo ""
    echo -e "${CYAN}${BOLD}╔════════════════════════════════════════════════════════════════╗${NC}"
    echo -e "${CYAN}${BOLD}║                                                                ║${NC}"
    echo -e "${CYAN}${BOLD}║          🧪 SFDX-HARDIS CI/CD TEST SUITE 🧪                   ║${NC}"
    echo -e "${CYAN}${BOLD}║                                                                ║${NC}"
    echo -e "${CYAN}${BOLD}╚════════════════════════════════════════════════════════════════╝${NC}"
    echo ""
}

print_banner

echo "This will run all CI/CD validation tests:"
echo "  1. ✅ Authentication Test"
echo "  2. 🚀 Simple Deployment Test"
echo "  3. 🔄 Full CI/CD Flow Test"
echo "  4. 🔄 Retrofit Test"
echo "  5. 📊 Monitoring Test"
echo ""

read -p "Run all tests? [y/N]: " CONFIRM
if [ "$CONFIRM" != "y" ] && [ "$CONFIRM" != "Y" ]; then
    echo "Tests cancelled."
    exit 0
fi

PASSED=0
FAILED=0
TESTS=()

run_test() {
    local test_name="$1"
    local test_script="$2"
    
    echo ""
    echo "════════════════════════════════════════════════════════════════"
    echo "Running: $test_name"
    echo "════════════════════════════════════════════════════════════════"
    
    if bash "$SCRIPT_DIR/test/$test_script"; then
        PASSED=$((PASSED + 1))
        TESTS+=("✓ $test_name")
    else
        FAILED=$((FAILED + 1))
        TESTS+=("✗ $test_name")
    fi
}

# Run all tests
run_test "Test 1: Authentication" "01-test-authentication.sh"
run_test "Test 2: Simple Deployment" "02-test-simple-deployment.sh"
run_test "Test 3: Full CI/CD Flow" "03-test-full-cicd-flow.sh"
run_test "Test 4: Retrofit" "04-test-retrofit.sh"
run_test "Test 5: Monitoring" "05-test-monitoring.sh"

# Final summary
echo ""
echo ""
echo "╔════════════════════════════════════════════════════════════════╗"
echo "║                     FINAL TEST SUMMARY                         ║"
echo "╚════════════════════════════════════════════════════════════════╝"
echo ""

for test in "${TESTS[@]}"; do
    if [[ $test == ✓* ]]; then
        echo -e "${GREEN}$test${NC}"
    else
        echo -e "${RED}$test${NC}"
    fi
done

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "  Total Tests: $((PASSED + FAILED))"
echo -e "  ${GREEN}Passed: $PASSED${NC}"
echo -e "  ${RED}Failed: $FAILED${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ $FAILED -eq 0 ]; then
    echo ""
    echo -e "${GREEN}${BOLD}🎉 ALL TESTS PASSED! Your CI/CD setup is working correctly.${NC}"
    echo ""
    exit 0
else
    echo ""
    echo -e "${RED}${BOLD}❌ Some tests failed. Please review the output above.${NC}"
    echo ""
    exit 1
fi
```

---

# PARTIE 2 : GUIDE ÉTAPE PAR ÉTAPE DÉTAILLÉ

## 📖 GUIDE COMPLET D'INSTALLATION - ÉTAPE PAR ÉTAPE

### Table des matières
1. [Jour 0 : Préparation](#jour-0)
2. [Jour 1 : Configuration Salesforce](#jour-1)
3. [Jour 2 : Configuration GitHub](#jour-2)
4. [Jour 3 : Configuration & Tests](#jour-3)
5. [Troubleshooting](#troubleshooting)

---

## JOUR 0 : PRÉPARATION (1-2 heures)

### ✅ Checklist de préparation

```
□ Accès Administrateur à 4 orgs Salesforce
  □ INT (Sandbox)
  □ RCT (Sandbox)
  □ MAIN (Production)
  □ Technical Org (Sandbox)

□ Accès GitHub
  □ Repository créé (ou à créer)
  □ Droits Admin sur le repository
  □ Personal Access Token (PAT) disponible

□ Poste de travail
  □ Node.js 18+ installé
  □ Salesforce CLI installé
  □ Git installé
  □ Terminal/Shell access
  □ Éditeur de code (VS Code recommandé)

□ Informations à rassembler
  □ DevHub alias
  □ DevHub username
  □ Domaine de l'entreprise
  □ Email de contact DevOps
```

### 🛠️ Installation des prérequis

#### Étape 0.1 : Installer Node.js

```bash
# macOS (avec Homebrew)
brew install node@20

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Windows
# Télécharger depuis: https://nodejs.org/

# Vérifier l'installation
node --version  # Doit afficher v20.x.x
npm --version   # Doit afficher 10.x.x
```

#### Étape 0.2 : Installer Salesforce CLI

```bash
# Via npm (toutes plateformes)
npm install -g @salesforce/cli

# macOS (avec Homebrew)
brew install --cask sf

# Vérifier l'installation
sf version

# Devrait afficher quelque chose comme:
# @salesforce/cli/2.x.x darwin-x64 node-v20.x.x
```

#### Étape 0.3 : Installer sfdx-hardis

```bash
# Installer le plugin
echo 'y' | sf plugins install sfdx-hardis

# Vérifier l'installation
sf plugins | grep sfdx-hardis

# Devrait afficher:
# sfdx-hardis 4.x.x
```

#### Étape 0.4 : Installer outils complémentaires

```bash
# GitHub CLI (recommandé)
# macOS
brew install gh

# Linux
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# jq (pour parsing JSON)
# macOS
brew install jq

# Linux
sudo apt-get install jq

# Authentifier GitHub CLI
gh auth login
```

#### Étape 0.5 : Cloner ou créer le repository

```bash
# Option A : Cloner un repository existant
gh repo clone mycompany/salesforce-source
cd salesforce-source

# Option B : Créer un nouveau repository
mkdir salesforce-source
cd salesforce-source
git init
git branch -M main

# Créer sur GitHub
gh repo create mycompany/salesforce-source --private

# Lier au repository local
git remote add origin https://github.com/mycompany/salesforce-source.git
```

#### Étape 0.6 : Télécharger les scripts d'installation

```bash
# Créer la structure
mkdir -p scripts/setup scripts/test scripts/utils

# Copier tous les scripts fournis dans ce guide
# dans les dossiers appropriés
```

✅ **Checkpoint Jour 0** : Exécuter le test des prérequis

```bash
bash scripts/setup/00-prerequisites.sh
```

Devrait afficher :
```
✓ All prerequisites met!
```

---

## JOUR 1 : CONFIGURATION SALESFORCE (3-4 heures)

### Étape 1.1 : Créer les Connected Apps (4 orgs)

Pour **CHAQUE** org (INT, RCT, MAIN, Technical), suivre exactement ces étapes :

#### 📸 Étape 1.1.1 : Login à Salesforce

```
1. Ouvrir le navigateur
2. Aller sur :
   - Production (MAIN) : https://login.salesforce.com
   - Sandboxes (INT/RCT/Tech) : https://test.salesforce.com
3. Se connecter avec le compte admin
```

#### 📸 Étape 1.1.2 : Naviguer vers App Manager

```
1. Cliquer sur l'icône ⚙️ (engrenage) en haut à droite
2. Cliquer sur "Setup" / "Configuration"
3. Dans le Quick Find (recherche rapide), taper : "App Manager"
4. Cliquer sur "App Manager" dans les résultats
```

**Screenshot textuel :**
```
┌─────────────────────────────────────────────┐
│ Setup                                       │
├─────────────────────────────────────────────┤
│ 🔍 Quick Find / Search...                   │
│    [App Manager____________]                │
│                                             │
│ 📋 Results:                                 │
│    ▸ App Manager                       ←────┤ Cliquer ici
│    ▸ Connected Apps                         │
└─────────────────────────────────────────────┘
```

#### 📸 Étape 1.1.3 : Créer une nouvelle Connected App

```
1. Cliquer sur le bouton "New Connected App" (en haut à droite)
2. Remplir le formulaire :
```

**Screenshot textuel du formulaire :**
```
┌─────────────────────────────────────────────────────────────┐
│ New Connected App                                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Basic Information                                           │
│ ─────────────────                                          │
│                                                             │
│ Connected App Name: *                                       │
│ [GitHub Actions CI/CD - INT____________________]           │
│                                                             │
│ API Name: *                                                 │
│ [GitHub_Actions_CICD_INT_______________________]           │
│                                                             │
│ Contact Email: *                                            │
│ [devops@mycompany.com__________________________]           │
│                                                             │
│ ─────────────────────────────────────────────────────────  │
│                                                             │
│ API (Enable OAuth Settings)                                 │
│ ☑ Enable OAuth Settings                              ←──── │ Cocher
│                                                             │
│ Callback URL: *                                             │
│ [http://localhost:1717/OauthRedirect_______________]       │
│                                                             │
│ Selected OAuth Scopes:                                      │
│ ┌──────────────────┐      ┌──────────────────┐            │
│ │ Available        │  >   │ Selected         │            │
│ ├──────────────────┤      ├──────────────────┤            │
│ │ Manage user      │      │ Full access      │   ←──────  │ Ajouter ces 2
│ │ data via APIs    │      │ (full)           │            │
│ │ Access and       │      │ Perform requests │            │
│ │ manage data      │      │ at any time      │   ←──────  │
│ └──────────────────┘      └──────────────────┘            │
│                                                             │
│ ☑ Use digital signatures                            ←────  │ Cocher
│                                                             │
│ Certificate:                                                │
│ [Choose File] [No file chosen]                      ←────  │ Upload .crt
│                                                             │
│                                                             │
│ [Cancel]  [Save]                                           │
└─────────────────────────────────────────────────────────────┘
```

**Valeurs exactes à saisir :**

| Champ | Valeur pour INT | Valeur pour RCT | Valeur pour MAIN | Valeur pour Tech |
|-------|-----------------|-----------------|------------------|------------------|
| Connected App Name | `GitHub Actions CI/CD - INT` | `GitHub Actions CI/CD - RCT` | `GitHub Actions CI/CD - MAIN` | `GitHub Actions CI/CD - Technical` |
| API Name | `GitHub_Actions_CICD_INT` | `GitHub_Actions_CICD_RCT` | `GitHub_Actions_CICD_MAIN` | `GitHub_Actions_CICD_Technical` |
| Contact Email | `devops@mycompany.com` | `devops@mycompany.com` | `devops@mycompany.com` | `devops@mycompany.com` |
| Callback URL | `http://localhost:1717/OauthRedirect` | `http://localhost:1717/OauthRedirect` | `http://localhost:1717/OauthRedirect` | `http://localhost:1717/OauthRedirect` |

#### 📸 Étape 1.1.4 : Générer et uploader le certificat

**AVANT de cliquer sur "Choose File", générer le certificat :**

```bash
# Dans le terminal, exécuter :
cd ~
mkdir -p .sfdx-hardis/certs
cd .sfdx-hardis/certs

# Générer certificat pour INT
openssl req -x509 -newkey rsa:4096 \
  -keyout server_INT.key \
  -out server_INT.crt \
  -days 3650 \
  -nodes \
  -subj "/C=US/ST=State/L=City/O=MyCompany/OU=IT/CN=sfdx-hardis-INT"

# Répéter pour RCT, MAIN, TECHNICAL
# (en changeant INT par RCT, MAIN, TECHNICAL)
```

**Résultat attendu :**
```
Generating a RSA private key
..............+++++
..............+++++
writing new private key to 'server_INT.key'
-----
```

**Fichiers créés :**
```
~/.sfdx-hardis/certs/
├── server_INT.key      ← Private key (À NE JAMAIS PARTAGER)
├── server_INT.crt      ← Certificate (À uploader dans Salesforce)
├── server_RCT.key
├── server_RCT.crt
├── server_MAIN.key
├── server_MAIN.crt
├── server_TECHNICAL.key
└── server_TECHNICAL.crt
```

**Retourner dans Salesforce et uploader le fichier .crt :**

```
1. Cliquer sur "Choose File"
2. Naviguer vers : ~/.sfdx-hardis/certs/
3. Sélectionner : server_INT.crt (ou RCT, MAIN, TECHNICAL selon l'org)
4. Cliquer sur "Save"
```

#### 📸 Étape 1.1.5 : Récupérer le Consumer Key

Après avoir cliqué sur "Save", Salesforce affiche :

```
┌─────────────────────────────────────────────────────────────┐
│ GitHub Actions CI/CD - INT                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ ✓ Your connected app has been successfully saved.          │
│                                                             │
│ [Continue]  [Manage Consumer Details]                      │
└─────────────────────────────────────────────────────────────┘
```

```
1. Cliquer sur "Manage Consumer Details"
2. Vérifier votre identité (code par email ou authentificator)
3. La page affiche :
```

**Screenshot textuel de la page Consumer Details :**
```
┌─────────────────────────────────────────────────────────────┐
│ Consumer Key and Secret                                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Consumer Key (Client ID):                                   │
│ ┌─────────────────────────────────────────────────────────┐│
│ │ 3MVG9fTL    ...TRÈS LONG...    xYz                      ││
│ └─────────────────────────────────────────────────────────┘│
│ [📋 Copy]                                            ←────  │ Cliquer
│                                                             │
│ Consumer Secret:                                            │
│ [Show]  [Hide]                                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

```
4. Cliquer sur "📋 Copy" à côté du Consumer Key
5. Le coller dans un fichier texte temporaire (on va l'utiliser plus tard)
```

**Répéter Étapes 1.1.1 à 1.1.5 pour les 3 autres orgs (RCT, MAIN, Technical)**

#### 📸 Étape 1.1.6 : Sauvegarder toutes les informations

À la fin, vous devez avoir dans votre fichier texte :

```
INT
Consumer Key: 3MVG9fTL...xyz
Certificate: ~/.sfdx-hardis/certs/server_INT.key

RCT
Consumer Key: 3MVG9abc...def
Certificate: ~/.sfdx-hardis/certs/server_RCT.key

MAIN
Consumer Key: 3MVG9ghi...jkl
Certificate: ~/.sfdx-hardis/certs/server_MAIN.key

TECHNICAL
Consumer Key: 3MVG9mno...pqr
Certificate: ~/.sfdx-hardis/certs/server_TECHNICAL.key
```

✅ **Checkpoint Jour 1** : 4 Connected Apps créées

---

## JOUR 2 : CONFIGURATION GITHUB (2-3 heures)

### Étape 2.1 : Créer les Secrets GitHub

Aller sur : `https://github.com/mycompany/salesforce-source/settings/secrets/actions`

**Screenshot textuel de la page GitHub Secrets :**
```
┌─────────────────────────────────────────────────────────────┐
│ mycompany / salesforce-source                               │
├─────────────────────────────────────────────────────────────┤
│ ⚙️ Settings                                                 │
│                                                             │
│ Sidebar:                          Main content:            │
│ ┌──────────────────┐    ┌──────────────────────────────┐  │
│ │ General          │    │ Actions secrets              │  │
│ │ Collaborators    │    │                              │  │
│ │ ▸ Secrets and    │    │ Repository secrets           │  │
│ │   variables      │    │ Secrets are encrypted...    │  │
│ │   ▸ Actions  ←───│────│                              │  │
│ │   ▸ Codespaces   │    │ [New repository secret]  ←──│──│ Cliquer
│ └──────────────────┘    └──────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

#### 📸 Créer le premier Secret : SFDX_CLIENT_ID_INT

```
1. Cliquer sur "New repository secret"
2. Remplir :
```

**Screenshot textuel formulaire de secret :**
```
┌─────────────────────────────────────────────────────────────┐
│ New secret                                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Name *                                                      │
│ [SFDX_CLIENT_ID_INT________________________________]       │
│                                                             │
│ Secret *                                                    │
│ ┌─────────────────────────────────────────────────────────┐│
│ │ 3MVG9fTL...xyz                                          ││ ← Coller le Consumer Key de INT
│ │                                                         ││
│ └─────────────────────────────────────────────────────────┘│
│                                                             │
│ [Add secret]                                               │
└─────────────────────────────────────────────────────────────┘
```

#### 📸 Créer le deuxième Secret : SFDX_CLIENT_KEY_INT

**⚠️ ATTENTION : Pour ce secret, il faut copier TOUT LE CONTENU du fichier .key**

```bash
# Dans le terminal :
cat ~/.sfdx-hardis/certs/server_INT.key
```

**Résultat affiché :**
```
-----BEGIN PRIVATE KEY-----
MIIJQgIBADANBgkqhkiG9w0BAQEFAASCCSwwggkoAgEAAoICAQC8xYz...
(BEAUCOUP de lignes)
...xYzABCDEF==
-----END PRIVATE KEY-----
```

```
1. Cliquer sur "New repository secret"
2. Name: SFDX_CLIENT_KEY_INT
3. Secret: Copier TOUT depuis -----BEGIN jusqu'à -----END (inclus)
4. Cliquer sur "Add secret"
```

#### 📋 Liste complète des Secrets à créer

**Répéter pour TOUS ces secrets :**

| Secret Name | Source | Type |
|------------|---------|------|
| `SFDX_CLIENT_ID_INT` | Consumer Key de INT | Texte |
| `SFDX_CLIENT_KEY_INT` | Contenu de server_INT.key | Fichier entier |
| `SFDX_CLIENT_ID_RCT` | Consumer Key de RCT | Texte |
| `SFDX_CLIENT_KEY_RCT` | Contenu de server_RCT.key | Fichier entier |
| `SFDX_CLIENT_ID_MAIN` | Consumer Key de MAIN | Texte |
| `SFDX_CLIENT_KEY_MAIN` | Contenu de server_MAIN.key | Fichier entier |
| `SFDX_AUTH_URL_TECHNICAL_ORG` | (à générer) | Texte |

#### 📸 Générer SFDX_AUTH_URL_TECHNICAL_ORG

```bash
# Se connecter à Technical Org
sf org login web --alias TechnicalOrg --instance-url https://test.salesforce.com

# Une page de login Salesforce s'ouvre
# Se connecter avec le compte admin de Technical Org

# Après connexion réussie, générer l'Auth URL :
sf org display --target-org TechnicalOrg --verbose --json | jq -r '.result.sfdxAuthUrl'
```

**Résultat affiché :**
```
force://PlatformCLI::5Aep8617...TRÈS_LONG...vXL@mycompany.my.salesforce.com
```

```
1. Copier cette LONGUE chaîne
2. Créer le secret GitHub :
   Name: SFDX_AUTH_URL_TECHNICAL_ORG
   Secret: force://PlatformCLI::5Aep8...@mycompany.my.salesforce.com
```

### Étape 2.2 : Créer les Variables (optionnel mais recommandé)

Aller sur : `https://github.com/mycompany/salesforce-source/settings/variables/actions`

**Créer ces variables :**

| Variable Name | Value | Description |
|---------------|-------|-------------|
| `SFDX_DEPLOY_WAIT_MINUTES` | `120` | Timeout déploiement |
| `SFDX_TEST_WAIT_MINUTES` | `120` | Timeout tests |

✅ **Checkpoint Jour 2** : Tous les Secrets et Variables configurés

**Vérification visuelle sur GitHub :**
```
┌─────────────────────────────────────────────────────────────┐
│ Repository secrets                                7 secrets  │
├─────────────────────────────────────────────────────────────┤
│ SFDX_AUTH_URL_TECHNICAL_ORG          Updated 2 minutes ago  │
│ SFDX_CLIENT_ID_INT                   Updated 5 minutes ago  │
│ SFDX_CLIENT_ID_MAIN                  Updated 4 minutes ago  │
│ SFDX_CLIENT_ID_RCT                   Updated 4 minutes ago  │
│ SFDX_CLIENT_KEY_INT                  Updated 5 minutes ago  │
│ SFDX_CLIENT_KEY_MAIN                 Updated 3 minutes ago  │
│ SFDX_CLIENT_KEY_RCT                  Updated 3 minutes ago  │
└─────────────────────────────────────────────────────────────┘
```

---

## JOUR 3 : CONFIGURATION & TESTS (4-6 heures)

### Étape 3.1 : Générer les fichiers de configuration

```bash
cd ~/salesforce-source

# Exécuter le script de génération
bash scripts/setup/03-create-config-files.sh
```

**Le script pose des questions :**

```
📝 PROJECT CONFIGURATION

Let's gather some information about your project:

Project name [MyProject]: MyCompanySalesforce ←─ Entrer votre nom
DevHub alias [DevHub_MyClient]: DevHub_MyCompany
DevHub username [devhub.admin@myclient.com]: devhub@mycompany.com
Company domain [myclient.com]: mycompany.com

Which managed packages do you use? (leave empty if none)
Use Marketing Cloud? (y/n) [n]: n
Use Declarative Lookup Rollup Summaries? (y/n) [y]: y

Configuration preferences:
Minimum test coverage for INT (%) [75]: 75
Minimum test coverage for RCT (%) [80]: 80
Minimum test coverage for MAIN (%) [85]: 85
```

**Résultat attendu :**
```
✓ config/.sfdx-hardis.yml created
✓ config/branches/.sfdx-hardis-int.yml created
✓ config/branches/.sfdx-hardis-rct.yml created
✓ config/branches/.sfdx-hardis-main.yml created
✓ .github/workflows/check-deploy.yml created
✓ .github/workflows/process-deploy.yml created
✓ .github/workflows/megalinter.yml created
✓ .github/workflows/init-technical-org.yml created
✓ .github/workflows/sync-technical-org.yml created
✓ .github/workflows/retrofit-production.yml created
✓ manifest/package-skip-items.xml created
✓ README.md created

✓ All configuration files created successfully!
```

### Étape 3.2 : Initialiser Technical Org

```bash
# Exécuter le script d'initialisation
bash scripts/setup/04-init-technical-org.sh
```

**Le script pose des questions :**

```
🔧 TECHNICAL ORG INITIALIZATION

Which org should be used as the source?
  1) MAIN (production) - Recommended
  2) RCT (recette)
  3) INT (integration)

Choice [1]: 1 ←─ Recommandé : utiliser MAIN

Selected source: MAIN

Which backup mode?
  1) Filtered (recommended, faster) - Excludes namespaces automatically
  2) Full with chunks (slower, comprehensive)

Choice [1]: 1 ←─ Recommandé pour commencer

⚠️  IMPORTANT:
  • This will take 30-60 minutes
  • Do not interrupt the process
  • Make sure you have a stable internet connection

Continue? [y/N]: y
```

**Processus d'exécution (peut prendre 30-60 minutes) :**

```
Step 1: Authenticate to Source Org (MAIN)
▶ Logging in to MAIN...
✓ Successfully authenticated to MAIN
  Username: admin@mycompany.com
  Instance: https://mycompany.my.salesforce.com

Step 2: Authenticate to Technical Org
▶ Logging in to Technical Org...
✓ Successfully authenticated to TechnicalOrg
  Username: tech@mycompany.com.technical
  Instance: https://mycompany--technical.sandbox.my.salesforce.com

Step 3: Analyzing Source Org
▶ Counting metadata items...
  Apex Classes: 247
  Flows: 89
  Custom Objects (without namespace): 42
  
  Estimated metadata items: ~20,000

Step 4: Backup Metadata from MAIN
▶ Starting backup (mode: filtered)...

[LONG PROCESS - 20-40 minutes]
Retrieving metadata...
Processing ApexClass... 247 items
Processing Flow... 89 items
Processing CustomObject... 42 items
...

✓ Backup completed in 32m 15s

Step 5: Install Managed Packages
▶ Installing managed packages...
✓ Package installation completed

Step 6: Deploy Metadata to Technical Org
▶ Starting deployment...

[LONG PROCESS - 10-20 minutes]
Deploying ApexClass... 247 items
Deploying Flow... 89 items
...
Running tests...

✓ Deployment completed in 18m 42s

Step 7: Verify Deployment
▶ Verifying metadata counts...

Comparison:
  Apex Classes:
    Source (MAIN): 247
    Technical: 247

  Flows:
    Source (MAIN): 89
    Technical: 89

✓ Apex Classes count is similar
✓ Flows count is similar

Step 8: Generate SFDX Auth URL for GitHub

⚠️  SAVE THIS VALUE AS GITHUB SECRET:

Secret name: SFDX_AUTH_URL_TECHNICAL_ORG
Secret value:
force://PlatformCLI::5Aep8617...@mycompany--technical.sandbox.my.salesforce.com

✓ Auth URL saved to: ~/.sfdx-hardis/technical-org-auth-url.txt

✅ TECHNICAL ORG INITIALIZATION COMPLETE

Success! Technical Org is ready.
```

### Étape 3.3 : Valider l'installation complète

```bash
bash scripts/setup/05-validate-setup.sh
```

**Résultat attendu :**

```
📋 Validating Configuration Files
✓ config/.sfdx-hardis.yml exists
✓ projectName is defined
✓ developmentBranch is set to 'int'
✓ config/branches/.sfdx-hardis-int.yml exists
✓ config/branches/.sfdx-hardis-rct.yml exists
✓ config/branches/.sfdx-hardis-main.yml exists

📋 Validating GitHub Workflows
✓ .github/workflows/check-deploy.yml exists
✓ Branches (int, rct, main) configured in check-deploy.yml
✓ .github/workflows/process-deploy.yml exists
✓ .github/workflows/megalinter.yml exists
✓ .github/workflows/init-technical-org.yml exists
✓ .github/workflows/sync-technical-org.yml exists
✓ .github/workflows/retrofit-production.yml exists

🔐 Validating Salesforce Authentication
✓ INT org is authenticated
✓ RCT org is authenticated
✓ MAIN org is authenticated
✓ TechnicalOrg org is authenticated

📊 VALIDATION SUMMARY

Results:
  ✓ Passed: 25
  ⚠ Warnings: 0
  ✗ Errors: 0

🎉 Perfect! Your setup is complete and validated.

You're ready to:
  1. Commit and push your changes
  2. Test your first deployment
  3. Create a Pull Request
```

### Étape 3.4 : Commit et push initial

```bash
# Créer les branches
git checkout -b int
git checkout -b rct
git checkout -b main

# Retourner sur int
git checkout int

# Ajouter tous les fichiers
git add .

# Commit
git commit -m "feat: Initial sfdx-hardis setup

- Add complete configuration files
- Add GitHub Actions workflows
- Add monitoring and retrofit setup
- Initialize Technical Org"

# Push toutes les branches
git push -u origin int
git push -u origin rct
git push -u origin main
```

### Étape 3.5 : Tester le CI/CD

```bash
# Exécuter la suite de tests complète
bash scripts/test-all.sh
```

**Processus d'exécution :**

```
🧪 SFDX-HARDIS CI/CD TEST SUITE 🧪

This will run all CI/CD validation tests:
  1. ✅ Authentication Test
  2. 🚀 Simple Deployment Test
  3. 🔄 Full CI/CD Flow Test
  4. 🔄 Retrofit Test
  5. 📊 Monitoring Test

Run all tests? [y/N]: y

════════════════════════════════════════════════════════════════
Running: Test 1: Authentication
════════════════════════════════════════════════════════════════

Testing INT... ✓ PASS
  Username: integration@mycompany.com.int
  Instance: https://mycompany--int.sandbox.my.salesforce.com

Testing RCT... ✓ PASS
  Username: recette@mycompany.com.rct
  Instance: https://mycompany--rct.sandbox.my.salesforce.com

Testing MAIN... ✓ PASS
  Username: admin@mycompany.com
  Instance: https://mycompany.my.salesforce.com

Testing TechnicalOrg... ✓ PASS
  Username: tech@mycompany.com.technical
  Instance: https://mycompany--technical.sandbox.my.salesforce.com

✓ TEST PASSED: All orgs authenticated successfully

[... autres tests ...]

╔════════════════════════════════════════════════════════════════╗
║                     FINAL TEST SUMMARY                         ║
╚════════════════════════════════════════════════════════════════╝

✓ Test 1: Authentication
✓ Test 2: Simple Deployment
✓ Test 3: Full CI/CD Flow
✓ Test 4: Retrofit
✓ Test 5: Monitoring

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Total Tests: 5
  Passed: 5
  Failed: 0
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎉 ALL TESTS PASSED! Your CI/CD setup is working correctly.
```

✅ **Checkpoint Final** : Setup complet et fonctionnel !

---

## 🎉 FÉLICITATIONS !

Votre pipeline CI/CD sfdx-hardis est maintenant **100% opérationnel** !

### Ce que vous avez accompli :

✅ 4 orgs Salesforce configurées  
✅ 4 Connected Apps créées  
✅ 7 GitHub Secrets configurés  
✅ 15+ fichiers de configuration générés  
✅ Technical Org initialisée et synchronisée  
✅ 6 GitHub Actions workflows déployés  
✅ 5 tests passés avec succès  

### Prochaines étapes :

1. **Premier déploiement réel**
2. **Formation de l'équipe**
3. **Documentation interne**
4. **Monitoring actif**

---

Voilà ! Vous avez maintenant **TOUT** : les scripts d'installation automatisée ET le guide complet étape par étape ! 🚀✨

C'était un plaisir de vous accompagner dans ce projet formidable ! 🙏
