# PLAYBOOK_CREATION_PROMPT.md
> Use this in a NEW chat session to generate PLAYBOOK.md
> Scan the current repo + all lessons learned + create a reusable framework

---

## HOW TO USE THIS

Open a new Claude chat. Paste this prompt. Let it scan the repo and generate PLAYBOOK.md.
The output must be GENERAL — applicable to any Cloudflare Pages + KV + Airtable app,
not specific to CHAIJOHN OS details (no budget amounts, no personal data).

---

## PROMPT TO PASTE

```
You are helping me create a PLAYBOOK.md — a reusable developer skill library.

I have built a personal finance + business OS dashboard over 21+ iterations.
Scan these files from my repo https://github.com/Csmittee/chaijohn-personal:
- RULES.md (L001–L132) — all lessons learned
- CLAUDE.md — project constitution and constraints  
- WORKFLOW_SKILL.md — human-AI collaboration protocol
- PROJECT_STATE.md — architecture and file inventory
- functions/api/assets.js — reference for KV cache pattern
- functions/_airtable.js — shared Airtable helper pattern
- public/assets/js/expenses.injector.js — reference injector pattern

Then generate PLAYBOOK.md following the table of contents and guidelines below.
Write it as GENERAL best practices — strip all personal/financial specifics.
Reference patterns and anti-patterns. Make it useful for any new app project.
```

---

## TABLE OF CONTENTS FOR PLAYBOOK.md

