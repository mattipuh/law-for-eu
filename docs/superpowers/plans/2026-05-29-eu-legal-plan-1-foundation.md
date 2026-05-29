# eu-legal Plugin — Plan 1: Foundation + Working Demo

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold the `eu-legal` plugin with a unified cold-start, 4 regulatory skills backed by the Velvoite MCP, and a marketplace entry — producing a working demo where a EU financial institution can run setup and immediately see their DORA obligations, upcoming deadlines, and recent regulatory changes.

**Architecture:** Single plugin directory `eu-legal/` at the repo root. Skills are flat under `skills/<name>/SKILL.md`. One shared config written by `cold-start-interview` to `~/.claude/plugins/config/eu-legal/CLAUDE.md`. Velvoite MCP declared in `.mcp.json` — regulatory skills call it directly when `VELVOITE_API_KEY` is set, degrade gracefully to general guidance without it.

**Tech stack:** Claude plugin format (SKILL.md with YAML frontmatter + markdown body), `plugin.json` (JSON), `.mcp.json` (JSON). Validation: `claude plugin validate eu-legal`. No build step.

**This plan covers:** Plugin scaffold, `cold-start-interview`, `customize`, `obligations`, `deadlines`, `reg-watch`, `reg-gap-analysis`, `marketplace.json` entry.

**Plans 2–4:** Privacy skills (Plan 2), Commercial skills (Plan 3), Employment + Corporate (Plan 4). This plan produces a standalone working demo without them.

---

## Files created in this plan

```
eu-legal/
  .claude-plugin/plugin.json
  .mcp.json
  CLAUDE.md
  .gitignore
  hooks/hooks.json
  skills/
    cold-start-interview/SKILL.md
    customize/SKILL.md
    obligations/SKILL.md
    deadlines/SKILL.md
    reg-watch/SKILL.md
    reg-gap-analysis/SKILL.md
.claude-plugin/marketplace.json          ← modified (add eu-legal entry)
```

---

## Task 1: Plugin scaffold — directory structure + plugin.json + .mcp.json + hooks.json + .gitignore

**Files:**
- Create: `eu-legal/.claude-plugin/plugin.json`
- Create: `eu-legal/.mcp.json`
- Create: `eu-legal/hooks/hooks.json`
- Create: `eu-legal/.gitignore`

- [ ] **Step 1: Create directory structure**

```bash
mkdir -p eu-legal/.claude-plugin
mkdir -p eu-legal/hooks
mkdir -p eu-legal/skills
```

- [ ] **Step 2: Write `eu-legal/.claude-plugin/plugin.json`**

```json
{
  "name": "eu-legal",
  "version": "0.1.0",
  "description": "EU legal operations for regulated financial institutions — commercial contracts, privacy/GDPR, AI Act, employment, corporate governance, and regulatory compliance powered by the Velvoite EU finreg corpus.",
  "author": {
    "name": "Silly Pilot Oy",
    "url": "https://velvoite.eu"
  }
}
```

- [ ] **Step 3: Write `eu-legal/.mcp.json`**

```json
{
  "mcpServers": {
    "velvoite": {
      "type": "http",
      "url": "https://mcp.velvoite.eu/mcp",
      "title": "Velvoite",
      "description": "EU financial regulation corpus — obligations, deadlines, enforcement, regulatory changes across DORA, MiCA, GDPR, AI Act, and more.",
      "headers": {
        "X-API-Key": "${VELVOITE_API_KEY}"
      }
    },
    "Slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "title": "Slack",
      "description": "Post digests and alerts to Slack channels."
    },
    "Google Drive": {
      "type": "http",
      "url": "https://drivemcp.googleapis.com/mcp/v1",
      "title": "Google Drive",
      "description": "Search and read legal documents from Google Drive."
    }
  },
  "recommendedCategories": [
    "regulatory-intelligence",
    "legal-research",
    "financial-services",
    "documents",
    "chat"
  ]
}
```

- [ ] **Step 4: Write `eu-legal/hooks/hooks.json`**

```json
{"hooks": {}}
```

- [ ] **Step 5: Write `eu-legal/.gitignore`**

```
.DS_Store
*.skill
/logs/
/outputs/
/.claude/
__pycache__/
*.pyc
```

- [ ] **Step 6: Verify all files exist**

```bash
find eu-legal -not -path '*/.git/*' | sort
```

Expected output:
```
eu-legal
eu-legal/.claude-plugin
eu-legal/.claude-plugin/plugin.json
eu-legal/.gitignore
eu-legal/hooks
eu-legal/hooks/hooks.json
eu-legal/.mcp.json
eu-legal/skills
```

