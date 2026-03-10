**API Contract Overview**

**Base path**

/api

**Auth + roles**

- All reporting endpoints: auth (logged-in user via auth:sanctum)
- User can only access their own data (filtered by account_id)
- Admin-only endpoints: role:admin

**1) Health Metrics Trends**

GET /api/reporting/trends

**Query params (required)**

- metric=<string> (e.g., 'hr', 'bp', 'weight') - required
- from=YYYY-MM-DD - required
- to=YYYY-MM-DD - required

**Query params (optional)**

- bucket=day|week|month (default: day)

**Response**

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

**Notes**

- count reflects only numeric values (int, float, numeric-string)
- latest is the most recent raw value (may be non-numeric)
- Empty points array returned if no data in range

**2) Personal Summary**

GET /api/me/summary

**Query params (required)**

- from=YYYY-MM-DD
- to=YYYY-MM-DD

**Query params (optional)**

- keys=<comma-separated-metrics> (e.g., 'hr,bp,weight')

**Response**
``json
{  
"from": "2026-02-01",  
"to": "2026-02-28",  
"averages": {  
"hr": 75.5,  
"bp": 125.0,  
"weight": 170.2  
},  
"counts": {  
"hr": 14,  
"bp": 14,  
"weight": 10  
}  
}

**Notes**

- averages include only numeric values
- counts reflect numeric datapoints only
- if keys parameter is provided, only those metrics are returned
- user must have account_id linked to their user record

**3) Researcher Cohort**

POST /api/researcher/cohorts

**Auth + roles**

- Requires `auth:sanctum`
- Requires researcher role
- Intended for researcher-only cohort generation

**Request body**

```json
{
  "metric_key": "exercise_frequency",
  "status": "ACTIVE",
  "timeframe": "week",
  "start_date": "2026-03-01",
  "end_date": "2026-05-31"
}

Supported filters
- metric_key - string
- status - string (e.g., ACTIVE, EXPIRED)
- timeframe - string (e.g., day, week, month)
- start_date - ISO date (YYYY-MM-DD)
- end_date - ISO date (YYYY-MM-DD)

Response:
```json
{
  "message": "Cohort generated successfully",
  "cohort_size": 2,
  "filters_applied": {
    "status": "ACTIVE"
  },
  "data": [
    {
      "id": "uuid",
      "account_id": "uuid",
      "metric_key": "alcohol_consumption",
      "comparison_operator": "<=",
      "target_value": 2,
      "timeframe": "month",
      "start_date": "2026-03-09",
      "end_date": "2026-04-24",
      "status": "ACTIVE"
    }
  ]
}
Notes:
- Returns filtered cohort records for researcher analysis
- Response excludes direct identifiers such as name and email
- cohort_size must match the number of returned rows

**4) Researcher Aggregated Report**
POST /api/researcher/reports/aggregated

**Auth + roles**
- Requires auth:sanctum
- Requires researcher role

**Request body**
Same filters as /api/researcher/cohorts

Response:
```json
{
  "message": "Aggregated report generated successfully",
  "filters_applied": {
    "status": "ACTIVE"
  },
  "report": {
    "cohort_size": 1,
    "active_goals": 1,
    "expired_goals": 0,
    "average_target_value": 2,
    "metric_breakdown": {
      "alcohol_consumption": 1
    }
  }
}
**Notes**
- Returns summarized statistics instead of raw cohort rows
- average_target_value is numeric and rounded
- metric_breakdown maps metric_key to count

**5) Researcher Aggregated Report CSV Export**
POST /api/researcher/reports/aggregated/export.csv

**Auth + roles**
- Requires auth:sanctum
- Requires researcher role

**Request body**
Same filters as /api/researcher/reports/aggregated

Response:
Content-Type: text/csv

Example CSV:
metric,value
cohort_size,2
active_goals,1
expired_goals,1
average_target_value,3
metric_breakdown_exercise_frequency,1
metric_breakdown_alcohol_consumption,1

Notes
- Export reflects the same filtered aggregated dataset as the JSON aggregated report endpoint
- CSV contains summary rows, not raw health records

**6) Patients (Legacy)**

GET /api/patients

POST /api/patients

GET /api/patients/{id}

**Response**

Standard REST resource endpoints for patient management.

**Standard filter rules**

- from, to dates are required for trend and summary endpoints
- metric is required for trends endpoint
- bucket defaults to day
- keys optional for summary endpoint (if omitted, all metrics returned)

**Allowed Parameter Values**

- metric: any alphanumeric string with underscores, hyphens, dots (e.g., 'hr', 'blood_pressure', 'bmi')
- bucket: day, week, month (default: day)
- from/to: ISO date format (YYYY-MM-DD)

**Security rules**

- All endpoints require Sanctum authentication
- Users can only access their own aggregated data (filtered by account_id)
- Responses never include raw encrypted_values or unencrypted health details
- Non-authenticated requests return 401
- Forbidden requests return 403
- Researcher reporting endpoints require researcher role in addition to Sanctum authentication
- Researcher cohort endpoint returns only safe reporting fields
- Researcher aggregated export returns summary CSV only

**Audit rules**

All reporting endpoints automatically log:

- reporting_trends_view – logged when /api/reporting/trends is called
  - includes metric, bucket, from, to
- reporting_summary_view – logged when /api/me/summary is called
  - includes from, to, keys requested
- reporting_trends_view
- reporting_summary_view
- researcher_cohort_generated
- researcher_aggregated_report_viewed
- researcher_aggregated_report_exported

Audit log entries:

- Include event type, tags, URL, IP address, user agent
- Exclude sensitive data (passwords, emails, health values, encrypted payloads)

**Error Responses (Global)**

All endpoints may return:

- 401 – Unauthenticated (missing or invalid Sanctum token)
- 403 – Forbidden (insufficient permissions)
- 422 – Validation error (missing required params, invalid dates, invalid bucket value)
- 500 – Internal server error

**Validation Rules**

For /api/reporting/trends:

- metric: required, string, max 64 chars, alphanumeric + underscore/hyphen/dot
- from: required, valid date
- to: required, valid date, must be >= from
- bucket: optional, must be day|week|month

For /api/me/summary:

- from: required, valid date
- to: required, valid date, must be after from
- keys: optional, comma-separated metric names
