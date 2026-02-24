**Audit Logging - Developer Guide (Updated for Event-Based System)**

**Purpose**

All audit logging in the Health-Data Bank system must go through the centralized **AuditLogger** service.

This ensures that:

- Audit entries are consistent across the system
- No PHI or sensitive data is logged
- All logs follow the same event-based structure
- Logs are compatible with the audit_logs table schema
- IP address and user agent are automatically captured

Audit events must **never** be written directly to the audit_logs table from controllers or models.

**Audit Logging Pattern**

Use the centralized service:

AuditLogger::log(

actionType,

outcome,

reasonCode,

targetType,

targetId,

metadata,

actorOverride

);

**Parameter Definitions**

|     |     |
| --- | --- |
| **Parameter** | **Description** |
| actionType | String name of the event (e.g., form_submission_success) |
| outcome | success, failure, or blocked |
| reasonCode | Short, non-sensitive reason string (nullable) |
| targetType | Logical target category (e.g., form_submission, form_template, account) |
| targetId | Identifier of affected record (nullable) |
| metadata | JSON-safe array (no PHI) |
| actorOverride | Optional UUID override for accounts.id |

**Example Usage**

**Successful Form Submission**

AuditLogger::log(  
'form_submission_success',  
'success',  
null,  
'form_submission',  
(string) $submission->id,  
\[  
'form_template_id' => $validated\['form_template_id'\],  
'entry_count' => count($validated\['entries'\]),  
\],  
$accountId  
);

**Event Naming Rules**

All event names must:

- Use lowercase snake_case
- Be descriptive and consistent
- Represent a single logical action

**Current Standard Events**

Authentication:

- login_success
- login_attempt
- logout
- register_success
- password_reset_requested
- password_reset_completed
- password_changed
- access_denied

Form Submission:

- form_submission_success
- form_submission_failed

Admin Workflow:

- form_template_submitted
- form_template_approved
- form_template_rejected
- form_template_version_created

**Outcome Values**

Allowed values:

- success
- failure
- blocked

**Reason Codes (Short, Non-Sensitive Only)**

Examples:

- validation_failed
- server_error
- account_mapping_failed
- invalid_state
- invalid_credentials

Reason codes must:

- Be short
- Contain no PHI
- Not contain user input
- Not contain free text

**What to Use as targetType**

Use logical domain groupings:

|     |     |
| --- | --- |
| **Action** | **targetType** |
| Login/logout | account |
| Form submission | form_submission |
| Form approval | form_template |
| Version snapshot | form_template |
| Access denied | route |

**Privacy & PHI Rules**

Allowed in metadata:

- IDs (submission_id, template_id)
- Status transitions
- Counts (entry_count, error_count)
- Boolean flags
- Version numbers

Never log:

- Health form entry values
- Symptoms or medical details
- Free-text rejection reasons
- Names, emails, DOB
- Full request payloads
- Provider notes

If debugging is needed, use development logs only - never audit logs.

**Automatic Context Captured**

The **AuditLogger** automatically stores:

- IP address
- User agent
- Timestamp
- Actor UUID (resolved from accounts table)

These should NOT be manually added to metadata.

**When to Log**

Log an audit entry when:

- Authentication succeeds or fails
- A user submits a form
- A form submission fails validation
- An admin changes workflow state
- Access is denied by authorization middleware
- A password is reset or changed

**Summary**

The system uses an **event-based, privacy-safe audit logging model**.

Each audit entry records:

- Who performed the action
- What happened
- Whether it succeeded or failed
- What object was affected
- Minimal non-sensitive metadata
- Request context (IP + user agent)

This approach ensures compliance, traceability, and protection of sensitive health information.