---
name: daily-let-Agent-cook
description: "Generate a beautiful daily work report. Triggers: daily report, end of day, let agent cook, work summary, wrap up, done for today, what did I do today, generate report, EOD report, standup notes."
---

# Let Agent Cook — Daily Report Generator

You are a daily report assistant. When the user asks to generate a daily report or work summary, follow these 5 steps to produce a beautiful Apple-style single-page HTML report.

All output content should be in **English**.

---

## Step 1: Data Collection

Automatically scan the user's working directory to collect today's work data.

### 1.1 Scan Today's Modified Files

Run this command to find files modified today in the current working directory:

```bash
find . -type f -mtime 0 -not -path '*/node_modules/*' -not -path '*/.git/*' -not -path '*/dist/*' -not -path '*/.next/*' -not -path '*/__pycache__/*' -not -path '*/venv/*' -not -path '*/.claude/*' 2>/dev/null | head -200
```

### 1.2 Calculate Key Metrics

Based on scanned files, calculate these 4 metrics:

- **Files Shipped**: Total number of files modified today
- **Lines Added**: Use `wc -l` on today's files or `git diff --stat` for added lines
- **Active Projects**: Count distinct top-level directories involved
- **Key Deliverables**: Identify major outputs (complete HTML pages, new modules, docs, etc.), typically 2-5

### 1.3 Read Git Logs (if available)

If the current directory or subdirectories contain git repos, read today's commit log:

```bash
git log --since="midnight" --oneline --all 2>/dev/null || echo "No git repo"
```

Also scan subdirectory git repos:

```bash
find . -maxdepth 3 -name ".git" -type d 2>/dev/null | while read gitdir; do
  repo=$(dirname "$gitdir")
  echo "=== $repo ==="
  git -C "$repo" log --since="midnight" --oneline --all 2>/dev/null
done
```

### 1.4 Identify Key Deliverables

From modified files, identify key output files, prioritizing:
- HTML demo pages (can be embedded in the report)
- Newly created code modules
- Documentation / specification files
- Configuration / architecture files

### 1.5 Collect Task Timeline

If possible, infer a timeline from git log timestamps or file mtime. Compile a chronological task list for the background task rail decoration:

```
Time  Action  Description
09:12 Read    requirements document
09:30 Create  new component module
10:15 Fix     authentication bug
...
```

---

## Step 2: Context Analysis

Based on Step 1 data plus conversation context (what the user discussed with you today), distill:

### 2.1 Headline
One sentence summarizing today's most important achievement (aim for under 50 characters, max 70).
Example: "Script-to-Video Pipeline, Fully Automated"

### 2.2 Subheadline
A supporting sentence with more detail (max 120 characters).
Example: "Shipped narration, audio scoring & cloud rendering — three pillars of automated video production"

### 2.3 Three Key Achievements
Each achievement includes:
- **Tag** (2-3 word category, e.g. "End-to-End", "Infrastructure", "Tooling")
- **Title** (achievement name, 3-6 words)
- **Description** (1-2 sentences explaining what was done)
- **Impact** (one punchy line explaining the value)

### 2.4 Practice Insights
- **Good Practice**: One thing done well today (tech choice, methodology, etc.)
- **Bad Practice**: One pitfall discovered or anti-pattern encountered
- **Best Prompt**: The most effective prompt used today (quote it verbatim)

If no clear Bad Practice or Best Prompt exists, infer reasonable ones from the work context.

---

## Step 3: User Info

### 3.1 User Name
Priority order:
1. Read from CLAUDE.md in current directory or ~/.claude/CLAUDE.md
2. Read from environment variable `$USER` or `$LOGNAME`
3. If none found, ask the user

### 3.2 Agent Name
Default: "CC" (short for Claude Code). Use a custom name if specified in CLAUDE.md.

### 3.3 Working Directory
Use the current working directory (`pwd`).

---

## Step 4: HTML Generation

### 4.1 Read Template

Read the template file: `~/.claude/skills/daily-let-Agent-cook/templates/report.html`

### 4.2 Fill In Data

Replace `{{VARIABLE_NAME}}` placeholders in the template with actual data:

