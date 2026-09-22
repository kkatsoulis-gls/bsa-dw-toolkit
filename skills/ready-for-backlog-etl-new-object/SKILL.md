---
name: ready-for-backlog-etl-new-object
description: >
  Evaluate whether an ETL new-object story is ready for the backlog. Use when
  the story creates net-new tables, views, schemas, stored procedures, or
  entirely new Prefect pipelines from scratch. Always trigger on "create table",
  "new schema", "new view", "new pipeline", "new Prefect flow", "new stored
  procedure", "DDL", "net new", "greenfield", or "new data source". Reads the
  ready-for-backlog parent and adds new-object-specific checks.
---

# Ready for Backlog — ETL: New Object

## Setup

Before evaluating the story, read and internalize the `ready-for-backlog` parent skill.
That skill defines the universal readiness framework — story structure rules, the INVEST
model, the base blocking issues, advisory items, stakeholder question patterns, and the
output format. This child skill adds new-object-specific rules on top of it.

Apply all parent rules first. Then apply the additional rules in this document.

---

## Story Type

**Type:** ETL — Create New Object

**Signals:** create table, new schema, new view, new pipeline, new Prefect flow, new
stored procedure, DDL, new data source, net new, greenfield, from scratch, new staging
table, new FACT table, new dimension, new integration, first time.

If the story modifies an **existing** table, SP, or view rather than creating one,
stop and use `ready-for-backlog-etl-parsing` instead.

---

## Type-Specific Pre-Grooming Blockers

*These are additions to the parent's blocking issue list. Apply parent blockers first.*

**DDL / Schema Spec — No object definition present**
A new-object story must include at minimum a column list with data types — or a
reference to an attached DDL spec. "We'll define the columns during development" is
not estimable. The target schema and object name are the developer's responsibility to
determine per DW naming standards; the BSA is not expected to supply them.
💬 Ask: "Can you provide a column list with data types, or attach the DDL spec?"

**Dependencies / Migration — Prerequisite objects not identified**
If the new object depends on another object being created or modified first (e.g., a
new view depends on a new staging table), those dependencies must be stated. Stories
with hidden dependencies cannot be accurately estimated or scheduled.
💬 Ask: "Does this story depend on any other objects being created or modified first?
If so, are those dependencies in the backlog as separate tickets?"

**New Pattern — Architectural decisions left open**
If this is the first object of its type (e.g., first real-time staging table, first use
of a new schema, first connection to a new source system), the story must acknowledge
that pattern decisions will be made — and either make them in grooming or explicitly
defer them. Leaving pattern decisions implicit is a blocker.
💬 Ask: "Is this story following an existing pattern or establishing a new one?
If new, which architectural decisions (naming, clustering, connection method) need to
be resolved before development begins?"

---

## Type-Specific Grooming Discussion Items

*Raise these in grooming. They are not blockers but should be resolved before or during
sprint planning.*

- **Clustering / indexing** — Should the new table be clustered? On what key? Clustering
  decisions affect query performance and cost and are easier to implement at creation
  than retroactively.
- **Retention / partitioning** — Will this table grow unbounded? Is there a partition
  or retention strategy, or will that be a follow-up story?
- **Historical seed load** — Does the new object need to be populated with historical
  data on go-live, or does it start empty and populate going forward?
- **Access / permissions** — Which Snowflake roles or downstream consumers will need
  access to the new schema or table? Should a GRANT statement be part of this story?

---

## Type-Specific Advisory Items

*Non-blocking. Note these to the BSA for awareness.*

- **No rollback plan mentioned** — Especially for new pipelines: if the first production
  run fails or produces bad data, what is the recovery strategy? Advisory to address
  before go-live.
- **Prefect alerting not specified** — If the story includes a new Prefect flow, confirm
  whether alerting follows the team's standard pattern or requires custom configuration.
- **No GitHub PR or migration script reference** — DDL for new objects should go through
  version control. If the story doesn't mention this, flag as advisory.
- **"Similar to [existing object]" without a clear reference** — If the story says the
  new object should follow an existing pattern but doesn't name or link the source
  object, flag so the developer knows what to look at.

---

## Type-Specific Judgment Guidelines

**Partial DDL:** A complete column list is a blocker. Exact data types on every field
can be finalized in grooming — but the developer needs enough to begin. "We'll need
columns for date, amount, and ID" is not sufficient.

**Object naming:** The developer is responsible for choosing the schema and object name
per DW naming standards. Do not flag a missing or placeholder object name as a blocker
— the BSA is not expected to supply it.

**"New pattern" stories:** If the developer will be making architectural decisions during
development (e.g., first use of a new schema, first real-time connection), flag this
explicitly in grooming. These stories tend to run long and are good candidates for
splitting off the pattern/design work as a spike.

**"Similar to X" stories:** Acceptable if the reference object is clearly named and
findable. "Build something like what we did for [Vendor Y]" is workable if the developer
can locate it. "Do it like we usually do" is not.

**Dependency bundling:** If a story both creates a new staging table AND loads a new
FACT table from it, that is two distinct objects and likely two stories. Flag for
splitting.

---

## Sizing Guidance

Use these anchors after all parent and type-specific blockers are resolved.
1 SP ≈ 8 hours of development work (hours do not need to fall on the same day).
Stories that feel larger than 3 SP should be split before going to grooming.

| Story Points | Typical Scope |
|---|---|
| **1 SP** | New view on top of existing objects; no new tables or flows |
| **2 SP** | New staging table + basic Prefect flow, following an existing pattern |
| **3 SP** | New staging table + SP + view + Prefect flow, following an existing pattern |

**If the story feels larger than 3 SP, split it.** Common split lines for this type:
- New object DDL + initial load in one ticket, Prefect orchestration in a separate ticket
- Staging layer in one ticket, FACT/dimension layer in a separate ticket
- New-pattern design / spike in one ticket, implementation in a follow-up ticket
- Historical seed load as a separate ticket from the go-forward pipeline
