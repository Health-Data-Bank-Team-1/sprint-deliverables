**Audit Logging - Developer Guide (Updated for AuditLogger Service)**

**Purpose**

All audit logging in the Health Data Bank system must go through the centralized **AuditLogger** service.

This ensures that:

- Audit entries are consistent across the system
- No PHI or sensitive data is logged (fail-closed guardrails)
- All logs follow the same event-based structure
- Logs are compatible with the audits table schema
- IP address, user agent, and timestamp are automatically captured
- Actor identification uses account_id when available

**Critical Rule:** Audit events must **NEVER** be written directly to the audits table from controllers or models. Always use `AuditLogger::log()`.

**Audit Logging Pattern**

Use the centralized service:

```php
AuditLogger::log(
    string $event,
    array $tags = [],
    ?Model $auditable = null,
    array $oldValues = [],
    array $newValues = []
);
```

**Parameter Reference**

**$event (required)**

Event name in snake_case format. Should be descriptive and specific.

Examples:
- `login_success`
- `reporting_trends_view`
- `form_template_approved`
- `access_denied`

**$tags (optional)**

Array of tags for categorization and filtering. Use tags to indicate:
- **Category**: auth, authz, form, reporting, workflow
- **Outcome**: outcome:success, outcome:failure, outcome:blocked
- **Resource**: resource:trends, resource:summary, resource:template

Examples:
```php
['auth', 'outcome:success']
['reporting', 'resource:trends']
['form', 'workflow', 'outcome:approved']
['authz', 'outcome:blocked']
```

**$auditable (optional)**

A Model instance being acted upon. If provided, the audit entry will be linked to that model.

Examples:
```php
$event->user                    // User model
$form->template                 // FormTemplate model
$submission                     // FormSubmission model
null                            // No specific model
```

**$oldValues (optional)**

Safe "before" values in associative array format. Should contain only:
- IDs (user_id, template_id, submission_id)
- Metadata (counts, status, timestamps)
- Safe enums (approval_status, submission_status)

Never include: health data, emails, names, passwords, encrypted_values

Examples:
```php
['approval_status' => 'draft']
['version' => 1, 'entry_count' => 5]
[]  // Empty if no changes tracked
```

**$newValues (optional)**

Safe "after" values in associative array format. Same constraints as oldValues.

Examples:
```php
['approval_status' => 'pending']
['version' => 2, 'entry_count' => 7]
[]  // Empty if no changes tracked
```

**Automatic Context Captured**

The AuditLogger automatically captures and logs:

- **user_type**: Class name of authenticated actor
- **user_id**: UUID from authenticated user's account_id
- **ip_address**: IP address of request
- **user_agent**: User agent string (truncated to 1023 chars)
- **url**: Full request URL with query parameters
- **created_at**: Current timestamp

These should **NOT** be manually added to oldValues or newValues.

**Authentication Events**

**Successful Login**

```php
AuditLogger::log(
    'login_success',
    ['auth', 'outcome:success'],
    $event->user,
    [],
    []
);
```

Location: `app/Listeners/LogLoginSuccess.php`

**Failed Login**

```php
AuditLogger::log(
    'login_failure',
    ['auth', 'outcome:failure'],
    $auditable,
    [],
    ['reason' => 'invalid_credentials']
);
```

Location: `app/Listeners/LogLoginFailure.php`

Note: $auditable may be null if email not found.

**User Registration**

```php
AuditLogger::log(
    'register_success',
    ['auth', 'outcome:success'],
    $event->user,
    [],
    []
);
```

Location: `app/Listeners/LogRegistered.php`

**Logout**

```php
AuditLogger::log(
    'logout',
    ['auth', 'outcome:success'],
    $event->user,
    [],
    []
);
```

Location: `app/Listeners/LogLogout.php`

**Authorization Events**

**Access Denied**

```php
AuditLogger::log(
    'access_denied',
    ['authz', 'outcome:blocked'],
    $actor,
    [],
    [
        'method' => $request->method(),
        'route' => optional($request->route())->getName() ?? 'unknown',
        'path' => $request->path(),
        'reason' => class_basename($e),
        'ip_address' => $request->ip(),
        'user_agent' => $request->userAgent(),
    ]
);
```

Location: `app/Exceptions/Handler.php`

Triggers on: AuthorizationException or AccessDeniedHttpException

**Reporting Access Events**

**Trends View** (API Access)

