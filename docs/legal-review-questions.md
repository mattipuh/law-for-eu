# eu-legal plugin — legal team review questions

We're building a Claude Code / Cowork plugin for EU-regulated financial institutions.
Six quick questions before we lock the skill behavior. Happy with any answer — just need a direction.

---

**1. Works council — country coverage**
For the `works-council-check` skill (which flags when a works council must be consulted),
we're planning to cover: Finland, Germany, France, Netherlands, Sweden in v1.
Is that the right set, or should we narrow to FI + DE only and expand later?

---

**2. Termination review — output format**
Should the `termination-review` skill produce:
- A **decision memo** (here's our analysis, here's the risk, here's our recommendation), or
- A **checklist** of required procedural steps (notice period, works council consultation, documentation)?

The memo carries more legal opinion risk; the checklist is safer but less actionable.

---

**3. AI Act compliance — depth of output**
The `ai-act-compliance` skill classifies AI systems under the EU AI Act and checks
GDPR + AI Act overlap. Should it:
- Produce a **full FRIA draft** (Fundamental Rights Impact Assessment), or
- Produce a **triage output** that assesses risk level and routes high-risk systems to outside counsel for the actual assessment?

---

**4. DORA in vendor agreements**
The `vendor-agreement-review` skill surfaces DORA ICT third-party requirements inline
when the customer is a financial institution. Should it also:
- **Draft the required contractual provisions** (DORA Art. 30 minimum contract clauses), or
- **Flag the gaps only** and leave drafting to counsel?

---

**5. ePrivacy**
The `reg-gap-analysis` skill in the privacy area — should it track the
**ePrivacy Regulation draft** (still in trilogue) or only reference the
existing **ePrivacy Directive (2002/58/EC)**? The draft has been stalled for years
but tracking it would be useful for clients planning ahead.

---

**6. Litigation out of v1**
We've deprioritised a litigation / disputes skill for v1 (EU civil procedure
is a substantial build and fundamentally different from the US model in the
base repo). Are you comfortable with the gap being covered by routing to
outside counsel via the `escalation-flagger` skill for now?
