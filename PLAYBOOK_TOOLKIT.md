# PLAYBOOK_TOOLKIT.md
> Complete 3-phase process to build your reusable developer skill library
> From 21+ projects → one universal PLAYBOOK.md

---

## OVERVIEW

```
Phase 1: EXTRACT    — one chat per project → PROJECT_X_CONTRIBUTION.md
Phase 2: COMPILE    — neutral Claude chat → PLAYBOOK_DRAFT.md  
Phase 3: AUDIT      — send back to each project → PLAYBOOK_v1.md
```

Do phases in any order of availability. No rush. Each project chat is independent.
Store all contribution files in a folder: `playbook-contributions/`

---

---

# PHASE 1 — CONTRIBUTION PROMPT
> Use this in each project's chat session
> Output: PROJECT_X_CONTRIBUTION.md

---

## PASTE THIS INTO EACH PROJECT CHAT

```
I am building a universal developer PLAYBOOK — a reusable skill library 
extracted from all my projects. Your job is to analyze THIS project only 
and produce a CONTRIBUTION.md that captures what this project uniquely 
teaches or confirms.

Do NOT write a full playbook. Write only what this project contributes.
Be honest — include failures and anti-patterns, not just successes.

Scan the following files from this project:
- CLAUDE.md or equivalent project constitution
- RULES.md or LESSONS.md — all lessons learned
- PROJECT_STATE.md or equivalent architecture doc
- 2-3 key API files (the most complex ones)
- 2-3 key frontend/injector files (the most complex ones)
- Any config files (wrangler.toml, package.json, etc.)

Then answer ALL sections below. Be specific — give code examples where possible.
Use the exact format shown. Output as markdown ready to save as a .md file.
```

---

## CONTRIBUTION TEMPLATE (output format)