---

## Task 2: CLAUDE.md template

**Files:**
- Create: `eu-legal/CLAUDE.md`

This is the practice profile template — ships with the plugin, replaced on updates. Skills read from `~/.claude/plugins/config/eu-legal/CLAUDE.md` (the user's config), never from this file. The comment block at the top makes that explicit.

- [ ] **Step 1: Write `eu-legal/CLAUDE.md`**

```markdown
<!--
CONFIGURATION LOCATION

User-specific configuration lives at a version-independent path:
  ~/.claude/plugins/config/eu-legal/CLAUDE.md

Rules for every skill in this plugin:
1. READ from that path. Not from this file.
2. If that file is missing or contains [PLACEHOLDER] markers, STOP. Say:
   "This plugin needs setup. Run /eu-legal:cold-start-interview — it takes about
   10 minutes and every skill depends on it."
3. cold-start-interview WRITES to that path, creating parent dirs as needed.
4. On first run after a plugin update, if a populated CLAUDE.md exists at the
   old cache path (~/.claude/plugins/cache/eu-legal/<version>/CLAUDE.md) but
   not at the config path, copy it forward before proceeding.
5. This file is the TEMPLATE. It ships with the plugin and is replaced on every
   plugin update. Never write user data here.
-->

# EU Legal Practice Profile
*Written by the cold-start interview. Run `/eu-legal:cold-start-interview` to set up.*

---

## Company

**Name:** [PLACEHOLDER]
**Entity type:** [PLACEHOLDER — credit_institution | payment_institution | e_money_institution | casp | investment_firm | aifm | ucits | insurance | reinsurance | other]
**Primary jurisdiction:** [PLACEHOLDER — e.g., Finland, Germany, Netherlands]
**Operating countries:** [PLACEHOLDER — EU member states where the company operates]
**Legal team:** [PLACEHOLDER — Solo GC | Small team 2–5 | Team with outside counsel]
**Outside counsel:** [PLACEHOLDER — Name/firm or N/A]

---

## Regulatory posture

**Velvoite workspace:** [PLACEHOLDER — workspace ID or N/A]
**Velvoite API key env var:** VELVOITE_API_KEY

**Actor roles per regulation:**

| Regulation | Actor role |
|---|---|
| GDPR | [PLACEHOLDER — controller \| processor \| joint_controller] |
| DORA | [PLACEHOLDER — financial_entity \| ict_third_party \| critical_ict \| N/A] |
| AI Act | [PLACEHOLDER — provider \| deployer \| distributor \| N/A] |
| MiCA | [PLACEHOLDER — casp \| issuer \| N/A] |
| AML/CFT | [PLACEHOLDER — obliged_entity \| N/A] |
| MiFID II | [PLACEHOLDER — investment_firm \| N/A] |

---

## Active practice areas

[PLACEHOLDER — mark which apply:]
- [ ] Commercial contracts
- [ ] Privacy / GDPR + AI Act
- [ ] Employment
- [ ] Corporate governance
- [x] Regulatory

---

## Available integrations

| Integration | Status | Notes |
|---|---|---|
| Velvoite MCP | [PLACEHOLDER ✓/✗] | Required for corpus-backed regulatory skills |
| Slack | [PLACEHOLDER ✓/✗] | For digest and alert posting |
| Google Drive | [PLACEHOLDER ✓/✗] | For document read access |

*Re-check: `/eu-legal:cold-start-interview --check-integrations`*
```

- [ ] **Step 2: Verify file created**

```bash
head -5 eu-legal/CLAUDE.md
```

Expected: the `<!--` comment line as first line.

---

## Task 3: Validate scaffold and commit

**Files:** None new — validation only.

- [ ] **Step 1: JSON sanity check**

```bash
python3 -c "
import json, glob
files = glob.glob('eu-legal/**/*.json', recursive=True)
for f in files:
    json.load(open(f))
    print(f'OK: {f}')
"
```

Expected: three `OK:` lines for plugin.json, .mcp.json, hooks.json.

- [ ] **Step 2: Plugin validation**

```bash
claude plugin validate eu-legal
```

Expected: validation passes. If it fails with "no skills found" — that's expected at this stage; the scaffold is valid, skills come in later tasks.

- [ ] **Step 3: Commit**

```bash
git add eu-legal/
git commit -m "feat(eu-legal): scaffold plugin — plugin.json, .mcp.json, CLAUDE.md template"
```

---

## Task 4: cold-start-interview skill

**Files:**
- Create: `eu-legal/skills/cold-start-interview/SKILL.md`

This is the most important skill — everything else gates on the profile it writes. It follows the Velvoite probe pattern from `eu-finreg:setup` (checks `VELVOITE_API_KEY`, calls `get_company_profile()`, imports actor roles on success) and the interview pattern from `privacy-legal:cold-start-interview`.

- [ ] **Step 1: Create directory and write `eu-legal/skills/cold-start-interview/SKILL.md`**

```markdown
---
name: cold-start-interview
description: >
  Set up the eu-legal plugin — learns your company profile, entity type,
  jurisdiction, and regulatory posture, then writes the practice configuration.
  Run on first install, when CLAUDE.md is missing or has placeholders, or when
  your company profile changes. Use --redo to force a re-run, or
  --check-integrations to re-probe Velvoite, Slack, and Drive only.
argument-hint: "[--redo] [--check-integrations]"
---

# /eu-legal:cold-start-interview

1. Check `~/.claude/plugins/config/eu-legal/CLAUDE.md` — if populated and no `--redo`, confirm before overwriting.
2. Run the interview workflow below.
3. Probe integration availability (Velvoite, Slack, Google Drive).
4. Write `~/.claude/plugins/config/eu-legal/CLAUDE.md` (create parent directories as needed). Show summary. Offer first task.

---

## --check-integrations

Re-probes Velvoite, Slack, and Google Drive. Updates `## Available integrations` only. Does not re-interview. Use after connecting or disconnecting an MCP server.

Report ✓ only if a tool call actually succeeded. Never report ✓ based on `.mcp.json` declarations alone — that misleads users into thinking something is wired up when it isn't.

---

## Migration

On first run after a plugin update: if a populated CLAUDE.md (no `[PLACEHOLDER]` markers) exists at the old cache path (`~/.claude/plugins/cache/eu-legal/<version>/CLAUDE.md` for any version) but not at the config path, copy it to `~/.claude/plugins/config/eu-legal/CLAUDE.md` and tell the user what was migrated. Then offer `--check-integrations` to verify integrations still work.

---

## Interview workflow

Ask questions conversationally — one topic per message. Never dump all questions at once.

### 1. Company basics

Ask:
> "Let's set up eu-legal. What's your company name, and which EU country is your primary legal home — where you're incorporated or where your main regulatory supervisor sits?"

Then ask: which other EU member states does the company operate in? (Optional — press Enter to skip.)

### 2. Entity type

Ask:
> "What type of regulated entity are you? This determines which regulatory obligations surface in your skills."

Present as a numbered list:
1. Credit institution (bank, savings bank, credit union)
2. Payment institution
3. E-money institution
4. CASP — crypto-asset service provider (MiCA)
5. Investment firm (MiFID II)
6. AIFM — alternative investment fund manager
7. UCITS management company
8. Insurance undertaking
9. Reinsurance undertaking
10. Other — general EU company, no FSA supervision

Save the machine-readable code:
| Selection | Code |
|---|---|
| 1 | `credit_institution` |
| 2 | `payment_institution` |
| 3 | `e_money_institution` |
| 4 | `casp` |
| 5 | `investment_firm` |
| 6 | `aifm` |
| 7 | `ucits` |
| 8 | `insurance` |
| 9 | `reinsurance` |
| 10 | `other` |

### 3. Velvoite integration

Ask:
> "Do you have a Velvoite account? (velvoite.eu — free 30-day trial available.) Velvoite provides the EU finreg corpus that powers the regulatory skills. Without it, those skills give general guidance only."

**If yes:** Ask for their workspace ID (found at app.velvoite.eu → Settings → Workspace). Ask which env var holds their API key (default: `VELVOITE_API_KEY`).

Then probe: run `echo "${VELVOITE_API_KEY:+set}"`. If it prints `set`, call `mcp__velvoite__get_company_profile()`.

- **On success:** mark Velvoite ✓. Import actor roles directly from `profile.actor_roles` (a dict of regulation code → list of role codes). Use this mapping for display names in the config:

  | Code | Display name |
  |---|---|
  | `gdpr` | GDPR |
  | `ai_act` | AI Act |
  | `dora` | DORA |
  | `mica` | MiCA |
  | `mifid2` | MiFID II |
  | `aml` | AML/CFT |
  | `crd_crr` | CRD/CRR |
  | `psd` | PSD |
  | `idd` | IDD |
  | `solvency_2` | Solvency II |
  | `sfdr` | SFDR |
  | `emir` | EMIR |
  | `bmr` | BMR |

  Skip step 4 (manual actor role entry) — roles are imported.

- **On failure (env var not set):** Tell the user: "`VELVOITE_API_KEY` is not set. Add it to your `.envrc` (`export VELVOITE_API_KEY=vv_your_key_here`) and run `direnv allow`, then re-run `/eu-legal:cold-start-interview`." Mark Velvoite ✗. Continue interview; set all actor roles to `[PLACEHOLDER]`.
- **On failure (API error):** Tell the user what went wrong in one line and how to fix it (bad key → re-create at app.velvoite.eu/account; no profile → finish workspace setup). Mark Velvoite ✗. Continue.

**If no:** Mark Velvoite ✗. Offer: "No account yet? Free 30-day trial at velvoite.eu." Proceed to step 4.

### 4. Actor roles (only if Velvoite probe did not succeed)

Ask one regulation at a time. Skip any where the entity type makes it clearly inapplicable (e.g., don't ask about MiCA for a credit institution that isn't a CASP).

- **GDPR:** controller / processor / both controller and processor / not applicable
- **DORA:** financial_entity / ict_third_party / critical_ict / not applicable
- **AI Act:** provider / deployer / both provider and deployer / not applicable
- **MiCA:** casp / issuer / not applicable
- **AML/CFT:** obliged_entity / not applicable
- **MiFID II:** investment_firm / not applicable

### 5. Active practice areas

Ask:
> "Which practice areas should be active? I'll suppress skills for areas you don't use."

Show as checkboxes. Default: all checked (user can uncheck).
- [ ] Commercial contracts
- [ ] Privacy / GDPR + AI Act
- [ ] Employment
- [ ] Corporate governance
- [x] Regulatory (always recommended — this is where Velvoite depth lives)

### 6. Legal team

Ask:
> "How is your legal team structured?"

Options:
1. Solo GC or in-house lawyer
2. Small in-house team (2–5 lawyers)
3. In-house team + regular outside counsel

If outside counsel: ask for firm name(s) and jurisdiction/practice area coverage (one question, free text).

### 7. Integration probe

After the interview, probe Slack and Google Drive by attempting a lightweight read on each. Report honestly using ✓ / ✗ / ⚪.

---

## Write the config

Read the CLAUDE.md template from the plugin directory. Replace every `[PLACEHOLDER]` marker with the answer from the interview. Keep all headings, table structure, and comment blocks byte-for-byte identical — other skills parse specific heading text.

Write to `~/.claude/plugins/config/eu-legal/CLAUDE.md` (create parent dirs: `mkdir -p ~/.claude/plugins/config/eu-legal/`).

Show a brief summary of what was written (company name, entity type, Velvoite status, active practice areas).

Then offer:
> "Setup complete. Try:
> - `/eu-legal:obligations` — what regulatory obligations apply to you
> - `/eu-legal:deadlines` — upcoming compliance dates in the next 90 days
> - `/eu-legal:reg-watch` — what changed in EU regulation this week"
```

- [ ] **Step 2: Verify file exists**

```bash
ls eu-legal/skills/cold-start-interview/SKILL.md
```

---

## Task 5: Validate cold-start and commit

- [ ] **Step 1: Validate skill frontmatter**

```bash
claude plugin validate eu-legal
```

Expected: passes. The `description` field is present in frontmatter, `name` matches the directory name — both required by the plugin validator.

- [ ] **Step 2: Commit**

```bash
git add eu-legal/skills/cold-start-interview/
git commit -m "feat(eu-legal): add cold-start-interview skill"
```

---

## Task 6: customize skill

**Files:**
- Create: `eu-legal/skills/customize/SKILL.md`

Adapts the upstream pattern from `privacy-legal/skills/customize/SKILL.md`. Presents the configurable sections of the eu-legal profile and lets the user change one thing without re-running the full interview.

- [ ] **Step 1: Create directory and write `eu-legal/skills/customize/SKILL.md`**

```markdown
---
name: customize
description: >
  Change one thing in your eu-legal profile without re-running the full
  cold-start interview. Adjust entity type, jurisdictions, actor roles,
  active practice areas, Velvoite workspace, legal team, or outside counsel.
  Use when the user says "change my entity type", "update my jurisdiction",
  "add a practice area", "update my profile", or similar.
argument-hint: "[section name, or describe what you want to change]"
---

# /eu-legal:customize

1. Read `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If missing or placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Show the customizable map (see below).
3. Ask what to change. Make the change. Write the updated config back.
4. Confirm what was updated.

---

## Customizable sections

Show the current value for each section, grouped:

**Company identity**
- Entity type — current: [from profile]
- Primary jurisdiction — current: [from profile]
- Operating countries — current: [from profile]

**Regulatory posture**
- Actor roles per regulation — current: [table from profile]
- Velvoite workspace ID — current: [from profile]

**Practice areas**
- Active practice areas — current: [checked list from profile]

**Legal team**
- Team structure — current: [from profile]
- Outside counsel — current: [from profile]

**Integrations**
- Re-probe integrations: suggest `/eu-legal:cold-start-interview --check-integrations`

---

## Making the change

Ask exactly one clarifying question if needed. Then update the relevant section in `~/.claude/plugins/config/eu-legal/CLAUDE.md` using Edit (not a full rewrite). Show a one-line diff of what changed. Confirm: "Updated. All skills will use the new value on their next run."

For actor role changes: offer to re-import from Velvoite if the API key is set — call `mcp__velvoite__get_company_profile()` and update the whole actor roles table.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate eu-legal
git add eu-legal/skills/customize/
git commit -m "feat(eu-legal): add customize skill"
```

---

## Task 7: obligations skill

**Files:**
- Create: `eu-legal/skills/obligations/SKILL.md`

Shows regulatory obligations scoped to the entity's type and actor roles. The demo centrepiece — a credit institution running `/eu-legal:obligations dora` gets their DORA financial_entity obligations from the corpus, sorted by enforcement frequency.

- [ ] **Step 1: Create directory and write `eu-legal/skills/obligations/SKILL.md`**

```markdown
---
name: obligations
description: >
  List the EU regulatory obligations that apply to your entity — scoped to
  your entity type and actor roles. Supports an optional regulation filter.
  Use when asking "what obligations apply to us", "what does DORA require",
  "our MiCA obligations", "show me GDPR duties", or similar.
argument-hint: "[regulation] [limit] — e.g. 'dora' or 'gdpr 20'"
---

# /eu-legal:obligations

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Check Velvoite: if `VELVOITE_API_KEY` not set, run Fallback.
3. Run the workflow below.

---

## Purpose

Show what EU law actually requires — scoped to this entity's type and roles, not a generic list. Sorted by enforcement frequency so the highest-risk obligations surface first.

## Parse args

- `regulation` (optional): one of `dora`, `gdpr`, `mica`, `ai_act`, `aml`, `mifid2`, `crd_crr`, `psd`, `idd`, `solvency_2`, `sfdr`, `emir`, `bmr`. If not provided, query all regulations.
- `limit` (optional integer, default 30): max obligations to show.

## Read from profile

From `~/.claude/plugins/config/eu-legal/CLAUDE.md`:
- Entity type (for context)
- Actor roles table — find the row for the requested regulation, use that role as `actor_role` param

## Workflow

### Step 1: Summary count

Call `mcp__velvoite__get_obligation_summary` with `actor_role` from profile for the requested regulation (omit if querying all).

Show a summary table:
| Regulation | Obligations | Top enforcement area |
|---|---|---|

### Step 2: Obligation detail

Call `mcp__velvoite__get_canonical_obligations` with:
- `regulation`: from args (omit if all)
- `actor_role`: from profile for that regulation
- `limit`: from args

Render as a markdown table, sorted by enforcement count descending:
| # | Regulation | Article | Obligation summary | Enforcement count |
|---|---|---|---|---|

### Step 3: Highlight top 3

Bold the top 3 rows. Add below the table:
> "The top 3 obligations above have the highest enforcement frequency in the Velvoite corpus — prioritise these in your compliance programme."

### Step 4: Related deadlines prompt

If any obligations have upcoming deadlines, note: "Run `/eu-legal:deadlines` to see your compliance calendar."

---

## Fallback (VELVOITE_API_KEY not set)

Show general obligations for the entity type based on public regulation text. Cover: GDPR Art. 5/6/13/15/17/25/32/33, DORA Art. 5-15 (for financial entities), AI Act Art. 10/13/14/15 (for deployers/providers) — whichever apply to the entity type from profile.

Add at the end:
> **Velvoite corpus not connected.** These are public-text obligations. For article-level requirements with enforcement frequency, authority guidance, and obligation-specific corpus answers: add `VELVOITE_API_KEY` to your `.envrc`. Free 30-day trial at velvoite.eu.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate eu-legal
git add eu-legal/skills/obligations/
git commit -m "feat(eu-legal): add obligations skill"
```

---

## Task 8: deadlines skill

**Files:**
- Create: `eu-legal/skills/deadlines/SKILL.md`

- [ ] **Step 1: Create directory and write `eu-legal/skills/deadlines/SKILL.md`**

```markdown
---
name: deadlines
description: >
  Upcoming EU regulatory compliance deadlines for your entity — default 90-day
  horizon, configurable. Use when asking "what's due soon", "compliance
  calendar", "DORA deadlines", "what do we need to do by end of quarter", or
  similar.
argument-hint: "[days] [regulation] — e.g. '30' or '60 dora'"
---

# /eu-legal:deadlines

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Check Velvoite: if `VELVOITE_API_KEY` not set, run Fallback.
3. Run the workflow below.

---

## Parse args

- `days` (optional integer, default 90): look-ahead window.
- `regulation` (optional): filter to a specific regulation code.

## Workflow

Call `mcp__velvoite__get_deadlines` with:
- `days`: from args
- `regulation`: from args if provided
- `include_overdue`: false

Render date-ascending:
| Date | Regulation | Deadline description | Notes |
|---|---|---|---|

Formatting:
- Within 7 days: prefix row with ⚠ and bold the date
- Within 30 days: bold the date
- If more than 8 rows: group by regulation

Add after the table:
> "Source: Velvoite corpus. Verify against the Official Journal and your competent authority's published compliance calendar — deadlines can shift via implementing acts or supervisory guidance."

If no results: say explicitly "No deadlines found in the Velvoite corpus for the next [N] days" and suggest checking EUR-Lex directly for the regulation.

---

## Fallback (VELVOITE_API_KEY not set)

Show key publicly-known EU regulatory dates relevant to the entity type from profile. Mark each `[public information — verify against Official Journal]`. Add:
> **Velvoite corpus not connected.** For live deadline tracking with authority-specific dates: add `VELVOITE_API_KEY` to your `.envrc`. Free 30-day trial at velvoite.eu.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate eu-legal
git add eu-legal/skills/deadlines/
git commit -m "feat(eu-legal): add deadlines skill"
```

---

## Task 9: reg-watch skill

**Files:**
- Create: `eu-legal/skills/reg-watch/SKILL.md`

- [ ] **Step 1: Create directory and write `eu-legal/skills/reg-watch/SKILL.md`**

```markdown
---
name: reg-watch
description: >
  What changed in EU financial regulation recently — new publications,
  guidelines, Q&As, RTS/ITS, supervisory notices — scoped to your entity
  type. Default 7-day lookback. Use when asking "what's new", "any DORA
  updates this week", "what changed in EU reg", "weekly regulatory digest",
  or similar.
argument-hint: "[days] — e.g. '14' for two weeks"
---

# /eu-legal:reg-watch

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Check Velvoite: if not available, run Fallback.
3. Run the workflow below.

---

## Parse args

- `days` (optional integer, default 7): lookback window.

## Workflow

Call `mcp__velvoite__get_recent_changes` with `days` from args.

Group results by regulation (most-active regulation first). For each item render:
- **Title** (linked if URL available)
- Source: EBA / ESMA / EIOPA / FIN-FSA / BaFin / EUR-Lex / Commission / other
- Document type: guideline / Q&A / RTS / ITS / supervisory notice / consultation / decision
- One-line plain-language summary of what it says

Regulations the entity does not participate in (actor role = N/A in profile): show in a collapsed "Other EU regulatory activity" section at the bottom, not in the main body.

If no results for a regulation the entity cares about: "No new publications for [regulation] in the last [N] days." — say this explicitly rather than omitting.

Add at the end:
> "Source: Velvoite corpus (EUR-Lex, EBA, ESMA, EIOPA, FIN-FSA, BaFin). AI-classified — verify at the original source for formal compliance purposes."

---

## Fallback (VELVOITE_API_KEY not set)

Tell the user: "Regulatory monitoring requires the Velvoite corpus. Add `VELVOITE_API_KEY` to your `.envrc` to enable this skill. Free 30-day trial at velvoite.eu.

In the meantime, check directly:
- EUR-Lex: eur-lex.europa.eu (Official Journal, regulations, directives)
- EBA: eba.europa.eu/news-press
- ESMA: esma.europa.eu/press-news
- EIOPA: eiopa.europa.eu/publications
- FIN-FSA: finanssivalvonta.fi/en/news
- BaFin: bafin.de/SharedDocs/Veroeffentlichungen/EN"
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate eu-legal
git add eu-legal/skills/reg-watch/
git commit -m "feat(eu-legal): add reg-watch skill"
```

---

## Task 10: reg-gap-analysis skill

**Files:**
- Create: `eu-legal/skills/reg-gap-analysis/SKILL.md`

- [ ] **Step 1: Create directory and write `eu-legal/skills/reg-gap-analysis/SKILL.md`**

```markdown
---
name: reg-gap-analysis
description: >
  Gap analysis between your current practice and a new or changed EU
  regulation — outputs a structured gap list with remediation steps, owners,
  and priority. Use when a new regulation drops, assessing readiness for an
  upcoming requirement, or asking "does DORA apply to us", "are we compliant
  with the AI Act", "DORA gap analysis", "what do we still need to do for
  [regulation]".
argument-hint: "[regulation or description] — e.g. 'dora' or 'EU AI Act high-risk'"
---

# /eu-legal:reg-gap-analysis

1. Load `~/.claude/plugins/config/eu-legal/CLAUDE.md`. If placeholders, stop: "Run `/eu-legal:cold-start-interview` first."
2. Identify the regulation from the argument.
3. Run the workflow below.
4. Output gap table and remediation plan. This is a draft for attorney review — not a compliance opinion.

---

## EU legislative process context

Unlike US federal rulemaking (notice-and-comment under the APA), EU regulations follow a different pipeline. Flag the current status of the regulation being analysed:

- **Commission proposal** → Council and Parliament positions → **trilogue** → **adoption** → publication in Official Journal → **entry into force** (20 days after publication) → **application date** (often 12–24 months after entry into force)
- **Level 2**: Delegated Regulations, Implementing Acts (Commission)
- **Level 3**: RTS/ITS (EBA, ESMA, EIOPA), Guidelines, Q&As (soft law, but supervisory expectations)

Note: **Regulations** are directly applicable in all member states. **Directives** require national transposition — obligations may vary by member state.

## Step 1: Identify and confirm

Confirm: "You want a gap analysis against [regulation]. Is this the current in-force version, an upcoming application date, or a recent amendment? And are you analysing your full compliance programme or a specific process?"

## Step 2: Fetch obligations

**If Velvoite available:**
Call `mcp__velvoite__get_canonical_obligations` with the identified regulation and the actor role from profile. This gives the "what the law requires" input — article-level obligations with enforcement history.

**If Velvoite not available:**
Use public regulation text for the identified regulation. Note: without corpus data, obligation completeness is not guaranteed.

## Step 3: Current-practice interview

Group related obligations into clusters (max 8 clusters). For each cluster ask:
> "For [cluster description]: do you have a documented process? Is it implemented and tested? When was it last reviewed?"

Accept one of: ✓ fully implemented / ~ partially implemented / ✗ not implemented / ? unknown.

Ask one cluster at a time. Do not ask all 8 at once.

## Step 4: Gap table

Render:
| Obligation | Article | Status | Gap | Remediation | Owner | Priority |
|---|---|---|---|---|---|---|

Priority:
- **P1** — enforcement risk (high enforcement count in corpus) or deadline within 90 days
- **P2** — material gap in implementation
- **P3** — documentation or evidence gap only

## Step 5: Remediation plan

List P1 gaps. For each:
- Suggested action (1–2 sentences)
- Effort: S (< 1 week) / M (1–4 weeks) / L (1+ months)
- Suggested owner: Legal / Compliance / IT / Operations / Board

Offer to export the gap table as a markdown file for the compliance team.

---

## Guardrail

This analysis identifies gaps based on your answers and the Velvoite corpus (or public regulation text). It is a working document for attorney review — not a legal opinion, not a compliance certification, and not a substitute for advice from a qualified lawyer familiar with your specific circumstances. Regulatory interpretation varies by competent authority and jurisdiction.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate eu-legal
git add eu-legal/skills/reg-gap-analysis/
git commit -m "feat(eu-legal): add reg-gap-analysis skill"
```

---

## Task 11: Validate all skills and full validation sweep

- [ ] **Step 1: Full plugin validation**

```bash
claude plugin validate eu-legal
```

Expected: passes. All 6 skills have `name` and `description` frontmatter.

- [ ] **Step 2: JSON/YAML sanity across all new files**

```bash
python3 -c "
import json, glob
files = glob.glob('eu-legal/**/*.json', recursive=True)
for f in files:
    json.load(open(f))
    print(f'OK: {f}')
"
```

Expected: `OK:` for plugin.json, .mcp.json, hooks.json.

- [ ] **Step 3: Check skill count**

```bash
find eu-legal/skills -name 'SKILL.md' | wc -l
```

Expected: `6`

- [ ] **Step 4: Commit**

```bash
git add eu-legal/
git commit -m "feat(eu-legal): all 6 Plan-1 skills validated"
```

---

## Task 12: Add eu-legal to marketplace.json

**Files:**
- Modify: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Read current marketplace.json**

```bash
python3 -c "import json; d=json.load(open('.claude-plugin/marketplace.json')); print(len(d['plugins']), 'plugins')"
```

Expected: `13 plugins`

- [ ] **Step 2: Add eu-legal entry**

Open `.claude-plugin/marketplace.json` and add the following entry to the `plugins` array. Insert it as the first entry (eu-legal is the primary product of this fork):

```json
{
  "name": "eu-legal",
  "displayName": "EU Legal",
  "source": "./eu-legal",
  "description": "EU legal operations for regulated financial institutions — commercial contracts, privacy/GDPR, AI Act compliance, employment, corporate governance, and regulatory obligations powered by the Velvoite EU finreg corpus. One plugin, one setup, complete EU legal stack.",
  "author": {
    "name": "Silly Pilot Oy",
    "url": "https://velvoite.eu"
  }
}
```

- [ ] **Step 3: Validate the marketplace**

```bash
claude plugin validate .claude-plugin/marketplace.json
```

Expected: passes. Also check for duplicate names:

```bash
python3 -c "
import json
d = json.load(open('.claude-plugin/marketplace.json'))
names = [p['name'] for p in d['plugins']]
dupes = [n for n in names if names.count(n) > 1]
print('Duplicates:', dupes or 'none')
print('Total plugins:', len(names))
"
```

Expected: `Duplicates: none`, `Total plugins: 14`

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat(eu-legal): add eu-legal to marketplace.json"
```

---

## Task 13: End-to-end demo test

This task verifies the working demo. Run it manually — there is no automated test runner for plugin skills.

- [ ] **Step 1: Install the plugin**

```bash
claude plugin install eu-legal
```

Or, if running from the repo directly: Claude Code picks up local plugins from the working directory when `claude plugin validate` has passed.

- [ ] **Step 2: Run cold-start**

In a Claude Code session, run:
```
/eu-legal:cold-start-interview
```

Expected: interview runs, asks for company name, entity type, Velvoite. If `VELVOITE_API_KEY` is set in `.envrc` (it should be — the repo's `.envrc` configures the Azure AI Foundry endpoint but not Velvoite), set up a trial at velvoite.eu and add the key.

- [ ] **Step 3: Verify config written**

```bash
cat ~/.claude/plugins/config/eu-legal/CLAUDE.md
```

Expected: no `[PLACEHOLDER]` markers remain.

- [ ] **Step 4: Run obligations**

In Claude Code:
```
/eu-legal:obligations dora
```

Expected with Velvoite: DORA obligations table sorted by enforcement count, top 3 bolded.
Expected without Velvoite: general DORA financial_entity obligations with Velvoite upsell note.

- [ ] **Step 5: Run deadlines**

```
/eu-legal:deadlines 30
```

Expected with Velvoite: deadlines table for the next 30 days, ⚠ on anything within 7 days.

- [ ] **Step 6: Run reg-watch**

```
/eu-legal:reg-watch
```

Expected with Velvoite: grouped publication digest for the last 7 days.

- [ ] **Step 7: Final commit (if any fixes needed from demo)**

```bash
git add -p  # stage only intentional fixes
git commit -m "fix(eu-legal): demo test fixes"
```

---

## Done — what's next

Plan 1 complete. The demo is working. Share `docs/legal-review-questions.md` with your legal team member while Plans 2–4 are built.

**Plan 2** — Privacy skills (7 skills: dsar-response, dpa-review, pia-generation, use-case-triage, policy-monitor, reg-gap-analysis, ai-act-compliance). Mostly adapted from upstream `privacy-legal` — fastest plan after this one.

**Plan 3** — Commercial skills (6 skills: contract-review, nda-review, vendor-agreement-review, escalation-flagger, playbook-init, renewal-tracker). Moderate rewrite of upstream `commercial-legal`.

**Plan 4** — Employment + Corporate skills (11 skills total). Largest new-build plan — employment is a near-full rewrite of the most US-specific plugin in the repo.