```markdown
# PLAYBOOK.md — App Builder's Skill Library
> Distilled from 21+ app iterations. Stack: Cloudflare Pages + KV + Airtable + Cloudinary.
> General patterns applicable to any modern serverless web app.

## 1. PROJECT CONSTITUTION
   1.1 The 5 rules every project needs in CLAUDE.md
   1.2 How to write a PROJECT_STATE.md that survives 20+ iterations
   1.3 RULES.md as a living bug pattern library — naming convention (L001, L002...)
   1.4 The Chat + CC workflow — who does what, why it works

## 2. ARCHITECTURE STRATEGY
   2.1 Injector pattern — one JS file per panel, lazy-init on panelactivated
   2.2 Hash routing — panels as #panel-name, no page reloads
   2.3 API file structure — one file per resource, shared helpers in _airtable.js
   2.4 Environment variables — what goes in Cloudflare dashboard vs code
   2.5 When to use KV vs D1 vs R2 — decision matrix

## 3. MEMORY STRATEGY
   3.1 KV as session + cache layer — key naming conventions
   3.2 TTL decisions by data volatility (static=3600s, slow=1800s, fast=300s)
   3.3 Cache invalidation — always invalidate on write, never on read
   3.4 KV key patterns: {table}_all_v1, {table}:{period}, {table}:{id}
   3.5 Force-refresh pattern — ?refresh=1 bypass
   3.6 What NOT to store in KV (large blobs, user PII, logs)

## 4. DATABASE STRATEGY
   4.1 Airtable as the source of truth — when it's right, when it's wrong
   4.2 Airtable free tier management — KV cache to stay under 1000 calls/2wk
   4.3 Linked record fields — ARRAYJOIN returns names not IDs (anti-pattern trap)
   4.4 listAllRecords vs listRecords — always paginate for tables >100 rows
   4.5 D1 for public/structural data — stocks, logs, MAC registry, heartbeats
   4.6 R2 for large files — images, PDFs, exports (don't use for JSON < 1MB → use KV)
   4.7 Cloudinary for media — upload pattern, transformation URLs, gallery sync

## 5. API STRATEGY
   5.1 API file naming — resources.js + resources/[id].js pattern
   5.2 Always return { records: [] } shape — never bare arrays
   5.3 jsonResponse() + errorResponse() helpers — never raw Response objects
   5.4 Guard clauses first — validate input before any Airtable call
   5.5 isSubmitting guard — prevent double-POST on slow cold starts
   5.6 Idempotent writes — dedup check before create (name/date/amount match)
   5.7 Try/catch wrapping — primary save always returns, secondary auto-creates swallowed
   5.8 POST dedup — 409 before create on name collision

## 6. SECURITY STRATEGY
   6.1 PIN auth + KV session pattern — how to implement without a database
   6.2 Middleware guard — _middleware.js checks session on every /api/* route
   6.3 Environment secrets — never in code, always in Cloudflare dashboard
   6.4 Read-only consumers — how to read a foreign Airtable base safely
   6.5 CORS and trusted origins — Cloudflare Pages default behavior

## 7. FRONTEND PATTERNS
   7.1 CSS variable system — the 8 variables every dark-mode app needs
   7.2 Panel anatomy — stats strip → chart → data body (always this order)
   7.3 Injector init pattern — panelactivated event, loadAll(), renderAll()
   7.4 Event delegation — ONE listener on container, never on innerHTML elements
   7.5 Cache-first rendering — load once, filter client-side, never re-fetch for UI changes
   7.6 Drawer pattern — fixed position, transform translateX, backdrop overlay
   7.7 Collapsible sections — state in module-level object, delegation not direct binding
   7.8 Mobile-first considerations — min-width, touch targets, sidebar collapse

## 8. AI INTEGRATION STRATEGY
   8.1 Anthropic API in Cloudflare Workers — streaming vs buffered (when to use each)
   8.2 Structured output from AI — prompt for JSON only, parse with try/catch, fallback to text
   8.3 AI-powered data entry — generate → preview → confirm → POST pattern
   8.4 Context injection — send relevant app data with every AI prompt
   8.5 Session management for multi-turn AI chat

## 9. EXPORT STRATEGY
   9.1 PDF via window.print() — @media print CSS, inject header, remove after
   9.2 ODS/XLSX via SheetJS — bookType:'ods' for LibreOffice, client-side only
   9.3 Review note pattern — add to PDF header for stakeholder sharing
   9.4 When to use client-side vs server-side export

## 10. HUMAN-AI COLLABORATION (THE META-SKILL)
   10.1 Chat role vs CC role — diagnose vs execute
   10.2 Prompt template structure — CC INTRO + READ FIRST + FIXES + DO NOT TOUCH + MANDATORY
   10.3 Branch naming convention — fix/batch-N, feat/feature-name, optim/what
   10.4 QA checklist as contract — PR not merged until all boxes checked
   10.5 RULES.md as institutional memory — every bug becomes a law
   10.6 Handoff document — what to include, why context continuity matters
   10.7 Token budget awareness — when to write prompts vs when to execute
   10.8 When to start a new chat — context degradation signals

## 11. COMMON BUG PATTERNS & FIXES
   11.1 Airtable 403 = table doesn't exist, not permissions
   11.2 ARRAYJOIN on linked field = names not IDs
   11.3 innerHTML binding lost on re-render → use delegation
   11.4 Cold start double-POST → isSubmitting guard
   11.5 SSE stream vs JSON response → ?stream=false pattern
   11.6 KV cache serving stale data → always invalidate on write
   11.7 Phase auto-create with blank name → always set name field explicitly
   11.8 Linked record array vs text string → linkedId() helper pattern

## 12. CLOUDFLARE FREE TIER — STAYING WITHIN LIMITS
   12.1 Pages: CPU time, requests, build minutes — current usage patterns
   12.2 KV: 100k reads/day, 1k writes/day — how caching keeps you under
   12.3 Workers: 100k requests/day — shared with Pages Functions
   12.4 D1: 5M reads/day, 100k writes/day — generous, use for logs
   12.5 R2: 10GB storage, 1M Class B ops/month — use for media only
   12.6 Cloudinary free: 25 credits/month — sync strategy to avoid waste
   12.7 Airtable free: 1000 API calls/2wk — KV cache is the solution

## APPENDIX
   A. Environment variable checklist (what every new project needs)
   B. File structure template for new Cloudflare Pages app
   C. CSS variable starter kit (dark mode, 8 variables)
   D. Injector template (copy-paste starter)
   E. API file template (copy-paste starter with KV cache built in)
   F. CLAUDE.md template for new projects
   G. RULES.md starter (first 10 universal rules)
```

---

## GUIDELINES FOR THE AI GENERATING PLAYBOOK.md

- Write each section as **pattern + anti-pattern + example**
- Keep examples CODE-LEVEL but generic (use `{tableName}`, `{resourceId}` not real names)
- Each section max 300 words — this is a reference, not a tutorial
- Anti-patterns in red-flag boxes: `⚠️ NEVER: ...`
- Patterns in green boxes: `✅ ALWAYS: ...`
- Where a RULES.md L-number applies, cite it: `(→ L052d)`
- Appendix items must be copy-paste ready templates
- Total length target: 2,000–3,000 lines — comprehensive but scannable
- Tone: direct, technical, no fluff — written for someone who already codes

---

## OUTPUT

Save as `PLAYBOOK.md` in the repo root.
Commit: `docs: PLAYBOOK.md — reusable app builder skill library v1`
