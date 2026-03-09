Sprint 6 **Reporting Architecture**

**Purpose**

Reporting enables users to view and understand their personal health data through:

- dashboard summaries (latest values, counts, basic aggregates),
- time-series trends (daily/weekly/monthly),
- health metric aggregation and analysis.

Reporting is **read-only** and must enforce strict access control (users only see their own data) and maintain confidentiality (no decrypted PHI is persisted or logged).

**Data Model Used for Reporting**

**Primary Source of Truth: health_entries**

health_entries stores the user's timestamped health data payload:

- account_id - ownership and access control boundary
- timestamp - time-series axis for charts and trends
- encrypted_values (JSONB) - encrypted PHI payload (field/value pairs, e.g., {"hr": 72, "bp": 120})
- submission_id - optional linkage to submission metadata

For submission-status–based reporting, queries may start from form_submissions and join health_entries via submission_id

**Supporting Metadata: form_submissions**

form_submissions stores submission context:

- status - SUBMITTED, FLAGGED, APPROVED
- submitted_at
- form_template_id - origin form template

**Architecture rule:** Reporting queries start from health_entries. Join form_submissions only when filtering by submission attributes (e.g., status/template).

Personal queries filter by account_id

Cohort queries filter by a cohort membership rule (future implementation)

**Reporting Layers**

**1) Storage Layer (Database)**

- Stores encrypted payloads in health_entries.encrypted_values
- Stores submission metadata in form_submissions
- No reporting-specific tables required for Sprint 5 (on-the-fly aggregation)
- Optional future: pre-aggregated tables for Reports and AggregatedData

**2) Domain/Service Layer (Reporting Logic)**

Reporting logic is implemented in services to keep controllers thin and consistent.

**Actual services (implemented):**

1. **ReportingAggregationService**
   - Aggregates all metrics for a single account across a time range
   - Returns for each metric key: count (numeric only), min, max, avg (numeric only), latest (any type), latest_at
   - Filters health_entries by account_id and timestamp range
   - Handles mixed numeric/non-numeric values intelligently (numeric aggregates skip non-numeric; latest captures any)
   - Used by dashboard summaries and trend calculations

2. **PersonalSummaryService**
   - Wraps ReportingAggregationService for dashboard cards
   - Extracts averages and counts across requested metric keys
   - Supports optional key filtering (comma-separated metric names)
   - Returns: from, to, averages (map of metric → float), counts (map of metric → int)

3. **TrendCalculationService**
   - Builds time-series aggregation bucketed by period (day/week/month)
   - Generates one point per bucket with: bucket_start, count, min, max, avg, latest, latest_at
   - Handles non-numeric values in latest field (e.g., notes, text)
   - Returns empty points array if no data in range
   - Validates metric name, bucket type, date range

4. **AuditLogger** (Supporting Service)
   - Logs all reporting access with event types: reporting_trends_view, reporting_summary_view
   - Blocks sensitive keys from logs (passwords, emails, health details, encrypted_values, responses, form_response)
   - Includes metadata: metric, bucket, from, to, keys (but not data values)
   - Enforces fail-closed: throws InvalidArgumentException if sensitive key detected

**Note on Decryption:**

encrypted_values are stored in the database with Laravel's envelope encryption. When health_entries records are loaded from the database, encrypted_values are automatically decrypted by the ORM (transparent decryption). Reporting services work with the already-decrypted JSONB data in memory. No separate HealthEntryDecryptService is required.

**3) API Layer (Controllers)**

Controllers:

- validate request params (metric, from, to, bucket, keys)
- enforce authentication via auth:sanctum middleware
- enforce user scoping by account_id
- call service layer
- return stable response DTOs (JSON)
- log access via AuditLogger

**Actual controllers (implemented):**

- **TrendController** (`app/Http/Controllers/Reporting/TrendController.php`)
  - Endpoint: GET /api/reporting/trends
  - Query params: metric (required), from (required), to (required), bucket (optional, default: day)
  - Validates inputs; metric must match regex ^[A-Za-z0-9_\-\.]+$
  - Calls TrendCalculationService
  - Logs reporting_trends_view audit event with metric, bucket, from, to
  - Returns: {metric, bucket, from, to, points: [{bucket_start, count, min, max, avg, latest, latest_at}, ...]}

- **MeSummaryController** (`app/Http/Controllers/Api/MeSummaryController.php`)
  - Endpoint: GET /api/me/summary
  - Query params: from (required), to (required), keys (optional, comma-separated)
  - Validates date range; to must be after from
  - Calls PersonalSummaryService
  - Logs reporting_summary_view audit event with from, to, keys
  - Returns: {from, to, averages: {metric: float}, counts: {metric: int}}

**4) Presentation Layer (User Dashboard)**

Frontend consumes:

- **summary endpoint** → cards/tiles (latest BP, entries this week, average metrics, etc.)
- **trends endpoint** → charts (line graphs of metrics over time bucketed by day/week/month)
- Future: **export endpoint** → CSV download button (not yet implemented in Sprint 5)

**Cohort Comparison Reporting (Future)**

The architecture is designed to support cohort-level comparison in future sprints.

This would enable:

- A user to compare their personal metrics against aggregated values of a similar demographic or assigned cohort
- The system to calculate cohort-level aggregates without exposing individual-level data

**Data Scope Rules (Future Implementation)**

Personal data (implemented):

health_entries.account_id = current_account_uuid

Cohort aggregate data (future):

health_entries.account_id IN (users belonging to same cohort/demographic group)

Cohort queries must (future):

- Return aggregated values only (count, avg, min, max)
- Never return individual rows
- Apply a k-anonymity threshold (e.g., k ≥ 5 or k ≥ 10)
- Suppress results if cohort size is below threshold

