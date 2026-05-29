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
2. Call `mcp__velvoite__get_finnish_statute("OYL", "6")` — board rules, quorum, decision-making.
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
