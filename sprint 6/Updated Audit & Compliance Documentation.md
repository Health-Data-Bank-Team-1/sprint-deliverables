**Updated Audit & Compliance Documentation (Sprint 5)**

**Purpose**

The Health Data Bank system handles sensitive user health information for precariously housed individuals. To support accountability, security, and compliance requirements, the system implements **centralized event-based audit logging** through the **AuditLogger** service. Audit logs provide a tamper-evident record of significant actions (authentication, authorization, reporting access, form submissions, and admin workflow actions) without storing PHI/PII in the audit metadata.

Audit logging is built on the industry-standard **OwenIT/laravel-auditing** package, ensuring compatibility with HIPAA, GDPR, and CCPA requirements.

**Audit Log Storage**

Audit events are recorded in the **audits** database table using a centralized **AuditLogger** service. This table is designed as an **append-only event stream** for security and workflow monitoring.

**Core Fields Recorded for Each Event:**

- **user_type**: Class name of authenticated actor (e.g., 'App\Models\User')
- **user_id**: UUID of authenticated actor from accounts table (nullable for guest/system events)
- **event**: String event name in snake_case (e.g., 'login_success', 'reporting_trends_view')
- **auditable_type**: Model class being acted upon (e.g., 'App\Models\FormTemplate')
- **auditable_id**: UUID of model being acted upon (nullable for non-model events)
- **old_values**: JSON of "before" values (safe metadata only, never PHI)
- **new_values**: JSON of "after" values (safe metadata only, never PHI)
- **url**: Full request URL including query parameters
- **ip_address**: IPv4 or IPv6 address of requester
- **user_agent**: User agent string from HTTP request (truncated to 1023 chars)
- **tags**: Comma-separated tags for categorization (e.g., 'auth,outcome:success')
- **created_at**: ISO 8601 timestamp of event (auto-captured)

**Audit Logger Service**

**Location:** `app/Services/AuditLogger.php`

**Purpose:** Centralized service for all audit logging to ensure:
- Consistent audit entry structure
- Prevention of PHI/PII logging via guardrails
- Automatic capture of context (IP, user agent, URL)
- Integration with OwenIT/laravel-auditing package
- Fail-closed approach (throws exception on sensitive data)

**Method Signature:**

