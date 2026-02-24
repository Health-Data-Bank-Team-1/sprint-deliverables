**Updated Audit & Compliance Documentation (Sprint 4)**

**Purpose**

The Health-Data Bank system handles sensitive user health information for precariously housed individuals. To support accountability, security, and compliance requirements, the system implements **event-based audit logging**. Audit logs provide a tamper-evident record of significant actions (authentication, access control denials, form submissions, and admin workflow actions) without storing PHI/PII in the audit metadata.

**Audit Log Storage**

Audit events are recorded in the **audit_logs** database table using a centralized **AuditLogger** service. This table is designed as an **append-only event stream** for security and workflow monitoring.

**Core fields recorded for each event include:**

- actor_id (UUID from accounts, nullable for guest/unknown)
- action_type (string event name)
- outcome (nullable string: e.g., success, failure, blocked)
- reason_code (nullable string: short code for failures/blocks)
- target_type / target_id (nullable strings identifying what was acted on)
- ip_address, user_agent (request context)
- metadata (JSON, sanitized, no PHI/PII)
- timestamp (event time)

**Audit Logger Controls and PHI/PII Protection**

To prevent accidental logging of sensitive information:

- Audit logging is **event-based** and only records identifiers, counts, statuses, and controlled codes.
- The AuditLogger sanitizes metadata and blocks risky keys and patterns (e.g., password/token/email/notes/payload/response fields).
- Large free-text values and oversized arrays are excluded to reduce the risk of capturing raw request bodies or health responses.
- **Form entry values (PHI)** are never written into the audit metadata.

If an audit insert fails for any reason, the logger records a warning in application logs and does not block core user operations.

**Audit Event Coverage**

Audit events are captured across four major areas:

**1) Authentication and Account Security (Fortify/Jetstream)**

Audit listeners are registered in EventServiceProvider to capture:

- login_success (successful authentication)
- login_attempt (failed authentication; reason_code invalid_credentials)
- logout
- register_success
- password_reset_requested (reset notification sent)
- password_reset_completed (password successfully reset)
- password_changed (password changed by authenticated user)
- access_denied (AuthorizationException / AccessDeniedHttpException; outcome blocked)

**Key compliance behavior:** credentials, tokens, and reset data are not logged. Only event context (guard, channel) and account identifiers are stored.

**2) Form Submission (User Data Entry)**

The API form submission endpoint logs:

- form_submission_success after successful validation and persistence of submission + entries.
- form_submission_failed for validation failures (reason_code = validation_failed) with **failed field names only**.
- form_submission_failed for server failures (reason_code = server_error) without exception traces in the DB.
- form_submission_failed for identity mapping issues (reason_code = account_mapping_failed) when an account UUID cannot be resolved.

**Metadata stored for success includes:**

- form_template_id
- entry_count
- submission_id (as target_id)

**PHI protection:** no entry values are included in audit metadata.

**3) Admin Workflow: Form Template Approval**

Admin approval actions are audited in FormTemplateApprovalService:

- form_template_submitted (draft → pending)
- form_template_approved (pending → approved)
- form_template_rejected (pending → rejected)

**Metadata includes:**

- status transitions (from_status, to_status)
- template_version
- reason_provided boolean for rejection (the rejection text is not stored in audit logs)

**4) Versioning / Snapshot Events**

When version snapshots are created (approval/versioning flow), the system logs:

- form_template_version_created (version snapshot created)

Metadata includes from/to template ID and version numbers only.

**Access Control and Administrative Protections**

Administrative routes are protected using role middleware (role:admin). Unauthorized access attempts are captured by the exception handler as access_denied events. Audit logging supports investigation of privilege misuse or attempted access escalation without requiring sensitive request contents.

**Data Integrity, Retention, and Backup Considerations**

- Audit logs are designed to be **append-only**; application logic does not update or delete audit records during normal operation.
- Audit logs contain only operational/security context; PHI remains in dedicated health data tables.
- Backup processes should include the audit_logs table to preserve compliance records. Restoration procedures should ensure audit logs remain consistent with application data history.

**Verification With Real Test Data (Sprint 4 Validation)**

To verify audit log persistence, the following tests are executed:

1.  Successful login → login_success
2.  Failed login attempt → login_attempt (failure)
3.  Logout → logout
4.  Password reset request → password_reset_requested
5.  Password reset completion → password_reset_completed
6.  Submit valid health form → form_submission_success
7.  Submit invalid health form (missing required fields) → form_submission_failed with validation_failed
8.  Admin submits template for approval → form_template_submitted
9.  Admin approves template → form_template_approved
10. Admin rejects template → form_template_rejected

Each test is verified by checking the latest rows in audit_logs for correct action_type, outcome, target_type/target_id, and presence of ip_address/user_agent.

**Audit Event Summary Table**

The table defines each audit event, trigger conditions, auditable target, and metadata stored. This table is maintained as the authoritative reference for audit behavior and is updated whenever new auditable events are introduced.