# eu-legal Retrofit — Finlex + PRH MCP Integration Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add `finlex-mcp` and `prh-mcp` to eu-legal's MCP server list, then update 6 existing skills to call live Finlex statute data instead of relying on training data for Finnish law citations.

**Architecture:** Add two entries to `eu-legal/.mcp.json`. Update the 6 Finnish-law-dependent skills to call `mcp__finlex__get_statute()` at the start of their workflow, using the live statute text to populate notice period tables, thresholds, and statutory references.

**Tech Stack:** Claude plugin SKILL.md edits only. No build step. Validate with `claude plugin validate ./eu-legal`.

**Prerequisite:** finlex-mcp and prh-mcp must be deployed before these skills can make live calls.

---

## Files modified

```
eu-legal/.mcp.json                                    ← add finlex + prh servers
eu-legal/skills/termination-review/SKILL.md           ← TSL §6:3 notice periods
eu-legal/skills/works-council-check/SKILL.md          ← YTL §51 thresholds
eu-legal/skills/leave-review/SKILL.md                 ← VLL accrual rates
eu-legal/skills/worker-classification/SKILL.md        ← TSL §1:1 employment test
eu-legal/skills/entity-compliance/SKILL.md            ← PRH company lookup
eu-legal/skills/nda-review/SKILL.md                   ← LSL §2 trade secret definition
```

---

## Task 1: Add finlex-mcp and prh-mcp to eu-legal/.mcp.json

- [ ] **Step 1: Read current `.mcp.json`**

```bash
cat eu-legal/.mcp.json
```

- [ ] **Step 2: Add the two new servers**

Open `eu-legal/.mcp.json` and add to `mcpServers`:

```json
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
}
```

- [ ] **Step 3: Validate JSON**

```bash
python3 -c "import json; json.load(open('eu-legal/.mcp.json')); print('OK')"
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/.mcp.json && git commit -m "feat(eu-legal): add finlex-mcp and prh-mcp to .mcp.json"
```

---

## Task 2: Update termination-review with live TSL notice periods

**File:** `eu-legal/skills/termination-review/SKILL.md`

- [ ] **Step 1: Read current skill**

```bash
head -30 eu-legal/skills/termination-review/SKILL.md
```

- [ ] **Step 2: Add Finlex call at top of Finnish workflow section**

Find the section that handles Finnish termination (TSL notice periods). Before presenting notice period tables, add:

```markdown
### Live statute lookup

Call `mcp__finlex__get_statute("TSL", "6:3")` to retrieve current employer notice periods.
Call `mcp__finlex__get_statute("TSL", "6:4")` to retrieve current employee notice periods.

Present the notice period table extracted from the live statute text. Do NOT use hardcoded notice period values — use the live text in case of legislative amendments.
```

- [ ] **Step 3: Validate**

```bash
claude plugin validate ./eu-legal
```

- [ ] **Step 4: Commit**

```bash
git add eu-legal/skills/termination-review/SKILL.md && git commit -m "feat(eu-legal): termination-review uses live TSL §6:3-4 via Finlex"
```

---

## Task 3: Update works-council-check with live YT-laki thresholds

**File:** `eu-legal/skills/works-council-check/SKILL.md`

- [ ] **Step 1: Add Finlex call for YT-laki cooperation thresholds**

Find the Finland (YT-laki) section. Before presenting threshold tables, add:

```markdown
### Live statute lookup

Call `mcp__finlex__get_statute("YTL", "51")` for cooperation negotiation requirements (collective redundancies).
Call `mcp__finlex__get_statute("YTL", "16")` for general cooperation obligation scope.

Present thresholds and negotiation period requirements from the live statute text.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./eu-legal
git add eu-legal/skills/works-council-check/SKILL.md && git commit -m "feat(eu-legal): works-council-check uses live YTL §16 and §51 via Finlex"
```

---

## Task 4: Update leave-review with live Vuosilomalaki accrual

**File:** `eu-legal/skills/leave-review/SKILL.md`

- [ ] **Step 1: Add Finlex call for holiday entitlement**

Find the Finnish leave section. Add:

```markdown
### Live statute lookup

Call `mcp__finlex__get_statute("VLL", "5")` for annual holiday accrual rates (2/2.5 days per month).
Call `mcp__finlex__get_statute("VLL", "7")` for holiday pay calculation.

Present accrual rates from live statute text.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./eu-legal
git add eu-legal/skills/leave-review/SKILL.md && git commit -m "feat(eu-legal): leave-review uses live VLL §5 and §7 via Finlex"
```

---

## Task 5: Update worker-classification with live TSL employment test

**File:** `eu-legal/skills/worker-classification/SKILL.md`

- [ ] **Step 1: Add Finlex call for employment relationship definition**

Find the Finland TSL employment test section. Add:

```markdown
### Live statute lookup

Call `mcp__finlex__get_statute("TSL", "1:1")` for the statutory definition of employment relationship.

Present the three-element test from the live statute:
1. Work performed for another party
2. Under direction and supervision
3. For remuneration
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./eu-legal
git add eu-legal/skills/worker-classification/SKILL.md && git commit -m "feat(eu-legal): worker-classification uses live TSL §1:1 via Finlex"
```

---

## Task 6: Update entity-compliance with PRH live lookup

**File:** `eu-legal/skills/entity-compliance/SKILL.md`

- [ ] **Step 1: Add PRH lookup at start of Finnish entity workflow**

Find the Finland OY section. Add at the top:

```markdown
### Live company lookup

If business ID is known from profile: call `mcp__prh__lookup_company(business_id)`.
Returns: company name, status (Active/Dissolved), registered address, company form.

Use returned data to pre-populate the compliance check. If status = Dissolved: alert immediately.
If no business ID in profile: call `mcp__prh__search_companies(company_name)` to find it.
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./eu-legal
git add eu-legal/skills/entity-compliance/SKILL.md && git commit -m "feat(eu-legal): entity-compliance uses live PRH company lookup"
```

---

## Task 7: Update nda-review with live LSL trade secret definition

**File:** `eu-legal/skills/nda-review/SKILL.md`

- [ ] **Step 1: Add Finlex call for trade secret definition**

Find the Finnish/EU Trade Secrets section. Add:

```markdown
### Live statute lookup

Call `mcp__finlex__get_statute("LSL", "2")` for the current Finnish trade secret definition.

Present the three-element definition from the live statute text:
1. Information is secret
2. Has commercial value because it is secret
3. Subject to reasonable steps to keep it secret
```

- [ ] **Step 2: Validate and commit**

```bash
claude plugin validate ./eu-legal
git add eu-legal/skills/nda-review/SKILL.md && git commit -m "feat(eu-legal): nda-review uses live LSL §2 trade secret definition via Finlex"
```

---

## Task 8: Final validation sweep

- [ ] **Step 1: Full validation**

```bash
claude plugin validate ./eu-legal
```

- [ ] **Step 2: Confirm skill count unchanged**

```bash
find eu-legal/skills -name 'SKILL.md' | wc -l
```

Expected: 28 (no new skills added, only existing ones updated).

- [ ] **Step 3: Push to origin**

```bash
git push origin main
```
