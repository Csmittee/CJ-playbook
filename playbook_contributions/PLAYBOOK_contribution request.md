
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
- Key naming convention used:
- TTL strategy:
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
[describe]

### Patterns used:
- [ ] Shared helper functions
- [ ] Input validation at API layer
- [ ] Consistent response shape
- [ ] Error response helper
- [ ] Idempotent writes
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

### Auth method used:
[describe]

### What was protected:
- [ ] All API routes behind auth middleware
- [ ] Environment secrets in platform dashboard
- [ ] Read-only access for foreign data sources
- [ ] Input sanitization

### ✅ Security decision that worked:
[describe]

### ⚠️ Security gap discovered (if any):
[describe]

---

## 6. FRONTEND PATTERNS

### Component/panel anatomy used:
[describe]

### Event handling approach:
- [ ] Event delegation
- [ ] Direct binding on elements
- [ ] Framework-managed

### ✅ UI pattern that scaled well:
[describe]

### ⚠️ UI anti-pattern that caused bugs:
[describe]

### CSS approach that worked:
[describe]

---

## 7. AI INTEGRATION (if used)

### AI used for:
- [ ] Content generation
- [ ] Data extraction
- [ ] Code assistance (via Claude Code)
- [ ] In-app AI features
- [ ] Structured data generation

### ✅ AI integration that added most value:
[describe]

### ⚠️ AI integration mistake:
[describe]

---

## 8. HUMAN-AI COLLABORATION (if Claude Code used)

### Workflow used:
- [ ] Chat role (diagnose) + CC role (execute)
- [ ] Direct prompting only
- [ ] Mixed approach

### Prompt structure that worked best:
[describe]

### Context management:
- How many iterations before context degraded: [~N]
- Handoff strategy used:
- Biggest context loss mistake:

### ✅ Collaboration pattern that saved most time:
[describe]

### ⚠️ Collaboration anti-pattern:
[describe]

---

## 9. THE 5 BIGGEST BUGS & FIXES

| # | Bug description | Root cause | Fix | Rule to add |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

---

## 10. THE 5 RULES THIS PROJECT WOULD ADD TO EVERY NEW PROJECT

```
R1: [rule text — specific, actionable, testable]
R2:
R3:
R4:
R5:
```

---

## 11. FREE TIER USAGE REPORT

| Service | Free limit | Peak usage | Hit limit? | Solution used |
|---|---|---|---|---|
| Cloudflare Pages | 500 builds/mo | | | |
| Cloudflare KV | 100k reads/day | | | |
| Airtable | 1000 calls/2wk | | | |
| Anthropic API | pay-per-use | | | |
| Other: | | | | |

---

## 12. ONE THING TO STEAL FOR EVERY FUTURE PROJECT
[describe in 2-3 sentences]

---

## 13. ONE THING TO NEVER REPEAT
[describe in 2-3 sentences]

---

## 14. MIND MAP NODES — structured extract for direct import
> This section feeds Phase 4 — bulk POST to MindMapNodes API
> Use exact field names: label, node_type, description, tags, context, status
> node_type must be one of: Business, Project, Tool, People, Skill, Resource, Idea, Pattern
> status must be one of: Active, Paused, Archived, Idea

```json
{
  "nodes": [
    {
      "label": "[project name]",
      "node_type": "Project",
      "status": "Active",
      "description": "[one sentence from Section 1]",
      "tags": "[stack keywords comma-separated, e.g. Cloudflare,Airtable,KV]",
      "context": "[deployment platform] · [auth method] · [maturity level] · [primary pattern used]"
    },
    {
      "label": "[tool/service used — one node per tool]",
      "node_type": "Tool",
      "status": "Active",
      "description": "[how this tool was used in this project]",
      "tags": "[category, e.g. database,cache,hosting]",
      "context": "[free tier limit if relevant] · [key constraint discovered]"
    },
    {
      "label": "[pattern name — e.g. 'KV cache-first render']",
      "node_type": "Pattern",
      "status": "Active",
      "description": "[what the pattern does — one sentence]",
      "tags": "[applicable stack keywords]",
      "context": "[when to use] · [when NOT to use] · [anti-pattern it replaces]"
    }
  ],
  "edges": [
    {
      "from": "[label of from node]",
      "to": "[label of to node]",
      "edge_type": "uses|requires|spawned|inspired_by|supports|knows|led_to|funds|failed_to",
      "label": "[short description of relationship]",
      "strength": 1
    }
  ]
}
```

### Instructions for filling Section 14:
- Always create ONE node for the project itself (node_type: Project)
- Create ONE node per major tool/service (Airtable, Cloudflare KV, etc.) — skip if already exists in Mind Map
- Create ONE node per unique architectural pattern discovered (node_type: Pattern)
- Create ONE node per key person involved if relevant (node_type: People)
- Keep descriptions under 100 chars — they display as tooltips in the graph
- Tags are for agent queries — use consistent keywords across all contributions
- Edges: connect project → tools it requires, project → patterns it uses, patterns → tools they require
```

---
