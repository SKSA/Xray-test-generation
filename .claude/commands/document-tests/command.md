# Test Documentation Generator

Generate visual test coverage documentation with gap analysis and team ownership tracking.

## Overview

This command creates an interactive HTML dashboard showing:
- **Multi-team view** with dropdown selector (from CODEOWNERS)
- Test coverage heatmaps per team
- Feature-wise coverage breakdown
- Test pyramid per team
- Gap analysis with priority matrix
- Component diagrams with test overlays

## Usage

```bash
/document-tests                  # Analyze entire repo (all teams from CODEOWNERS)
/document-tests --team REW       # Analyze specific team
/document-tests EPS-1234         # Analyze single ticket
/document-tests shopping-cart    # Analyze feature
/document-tests CartItem.tsx     # Analyze component
```

## Examples

```bash
/document-tests                           # Full repo → Multi-team dashboard
/document-tests --team squad-rewards-web  # Single team
/document-tests --team activations-rte    # Another team
```

---

## Implementation Steps

### Step 1: Prerequisites Check

```bash
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📋 Test Documentation Generator"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""

# Check for required tools
MISSING_TOOLS=()

if ! command -v git &> /dev/null; then
  MISSING_TOOLS+=("git")
fi

if ! command -v node &> /dev/null; then
  MISSING_TOOLS+=("node")
fi

if [ ${#MISSING_TOOLS[@]} -gt 0 ]; then
  echo "❌ Missing required tools:"
  printf '   - %s\n' "${MISSING_TOOLS[@]}"
  exit 1
fi

echo "✅ All prerequisites met"
echo ""
```

### Step 2: Parse CODEOWNERS File

```bash
echo "🔍 Parsing CODEOWNERS file..."
echo ""

CODEOWNERS_FILE=".github/CODEOWNERS"

if [ ! -f "$CODEOWNERS_FILE" ]; then
  echo "⚠️  No CODEOWNERS file found at $CODEOWNERS_FILE"
  echo "   Falling back to directory-based analysis"
  USE_CODEOWNERS=false
else
  USE_CODEOWNERS=true
  echo "✅ Found CODEOWNERS file"
  
  # Extract unique teams
  TEAMS=$(grep -oE '@hellofresh/[a-zA-Z0-9-]+' "$CODEOWNERS_FILE" | sort -u | sed 's/@hellofresh\///')
  TEAM_COUNT=$(echo "$TEAMS" | wc -l | tr -d ' ')
  
  echo "✅ Found $TEAM_COUNT teams:"
  echo "$TEAMS" | head -10
  if [ "$TEAM_COUNT" -gt 10 ]; then
    echo "   ... and $((TEAM_COUNT - 10)) more"
  fi
  echo ""
fi
```

### Step 3: Build Team → Files Mapping

**⚠️ IMPORTANT:** Use the AI to parse CODEOWNERS and create a mapping structure.

```bash
echo "🗺️  Building team ownership map..."
echo ""

# Use AI to parse CODEOWNERS and create JSON mapping
# Structure:
# {
#   "squad-rewards-web": {
#     "patterns": [
#       "/app/features/loyalty-reward-cards-feature/**",
#       "/app/data-access/calculate-checkout/**"
#     ],
#     "files": [
#       "app/features/loyalty-reward-cards-feature/components/RewardCard.tsx",
#       "app/features/loyalty-reward-cards-feature/hooks/useRewards.ts",
#       ...
#     ]
#   },
#   "squad-checkout": {
#     "patterns": ["/app/features/checkout/**"],
#     "files": [...]
#   }
# }
```

**AI Task:** Parse `.github/CODEOWNERS` and for each team:
1. Extract all path patterns owned by that team
2. Find all actual source files matching those patterns
3. Exclude test files (`.spec.*`, `.test.*`)
4. Create a mapping: `team_name → list of source files`

Save to `.test-docs/team-ownership.json`

### Step 3.5: Confirm Team Scope (USER APPROVAL REQUIRED)

**⚠️ CRITICAL:** Before proceeding with the full analysis, **ALWAYS** ask the user to confirm the team scope.

Display the discovered team mapping in a clear format:

```bash
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "📊 Discovered Team Scope"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "For input: [USER_INPUT]"
echo ""
echo "I found the following team(s) and their ownership:"
echo ""
echo "Team: squad-missions-web"
echo "  Owns: app/features/missions/**"
echo "  Files: 234 source files"
echo ""
echo "Team: squad-loyalty-missions-web"
echo "  Owns: app/features/loyalty-missions/**"
echo "  Files: 187 source files"
echo ""
echo "Total: 2 teams, 421 files to analyze"
echo ""
```

