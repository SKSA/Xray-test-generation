# Developer Risk Analysis

Analyzes developer contribution quality by tracking features, bug fixes, and code ownership to calculate risk scores per developer in a squad.

## What It Does

1. **Prompts for Squad Name** - User enters squad to analyze (e.g., "squad-rewards-web")
2. **Analyzes 6 Months of PRs** - Fetches all merged PRs for the squad from last 6 months
3. **Tracks Bug Fixes** - Links bug fixes back to original features and developers
4. **Calculates Risk Scores** - Generates risk scores based on bug-to-feature ratio
5. **Creates Dashboard** - Generates interactive HTML dashboard with developer insights

## How It Works

### Step 1: Get Squad Name or JIRA Project from User

**USER INTERACTION REQUIRED**: Use the `AskQuestion` tool to prompt for either squad name or JIRA project.

```typescript
AskQuestion({
  title: "Developer Risk Analysis",
  questions: [{
    id: "input_type",
    prompt: "What do you want to analyze by?",
    options: [
      { id: "squad", label: "Squad Name (e.g., squad-rewards-web)" },
      { id: "jira", label: "JIRA Project (e.g., MIO, RE, GOC)" }
    ]
  }]
})
```

Then ask for the specific value:

```typescript
// If user selected "squad"
AskQuestion({
  title: "Developer Risk Analysis",
  questions: [{
    id: "squad_name",
    prompt: "Enter the squad name (e.g., squad-rewards-web):",
    options: [
      { id: "manual", label: "I'll type it manually" }
    ]
  }]
})

// If user selected "jira"
AskQuestion({
  title: "Developer Risk Analysis",
  questions: [{
    id: "jira_project",
    prompt: "Enter the JIRA project key (e.g., MIO, RE, GOC, ACCEL):",
    options: [
      { id: "manual", label: "I'll type it manually" }
    ]
  }]
})
```

After receiving the input, store it in variables:
```bash
INPUT_TYPE="<squad|jira>"
ANALYSIS_KEY="<user_provided_value>"

if [ "$INPUT_TYPE" = "squad" ]; then
  SQUAD_NAME="$ANALYSIS_KEY"
  echo "Analyzing by Squad: $SQUAD_NAME"
else
  JIRA_PROJECT="$ANALYSIS_KEY"
  echo "Analyzing by JIRA Project: $JIRA_PROJECT"
fi
```

---

### Step 2: Set Analysis Parameters

```bash
# Create output directory
OUTPUT_DIR=".dev-risk-analysis"
mkdir -p "$OUTPUT_DIR/data"

# Calculate date range (6 months back)
END_DATE=$(date +%Y-%m-%d)
START_DATE=$(date -v-6m +%Y-%m-%d 2>/dev/null || date -d '6 months ago' +%Y-%m-%d)

echo "📊 Developer Risk Analysis"
if [ "$INPUT_TYPE" = "squad" ]; then
  echo "   Squad: $SQUAD_NAME"
else
  echo "   JIRA Project: $JIRA_PROJECT"
fi
echo "   Period: $START_DATE to $END_DATE"
echo ""
```

---

### Step 3: Fetch PRs (Last 6 Months)

Use GitHub CLI to fetch all merged PRs filtered by either squad label OR JIRA project in title.

```bash
if [ "$INPUT_TYPE" = "squad" ]; then
  echo "🔍 Fetching PRs for squad: $SQUAD_NAME..."
  
  # Fetch merged PRs with labels matching squad
  gh pr list \
    --repo $(gh repo view --json nameWithOwner -q .nameWithOwner) \
    --state merged \
    --limit 500 \
    --json number,title,author,mergedAt,labels,files,additions,deletions \
    --search "label:\"$SQUAD_NAME\" merged:>=$START_DATE" \
    > "$OUTPUT_DIR/data/all-prs.json"
    
else
  echo "🔍 Fetching PRs for JIRA project: $JIRA_PROJECT..."
  
  # Fetch merged PRs with titles containing JIRA project key (e.g., [MIO-123])
  gh pr list \
    --repo $(gh repo view --json nameWithOwner -q .nameWithOwner) \
    --state merged \
    --limit 500 \
    --json number,title,author,mergedAt,labels,files,additions,deletions \
    --search "merged:>=$START_DATE" \
    | jq "[.[] | select(.title | test(\"\\\\[$JIRA_PROJECT-[0-9]+\\\\]\"; \"i\"))]" \
    > "$OUTPUT_DIR/data/all-prs.json"
fi

echo "✅ Fetched PRs"
```

