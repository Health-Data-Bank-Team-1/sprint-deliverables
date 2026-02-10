**Audit Event Summary Table**

|     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- |
| **Event Name** | **Triggered By** | **When It Fires** | **Auditable** | **Tags** | **Metadata Stored** |
| form_submit_success | User | After a form submission is validated and saved successfully | FormSubmission | forms, submit, outcome:success | form_id, submission_id, form_version,<br><br>Field_count,<br><br>Route, method, ip_address, user_agent |
| form_submit_failed_validation | User | When submission is blocked due to validation errors | User | forms, submit, outcome:blocked, reason:validation | form_id, form_version, error_count, route, method, ip_address, user_agent |
| form_submit_failed_server | User | When submission fails due to server/DB exception | User | forms, submit, outcome:error, reason:server | form_id, exception_class, route, method, ip_address, user_agent |
| admin_form_approve | Admin | When admin approves a new/updated form | formTemplate / FormVersion | forms, admin, approval, outcome:approved | form_id, form_version, old_status, new_status, route, method, ip_address, user_agent |
| admin_form_reject | Admin | When admin rejects a new/updated form | FromTemplate/ FormVersion | forms, admin, approval, outcome:rejected | form_id, form_version, old_status, new_status, reason_code, route, method, ip_address, user_agent |
| admin_form_archive | Admin | When admin archives/disables a form | FormTemplate | forms, admin, lifecycle, outcome:archived | form_id, form_version, old_status, new_status, route, method, ip_address, user_agent |

Success submissions attach to FormSubmission; failures attach to the User; admin moderation attaches to FormTemplate/FormVersion.

Reason codes must be non-sensitive and short (e.g., missing_required_fields).