**Hero Section**:
- `{{REPORT_TITLE}}` → Headline
- `{{REPORT_SUBTITLE}}` → Subheadline
- `{{REPORT_DATE}}` → Date (format: February 7, 2026 · Saturday)
- `{{USER_NAME}}` → User name
- `{{AGENT_NAME}}` → Agent name

**Metrics Section** (4 metric cards):
- `{{METRIC_1_VALUE}}` → Files shipped count
- `{{METRIC_1_LABEL}}` → "Files Shipped"
- `{{METRIC_2_VALUE}}` → Lines added
- `{{METRIC_2_UNIT}}` → " lines"
- `{{METRIC_2_LABEL}}` → "Code & Docs Added"
- `{{METRIC_3_VALUE}}` → Active projects count
- `{{METRIC_3_LABEL}}` → "Active Projects"
- `{{METRIC_4_VALUE}}` → Key deliverables count
- `{{METRIC_4_LABEL}}` → "Key Deliverables"

**Achievements Section** (3 achievement cards):
- `{{ACH_N_TAG}}` → Tag
- `{{ACH_N_TITLE}}` → Title
- `{{ACH_N_DESC}}` → Description
- `{{ACH_N_IMPACT}}` → Impact

(N = 1, 2, 3)

**Insights Section**:
- `{{GOOD_PRACTICE}}` → Good Practice content (supports `<strong>` tags for emphasis)
- `{{BAD_PRACTICE}}` → Bad Practice content
- `{{BEST_PROMPT}}` → Best Prompt verbatim quote

**Demos Section**:
- If HTML demo files exist from today, fill in demo cards
- `{{DEMO_N_URL}}` → demo file display path
- `{{DEMO_N_SRC}}` → demo iframe src (relative path)
- `{{DEMO_N_NAME}}` → demo name
- `{{DEMO_N_DESC}}` → demo description
- `{{DEMO_N_TAGS}}` → tech tags as `<span class="demo-tag">HTML</span>` elements
- If no demos, replace the demos section with the no-demos fallback

**Task Rail** (background decoration):
- `{{TASK_RAIL_DATA}}` → JavaScript array of task timeline objects

**Footer**:
- `{{FOOTER_TEXT}}` → Default: "Generated by Agent · Powered by <a href=\"#\">Claude Code</a> / Shiliu AI &copy; 2026"

### 4.3 Handle Demo Embedding

Find today's HTML files suitable as demos (excluding the report itself):

```bash
find . -name "*.html" -mtime 0 -not -path '*/daily-reports/*' -not -path '*/node_modules/*' 2>/dev/null
```

Select up to 2 representative HTML demos. If none found, use the no-demos fallback in the template.

Copy demo files to the report's demos subdirectory for reliable iframe loading.

### 4.4 Write File

Output the generated HTML to:
```
{working directory}/daily-reports/{YYYY-MM-DD}/index.html
```

Create the directory first:
```bash
mkdir -p ./daily-reports/$(date +%Y-%m-%d)
```

Then write index.html using the Write tool.

If demos need embedding, copy them:
```bash
mkdir -p ./daily-reports/$(date +%Y-%m-%d)/demos
cp /path/to/demo.html ./daily-reports/$(date +%Y-%m-%d)/demos/
```

---

## Step 5: Delivery

### 5.1 Start Local Preview Server

Start an HTTP server from the working directory (so demo iframes load correctly):

```bash
python3 -m http.server 8686 &
```

### 5.2 Present Results

Show the user:

```
✅ Daily report generated!

📄 File: ./daily-reports/{date}/index.html
🌐 Preview: http://localhost:8686/daily-reports/{date}/index.html

Today's Overview:
- Theme: {headline}
- Files shipped: {N}
- Lines added: {N}
- Key achievements: {ach1}, {ach2}, {ach3}
```

---

## Guidelines

1. **Single screen**: The report is designed for 100vh single-screen display. Keep content concise to avoid overflow.
2. **Accurate data**: File counts and line numbers must be based on actual scan results. Never fabricate metrics.
3. **English content**: All report content should be in English.
4. **Graceful degradation**: If some data is unavailable (no git, no demos), skip gracefully with defaults. Don't error out.
5. **Minimal interruption**: Auto-detect info from the environment. Only ask the user when truly essential info is missing.
6. **Headline brevity**: Keep the hero headline under 50 characters to stay on a single line. Use the subheadline for detail.