---

### Step 4: Analyze PRs and Categorize

**CRITICAL**: You MUST analyze the PR data to:
1. Identify feature PRs vs bug fix PRs
2. Extract author information (use `name` field, fallback to `login`)
3. Link bug fixes to original features via:
   - Same ticket number in title (e.g., [MIO-339])
   - Files changed overlap
   - Time proximity (within 30 days)

**Author Name Extraction:**
- **Primary**: Use `author.name` from GitHub API (e.g., "Philip Tauro")
- **Fallback**: Use `author.login` if name is empty (e.g., "philiptauro")
- **Display Format**: "Philip Tauro (@philiptauro)" in dashboard

**Example:**
```bash
# Extract author info from PR
author_login=$(jq -r '.author.login' pr.json)
author_name=$(jq -r '.author.name' pr.json)

# Use name if available, otherwise use login
if [ -z "$author_name" ] || [ "$author_name" = "null" ]; then
  display_name="$author_login"
else
  display_name="$author_name"
fi
```

**Classification Rules:**
- **Feature PR**: Has label `type: feature` OR branch starts with `feature/`
- **Bug Fix PR**: Has label `type: bugfix`, `type: bug`, OR title contains "fix bug", "bugfix", "hotfix"
- **NOT a bug fix**: `type: patch` alone (unless explicitly says "bug")

**Output Required**: Create `$OUTPUT_DIR/data/${ANALYSIS_KEY}-analyzed-prs.json` (and symlink to `analyzed-prs.json` for backward compatibility).
```json
{
  "features": [
    {
      "pr_number": 67382,
      "title": "[MIO-339] Feature title",
      "author_login": "philiptauro",
      "author_name": "Philip Tauro",
      "co_authors": [],
      "merged_at": "2026-04-01T21:15:29Z",
      "ticket": "MIO-339",
      "files_changed": ["app/features/wallet/component.tsx"],
      "related_bug_fixes": []
    }
  ],
  "bug_fixes": [
    {
      "pr_number": 67400,
      "title": "[MIO-339] Fix validation bug",
      "author_login": "johndoe",
      "author_name": "John Doe",
      "merged_at": "2026-04-05T10:30:00Z",
      "ticket": "MIO-339",
      "files_changed": ["app/features/wallet/component.tsx"],
      "fixes_feature_pr": 67382,
      "severity": "medium"
    }
  ]
}
```

---

### Step 5: Calculate Developer Risk Scores

For each developer who contributed to the squad in the 6-month period:

**Scoring Algorithm:**

```python
# Base score starts at 100 (low risk)
base_score = 100

for developer in developers:
    points = 0
    
    # Count weighted contributions
    features_count = 0
    bugs_caused = 0
    
    for feature in developer.features:
        weight = 1.0 / feature.num_authors  # Split credit
        features_count += weight
        points += 10 * weight  # +10 points per feature
        
        # Check if this feature had bug fixes
        for bug in feature.related_bug_fixes:
            bugs_caused += weight
            
            if bug.severity == "critical":
                points -= 15 * weight  # -15 for critical bugs
            else:
                points -= 8 * weight   # -8 for normal bugs
    
    # Calculate final score (0-100 scale)
    if features_count > 0:
        raw_score = base_score + (points / features_count)
        risk_score = max(0, min(100, raw_score))
    else:
        risk_score = 50  # Neutral for no data
    
    developer.risk_score = risk_score
    developer.bug_rate = bugs_caused / features_count if features_count > 0 else 0
```

**Output Required**: Create `$OUTPUT_DIR/data/${ANALYSIS_KEY}-scores.json` (and symlink to `developer-scores.json` for backward compatibility):
```json
{
  "input_type": "squad",
  "analysis_key": "squad_rewards-web",
  "period": {
    "start": "2025-09-24",
    "end": "2026-03-24"
  },
  "developers": [
    {
      "login": "philiptauro",
      "name": "Philip Tauro",
      "display_name": "Philip Tauro",
      "risk_score": 87,
      "features_count": 15,
      "bugs_caused": 2,
      "bug_rate": 0.13,
      "risk_level": "low",
      "features": [67382, 67301],
      "bugs": [67400]
    }
  ],
  "squad_average": 82,
  "generated_at": "2026-03-24T10:30:00Z"
}
```

**CRITICAL**: The `analysis_key` field must use underscores to match the file naming convention (e.g., `squad_SSX`, `jira_REW`). The format is: `${INPUT_TYPE}_${SQUAD_NAME_OR_PROJECT}`.