```php
AuditLogger::log(
    'reporting_trends_view',
    ['reporting', 'resource:trends'],
    null,
    [],
    [
        'metric' => $metric,
        'bucket' => $bucket,
        'from' => $validated['from'],
        'to' => $validated['to'],
    ]
);
```

Location: `app/Http/Controllers/Reporting/TrendController.php`

Triggered: When user calls GET /api/reporting/trends

Metadata Logged: Request parameters (metric, bucket, date range), not aggregated values

**Summary View** (API Access)

```php
AuditLogger::log(
    'reporting_summary_view',
    ['reporting', 'resource:summary'],
    null,
    [],
    [
        'from' => $request->from,
        'to' => $request->to,
        'keys' => $keys,  // list of metric keys only
    ]
);
```
Location: `app/Http/Controllers/Api/MeSummaryController.php`

Triggered: When user calls GET /api/me/summary

Metadata Logged: Date range and requested metric keys, not summary values

**Researcher Cohort Generation** (API Access)

```php
AuditLogger::log(
    'researcher_cohort_generated',
    ['reporting', 'researcher', 'outcome:success'],
    null,
    [],
    [
        'filters' => $filters,
        'cohort_size' => $results->count(),
    ]
);
```
Location: app/Http/Controllers/Researcher/ResearcherCohortController.php

Triggered: When researcher calls POST /api/researcher/cohorts

Metadata Logged: Applied filters and cohort size only

**Researcher Aggregated Report View** (API Access)
```php
AuditLogger::log(
    'researcher_aggregated_report_viewed',
    ['reporting', 'researcher', 'outcome:success'],
    null,
    [],
    [
        'filters' => $filters,
        'cohort_size' => $report['cohort_size'],
    ]
);
```
Location: app/Http/Controllers/Researcher/ResearcherReportController.php

Triggered: When researcher calls POST /api/researcher/reports/aggregated

Metadata Logged: Applied filters and cohort size only

**Researcher Aggregated Report Export** (API Access)
```php
AuditLogger::log(
    'researcher_aggregated_report_exported',
    ['reporting', 'researcher', 'outcome:success', 'format:csv'],
    null,
    [],
    [
        'filters' => $filters,
        'cohort_size' => $report['cohort_size'],
        'format' => 'csv',
    ]
);
```
Location: app/Http/Controllers/Researcher/ResearcherReportController.php

Triggered: When researcher calls POST /api/researcher/reports/aggregated/export.csv

Metadata Logged: Applied filters, cohort size, and export format only

**Form Submission Events** (Future Implementation)

**Successful Submission**

```php
AuditLogger::log(
    'form_submission_success',
    ['form', 'outcome:success'],
    $submission,
    [],
    [
        'form_template_id' => $submission->form_template_id,
        'entry_count' => count($submission->entries),
    ]
);
```

Metadata: Template ID, number of entries submitted (not values)

**Form Template Events** (Future Implementation)

**Template Submitted for Approval**

```php
AuditLogger::log(
    'form_template_submitted',
    ['form', 'workflow'],
    $template,
    ['approval_status' => 'draft'],
    ['approval_status' => 'pending']
);
```

**Template Approved**

```php
AuditLogger::log(
    'form_template_approved',
    ['form', 'workflow', 'outcome:approved'],
    $template,
    ['approval_status' => 'pending'],
    ['approval_status' => 'approved', 'approved_by' => $admin->account_id]
);
```

**Template Rejected**

```php
AuditLogger::log(
    'form_template_rejected',
    ['form', 'workflow', 'outcome:rejected'],
    $template,
    ['approval_status' => 'pending'],
    ['approval_status' => 'rejected', 'reason' => 'unclear_field_labels']
);
```

Note: Rejection reason must be safe metadata, not user feedback text

**Sensitive Data Protection**

**The Guardrail System**

Before any audit entry is created, `AuditLogger.guardAgainstSensitiveData()` scans oldValues and newValues for blocked keys.

**If a blocked key is detected:**
- Throws `InvalidArgumentException` immediately
- Prevents the audit entry from being written
- Logs error with the key name (but not the value)
- Fail-closed: errors on the side of safety

**Blocked Key Patterns:**

password, pass, pwd, token, secret
email, name, address
dob, date_of_birth
health, symptom, diagnosis, notes
form_response, encrypted_values, responses

**Blocked Value Patterns:**

Text values > 500 characters

