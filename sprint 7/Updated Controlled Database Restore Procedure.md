
# Provider & Admin Workflow Architecture (Sprint 7)

## Purpose

This document defines the complete technical architecture for provider and administrator workflows in the Health-Data Bank system. It covers the system design, data models, API endpoints, service layer logic, and integration patterns that enable form template management, submission workflows, and approval processes.

---

## Architecture Overview

The Health-Data Bank follows a layered, service-oriented architecture:

```
┌─────────────────────────────────────────────────────┐
│                 Client Layer (Frontend)              │
│         Livewire Components / Vue / API Calls        │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│              HTTP Routing Layer (routes/)            │
│      Maps URLs to Controllers (web.php / api.php)   │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│           Controller Layer (HTTP Handling)           │
│   • FormTemplateController                          │
│   • FormTemplateApprovalController                  │
│   • AdminFormTemplateController                     │
│   Request validation, auth checks, response format  │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│         Service Layer (Business Logic)              │
│   • FormTemplateApprovalService                     │
│   • FormSubmissionService (future)                  │
│   • AuditLogger Service                            │
│   Workflow rules, state transitions, validations    │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│     Repository/Repository Pattern (Data Access)     │
│   • FormTemplateRepository                          │
│   • FormSubmissionRepository                        │
│   Database queries, relationships, collections      │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│           Model Layer (Data Structure)              │
│   • FormTemplate                                    │
│   • FormSubmission                                  │
│   • FormTemplateVersion                            │
│   • User (with roles/permissions)                  │
│   Eloquent models, relationships, attributes        │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│       Database Layer (MySQL/PostgreSQL)             │
│   • form_templates table                            │
│   • form_submissions table                          │
│   • form_template_versions table                    │
│   • audit_logs table                               │
└─────────────────────────────────────────────────────┘
```

---

## Data Models & Database Schema

### FormTemplate Model

**Location**: `app/Models/FormTemplate.php`

**Purpose**: Represents a reusable form template that providers create and admins approve.

**Database Table**: `form_templates`

**Fields**:
```php
- id (UUID primary key)
- title (string, max 255)
- description (text, nullable)
- schema (JSON) // Form field definitions
- version (integer, default 1)
- approval_status (enum: 'draft', 'pending', 'approved', 'rejected')
- approved_by (UUID, nullable) // Foreign key to users.id
- approved_at (timestamp, nullable)
- rejection_reason (text, nullable)
- created_at (timestamp)
- updated_at (timestamp)
```

**Relationships**:
```php
// Has many form submissions
$template->submissions();

// Has many versions
$template->versions();

// Belongs to approving admin
$template->approver();

// Has many form fields
$template->fields();
```

**Fillable Attributes**:
```php
protected $fillable = [
    'id', 'title', 'schema', 'version', 'approval_status',
    'approved_by', 'approved_at', 'rejection_reason', 'description'
];
```

**Default Attributes**:
```php
protected $attributes = [
    'version' => 1,
    'approval_status' => 'draft'
];
```

### FormSubmission Model

**Location**: `app/Models/FormSubmission.php`

**Purpose**: Represents a user's submission of an approved form template.

**Database Table**: `form_submissions`

**Fields**:
```php
- id (UUID primary key)
- form_template_id (UUID, foreign key)
- user_id (UUID, foreign key to users.id)
- provider_id (UUID, nullable, foreign key)
- status (enum: 'draft', 'submitted', 'approved', 'flagged')
- submitted_at (timestamp, nullable)
- approved_at (timestamp, nullable)
- approval_notes (text, nullable)
- encrypted_values (JSON) // Encrypted health data
- created_at (timestamp)
- updated_at (timestamp)
```

**Relationships**:
```php
$submission->template();      // Belongs to FormTemplate
$submission->user();          // Belongs to User
$submission->provider();      // Belongs to provider User
$submission->entries();       // Has many FormEntries
```

### FormTemplateVersion Model

**Location**: `app/Models/FormTemplateVersion.php`

**Purpose**: Immutable snapshots of form templates at approval time for audit trail.

**Database Table**: `form_template_versions`

**Fields**:
```php
- id (UUID primary key)
- form_template_id (UUID, foreign key)
- version (integer)
- title (string)
- schema (JSON)
- created_by (UUID, foreign key) // Admin who approved
- created_at (timestamp)
```