```php
AuditLogger::log(
    string $event,
    array $tags = [],
    ?Model $auditable = null,
    array $oldValues = [],
    array $newValues = []
): void
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| **$event** | string | Yes | Event name in snake_case (e.g., 'login_success', 'reporting_trends_view') |
| **$tags** | array | No | Array of tags for categorization (e.g., ['auth', 'outcome:success']) |
| **$auditable** | Model\|null | No | Model instance being acted upon (User, FormTemplate, etc.) |
| **$oldValues** | array | No | Safe "before" values (IDs/metadata only, no PHI) |
| **$newValues** | array | No | Safe "after" values (IDs/metadata only, no PHI) |

**Automatic Context Captured:**
- IP address (from request)
- User agent (from request)
- Full request URL (from request)
- Timestamp (current time)
- Actor UUID (resolved from authenticated user's account_id)

These are automatically added and should NOT be manually included in oldValues/newValues.

**Audit Event Categories**

**Authentication Events**

| Event | Tags | Triggered By | oldValues | newValues |
|-------|------|--------------|-----------|-----------|
| login_success | ['auth', 'outcome:success'] | Successful user login | [] | [] |
| login_failure | ['auth', 'outcome:failure'] | Failed login attempt | [] | ['reason' => 'invalid_credentials'] |
| register_success | ['auth', 'outcome:success'] | User account registration | [] | [] |
| logout | ['auth', 'outcome:success'] | User logout | [] | [] |

**Authorization Events**

| Event | Tags | Triggered By | oldValues | newValues |
|-------|------|--------------|-----------|-----------|
| access_denied | ['authz', 'outcome:blocked'] | Authorization check fails | [] | ['method' => 'GET', 'route' => 'admin.forms.index', 'reason' => 'AuthorizationException'] |

**Reporting Access Events**

| Event | Tags | Triggered By | Metadata Logged |
|-------|------|--------------|-----------------|
| reporting_trends_view | ['reporting', 'resource:trends'] | GET /api/reporting/trends | metric, bucket, from, to |
| reporting_summary_view | ['reporting', 'resource:summary'] | GET /api/me/summary | from, to, keys |
| researcher_cohort_generated | ['reporting', 'researcher', 'outcome:success'] | POST /api/researcher/cohorts | filters, cohort_size |
| researcher_aggregated_report_viewed | ['reporting', 'researcher', 'outcome:success'] | POST /api/researcher/reports/aggregated | filters, cohort_size |
| researcher_aggregated_report_exported | ['reporting', 'researcher', 'outcome:success', 'format:csv'] | POST /api/researcher/reports/aggregated/export.csv | filters, cohort_size, format |

**Form Workflow Events** (Future)

| Event | Tags | Purpose |
|-------|------|---------|
| form_submission_success | ['form', 'outcome:success'] | User submits health form |
| form_template_submitted | ['form', 'workflow'] | User submits form template for approval |
| form_template_approved | ['form', 'workflow', 'outcome:approved'] | Admin approves form template |
| form_template_rejected | ['form', 'workflow', 'outcome:rejected'] | Admin rejects form template |

**Sensitive Data Protection**

**AuditLogger.guardAgainstSensitiveData() Guardrail**

Before logging, the AuditLogger checks oldValues and newValues for keys that might contain PHI/PII. If any blocked keys are detected, it throws an `InvalidArgumentException` with fail-closed behavior.

**Blocked Key Patterns:**

- password, pass, pwd, token, secret
- email, name, address
- dob, date_of_birth
- health, symptom, diagnosis, notes, form_response
- encrypted_values, responses

**Blocked Value Patterns:**

- Text values > 500 characters (prevents large text blobs)

**Safe Metadata to Log:**

IDs (user_id, account_id, form_id)
Counts (entry_count, submission_count)
Status changes (approval_status: 'pending' → 'approved')
Event codes (outcome: 'success', reason: 'invalid_credentials')
Timestamps (submitted_at, created_at)
Researcher reporting metadata (filters, cohort_size, export format)

**Never Log:**

Raw health values (hr: 72, bp: 120)
Form responses (encrypted data)
Names, emails, addresses
Passwords or tokens
Free-text diagnoses or notes
Unencrypted encrypted_values

**Audit Log Retention**

**Audit Log Retention Schedule**

- **Storage Duration**: Audit logs retained for **2 years** minimum
- **Rationale**: Audit logs are governance/security records, not health data; longer retention supports:
  - Security incident investigations
  - Compliance audits and reviews
  - Breach detection and response
  - Dispute resolution
  - Legal holds and discovery

**Archival & Purge:**

After 2-year retention period:
- Audit logs may be securely archived to cold storage (S3 Glacier, etc.)
- Or securely purged using cryptographic deletion
- Reconstruction of purged records not reasonably foreseeable

**Configurable Retention:**

Retention period adjustable per deployment via environment configuration:
```
AUDIT_RETENTION_YEARS=2
```

**Secure Disposal:**

All deletion/archival uses cryptographically secure methods per PHIPA/HIPAA standards:
- Database-level deletion with cascading removes
- Verification that no orphaned audit records remain
- No reliance on soft deletes for sensitive audit data

**Compliance Framework**

**HIPAA Compliance (U.S. Health Insurance Portability and Accountability Act)**

- **45 CFR § 164.312(b) - Audit Controls**: Audit logging mechanism implemented for all access
- **45 CFR § 164.312(a)(2)(i) - User ID**: User identity (account_id) logged for all events
- **45 CFR § 164.312(a)(2)(ii) - Emergency Access Procedure**: Audit trail tracks all access attempts
- **45 CFR § 164.400-414 - Breach Notification**: Audit logs support breach detection and investigation
- **45 CFR § 164.410 - Notification Procedures**: Audit history supports notification workflow

**GDPR Compliance (European General Data Protection Regulation)**

- **Article 5(1)(f) - Integrity and Confidentiality**: Audit logs demonstrate security measures
- **Article 32 - Security Measures**: Audit logging is part of security design
- **Article 33-34 - Breach Notification**: Audit logs enable breach detection and notification
- **Article 35 - Data Protection Impact Assessment**: Audit logging documented in DPIA

**CCPA Compliance (California Consumer Privacy Act)**

- **Section 1798.100 - Consumer Right to Know**: Audit logs support access requests
- **Section 1798.105 - Right to Delete**: Audit logs track deletion requests and completion
- **Section 1798.120 - Consumer Right to Correct**: Audit logs track corrected data
- **Section 1798.150 - Right to Opt-Out**: Audit logs document opt-out requests

**PHIPA Compliance (Personal Health Information Protection Act, Ontario)**

- **Section 9.1 - Reasonable Steps**: Audit logging demonstrates data protection steps
- **Section 11 - Individual Access**: Audit logs support access request fulfillment
- **Schedule 2 - Privacy Impact Assessment**: Audit logging documented in PIA

**Implementation Notes**

**Code Structure**

```
app/Services/AuditLogger.php                      ← Central audit logging service
app/Listeners/LogLoginSuccess.php                 ← Authentication event listeners
app/Listeners/LogLoginFailure.php
app/Listeners/LogLogout.php
app/Listeners/LogRegistered.php
app/Http/Controllers/Reporting/TrendController.php        ← Reporting audit logging
app/Http/Controllers/Api/MeSummaryController.php
app/Http/Controllers/Researcher/ResearcherCohortController.php
app/Http/Controllers/Researcher/ResearcherReportController.php
app/Exceptions/Handler.php                        ← Authorization failure logging
config/audit.php                                  ← OwenIT/laravel-auditing configuration
database/migrations/2026_02_01_031735_create_audits_table.php
tests/Feature/Reporting/ReportAccessAuditLogTest.php