**Note on Display Names:**
- Store both `login` (GitHub username) and `name` (real name)
- `display_name` should be the real name if available, otherwise username
- In dashboard, show: "Philip Tauro (@philiptauro)" or just "philiptauro" if no real name

**Risk Levels:**
- `risk_score >= 80`: "low" (🟢)
- `risk_score 60-79`: "medium" (🟡)
- `risk_score < 60`: "high" (🔴)

---

### Step 6: Save Data and Update Index

**CRITICAL**: Save analysis data with prefixed filenames to support multiple squads/projects:

```bash
ANALYSIS_KEY="${INPUT_TYPE}_${SQUAD_NAME:-$JIRA_PROJECT}"

# Save with prefixed names
cp developer-scores.json "$OUTPUT_DIR/data/${ANALYSIS_KEY}-scores.json"
cp analyzed-prs.json "$OUTPUT_DIR/data/${ANALYSIS_KEY}-analyzed-prs.json"

# Create/update symlinks for latest analysis (backward compatibility)
ln -sf "${ANALYSIS_KEY}-scores.json" "$OUTPUT_DIR/data/developer-scores.json"
ln -sf "${ANALYSIS_KEY}-analyzed-prs.json" "$OUTPUT_DIR/data/analyzed-prs.json"

# Update all-analyses.json index
INDEX_FILE="$OUTPUT_DIR/data/all-analyses.json"
if [ -f "$INDEX_FILE" ]; then
  # Add or update this analysis
  jq --arg key "$ANALYSIS_KEY" --arg label "$LABEL" --arg type "$INPUT_TYPE" \
    '.analyses |= (map(select(.key != $key)) + [{key: $key, label: $label, type: $type, updated: now | todate}])' \
    "$INDEX_FILE" > "$INDEX_FILE.tmp" && mv "$INDEX_FILE.tmp" "$INDEX_FILE"
else
  # Create new index
  echo "{\"analyses\": [{\"key\": \"$ANALYSIS_KEY\", \"label\": \"$LABEL\", \"type\": \"$INPUT_TYPE\", \"updated\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}]}" > "$INDEX_FILE"
fi
```

---

### Step 7: Generate Interactive Dashboard

**ONLY generate on first run** (check if `$OUTPUT_DIR/index.html` exists):

```bash
if [ ! -f "$OUTPUT_DIR/index.html" ]; then
  echo "   Generating dashboard..."
  # Copy dashboard template
  cp /path/to/dashboard-template.html "$OUTPUT_DIR/index.html"
else
  echo "   ✓ Dashboard already exists (data updated only)"
fi
```

Create `$OUTPUT_DIR/index.html` with:

**Dashboard Features:**
1. **Squad/Project Selector Dropdown**
   - Auto-populated from `all-analyses.json`
   - Shows "Squad: X" or "JIRA: Y" format
   - Dynamically loads data on selection

2. **Overview Card**
   - Analysis type (Squad or JIRA Project)
   - Analysis key (squad name or JIRA project)
   - Date range
   - Total developers analyzed
   - Average risk score

3. **Developer Risk Table**
   - Columns: Developer Name (with username), Risk Score, Features, Bugs, Bug Rate, Risk Level
   - Display format: "Philip Tauro (@philiptauro)" or just username if name unavailable
   - Sortable by risk score (highest risk first by default)
   - Color-coded risk levels

4. **Individual Developer Details** (expandable rows)
   - List of features contributed to (with PR links)
   - List of bugs caused (with PR links to both feature and fix)
   - Timeline of contributions
   - Risk trend (if data available)

4. **Charts**
   - Risk score distribution (bar chart)
   - Bug rate comparison (developer comparison)
   - Timeline of features vs bugs over 6 months

**Design Requirements:**
- Dark theme (matching the test coverage dashboard style)
- Professional, sleek design
- No AI-generated patterns
- Responsive layout
- Click developer name → expand details

**HTML Structure:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Developer Risk Analysis - {SQUAD_NAME}</title>
  <style>
    /* Use same dark theme as test coverage dashboard */
    :root {
      --bg-primary: #0a0a0a;
      --bg-secondary: #141414;
      --bg-tertiary: #1a1a1a;
      --border-primary: #2a2a2a;
      --text-primary: #e0e0e0;
      --text-secondary: #9a9a9a;
      --accent-orange: #F99D33;
      --success: #10b981;
      --warning: #f59e0b;
      --error: #ef4444;
    }
    /* ... rest of styles ... */
  </style>