**Relationships**:
```php
$version->template();     // Belongs to FormTemplate
$version->creator();      // Belongs to User (approving admin)
```

### User Model (with Roles)

**Location**: `app/Models/User.php`

**Purpose**: Represents system users with role-based access control.

**Roles in Workflow**:
- `admin`: Can approve/reject form templates, manage system
- `provider`: Can create and submit form templates for approval
- `researcher`: Can generate reports and cohort analyses
- `user`: Can submit health data through approved forms

**Permissions**:
```php
// Admin permissions
- 'approve-forms'
- 'reject-forms'
- 'view-audit-logs'
- 'restore-database'

// Provider permissions
- 'create-forms'
- 'submit-forms'
- 'view-own-forms'

// User permissions
- 'submit-forms'
- 'view-own-submissions'
```

---

## API Endpoints

### Form Template Management

#### Create Form Template
```
POST /api/form-templates
Authentication: Required (Bearer token)
Authorization: Provider or Admin

Request:
{
    "title": "Weekly Vitals Report",
    "description": "Collect vital signs weekly",
    "schema": {
        "fields": [
            {"name": "heart_rate", "type": "number", "required": true},
            {"name": "blood_pressure", "type": "string", "required": true}
        ]
    }
}

Response:
{
    "id": "uuid...",
    "title": "Weekly Vitals Report",
    "approval_status": "draft",
    "version": 1,
    "created_at": "2026-03-17T..."
}

Status Codes:
- 201: Created
- 422: Validation failed
- 403: Not authorized
```

#### Update Form Template
```
PUT /api/form-templates/{id}
Authentication: Required
Authorization: Owner (creator) or Admin

Request: (same as create)

Response: Updated template object

Status Codes:
- 200: Updated
- 404: Not found
- 422: Validation failed
```

#### List Form Templates (Providers)
```
GET /api/form-templates
Authentication: Required
Authorization: Provider

Query Parameters:
- approval_status: 'draft', 'pending', 'approved', 'rejected'
- search: Search by title
- page: Pagination
- per_page: Items per page

Response:
{
    "data": [...],
    "links": {...},
    "meta": {...}
}
```

### Form Template Approval Workflow

#### Submit Template for Approval
```
POST /api/form-templates/{id}/submit
Authentication: Required
Authorization: Admin

Description: Moves template from draft → pending

Request: (empty body)

Response:
{
    "message": "Submitted for approval",
    "approval_status": "pending"
}

Audit Log Event: form_template_submitted
```

#### Approve Template
```
POST /api/admin/forms/{id}/approve
Authentication: Required
Authorization: Admin role

Description: Moves template from pending → approved

Request: (empty body)

Response:
{
    "message": "Template approved",
    "approval_status": "approved",
    "approved_at": "2026-03-17T..."
}

Audit Log Event: form_template_approved
```

#### Reject Template
```
POST /api/admin/forms/{id}/reject
Authentication: Required
Authorization: Admin role

Description: Moves template from pending → rejected

Request:
{
    "reason": "Invalid field structure"
}

Response:
{
    "message": "Template rejected",
    "approval_status": "rejected",
    "rejection_reason": "Invalid field structure"
}

Audit Log Event: form_template_rejected
```

#### Admin List Form Templates
```
GET /api/admin/forms
Authentication: Required
Authorization: Admin role

Query Parameters:
- approval_status: 'draft', 'pending', 'approved', 'rejected'
- search: Search by title
- sort_by: 'created_at', 'title', 'approval_status', 'version'
- sort_dir: 'asc', 'desc'
- page: Pagination
- per_page: Items per page

Response:
{
    "data": [
        {
            "id": "uuid...",
            "title": "Form Name",
            "approval_status": "pending",
            "version": 1,
            "created_at": "...",
            "approver": {...}  // null if not approved/rejected
        }
    ],
    "pagination": {...}
}
```

### Form Versioning

#### Get Version History
```
GET /api/form-templates/{id}/versions
Authentication: Required

Response:
{
    "data": [
        {
            "version": 3,
            "created_at": "2026-03-10T...",
            "created_by": "admin_uuid"
        },
        {
            "version": 2,
            "created_at": "2026-03-05T...",
            "created_by": "admin_uuid"
        }
    ]
}
```

