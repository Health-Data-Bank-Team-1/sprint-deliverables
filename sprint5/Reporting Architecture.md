Sprint 5 **Reporting Architecture**

**Purpose**

Sprint 5 reporting enables users to view and understand their personal health data through:

- dashboard summaries (latest values, counts, basic aggregates),
- time-series trends (daily/weekly/monthly),
- data export (CSV).

Reporting is **read-only** and must enforce strict access control (users only see their own data) and maintain confidentiality (no decrypted PHI is persisted or logged).

**Data Model Used for Reporting**

**Primary Source of Truth: health_entries**

health_entries stores the user’s timestamped health data payload:

- account_id - ownership and access control boundary
- timestamp - time-series axis for charts and trends
- encrypted_values (JSONB) - encrypted PHI payload (field/value pairs)
- submission_id - optional linkage to submission metadata

For submission-status–based reporting, queries may start from form_submissions and join health_entries via submission_id

**Supporting Metadata: form_submissions**

form_submissions stores submission context:

- status - SUBMITTED, FLAGGED
- submitted_at
- form_template_id - origin form template

**Architecture rule:** Reporting queries start from health_entries. Join form_submissions only when filtering by submission attributes (e.g., status/template).

Personal queries filter by account_id

Cohort queries filter by a cohort membership rule

**Reporting Layers**

**1) Storage Layer (Database)**

- Stores encrypted payloads in health_entries.encrypted_values
- Stores submission metadata in form_submissions
- No reporting-specific tables are required for Sprint 5 (on-the-fly aggregation)

**2) Domain/Service Layer (Reporting Logic)**

Reporting logic is implemented in services to keep controllers thin and consistent.

**Recommended services:**

1.  **HealthEntryQueryService**
    - Fetches records from health_entries scoped to the authenticated user
    - Applies filters: date range, template, submission status (via join)

1.  **HealthEntryDecryptService**
    - Decrypts encrypted_values safely
    - Supports a **field allowlist** (only permitted metrics can be requested)
    - Never logs decrypted values

1.  **ReportingAggregationService**
    - Builds dashboard summaries (latest values, counts, avg/min/max where numeric)
    - Builds time-series series by interval (day | week | month)

1.  **ReportingExportService**
    - Produces export rows (flattened fields with timestamps)
    - Streams output for large ranges (avoids memory spikes)

**3) API Layer (Controllers)**

Controllers:

- validate request params (from, to, interval, fields)
- enforce authentication + user scoping
- call service layer
- return stable response DTOs (JSON) or stream CSV

**Suggested controllers**

- ReportController (summary + trends)
- ReportExportController (CSV)

**4) Presentation Layer (User Dashboard)**

Frontend consumes:

- **summary endpoint** → cards/tiles (latest BP, entries this week, etc.)
- **series endpoint** → charts (trends over time)
- **export endpoint** → CSV download button

**Cohort Comparison Reporting (User vs Aggregate)**

In addition to personal dashboard summaries, the reporting architecture supports cohort-level comparison.

This enables:

- A user to compare their personal metrics against aggregated values of a similar demographic or assigned cohort.
- The system to calculate cohort-level aggregates without exposing individual-level data.

**Data Scope Rules**

Personal data:

health_entries.account_id = current_account_uuid

Cohort aggregate data:

health_entries.account_id IN (users belonging to same cohort/demographic group)

Cohort queries must:

- Return aggregated values only (count, avg, min, max)
- Never return individual rows
- Apply a k-anonymity threshold (e.g., k ≥ 10)
- Suppress results if cohort size is below threshold

**Privacy Enforcement**

- Cohort comparisons must never expose:
    - other users’ decrypted values
    - individual-level entries
    - free-text or raw encrypted payloads
- Only aggregate numeric results are returned.

**Data Flow**

**A) Dashboard Summary Flow**

1.  User requests summary with date range
2.  API fetches health_entries for that user + time window
3.  Decrypt requested/allowed fields
4.  Aggregate values (latest, count, avg/min/max)
5.  Return JSON summary payload

**B) Trend/Chart Flow**

1.  User requests a metric series and interval
2.  API fetches matching entries for user + time window
3.  Decrypt the requested metric only (allowlist)
4.  Bucket into intervals (day/week/month)
5.  Return JSON time-series payload for chart rendering

**C) CSV Export Flow**

1.  User requests export with date range and selected fields
2.  API fetches entries (scoped to user)
3.  For dashboard trends export, data is sourced from form_submissions and does not require decryption.
4.  Stream CSV rows back to client

**Security and Privacy Requirements**

**Ownership enforcement**

All reporting operations must enforce:

- health_entries.account_id = = current_account_uuid (or the authenticated account UUID)

Users must not be allowed to pass or override another account_id.

**Encrypted payload handling**

- encrypted_values is decrypted only in the service layer
- Decryption is limited to explicitly requested fields (allowlist)
- Decrypted values must never be:
    - written back to the database,
    - logged,
    - placed into audit metadata.

**Submission status filtering (recommended rule)**

Reporting should exclude data tied to FLAGGED submissions by default (or only include APPROVED if approval workflow is required for user reporting).

Document the current sprint rule clearly, e.g.:

- Default: include SUBMITTED and APPROVED, exclude FLAGGED
    - or: include APPROVED only

**Performance Approach (Sprint 5)**

Sprint 5 uses **on-the-fly** aggregation:

- no background jobs required
- no precomputed reporting tables
- CSV export uses streaming to remain stable under larger ranges

Future optimization (optional):

- add daily/weekly aggregates via scheduled jobs if volume grows.

**Implementation Notes (Suggested Code Structure)**

- app/Http/Controllers/Api/ReportController.php
- app/Http/Controllers/Api/ReportExportController.php
- app/Services/Reporting/HealthEntryQueryService.php
- app/Services/Reporting/HealthEntryDecryptService.php
- app/Services/Reporting/ReportingAggregationService.php
- app/Services/Reporting/ReportingExportService.php

**Architecture Diagram (Text)**

**User Dashboard**

1.  **Reporting API (Controllers)**
2.  **Reporting Services (Query + Decrypt + Aggregate/Export)**
3.  **Database (health_entries + form_submissions)**
4.  **Response (JSON summaries/series OR streamed CSV)**