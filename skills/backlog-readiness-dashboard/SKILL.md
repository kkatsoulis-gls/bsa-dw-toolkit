---
name: "backlog-readiness-dashboard"
description: "Evaluate and visualize backlog-readiness across a whole sprint (not a single named story) for the current user's reported Story/Fast Track/Spike tickets in the DATA project. Use for any sprint-level or generic readiness question: \"is my next sprint ready\", \"is sprint 124 ready for backlog\", \"are my stories ready\", \"dashboard of my backlog readiness\", \"show me a readiness dashboard\", \"visualize sprint readiness\" — as well as plain \"are my stories ready for backlog\" type asks that don't name a specific ticket. Defaults to the soonest upcoming sprint if none is named, without asking. Only considers pre-dev (Work Queue or earlier) stories; lets the user click a not-ready row to get the full detailed breakdown from cached data. Builds on bsa-dw-toolkit:ready-for-backlog and its child skills for the per-story verdicts (which still trigger directly for a single named ticket), then renders a widget dashboard summarizing the whole sprint."
---
 
---
name: "backlog-readiness-dashboard"
description: "Evaluate and visualize backlog-readiness across a whole sprint (not a single named story) for the current user's reported Story/Fast Track/Spike tickets in the DATA project. Use for any sprint-level or generic readiness question: 'is my next sprint ready', 'is sprint 124 ready for backlog', 'are my stories ready', 'dashboard of my backlog readiness', or any readiness ask that doesn't name a specific ticket. Defaults to the soonest upcoming sprint if none is named, without asking."
---
 
# Backlog Readiness Dashboard
 
This skill is a presentation layer on top of the `bsa-dw-toolkit:ready-for-backlog`
parent skill and its children (`ready-for-backlog-etl-new-object`,
`ready-for-backlog-etl-parsing`, `ready-for-backlog-file-transfer`,
`ready-for-backlog-reporting`). It gathers the right set of stories for a
sprint, grades each one, and renders the results as a visual dashboard
(metric cards + status table) instead of a long text report per story — with
a way to drill into any not-ready story afterward.
 
## When this skill applies vs. a single-story child skill
 