</head>
<body>
  <!-- Dashboard content here -->
  <script>
    // Load data from data/developer-scores.json
    // Render tables and charts
  </script>
</body>
</html>
```

---

### Step 7: Display Results

```bash
echo ""
echo "✅ Developer Risk Analysis Complete!"
echo ""
echo "📊 Results:"
if [ "$INPUT_TYPE" = "squad" ]; then
  echo "   Squad: $ANALYSIS_KEY"
else
  echo "   JIRA Project: $ANALYSIS_KEY"
fi
echo "   Developers Analyzed: $(jq '.developers | length' $OUTPUT_DIR/data/${ANALYSIS_KEY}-scores.json)"
echo "   Average Risk Score: $(jq '.squad_average' $OUTPUT_DIR/data/${ANALYSIS_KEY}-scores.json)"
echo ""
echo "📄 Generated Files:"
echo "   ✅ $OUTPUT_DIR/index.html (dashboard)"
echo "   ✅ $OUTPUT_DIR/data/${ANALYSIS_KEY}-scores.json"
echo "   ✅ $OUTPUT_DIR/data/${ANALYSIS_KEY}-analyzed-prs.json"
echo "   ✅ $OUTPUT_DIR/data/all-analyses.json (index)"
echo ""
echo "🌐 Dashboard:"
if [ ! -f "$OUTPUT_DIR/.server-running" ]; then
  echo "   Starting server on port 8766..."
  cd "$OUTPUT_DIR" && python3 -m http.server 8766 >/dev/null 2>&1 &
  echo $! > "$OUTPUT_DIR/.server-pid"
  touch "$OUTPUT_DIR/.server-running"
  sleep 2
  echo "   ✓ Server started"
  echo "   Opening: http://localhost:8766"
  open "http://localhost:8766"
else
  echo "   Server already running on port 8766"
  echo "   Refresh browser to see updated data: http://localhost:8766"
fi
echo ""
```

---

## Implementation Notes

### Co-Author Detection

Use multiple methods to detect co-authors:
1. **GitHub PR API**: Check `author` field (with `name` and `login`) and look for co-authored-by in commit messages
2. **Git Commits**: Parse commit messages for `Co-authored-by:` trailers
3. **PR Comments**: Check if multiple developers collaborated (optional)

**Extract Real Names:**
```bash
# Get PR author info
gh pr view PR_NUMBER --json author --jq '{login: .author.login, name: .author.name}'

# Get commits for a PR and check for co-authors
gh pr view PR_NUMBER --json commits --jq '.commits[].messageBody' | grep -i "co-authored-by" | \
  sed -n 's/.*Co-authored-by: \(.*\) <\(.*\)>/\1 (\2)/p'
```

**Name Fallback Strategy:**
1. Try `author.name` from GitHub API
2. If empty/null, try git config `user.name` for that email
3. If still empty, use `author.login` (username)

**Best approach**: Combine GitHub PR author + git commit co-authors, always prefer real names over usernames.

### Critical Bug Detection

A bug is "critical" if:
1. Label contains: `severity: critical`, `priority: high`, `type: hotfix`
2. Files affected include: production configs, authentication, payment flows
3. PR title contains: "critical", "hotfix", "urgent", "production"

```bash
# Check if bug fix is critical
is_critical=false
if echo "$labels" | grep -iq "critical\|hotfix\|severity: high"; then
  is_critical=true
fi
```

### Legacy Code Attribution

When a bug fix touches code written by someone else:

```bash
# For each file in bug fix PR, find original authors
for file in $bug_fix_files; do
  # Get authors of lines changed in this file (last 6 months)
  git log --since="$START_DATE" --pretty=format:"%an" -- "$file" | sort | uniq
  
  # Use git blame to find who wrote the buggy lines
  # (requires diff analysis of what lines were changed)
done
```

**Simplified approach**: If bug fix happens within 6 months and touches same files as a feature, attribute to feature author(s).

---

## Error Handling

- If `gh` CLI not available → show error and installation instructions
- If squad has no PRs → show "No data for this squad in the last 6 months"
- If API rate limits hit → cache partial results and retry
- If JSON parsing fails → show error with raw data location

---

## Future Enhancements

- Compare risk scores across multiple squads
- Track risk score trends over time (month-by-month)
- Integrate with JIRA to track bug severity from tickets
- Add developer comparison view (head-to-head)
- Export to PDF for management reports
