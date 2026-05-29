# eu-legal Plugin — Design Spec

**Date:** 2026-05-29
**Author:** Matti Puhakka / Silly Pilot Oy
**Status:** Approved for implementation planning

---

## Problem

`anthropics/claude-for-legal` is a US-centric legal plugin suite. Its regulatory backbone runs on the Federal Register API; its employment skills are built on FLSA/FMLA/EEOC; its corporate skills default to Delaware. Velvoite (velvoite.eu) already provides a deep EU financial regulation corpus (DORA, MiCA, MiFID II, GDPR, AI Act, AML, Solvency II and more) via MCP, and a working `eu-finreg` plugin. What is missing is the generic legal operations layer: commercial contracts, employment, corporate governance, privacy workflows — adapted for EU law and EU-regulated institutions specifically.

## Goal

Build `eu-legal`: a single Claude Code/Cowork plugin that gives EU-regulated financial institutions a complete legal automation stack. Generic EU legal ops skills + Velvoite finreg corpus as the depth layer. One install, one profile, one coherent product.

---

## Target user

**Primary:** Legal teams at EU-regulated financial institutions — credit institutions, payment institutions, e-money institutions, CASPs, investment firms, insurance companies. These teams handle both regulatory compliance and everyday legal ops (vendor contracts, employment, M&A) from the same desk.

**Secondary (falls out for free):** General EU in-house counsel. The EU law skills are jurisdiction-neutral at the EU directive level — they work for any EU company, not just financial institutions. Velvoite depth is the differentiator for finreg customers; the general skills work without it.

**Geography:** EU-wide as baseline. Finland (FIN-FSA) and Germany (BaFin) as the two named national jurisdictions in v1, matching the Velvoite corpus coverage.

---

## Architecture

### Single plugin, unified profile

`eu-legal` is one plugin directory. All practice area skills live inside it. One cold-start interview writes one profile file. Every skill reads that profile.

No multi-plugin complexity. The upstream repo's separate-plugin structure is designed for a general marketplace — `eu-legal` is a product for a specific customer type and should be installed as a unit.

### Plugin structure

```
eu-legal/
  .claude-plugin/
    plugin.json                  # name: eu-legal, author: Silly Pilot Oy
  .mcp.json                      # velvoite MCP server declaration
  CLAUDE.md                      # practice profile template (placeholders only)
  hooks/
    hooks.json
  agents/
    reg-feed-watcher.md          # weekly EU regulatory digest
    renewal-watcher.md           # contract renewal alerts
  skills/
    cold-start-interview/        # unified onboarding
    customize/                   # change one thing without re-running cold-start
    matter-workspace/            # per-matter context isolation (adapted from upstream)
    commercial/
      SKILL.md                   # dispatcher — routes to sub-skills
      contract-review/
      nda-review/
      vendor-agreement-review/
      escalation-flagger/
      playbook-init/
      renewal-tracker/
    privacy/
      SKILL.md
      dsar-response/
      dpa-review/
      pia-generation/
      use-case-triage/
      policy-monitor/
      reg-gap-analysis/
      ai-act-compliance/
    employment/
      SKILL.md
      worker-classification/
      termination-review/
      works-council-check/
      leave-review/
      international-expansion/
      policy-drafting/
    corporate/
      SKILL.md
      entity-compliance/
      board-minutes/
      written-consent/
      m-a-diligence-summary/
      closing-checklist/
    regulatory/
      SKILL.md
      reg-watch/
      reg-gap-analysis/
      obligations/
      deadlines/
```

### Profile architecture

The cold-start interview writes to:
`~/.claude/plugins/config/eu-legal/CLAUDE.md`

Profile captures:
- **Company name and entity type** — drawn from the Velvoite taxonomy (credit_institution, payment_institution, e_money_institution, casp, investment_firm, aifm, ucits, insurance, reinsurance). Entity type drives which regulatory obligations surface in regulatory and commercial skills.
- **Primary jurisdiction + operating countries** — EU-wide baseline; specific countries (FI, DE, FR, NL, etc.) enable country-specific skill branches (works council rules, termination notice periods, entity filing requirements).
- **Active practice areas** — customer selects which domains they use. Skills for inactive areas are suppressed from `/help` output.
- **Velvoite workspace ID and API key env var name** — enables regulatory depth. Skills degrade gracefully without it (general EU guidance instead of corpus-specific obligations).
- **Legal team structure** — solo GC, small team, or team with outside counsel. Affects escalation routing and output verbosity.
- **Outside counsel relationships** — named firms per jurisdiction, for escalation skills.

### Velvoite as depth layer, not hard dependency

`.mcp.json` declares the Velvoite MCP server (`https://mcp.velvoite.eu/mcp`, auth via `VELVOITE_API_KEY`). Skills that touch regulatory territory call it when the key is present. Without the key, skills return general EU law guidance with a prompt to add the key for obligation-specific answers.