This skill owns any **sprint-level or generic** readiness request — meaning
the request is about multiple stories, a whole sprint, or readiness in the
abstract, not a single ticket the user names. Trigger phrasing includes
things like:
- "Is my next sprint ready [for backlog]?"
- "Is sprint 124 ready?"
- "Are my stories ready for backlog?"
- "Can you check backlog readiness?" (with no ticket named)
- "Dashboard / table / visualize my backlog readiness"
If the user instead names one specific ticket (e.g. "is DATA-3401 ready for
grooming?", or describes one story's content directly), that's a single-story
evaluation — use the matching `ready-for-backlog-*` child skill directly and
give the normal narrative report; don't build a dashboard for one ticket.
 
When a request is ambiguous about scope, default to sprint-level (this
skill) — it's the safer default since it still surfaces a single ticket
clearly if the sprint only has one qualifying story.
 
Jira access: use the Atlassian MCP tools (search names containing
`62728053-ef3b-49a1-8c46-3f39e59cc53e`, e.g. `searchJiraIssuesUsingJql`,
`getJiraIssue`). The GLS Jira cloudId is `9e0095bb-a754-4742-ad0d-e7eb69adf4eb`
(try this directly first; fall back to `getAccessibleAtlassianResources` only
if it fails). The reporter is always the current user — resolve via
`atlassianUserInfo` if the accountId isn't already known in-session.
 
**Performance**: do everything in as few Jira calls as possible. A single
`searchJiraIssuesUsingJql` call can return `summary`, `issuetype`, `status`,
`sprint`, and `description` for every matching issue at once — do not follow
it up with a separate `getJiraIssue` call per issue just to fetch the
description. Only call `getJiraIssue` on a specific issue when drilling down
later (Step 5) if its description wasn't already pulled in Step 2.
 
---
 
## Step 1: Determine the target sprint
 
GLS sprints are named with a leading sequence number, e.g. "123: Boo-sted
Capacity". "Upcoming sprint" / "next sprint" always means the next sprint
after whatever is currently active, by that leading number. "My current
sprint" means the active one, not the upcoming one — read the user's wording
carefully (next/upcoming vs. current/this sprint).
 
- **If the user names a sprint** (by number or full name) in their request,
  use that sprint directly — this always takes priority over auto-detection.
- **If no sprint is named**, auto-detect the upcoming sprint automatically —
  do not ask the user to pick:
  1. Run a JQL search scoped to the user's own reported issues in future
     sprints within the DATA project, e.g.:
     `project = DATA AND reporter = currentUser() AND issuetype in ("Story", "Fast Track", "Spike") AND sprint in futureSprints()`
     with fields `["summary","issuetype","status","sprint"]`.
  2. Collect the distinct sprints present in the results, along with each
     sprint's start date (from the sprint field) or leading number.
  3. **Always proceed automatically with the sprint that has the earliest
     start date (or, if start dates are missing/equal, the lowest leading
     number).** This is true even when multiple future sprints are returned —
     multiple future sprints being planned is the normal case, not an
     ambiguous one, since "upcoming"/"next" unambiguously means "soonest." Do
     not use AskUserQuestion for this — just pick it and proceed, and name the
     sprint you picked in your one-sentence summary at the end so the user
     can correct you if they meant a different one.
  4. If the request says "current sprint" rather than "next/upcoming", use
     `sprint in openSprints()` instead of `futureSprints()` in the query above.
  5. If zero matching sprints are found, say so and ask what to use instead
     (this is the one genuine ambiguity worth pausing for).
---
 
## Step 2: Gather and filter issues
 
Single query, requesting everything needed up front:
`project = DATA AND sprint = "<resolved sprint name>" AND reporter = currentUser() AND issuetype in ("Story", "Fast Track", "Spike")`
with `fields: ["summary","issuetype","status","description"]`.
 
Then apply a status-stage filter in-memory (no extra Jira calls): only keep
issues whose **status category is "To Do"** (i.e. the issue is at Work Queue
or an earlier pre-dev stage — e.g. Identified, Work Queue, Refinement, Ready
for Backlog — whatever the team's exact "To Do"-category stages are named at
the time). This single status-category check is sufficient to:
 
- **Exclude Disqualified and Closed** issues (these fall under the "Done"
  status category) — drop them entirely from the dashboard.
- **Exclude anything already in dev or past it** (In Dev, Peer review,
  Validation, Staging, UAT, Waiting for approval, or any other status under
  the "In Progress" status category) — drop these too. The dashboard is only
  for stories that haven't started development yet.
**Issue type scope** (from the parent skill): only **Story**, **Fast Track**,
and **Spike** — already enforced by the JQL above, which also naturally
excludes Maintenance and other out-of-scope types.
 
If zero issues remain after filtering, don't build an empty dashboard — just
say plainly that there are no qualifying pre-dev stories in that sprint (and
why, e.g. "all 4 of your stories in sprint 124 are already past Work Queue").
 
---
 
## Step 3: Grade each in-scope issue
 
Grade using the description already fetched in Step 2 (not comments — per the
parent skill's rule, the description is the source of truth, since it may
have been edited after any comments were posted). Do not re-fetch anything
per-issue at this stage.
 
- **Spikes**: do not grade against the Pre-Grooming Blocker framework. Mark
  status `na` and summarize in one short phrase what it's investigating.
- **Story / Fast Track**: determine which child skill fits the description
  content (ETL new-object, ETL parsing, file-transfer, reporting), invoke it
  once per distinct category needed to load its checklist, and apply that
  checklist to produce a `ready` or `not-ready` verdict. For `not-ready`,
  capture one short, specific phrase on what's missing (this becomes the
  dashboard's Notes column — keep it to one sentence). **Keep each issue's
  key, type, description, and full per-item findings in memory for the rest
  of this turn** — Step 5's drill-down should reuse this, not refetch.
---
 
## Step 4: Render the dashboard
 
Call `mcp__visualize__read_me` with `modules: ["data_viz"]` once if not
already loaded this session, then call `mcp__visualize__show_widget` with a
title like `sprint_<number>_backlog_readiness`.
 
Layout (reuse exactly, don't redesign each time):
- A metric-card row at the top: one card per status present among
  **Ready**, **Not ready**, and **N/A (spikes)** — card label + count.
  Omit a card entirely if its count is 0 rather than showing a 0.
- Below it, a table with columns: Key, Title, Category, Status (colored
  badge), Notes. Key issue links use `var(--text-accent)`. Status badges:
  ready → `var(--bg-success)` / `var(--text-success)`; not-ready →
  `var(--bg-danger)` / `var(--text-danger)`; na → `var(--surface-1)` /
  `var(--text-muted)`.
- **Drill-down**: every row with status `not-ready` gets a small inline
  action (e.g. a text button or `<i class="ti ti-chevron-right">` affordance)
  that calls `sendPrompt('Give me the full ready-for-backlog breakdown for
  <KEY>')` when clicked. Ready and N/A rows don't need this — only not-ready
  stories benefit from a deeper look.
- Include a visually-hidden `<h2 class="sr-only">` summarizing the sprint
  name and story count for screen readers.
- Do not restate the table in response text afterward — the widget already
  shows it. In the reply, directly answer whatever the user actually asked
  (e.g. if they asked "is my next sprint ready?", lead with a plain yes/no —
  "Not yet — 2 of 6 stories still need work" — then the one-sentence summary:
  counts, which sprint was used if auto-detected, and anything needing
  attention).
---
 
## Step 5: Drill-down into a not-ready story
 
When the user clicks a drill-down action (or later asks directly, e.g. "tell
me more about DATA-3260") **in the same turn/session where the dashboard was
just built**, produce the full per-story output using the matching child
skill's complete **Output Template** (Pre-Grooming Blockers, Grooming
Discussion Items, Advisory Items, Sizing, Questions, Grooming Presentation)
for that one issue — the normal `ready-for-backlog` narrative report, not the
dashboard format. **Reuse the description and findings already gathered in
Step 3** rather than calling Jira again — the data was just fetched moments
ago as part of building the dashboard, so there's no need to re-query it.
 
Only re-fetch from Jira if the drill-down request comes in a fresh
conversation/session that doesn't have the Step 3 data in context (i.e. the
dashboard wasn't built earlier in this same conversation).
 
---
 
## Notes
 
- This skill never modifies Jira — it's read-only, same as the parent skill.
- If the user asks for both a narrative report AND a dashboard, produce the
  narrative first (per the parent skill's output template) and the dashboard
  as a follow-up visual, not interleaved.
- Re-running this skill later (e.g. "now show me 124") re-runs Steps 1–4 from
  scratch; it does not cache or diff against a prior dashboard run.
 
