# PLAYBOOK PHASE 2 — STRATEGIC NOTES
> Insert into neutral chat during Phase 2 compilation
> Author: Chat session S15 — 2026-06-11
> Purpose: Architectural additions and future scope guidance for the compiler

---

## INSTRUCTION TO NEUTRAL CHAT (Phase 2 compiler)

When compiling all contributions into PLAYBOOK_DRAFT.md, incorporate the following
strategic notes alongside the extracted patterns. These are NOT project-specific —
they are architectural decisions about how the Playbook itself should be structured
and used. Insert them into the appropriate sections of the draft.

---

## 1. HIERARCHY OF TRUTH — add to Playbook introduction

The Playbook governs a multi-layer system. When layers conflict, this priority order wins:

```
Layer 1 — RULES.md (project-level)
  Wins for: anything specific to that project
  Read: every CC session, every Chat session for that project

Layer 2 — Playbook.md (universal constitutional law)
  Wins for: any decision not covered by project-level rules
  Read: first time a new project opens, and once more before project closes

Layer 3 — Mind Map (cross-project knowledge graph)
  Wins for: strategy, relationships, what already exists across all projects
  Query: at project start to find reusable patterns, tools, people, decisions

Layer 4 — PROJECT_STATE.md (current state snapshot)
  Wins for: what files exist right now, what is working, what is broken
  Read: every CC session

Layer 5 — CHAT_HANDOFF.md (session continuity)
  Wins for: what happened in the last session, what is pending
  Read: first message of every new chat session
```

**Rule:** Never contradict a higher layer without an explicit documented reason.
When a project-level rule upgrades to universal law, it moves from RULES.md into
the Playbook via the audit process — not by direct edit.

---

## 2. PLAYBOOK INTRODUCTION POINT — when to read it

The Playbook is introduced exactly twice per project:

1. **At project start** — before any code is written. Chat reads it to orient
   architecture decisions, tooling choices, and collaboration patterns.

2. **Before project closes (~90% complete)** — full audit pass. Check what
   the project confirms, contradicts, or adds to the Playbook. This is the
   amendment trigger.

It is NOT read every session. It is constitutional — you read the constitution
when you start a country and when you amend it, not every morning.

---

## 3. MIND MAP AS AGENT QUERY SURFACE — add to Section on Memory Strategy

The Mind Map is currently a visual tool for the owner. To serve agents and
future automation, it needs a query contract — not just a list endpoint.

**Recommended API additions:**

```
GET /api/mindmap-nodes?tags=cache,cloudflare     → find nodes by tag keywords
GET /api/mindmap-nodes?node_type=Pattern          → find all patterns
GET /api/mindmap-nodes?node_type=Tool             → find all tools
GET /api/mindmap-edges?from_label=Chaijohn OS     → find everything a project connects to
GET /api/mindmap-nodes?query=batch+write          → free text search on label+description
```

This turns the Mind Map from a visual into a **queryable memory system**.
Any new Chat, CC session, or future agent can call these endpoints to orient
itself without reading 800 lines of context. This is the foundation of the
agent memory layer.

---

## 4. PLAYBOOK AMENDMENT PROCESS — add to Playbook appendix

A constitution without an amendment process becomes outdated and ignored.

**Amendment triggers:**
- Every 3 new projects completed → re-run Phase 1 contributions + re-compile
- Any major architectural decision that contradicts existing Playbook rules
- Any pattern confirmed in 3+ projects that is not yet in the Playbook
- Any anti-pattern that caused a bug in 2+ projects

**Amendment process:**
1. Re-run Phase 1 contribution for affected projects (Section 10 + 12 + 13 only — not full rescan)
2. Neutral chat compares new contributions against current Playbook
3. Rules that upgrade: move from RULES.md → Playbook with `(confirmed in N projects)` citation
4. Rules that are superseded: mark `[SUPERSEDED vX.X — date — reason]` — never delete
5. Version the Playbook: `PLAYBOOK_v1.md`, `PLAYBOOK_v2.md` — keep all versions in repo

---

## 5. SCOPE EXPANSION — three horizons

The Playbook currently covers Horizon 1. Design it to expand into Horizons 2 and 3
without a full rewrite.

### Horizon 1 — Coding Development (NOW)
What owner, Chat, and CC do together.
- Architecture patterns, API design, caching, security, frontend, human-AI workflow
- File: `PLAYBOOK.md` (current scope)
- Tools: CLAUDE.md, RULES.md, PROJECT_STATE.md, Mind Map