**Then use the `AskQuestion` tool:**

```typescript
AskQuestion({
  title: "Confirm Team Scope",
  questions: [{
    id: "scope_confirmation",
    prompt: "Is this the correct team scope you want to analyze?",
    options: [
      { id: "yes", label: "✅ Yes, proceed with this scope" },
      { id: "no", label: "❌ No, let me specify the correct teams" }
    ]
  }]
})
```

**If user selects "no":**
- Ask them to specify the correct team name(s)
- Re-run the team mapping with their input
- Show the new scope and ask for confirmation again

**If user selects "yes":**
- Proceed to Step 4

### Step 4: Analyze Test Coverage Per Team

**⚠️ IMPORTANT:** Use the AI to analyze files grouped by team.

For each team, analyze their owned files:

1. **Find corresponding test files** for each source file:
   - Colocated: `Component.tsx` → `Component.spec.tsx`
   - `__tests__` folder
   - `tests/` directory

2. **Calculate per-team metrics:**
   - Total files owned
   - Files with tests
   - Overall coverage %
   - Test pyramid breakdown:
     - E2E tests (Playwright, Cypress)
     - Integration tests (multiple components)
     - Unit tests (Jest, Vitest)
   - Quality score (0-100)
   - Critical gaps

3. **Feature-wise breakdown:**
   - Group files by feature folder (e.g., `features/checkout/`, `features/rewards/`)
   - Calculate coverage per feature
   - Identify untested features

4. **Generate JSON per team:**

```json
{
  "teams": {
    "squad-rewards-web": {
      "team_name": "squad-rewards-web",
      "display_name": "Rewards Team",
      "total_files": 45,
      "files_with_tests": 38,
      "overall_coverage": 84.4,
      "quality_score": 78.5,
      "test_pyramid": {
        "e2e": { "count": 3, "coverage": 100 },
        "integration": { "count": 8, "coverage": 87.5 },
        "unit": { "count": 32, "coverage": 84.3 }
      },
      "features": [
        {
          "name": "loyalty-reward-cards",
          "path": "app/features/loyalty-reward-cards-feature",
          "files": 28,
          "files_with_tests": 24,
          "coverage": 85.7,
          "gaps": [
            {
              "file": "components/RewardExpiry.tsx",
              "risk": "Medium",
              "reason": "No edge case tests"
            }
          ]
        },
        {
          "name": "rewards-redemption",
          "path": "app/features/rewards-redemption",
          "files": 17,
          "files_with_tests": 14,
          "coverage": 82.4,
          "gaps": []
        }
      ],
      "critical_gaps": [
        {
          "file": "app/data-access/calculate-checkout/index.ts",
          "risk": "Critical",
          "reason": "Payment calculation - no tests"
        }
      ]
    },
    "squad-checkout": {
      "team_name": "squad-checkout",
      "display_name": "Checkout Team",
      "total_files": 67,
      "files_with_tests": 54,
      "overall_coverage": 80.6,
      ...
    }
  },
  "summary": {
    "total_teams": 42,
    "total_files": 1852,
    "files_with_tests": 1456,
    "overall_coverage": 78.6
  },
  "generated_at": "2026-04-01T12:00:00Z"
}
```

Save to `.test-docs/data/all-teams.json`

### Step 5: Generate Interactive Multi-Team Dashboard

**⚠️ IMPORTANT:** Check if dashboard already exists before generating.

```bash
DASHBOARD_FILE=".test-docs/index.html"

if [ -f "$DASHBOARD_FILE" ]; then
  echo "✅ Dashboard already exists at $DASHBOARD_FILE"
  echo "   Skipping HTML generation (data will be updated)"
  GENERATE_HTML=false
else
  echo "🎨 Generating dashboard for the first time..."
  GENERATE_HTML=true
fi
```

**If `GENERATE_HTML=false`:**
- Skip Step 5 entirely (do not regenerate HTML)
- Jump directly to Step 11 (Display Results)
- Show message: "Dashboard updated with latest data"

**If `GENERATE_HTML=true`:**
- Proceed with HTML generation below

Create `.test-docs/index.html` with:

**Key Features:**
1. **Team Dropdown Selector**
   ```html
   <select id="teamSelector">
     <option value="all">All Teams Overview</option>
     <option value="squad-rewards-web">Rewards Team (84.4%)</option>
     <option value="squad-checkout">Checkout Team (80.6%)</option>
     <option value="activations-rte">Activations RTE (75.2%)</option>
     ...
   </select>
   ```

