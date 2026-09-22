---
name: ready-for-backlog-reporting
description: >
  Evaluate whether a reporting story is ready for the backlog. Use when the
  story creates or modifies Power BI reports, SSRS reports, dashboards, or
  Python-based reports distributed via email or SFTP. Always trigger when user
  mentions "Power BI", "SSRS", "report", "dashboard", "visual", "measure",
  "slicer", "filter", "chart", "KPI", "metric", "email distribution",
  "scheduled report", or "Python report".
---

# Ready for Backlog — Reporting

## Setup

Before evaluating the story, read and internalize the `ready-for-backlog` parent skill.
That skill defines the universal readiness framework — story structure rules, the INVEST
model, the base blocking issues, advisory items, stakeholder question patterns, and the
output format. This child skill adds reporting-specific rules on top of it.

Apply all parent rules first. Then apply the additional rules in this document.

---

## Story Type

**Type:** Analytics / Reporting

**Signals:** Power BI, SSRS, report, dashboard, visual, measure, calculated column,
slicer, filter, chart, page, KPI, metric, Python report, email distribution, SFTP
delivery, scheduled report, tabular report, paginated report.

If the story primarily involves creating or modifying Snowflake objects (tables, SPs,
views) without a reporting deliverable, stop and use `ready-for-backlog-etl-new-object`
or `ready-for-backlog-etl-parsing` instead.

---

## Type-Specific Pre-Grooming Blockers

*These are additions to the parent's blocking issue list. Apply parent blockers first.*

**Data Source — Source not specified**
The story must identify which Snowflake schema, view, or table (or other data source)
the report reads from. "The data warehouse" or "our existing data" is not sufficient
for a developer to build a dataset or connect a report.
💬 Ask: "Which specific Snowflake view or table should this report pull from?
If that object doesn't exist yet, is there a separate story to create it?"

**Visual Description — No output description present**
A report story without any description of the expected output is not estimable.
The story needs at minimum one of: a mockup or wireframe, a list of visuals with
their type and measure, or a written description of each page and its content.
💬 Ask: "Can you share a mockup, rough sketch, or written description of what each
page or visual should show? The developer needs to know what they're building."

**AC — Only vague accuracy language**
Acceptance criteria for reports must be specific and testable. "Data is accurate" or
"numbers match" is not a criterion. AC must specify what field or measure to validate,
against what reference value or source, and under what conditions.
💬 Ask: "What specific field or measure should we validate in testing? What is the
expected value or source of truth we should compare against?"

**Refresh / Schedule — Not defined**
If the report is expected to reflect data as of a certain point in time (e.g., prior
business day, hourly), the refresh cadence must be stated. An undefined refresh
schedule leads to reports that appear stale with no visibility into why.
💬 Ask: "How often should this report refresh, and as of what data cutoff?
For example: 'daily at 6 AM, reflecting the prior business day's data.'"

---

## Type-Specific Grooming Discussion Items

*Raise these in grooming. They are not blockers but should be resolved before or during
sprint planning.*

- **Deployment steps** — Where does the report live after development? Which Power BI
  workspace, SSRS folder, or distribution list? Who publishes it to production?
- **Row-level security (RLS)** — Does the report need to filter data by user, role,
  dealer, or region? RLS is a significant scope driver and should be confirmed
  before estimation.
- **Mobile layout** — Is a mobile-optimized layout required, or is desktop-only
  acceptable? Mobile layouts add meaningful development time.
- **Page and visual count** — More pages and more visuals mean more development time.
  Walk through the expected page/visual count in grooming to validate the estimate.
- **Distribution method** — For Python or SSRS reports: who receives it, in what file
  format, at what path or address, and with what file naming convention?

---

## Type-Specific Advisory Items

*Non-blocking. Note these to the BSA for awareness.*

- **Story specifies Snowflake objects (views, tables, schemas)** — Reporting stories
  should describe the business output, not the data layer. If DDL or view creation
  requirements appear, flag whether this is intentional (mixed story) or
  over-specification. If intentional, also apply the relevant ETL child skill rules.
- **No edge case in AC** — What should the report show if there is no data for the
  selected filter period? An empty visual, a "No data available" message, or something
  else? Commonly missed; worth adding.
- **Distribution details not confirmed** — If the report will be emailed or SFTP'd,
  the recipient list, file format, and naming convention should be confirmed before
  development begins.
- **"Matches existing report" without a clear reference** — If the story says the new
  report should match or replace an existing one, the existing report must be named
  and accessible to the developer.

---

## Type-Specific Judgment Guidelines

**Over-specification of Snowflake objects:** If a reporting story also includes DDL,
view creation, or SP changes, determine whether this is intentional (a mixed story)
or whether the BSA has over-specified the data layer. Mixed stories should apply the
relevant ETL child skill rules as well and are good candidates for splitting.

**"Matches existing report" stories:** Acceptable if the existing report is named and
the developer can access it. "Build something like the old dealer summary report" is
not sufficient unless the report is accessible and identified clearly.

**Python report stories:** Confirm whether the Python output is a file (CSV, Excel,
PDF) or a rendered visual (Streamlit, HTML). File-based reports need destination path,
naming convention, and format in AC. Rendered visuals need a hosting/deployment plan.

**Measure-only stories (no new page or visual):** New DAX measures or calculated
columns added to an existing Power BI report, with no new pages, are typically 1 SP.
Be cautious about scope additions surfaced during grooming.

**Paginated vs. interactive reports:** SSRS and Power BI paginated reports have
different development patterns than interactive Power BI dashboards. Confirm which
type is expected — they are not interchangeable in scope or tooling.

---

## Sizing Guidance

Use these anchors after all parent and type-specific blockers are resolved.
1 SP ≈ 8 hours of development work (hours do not need to fall on the same day).
Stories that feel larger than 3 SP should be split before going to grooming.

| Story Points | Typical Scope |
|---|---|
| **1 SP** | New measure or calculated column in an existing Power BI report; no new page or visual |
| **2 SP** | New report page on an existing dataset with 2–4 visuals |
| **3 SP** | New report on an existing dataset with multiple pages; or a new dataset + single-page report |

**If the story feels larger than 3 SP, split it.** Common split lines for this type:
- Dataset / data model work in one ticket, report visuals in a separate ticket
- One report page per ticket if the page count is high
- Distribution / deployment setup as a separate ticket from report development
- RLS implementation as a separate ticket if it adds significant scope