### Horizon 2 — Agentic AI (NEXT — 6-18 months)
When CC is replaced or augmented by agent swarms, QA agents, and automated loops.

**New documents needed at this horizon:**
- `AGENT_ROLES.md` — what each agent is authorized to do, read, write, decide
- `HOOKS.md` — trigger definitions: what event fires which agent (PR opened → QA agent, Airtable field error → schema agent)
- `SECURITY.md` (expanded) — agent authorization boundaries, what agents can never do without human approval (merge to main, delete records, spend money)
- `WORKFLOW_LOOPS.md` — goal → decompose → assign → execute → verify → report loop definition per project type

**The Playbook survives this horizon** — it becomes Section 1 (foundation) of a
larger Agent Constitution. The coding patterns do not change; the execution layer
does.

### Horizon 3 — Machine Learning (FUTURE)
When the system begins training on its own outputs.

**New documents needed:**
- `TRAINING_DATA_RULES.md` — what sessions, outputs, and decisions are eligible for training data
- `MODEL_REGISTRY.md` — which fine-tuned models exist, what they know, when they were trained
- `FEEDBACK_LOOP.md` — how production outcomes feed back into training signal

**At this horizon, the Playbook becomes input data** — not just a governance document
but a structured corpus that a model can learn the owner's decision style from.
The Mind Map becomes the knowledge graph that a model navigates at inference time.

**The Playbook does not need to be replaced at any horizon** — it grows by adding
new sections and new files, never by rewriting the foundation.

---

## 6. FULL LOOP SYSTEM — FOUNDATION REQUIREMENTS

For a future session where CC and agents work without the owner in the loop:

### What is genuinely automatable today:
- Code generation from a precise prompt
- File reading, diagnosis, writing, committing
- RULES.md + PROJECT_STATE.md updates after each fix
- PR creation, prompt archiving

### What still requires the owner (honest assessment):

| Task | Why owner is still needed | Future solution |
|---|---|---|
| Airtable field inspection | Agent cannot see actual field types without API call — and API can be wrong vs what was manually set in UI | Airtable Meta API read at session start — auto-verify field types before writing |
| Visual QA on live page | Agent cannot see rendered UI, spot misaligned elements, or feel UX friction | Screenshot capture tool + vision model review (not available yet in CC) |
| Approve PR | Trust boundary — merging to production requires human confirmation | Could be automated for fix branches if QA checklist passes automatically |
| Answer 3 questions before prompt | Architecture decisions require owner's intent, not just code state | Pre-session survey form that owner fills in 2 minutes — structured not conversational |
| Bug found by using real page | Lived interaction reveals bugs that code review misses | Playwright/Puppeteer agent running test scripts against live site |

### Minimum foundation for a working loop without the owner:

```
1. Structured goal input        ← owner fills a short form (goal, constraints, files to touch)
2. CC reads goal + repo         ← standard session start
3. CC executes + self-QA        ← runs its own checklist before committing
4. QA agent reviews diff        ← checks against RULES.md violations, pattern compliance
5. Auto-merge if QA passes      ← only for pre-approved branch types (fix/*, optim/*)
6. Owner notified async         ← summary report, not a blocker
7. Owner reviews at leisure     ← approves feat/* branches manually
```

**The honest limit:** Until a vision model can inspect a live rendered page and
compare it against a design spec, the owner remains the final QA for anything
visual. That is not a weakness — that is appropriate trust boundary design.

---

## 7. REFLECTION — 8 WEEKS TO SESSION 14

This note belongs in the Playbook introduction as context for whoever reads it next.

What changed between week 1 and week 13:

| Week 1-4 | Week 13 |
|---|---|
| Owner writes code manually | CC writes complete files from prompt |
| Files uploaded and downloaded to show Chat | Chat reads repo directly |
| Context rebuilt from scratch each session | HANDOFF doc + project knowledge restores full context in one message |
| Owner copy-pastes between Chat and CC | Owner never touches source files |
| Lessons learned informally | Every bug becomes a numbered law in RULES.md |
| One repo, one roadmap | Multiple connected repos, shared memory layer |

The velocity increase is not from better prompts. It is from **better architecture
around the human-AI collaboration** — the roles, the documents, the loops, the rules.

The Playbook is the formalization of that architecture. Its purpose is to make
the next project start at week 13 velocity, not week 1.
