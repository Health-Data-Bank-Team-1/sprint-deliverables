**Audit Event Summary Table**

|     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- |
| **Event Name** | **Triggered By** | **When It Fires** | **Auditable** | **Tags** | **Metadata Stored** |
| form_submission_success | User | After a form submission is validated and saved successfully | FormSubmission | forms, submit, outcome:success | form_template_id,<br><br>submission_id, entry_count, route, method, ip_address, user_agent |
| form_submission_failed | User | When submission is blocked due to **validation errors** | FormTemplate | forms, submit, outcome:failed, reason:validation | failed_fields, error_count, form_template_id, route, method, ip_address, user_agent |
| form_submission_failed | User | When submission fails due to **mapping errors** | FormTemplate | forms, submit, outcome:failed, reason:validation | form_template_id,<br><br>route, method, ip_address, user_agent |
| form_submit_failed_server | User | When submission fails due to **server/DB exception** | FormTemplate | forms, submit, outcome:error, reason:server | form_template_id, route, method, ip_address, user_agent |
| form_template_submitted | Admin | When a draft template is submitted for approval | formTemplate | forms, workflow, outcome:success | from_status, to_status(pending), template_version, route, method, ip_address, user_agent |
| form_template_approved | Admin | When a pending template is approved | formTemplate | forms, workflow, outcome:success | from_status, to_status(approve), template_version, route, method, ip_address, user_agent |
| form_template_rejected | Admin | When admin rejects a new/updated form | FromTemplate | forms, workflow, outcome:rejected | from_status, to_status(rejected), template_version, reason_provided, route, method, ip_address, user_agent |
| form_template_version_created | Admin (via approval workflow) | When a new immutable version snapshot is created | FormTemplateVersion | forms, versioning, outcome:success | from_template_id, to_template_id, from_version, to_version, route, method, ip_address, user_agent |

Security & Compliance Notes

- No PHI or form entry values are stored in audit logs.
- Only metadata such as counts, IDs, and status transitions are recorded.
- Rejection reasons are not stored in audit logs (only a boolean flag).
- All logs are append-only.
- IP address and user agent are automatically captured by the **AuditLogger** service.
