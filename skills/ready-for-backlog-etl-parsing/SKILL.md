---
name: ready-for-backlog-etl-parsing
description: >
  Evaluate whether an ETL field-parsing or object-modification story is ready
  for the backlog. Use when the story adds columns to existing tables, updates
  stored procedure logic, modifies existing views, or updates calculations in
  existing FACT/dimension objects. Always trigger when user mentions "add field",
  "new column", "update SP", "modify stored procedure", "update view", "update
  calculation", "DeFi XML", "custom field", "field mapping", or "backfill" in
  the context of an existing table or object.
---

# Ready for Backlog — ETL: Modify Existing Object (Parsing / Field Additions)

## Setup

Before evaluating the story, read and internalize the `ready-for-backlog` parent skill.
That skill defines the universal readiness framework — story structure rules, the INVEST
model, the base blocking issues, advisory items, stakeholder question patterns, and the
output format. This child skill adds ETL-parsing-specific rules on top of it.

Apply all parent rules first. Then apply the additional rules in this document.

---

## Story Type

**Type:** ETL — Modify Existing Object

**Signals:** add field, new column, update stored procedure (SP), modify view, update
calculation, update FACT table, update dimension table, DeFi XML, custom field, field
mapping, SOR rule, XPath, backfill, FACTAPPRULES, SORRULEID, existing object, existing
pipeline, existing flow.

If the story creates a **net-new** table, view, schema, or pipeline, stop and use
`ready-for-backlog-etl-new-object` instead.

---

## Type-Specific Pre-Grooming Blockers

*These are additions to the parent's blocking issue list. Apply parent blockers first.*

**Field Mapping — No field mapping table present**
A field mapping table must be present with at minimum: source system field name or XPath,
QA field/custom ID, Prod field/custom ID (or clearly marked TBD), data type, and
transformation logic or formula. A story that says "add field X from DeFi" without
this table is not estimable.
💬 Ask: "Can you provide the field mapping table with QA ID, Prod ID (if known), data
type, and transformation logic for each field being added?"

**DeFi XML — Field reference is a display name only**
If the story references a DeFi custom field by its UI display name only (e.g.,
"Proof of Income Type") without the field ID, XPath, or custom field number, the
developer cannot locate it in the XML envelope.
💬 Ask: "What is the DeFi custom field ID or XML element name for '[field name]'?
The display name alone isn't enough for the developer to parse the XML."

**Backfill — No backfill decision**
When a column is being added to an existing table that already has rows, the story must
include at least a one-sentence statement on backfill: will historical records be
backfilled, or will existing rows remain NULL? Silence on this point is a blocker.
💬 Ask: "Should existing records in [table name] be backfilled with this field's value,
or will only new records going forward be populated?"

**Downstream Impact — Column rename or type change with no consumer assessment**
If the story renames an existing column or changes its data type, it must identify which
downstream views, flows, or reports consume that column. Failing to do so can cause
silent production breaks.
💬 Ask: "Are there existing views, Prefect flows, or Power BI reports that read
[column name]? Those objects will break if the column is renamed or its type changes."

---

## Type-Specific Grooming Discussion Items

*Raise these in grooming. They are not blockers but should be resolved before or during
sprint planning.*

- **TBD Prod IDs** — If QA IDs are known but Prod IDs are marked TBD, confirm whether
  Prod IDs will be available before development begins, or whether the developer should
  proceed with QA first. This is a workflow question, not a blocker.
- **Null / missing source data handling** — What should the field contain when the source
  XML does not include the element? NULL, empty string, or a default value? Discuss and
  confirm before or during development.
- **Backfill timing and priority** — Should the backfill run immediately after the field
  is deployed, or is it a separate follow-up ticket? If separate, is the backfill ticket
  already in the backlog?
- **Historical data volume** — Approximately how many records need to be backfilled?
  Large backfills (millions of rows) affect sizing and may require a separate execution
  strategy.

---

## Type-Specific Advisory Items

*Non-blocking. Note these to the BSA for awareness.*

- **AC does not cover null / missing source data** — The happy-path AC may be present,
  but no criterion covers what happens when the source field is absent or null in the
  source system. Add an edge case criterion.
- **Field mapping table present but Prod ID column absent** — If the mapping table has
  no Prod ID column at all (not even a TBD placeholder), flag this — the developer will
  need to know where to record it when Prod IDs are confirmed.
- **No mention of downstream dependencies** — If columns are being modified but the story
  doesn't mention known consumers (views, reports, flows), note this as an awareness
  item even if no breaking changes are expected.

---

## Type-Specific Judgment Guidelines

**TBD Prod IDs:** Not a blocker. If the QA ID is known and transformation logic is clear,
a developer can build and test against QA. TBD Prod IDs are a grooming discussion item,
not a reason to hold the story.

**DeFi field names vs. IDs:** If a story references a DeFi custom field by display name
only, this is a blocker — developers need the field ID or XML element name, not the label
shown in the DeFi UI.

**Backfill language:** A single sentence ("historical records will not be backfilled" or
"a separate backfill ticket will be created") is sufficient. Do not require a full
backfill spec in the story itself.

**Column renames:** Any rename of an existing column is automatically advisory for
downstream impact, even if the story describes it as minor. Flag it every time.

**Scope creep check:** If what looks like a field addition actually involves creating a
new staging table or a new SP from scratch, reclassify as `ready-for-backlog-etl-new-object`
and re-evaluate.

---

## Sizing Guidance

Use these anchors after all parent and type-specific blockers are resolved.
1 SP ≈ 8 hours of development work (hours do not need to fall on the same day).
Stories that feel larger than 3 SP should be split before going to grooming.

| Story Points | Typical Scope |
|---|---|
| **1 SP** | Single field added to existing SP + view; no backfill; no downstream consumer updates |
| **2 SP** | 2–4 fields added; or 1 field with backfill; or 1 field with one downstream view update |
| **3 SP** | 5+ fields; or complex transformation / formula logic; or backfill + downstream consumer updates |

**If the story feels larger than 3 SP, split it.** Common split lines for this type:
- Field additions in one ticket, backfill in a separate ticket
- SP logic changes in one ticket, downstream view/report updates in a separate ticket
- Group fields by object (one ticket per target table) if multiple tables are affected