This means:
1. The plugin can be demoed and trialled without a Velvoite account.
2. The API key is the natural upsell moment — it appears in the output of any skill that would benefit from it.
3. Financial institution customers who already have Velvoite get immediate depth; general EU in-house counsel can use the plugin standalone.

---

## Practice area skills — v1 scope

### Commercial (6 skills)

**Source:** Adapted from `commercial-legal` upstream. Moderate rewrite — structure reusable, US defaults replaced.

| Skill | Description | Key EU changes |
|---|---|---|
| `contract-review` | Review any commercial agreement | EU governing law defaults; GDPR data processing clause check; DSA/DMA applicability flag |
| `nda-review` | Review NDAs | EU Trade Secrets Directive (2016/943) framing; no US common law trade secret analysis |
| `vendor-agreement-review` | Review vendor/supplier agreements | Calls Velvoite `get_canonical_obligations(regulation='dora', actor_role='financial_entity')` inline when entity is a financial institution — surfaces ICT third-party requirements from the corpus without the user having to know to ask |
| `escalation-flagger` | Flag non-standard contract positions | EU-calibrated thresholds (€ not $); EU norm deviations (jurisdiction, limitation of liability caps under applicable law) |
| `playbook-init` | Set up standard negotiating positions | EU jurisdiction defaults: governing law options (Finnish law, German law, English law), dispute resolution (ICC/SCC/LCIA, not AAA/JAMS) |
| `renewal-tracker` | Track expiring contracts | Jurisdiction-neutral; adapted from upstream |

### Privacy (7 skills)

**Source:** Adapted from `privacy-legal` upstream — the strongest EU-ready plugin in the repo. Light touch on most skills; ai-act-compliance is new.

| Skill | Description | Key EU changes |
|---|---|---|
| `dsar-response` | Handle data subject access/deletion/portability/correction requests | Keep GDPR flow; strip CCPA-first framing and US-specific rights language |
| `dpa-review` | Review data processing agreements | Keep — already GDPR-native with controller/processor split |
| `pia-generation` | Generate PIAs and DPIAs | Keep structure; add GDPR Art. 35 mandatory DPIA trigger check before starting |
| `use-case-triage` | Does this processing need a DPIA? | Keep; remove FTC Act §5 and CCPA paths |
| `policy-monitor` | Detect drift between privacy policy and actual practice | Keep |
| `reg-gap-analysis` | Gap analysis when a new regulation drops | Point at EU regs: GDPR, ePrivacy Regulation, AI Act data provisions, national GDPR implementations |
| `ai-act-compliance` | **New** — AI system compliance for GDPR + EU AI Act | Risk classification (prohibited / high-risk / limited risk / minimal risk); FRIA generation (Art. 27 EU AI Act); combined GDPR + AI Act analysis for systems processing personal data; pre-mapped Annex III high-risk categories for financial institutions (credit scoring, fraud detection, insurance underwriting, employment decisions); flags where DPIA and FRIA overlap and can be run as a joint assessment; adapted from ai-governance-legal upstream which already has solid EU AI Act coverage |

### Employment (6 skills)

**Source:** Near-full rewrite of `employment-legal`. Most US-specific plugin in the upstream repo — FLSA, FMLA, EEOC, state wage/hour law all replaced.

| Skill | Description | EU build |
|---|---|---|
| `worker-classification` | Employee vs. independent contractor | EU worker status tests (Platform Work Directive presumption of employment, national tests for FI/DE/FR); replaces FLSA/ABC test entirely |
| `termination-review` | Review termination decisions | EU notice periods under EU directives + country-specific (FI: YT-laki, DE: KSchG, FR: Code du travail); works council consultation triggers; no OWBPA/EEOC references |
| `works-council-check` | **New** — when must a works council be consulted? | Routes by country: Germany (BetrVG — mandatory for 5+ employees), Finland (YT-laki — cooperation negotiations), France (CSE — mandatory from 11 employees), Netherlands (WOR), Sweden (MBL); flags headcount and transaction thresholds |
| `leave-review` | Leave entitlements and obligations | EU Working Time Directive (2003/88/EC), Parental Leave Directive (2019/1158); country-specific entitlements for FI/DE; replaces FMLA/COBRA entirely |
| `international-expansion` | Hiring in a new EU country | EU-first (not US-parent-expanding); covers posted workers (Directive 2018/957), social security coordination (Reg 883/2004), local employment contracts |
| `policy-drafting` | Draft employment policies | EU defaults: remote work policy (GDPR Art. 88 employee data monitoring limits), equal treatment, whistleblower policy (EU Whistleblower Directive 2019/1937) |

### Corporate (5 skills)

**Source:** Rewrite entity management; keep M&A skeleton with EU additions.

