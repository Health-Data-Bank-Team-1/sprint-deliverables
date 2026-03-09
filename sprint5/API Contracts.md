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

**3) Patients (Legacy)**

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

**Audit rules**

All reporting endpoints automatically log:

- reporting_trends_view – logged when /api/reporting/trends is called
  - includes metric, bucket, from, to
- reporting_summary_view – logged when /api/me/summary is called
  - includes from, to, keys requested

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
