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
| Regulation | Total obligations | Notes |
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