```markdown
# CONTRIBUTION — [Project Name]
> Scanned: [date]
> Stack: [e.g. Cloudflare Pages + KV + Airtable + React]
> Project type: [e.g. personal finance OS / e-commerce / portfolio / SaaS tool]
> Maturity: [e.g. 21 iterations / MVP / production]

---

## 1. PROJECT IDENTITY
- **What it does in one sentence:**
- **Primary users:** 
- **Data sources:** (APIs, databases, files)
- **Deployment platform:**
- **Authentication method:**

---

## 2. ARCHITECTURE DECISIONS
> How was the app structured? What worked, what didn't?

### What this project used:
- Frontend pattern: [e.g. injector per panel / React components / vanilla JS modules]
- Routing: [e.g. hash routing / Next.js / SPA]
- State management: [e.g. module-level variables / Redux / Zustand]
- API pattern: [e.g. one file per resource / monolithic / tRPC]
- CSS approach: [e.g. CSS variables / Tailwind / CSS modules]

### ✅ Architecture decisions that worked well:
1. [decision] — [why it worked]
2.
3.

### ⚠️ Architecture decisions that caused problems:
1. [decision] — [what went wrong] — [how fixed or what to do instead]
2.
3.

### 🔁 What I would change on day 1 of a new project:
1.
2.

---

## 3. MEMORY STRATEGY
> How did this project store and retrieve data?

### Storage used:
- [ ] KV (key-value cache/session)
- [ ] D1 (SQLite)
- [ ] R2 (object storage)
- [ ] localStorage / sessionStorage
- [ ] External database (Airtable / Supabase / PlanetScale / etc.)
- [ ] In-memory only (module variables)

### KV usage (if applicable):
- Key naming convention used: [e.g. `table_all_v1`, `session:{id}`]
- TTL strategy: [e.g. static=3600s, dynamic=300s]
- Cache invalidation approach:
- Biggest KV mistake made:

### Database usage (if applicable):
- Which database and why:
- Free tier limits hit or approached:
- How managed (caching, rate limiting, batching):
- Biggest database mistake made:

### ✅ Memory pattern that worked best:
[describe with code example if possible]

### ⚠️ Memory anti-pattern discovered:
[describe with code example of what NOT to do]

---

## 4. API STRATEGY
> How were server-side APIs designed and called?

### API file structure:
[describe e.g. one file per resource, monolithic, etc.]

### Patterns used:
- [ ] Shared helper functions
- [ ] Input validation at API layer
- [ ] Consistent response shape `{ records: [] }` or similar
- [ ] Error response helper
- [ ] Idempotent writes (dedup before create)
- [ ] Guard against double-submit
- [ ] Pagination for large datasets

### ✅ API pattern that saved the most bugs:
[describe]

### ⚠️ API mistake made more than once:
[describe — and the fix]

### Rate limits hit on any external API:
- API: [name] — Limit: [X/period] — Hit: [yes/no] — Solution: [how managed]

---

## 5. SECURITY STRATEGY
> How was the app secured?

### Auth method used:
[e.g. PIN + KV session / JWT / OAuth / none]

### What was protected:
- [ ] All API routes behind auth middleware
- [ ] Environment secrets in platform dashboard (never in code)
- [ ] Read-only access for foreign data sources
- [ ] Input sanitization

### ✅ Security decision that worked:
[describe]

### ⚠️ Security gap discovered (if any):
[describe — be honest, this is for learning]

---

## 6. FRONTEND PATTERNS
> What UI patterns were used and how did they hold up?

### Component/panel anatomy used:
[e.g. stats strip → chart → data body]

### Event handling approach:
- [ ] Event delegation (one listener on container)
- [ ] Direct binding on elements
- [ ] Framework-managed (React/Vue/etc.)

### ✅ UI pattern that scaled well across many panels:
[describe]

### ⚠️ UI anti-pattern that caused bugs:
[describe — e.g. binding to innerHTML elements, lost on re-render]

### CSS approach that worked:
[describe]

---

## 7. AI INTEGRATION (if used)
> How was AI integrated into this project?

### AI used for:
- [ ] Content generation
- [ ] Data extraction
- [ ] Code assistance (via Claude Code)
- [ ] In-app AI features (chat, suggestions, analysis)
- [ ] Structured data generation (JSON output)

### Streaming vs buffered:
- Used streaming: [yes/no]
- Used buffered JSON: [yes/no]
- Issue encountered:

### Structured output pattern used:
[e.g. prompt for JSON only → parse → fallback to text]

### ✅ AI integration that added most value:
[describe]

### ⚠️ AI integration mistake:
[describe — e.g. trusting AI output without validation]

---

## 8. HUMAN-AI COLLABORATION (if Claude Code used)
> How did the development workflow with AI assistance work?

### Workflow used:
- [ ] Chat role (diagnose) + CC role (execute)
- [ ] Direct prompting only
- [ ] Mixed approach

### Prompt structure that worked best:
[describe — e.g. READ FIRST + FIXES + DO NOT TOUCH + MANDATORY]

### Context management:
- How many iterations before context degraded: [~N]
- Handoff strategy used: [describe]
- Biggest context loss mistake:

### ✅ Collaboration pattern that saved most time:
[describe]

### ⚠️ Collaboration anti-pattern:
[describe — e.g. not searching RULES.md before writing a new fix]

---

## 9. THE 5 BIGGEST BUGS & FIXES
> Most valuable learning — be specific

| # | Bug description | Root cause | Fix | Rule to add |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

---

## 10. THE 5 RULES THIS PROJECT WOULD ADD TO EVERY NEW PROJECT
> Universal laws discovered — write as RULES.md style entries

```
R1: [rule text — specific, actionable, testable]
R2:
R3:
R4:
R5:
```

---

## 11. FREE TIER USAGE REPORT
> Reality check on platform costs

| Service | Free limit | Peak usage | Hit limit? | Solution used |
|---|---|---|---|---|
| Cloudflare Pages | 500 builds/mo | | | |
| Cloudflare KV | 100k reads/day | | | |
| Cloudflare Workers | 100k req/day | | | |
| Airtable | 1000 calls/2wk | | | |
| Cloudinary | 25 credits/mo | | | |
| Anthropic API | pay-per-use | | | |
| Other: | | | | |

---

## 12. ONE THING TO STEAL FOR EVERY FUTURE PROJECT
> The single most transferable pattern from this project

[describe in 2-3 sentences — make it concrete and actionable]

---

## 13. ONE THING TO NEVER REPEAT
> The single worst decision or pattern — warn future self

[describe in 2-3 sentences — be brutally honest]
```

---

---

# PHASE 2 — COMPILATION PROMPT
> Use in a NEUTRAL Claude chat — NO project connected
> Input: all PROJECT_X_CONTRIBUTION.md files
> Output: PLAYBOOK_DRAFT.md

---

## PASTE THIS INTO A NEUTRAL CLAUDE CHAT

