# BSA Data Warehouse Toolkit

A Claude Code plugin for the GLS Auto Data Warehouse BSA team. Provides two core workflows:

1. **Write structured Jira-ready requirements** from rough notes, meeting summaries, or plain-language requests.
2. **Evaluate stories for backlog grooming readiness** using a tiered review framework — Pre-Grooming Blockers, Grooming Discussion items, and Advisory notes.

---

## Skills Included

### `requirements-writer`

Drafts a polished, Jira-ready requirements document from whatever you give it — rough notes, a problem description, or a meeting summary.

**Output format:**
- User Story (As a / I need / so that)
- Background (current state, context, desired future state)
- Requirements (numbered, system-specific, actionable)
- Acceptance Testing Criteria (Given-When-Then)
- Notes for Review (assumptions and open questions, if any)

**Trigger phrases:** "write a story", "draft requirements", "turn this into a Jira ticket", "help me write this up", or just describe the feature in plain language.

---

### `ready-for-backlog-etl-parsing`

Evaluates stories that add fields, update stored procedures, modify views, or change calculations in **existing** Snowflake objects.

**Trigger phrases:** "add field", "new column", "update SP", "modify stored procedure", "DeFi XML", "field mapping", "backfill" (on an existing table).

---

### `ready-for-backlog-etl-new-object`

Evaluates stories that create **net-new** Snowflake tables, views, schemas, stored procedures, or Prefect pipelines from scratch.

**Trigger phrases:** "create table", "new schema", "new view", "new pipeline", "new Prefect flow", "DDL", "net new", "greenfield".

---

### `ready-for-backlog-reporting`

Evaluates stories that create or modify Power BI reports, SSRS reports, dashboards, or Python-based reports.

**Trigger phrases:** "Power BI", "SSRS", "report", "dashboard", "visual", "measure", "slicer", "KPI", "email distribution", "scheduled report".

---

### `ready-for-backlog-file-transfer`

Evaluates stories involving inbound or outbound file movement — SFTP, S3, vendor file ingestion, CSV imports, or scheduled exports.

**Trigger phrases:** "SFTP", "file transfer", "inbound file", "outbound file", "import", "export", "CSV", "flat file", "S3", "Azure Blob".

---

### `ready-for-backlog` (parent — not triggered directly)

Shared foundation loaded automatically by the four child skills above. Defines the universal INVEST-based readiness framework, the three-tier classification system, output format, sizing guidance, and grooming presentation template.

---

## How to Install

1. Open **Claude Code** (desktop app)
2. Click the **Directory** icon in the left sidebar
3. Click **Plugins** → **Code** tab
4. Search for `bsa-dw-toolkit`
5. Click **Install**

Skills are active in every conversation immediately after install. Updates pushed to this repo are picked up automatically — no reinstall needed.

---

## Team

GLS Auto Data Warehouse