#### Rollback to Previous Version
```
POST /api/form-templates/{id}/rollback/{version}
Authentication: Required
Authorization: Admin role

Description: Revert template to a previous approved version

Response:
{
    "message": "Rolled back to version {version}",
    "current_version": version_number
}

Audit Log Event: form_template_version_rollback
```

---

## Service Layer Logic

### FormTemplateApprovalService

**Location**: `app/Services/FormTemplateApprovalService.php`

**Purpose**: Handles form template approval workflow logic.

#### submitForApproval()
```php
public function submitForApproval(FormTemplate $template): void
```

**Logic**:
1. Verify template is in 'draft' status
2. Update approval_status to 'pending'
3. Log audit event with old/new status
4. Return success or throw WorkflowException

**Audit Event**:
```php
AuditLogger::log(
    'form_template_submitted_for_approval',
    ['forms', 'resource:template', 'workflow:approval', 'outcome:success'],
    null,
    ['approval_status' => 'draft'],
    [
        'template_id' => (string) $template->id,
        'approval_status' => 'pending'
    ]
);
```

#### approve()
```php
public function approve(FormTemplate $template, User $admin): void
```

**Logic**:
1. Verify template is in 'pending' status
2. Update approval_status to 'approved'
3. Set approved_by = admin ID
4. Set approved_at = current timestamp
5. Clear rejection_reason field
6. Create immutable version snapshot
7. Log audit event

**Version Snapshot**:
- Creates FormTemplateVersion record
- Captures template schema at approval time
- Links to approving admin
- Cannot be modified (immutable)

#### reject()
```php
public function reject(FormTemplate $template, User $admin, string $reason): void
```

**Logic**:
1. Verify template is in 'pending' status
2. Validate rejection reason (max 255 chars)
3. Update approval_status to 'rejected'
4. Set approved_by = admin ID
5. Set approved_at = current timestamp
6. Store rejection_reason
7. Log audit event (reason stored separately, not in audit)

**Rejection Workflow**:
- Provider receives notification of rejection
- Provider can revise template
- Provider resubmits for approval (workflow repeats)
- Audit trail shows all rejection attempts

### AuditLogger Service

**Location**: `app/Services/AuditLogger.php`

**Purpose**: Centralized audit logging for all workflow events.

**Method Signature**:
```php
public static function log(
    string $event,
    array $tags = [],
    ?Model $auditable = null,
    array $oldValues = [],
    array $newValues = []
): void
```

**Implementation**:
```php
AuditLogger::log(
    'form_template_approved',                          // Event name
    ['forms', 'workflow', 'outcome:success'],          // Tags
    $formTemplate,                                      // Model being acted upon
    ['approval_status' => 'pending'],                  // Old values
    [                                                   // New values
        'approval_status' => 'approved',
        'approved_by' => $admin->id
    ]
);
```

**Features**:
- Automatic capture of IP address, user agent, timestamp
- PHI/PII blocking (sensitive data guardrails)
- Append-only audit trail
- No ability to modify/delete audit records
- 2-year retention policy

---

## Workflow State Machine

### Form Template States

```
┌─────────────────────────────────────┐
│        INITIAL STATE: DRAFT         │
│  (Provider creates/edits template)  │
└──────────────┬──────────────────────┘
               │
       submitForApproval()
               │
               ▼
┌──────────────────────────────────────┐
│      PENDING APPROVAL                │
│ (Awaiting admin review)              │
│ Can only: view, edit (if admin adds) │
└──────────┬──────────────────┬────────┘
           │                  │
      approve()          reject()
           │                  │
           ▼                  ▼
    ┌────────────┐     ┌──────────────┐
    │ APPROVED   │     │  REJECTED    │
    │ (Active)   │     │ (Needs revision)
    │            │     │              │
    │ Create     │     │ Can resubmit │
    │ Version    │     │ for approval │
    │ Snapshot   │     │              │
    └────────────┘     └──────────┬───┘
         │                        │
         │              submitForApproval()
         │                        │
         │                        ▼
         │              Back to PENDING
         │
    Templates can be
    used for form
    submissions only
    if APPROVED
```

### State Transition Validation

**Allowed Transitions**:
```
draft → pending (submitForApproval)
pending → approved (approve)
pending → rejected (reject)
rejected → pending (submitForApproval - provider revises)
approved → draft (rollback version)
```

