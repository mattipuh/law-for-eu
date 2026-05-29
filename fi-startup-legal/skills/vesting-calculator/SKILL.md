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