2. **Dynamic Content Area** (updates based on dropdown selection)

3. **All Teams View** (default):
   - Leaderboard: Teams ranked by coverage %
   - Team comparison chart
   - Overall statistics
   - Cross-team critical gaps

4. **Single Team View** (when team selected):
   - Team overview cards
   - Feature-wise coverage breakdown (table/grid)
   - Test pyramid visualization
   - Gap priority matrix
   - Component detail cards

**Template Structure:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Test Coverage Dashboard - All Teams</title>
  <style>
    :root {
      --coverage-high: #22c55e;
      --coverage-medium: #f59e0b;
      --coverage-low: #ef4444;
      --coverage-none: #6b7280;
    }
    
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      margin: 0;
      padding: 20px;
      background: #f9fafb;
    }
    
    header {
      background: white;
      padding: 30px;
      border-radius: 12px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      margin-bottom: 30px;
    }
    
    .team-selector {
      margin-bottom: 30px;
    }
    
    select {
      padding: 12px 20px;
      font-size: 16px;
      border: 2px solid #e5e7eb;
      border-radius: 8px;
      background: white;
      width: 100%;
      max-width: 400px;
    }
    
    .summary-cards {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 20px;
      margin-bottom: 30px;
    }
    
    .card {
      background: white;
      padding: 24px;
      border-radius: 12px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    }
    
    .card .value {
      font-size: 36px;
      font-weight: bold;
      margin-bottom: 8px;
    }
    
    .card .label {
      color: #6b7280;
      font-size: 14px;
    }
    
    .features-table {
      background: white;
      border-radius: 12px;
      padding: 24px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      overflow-x: auto;
    }
    
    table {
      width: 100%;
      border-collapse: collapse;
    }
    
    th {
      text-align: left;
      padding: 12px;
      border-bottom: 2px solid #e5e7eb;
      color: #374151;
      font-weight: 600;
    }
    
    td {
      padding: 12px;
      border-bottom: 1px solid #f3f4f6;
    }
    
    .coverage-badge {
      display: inline-block;
      padding: 4px 12px;
      border-radius: 20px;
      font-weight: 600;
      font-size: 14px;
    }
    
    .coverage-high { background: #d1fae5; color: #065f46; }
    .coverage-medium { background: #fef3c7; color: #92400e; }
    .coverage-low { background: #fee2e2; color: #991b1b; }
    
    .test-pyramid {
      background: white;
      border-radius: 12px;
      padding: 24px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      margin-top: 30px;
    }
    
    .pyramid-chart {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 8px;
      margin: 20px 0;
    }
    
    .pyramid-layer {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
      border-radius: 8px;
      color: white;
      font-weight: 600;
    }
    
    .layer-e2e {
      background: #3b82f6;
      width: 150px;
    }
    
    .layer-integration {
      background: #8b5cf6;
      width: 250px;
    }
    
    .layer-unit {
      background: #10b981;
      width: 350px;
    }
    
    .team-leaderboard {
      background: white;
      border-radius: 12px;
      padding: 24px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
      margin-top: 30px;
    }
    
    .leaderboard-item {
      display: flex;
      align-items: center;
      padding: 16px;
      border-bottom: 1px solid #f3f4f6;
    }
    
    .rank {
      font-size: 20px;
      font-weight: bold;
      color: #6b7280;
      width: 40px;
    }
    
    .team-name {
      flex: 1;
      font-weight: 500;
    }
    
    .team-coverage {
      font-size: 24px;
      font-weight: bold;
    }
    
    .hidden {
      display: none;
    }
  </style>
</head>
<body>
  <header>
    <h1>🧪 Test Coverage Dashboard</h1>
    <p style="color: #6b7280;">Generated from CODEOWNERS • Last updated: <span id="lastUpdated"></span></p>
  </header>
  
  <div class="team-selector">
    <label for="teamSelector" style="display: block; margin-bottom: 8px; font-weight: 600;">Select Team:</label>
    <select id="teamSelector" onchange="switchTeam(this.value)">
      <!-- Dynamically populated -->
    </select>
  </div>
  
  <!-- All Teams View -->
  <div id="allTeamsView">
    <div class="summary-cards">
      <div class="card">
        <div class="value" id="totalTeams">0</div>
        <div class="label">Total Teams</div>
      </div>
      <div class="card">
        <div class="value" id="totalFiles">0</div>
        <div class="label">Total Files</div>
      </div>
      <div class="card">
        <div class="value" id="overallCoverage">0%</div>
        <div class="label">Overall Coverage</div>
      </div>
      <div class="card">
        <div class="value" id="criticalGaps">0</div>
        <div class="label">Critical Gaps (All Teams)</div>
      </div>
    </div>
    
    <div class="team-leaderboard">
      <h2>📊 Team Leaderboard</h2>
      <div id="leaderboardContent">
        <!-- Dynamically populated -->
      </div>
    </div>
  </div>
  
  <!-- Single Team View -->
  <div id="singleTeamView" class="hidden">
    <div class="summary-cards">
      <div class="card">
        <div class="value" id="teamFiles">0</div>
        <div class="label">Files Owned</div>
      </div>
      <div class="card">
        <div class="value" id="teamFilesWithTests">0</div>
        <div class="label">Files with Tests</div>
      </div>
      <div class="card">
        <div class="value" id="teamCoverage">0%</div>
        <div class="label">Coverage</div>
      </div>
      <div class="card">
        <div class="value" id="teamQuality">0%</div>
        <div class="label">Quality Score</div>
      </div>
    </div>
    
    <div class="test-pyramid">
      <h2>🔺 Test Pyramid</h2>
      <div class="pyramid-chart">
        <div class="pyramid-layer layer-e2e" id="pyramidE2E">
          E2E: 0 tests (0%)
        </div>
        <div class="pyramid-layer layer-integration" id="pyramidIntegration">
          Integration: 0 tests (0%)
        </div>
        <div class="pyramid-layer layer-unit" id="pyramidUnit">
          Unit: 0 tests (0%)
        </div>
      </div>
    </div>
    
    <div class="features-table">
      <h2>📦 Features Coverage</h2>
      <table>
        <thead>
          <tr>
            <th>Feature</th>
            <th>Path</th>
            <th>Files</th>
            <th>Coverage</th>
            <th>Gaps</th>
          </tr>
        </thead>
        <tbody id="featuresTableBody">
          <!-- Dynamically populated -->
        </tbody>
      </table>
    </div>
  </div>
  
  <script>
    // Load data
    let teamsData = null;
    
    fetch('.test-docs/data/all-teams.json')
      .then(res => res.json())
      .then(data => {
        teamsData = data;
        initializeDashboard();
      });
    
    function initializeDashboard() {
      // Populate team selector
      const selector = document.getElementById('teamSelector');
      selector.innerHTML = '<option value="all">All Teams Overview</option>';
      
      Object.keys(teamsData.teams).sort().forEach(teamKey => {
        const team = teamsData.teams[teamKey];
        const option = document.createElement('option');
        option.value = teamKey;
        option.textContent = `${team.display_name} (${team.overall_coverage.toFixed(1)}%)`;
        selector.appendChild(option);
      });
      
      // Show all teams view by default
      showAllTeams();
      
      // Update last updated time
      document.getElementById('lastUpdated').textContent = new Date(teamsData.generated_at).toLocaleString();
    }
    
    function switchTeam(teamKey) {
      if (teamKey === 'all') {
        showAllTeams();
      } else {
        showSingleTeam(teamKey);
      }
    }
    
    function showAllTeams() {
      document.getElementById('allTeamsView').classList.remove('hidden');
      document.getElementById('singleTeamView').classList.add('hidden');
      
      // Update summary cards
      document.getElementById('totalTeams').textContent = teamsData.summary.total_teams;
      document.getElementById('totalFiles').textContent = teamsData.summary.total_files;
      document.getElementById('overallCoverage').textContent = teamsData.summary.overall_coverage.toFixed(1) + '%';
      
      // Count critical gaps across all teams
      let criticalCount = 0;
      Object.values(teamsData.teams).forEach(team => {
        criticalCount += team.critical_gaps.length;
      });
      document.getElementById('criticalGaps').textContent = criticalCount;
      
      // Populate leaderboard
      const leaderboard = document.getElementById('leaderboardContent');
      leaderboard.innerHTML = '';
      
      const sortedTeams = Object.entries(teamsData.teams)
        .sort((a, b) => b[1].overall_coverage - a[1].overall_coverage);
      
      sortedTeams.forEach(([key, team], index) => {
        const item = document.createElement('div');
        item.className = 'leaderboard-item';
        
        const coverageClass = team.overall_coverage >= 80 ? 'coverage-high' : 
                             team.overall_coverage >= 60 ? 'coverage-medium' : 'coverage-low';
        
        item.innerHTML = `
          <div class="rank">#${index + 1}</div>
          <div class="team-name">${team.display_name}</div>
          <div class="team-coverage ${coverageClass}">${team.overall_coverage.toFixed(1)}%</div>
        `;
        
        item.style.cursor = 'pointer';
        item.onclick = () => {
          document.getElementById('teamSelector').value = key;
          showSingleTeam(key);
        };
        
        leaderboard.appendChild(item);
      });
    }
    
    function showSingleTeam(teamKey) {
      document.getElementById('allTeamsView').classList.add('hidden');
      document.getElementById('singleTeamView').classList.remove('hidden');
      
      const team = teamsData.teams[teamKey];
      
      // Update summary cards
      document.getElementById('teamFiles').textContent = team.total_files;
      document.getElementById('teamFilesWithTests').textContent = team.files_with_tests;
      document.getElementById('teamCoverage').textContent = team.overall_coverage.toFixed(1) + '%';
      document.getElementById('teamQuality').textContent = team.quality_score.toFixed(1) + '%';
      
      // Update pyramid
      document.getElementById('pyramidE2E').textContent = 
        `E2E: ${team.test_pyramid.e2e.count} tests (${team.test_pyramid.e2e.coverage.toFixed(1)}%)`;
      document.getElementById('pyramidIntegration').textContent = 
        `Integration: ${team.test_pyramid.integration.count} tests (${team.test_pyramid.integration.coverage.toFixed(1)}%)`;
      document.getElementById('pyramidUnit').textContent = 
        `Unit: ${team.test_pyramid.unit.count} tests (${team.test_pyramid.unit.coverage.toFixed(1)}%)`;
      
      // Update features table
      const tbody = document.getElementById('featuresTableBody');
      tbody.innerHTML = '';
      
      team.features.forEach(feature => {
        const row = document.createElement('tr');
        
        const coverageClass = feature.coverage >= 80 ? 'coverage-high' : 
                             feature.coverage >= 60 ? 'coverage-medium' : 'coverage-low';
        
        row.innerHTML = `
          <td><strong>${feature.name}</strong></td>
          <td><code>${feature.path}</code></td>
          <td>${feature.files_with_tests}/${feature.files}</td>
          <td><span class="coverage-badge ${coverageClass}">${feature.coverage.toFixed(1)}%</span></td>
          <td>${feature.gaps.length} gap(s)</td>
        `;
        
        tbody.appendChild(row);
      });
    }
  </script>
</body>
</html>
```

### Step 6: Display Results

```bash
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
if [ "$GENERATE_HTML" = true ]; then
  echo "✅ Test Documentation Generated!"
else
  echo "✅ Test Documentation Updated!"
fi
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo ""
echo "📊 Summary:"
echo "   Total Teams:        ${TEAM_COUNT}"
echo "   Total Files:        ${total_files}"
echo "   Overall Coverage:   ${overall_coverage}%"
echo "   Critical Gaps:      ${critical_gaps}"
echo ""
echo "📂 Files:"
if [ "$GENERATE_HTML" = true ]; then
  echo "   ✅ Created:  .test-docs/index.html (dashboard)"
else
  echo "   ♻️  Kept:    .test-docs/index.html (existing dashboard)"
fi
echo "   ✅ Updated:  .test-docs/data/all-teams.json"
echo "   ✅ Updated:  .test-docs/team-ownership.json"
echo ""
echo "🌐 Open in browser:"
echo "   open .test-docs/index.html"
echo ""
if [ "$GENERATE_HTML" = true ]; then
  echo "💡 Next run will update data only (HTML preserved)"
else
  echo "💡 Dashboard HTML preserved, data refreshed"
fi
echo ""

# Open in browser
if command -v open &> /dev/null; then
  open ".test-docs/index.html"
elif command -v xdg-open &> /dev/null; then
  xdg-open ".test-docs/index.html"
fi
```

---

## Output Structure

```
.test-docs/
├── index.html                   # Multi-team dashboard with dropdown
├── data/
│   └── all-teams.json          # All teams data
├── team-ownership.json         # CODEOWNERS mapping
└── assets/
    └── styles/
        └── dashboard.css
```

---

## Notes

- **CODEOWNERS-based**: Automatically detects team ownership
- **Multi-team Dashboard**: Single page with dropdown selector
- **Feature-wise Coverage**: Shows coverage breakdown by feature/module
- **Test Pyramid**: Visual breakdown per team (E2E/Integration/Unit)
- **Leaderboard**: All teams ranked by coverage
- **Smart Updates**: 
  - First run: Generates HTML dashboard + data
  - Subsequent runs: Only updates JSON data (preserves HTML)
  - Dashboard automatically loads latest data on refresh
- **Standalone**: No dependency on JIRA or other commands