**Safe Metadata to Include:**

IDs (user_id, account_id, form_id, template_id)
Counts (entry_count, submission_count)
Status/Enum values (approval_status: 'pending', submission_status: 'submitted')
Event codes (reason: 'invalid_credentials', outcome: 'success')
Timestamps (submitted_at, created_at)

**When to Log**

Log an audit entry when:

**Authentication succeeds or fails**
- User logs in
- User registers
- User logs out
- Login attempt fails

**Authorization check fails**
- User attempts access to admin page without role
- User attempts to access another user's data

**Sensitive operations complete**
- Form submission succeeds
- Form template submitted for approval
- Admin approves/rejects template
- User accesses reporting endpoints
- Researcher generates a cohort
- Researcher generates an aggregated report
- Researcher exports an aggregated report to CSV

**Reporting access occurs**
- User calls /api/reporting/trends
- User calls /api/me/summary
- User exports data (future)
- Researcher calls /api/researcher/cohorts
- Researcher calls /api/researcher/reports/aggregated
- Researcher calls /api/researcher/reports/aggregated/export.csv


**Do NOT log:**
- Every page load
- Every database query
- Every validation error (unless authorization-related)
- Development/debugging operations

**Error Handling**

If `AuditLogger::log()` throws `InvalidArgumentException`:

```php
try {
    AuditLogger::log(...);
} catch (\InvalidArgumentException $e) {
    // Log the error to application logs
    \Log::error("Audit logging failed: " . $e->getMessage());
    
    // In development, re-throw; in production, send to Sentry/monitoring
    if (config('app.debug')) {
        throw $e;
    }
    \Sentry\captureException($e);
}
```

The exception indicates a programming error (attempted PHI logging), not a runtime error.

**Testing Audit Logging**

**Verify Event Was Logged**

```php
public function test_login_creates_audit_log()
{
    $user = User::factory()->create();
    
    $this->post('/login', [
        'email' => $user->email,
        'password' => 'password',
    ]);
    
    $this->assertDatabaseHas('audits', [
        'event' => 'login_success',
        'user_id' => $user->account_id,
    ]);
}
```

**Verify Tags Were Set**

```php
public function test_reporting_access_logged_with_tags()
{
    $account = Account::factory()->create();
    $user = User::factory()->create(['account_id' => $account->id]);
    
    $this->actingAs($user, 'sanctum')->getJson(
        '/api/reporting/trends?metric=hr&from=2026-02-01&to=2026-02-01&bucket=day'
    );
    
    $audit = \OwenIt\Auditing\Models\Audit::where('event', 'reporting_trends_view')->first();
    
    $this->assertStringContainsString('reporting', $audit->tags);
    $this->assertStringContainsString('resource:trends', $audit->tags);
}
```

**Verify Sensitive Data Not Logged**

```php
public function test_health_data_blocked_from_audit_log()
{
    try {
        AuditLogger::log(
            'test_event',
            [],
            null,
            [],
            ['health_score' => 85]  // Blocked key
        );
        $this->fail('Expected InvalidArgumentException');
    } catch (\InvalidArgumentException $e) {
        $this->assertStringContainsString('health_score', $e->getMessage());
    }
}
```

**Query Audit Logs**

```php
// Find all events for a user
$events = \OwenIt\Auditing\Models\Audit::where('user_id', $accountId)->get();

// Find all reporting access events
$reportingAccess = \OwenIt\Auditing\Models\Audit::where('event', 'like', 'reporting%')->get();

// Find failed logins
$failedLogins = \OwenIt\Auditing\Models\Audit::where('event', 'login_failure')->get();

// Find events with specific tag
$authEvents = \OwenIt\Auditing\Models\Audit::where('tags', 'like', '%auth%')->get();

// Find events in date range
$recentEvents = \OwenIt\Auditing\Models\Audit::whereBetween('created_at', [$from, $to])->get();
```

**Summary**

The system uses a **centralized, fail-closed, privacy-safe audit logging model** via AuditLogger.

Each audit entry records:

- **Who**: User identity (account_id)
- **What**: Event code (event name)
- **When**: Timestamp and date range (created_at)
- **Context**: IP address, user agent, request URL
- **Safe Metadata**: IDs, counts, status changes (never PHI)

This approach ensures compliance with HIPAA/GDPR/CCPA while maintaining an audit trail for security and accountability.

**Never log PHI. Use AuditLogger::log(). Always.**