```
I have extracted contributions from [N] different app projects.
Each file is a structured analysis of one project's patterns, bugs, and lessons.
Your job is to compile them into a single universal PLAYBOOK.md.

I will paste all contribution files below, separated by --- markers.

COMPILATION RULES:
1. A pattern that appears in 3+ projects = UNIVERSAL LAW → mark ✅ ALWAYS
2. A pattern in 2 projects = STRONG PATTERN → mark 💡 RECOMMENDED  
3. A pattern in 1 project = SITUATIONAL → mark 🔧 SITUATIONAL
4. An anti-pattern in 2+ projects = RED FLAG → mark ⚠️ NEVER
5. Contradictions between projects = flag as DECISION POINT with both options shown
6. Strip all project-specific details (names, amounts, personal data)
7. Keep all code examples but make them generic ({tableName}, {resourceId}, etc.)
8. Every rule must be: specific, actionable, testable — not vague advice
9. Cite frequency: "(seen in 4/8 projects)" after each pattern

Use this structure for PLAYBOOK.md:
[paste the TABLE OF CONTENTS from PLAYBOOK_CREATION_PROMPT.md]

For each section, format entries as:

✅ ALWAYS: [pattern name]
Frequency: N/N projects
Why: [one sentence]
How: [code example or specific steps]
Anti-pattern: [what people do instead and why it fails]

⚠️ NEVER: [anti-pattern name]  
Frequency: seen in N projects
Why it fails: [specific failure mode]
Instead: [what to do]

🔧 SITUATIONAL: [pattern name]
When to use: [specific condition]
When NOT to use: [specific condition]

DECISION POINT: [topic]
Option A: [approach] — best when [condition]
Option B: [approach] — best when [condition]

Output as a complete PLAYBOOK.md ready to save.
Target: 2000-3000 lines, comprehensive but scannable.

--- CONTRIBUTIONS START ---

[paste all PROJECT_X_CONTRIBUTION.md files here]

--- CONTRIBUTIONS END ---
```

---

---

# PHASE 3 — AUDIT PROMPT
> Use in EACH project chat after PLAYBOOK_DRAFT.md is created
> Output: list of conflicts or additions only (not a full rewrite)

---

## PASTE THIS INTO EACH PROJECT CHAT (with PLAYBOOK_DRAFT.md attached)

```
I have created a PLAYBOOK_DRAFT.md compiled from multiple projects.
Your job is to AUDIT it against THIS project only.

Read:
1. The attached PLAYBOOK_DRAFT.md
2. This project's RULES.md / CLAUDE.md / PROJECT_STATE.md

Then produce a SHORT audit report covering ONLY:

CONFLICTS: Rules in the playbook that this project contradicts
(with explanation of why this project does it differently — is it an exception or a playbook error?)

MISSING: Important patterns this project uses that are NOT in the playbook

CONFIRM: Top 5 rules in the playbook that this project most strongly validates

DO NOT rewrite the whole playbook. Just flag conflicts, gaps, and confirmations.
Output as a short numbered list — max 20 items total.
```

---

---

# STORAGE & FILE STRUCTURE

```
playbook-contributions/
├── PROJECT_chaijohn_CONTRIBUTION.md      ← this project first
├── PROJECT_[name2]_CONTRIBUTION.md
├── PROJECT_[name3]_CONTRIBUTION.md
├── ... (one per project)
├── PLAYBOOK_DRAFT.md                     ← Phase 2 output
├── AUDIT_chaijohn.md                     ← Phase 3 outputs
├── AUDIT_[name2].md
└── PLAYBOOK_v1.md                        ← final compiled + audited

PLAYBOOK.md                               ← symlink or copy to repo root
```

---

# PROCESS CHECKLIST

## Phase 1 — Extract (do whenever you have time per project)
- [ ] chaijohn-personal
- [ ] [project 2]
- [ ] [project 3]
- [ ] ... (all 21+)

## Phase 2 — Compile (after 5+ contributions collected, don't wait for all)
- [ ] Open neutral Claude chat (no project connected)
- [ ] Paste compilation prompt + all contribution files
- [ ] Save output as PLAYBOOK_DRAFT.md

## Phase 3 — Audit (after draft exists)
- [ ] Send audit prompt to each project chat
- [ ] Collect audit reports
- [ ] Apply fixes to draft → PLAYBOOK_v1.md

## Maintenance
- [ ] After every 3 new projects: re-run Phase 1 on new ones + re-compile
- [ ] After major bug discovery: add to nearest project's contribution + re-compile
- [ ] Target: PLAYBOOK_v2.md after 10 projects, v3 after 20

---

# QUICK REFERENCE — KEY QUESTIONS TO ALWAYS ASK

When auditing any project for contribution, these 5 questions get the most signal:

1. **"What was your biggest recurring bug?"** → becomes ⚠️ NEVER rule
2. **"What would you add to RULES.md on day 1?"** → becomes ✅ ALWAYS rule  
3. **"What external API limit did you hit?"** → becomes free tier strategy
4. **"What did the AI (Claude) get wrong most often?"** → becomes collaboration rule
5. **"What one pattern do you wish you had from the start?"** → becomes STEAL THIS

These 5 questions alone generate 80% of the playbook value.
