# CONTRIBUTION — Chaijohn OS (chaijohn-personal)
> Scanned: 2026-06-11
> Stack: Cloudflare Pages + KV + Airtable (dual-base) + Cloudinary + Anthropic API
> Project type: Personal Finance + Business OS — single-owner dashboard
> Maturity: 14 sessions, 214+ rules, 21+ iterations, production

---

## 1. PROJECT IDENTITY

- **What it does in one sentence:** A single-owner full-stack financial and business operating system — tracking cashflow, expenses, assets, projects, liabilities, and life timeline in one dashboard.
- **Primary users:** One person (owner). No multi-user. No public access.
- **Data sources:** Airtable (personal base + business read-only base), Cloudinary (media), Anthropic API (AI advisor + drop zone vision), Cloudflare KV (cache + session)
- **Deployment platform:** Cloudflare Pages + Pages Functions (serverless)
- **Authentication method:** PIN → SHA-256 + salt hash stored in KV → HttpOnly session cookie with 7-day TTL

---

## 2. ARCHITECTURE DECISIONS

> How was the app structured? What worked, what didn't?

### What this project used:
- Frontend pattern: One injector JS file per panel — lazy-init on `panelactivated` event
- Routing: Hash routing (`#panel-name`) — no page reloads, no framework
- State management: Module-level variables inside each IIFE injector
- API pattern: One file per resource (`transactions.js`, `budgets.js`, `liabilities.js`) + shared helpers in `functions/_airtable.js`
- CSS approach: CSS variables only — 8 root variables, dark mode native, no Tailwind, no build step

### ✅ Architecture decisions that worked well:
1. **One injector per panel** — isolated scope, changes never break other panels, easy to read 14 sessions later without context loss.
2. **`_airtable.js` shared helper** — all Airtable logic (listRecords, createRecord, patchRecord, jsonResponse, errorResponse) in one file; every API file imports from `../_airtable.js`. Zero duplication across 20+ API files.
3. **Hash routing + `panelactivated` event** — panels load on demand, not at shell boot. Cold start is fast. Heavy panels (budget matrix, mind map) never block dashboard init.
4. **KV as both session store and API cache** — one binding, two critical uses. PIN hash in KV, session tokens in KV, all heavy GET responses cached in KV. Airtable free tier protected from day 1.
5. **Dual Airtable base architecture** — personal data isolated from business data. Business base is read-only from personal dashboard. Clean boundary enforced in code.

### ⚠️ Architecture decisions that caused problems:
1. **`Promise.allSettled` for batch writes** — worked fine at 10–20 records per session, silently broke at 64 records (Budget panel S14). Cloudflare concurrency + Airtable 5 req/sec ceiling exceeded → 500 errors. Fixed: sequential `for...of` loop with 210ms delay (L211).
2. **`category_id` on transactions** — early decision to attach category routing via `category_id` on every transaction. Later discovered `source` field alone routes everything. Three separate API files had dead category lookup code for 10+ sessions before cleanup in S14 (L212, L214).
3. **Injector using `p.innerHTML = renderPanel()`** — overwrites the entire panel div including the header defined in `index.html`. Caused disappearing panel titles silently. Rule: panel header must be rendered INSIDE `renderPanel()` itself (L147).

### 🔁 What I would change on day 1 of a new project:
1. Define the transaction model (source field, type field, routing table) on day 1. Never let `category_id` drift into being a routing mechanism alongside `source`.
2. Add `isSubmitting` guard to every save button before writing the first API call. Cold-start double-POST is not a risk — it is a guarantee without the guard.

---

## 3. MEMORY STRATEGY

> How did this project store and retrieve data?

### Storage used:
- [x] KV (key-value cache/session)
- [ ] D1 (SQLite)
- [ ] R2 (object storage)
- [ ] localStorage / sessionStorage
- [x] External database (Airtable)
- [x] In-memory only (module variables inside injectors during session)

### KV usage:
- **Key naming convention:**
  - Static lists: `{table}_all_v1` (e.g. `categories_all_v1`, `budgets_all_v1`)
  - Period-scoped: `tx:{period}:{bust}` (e.g. `tx:2026-06:14`)
  - Bust counter: `tx:bust` (integer, TTL 24hr)
  - Session tokens: `session_{token}`
  - Auth PIN: `auth_pin`
  - Biz layer: `sales_biz_v1`
