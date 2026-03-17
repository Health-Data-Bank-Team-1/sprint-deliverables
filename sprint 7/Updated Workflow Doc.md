## Admin Workflow Overview

The admin workflow governs how form templates are submitted, reviewed, and approved within the Health-Data Bank system. The workflow ensures proper governance while maintaining an audit trail of all administrative actions.

### Workflow States

Form templates progress through the following states:

* **Draft** (`draft`): Initial creation state, editable by form creators.
* **Pending** (`pending`): Submitted for admin review.
* **Approved** (`approved`): Approved by admin, ready for use.
* **Rejected** (`rejected`): Rejected by admin, requires revisions.



---

## Form Template Approval Workflow

### Workflow Events

The admin workflow consists of four key events, each automatically audited:

#### 1. Form Template Submitted (`form_template_submitted`)

* **Trigger**: User/provider submits form template for approval.
* **State Transition**: `draft` → `pending`
* **Audit Details**:
    * **Event Name**: `form_template_submitted`
    * **Tags**: `['form', 'workflow']`
    * **Target Type**: `form_template`
    * **Metadata Logged**:
        * `status_transition`: `draft` → `pending`
        * `template_version`: Version number of submitted template
        * `form_template_id`: UUID of the template
    * **Actor**: User/provider account UUID (auto-captured)
    * **Context**: IP address, user agent, timestamp (auto-captured)

**Code Example**:

```php
AuditLogger::log(
    'form_template_submitted',
    ['form', 'workflow'],
    $formTemplate,
    ['status' => 'draft'],
    [
        'status' => 'pending',
        'template_version' => $formTemplate->version,
    ]
);
```

#### 2. Form Template Approved (`form_template_approved`)

* **Trigger**: Admin approves pending form template.
* **State Transition**: `pending` → `approved`
* **Audit Details**:
    * **Event Name**: `form_template_approved`
    * **Tags**: `['form', 'workflow', 'outcome:approved']`
    * **Target Type**: `form_template`
    * **Metadata Logged**:
        * `status_transition`: `pending` → `approved`
        * `template_version`: Version number of approved template
        * `form_template_id`: UUID of the template
        * `approved_by`: UUID of approving admin (auto-captured as actor_id)
    * **Actor**: Admin account UUID (auto-captured)
    * **Context**: IP address, user agent, timestamp (auto-captured)

**Code Example**:

```php
AuditLogger::log(
    'form_template_approved',
    ['form', 'workflow', 'outcome:approved'],
    $formTemplate,
    ['status' => 'pending'],
    [
        'status' => 'approved',
        'template_version' => $formTemplate->version,
    ]
);
```

#### 3. Form Template Rejected (`form_template_rejected`)

* **Trigger**: Admin rejects pending form template with optional feedback.
* **State Transition**: `pending` → `rejected`
* **Audit Details**:
    * **Event Name**: `form_template_rejected`
    * **Tags**: `['form', 'workflow', 'outcome:rejected']`
    * **Target Type**: `form_template`
    * **Metadata Logged**:
        * `status_transition`: `pending` → `rejected`
        * `template_version`: Version number of rejected template
        * `form_template_id`: UUID of the template
        * `reason_provided`: Boolean indicating if rejection feedback was provided
        * `rejected_by`: UUID of rejecting admin (auto-captured as actor_id)
    * **Actor**: Admin account UUID (auto-captured)
    * **Context**: IP address, user agent, timestamp (auto-captured)
> **PHI Protection**: Rejection text/feedback is **NOT** stored in audit logs; only a boolean flag (`reason_provided`) is recorded.

**Code Example**:

```php
AuditLogger::log(
    'form_template_rejected',
    ['form', 'workflow', 'outcome:rejected'],
    $formTemplate,
    ['status' => 'pending'],
    [
        'status' => 'rejected',
        'template_version' => $formTemplate->version,
        'reason_provided' => !empty($rejectionFeedback),
    ]
);
```

#### 4. Form Template Version Created (`form_template_version_created`)

* **Trigger**: Version snapshot created during approval/versioning flow.
* **Audit Details**:
    * **Event Name**: `form_template_version_created`
    * **Tags**: `['form', 'workflow', 'versioning']`
    * **Target Type**: `form_template`
    * **Metadata Logged**:
        * `from_template_id`: UUID of original template
        * `to_template_id`: UUID of version snapshot
        * `version_number`: New version number
        * `created_by`: UUID of user creating version (auto-captured as actor_id)
    * **Actor**: User account UUID (auto-captured)
    * **Context**: IP address, user agent, timestamp (auto-captured)

**Code Example**:

```php
AuditLogger::log(
    'form_template_version_created',
    ['form', 'workflow', 'versioning'],
    $formTemplate,
    [],
    [
        'from_template_id' => $originalTemplate->id,
        'to_template_id' => $versionSnapshot->id,
        'version_number' => $versionSnapshot->version,
    ]
);
```

---

## Admin Authorization and Access Control

### Role-Based Access
Administrative routes are protected using Laravel role middleware:

```php
Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::post('/admin/forms/approve', [FormApprovalController::class, 'approve']);
    Route::post('/admin/forms/reject', [FormApprovalController::class, 'reject']);
    Route::get('/admin/forms/pending', [FormApprovalController::class, 'pending']);
});
```

### Unauthorized Access Logging
Unauthorized access attempts to admin routes are captured as `access_denied` events:

* **Event Name**: `access_denied`
* **Tags**: `['authz', 'outcome:blocked']`
* **Metadata**: HTTP method, route name, authorization exception reason
* **Actor**: Attempting user account UUID (auto-captured)
* **Context**: IP address, user agent, timestamp (auto-captured)