**Privacy Enforcement**

Cohort comparisons (when implemented) must never expose:

- other users' decrypted values
- individual-level entries
- free-text or raw encrypted payloads

Only aggregate numeric results are returned.

**Data Flow**

**A) Personal Summary Flow (Implemented)**

1. User requests summary with date range (from, to)
2. API authenticates via Sanctum and validates params
3. MeSummaryController retrieves user.account_id
4. PersonalSummaryService queries health_entries filtered by account_id + timestamp range
5. ReportingAggregationService aggregates each metric key: count (numeric), avg (numeric), min/max (numeric), latest (any)
6. Return JSON summary payload: {from, to, averages, counts}
7. AuditLogger records: reporting_summary_view + metadata

**B) Trend/Chart Flow (Implemented)**

1. User requests a metric series and interval (metric, from, to, bucket)
2. API authenticates and validates params
3. TrendController retrieves user.account_id
4. TrendCalculationService queries health_entries filtered by account_id + timestamp range
5. For each metric occurrence, extract the field value
6. Bucket into intervals (day/week/month) based on timestamp
7. Within each bucket, aggregate: count (numeric), min/max/avg (numeric), latest (any type)
8. Return JSON time-series payload: {metric, bucket, from, to, points: []}
9. AuditLogger records: reporting_trends_view + metadata

**C) CSV Export Flow (Future)**

User requests export with date range and selected fields

API fetches entries (scoped to user)

Stream CSV rows back to client

Export does not log decrypted values (only metadata logged)

**Metric Type Handling**

The system intelligently handles both numeric and non-numeric metric values:

**Numeric-only aggregates (count, min, max, avg):**

- Only count and aggregate values that are int, float, or numeric-string
- Skip non-numeric values silently
- If all values for a metric are non-numeric, these fields are null

**Latest field (any type):**

- Always returns the most recent raw value for the metric key
- May be non-numeric (e.g., text note, boolean flag, string status)
- Includes latest_at timestamp
- Useful for capturing the latest status/note without aggregation

**Example:**

health_entries record: { account_id: "user-123", timestamp: "2026-02-01T10:00:00Z", encrypted_values: { "hr": 72, "bp": "120", "notes": "felt good" } }

Aggregation across multiple entries: { "hr": { count: 5, min: 68.0, max: 88.0, avg: 75.0, latest: 72, latest_at: "2026-02-01T18:00:00Z" }, "bp": { count: 5, min: 118.0, max: 132.0, avg: 125.0, latest: "120", latest_at: "2026-02-01T18:00:00Z" }, "notes": { count: 0, // non-numeric, no count min: null, max: null, avg: null, latest: "felt good", latest_at: "2026-02-01T18:00:00Z" } }

**Security and Privacy Requirements**

**Ownership enforcement**

All reporting operations enforce:

- health_entries.account_id = current_user.account_id
- User must be authenticated via Sanctum token
- User's account_id is fetched from request.user().account_id
- Users cannot pass or override account_id parameter

**Encrypted payload handling**

- encrypted_values is transparent via Laravel ORM (auto-decrypted on load)
- Decryption happens automatically; no manual decryption needed
- Decrypted data is held in memory only during request processing
- Service layer aggregates on decrypted values; only aggregates are returned (never raw decrypted values)
- Decrypted values must never be:
  - written back to the database
  - logged (AuditLogger blocks 'encrypted_values', 'health', 'diagnosis', 'symptom', etc.)
  - placed into audit metadata

**Submission status filtering**

Sprint 5 default: include all submission statuses (SUBMITTED, APPROVED, FLAGGED)

Future enhancement: filter by status if approval workflow is required

**Audit logging**

All reporting access is logged:

- Event: reporting_trends_view or reporting_summary_view
- Metadata logged: metric, bucket, from, to, keys (request params only)
- Data values NEVER logged
- Sensitive keys blocked by AuditLogger guardrail

**Performance Approach (Sprint 5)**

Sprint 5 uses on-the-fly aggregation:

- no background jobs required
- no precomputed reporting tables (optional future optimization)
- queries aggregate directly from health_entries table
- suitable for user-scale data volumes (thousands of entries per user)

**Future optimization (optional):**

- add scheduled jobs to pre-aggregate daily/weekly summaries if volume grows
- materialize Report + AggregatedData tables for faster large-cohort queries

**Response Data Transfer Objects**

**Trends Response DTO:**

```json
{
  "metric": "hr",
  "bucket": "day",
  "from": "2026-02-01",
  "to": "2026-02-02",
  "points": [
    {
      "bucket_start": "2026-02-01T00:00:00Z",
      "count": 2,
      "min": 70.0,
      "max": 90.0,
      "avg": 80.0,
      "latest": 90,
      "latest_at": "2026-02-01T18:00:00Z"
    },
    {
      "bucket_start": "2026-02-02T00:00:00Z",
      "count": 1,
      "min": 80.0,
      "max": 80.0,
      "avg": 80.0,
      "latest": 80,
      "latest_at": "2026-02-02T09:00:00Z"
    }
  ]
}
```

**Architecture Diagram**

-  User (Dashboard) ->
    
- Reporting API (Controllers) ->
     - TrendController (/api/reporting/trends)
     - MeSummaryController (/api/me/summary)
    
- Reporting Services ->
    - TrendCalculationService
    - ReportingAggregationService
    - PersonalSummaryService
    - AuditLogger

- Database Layer ->
    - health_entries (with encrypted_values JSONB)
    - form_submissions (metadata)

- Response (JSON + Audit Log Entry) ->
    - {metric, bucket, from, to, points: []}
    - {from, to, averages, counts}
    - audit record: {event, tags, metadata}