- **TTL strategy:** categories=3600s, liabilities=1800s, budgets=600s, projects=300s, transactions=300s, sales_biz=600s
- **Cache invalidation approach:** Always on write (POST/PATCH/DELETE), never on read. `?refresh=1` bypass available on every endpoint.
- **Biggest KV mistake made:** Early sessions used list+delete for cache invalidation on transactions (1 list + N deletes per POST). Replaced with bust counter pattern — 2 KV ops per POST regardless of how many cached keys exist.

### Database usage:
- **Which database and why:** Airtable — chosen for zero migration overhead, visual data review by owner, API-ready from day 1.
- **Free tier limits hit:** YES — ~1,000 API calls per 2 weeks. Hit at session 9.
- **How managed:** KV cache reduced Airtable calls from ~50/day to ~7/day on warm cache. ~85% reduction. All heavy GET endpoints are cache-first.
- **Biggest database mistake made:** Airtable ARRAYJOIN on linked record fields returns **names** not **IDs** — caused linked record lookups to fail silently across multiple sessions before the pattern was documented (RULES-data.md).

### ✅ Memory pattern that worked best:

KV cache-first with bust counter for high-write tables:

```javascript
// GET — read bust counter first, use as cache key suffix
const bust = await env.CHAIJOHN_KV.get('tx:bust').catch(() => '0') || '0';
const KEY  = `tx:${period}:${bust}`;
const cached = await env.CHAIJOHN_KV.get(KEY, { type: 'json' }).catch(() => null);
if (cached) return jsonResponse({ records: cached, cached: true });

// ... fetch from Airtable ...

await env.CHAIJOHN_KV.put(KEY, JSON.stringify(records), { expirationTtl: 300 }).catch(() => {});

// POST — increment bust (old keys expire by TTL naturally — no list+delete needed)
const cur = parseInt(await env.CHAIJOHN_KV.get('tx:bust').catch(() => '0') || '0');
await env.CHAIJOHN_KV.put('tx:bust', String(cur + 1), { expirationTtl: 86400 }).catch(() => {});
```

### ⚠️ Memory anti-pattern discovered:

```javascript
// NEVER — costs 1 list op + N delete ops per write. Destroys KV write budget (1,000/day limit).
const keys = await env.CHAIJOHN_KV.list({ prefix: 'tx:' });
for (const key of keys.keys) {
  await env.CHAIJOHN_KV.delete(key.name);
}
```

---

## 4. API STRATEGY

> How were server-side APIs designed and called?

### API file structure:
```
functions/
├── _middleware.js              — auth guard for all /api/* routes
├── _airtable.js               — ALL shared helpers (listRecords, createRecord, patch, jsonResponse, errorResponse)
└── api/
    ├── {resource}.js          — GET list + POST create
    └── {resource}/[id].js     — GET detail + PATCH + DELETE
```

### Patterns used:
- [x] Shared helper functions (`_airtable.js`)
- [x] Input validation at API layer (guard clauses before any Airtable call)
- [x] Consistent response shape (`{ records: [] }` always — never bare arrays)
- [x] Error response helper (`errorResponse()`)
- [x] Idempotent writes (name dedup with 409 before create)
- [x] Guard against double-submit (`isSubmitting` on all save buttons)
- [x] Pagination for large datasets (`listAllRecords` for tables >100 rows)

### ✅ API pattern that saved the most bugs:

Secondary auto-creates wrapped individually in try/catch — primary save always returns 201. Secondary failures (phases, milestones, tasks) log to console and appear in `warnings[]` but never block the primary response:

```javascript
const warnings = [];
try { await createProjectPhases(newProjectId); }
catch (e) { warnings.push('phases'); console.error('Phase auto-create failed:', e.message); }

return new Response(JSON.stringify({ ...newProject, warnings }), { status: 201 });
```

### ⚠️ API mistake made more than once:

Returning bare arrays instead of consistent shaped objects:

```javascript
// WRONG — breaks consumers when shape changes
return jsonResponse(records);

// RIGHT — always wrap (L082)
return jsonResponse({ records: records });
```