| Skill | Description | EU build |
|---|---|---|
| `entity-compliance` | Annual filing and maintenance obligations | EU company forms: OY/AB (Finland), GmbH/AG (Germany), SAS/SA (France), BV/NV (Netherlands), SA/NV (Belgium/Luxembourg); national company registers (PRH, Handelsregister, RCS, KVK); replaces Delaware/Secretary of State entirely |
| `board-minutes` | Draft board minutes | EU corporate governance code framing; two-tier vs. one-tier board awareness by jurisdiction; no US §102/§141 DGCL references |
| `written-consent` | Board/shareholder written resolutions | Adapted — concept exists across EU jurisdictions but mechanics vary; routes by country |
| `m-a-diligence-summary` | M&A diligence issue extraction and summary | Adds EU merger control trigger check (EC Regulation 139/2004 thresholds: €5B worldwide + €250M EU-wide turnover); adapted from corporate-legal upstream |
| `closing-checklist` | M&A transaction closing checklist | EU closing mechanics; jurisdiction-specific filing requirements; no Delaware Chancery or Secretary of State steps |

### Regulatory (4 skills — Velvoite MCP + EUR-Lex general feed)

**Source:** New. `eu-legal` declares the Velvoite MCP in its own `.mcp.json` — these skills call it directly. eu-finreg does not need to be installed. Users who have eu-finreg installed separately still benefit (both plugins share the same MCP server); eu-finreg's deeper skills remain available but are not required.

| Skill | Description | Build |
|---|---|---|
| `reg-watch` | What changed in EU regulation this week | Calls `mcp__velvoite__get_recent_changes` scoped to entity type from profile; adds EUR-Lex general legislative pipeline feed for non-finreg regulations (Commission proposals, adopted acts) |
| `obligations` | What obligations apply to us | Calls `mcp__velvoite__get_canonical_obligations` + `get_obligation_summary`; passes entity type and actor roles from profile automatically |
| `deadlines` | Upcoming compliance dates | Calls `mcp__velvoite__get_deadlines`; 90-day horizon default, configurable |
| `reg-gap-analysis` | Gap between current practice and a new regulation | EU legislative process framing: Commission proposal → trilogue → adoption → transposition; calls Velvoite for obligation detail when regulation is in corpus; no US notice-and-comment process |

---

## Agents (scheduled)

### `reg-feed-watcher`
Weekly EU regulatory digest. Adapted from eu-finreg's agent of the same name. Calls the regulatory skills, posts to Cowork inbox or Slack. Cadence: Monday 07:00 local time.

### `renewal-watcher`
Commercial contract renewal alerts. Adapted from commercial-legal upstream's `renewal-watcher.md`. Checks renewal-tracker data, alerts 90/30/7 days before expiry.

---

## What is dropped from the fork (v1)

| Plugin | Reason |
|---|---|
| `law-student` | US bar exam prep — not applicable |
| `legal-clinic` | US clinical framework — not applicable |
| `legal-builder-hub` | Meta-plugin for a general marketplace — not needed for a product |
| `litigation-legal` | EU civil procedure is a substantial build; EU discovery model is fundamentally different from FRCP; out of v1 scope |
| `ip-legal` | EUIPO clearance skill exists upstream and is good; defer to v2 |

The existing plugin directories remain in the fork repo as reference material and for potential upstream contribution back. The `eu-legal/` directory is the product.

---

## v2 candidates (not in scope now)

- **`ip-legal` EU adaptation** — build on the existing EUIPO clearance skill; add EU patent (EPO/national offices), EUIPO opposition, EU copyright (DSM Directive)
- **`litigation-legal` EU adaptation** — EU civil procedure (no FRCP discovery, ICC/SCC/LCIA arbitration, national court systems), privilege under EU competition law (Akzo Nobel), England & Wales CPR
- **Multi-language support** — Finnish and German skill output variants; particularly relevant for works council documentation (German: BetrVG mandates German-language communications)
- **Modular sub-plugin split** — once customer usage data shows which practice areas are actually used, split into independently versioned sub-plugins

---

## Open questions for legal team review

1. **Works council coverage** — is FI/DE/FR/NL/SE the right country set for `works-council-check` v1, or should we narrow to FI/DE only and expand later?
2. **Termination review depth** — should `termination-review` produce a decision memo or just a checklist of required steps? Different risk profile for each.
3. **AI Act risk classification** — should `ai-act-compliance` produce a full FRIA document or a triage output that routes to external counsel for the actual assessment?
4. **DORA scope in vendor-agreement-review** — the skill surfaces DORA ICT third-party obligations inline for financial institutions. Should it also draft contract language (e.g., DORA-required contractual provisions under Art. 30), or only flag gaps?
5. **ePrivacy** — the ePrivacy Regulation is still in trilogue. Should `reg-gap-analysis` in privacy track the current draft, or only cite the existing ePrivacy Directive (2002/58/EC)?
6. **Litigation-legal out of v1** — confirm this is acceptable. EU financial institutions do get sued and receive regulatory investigations. The gap could be partially covered by routing to outside counsel via escalation-flagger.