**Blocked Transitions**:
```
draft → approved (must go through pending first)
approved → pending (must use rollback)
rejected → approved (must resubmit through pending)
```

**Validation Rules**:
```php
if ($template->approval_status !== 'draft') {
    throw new WorkflowException(
        'Only draft templates can be submitted for approval.'
    );
}

if ($template->approval_status !== 'pending') {
    throw new WorkflowException(
        'Only pending templates can be approved.'
    );
}
```

---

## Controllers

### FormTemplateController

**Location**: `app/Http/Controllers/FormTemplateController.php`

**Methods**:

**store()**: Create new form template
- Route: `POST /api/form-templates`
- Validates title, schema, description
- Sets initial status to 'draft'
- Logs creation event

**update()**: Update existing template
- Route: `PUT /api/form-templates/{id}`
- Only allowed for draft/rejected templates
- Logs update event

### FormTemplateApprovalController

**Location**: `app/Http/Controllers/Admin/FormTemplateApprovalController.php`

**Methods**:

**submit()**: Submit for approval
- Route: `POST /api/form-templates/{id}/submit`
- Calls FormTemplateApprovalService::submitForApproval()
- Returns JSON response

**approve()**: Admin approves
- Route: `POST /api/admin/forms/{id}/approve`
- Calls FormTemplateApprovalService::approve()
- Creates version snapshot
- Logs approval event

**reject()**: Admin rejects
- Route: `POST /api/admin/forms/{id}/reject`
- Validates rejection reason
- Calls FormTemplateApprovalService::reject()
- Logs rejection event

### AdminFormTemplateController

**Location**: `app/Http/Controllers/Admin/AdminFormTemplateController.php`

**Methods**:

**index()**: List form templates (admin view)
- Route: `GET /api/admin/forms`
- Filters by approval_status, search
- Sorting by multiple fields
- Pagination

---

## Routes Configuration

### API Routes

**Location**: `routes/api.php`

```php
// Provider routes (authenticated)
Route::middleware('auth:sanctum')->group(function () {
    Route::post('/form-templates', [FormTemplateController::class, 'store']);
    Route::put('/form-templates/{id}', [FormTemplateController::class, 'update']);
    Route::get('/form-templates', [FormTemplateController::class, 'index']);
    Route::post('/form-templates/{id}/submit', 
        [FormTemplateApprovalController::class, 'submit']);
    Route::get('/form-templates/{id}/versions', 
        [FormTemplateVersionController::class, 'index']);
});

// Admin routes (authenticated + admin role)
Route::middleware(['auth:sanctum', 'role:admin'])->prefix('admin/forms')->group(function () {
    Route::get('/', [AdminFormTemplateController::class, 'index']);
    Route::post('{template}/approve', [FormTemplateApprovalController::class, 'approve']);
    Route::post('{template}/reject', [FormTemplateApprovalController::class, 'reject']);
    Route::post('{template}/submit', [FormTemplateApprovalController::class, 'submit']);
    Route::post('form-templates/{template}/rollback/{version}',
        [FormTemplateVersionController::class, 'rollback']);
});
```

### Web Routes

**Location**: `routes/web.php`

```php
// Admin UI (Livewire component)
Route::get('/admin/forms', FormTemplatesIndex::class)
    ->middleware('role:admin')
    ->name('admin.forms.index');

// Form template creation/editing
Route::prefix('form-templates')->group(function () {
    Route::post('/', [FormTemplateController::class, 'store'])
        ->name('form-templates.store');
    Route::put('{template}', [FormTemplateController::class, 'update'])
        ->name('form-templates.update');
});
```

---

## Livewire Components

### FormTemplatesIndex

**Location**: `app/Livewire/Admin/FormTemplatesIndex.php`

**Purpose**: Admin interface for reviewing and approving/rejecting forms.

**Features**:
- Real-time search by title
- Filter by approval status
- Pagination
- Reject modal with reason input
- Approve/reject buttons with confirmation

**State Management**:
```php
public string $search = '';
public string $approvalStatus = '';
public int $perPage = 15;
public bool $showRejectModal = false;
public ?string $rejectTemplateId = null;
public string $rejectReason = '';
```

**Methods**:
- `approve()`: Call service to approve template
- `openReject()`: Show rejection modal
- `reject()`: Call service to reject with reason
- `render()`: Query and display templates

---