### Rate limits hit on any external API:
- API: Airtable — Limit: 5 req/sec — Hit: YES (S14, 64 batch writes) — Solution: 210ms sequential delay
- API: Airtable — Limit: 1,000 calls/2wk — Hit: YES (S9) — Solution: KV cache layer (~85% reduction)

---

## 5. SECURITY STRATEGY

### Auth method used:

PIN → SHA-256 + 16-byte random salt → stored as `salt:hash` string in Cloudflare KV.
Session token = 32 random bytes hex → stored in KV with 7-day TTL.
Cookie: `HttpOnly; Secure; SameSite=Strict` — cannot be read by JavaScript in the browser.

### What was protected:
- [x] All API routes behind auth middleware (`_middleware.js` intercepts every `/api/*` call)
- [x] Environment secrets in Cloudflare dashboard (never in code or git)
- [x] Read-only access for business Airtable base
- [x] Input sanitization (guard clauses before any Airtable call)

### ✅ Security decision that worked:

`_middleware.js` as the single auth checkpoint for all `/api/*` routes. Adding a new API file is automatically protected — no per-file auth logic needed. The auth surface is one file.

### ⚠️ Security gap discovered:

PIN brute-force has no lockout mechanism. Acceptable for single-person, private-network use. Not acceptable if multi-user or public-facing — add rate limiting per IP on `/api/auth/verify` before deploying to shared use.

---

## 6. FRONTEND PATTERNS

### Component/panel anatomy used:

Every panel follows the same structure:
1. **Stats strip** — KPI row at top (3–5 key numbers)
2. **Chart** (optional) — stacked bar, line, or SVG visual
3. **Data body** — list / card grid / table

This order is locked. Panels that deviate become inconsistent and harder to scan.

### Event handling approach:
- [x] Event delegation (primary — one listener on stable container element)
- [ ] Direct binding on dynamically created elements (anti-pattern, causes binding loss)
- [ ] Framework-managed

### ✅ UI pattern that scaled well:

Event delegation on the panel container — survives all innerHTML re-renders:

```javascript
// RIGHT — one listener, survives re-render
panelEl().addEventListener('click', e => {
  if (e.target.closest('.btn-edit')) handleEdit(e.target.dataset.id);
  if (e.target.closest('.btn-delete')) handleDelete(e.target.dataset.id);
});
```

### ⚠️ UI anti-pattern that caused bugs:

Direct binding on innerHTML elements — destroyed on every re-render:

```javascript
// WRONG — binding is lost the moment innerHTML is re-rendered
document.querySelectorAll('.btn-edit').forEach(btn => {
  btn.addEventListener('click', handler);
});
```

### CSS approach that worked:

8 CSS custom properties at `:root` cover the entire dark-mode UI. No external CSS framework. No build step. Works in all modern browsers. Panel-level style overrides go in inline `<style>` in `index.html` shell — `global.css` is not loaded on all pages (L042 rule — do not assume global.css is available).

---

## 7. AI INTEGRATION

### AI used for:
- [x] Content generation (Diary AI assist — expand bullet notes to full prose)
- [x] Data extraction (Drop Zone vision — receipt photo → pre-fill transaction form)
- [x] Code assistance (Claude Code — every development session)
- [x] In-app AI features (AI Advisor panel with live financial context injected)
- [x] Structured data generation (Drop Zone returns JSON field extractions)

### ✅ AI integration that added most value:

**Drop Zone** — drag a receipt photo → Claude Vision extracts amount, date, merchant, type → pre-fills the transaction form. One approval click saves to Airtable. Eliminated manual data entry for ~80% of expense capture.

### ⚠️ AI integration mistake:

Early AI Advisor had no financial context injected — Claude gave generic, useless advice. Fix: inject live financial snapshot (net worth, top debts, cashflow trend, budget burn rate) into every AI Advisor system prompt as a structured JSON block. Context = value. An AI prompt without your data is just a search engine.

---

## 8. HUMAN-AI COLLABORATION

### Workflow used:
- [x] Chat role (diagnose) + CC role (execute)

### Prompt structure that worked best:

```
## CC INTRO
New session. Ignore all previous context. You are working on CHAIJOHN OS at:
https://github.com/Csmittee/chaijohn-personal
Before doing anything else, read: 1. CLAUDE.md  2. RULES.md  3. PROJECT_STATE.md

## READ FIRST (before touching any file)
[exact list of files to read — with reason for each]

## FIXES
[numbered bugs, root cause, exact files, exact fix]

## DO NOT TOUCH
[explicit exclusion list]

## MANDATORY (every session)
1. Archive this prompt → docs/prompts/ stamped ✅ COMPLETE
2. Append new rules to RULES.md with next L-number
3. Update PROJECT_STATE.md
4. Commit in correct order, merge to main
```

### Context management:
- How many iterations before context degraded: CC is stateless — each CC session is fresh by design
- Handoff strategy used: `CHAT_HANDOFF_[date]_S[N].md` — what was done, rules added, pending tasks, owner action items — pasted at start of next chat
- Biggest context loss mistake: CC writing from stale context without reading fresh repo files. Fixed: "read before write" is a hard, non-negotiable rule (L010).

### ✅ Collaboration pattern that saved most time:

Chat diagnoses (reads repo → identifies root cause → writes CC prompt with exact file list and fix specification). CC executes from the prompt. Owner QAs live site. No back-and-forth inside CC. One prompt → complete execution → QA → done.

### ⚠️ Collaboration anti-pattern:

Owner acting as messenger between Chat and CC — pasting code from CC back to Chat, or pasting Chat analysis into CC manually. This creates errors and wastes sessions. Owner never touches source files. Owner never relays code. Chat writes the prompt → CC reads it from the repo → result is QA'd by owner directly.

---

## 9. THE 5 BIGGEST BUGS & FIXES

| # | Bug description | Root cause | Fix | Rule to add |
|---|---|---|---|---|
| 1 | Budget batch save 500 errors on 64-record session | `Promise.allSettled` fires all requests simultaneously — exceeds Airtable 5 req/sec ceiling | Sequential `for...of` loop, 210ms delay, `Saving X/N` live progress label, `isSaving` guard | L211 |
| 2 | Dead `category_id` lookups in 3 API files surviving 10 sessions | Early design: `category_id` on transactions for routing. `source` field made it obsolete, but code was never cleaned up | Remove all category lookups from `collection.injector.js`, `hard-assets.injector.js`, `liabilities.js`, `liabilities/[id].js` | L212, L214 |
| 3 | Panel header disappears after injector runs | `p.innerHTML = renderPanel()` replaces the entire panel div including the header written in `index.html` | Render the panel header inside `renderPanel()` — never assume `index.html` header survives after injector runs | L147 |
| 4 | Airtable free tier limit hit at session 9 | No caching — every panel load triggered 3–5 Airtable API calls. 1,000 call limit reached. | KV cache-first pattern on all GET endpoints, bust counter for transactions, TTL-based for static data | L130–L132 |
| 5 | Event bindings lost on re-render across multiple panels | Direct `.addEventListener()` on dynamically created elements — destroyed every time `innerHTML` is re-rendered | Event delegation on stable container element — one listener survives all re-renders | L068 pattern |

---

## 10. THE 5 RULES THIS PROJECT WOULD ADD TO EVERY NEW PROJECT

```
R1: Define your transaction model on day 1. Pick ONE routing field (source/type/category).
    Never let two fields both claim routing authority. Document it in RULES.md from commit 1.
    Any field that becomes redundant must be marked [SUPERSEDED] immediately — do not let
    dead code accumulate across sessions.

R2: KV cache every external API GET from the start. Design your key naming convention
    before writing the first API file. Use bust counter for high-write tables (never
    list+delete). Wrap every KV put/delete in .catch(() => {}) — KV failure must never
    crash the API response.

R3: isSubmitting guard on every save button before writing the first POST call.
    Cold-start double-POST is a guarantee without it, not a risk. The guard costs 3 lines.
    Same pattern applies to any batch write — isSaving guard prevents double-submit.

R4: One injector file per panel. One API file per resource. Shared helpers in one file only.
    Never merge panels. Never merge APIs. Isolation is the architecture — a change to one
    panel must never be able to break another.

R5: Read before write — CC (or any AI) reads all relevant files fresh from repo before
    touching anything. A rule in RULES.md is only enforced if CC reads RULES.md.
    Make the read list explicit and mandatory in every prompt. Never assume prior context.
```

