# fi-startup-legal Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `fi-startup-legal` Claude Code plugin — 15 skills covering the Finnish startup journey from OY incorporation through Series A, backed by finlex-mcp, prh-mcp, and optional Velvoite MCP.

**Architecture:** Single plugin at `fi-startup-legal/` in the `law-for-eu` fork. Same structure as eu-legal — flat skills under `skills/<name>/SKILL.md`, one cold-start writes `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md`. MCP tools: `mcp__finlex__*`, `mcp__prh__*`, `mcp__velvoite__*` (optional). Apache 2.0, Silly Pilot Oy.

**Tech Stack:** Claude plugin format (SKILL.md with YAML frontmatter), JSON plugin metadata. No build step. Validate with `claude plugin validate ./fi-startup-legal`.

**Prerequisite:** finlex-mcp and prh-mcp servers must be deployed before skills that call them are tested end-to-end.

---

## Files created in this plan

```
fi-startup-legal/
  .claude-plugin/plugin.json
  .mcp.json
  CLAUDE.md
  .gitignore
  hooks/hooks.json
  LICENSE
  README.md
  skills/
    startup-cold-start/SKILL.md
    oy-setup/SKILL.md
    founder-agreement/SKILL.md
    ip-assignment/SKILL.md
    sha-review/SKILL.md
    term-sheet-review/SKILL.md
    esop-designer/SKILL.md
    vesting-calculator/SKILL.md
    bf-grant-check/SKILL.md
    first-hire-contract/SKILL.md
    non-compete-check/SKILL.md
    customer-contract/SKILL.md
    startup-nda/SKILL.md
    board-minutes/SKILL.md
    gdpr-basics/SKILL.md
```

---

## Task 1: Plugin scaffold

**Files:** `fi-startup-legal/.claude-plugin/plugin.json`, `.mcp.json`, `hooks/hooks.json`, `.gitignore`, `LICENSE`

- [ ] **Step 1: Create directories**

```bash
mkdir -p fi-startup-legal/.claude-plugin fi-startup-legal/hooks fi-startup-legal/skills
```

- [ ] **Step 2: Write `fi-startup-legal/.claude-plugin/plugin.json`**

```json
{
  "name": "fi-startup-legal",
  "version": "0.1.0",
  "description": "Finnish startup legal toolkit — OY incorporation, founder agreements, ESOP (TVL §66 2026 reform), SHA review, Business Finland grant compliance, first hires. Backed by live Finlex statute data and PRH company register.",
  "author": {
    "name": "Silly Pilot Oy",
    "url": "https://velvoite.eu"
  }
}
```

- [ ] **Step 3: Write `fi-startup-legal/.mcp.json`**

```json
{
  "mcpServers": {
    "finlex": {
      "type": "http",
      "url": "https://finlex-mcp.velvoite.eu/mcp",
      "title": "Finlex",
      "description": "Live Finnish statute text — TSL, OYL, TVL (§66 ESOP), YTL, LSL, VLL and more."
    },
    "prh": {
      "type": "http",
      "url": "https://prh-mcp.velvoite.eu/mcp",
      "title": "PRH",
      "description": "Finnish Trade Register — company lookup, name availability, entity status."
    },
    "velvoite": {
      "type": "http",
      "url": "https://mcp.velvoite.eu/mcp",
      "title": "Velvoite",
      "description": "EU finreg corpus — GDPR, AI Act, AML obligations (optional, for AI/fintech startups).",
      "headers": {
        "X-API-Key": "${VELVOITE_API_KEY}"
      }
    },
    "Slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "title": "Slack",
      "description": "Post alerts and digests."
    },
    "Google Drive": {
      "type": "http",
      "url": "https://drivemcp.googleapis.com/mcp/v1",
      "title": "Google Drive",
      "description": "Store and retrieve documents."
    }
  },
  "recommendedCategories": [
    "legal-research",
    "documents",
    "chat"
  ]
}
```

- [ ] **Step 4: Write `fi-startup-legal/hooks/hooks.json`**

```json
{"hooks": {}}
```

- [ ] **Step 5: Write `fi-startup-legal/.gitignore`**

```
.DS_Store
*.skill
/logs/
/outputs/
/.claude/
__pycache__/
*.pyc
```

- [ ] **Step 6: Copy LICENSE from eu-legal**

```bash
cp eu-legal/LICENSE fi-startup-legal/LICENSE
```

- [ ] **Step 7: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/ && git commit -m "feat(fi-startup-legal): plugin scaffold"
```

---

## Task 2: CLAUDE.md template

- [ ] **Step 1: Write `fi-startup-legal/CLAUDE.md`**

```markdown
<!--
CONFIGURATION LOCATION

User config lives at: ~/.claude/plugins/config/fi-startup-legal/CLAUDE.md

Rules for every skill:
1. READ from that path. Not from this file.
2. If missing or has [PLACEHOLDER] markers, STOP. Say:
   "Run /fi-startup-legal:startup-cold-start to set up — takes about 10 minutes."
3. startup-cold-start WRITES to that path.
4. This file is the TEMPLATE. Never write user data here.
-->

# Finnish Startup Legal Profile
*Run `/fi-startup-legal:startup-cold-start` to set up.*

---

## Company

**Name:** [PLACEHOLDER]
**Business ID (Y-tunnus):** [PLACEHOLDER — or 'not yet incorporated']
**Stage:** [PLACEHOLDER — pre-incorporation | incorporated (pre-seed) | post-seed | Series A]
**Incorporated:** [PLACEHOLDER — date or N/A]
**Primary jurisdiction:** Finland

---

## Founders

[PLACEHOLDER — name, role, equity % for each founder]

---

## Funding & grants

**Rounds completed:** [PLACEHOLDER — none | pre-seed | seed | Series A]
**Business Finland grants active:** [PLACEHOLDER — yes (list grants) | no]
**Tesi involvement:** [PLACEHOLDER — yes | no]

---

## Legal setup

**Outside counsel:** [PLACEHOLDER — firm name or none]
**SHA in place:** [PLACEHOLDER — yes (seriesseed.fi format / other) | no]
**Option pool:** [PLACEHOLDER — yes (size: %) | no]
**Option scheme type:** [PLACEHOLDER — TVL §66 | TVL §66a | none]

---

## Tech/AI flag

**AI product:** [PLACEHOLDER — yes | no]
**Velvoite API key:** [PLACEHOLDER ✓/✗]

---

## Available integrations

