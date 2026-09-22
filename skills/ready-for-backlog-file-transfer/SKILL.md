---
name: ready-for-backlog-file-transfer
description: >
  Evaluate whether a file transfer, import, or export story is ready for the
  backlog. Use when the story involves inbound or outbound file movement —
  SFTP drops, S3 transfers, vendor file ingestion, CSV/Excel imports, or
  scheduled data exports. Always trigger when user mentions "SFTP", "file
  transfer", "inbound file", "outbound file", "import", "export", "CSV",
  "flat file", "vendor file", "send file", "receive file", "file drop",
  "S3", "Azure Blob", or "FTP".
---

# Ready for Backlog — File Transfer / Import / Export

## Setup

Before evaluating the story, read and internalize the `ready-for-backlog` parent skill.
That skill defines the universal readiness framework — story structure rules, the INVEST
model, the base blocking issues, advisory items, stakeholder question patterns, and the
output format. This child skill adds file-transfer-specific rules on top of it.

Apply all parent rules first. Then apply the additional rules in this document.

---

## Story Type

**Type:** File Transfer / Import / Export

**Signals:** SFTP, FTP, file transfer, inbound file, outbound file, import, export,
CSV, Excel, pipe-delimited, fixed-width, flat file, vendor file, send file, receive
file, file drop, scheduled transfer, S3, Azure Blob, file exchange, file-based
integration.

If the story also loads file data into Snowflake objects (staging tables, FACT tables),
apply `ready-for-backlog-etl-new-object` or `ready-for-backlog-etl-parsing` rules in
addition to this one — this is a mixed story. Size the two components separately.

---

## Type-Specific Pre-Grooming Blockers

*These are additions to the parent's blocking issue list. Apply parent blockers first.*

**Source and Destination — Not fully specified**
Both the source location and the destination must be stated. "We get a file from the
vendor" is not sufficient. The story must name the protocol (SFTP, S3, local share),
the path or bucket, and the receiving system (Snowflake table, SFTP drop, email, etc.).
💬 Ask: "What is the full source location — protocol, server or bucket, and path?
What is the full destination — protocol, path, or Snowflake table name?"

**File Format and Layout — Not defined**
File type (CSV, Excel, pipe-delimited, fixed-width), encoding (UTF-8, Latin-1), header
row presence, and column names or positions must be specified or referenced in an
attached spec. Without a layout, a developer cannot write a parser.
💬 Ask: "Can you share the file layout spec or a sample file? We need the format,
delimiter, encoding, and column list to build the parser."

**Schedule / Trigger — Not defined**
When does the transfer run — on a fixed schedule, on-demand, triggered by file arrival,
or triggered by another system event? A story with no trigger definition is not
estimable.
💬 Ask: "When does this transfer run — fixed schedule, on-demand, or triggered by
file arrival? If scheduled, what is the cadence and time window?"

**Error Handling — No failure behavior specified**
What happens if the source file is missing, arrives late, is malformed, or fails
mid-transfer? The story must state at minimum whether the failure should alert someone,
retry automatically, or log and continue.
💬 Ask: "What should happen if the file is missing or malformed on the expected
schedule? Should we alert operations, retry automatically, or log the failure and move on?"

---

## Type-Specific Grooming Discussion Items

*Raise these in grooming. They are not blockers but should be resolved before or during
sprint planning.*

- **File naming convention** — Does the source file have a predictable static name, or
  does it include a date stamp or sequence number? For exports, what naming convention
  should the outbound file follow?
- **Archive / retention** — Should processed files be archived after transfer or deleted?
  How long should files be retained at source or destination?
- **Volume and frequency** — Approximately how many records per file, and how often does
  the file arrive or need to be sent? This affects performance and retry strategy design.
- **Credential / connection management** — Are SFTP credentials or cloud storage access
  already established, or does a new connection need to be set up? New credentials
  often require vendor coordination and lead time.

---

## Type-Specific Advisory Items

*Non-blocking. Note these to the BSA for awareness.*

- **AC does not cover the failure case** — The happy path (file arrives, transfers
  successfully) may be covered, but AC should also include at least one failure
  criterion: what happens when the file is missing, malformed, or late?
- **Partial file handling not addressed** — What happens if the file arrives but with
  far fewer records than expected (e.g., 10 rows when the usual volume is 10,000)?
  Is this treated as a failure, or processed normally? Advisory to define.
- **No monitoring or alerting mentioned** — If the transfer fails silently, operations
  won't know. Confirm whether this follows the team's standard Prefect alerting pattern
  or requires custom notification setup.
- **One-time vs. recurring not stated** — If the story doesn't distinguish between a
  one-time load and a recurring scheduled transfer, note this. Recurring transfers are
  substantially more complex and should be sized accordingly.

---

## Type-Specific Judgment Guidelines

**"Vendor will provide the file" is not sufficient:** The story must describe the file
layout the vendor will deliver, not just acknowledge vendor involvement. If the layout
is not yet available, treat the missing spec as a blocker and ask for a sample file.

**Inbound transfer + Snowflake load:** If the file transfer also includes loading data
into a Snowflake staging or FACT table, this is a mixed story. Apply the relevant ETL
child skill checklist (etl-parsing or etl-new-object) in addition to this one. Size
the two components separately — if the combined estimate exceeds 3 SP, split into
separate tickets.

**SFTP credential provisioning:** If new credentials need to be set up with a vendor
or a new firewall rule needs to be opened, this often requires lead time and may be
a separate dependency ticket. Flag if it appears bundled into the story's scope.

**One-time vs. recurring:** A one-time file load is substantially simpler than a
recurring scheduled transfer. If the story is ambiguous, treat it as recurring for
sizing purposes and confirm during grooming.

**File format ambiguity:** "CSV" and "Excel" are not interchangeable from a development
standpoint. If the story uses them interchangeably or says "CSV or Excel depending on
what the vendor sends," flag as a blocker — the format must be definitive before
development begins.

---

## Sizing Guidance

Use these anchors after all parent and type-specific blockers are resolved.
1 SP ≈ 8 hours of development work (hours do not need to fall on the same day).
Stories that feel larger than 3 SP should be split before going to grooming.

| Story Points | Typical Scope |
|---|---|
| **1 SP** | One-time file load into an existing Snowflake table; known format; no scheduling |
| **2 SP** | Recurring scheduled file transfer using an existing Prefect pattern; known format and destination |
| **3 SP** | New inbound file pipeline with a new Prefect flow, new staging table, and known format |

**If the story feels larger than 3 SP, split it.** Common split lines for this type:
- File transfer / ingestion in one ticket, Snowflake load into new objects in a separate ticket
- Error handling and alerting as a separate ticket from the core transfer logic
- Credential / connection setup as a separate dependency ticket with its own timeline
- Inbound and outbound directions as separate tickets if both are in scope