---

## 11. FREE TIER USAGE REPORT

| Service | Free limit | Peak usage | Hit limit? | Solution used |
|---|---|---|---|---|
| Cloudflare Pages builds | 500/mo | ~60/mo | No | Fine — well within |
| Cloudflare KV reads | 100,000/day | ~500/day | No | Well within |
| Cloudflare KV writes | 1,000/day | ~50/day | No | Bust counter pattern keeps this safe |
| Airtable API calls | 1,000 calls/2wk | ~1,000 (S9) | YES | KV cache → ~150 calls/2wk now |
| Anthropic API | Pay-per-use | Drop Zone + AI Advisor | No | Accepted cost — used sparingly |
| Cloudinary | 25 credits/mo | Low | No | Sync-on-demand only, not continuous |

---

## 12. ONE THING TO STEAL FOR EVERY FUTURE PROJECT

**The Chat + CC workflow protocol.** Chat diagnoses by reading the repo. CC executes from a precise, structured prompt. Owner only QAs. This three-role separation means no code is written without a root cause diagnosis, and no diagnosis drives code without a complete, committed, tested execution. The RULES.md as institutional memory — every bug becomes a numbered law — means the project gets smarter every session instead of re-learning the same mistakes. This scales from day 1 to session 214.

---

## 13. ONE THING TO NEVER REPEAT