---

## Audit Logging Standards for Admin Workflows

### Audit Logger Service
All admin workflow actions must use the centralized `AuditLogger` service:
* **Location**: `app/Services/AuditLogger.php`
* **Method Signature**:

```php
AuditLogger::log(
    string $event,
    array $tags = [],
    ?Model $auditable = null,
    array $oldValues = [],
    array $newValues = []
): void
```

### PHI & PII Protection Guardrails
Before logging, the `AuditLogger` checks metadata for sensitive patterns and throws an `InvalidArgumentException` with fail-closed behavior if detected.

**Blocked Key Patterns**:
* `password`, `pass`, `pwd`, `token`, `secret`
* `email`, `name`, `address`
* `dob`, `date_of_birth`
* `health`, `symptom`, `diagnosis`, `notes`, `form_response`
* `encrypted_values`, `responses`

**Blocked Value Patterns**:
* Text values > 500 characters (prevents large text blobs)

| Safe Metadata to Log | Never Log |
| :--- | :--- |
| IDs (`form_id`, `template_id`, `user_id`) | Health form entry values |
| Counts (`entry_count`, `submission_count`) | Symptoms or medical details |
| Status transitions (`pending` → `approved`) | Free-text rejection reasons |
| Event codes and reason codes | Names, emails, or addresses |
| Version numbers and timestamps | Passwords or tokens |

### Automatic Context Captured
The `AuditLogger` automatically captures and stores:
* **Actor UUID**: Resolved from authenticated user's `account_id`
* **IP Address**: IPv4 or IPv6 of requester
* **User Agent**: HTTP user agent string
* **Timestamp**: ISO 8601 creation time
* **URL**: Full request URL including query parameters

---

## Audit Event Reference Table

| Event | Trigger | State Transition | Tags | Metadata |
| :--- | :--- | :--- | :--- | :--- |
| `form_template_submitted` | User submits form | `draft` → `pending` | `['form', 'workflow']` | `status_transition`, `template_version`, `form_template_id` |
| `form_template_approved` | Admin approves form | `pending` → `approved` | `['form', 'workflow', 'outcome:approved']` | `status_transition`, `template_version`, `form_template_id` |
| `form_template_rejected` | Admin rejects form | `pending` → `rejected` | `['form', 'workflow', 'outcome:rejected']` | `status_transition`, `template_version`, `reason_provided` |
| `form_template_version_created` | Version snapshot created | N/A | `['form', 'workflow', 'versioning']` | `from_template_id`, `to_template_id`, `version_number` |
| `access_denied` | Unauthorized attempt | N/A | `['authz', 'outcome:blocked']` | `method`, `route`, `reason` |

---

## Testing Admin Workflows

### Audit Log Verification
Verify audit events are recorded correctly in tests:

```php
// Verify form submission for approval
$this->actingAs($provider, 'sanctum')
    ->postJson('/api/forms/submit-for-approval', ['form_id' => $form->id]);

$this->assertDatabaseHas('audits', [
    'event' => 'form_template_submitted',
    'user_id' => $provider->account_id,
]);

// Verify admin approval
$this->actingAs($admin, 'sanctum')
    ->postJson('/api/admin/forms/approve', ['form_id' => $form->id]);

$this->assertDatabaseHas('audits', [
    'event' => 'form_template_approved',
    'user_id' => $admin->account_id,
]);
```

### Querying Workflow Events

```php
// Find all form submission events
$submissions = \OwenIt\Auditing\Models\Audit::where('event', 'form_template_submitted')->get();

// Find approval events by date range
$approvals = \OwenIt\Auditing\Models\Audit::where('event', 'form_template_approved')
    ->whereBetween('created_at', [$from, $to])
    ->get();

// Find all admin actions by admin user
$adminActions = \OwenIt\Auditing\Models\Audit::where('user_id', $adminId)
    ->whereIn('event', ['form_template_approved', 'form_template_rejected'])
    ->get();
```

---

## Data Integrity & Audit Trail Management

### Append-Only Design
* Audit logs are designed as **append-only** records.
* Application logic does not update or delete audit records during normal operation.
* Audit logs contain only operational/security context; PHI remains in dedicated health data tables.

### Backup and Restoration
* Backup processes must include the `audits` table to preserve compliance records.
* Restoration procedures must ensure audit logs remain consistent with application data history.

### Audit Log Retention
* **Storage Duration**: Audit logs retained for **2 years minimum**.
* **Rationale**: Governance/security records support investigation, compliance reviews, and breach detection.
* **After 2-Year Period**: Logs may be securely archived to cold storage or securely purged using cryptographic deletion.

---

## Compliance Framework

* **HIPAA (U.S.)**: 45 CFR § 164.312(b) - Audit Controls; tracks all access attempts and modifications.
* **GDPR (Europe)**: Article 5(1)(f) - Integrity and Confidentiality; audit logs demonstrate security measures.
* **PHIPA (Ontario)**: Section 9.1 - Reasonable Steps; audit logging demonstrates data protection steps.

---

## Implementation Checklist

* [ ] Implement `form_template_submitted` audit logging in form submission endpoint.
* [ ] Implement `form_template_approved` audit logging in approval controller.
* [ ] Implement `form_template_rejected` audit logging in rejection controller.
* [ ] Implement `form_template_version_created` audit logging in versioning service.
* [ ] Verify `access_denied` events logged for unauthorized admin access.
* [ ] Add PHI guardrails to `AuditLogger` service.
* [ ] Implement audit retention policy and scheduled cleanup jobs.
* [ ] Add audit log verification tests.

Would you like me to generate the SQL schema for the `audits` table to complement this documentation?