## Role-Based Access Control

### Authorization Middleware

**Location**: `app/Http/Middleware/`

**Roles**:
- `admin`: Full access to all approval operations
- `provider`: Create and submit templates for approval
- `researcher`: View aggregated reports
- `user`: Submit health data through approved forms

**Middleware Stack**:
```php
Route::middleware(['auth:sanctum', 'role:admin'])->group(...)
```

**Permission Checks**:
```php
// Using Spatie/Laravel-permission
$user->hasRole('admin')
$user->can('approve-forms')
$user->can('reject-forms')
```

---

## Error Handling

### WorkflowException

**Location**: `app/Exceptions/WorkflowException.php`

**Purpose**: Thrown for invalid state transitions or workflow violations.

**Usage**:
```php
throw new WorkflowException(
    'Only draft templates can be submitted for approval.'
);
```

**Exception Handler**:
```php
// Returns JSON with error message
{
    "error": true,
    "message": "Only draft templates can be submitted for approval.",
    "status": 422
}
```

### Validation Exceptions

Laravel's built-in validation returns 422 status with validation errors:
```json
{
    "message": "The given data was invalid.",
    "errors": {
        "title": ["The title field is required."],
        "reason": ["The reason must be a string."]
    }
}
```

---

## Testing

### Test Files

**Location**: `tests/Feature/Admin/`

**Test Classes**:
- `FormTemplateApprovalTest.php`: Workflow transitions
- `AdminFormTemplateIndexTest.php`: Admin listing and filtering

**Example Test**:
```php
public function test_admin_can_approve_a_pending_form()
{
    $this->actingAsAdmin();
    
    $form = FormTemplate::factory()->create([
        'approval_status' => 'pending'
    ]);
    
    $response = $this->postJson("/api/admin/forms/{$form->id}/approve");
    
    $response->assertStatus(200);
    $this->assertDatabaseHas('form_templates', [
        'id' => $form->id,
        'approval_status' => 'approved'
    ]);
}
```

---

## Integration Points

### Audit Logging Integration
- All workflow events logged automatically
- IP address, user agent, timestamp captured
- Actor (admin ID) recorded
- PHI/PII blocking enforced

### Form Submission Integration (Future)
- Only APPROVED templates available for user submissions
- Submission status tracking
- Provider review of user submissions
- Health data encryption

### Report Generation Integration (Future)
- Approved templates used for report filters
- Submission data included in cohort analysis
- Version snapshots ensure historical accuracy

---

## Security Considerations

### Authentication
- All endpoints require Bearer token (Laravel Sanctum)
- Session-based for web UI (Laravel sessions)

### Authorization
- Role-based access control (Spatie/Laravel-Permission)
- Admin middleware ensures only admins can approve/reject
- Provider restriction on form creation

### Data Protection
- Audit logs append-only (cannot be modified/deleted)
- Form schema stored as JSON (queryable)
- Health data encrypted before storage
- No PHI in audit metadata

### Validation
- Input validation at controller level
- State validation at service level
- Workflow rules enforced at database level

---

## Performance Considerations

### Database Indexing
```sql
-- Recommended indexes for form_templates table
INDEX idx_approval_status (approval_status)
INDEX idx_created_at (created_at)
INDEX idx_title_search (title)
INDEX idx_approved_by (approved_by)
```

### Query Optimization
- Eager loading of relationships (forms → versions, approver)
- Pagination for large result sets (15-50 per page)
- Indexed searches on title and approval_status

### Caching (Future)
- Cache approved template list
- Invalidate on new approvals
- TTL: 1 hour or on-demand invalidation

---

## Deployment Checklist

- [ ] All migrations run successfully
- [ ] Routes registered correctly
- [ ] Role-based access control tested
- [ ] Audit logging verified
- [ ] Error handling tested
- [ ] Workflow state transitions validated
- [ ] Version snapshots created correctly
- [ ] API responses match schema
- [ ] Admin UI loads and responds correctly
- [ ] Authorization middleware working
- [ ] Database backups tested
- [ ] Monitoring/alerting configured

---

## Related Documentation

- Provider & Admin Workflow Documentation (Sprint 7)
- Controlled Database Restore Procedure (Sprint 7)
- Updated Audit & Compliance Documentation (Sprint 6)
- Data Retention & Audit Requirements (Sprint 2)