**Letting a dead pattern accumulate across multiple files for 10+ sessions.** The `category_id` on transactions was obsolete by session 4 but survived in 3 API files until session 14. Every session that passed without cleanup was compounding technical debt. Rule: when a pattern is superseded, mark it `[SUPERSEDED by L-number]` in RULES.md immediately and schedule a cleanup CC session within the next 1–2 sessions. Dead code that survives becomes invisible — it will bite you when you least expect it.

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
      "label": "Chaijohn OS",
      "node_type": "Project",
      "status": "Active",
      "description": "Personal finance + business OS — full-stack single-owner dashboard",
      "tags": "Cloudflare,Airtable,KV,Cloudinary,Anthropic,dashboard,finance",
      "context": "Cloudflare Pages · PIN auth · 14 sessions · injector-per-panel pattern · production"
    },
    {
      "label": "Cloudflare KV",
      "node_type": "Tool",
      "status": "Active",
      "description": "Session store + API cache layer — dual purpose in one binding",
      "tags": "cache,session,serverless,cloudflare",
      "context": "100k reads/day · 1k writes/day · bust counter pattern required for write safety"
    },
    {
      "label": "Airtable",
      "node_type": "Tool",
      "status": "Active",
      "description": "Primary database — dual base personal + business read-only",
      "tags": "database,nocode,api,spreadsheet",
      "context": "1000 calls/2wk free tier · hit at session 9 · KV cache is the required protection"
    },
    {
      "label": "Cloudflare Pages Functions",
      "node_type": "Tool",
      "status": "Active",
      "description": "Serverless API layer — one file per resource with shared helpers",
      "tags": "serverless,hosting,api,cloudflare",
      "context": "Free tier generous · cold start = double-POST risk · isSubmitting guard required always"
    },
    {
      "label": "Cloudinary",
      "node_type": "Tool",
      "status": "Active",
      "description": "Media hosting for collection assets — sync-on-demand strategy",
      "tags": "media,images,cdn,upload",
      "context": "25 credits/mo free · sync-on-demand avoids credit waste"
    },
    {
      "label": "Anthropic API",
      "node_type": "Tool",
      "status": "Active",
      "description": "AI Advisor context injection + Drop Zone vision extraction",
      "tags": "ai,llm,vision,claude",
      "context": "Pay-per-use · financial context injection required for value · structured JSON output pattern"
    },
    {
      "label": "Injector per panel",
      "node_type": "Pattern",
      "status": "Active",
      "description": "One IIFE JS file per UI panel — lazy-init on panelactivated event",
      "tags": "frontend,architecture,javascript,isolation,spa",
      "context": "Use for: multi-panel SPA · Not for: shared state across panels · replaces: monolithic JS bundle"
    },
    {
      "label": "KV bust counter cache",
      "node_type": "Pattern",
      "status": "Active",
      "description": "Cache invalidation via integer counter — avoids list+delete KV ops",
      "tags": "cache,kv,cloudflare,invalidation,performance",
      "context": "Use when: high-write tables need caching · Not for: near-static data (TTL only) · replaces: list+delete pattern"
    },
    {
      "label": "Source-field transaction routing",
      "node_type": "Pattern",
      "status": "Active",
      "description": "Single source field routes all transactions — no secondary routing field",
      "tags": "data-model,transactions,airtable,architecture",
      "context": "Use for: any money-flow system · Not when: truly multi-dimensional routing needed · replaces: category_id routing"
    },
    {
      "label": "Chat + CC workflow",
      "node_type": "Pattern",
      "status": "Active",
      "description": "Chat diagnoses from repo, CC executes from prompt, owner only QAs",
      "tags": "workflow,ai,collaboration,claude,human-ai",
      "context": "Use for: all dev sessions · Not for: one-line trivial fixes · replaces: ad-hoc direct prompting"
    },
    {
      "label": "Sequential batch write",
      "node_type": "Pattern",
      "status": "Active",
      "description": "for...of loop with 210ms delay for multi-record API writes",
      "tags": "airtable,performance,ratelimit,batch,write",
      "context": "Use when: writing 5+ records to rate-limited API · Not for: reads · replaces: Promise.allSettled writes"
    },
    {
      "label": "RULES.md as institutional memory",
      "node_type": "Pattern",
      "status": "Active",
      "description": "Every bug becomes a numbered law — project gets smarter each session",
      "tags": "workflow,documentation,rules,ai,memory",
      "context": "Use from day 1 · Number rules sequentially · Mark superseded rules — never delete · CC reads before every session"
    }
  ],
  "edges": [
    {
      "from": "Chaijohn OS",
      "to": "Airtable",
      "edge_type": "uses",
      "label": "primary database — dual base",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Cloudflare KV",
      "edge_type": "uses",
      "label": "session store + API cache",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Cloudflare Pages Functions",
      "edge_type": "uses",
      "label": "serverless API layer",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Cloudinary",
      "edge_type": "uses",
      "label": "media hosting for collection assets",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Anthropic API",
      "edge_type": "uses",
      "label": "AI advisor + Drop Zone vision",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Injector per panel",
      "edge_type": "uses",
      "label": "primary frontend architecture",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Chat + CC workflow",
      "edge_type": "uses",
      "label": "human-AI collaboration protocol — every session",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "Source-field transaction routing",
      "edge_type": "uses",
      "label": "sole transaction routing mechanism",
      "strength": 1
    },
    {
      "from": "Chaijohn OS",
      "to": "RULES.md as institutional memory",
      "edge_type": "uses",
      "label": "L001-L214 — project law library",
      "strength": 1
    },
    {
      "from": "KV bust counter cache",
      "to": "Cloudflare KV",
      "edge_type": "requires",
      "label": "runs on KV binding",
      "strength": 1
    },
    {
      "from": "KV bust counter cache",
      "to": "Airtable",
      "edge_type": "supports",
      "label": "protects Airtable free tier from exhaustion",
      "strength": 1
    },
    {
      "from": "Sequential batch write",
      "to": "Airtable",
      "edge_type": "supports",
      "label": "respects 5 req/sec rate ceiling",
      "strength": 1
    },
    {
      "from": "Injector per panel",
      "to": "Cloudflare Pages Functions",
      "edge_type": "requires",
      "label": "fetches data from API layer on panelactivated",
      "strength": 1
    },
    {
      "from": "Chat + CC workflow",
      "to": "Anthropic API",
      "edge_type": "uses",
      "label": "Claude Code as the CC execution role",
      "strength": 1
    },
    {
      "from": "Chat + CC workflow",
      "to": "RULES.md as institutional memory",
      "edge_type": "requires",
      "label": "CC reads RULES.md before every session",
      "strength": 1
    },
    {
      "from": "Injector per panel",
      "to": "KV bust counter cache",
      "edge_type": "uses",
      "label": "injectors trigger cache invalidation on write",
      "strength": 1
    }
  ]
}
```