```

**OwenIT/laravel-auditing Package**

- **Package**: OwenIt/laravel-auditing
- **Version**: Actively maintained and security-patched
- **Configuration**: `config/audit.php`
- **Table**: `audits` (append-only event log)
- **Driver**: Database (no external dependencies)

**Testing Audit Logging**

Audit log entries can be verified in tests:

```php
// Verify audit event was recorded
$this->actingAs($user, 'sanctum')->getJson('/api/reporting/trends?metric=hr&from=2026-02-01&to=2026-02-01&bucket=day');

$this->assertDatabaseHas('audits', [
    'event' => 'reporting_trends_view',
    'user_id' => $user->account_id,
]);
```

**Querying Audit Logs**

```php
// Find all login events
$loginEvents = \OwenIt\Auditing\Models\Audit::where('event', 'like', 'login%')->get();

// Find events by user
$userEvents = \OwenIt\Auditing\Models\Audit::where('user_id', $accountId)->get();

// Find events by date range
$recentEvents = \OwenIt\Auditing\Models\Audit::whereBetween('created_at', [$from, $to])->get();

// Find reporting access
$reportingAccess = \OwenIt\Auditing\Models\Audit::where('event', 'reporting_trends_view')->get();
```

**Future Enhancements**

1. **Automated Archival**: Scheduled jobs to archive old audit logs to cold storage
2. **Anomaly Detection**: Automated alerts for unusual access patterns (multiple access in short time, off-hours access)
3. **Audit Log Export/Compliance Reports**: Generate audit retention compliance reports
4. **Fine-Grained Retention**: Different retention periods for different event types
5. **Real-Time Alerting**: Stream audit events to security monitoring system (SIEM)
6. **Audit Trail Integrity**: Implement tamper-detection on audit logs (blockchain-style hashing)
