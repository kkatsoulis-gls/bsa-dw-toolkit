# Domain Patterns Reference

Common requirement patterns for the Data Warehouse team. Use these as structural guides when drafting requirements for familiar request types.

---

## Payment / Account Logic (Operations)

**Typical User Story shape:**
> As an Operations Team member, I need [accounts/records] to be flagged/updated based on [condition], so that [downstream process or report] reflects accurate status.

**Common requirement patterns:**
- A new field or flag must be added to `[table]` to capture `[status/condition]`
- The Prefect flow `[flow name]` must be updated to evaluate `[condition]` and set the flag accordingly
- The logic must account for: payments received vs. expected, date boundaries (current month, rolling 30 days, etc.), null/zero payment cases
- A Power BI report or measure must reflect the updated flag in near-real-time or on next scheduled refresh

**Common acceptance criteria patterns:**
- GIVEN account has met payment threshold → THEN NOT flagged
- GIVEN account has partial payment → THEN flagged
- GIVEN account has no payment activity → THEN flagged
- GIVEN data has not yet refreshed → THEN previous flag state is preserved (no false resets)

---

## ETL / Prefect Flow Changes

**Typical User Story shape:**
> As a Data Engineer, I need [flow/pipeline] to [new behavior], so that [downstream consumers] receive [accurate/timely/complete] data.

**Common requirement patterns:**
- A new Prefect flow or task must be created to handle `[process]`
- An existing flow must be modified to add/remove/update `[step or logic]`
- The flow must be scheduled to run `[frequency]` and retry on failure `[n]` times
- Flow execution must log `[specific metadata]` for monitoring purposes
- The flow must depend on / trigger after `[upstream flow or condition]`

**Common acceptance criteria patterns:**
- GIVEN source data is available → WHEN flow runs → THEN target table is updated within [X minutes]
- GIVEN source data is unavailable or malformed → WHEN flow runs → THEN flow fails gracefully with a logged error and does not write partial data
- GIVEN the flow has already run today → WHEN triggered again → THEN [idempotent behavior: skips / overwrites / appends as designed]

---

## Snowflake Object Changes (Data Layer)

**Typical User Story shape:**
> As a [Data Engineer / Analyst], I need [new table/view/column/schema], so that [consumers] can access [data] for [purpose].

**Common requirement patterns:**
- A new table/view must be created in schema `[SCHEMA]` named `[OBJECT_NAME]`
- An existing table must be altered to add column(s): `[column name, data type, nullable/not null]`
- A view must be updated to include/exclude `[logic or source]`
- Row-level security or access roles must be granted to `[role(s)]`
- Historical data must be backfilled for `[date range]` upon deployment

**Common acceptance criteria patterns:**
- GIVEN the object is deployed → THEN it is accessible to role `[ROLE_NAME]` without errors
- GIVEN a query against the new object → THEN results match expected row counts from source
- GIVEN backfill is complete → THEN date range `[start]` to `[end]` contains no gaps

---

## Power BI Reporting Changes (Analytics)

**Typical User Story shape:**
> As a [Report Consumer / Business Analyst], I need [new visual/measure/page/filter] in [Report Name], so that I can [analyze/monitor/track] [metric or dimension].

**Important:** Requirements for analytics/reporting stories should focus on the report itself — what it needs to show, how it should behave, and what filtering/slicing is needed. Do **not** include requirements for Snowflake views, tables, or models. Leave the data layer architecture to the engineers.

**Common requirement patterns:**
- A new page titled `[Page Name]` must be added to the `[Report Name]` report in Power BI
- The following measures must be created in the report dataset: `[Measure Name]` — [definition/logic]
- A slicer for `[dimension]` must be added and must filter all visuals on the page
- A visual (`[chart type]`) must display `[metric]` trended by `[time dimension]`
- A KPI card must display the current value of `[measure]` at the top of the page
- An existing visual on page `[page name]` must be updated to include/exclude `[field or filter]`
- The report must refresh on `[schedule]` or support on-demand refresh

**Common acceptance criteria patterns:**
- GIVEN the report page is opened → THEN all visuals load without errors and display data for the default date range
- GIVEN a slicer for `[dimension]` is applied → THEN all visuals on the page filter correctly to that selection
- GIVEN two slicers are applied simultaneously → THEN visuals reflect the intersection of both filters with no conflicts
- GIVEN a month/period with no data → THEN the visual displays zero rather than a gap or null
- GIVEN the report is refreshed → THEN measure totals match the expected values from the source data

---

## Data Transfer / File Ingestion

**Typical User Story shape:**
> As a Data Engineer, I need [external data source] to be ingested into [Snowflake target], so that [downstream process or report] has access to [data].

**Common requirement patterns:**
- A Prefect flow must be created to pull data from `[source: SFTP / API / S3 / flat file]` on a `[schedule]`
- Raw files must be staged in `[Snowflake stage or S3 path]` before loading
- A target table must be created or updated in Snowflake to receive the data
- Data validation rules must be applied before writing: `[e.g., non-null keys, expected row count range, date freshness check]`
- File archival or deletion must occur after successful load

**Common acceptance criteria patterns:**
- GIVEN a new file is available at the source → WHEN the flow runs → THEN the file is loaded into the target table within `[X minutes/hours]`
- GIVEN the file is malformed or empty → WHEN the flow runs → THEN the load is rejected, an error is logged, and no data is written
- GIVEN the transfer has already run for today's file → WHEN triggered again → THEN duplicate records are not created