| Integration | Status |
|---|---|
| Finlex MCP | [PLACEHOLDER ✓/✗] |
| PRH MCP | [PLACEHOLDER ✓/✗] |
| Velvoite MCP | [PLACEHOLDER ✓/✗] |
| Slack | [PLACEHOLDER ✓/✗] |
| Google Drive | [PLACEHOLDER ✓/✗] |
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/CLAUDE.md && git commit -m "feat(fi-startup-legal): CLAUDE.md practice profile template"
```

---

## Task 3: startup-cold-start skill

- [ ] **Step 1: Write `fi-startup-legal/skills/startup-cold-start/SKILL.md`**

```markdown
---
name: startup-cold-start
description: >
  Set up the fi-startup-legal plugin — learns your startup's stage, founders,
  funding, grants, and legal setup, then writes the practice profile. Run on
  first install or when your situation changes. Use --redo to re-run.
argument-hint: "[--redo]"
---

# /fi-startup-legal:startup-cold-start

1. Check `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md` — if populated and no --redo, confirm before overwriting.
2. Probe integrations: call `mcp__finlex__search_statutes("test")`, `mcp__prh__search_companies("test")`. Report ✓/✗ for each. Velvoite: check `VELVOITE_API_KEY` env var.
3. Run interview below. Write profile. Offer first action.

---

## Interview

One topic per message. Plain language — no legal jargon.

### 1. Company basics
"Let's set up fi-startup-legal. What's your company name? And have you incorporated yet?"

If yes: ask for business ID (Y-tunnus). Call `mcp__prh__lookup_company(business_id)` to verify and pre-fill name, registration date, address. Confirm: "Found [name], registered [date]. Is this right?"

If no: set stage = pre-incorporation.

### 2. Stage
"What best describes where you are?"
1. Just an idea — haven't incorporated yet
2. Incorporated, haven't raised money yet
3. Raised a pre-seed or seed round (€0–2M)
4. Raised a Series A or larger

### 3. Founders
"Who are the founders? (Name + role + approximate equity % for each. It's okay if not finalised.)"

### 4. Business Finland grants
"Have you received any Business Finland funding? (R&D grant, TUTLI, Tempo, etc.)"
If yes: ask which grants and approximate amounts. This affects IP transfer restrictions.

### 5. Legal setup
"Do you have a shareholders' agreement (osakassopimus) in place yet?"
If yes: ask if it follows the seriesseed.fi format or something else.

"Do you have an option/ESOP pool?"
If yes: ask size (%) and whether it's TVL §66 options or a §66a share issue scheme.

### 6. Outside counsel
"Are you working with a law firm? (Fondia, Nordic Law, Dottir, Lexia, or other — or none yet.)"

### 7. AI product flag
"Is your product or service AI-based? (Determines whether EU AI Act compliance is relevant for you.)"
If yes: offer Velvoite API key for AI Act depth. Note: free 30-day trial at velvoite.eu.

---

## Write profile

Read the CLAUDE.md template. Replace all [PLACEHOLDER] markers. Write to `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md`.

Show summary. Then offer based on stage:
- Pre-incorporation: "Try `/fi-startup-legal:oy-setup` to start the incorporation wizard."
- Incorporated, no SHA: "Try `/fi-startup-legal:founder-agreement` to set up your co-founder agreement."
- Post-seed: "Try `/fi-startup-legal:sha-review` to review your shareholders' agreement."
- Series A: "Try `/fi-startup-legal:term-sheet-review` to review an incoming term sheet."
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/skills/startup-cold-start/ && git commit -m "feat(fi-startup-legal): add startup-cold-start skill"
```

---

## Task 4: Incorporation skills (oy-setup, founder-agreement, ip-assignment)

Create each skill directory and write the SKILL.md. Validate and commit after all three.

- [ ] **Step 1: Write `fi-startup-legal/skills/oy-setup/SKILL.md`**

```markdown
---
name: oy-setup
description: >
  OY (Osakeyhtiö) incorporation wizard — walks through company name check,
  yhtiöjärjestys (articles of association), osakassopimus (shareholder agreement)
  checklist, and PRH registration steps. Use when incorporating a new Finnish
  company or when asking "how do I set up an OY", "incorporation checklist",
  "yhtiöjärjestys template".
argument-hint: "[company name to check]"
---

# /fi-startup-legal:oy-setup

1. Load `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md`. If placeholders, stop: "Run `/fi-startup-legal:startup-cold-start` first."
2. Run the incorporation wizard below.
3. Output: step-by-step checklist. Every output is a draft — not legal advice.

---

## Step 1: Company name check

If a company name was provided as an argument or is in the profile, call `mcp__prh__search_companies(name)` to check if the name is available. Report: "Available — no exact match found" or "Conflict — [company] already exists with business ID [X]."

Key name rules (OYL §2:3): must include "Oy", "Oyj", or "Ab"; must be distinguishable from existing names in the register; no misleading terms (e.g. cannot imply bank/insurance unless licensed).

## Step 2: Incorporation checklist

Call `mcp__finlex__get_statute("OYL", "2")` to retrieve current share capital and formation rules.

Walk through each item:

**Before PRH:**
- [ ] Company name confirmed available (PRH search above)
- [ ] Share capital: €0 minimum since 2019 (OYL §2:1). Typical startup: 2,500 shares at €0.01 = €25 paid-in capital.
- [ ] Shareholders: list all founders + initial equity split
- [ ] Perustamissopimus (memorandum of association): must include company name, address, founders, shares, share capital, board members, financial year
- [ ] Yhtiöjärjestys (articles of association): minimum content per OYL §2:3 — company name, domicile, line of business
- [ ] Board: minimum 1 member (+ deputy if only 1 member). EEA residency required unless PRH exemption obtained.
- [ ] Auditor: required if 2+ of: (1) turnover >€12M, (2) balance sheet >€6M, (3) >50 employees. Most early startups: no auditor required.

**PRH registration:**
- [ ] Register at prh.fi/yritykset/perustaminen (online, ~1–3 business days). 2026 fees: €300 guided package / €400 self-prepared documents. Paper forms no longer accepted from 1.1.2026.
- [ ] YEL (entrepreneur pension): sole founder working in own company must register YEL within 3 months if annual work income >€9,010.28 (2026 threshold)

