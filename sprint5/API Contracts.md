**API Contract Overview**

**Base path**

/api/reports

**Auth + roles**

- User dashboard endpoints: auth (logged-in user)
- Cohort comparison: auth + cohort access allowed
- Researcher/admin exports: role:researcher & admin

**1) List available report types (optional)**

GET /api/reports

**Response**

{  
"reports": \[  
{  
"id": "dashboard_summary",  
"name": "Dashboard Summary",  
"scope": "personal",  
"supports_export": false  
},  
{  
"id": "dashboard_trends",  
"name": "Dashboard Trends",  
"scope": "personal",  
"supports_export": true  
},  
{  
"id": "cohort_comparison",  
"name": "Cohort Comparison",  
"scope": "personal_vs_cohort",  
"supports_export": true,  
"privacy": { "k_threshold_default": 5 }  
}  
\]  
}

**2) Dashboard Summary (cards)**

GET /api/reports/dashboard/summary

**Query params (optional)**

- date_from=YYYY-MM-DD
- date_to=YYYY-MM-DD
- form_template_id=&lt;id&gt;

**Response**

{  
"generated_at": "2026-02-28T20:10:00Z",  
"filters": {  
"date_from": null,  
"date_to": null,  
"form_template_id": null  
},  
"summary": {  
"total_submissions": 14,  
"last_submission_at": "2026-02-25T18:22:00Z",  
"active_days": 9  
}  
}

**3) Dashboard Trends (graph series)**

GET /api/reports/dashboard/trends

**Query params**

- metric=submission_count (start with this; add more later)
- group_by=day|week|month (default week)
- date_from, date_to
- form_template_id=&lt;id&gt; (optional)

**Response**

{  
"generated_at": "2026-02-28T20:10:00Z",  
"metric": "submission_count",  
"group_by": "week",  
"rows": \[  
{ "period": "2026-W06", "value": 3 },  
{ "period": "2026-W07", "value": 5 }  
\]  
}

**4) Cohort Comparison (user vs cohort aggregates)**

POST /api/reports/cohort/compare

**Note:** Cohort comparison requires a cohort membership mapping (e.g., cohort_id on accounts/users or a cohort_members table). If not configured, the API will return either 501 Not Implemented or { cohort: { status: "not_configured" } }.

**Request body**

{  
"metric": "submission_count",  
"group_by": "month",  
"date_from": "2026-01-01",  
"date_to": "2026-02-28",  
"form_template_id": null,  
"k_threshold": 5  
}

**Response**

{  
"generated_at": "2026-02-28T20:10:00Z",  
"metric": "submission_count",  
"group_by": "month",  
"k_threshold": 10,  
"cohort": {  
"status": "ok",  
"member_count": 32,  
"suppressed": false  
},  
"rows": \[  
{ "period": "2026-01", "user_value": 2, "cohort_avg": 1.6, "cohort_min": 0, "cohort_max": 6 },  
{ "period": "2026-02", "user_value": 4, "cohort_avg": 2.1, "cohort_min": 0, "cohort_max": 7 }  
\]  
}

**If cohort is too small**

{  
"generated_at": "2026-02-28T20:10:00Z",  
"metric": "submission_count",  
"group_by": "month",  
"k_threshold": 10,  
"cohort": {  
"status": "suppressed",  
"member_count": 3,  
"suppressed": true  
},  
"rows": \[\]  
}

**5) CSV Export Endpoints**

- **Content-Type: text/csv**
- **Content-Disposition: attachment; filename="&lt;report-name&gt;\_&lt;date-range&gt;.csv"**

**A) Export dashboard trends**

GET /api/reports/dashboard/trends/export.csv?metric=submission_count&group_by=week&date_from=...

Returns streamed CSV:

**CSV columns**

- period
- value

**B) Export cohort comparison**

POST /api/reports/cohort/compare/export.csv

Same body as compare, returns CSV:

**CSV columns**

- period
- user_value
- cohort_avg
- cohort_min
- cohort_max

**Contract Notes**

**Standard filter rules**

- date_from, date_to are optional; default date_from = today - 90 days, date_to = today
- group_by defaults to week
- form_template_id optional

**Allowed Parameter Values**

- metric: submission_count (current supported value)
- group_by: day, week, month (default: week)
- k_threshold: integer ≥ 2 (default: 10; cohort endpoints only)

**Security rules**

- All endpoints require authentication.
- Cohort compare uses aggregation only and suppresses small cohorts (k_threshold).
- Responses never include raw health entry values or encrypted payload data.

**Audit rules (recommended)**

Log:

- report_run_requested / report_run_completed / report_run_failed
- report_export_requested / report_export_completed / report_export_failed

**Error Responses (Global)**

All endpoints may return:

- 401 – Unauthenticated
- 403 – Forbidden
- 422 – Validation error
- 500 – Internal server error
- 501 – Cohort comparison not configured