**After incorporation:**
- [ ] Open business bank account
- [ ] Register for VAT if turnover will exceed €15,000/year (threshold 2026)
- [ ] Sign osakassopimus (shareholders' agreement) — not required by law but essential for startups

## Step 3: Yhtiöjärjestys guidance

PRH provides an official template. Startups typically add these non-mandatory clauses:
- Lunastuslauseke (redemption clause): right for existing shareholders to buy shares before transfer to outsider
- Suostumuslauseke (consent clause): share transfers require board approval
- Note: drag-along and tag-along cannot be put in yhtiöjärjestys — they belong in osakassopimus

## Step 4: Next steps

After incorporation: run `/fi-startup-legal:founder-agreement` to set up your co-founder agreement.

---

## Guardrail

Incorporation involves legal and tax obligations. This checklist covers the standard path. If any founder is outside the EU/EEA, if the company has unusual share structure, or if you're in a regulated sector, consult a lawyer before filing.
```

- [ ] **Step 2: Write `fi-startup-legal/skills/founder-agreement/SKILL.md`**

```markdown
---
name: founder-agreement
description: >
  Co-founder agreement and vesting setup — equity split, roles, vesting schedule
  (Finnish good/bad/early leaver), IP assignment, non-compete validity check.
  Use when setting up co-founder terms, asking "founder agreement checklist",
  "vesting schedule", "what should a co-founder agreement cover in Finland".
argument-hint: "[number of founders, or describe the situation]"
---

# /fi-startup-legal:founder-agreement

1. Load `~/.claude/plugins/config/fi-startup-legal/CLAUDE.md`. If placeholders, stop: setup first.
2. Call `mcp__finlex__get_statute("TSL", "3:5")` for current non-compete rules.
3. Run the workflow below. Output is a draft checklist — attorney review required before signing.

---

## Purpose

A founder agreement (typically part of or alongside the osakassopimus) covers what happens if a co-founder leaves. Most Finnish startup disputes stem from a missing or vague founder agreement. This skill walks through the key terms.

## Key terms to cover

### Equity split
Ask: "How is equity split between founders?" Note: equal splits (50/50, 33/33/33) are common but can cause deadlock. Suggest a tiebreaker mechanism if equal split.

### Founder vesting
Finnish market standard (following seriesseed.fi): 4-year total, 1-year cliff.

**Leaver mechanics (three-tier standard):**

| Type | Definition | Share treatment |
|---|---|---|
| Good Leaver | Leaves after cliff, no fault | Keeps vested shares at fair market value |
| Early Leaver | Leaves before cliff OR voluntarily before 2 years | Returns unvested shares at nominal value (subscription price) |
| Bad Leaver | Termination for cause, breach, non-compete violation | Returns ALL shares (vested + unvested) at nominal value |

Call `mcp__finlex__get_statute("OYL", "3")` for share buyback/redemption rules under Finnish company law.

### IP assignment
Every founder must assign all relevant IP to the company at founding. This includes:
- Software, code, designs created before incorporation (if related to the business)
- Patents, trade secrets, know-how
- Domain names, brand assets

Flag: did any founder previously work for an employer in this field? Employee invention rules (Laki oikeudesta työntekijän tekemiin keksintöihin) may apply. Route to `/fi-startup-legal:ip-assignment` for full IP audit.

### Non-compete
Finnish law (TSL §3:5, fetched live from Finlex): non-compete max 12 months after employment ends. Compensation required if restriction exceeds 6 months:
- 6–12 months restriction → 50% of salary for restricted period
- Note: founder non-competes in SHA are shareholder agreements, not employment law — different rules may apply. Route to outside counsel for founder-specific non-competes >12 months.

### Decision-making
For 2 founders: veto rights allocation, deadlock resolution.
For 3+ founders: voting thresholds for major decisions (new share issuance, sale, key hires).

---

## Guardrail

Founder agreements and vesting mechanics have significant long-term consequences. Have a Finnish startup lawyer review the final terms before signing. Recommended firms: Nordic Law, Dottir, Lexia Growth, Fondia.
```

- [ ] **Step 3: Write `fi-startup-legal/skills/ip-assignment/SKILL.md`**

```markdown
---
name: ip-assignment
description: >
  IP assignment audit — checks that all founders, employees, and contractors have
  properly assigned IP to the company. Flags gaps. Checks Business Finland IP
  restrictions if grants are active. Use when asking "do we own our IP",
  "IP assignment checklist", "does BF grant affect our IP", or before any M&A.
argument-hint: "[describe the company's IP situation]"
---

# /fi-startup-legal:ip-assignment

1. Load profile. If placeholders, stop.
2. Call `mcp__finlex__get_statute("TSL", "7")` — employee invention rules.
3. Run the IP audit workflow. Flag gaps. This is advisory — attorney review required.

---

## IP ownership checklist

### Founders
For each founder: did they sign an IP assignment agreement at incorporation? Does it cover:
- [ ] Code/software written before incorporation (if related to the business)
- [ ] Code/software written after incorporation
- [ ] Domain names, brand assets, trade secrets, know-how
- [ ] Any patents or patent applications

**Finnish employee invention rule (Laki oikeudesta työntekijän tekemiin keksintöihin):** An employer has the right to claim an invention made by an employee in the course of their employment duties. For founder-employees this is automatic — but a written IP assignment removes ambiguity.

Prior employer risk: if a founder previously worked for a company in this field, that employer may claim rights to inventions made during employment. Ask: "Did any founder work for another company in this field in the 12 months before starting this company?" If yes → route to outside counsel.

### Employees and contractors
- [ ] All employees: standard IP assignment clause in employment contract (covered by `/fi-startup-legal:first-hire-contract`)
- [ ] Contractors/freelancers: written IP assignment in each contractor agreement. Note: in Finland, software copyright vests in the author by default — contractor agreements MUST contain explicit assignment or the contractor retains copyright.

### Business Finland grants
If profile shows BF grants active: call this check.

BF funding terms: IP developed in the funded project must remain with the funded entity (same Y-tunnus). Key restrictions:
- Cannot transfer project IP to a subsidiary or affiliate without BF approval
- Control transfer outside EEA within 5 years of final funding installment requires prior BF approval
- "Control transfer" = change in majority ownership (direct or indirect)

Flag: is any M&A, restructuring, or foreign investor acquiring >50% planned? Route to outside counsel + notify BF.

---

## Guardrail

IP assignment gaps are the most common cause of failed M&A and VC due diligence in Finnish startups. Any gap identified here requires attorney action before the next funding round or exit process.
```

- [ ] **Step 4: Validate all three skills and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/skills/oy-setup/ fi-startup-legal/skills/founder-agreement/ fi-startup-legal/skills/ip-assignment/ && git commit -m "feat(fi-startup-legal): add incorporation skills (oy-setup, founder-agreement, ip-assignment)"
```

---

## Task 5: Investment skills (sha-review, term-sheet-review)

- [ ] **Step 1: Write `fi-startup-legal/skills/sha-review/SKILL.md`**

```markdown
---
name: sha-review
description: >
  Review a Finnish shareholders' agreement (osakassopimus) against the seriesseed.fi
  market standard — three-tier leaver, pre-emption rights, drag-along, tag-along,
  ROFR, anti-dilution, board composition, protective provisions. Use when reviewing
  a SHA, asking "is our SHA standard", or preparing for a funding round.
argument-hint: "[paste or describe the SHA, or 'seriesseed-check' for a quick checklist]"
---

# /fi-startup-legal:sha-review

1. Load profile. If placeholders, stop.
2. Call `mcp__finlex__get_statute("OYL", "3")` for share transfer and buyback rules.
3. Call `mcp__finlex__get_statute("OYL", "9")` for shareholder rights.
4. Review against seriesseed.fi standard. Output: term-by-term RAG status.

---

## seriesseed.fi standard checklist

The Startup Foundation (startup-saatio.fi) seriesseed.fi templates are the Finnish market standard for seed rounds. Review each section:

### Leaver provisions
| Term | Market standard | Status |
|---|---|---|
| Good Leaver definition | Leaves voluntarily after cliff, or involuntary without cause | |
| Early Leaver definition | Leaves before cliff OR voluntarily before 2 years | |
| Bad Leaver definition | Termination for cause, material breach, non-compete violation | |
| Good Leaver share price | Fair market value | |
| Early/Bad Leaver share price | Subscription price (nominal) | |
| Vesting schedule | 4 years, 1-year cliff | |
| Acceleration | Single-trigger (involuntary termination post-M&A) market standard | |

### Transfer restrictions
| Term | Market standard | Notes |
|---|---|---|
| Pre-emption right | Existing shareholders have right to buy before external transfer | OYL §3 allows; must be in SHA or yhtiöjärjestys |
| ROFR (right of first refusal) | At same price as third-party offer | |
| Drag-along | Majority (e.g. 75%) can drag minority in exit | Must be in SHA; cannot be in yhtiöjärjestys alone |
| Tag-along | Minority can join any majority sale at same price | |
| Lock-up | Typically 12 months post-IPO if applicable | |

### Investor protections
| Term | Market standard | Notes |
|---|---|---|
| Anti-dilution | Broad-based weighted average (not full ratchet) | Full ratchet is aggressive — flag |
| Pro-rata right | Investor can maintain % in future rounds | |
| Information rights | Quarterly financials, annual audited accounts | |
| Board seat | Lead investor gets 1 board seat for rounds >€500k | |
| Protective provisions | New share issuance, M&A, budget approval require investor consent | List the standard matters |

### 2024 seriesseed.fi updates
The Startup Foundation updated templates in 2024 to add:
- KYC/AML representations from investors
- Sanctions screening obligations
- Three-tier leaver (replacing two-tier) — check if the SHA being reviewed uses three-tier or older two-tier format

---

## Guardrail

SHA review identifies deviations from market standard. Whether a deviation is acceptable depends on the specific investor relationship and negotiating dynamics. Attorney review required before signing.
```

- [ ] **Step 2: Write `fi-startup-legal/skills/term-sheet-review/SKILL.md`**

```markdown
---
name: term-sheet-review
description: >
  Review a VC or angel term sheet in plain language — valuation mechanics,
  liquidation preference, pro-rata rights, information rights, board, protective
  provisions, ESOP pool. Explains each term for a non-lawyer founder.
  Use when reviewing a term sheet or asking "what does this term mean".
argument-hint: "[paste term sheet or describe the key terms]"
---

# /fi-startup-legal:term-sheet-review

1. Load profile. If placeholders, stop.
2. Run term-by-term review in plain language. Flag aggressive terms. Explain each in 1–2 sentences.
3. Output: term table with explanation + RED/AMBER/GREEN status.

---

## Term sheet review

For each term, explain what it means in plain language and flag if it deviates from Finnish/Nordic market standard.

### Valuation
- **Pre-money valuation**: company value before the investment. Post-money = pre-money + investment amount.
- **Fully diluted**: valuation calculated including all options, convertible notes, warrants. Check: is the ESOP pool included pre-money (dilutes founders) or post-money (dilutes everyone)?

**Finnish/Nordic market: ESOP pool typically included pre-money. If post-money: note but not unusual.**

### Liquidation preference
- **1× non-participating**: investor gets 1× their money back before common shareholders in a liquidation/exit, then does NOT also participate in remaining proceeds. **Market standard — GREEN.**
- **1× participating**: investor gets 1× back AND participates pro-rata in remaining proceeds. **Aggressive — AMBER.**
- **>1× or full ratchet**: investor gets more than invested back. **Very aggressive — RED. Push back.**

### Pro-rata right
Right for investor to participate in future rounds to maintain their ownership %. **Market standard — GREEN.** Super pro-rata (right to more than maintain %) is aggressive.

### Board composition
Ask: how many seats total, who gets them? Market standard for seed:
- 2 founder seats
- 1 lead investor seat
- 1 independent (optional)
Giving investors majority board control at seed is **RED**.

### Protective provisions (veto rights)
Investors typically require consent for: new share issuance, M&A/exit, budget approval, incurring debt above threshold, hiring/firing CEO. List each one — standard list GREEN, unusual additions AMBER.

### Information rights
Quarterly financials + annual audited accounts = **GREEN**. Monthly board pack demands on a sub-€500k round = **AMBER**.

### Anti-dilution
See sha-review — broad-based weighted average = **GREEN**; full ratchet = **RED**.

### ESOP/option pool
What size? Typical at seed: 10–15% pre-money. If investor demands 20%+ pre-money: **AMBER** (dilutes founders significantly).

---

## Guardrail

Term sheet review identifies market-standard vs. aggressive terms. This is educational guidance for a non-lawyer founder — not legal advice. Have a Finnish startup lawyer review the full term sheet before signing. Non-binding term sheets can create moral obligations and set negotiation anchors.
```

- [ ] **Step 3: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/skills/sha-review/ fi-startup-legal/skills/term-sheet-review/ && git commit -m "feat(fi-startup-legal): add investment skills (sha-review, term-sheet-review)"
```

---

## Task 6: ESOP skills (esop-designer, vesting-calculator)

- [ ] **Step 1: Write `fi-startup-legal/skills/esop-designer/SKILL.md`**

```markdown
---
name: esop-designer
description: >
  Finnish ESOP/option scheme design — TVL §66 vs §66a decision tool including
  the 2026 reform (options now taxed at sale, not exercise), AGM approval
  requirements, option pool sizing, typical Finnish vesting terms.
  Use when asking "how do we set up options", "ESOP for our startup",
  "TVL §66 vs §66a", "how does the 2026 option reform affect us".
argument-hint: "[describe the situation: company stage, number of beneficiaries, amounts]"
---

# /fi-startup-legal:esop-designer

1. Load profile. If placeholders, stop.
2. Call `mcp__finlex__get_statute("TVL", "66")` — fetch live TVL §66 text.
3. Call `mcp__finlex__get_statute("TVL", "66a")` — fetch live TVL §66a text.
4. Call `mcp__finlex__get_statute("OYL", "10")` — AGM approval requirements for option schemes.
5. Run the §66 vs §66a decision workflow. Output is advisory — tax advice requires a qualified tax advisor.

---

## TVL §66 vs §66a — which to use?

This decision is the most important tax choice for Finnish startup equity compensation. Fetch current statute text before answering.

### TVL §66 (stock options) — current law + proposed reform

**Current law (in force as of May 2026):** Options taxed as earned income (ansiotulo) **at exercise** — the spread between fair market value at exercise and subscription price is taxed at progressive income tax rates. Tax bill arises at exercise, before any exit. This is the cash-flow problem for startup employees.

**2026 government proposal (NOT yet enacted):** The Finnish government has proposed deferring this earned-income taxation from exercise to sale for qualifying unlisted companies. As of May 2026 this is only a legislative proposal — it has not been enacted. **Do not present the reform as current law.**

After fetching the live TVL §66 text via Finlex: present the current enacted rules. If the proposal has been enacted (check the statute text — if it includes sale-timing language, the reform has passed), present the new rules and note the effective date.

Key current rule: on eventual share sale, acquisition cost = subscription price + amount previously taxed as earned income at exercise (to avoid double taxation).

### TVL §66a (directed share issue at mathematical value)

An alternative to options: company issues new shares to employees at the "mathematical value" (substansarvo — net assets divided by share count). If this is below market value, the difference is not income-taxed at subscription — it's capital income when the shares are sold (~30% vs. up to 40%+ income tax).

Conditions (from fetched statute): company must be unlisted; employee must work in the company; issue price at or above mathematical value.

**Practical comparison table:**

| Factor | TVL §66 (post-2026) | TVL §66a |
|---|---|---|
| Tax timing | At sale | At sale |
| Tax rate on gain | Income tax rate (progressive) | Capital gains (~30%) |
| Subscription price | At least FMV at grant | At least mathematical value (may be lower than FMV) |
| Flexibility | High (strike price, cliff, vesting) | Lower (shares issued immediately) |
| AGM approval | Required | Required |
| Good for | Standard startup ESOP with vesting | Early employees / founders wanting capital gains treatment |

### Recommendation logic

Present the right choice based on:
- If company has significant net assets (mathematical value close to market value): §66a advantage is small → use §66
- If early stage, low net assets (mathematical value << market value): §66a can give employees shares at low price with capital gains treatment
- If standard ESOP with vesting + performance conditions is needed: §66 is cleaner

### AGM mechanics

Call `mcp__finlex__get_statute("OYL", "10")` for current rules. Share options require:
- Extraordinary or annual general meeting approval
- Resolution must specify: total number of options, subscription price, subscription period, deviation from pre-emption rights (and reason)
- Board of directors then grants individual allocations

---

## Guardrail

Option scheme design has significant tax consequences for employees and the company. The TVL §66 reform is recent (2026) — verify current rules with a Finnish tax advisor (veroasiantuntija) before implementing. This skill reads live statute text but does not replace tax advice.
```

- [ ] **Step 2: Write `fi-startup-legal/skills/vesting-calculator/SKILL.md`**

```markdown
---
name: vesting-calculator
description: >
  Finnish startup vesting schedule generator — cliff, monthly vesting,
  good/early/bad leaver treatment, acceleration (single/double trigger),
  exercise window. Explains vesting mechanics in plain language.
  Use when asking "vesting schedule", "how much has [person] vested",
  "what happens if a co-founder leaves".
argument-hint: "[describe: grant size, start date, cliff, vesting period, or 'calculate [name]']"
---

# /fi-startup-legal:vesting-calculator

1. Load profile for option scheme type (TVL §66 vs §66a, option pool size).
2. Collect inputs: grant size, grant date, cliff period, total vesting period, leaver type if applicable.
3. Calculate and display. All outputs are illustrative — actual vesting per signed agreements.

---

## Finnish market standard

- **Total vesting period:** 4 years
- **Cliff:** 1 year (nothing vests before the 1-year anniversary)
- **Post-cliff:** monthly vesting (1/48th per month after the cliff)
- **Exercise window:** 10 years from grant (Finnish market standard; some companies use 5 years — flag if <5 years as this is restrictive)

## Vesting calculation

For any grant, calculate:
- Shares vested at cliff date (25% of total, after 12 months)
- Shares vesting per month after cliff (1/48th of total)
- Total vested as of any given date
- Total unvested remaining

Show as a table: Month | Cumulative vested | Cumulative unvested.

## Leaver calculation

Given a leaver date and leaver type (good/early/bad), calculate:
- Shares already vested → treatment per SHA leaver provisions
- Shares unvested → treatment per SHA leaver provisions
- If bad leaver: all shares (vested + unvested) at nominal value

## Acceleration

**Single-trigger:** full vesting acceleration on involuntary termination following M&A. Finnish market standard for founders.
**Double-trigger:** acceleration only on (1) M&A AND (2) involuntary termination within 12 months. Common for non-founder employees.

Show the post-acceleration vested share count.

---

## Guardrail

Vesting calculations are based on the inputs provided and market-standard terms. Actual vesting is governed by the signed option agreement and SHA. Consult the actual documents — this skill does not read them.
```

- [ ] **Step 3: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/skills/esop-designer/ fi-startup-legal/skills/vesting-calculator/ && git commit -m "feat(fi-startup-legal): add ESOP skills (esop-designer, vesting-calculator)"
```

---

## Task 7: Grants and employment skills (bf-grant-check, first-hire-contract, non-compete-check)

- [ ] **Step 1: Write `fi-startup-legal/skills/bf-grant-check/SKILL.md`**

```markdown
---
name: bf-grant-check
description: >
  Business Finland grant IP compliance check — assesses whether a planned
  action (M&A, restructuring, foreign investment, IP transfer) triggers the
  5-year EEA control transfer restriction or IP retention requirement.
  Use when asking "can we sell to a US company", "does this VC deal affect
  our BF grant", "IP clawback risk", or before any major transaction.
argument-hint: "[describe the planned transaction or situation]"
---

# /fi-startup-legal:bf-grant-check

1. Load profile — check Business Finland grants active flag.
2. If no BF grants: "No Business Finland grants recorded in your profile. If you have grants, update via `/fi-startup-legal:startup-cold-start --redo`."
3. If BF grants active: run the compliance check below.

---

## Business Finland IP and control transfer restrictions

### IP retention requirement
IP developed in the BF-funded project must remain with the funded entity (same Y-tunnus) throughout the project and for a retention period after. Key rule: the IP cannot be transferred to a subsidiary, affiliate, or third party without BF approval.

**Triggers requiring BF notification or approval:**
- [ ] Transferring project IP to a new holding company
- [ ] Assigning patents or copyright from the funded entity to any other entity
- [ ] Licensing project IP exclusively to a single party (may be treated as transfer)
- [ ] M&A where the funded entity ceases to exist (merger, absorption)

### EEA control transfer restriction (5-year rule)
Within 5 years of receiving the final BF funding installment: a control transfer outside the European Economic Area requires prior BF approval.

**"Control transfer"** = a change resulting in a non-EEA entity (person or company) acquiring direct or indirect majority control (>50% of shares or voting rights).

**Triggers (check all):**
- [ ] New investor acquires >50% of shares
- [ ] Existing non-EEA investor's stake crosses 50% threshold
- [ ] M&A where acquirer is headquartered outside EEA
- [ ] Complex ownership structures where indirect non-EEA control crosses 50%

**Foreign LP risk:** If a VC fund has non-EEA LPs, the fund itself is typically the direct shareholder — not the LPs. Whether LP-level non-EEA ownership constitutes "indirect control" requires case-by-case BF analysis. **Route to outside counsel + BF inquiry for any fund with significant non-EEA LP exposure.**

### Clawback
Non-compliance can result in full or partial recovery (clawback) of the grant. BF has discretion — cooperation and proactive notification typically reduces risk.

### Next steps
If a planned transaction may trigger these restrictions:
1. Do not proceed without checking
2. Contact Business Finland directly (puh. 029 5069 000 or businessfinland.fi/yhteystiedot)
3. Engage a lawyer familiar with BF grant terms

---

## Guardrail

This check identifies potential BF grant compliance issues. Business Finland grant terms vary by program and year — always review the specific grant agreement. This is not legal advice; route any material transaction to outside counsel and notify BF proactively.
```

- [ ] **Step 2: Write `fi-startup-legal/skills/first-hire-contract/SKILL.md`**

```markdown
---
name: first-hire-contract
description: >
  Finnish startup first employment contract — mandatory terms under TSL,
  trial period, notice period, non-compete validity, IP assignment, ESOP
  offer mechanics. Use when hiring the first employee, reviewing a draft
  contract, or asking "what must be in a Finnish employment contract".
argument-hint: "[describe the role or paste the contract draft]"
---

# /fi-startup-legal:first-hire-contract

1. Load profile.
2. Call `mcp__finlex__get_statute("TSL", "1:4")` — mandatory employment contract terms.
3. Call `mcp__finlex__get_statute("TSL", "3:5")` — non-compete rules.
4. Call `mcp__finlex__get_statute("TSL", "1:4")` — trial period rules.
5. Run checklist. Draft for attorney review.

---

## Mandatory terms (TSL §1:4, fetched live)

Every Finnish employment contract must cover (from fetched statute):
- [ ] Employer name and address
- [ ] Employee name and address
- [ ] Start date
- [ ] Duration: indefinite (toistaiseksi voimassa oleva) or fixed-term (with reason if fixed-term)
- [ ] Place of work (or principle that employee works in multiple locations)
- [ ] Main duties (työn pääasialliset tehtävät)
- [ ] Applicable collective agreement (TES), if any
- [ ] Salary and payment interval
- [ ] Normal working hours
- [ ] Annual holiday accrual reference (Vuosilomalaki)
- [ ] Notice period (irtisanomisaika)

If not given in writing at start: employer must provide within 7 days of start date (TSL §2:4).

## Trial period (koeaika)
Maximum 6 months (TSL §1:4). During trial: either party can terminate immediately with no notice, no reason required. If fixed-term contract < 12 months: trial period cannot exceed half the contract duration.

## Notice periods (from fetched statute)
Employer notice (TSL §6:4) by service length:
- < 1 year: 14 days
- 1–4 years: 1 month
- 4–8 years: 2 months
- 8–12 years: 3 months
- 12–15 years: 4 months
- > 15 years: 6 months

Employee notice (TSL §6:3):
- < 5 years: 14 days
- > 5 years: 1 month

## Non-compete clause
See `/fi-startup-legal:non-compete-check` for full analysis. Key rules (TSL §3:5, live):
- Max 12 months after employment ends
- Compensation required if restriction > 6 months
- Must be "particularly weighty reason" (erityisen painava syy) to justify a non-compete

## IP assignment
Include in every employment contract:
"All inventions, works, and developments created by the employee in the course of their duties, and all related intellectual property rights, are assigned to and vest in [company name] automatically upon creation."

Note: Finnish employee invention law (Laki oikeudesta työntekijän tekemiin keksintöihin) gives employer rights to employment-related inventions by default — explicit assignment clause removes any ambiguity.

## YEL for founders
Remind: founders working in their own company with annual work income >€9,010.28 (2026) must register YEL pension insurance within 3 months. Failure results in retroactive claims with interest.

---

## Guardrail

Employment contracts create binding legal obligations. Have a Finnish employment lawyer or HR legal service (Fondia, Lexly) review before issuing. Collective agreement obligations depend on the industry sector — check if a TES applies.
```

- [ ] **Step 3: Write `fi-startup-legal/skills/non-compete-check/SKILL.md`**

```markdown
---
name: non-compete-check
description: >
  Validate a Finnish non-compete clause against TSL §3:5 — maximum duration,
  compensation requirements, scope reasonableness. Use when reviewing or
  drafting a non-compete, or asking "is this non-compete valid in Finland",
  "do we need to pay for non-compete", "how long can a non-compete be".
argument-hint: "[describe or paste the non-compete clause]"
---

# /fi-startup-legal:non-compete-check

1. Load profile.
2. Call `mcp__finlex__get_statute("TSL", "3:5")` — fetch current non-compete statute text.
3. Validate against the live statute. Flag any issues.

---

## Validation (from live statute text)

Apply the fetched TSL §3:5 text to check:

### Duration
- Maximum: 12 months after employment ends
- If restriction > 12 months: **INVALID** under Finnish law — clause is unenforceable beyond 12 months

### Compensation requirement
- 0–6 months restriction: no compensation required
- 6–12 months restriction: compensation must be paid to the employee during the restricted period
  - Amount: extract from current statute text (historically 50% of salary — verify with live text)
  - Payment timing: typically monthly during the restriction period
- If compensation is required but not included in the contract: **RED — clause may be unenforceable**

### "Particularly weighty reason" requirement
Finnish law requires a "particularly weighty reason" (erityisen painava syy) for a non-compete to be valid. Legitimate reasons: access to trade secrets, customer relationships, sensitive competitive information. A blanket non-compete for all employees regardless of role is vulnerable.

Ask: does the employee have access to genuine trade secrets or key customer relationships? If no clear justification: **AMBER — validity risk**.

### Geographic scope
Unreasonably broad geographic scope (e.g. entire world for a role with no international dealings) is a validity risk. Finnish courts assess reasonableness.

### Scope of activities restricted
Must be related to the employee's actual duties and the employer's actual business. "Any employment in any competing company" without qualification is too broad.

## Output

| Issue | Status | Recommendation |
|---|---|---|
| Duration (max 12 months) | ✓/✗ | |
| Compensation (if >6 months) | ✓/✗ | |
| Weighty reason documented | ✓/✗ | |
| Scope reasonable | ✓/✗ | |

---

## Guardrail

Non-compete enforceability depends on facts and circumstances. Finnish courts have voided overly broad non-competes. This check identifies clear legal issues — full enforceability assessment requires legal advice.
```

- [ ] **Step 4: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/skills/bf-grant-check/ fi-startup-legal/skills/first-hire-contract/ fi-startup-legal/skills/non-compete-check/ && git commit -m "feat(fi-startup-legal): add grants and employment skills"
```

---

## Task 8: Commercial and governance skills (customer-contract, startup-nda, board-minutes)

- [ ] **Step 1: Write `fi-startup-legal/skills/customer-contract/SKILL.md`**

```markdown
---
name: customer-contract
description: >
  First customer contract checklist for a Finnish startup — liability cap,
  IP ownership, GDPR data processing clause, governing law (Finnish default),
  SLA. Reviews a contract or helps draft key terms. Use when signing first
  customer, asking "what should our customer contract say", "is this contract
  fair", or reviewing a customer's paper.
argument-hint: "[describe the deal or paste the contract]"
---

# /fi-startup-legal:customer-contract

1. Load profile.
2. Run the review/checklist. Route to `/fi-startup-legal:gdpr-basics` if data processing is involved.

---

## Key terms checklist

### IP ownership
We (the startup) must retain all IP in our product/platform. The customer gets a license to use, not ownership. Key question: does the contract try to claim ownership of anything we build?
- [ ] Our product IP clearly remains ours
- [ ] Customer data remains theirs (we are processor)
- [ ] Customisations/configurations: who owns them? (We should, unless paid bespoke work)

### Liability
- [ ] Limitation of liability clause present
- Our recommended starting position: liability capped at fees paid in 12 months preceding the claim
- Customer may push for higher cap for data breaches — consider accepting for: personal data breaches (GDPR exposure), death/personal injury (cannot be limited in Finland)
- [ ] Mutual limitation (not one-sided against us)
- [ ] Exclusion of consequential loss (indirect damages, lost profits)

### GDPR / data processing
Does the customer send us personal data to process on their behalf?
If yes: a Data Processing Agreement (DPA) is required by GDPR Art. 28 — route to `/fi-startup-legal:gdpr-basics`

### Governing law
Finnish law default for Finnish customers. For international customers:
- EU customers: Finnish law or the customer's local law — avoid US governing law
- Note: Finnish courts apply Finnish law; disputes in Finnish courts are in Finnish

### SLA (if relevant)
- Uptime commitment: 99.5% monthly (industry standard for SaaS)
- Scheduled maintenance windows excluded
- Remedies: service credits, not cash

### Termination
- Notice period: 30-90 days for convenience
- Data return/deletion obligations on termination
- Survival of confidentiality provisions

---

## Guardrail

Customer contracts are legally binding. This checklist covers key issues for a standard SaaS or service contract. For large enterprise contracts, pilot programs, or regulated sector customers, have a lawyer review before signing.
```

- [ ] **Step 2: Write `fi-startup-legal/skills/startup-nda/SKILL.md`**

```markdown
---
name: startup-nda
description: >
  NDA review and generation for Finnish startups — mutual vs one-way,
  EU Trade Secrets Directive (Liikesalaisuuslaki 595/2018) framing, duration,
  scope. Use when reviewing or drafting an NDA, asking "is this NDA okay",
  "NDA with investor", "NDA with employee".
argument-hint: "[paste NDA or describe: mutual/one-way, with whom, purpose]"
---

# /fi-startup-legal:startup-nda

1. Load profile.
2. Call `mcp__finlex__get_statute("LSL", "2")` — trade secret definition under Finnish law.
3. Review or generate NDA. Output is draft — attorney review for high-stakes NDAs.

---

## Finnish NDA framework

The Business Secrets Act (Liikesalaisuuslaki 595/2018) implements the EU Trade Secrets Directive in Finland. Key definition from live statute:

A trade secret is information that:
1. Is secret (not generally known or readily accessible)
2. Has commercial value because it is secret
3. Has been subject to reasonable steps to keep it secret

### Review checklist

| Term | Finnish market standard | Status |
|---|---|---|
| Definition of confidential information | Broad but excluding: public domain, prior knowledge, independent development, required disclosure | |
| Mutual vs one-way | Mutual for most startup contexts; one-way acceptable for investor NDA | |
| Duration of obligation | 2–5 years standard; perpetual for true trade secrets (acceptable) | |
| Return/destruction | On request or termination | |
| Governing law | Finnish law | |
| Dispute resolution | Finnish courts or arbitration | |
| Remedies | Injunction + damages (LSL §11) | |

### Investor NDA note
Finnish VCs and angels typically do NOT sign NDAs before a first meeting — standard practice. If an investor insists on signing before seeing your deck, it may signal inexperience. Consider sharing your deck without requiring an NDA for early-stage fundraising conversations.

### Employee NDA
Employment contracts should include confidentiality obligations. A separate NDA is typically redundant for employees — the employment contract + TSL Chapter 3 confidentiality provisions cover it.

---

## Guardrail

NDAs are enforceable contracts. For high-value situations (sharing trade secrets with a strategic partner, M&A diligence), have a lawyer review or draft.
```

- [ ] **Step 3: Write `fi-startup-legal/skills/board-minutes/SKILL.md`**

```markdown
---
name: board-minutes
description: >
  Draft Finnish startup board meeting minutes — OYL-compliant format, quorum
  check, decision types, signing requirements. Use when asking "draft board
  minutes", "what needs board approval", or after a board meeting.
argument-hint: "[describe decisions made, or paste meeting notes]"
---

# /fi-startup-legal:board-minutes

1. Load profile.
2. Call `mcp__finlex__get_statute("OYL", "6")` — board rules, quorum, decision-making.
3. Draft minutes from the meeting notes/decisions described. Draft for review before signing.

---

## OYL board governance (from live statute)

From fetched OYL §6 text, apply current rules for:

**Quorum:** majority of board members must be present (unless SHA requires higher threshold).

**Decision-making:** simple majority of those present, unless SHA specifies otherwise.

**Signing:** minutes must be signed by the chair and at least one board member present (or all members who participated).

## Decision types

**Board decisions (hallitus):** day-to-day management, operational decisions, entering contracts below the threshold in SHA, hiring/firing employees below CEO level.

**Shareholder/AGM decisions (yhtiökokous):** required for share issuance, option schemes, changes to yhtiöjärjestys, auditor appointment, dividend, merger/demerger, dissolution.

**Items typically requiring board approval:**
- Contracts above the threshold defined in SHA
- Hiring the CEO
- Opening/closing bank accounts
- Taking on debt above threshold
- Any matter with potential conflict of interest (must be disclosed)

## Minutes structure

```
[COMPANY NAME] OY — BOARD MEETING MINUTES

Date: [date]
Location / method: [physical / Teams / written procedure]
Present: [names + roles]
Chair: [name]
Minutes secretary: [name]

§1 Opening and quorum
The chair opened the meeting. Quorum confirmed: [N] of [N] members present.

§2 Agenda
[List items]

§3 [Agenda item 1]
Proposal: [text]
Decision: The board resolved [text] / Passed unanimously / Passed [N]-[N].

[Repeat for each item]

§X Closing
The chair closed the meeting.

Signatures:
Chair: _______________
Board member: _______________
```

---

## Guardrail

Board minutes are legal documents. Ensure they are signed promptly after the meeting. Decisions not properly documented in signed minutes may be challenged. Store signed minutes securely — they are required for M&A due diligence.
```

- [ ] **Step 4: Validate and commit**

```bash
claude plugin validate ./fi-startup-legal
git add fi-startup-legal/skills/customer-contract/ fi-startup-legal/skills/startup-nda/ fi-startup-legal/skills/board-minutes/ && git commit -m "feat(fi-startup-legal): add commercial and governance skills"
```

---

## Task 9: Privacy skill (gdpr-basics) + README + final validation

- [ ] **Step 1: Write `fi-startup-legal/skills/gdpr-basics/SKILL.md`**

```markdown
---
name: gdpr-basics
description: >
  Minimum GDPR compliance for Finnish startups — privacy notice, cookie consent,
  first DPA with processors, DSAR process. Routes to Velvoite for DPIA if needed.
  Use when asking "do we need a privacy policy", "GDPR for our startup",
  "cookie consent", or "first DPA".
argument-hint: "[describe your product and what user data you handle]"
---

# /fi-startup-legal:gdpr-basics

1. Load profile. Check AI flag and Velvoite availability.
2. Run the GDPR basics checklist.
3. If DPIA or AI Act check needed: route to eu-legal or suggest Velvoite for depth.

---

## GDPR startup minimum

### Do you need to comply?
Yes, if you: (a) are established in the EU, or (b) offer goods/services to EU individuals, or (c) monitor EU individuals. Finnish startups: always yes.

### Privacy notice (tietosuojaseloste)
Required by GDPR Art. 13/14 before collecting personal data. Must cover:
- [ ] Who you are (controller identity + contact)
- [ ] What data you collect
- [ ] Why (purposes) and legal basis for each purpose (Art. 6: contract / legitimate interests / consent / legal obligation)
- [ ] How long you keep it
- [ ] Who you share it with (categories of recipients, transfers outside EU)
- [ ] Data subject rights (access, erasure, correction, portability, objection)
- [ ] Right to complain to Tietosuojavaltuutettu (Finnish DPA)
- [ ] Contact for DPO (if appointed) or privacy queries

### Cookie consent
ePrivacy Directive (implemented in Finland via Sähköisen viestinnän palvelulaki) requires consent before non-essential cookies. Functional/strictly necessary cookies: no consent required. Analytics, advertising, tracking: consent required before setting.

### First processor DPA
If you use any service that processes your users' personal data on your behalf (Stripe, Mailchimp, AWS, Google Analytics, Intercom, Mixpanel, etc.) — you need a Data Processing Agreement with them. Most have standard DPAs available in their terms/settings.

### Record of Processing Activities (RoPA)
Required if: >250 employees, OR processing is not occasional, OR high-risk. Most early startups: not required, but good practice to maintain one.

### DSAR process
Have a named person or email address for data subject requests. Must respond within 1 month (extendable to 3 for complex requests).

### Breach notification
Personal data breach must be notified to Tietosuojavaltuutettu within 72 hours if likely to result in risk to individuals. Keep a breach register regardless.

---

## Velvoite depth (optional)

If you're an AI startup processing significant personal data, or building a product involving profiling, automated decisions, or biometrics — add your Velvoite API key for:
- Full DPIA workflow (GDPR Art. 35 mandatory trigger check)
- AI Act compliance check (risk classification, FRIA)
- Enforcement intelligence (what Finnish/EU startups get fined for)

Free 30-day trial at velvoite.eu.

---

## Guardrail

GDPR compliance is ongoing, not a one-time checklist. Finnish Tietosuojavaltuutettu (TSV) can issue fines up to €20M or 4% of global turnover. This checklist covers the minimum for an early-stage startup — as you grow, engage a privacy lawyer.
```

- [ ] **Step 2: Write `fi-startup-legal/README.md`**

```markdown
# fi-startup-legal — Finnish Startup Legal Toolkit for Claude Code

A Claude Code / Cowork plugin for Finnish startup founders and early ops hires.
Covers the pre-seed through Series A journey. Backed by live Finnish statute data
(Finlex), the Finnish Trade Register (PRH), and optional Velvoite EU finreg corpus.

Built by [Silly Pilot Oy](https://velvoite.eu).

## What it covers

| Skill | What it does |
|---|---|
| `/fi-startup-legal:oy-setup` | OY incorporation wizard — yhtiöjärjestys, osakassopimus, PRH registration |
| `/fi-startup-legal:founder-agreement` | Co-founder mechanics + vesting + IP assignment |
| `/fi-startup-legal:ip-assignment` | IP audit — founders, employees, contractors, BF grants |
| `/fi-startup-legal:sha-review` | SHA review against seriesseed.fi standard |
| `/fi-startup-legal:term-sheet-review` | Term sheet in plain language — red/amber/green per term |
| `/fi-startup-legal:esop-designer` | TVL §66 vs §66a decision + 2026 reform + AGM mechanics |
| `/fi-startup-legal:vesting-calculator` | Vesting schedule + leaver calculations |
| `/fi-startup-legal:bf-grant-check` | Business Finland IP restrictions + clawback risk |
| `/fi-startup-legal:first-hire-contract` | First employment contract checklist |
| `/fi-startup-legal:non-compete-check` | Non-compete validity + compensation check |
| `/fi-startup-legal:customer-contract` | First customer contract review |
| `/fi-startup-legal:startup-nda` | NDA review + generation (LSL framework) |
| `/fi-startup-legal:board-minutes` | OYL-compliant board minutes |
| `/fi-startup-legal:gdpr-basics` | GDPR minimum for startups |

## What makes it different

Skills call **live Finnish statute data** via Finlex MCP — TSL notice periods,
OYL share rules, TVL §66 ESOP provisions — not training-data guesses.
Company lookups go to **live PRH data**. The 2026 TVL §66 option tax reform
(options now taxed at sale, not exercise) is reflected in real time.

## Install

```bash
claude plugin install github:Velvoite/fi-startup-legal
```

## Setup

```
/fi-startup-legal:startup-cold-start
```

## Requirements

- Claude Code or Cowork
- No API key required for core features (Finlex + PRH are public)
- Velvoite API key for GDPR/AI Act depth (free trial: velvoite.eu)

## License

Apache 2.0 — Copyright 2026 Silly Pilot Oy
```

- [ ] **Step 3: Full validation sweep**

```bash
claude plugin validate ./fi-startup-legal
find fi-startup-legal/skills -name 'SKILL.md' | wc -l
```

Expected: validation passes, 15 skills.

- [ ] **Step 4: Final commit**

```bash
git add fi-startup-legal/ && git commit -m "feat(fi-startup-legal): Plan complete — 15 skills, README, Apache 2.0"
```
