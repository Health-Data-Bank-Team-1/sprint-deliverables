
# SYSTEM GOVERNANCE PROCEDURES
## Administrator Documentation

**Document Version:** 1.0  
**Last Updated:** 2026-03-23  
**Status:** Complete  
**Audience:** System Administrators


## TABLE OF CONTENTS

1. [Introduction](#introduction)
2. [Administrator Role & Responsibilities](#administrator-role--responsibilities)
3. [Access Control & Permissions](#access-control--permissions)
4. [User Account Management](#user-account-management)
5. [Form Template Governance](#form-template-governance)
6. [Audit Logging & Compliance](#audit-logging--compliance)
7. [Database Management](#database-management)
8. [Report Review & Approval](#report-review--approval)
9. [System Maintenance](#system-maintenance)
10. [Security & Access Protocols](#security--access-protocols)
11. [Disaster Recovery & Backups](#disaster-recovery--backups)
12. [Administrator Workflows](#administrator-workflows)
13. [Troubleshooting & Support](#troubleshooting--support)
14. [Change Log & Auditing](#change-log--auditing)

---

## INTRODUCTION

### Purpose

This document provides comprehensive guidance for Health Data Bank System Administrators responsible for maintaining system integrity, managing user accounts, approving content, and ensuring HIPAA compliance.

### Administrative Authority

Administrators possess **full system access** and carry full responsibility for:
- System security and data protection
- User account lifecycle management
- Content approval workflows
- Audit trail maintenance
- Compliance monitoring
- Incident response
- Data integrity verification

### Scope

This documentation covers administrative procedures for:
- **Users & Roles:** Account creation, modification, deletion, role assignment
- **Forms:** Template approval workflows, version management
- **Data:** Audit logs, compliance reporting, data validation
- **Infrastructure:** Database health, backups, maintenance
- **Security:** Access control, authentication, authorization

---

## ADMINISTRATOR ROLE & RESPONSIBILITIES

### Role Definition

**Administrator (Admin)** is the highest privilege level in the Health Data Bank system with unrestricted access to all system functions.

### Key Responsibilities

#### 1. User & Account Management
- Create and activate user accounts
- Assign roles and permissions
- Modify user account information
- Disable/deactivate accounts
- Delete accounts (with proper procedures)
- Reset user passwords
- Manage account status

#### 2. Content Governance
- Review and approve form templates
- Reject inappropriate forms with feedback
- Manage form versioning
- Monitor template lifecycle
- Ensure form quality standards

#### 3. Compliance & Audit
- Monitor audit logs
- Generate compliance reports
- Investigate suspicious activities
- Maintain HIPAA compliance
- Document all sensitive actions
- Review access patterns
- Generate audit reports for external review

#### 4. System Operations
- Monitor system health
- Manage database operations
- Perform backups
- Handle disaster recovery
- Optimize system performance
- Apply system updates
- Manage infrastructure

#### 5. Data Integrity
- Verify data consistency
- Monitor database health
- Validate data migrations
- Check for data corruption
- Perform cleanup operations
- Archive historical data

#### 6. Incident Response
- Respond to security incidents
- Investigate data breaches
- Handle account compromises
- Manage emergency situations
- Document incidents
- Coordinate remediation

---

## ACCESS CONTROL & PERMISSIONS

### Permission Model

The Health Data Bank uses **Spatie Permission** for role-based access control (RBAC).

### Administrator Permissions

Administrators have unrestricted access to all CRUD operations:

| Operation | Permission | Scope |
|-----------|-----------|-------|
| **Create** | create | All resources, all users |
| **Read** | read | All resources, all users |
| **Update** | update | All resources, all users |
| **Delete** | delete | All resources, all users |

### System Roles Hierarchy

```
Administrator (ADMIN)
├── Full System Access
├── User Management
├── Form Approval
├── Database Operations
└── Audit Trail Access

Healthcare Provider (PROVIDER)
├── Patient Management
├── Patient Record Access
├── Report Generation
└── Limited Audit Access (own records)

Researcher (RESEARCHER)
├── Cohort Creation
├── Aggregated Data Access
├── Report Generation
└── Audit Trail Access (own research)

Patient/User (USER)
├── Form Completion
├── Personal Data Access
├── Health Tracking
└── Audit Trail (own records)
```

### Permission Assignment

**Admin Dashboard Path:** `/admin/dashboard`

To manage permissions:

```php
// Programmatic permission assignment
$admin = User::find($userId);
$admin->assignRole('admin');
$admin->givePermissionTo('create', 'read', 'update', 'delete');

// Check permissions
$admin->hasPermissionTo('update');  // Returns true
$admin->hasRole('admin');           // Returns true
```

### Access Enforcement

All routes with admin restrictions use middleware:

```php
Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    // Admin-only routes
    Route::get('/admin/forms', [AdminFormTemplateController::class, 'index']);
    Route::post('/admin/forms/{id}/approve', [FormTemplateApprovalController::class, 'approve']);
});
```

---

## USER ACCOUNT MANAGEMENT

### User Account Lifecycle

#### 1. Account Creation

**Prerequisites:**
- Administrator credentials
- User information (name, email)
- Intended role assignment
- Account type selection

**Creation Process:**

```bash
# Option 1: Via Admin Dashboard
Navigate to: /admin/users/create
Enter: Name, Email, Role, Account Type
Submit: Create User

# Option 2: Via Artisan Command
php artisan make:user {name} {email} {role}

# Option 3: Programmatically
$user = User::factory()->create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'account_id' => $account->id,
]);
$user->assignRole('user');  // or 'provider', 'researcher', 'admin'
```

**Post-Creation Actions:**
1. Send welcome email
2. Provide temporary password
3. Request password change on first login
4. Enable 2FA setup
5. Document creation in audit log

**Example User Data:**

| Field | Value | Notes |
|-------|-------|-------|
| Name | John Doe | Full name |
| Email | john@example.com | Valid email |
| Role | user | user, provider, researcher, admin |
| Account Type | User | User, Researcher, HealthcareProvider, Admin |
| Status | ACTIVE | ACTIVE or DEACTIVATED |
| 2FA | Enabled | Required for all users |

#### 2. Account Modification

**Allowed Changes:**
- Name and contact information
- Email address
- Role and permissions
- Account status
- 2FA settings
- API tokens

**Modification Process:**

```bash
# Navigate to Admin Dashboard
/admin/users/{userId}/edit

# Update fields
- Name
- Email
- Role (dropdown)
- Status (ACTIVE/DEACTIVATED)
- Account type

# Save changes
Submit changes → Audit log entry created
```

**Audit Trail Entry:**
```
Event: user_profile_updated
Tags: admin, resource:user
Old Values: {name: "Old Name"}
New Values: {name: "New Name"}
Timestamp: [automated]
Admin: [current user]
```

#### 3. Role Assignment

**Role Assignment Procedure:**

```php
// Assign role
$user->assignRole('provider');

// Verify assignment
$user->hasRole('provider');  // Returns true

// Get all roles
$user->getRoleNames();  // Returns array of role names

// Revoke role
$user->removeRole('user');
```

**Role Descriptions:**

| Role | Permissions | Typical Users | Dashboard |
|------|------------|---------------|-----------|
| **user** | Read own data, create forms, view health summary | Patients | `/user/dashboard` |
| **provider** | Read multiple patients, create reports, manage care | Clinicians, Nurses | `/provider/dashboard` |
| **researcher** | Create cohorts, access aggregated data, generate reports | Research staff | `/researcher/dashboard` |
| **admin** | All operations, system management, compliance | IT staff, Super users | `/admin/dashboard` |

#### 4. Password Management

**Password Reset Procedure:**

```bash
# Via Admin Dashboard
1. Navigate to /admin/users/{userId}
2. Click "Reset Password"
3. Send temporary password to user email
4. User receives password reset link
5. User sets new password on first login

# Via Artisan
php artisan tinker
>>> $user = User::find($userId);
>>> $user->forceFill(['password' => Hash::make('temporary_password')])->save();
>>> // Send user the temporary password
```

**Password Requirements:**
- Minimum 8 characters
- Mixed case (uppercase and lowercase)
- Numbers and special characters
- Cannot reuse last 3 passwords

#### 5. Two-Factor Authentication (2FA)

**2FA Configuration:**

All users must enable 2FA using Time-based One-Time Password (TOTP):

```bash
# User Setup
1. Login to account
2. Navigate to Profile → Security
3. Click "Enable 2FA"
4. Scan QR code with authenticator app (Google Authenticator, Microsoft Authenticator)
5. Enter verification code
6. Save backup codes in secure location

# Admin Enforcement
Admins can view but CANNOT disable 2FA for users
Only user can disable 2FA after re-verification
```

**Supported 2FA Apps:**
- Google Authenticator
- Microsoft Authenticator
- Authy
- FreeOTP

#### 6. Account Deactivation

**Deactivation Procedure:**

```bash
# Step 1: Notify user
Send email notification about upcoming deactivation
Provide 48-hour notice

# Step 2: Archive user data
Export user's health data
Backup records for compliance

# Step 3: Deactivate account
Navigate to /admin/users/{userId}
Click "Deactivate Account"
Confirm action
Document reason in audit log

# Step 4: Access revocation
User loses login access
API tokens invalidated
Session terminated

# Step 5: Audit entry
Event: account_deactivated
Tags: admin, resource:user, outcome:success
Timestamp: [automated]
Reason: [documented]
```

**Deactivation Effects:**
- User cannot login
- API tokens invalid
- Active sessions terminated
- Data remains in system (not deleted)
- Can be reactivated by admin

#### 7. Account Deletion

⚠️ **WARNING: Permanent Data Loss Operation**

**Deletion Criteria:**
- User consent obtained (in writing or via system confirmation)
- Data retention period elapsed (if applicable)
- No active research projects
- Compliance requirements met
- Final audit entry generated

**Deletion Procedure:**

```bash
# Step 1: Backup verification
Confirm all data backed up
Export records for legal hold
Document backup location

# Step 2: Dependent data cleanup
Remove API tokens
Cancel pending requests
Terminate active sessions

# Step 3: Data deletion
Navigate to /admin/users/{userId}/delete
Confirm user identity
Enter deletion reason
Confirm irreversibility warning

# Step 4: Permanent deletion
Account deleted from system
Associated data removed (configurable)
Audit entry created
Deletion notice sent to compliance team
```

**Deletion Audit Entry:**
```
Event: account_deleted
Tags: admin, resource:user, action:delete, compliance
Old Values: {user_id, email, account_type}
New Values: null
Deletion Reason: [documented]
Timestamp: [automated]
Admin: [current user]
Permanent: true
```

---

## FORM TEMPLATE GOVERNANCE

### Form Template Lifecycle

#### Overview

Forms follow a structured approval workflow ensuring quality and compliance:

```
Draft → Pending Approval → Approved → Published → In Use
           ↓
         Rejected → Revision → Resubmit
```

#### 1. Form Review Dashboard

**Access:** `/admin/dashboard` → Form Review

**Features:**
- View all form templates
- Filter by approval status
- Search by form name
- Approve/reject templates
- View form history

**Status Categories:**

| Status | Meaning | Action Available |
|--------|---------|-----------------|
| **draft** | Creator editing | (Not reviewable) |
| **pending** | Awaiting admin review | Approve or Reject |
| **approved** | Admin approved, published | View/Manage versions |
| **rejected** | Admin rejected | Creator can revise and resubmit |

#### 2. Approval Workflow

**Step 1: Review Pending Form**

```bash
Navigate to: /admin/dashboard → Form Review

View pending forms:
- Form name
- Created by (creator name)
- Created date
- Last modified date
- Preview form structure
```

**Step 2: Evaluate Form Quality**

Review form for:
- ✓ Clear field labels
- ✓ Appropriate question types
- ✓ Logical flow
- ✓ No duplicate fields
- ✓ Proper field validation
- ✓ Completeness
- ✓ HIPAA compliance
- ✓ No PHI in form labels

**Step 3: Approve Form**

```bash
Click "Approve" button on pending form
→ System creates version snapshot
→ Form marked as "approved"
→ Form becomes available to users
→ Audit entry created
→ Creator receives approval notification
```

**Approval Audit Entry:**
```
Event: form_template_approved
Tags: forms, resource:template, workflow:approval, outcome:success
Old Values: {approval_status: "pending"}
New Values: {approval_status: "approved", version: 1}
Template ID: [form template ID]
Admin: [current admin]
Timestamp: [automated]
```

**Step 4: Rejection (if needed)**

```bash
Click "Reject" button on pending form

Modal opens:
- Enter rejection reason (required)
- Character limit: 255 characters
- Click "Confirm Rejection"

Form marked as "rejected"
Creator receives rejection email with reason
Creator can revise and resubmit
Audit entry created
```

**Rejection Audit Entry:**
```
Event: form_template_rejected
Tags: forms, resource:template, workflow:approval, outcome:success
Old Values: {approval_status: "pending"}
New Values: {approval_status: "rejected", rejection_reason: "[reason]"}
Template ID: [form template ID]
Admin: [current admin]
Timestamp: [automated]
Reason: [rejection reason]
```

**Rejection Examples:**

❌ **"This form contains personally identifiable information (PII) in the labels. Please remove: SSN field name."**

❌ **"Field 'Medications' is too open-ended. Please use a dropdown with predefined medication list."**

❌ **"Form title is not descriptive. Change from 'Health' to 'Weekly Health Assessment'."**

#### 3. Version Management

**Version Tracking:**

Each approved form creates an immutable version snapshot:

```bash
Navigate to: Form detail view → Version History

View all versions:
- Version number (1, 2, 3, etc.)
- Created date
- Created by (admin)
- Form structure snapshot
- Rollback option
```

**Version Rollback:**

```bash
Click "Rollback to Version X"
→ System saves current version as new snapshot
→ Restores previous version as current
→ New version number assigned
→ Audit entry created
→ Users now see previous form version
```

**Rollback Audit Entry:**
```
Event: form_template_rollback
Tags: forms, resource:template, version:management
Old Version: [current version number]
New Version: [target version number]
Admin: [current admin]
Timestamp: [automated]
Reason: [if documented]
```

**When to Rollback:**

✓ **Approve:** Recently approved form has critical error discovered

✓ **Approve:** Newer version breaks compatibility with existing data

✓ **Approve:** Field requirements changed, need previous version

❌ **Do Not:** Rollback without documentation

❌ **Do Not:** Rollback to recover deleted form (use backup)

#### 4. Form Version History

**Viewing History:**

```bash
Navigate to: Form detail → Version History

View complete audit trail:
- All versions created
- Who created/approved each version
- When version was created
- Changes made in each version
- Rollback operations
```

**Version Information:**

| Field | Example | Purpose |
|-------|---------|---------|
| Version # | 1, 2, 3 | Incremental version identifier |
| Created | 2026-03-20 | Date version created |
| Created By | admin@example.com | Admin who approved |
| Form Title | Weekly Health Check | Form name at that version |
| Schema | [JSON structure] | Form field definitions |

---

## AUDIT LOGGING & COMPLIANCE

### Audit System Overview

All significant system actions are logged to ensure HIPAA compliance and accountability.

### Audit Log Access

**Access:** `/admin/dashboard` → Audit Log

**Log Viewer Features:**
- View all audit entries
- Filter by date range
- Filter by user
- Filter by event type
- Search by keywords
- Export to CSV
- Generate compliance reports

### Audit Entry Structure

Each audit entry contains:

```json
{
  "id": "uuid",
  "user_type": "App\\Models\\User",
  "user_id": "admin-id",
  "event": "form_template_approved",
  "auditable_type": "App\\Models\\FormTemplate",
  "auditable_id": "form-id",
  "old_values": {
    "approval_status": "pending"
  },
  "new_values": {
    "approval_status": "approved"
  },
  "url": "/api/admin/forms/form-id/approve",
  "ip_address": "192.168.1.1",
  "user_agent": "Mozilla/5.0...",
  "tags": "forms,resource:template,workflow:approval,outcome:success",
  "created_at": "2026-03-23T10:15:30Z"
}
```

### Logged Events

**Authentication Events:**
- `login_success` - User successful login
- `login_failed` - Login failure
- `logout` - User logout
- `password_changed` - Password update
- `2fa_enabled` - 2FA activation
- `2fa_disabled` - 2FA deactivation

**User Management Events:**
- `user_created` - New user account creation
- `user_updated` - User profile modification
- `user_deleted` - User account deletion
- `user_deactivated` - Account deactivation
- `role_assigned` - Role granted to user
- `role_revoked` - Role removed from user
- `permission_granted` - Permission assigned
- `permission_revoked` - Permission removed

**Form Events:**
- `form_template_created` - Form created
- `form_template_updated` - Form modified
- `form_template_submitted_for_approval` - Form submitted for review
- `form_template_approved` - Admin approved form
- `form_template_rejected` - Admin rejected form
- `form_template_rollback` - Version rolled back

**Data Events:**
- `health_entry_created` - User entered health data
- `health_entry_updated` - Health data modified
- `health_entry_deleted` - Health data deleted
- `provider_patient_record_view` - Provider accessed patient record
- `researcher_cohort_created` - Researcher created cohort
- `researcher_report_generated` - Report created

**Compliance Events:**
- `audit_log_viewed` - Admin reviewed logs
- `compliance_report_generated` - Report created
- `data_export_requested` - User exported data
- `security_alert_triggered` - Security concern detected

### Audit Log Queries

**View all user creation events:**
```bash
Navigate to Audit Log
Filter: Event = "user_created"
Date range: [select range]
```

**Find all actions by specific admin:**
```bash
Navigate to Audit Log
Filter: User = "[admin name]"
View all actions performed by that admin
```

**Track form approval workflow:**
```bash
Navigate to Audit Log
Filter: Event contains "form_template"
View complete form lifecycle
```

**Security audit trail:**
```bash
Navigate to Audit Log
Filter: Event contains "login" OR "password" OR "2fa"
Review authentication security
```

### Audit Log Retention

- **Retention Period:** 7 years (per HIPAA requirements)
- **Storage Location:** Database (audits table)
- **Backup Schedule:** Daily automated backups
- **Access Control:** Admins only (encrypted at rest)

### Sensitive Data Protection

⚠️ **IMPORTANT:** Audit logs NEVER contain:
- Passwords or password hashes
- Full PHI or health data values
- Credit card information
- Social Security numbers
- Raw form responses

**Safe Audit Values:**
```
✓ User ID (not email)
✓ Resource ID (not content)
✓ Action type (not action details)
✓ IP address
✓ Timestamp
✓ Event name
```

---

## DATABASE MANAGEMENT

### Database Overview

- **Engine:** MySQL 8.0
- **Host:** `mysql` (Docker) or configured host
- **Backup:** Daily automated backups
- **Size:** Grows with health entries and forms

### Database Health Monitoring

**Access:** `/admin/dashboard` → Database Management

**Monitoring Checklist:**

- [ ] Check connection status
- [ ] Verify available disk space
- [ ] Monitor query performance
- [ ] Review table sizes
- [ ] Check for corruption
- [ ] Validate backup completion

### Database Operations

#### 1. View Database Status

```bash
php artisan tinker

# Check connection
>>> DB::connection()->getPDO();
>>> DB::select('SELECT VERSION()');  // Shows MySQL version

# Check database size
>>> DB::select("SELECT SUM(data_length + index_length) FROM information_schema.tables WHERE table_schema = 'health_data_bank'");

# Check table count
>>> DB::select("SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'health_data_bank'");
```

#### 2. Backup Operations

**Automated Backups:**

Backups run automatically based on Laravel Backup configuration:

```php
// config/backup.php
'backup' => [
    'enabled' => env('BACKUP_ENABLED', true),
    'name' => env('APP_NAME', 'health-data-bank'),
    
    'source' => [
        'files' => [
            'include' => [base_path()],
            'exclude' => [
                base_path('vendor'),
                base_path('node_modules'),
            ],
        ],
        'databases' => [
            env('DB_CONNECTION', 'mysql'),
        ],
    ],
],
```

**Manual Backup:**

```bash
# Create backup
php artisan backup:run

# Output shows:
# ✓ Database dumped
# ✓ Files collected
# ✓ Backup created
# ✓ Backup verification successful

# List backups
php artisan backup:list

# Output shows all backups with sizes and dates
```

**Backup Location:**
- Local: `storage/backups/` directory
- Cloud: S3, GCS, or configured storage

**Backup Contents:**
- Full MySQL database dump
- Application code
- Configuration files
- User uploads
- Log files

#### 3. Restore from Backup

⚠️ **CRITICAL:** Only perform restores under proper procedures with approval.

**Restore Process:**

```bash
# Step 1: Identify backup to restore
php artisan backup:list
# Identify backup file: health-data-bank-2026-03-20-*.zip

# Step 2: Stop application
docker-compose down
# Prevents concurrent access

# Step 3: Restore database
mysql -h mysql -u root -p health_data_bank < backup.sql
# Restores database tables and data

# Step 4: Restore files (if needed)
unzip backup-2026-03-20-*.zip -d /tmp
cp -r /tmp/health-data-bank/* /var/www/html/
# Restores application files

# Step 5: Clear caches
php artisan cache:clear
php artisan config:clear

# Step 6: Restart application
docker-compose up -d
# Brings application back online

# Step 7: Verify restoration
curl http://localhost/api/patients
# Confirms application operational
```

**Restore Audit Entry:**
```
Event: database_restored
Tags: admin, database, disaster-recovery
Backup Date: [original backup date]
Restore Date: [current date]
Reason: [documented reason]
Admin: [current admin]
Records Affected: [number of records]
```

#### 4. Database Maintenance

**Regular Maintenance Tasks:**

```bash
# Weekly: Optimize tables
php artisan tinker
>>> DB::statement("OPTIMIZE TABLE patients, users, form_templates, health_entries");

# Monthly: Check table integrity
>>> DB::statement("CHECK TABLE patients, users, form_templates, health_entries");

# Quarterly: Full database analysis
php artisan backup:run --only-db
php artisan tinker
>>> DB::select("ANALYZE TABLE *");

# Annually: Archive old records
# Move records older than 3 years to archive database
```

**Maintenance Schedule:**

| Task | Frequency | Time | Duration |
|------|-----------|------|----------|
| Backup | Daily | 2:00 AM | 30 min |
| Optimize | Weekly | Sunday 3:00 AM | 15 min |
| Check Tables | Monthly | 1st of month 3:00 AM | 30 min |
| Full Analysis | Quarterly | 3:00 AM | 1 hour |

#### 5. Data Cleanup

**Purge Expired Sessions:**

```bash
php artisan session:prune-stale-files
# Removes sessions older than configured lifetime
```

**Clear Old Logs:**

```bash
# Delete logs older than 14 days
find storage/logs -type f -mtime +14 -delete

# Or configure in config/logging.php
'days' => env('LOG_DAILY_DAYS', 14),
```

**Archive Old Audit Records:**

```bash
# Keep audit logs for 7 years (HIPAA)
# Archive logs older than 3 years to archive table

php artisan tinker
>>> $cutoff = now()->subYears(3);
>>> $old_audits = DB::table('audits')->where('created_at', '<', $cutoff)->paginate(1000);
>>> // Transfer to archive, then delete
```

---

## REPORT REVIEW & APPROVAL

### Report Types

Users can generate various reports:
- Health summaries
- Trend reports
- Comparative analysis
- Research reports (researchers only)

### Report Review Process

**Access:** `/admin/dashboard` → Report Review

**Review Workflow:**

```
User Generates Report
        ↓
Report queued for review
        ���
Admin reviews report
        ↓
Admin approves OR requests revisions
        ↓
User notified of status
        ↓
Approved report published
```

### Approval Criteria

Reports should be approved if:

✓ Data is accurate
✓ Calculations are correct
✓ No formatting issues
✓ Clear and understandable
✓ HIPAA compliant
✓ Appropriate for intended use

Reports should be rejected if:

❌ Contains identifiable PHI
❌ Contains calculated errors
❌ Insufficient data sample
❌ Unclear methodology
❌ Not appropriate for requester role
❌ Contains sensitive information inappropriately

### Report Approval Actions

```bash
Navigate to: /admin/dashboard → Report Review

For each report:
1. Click "View" to preview content
2. Review for compliance and accuracy
3. Click "Approve" to publish
   OR
   Click "Request Revision" to ask for changes
   - Enter revision request message
   - Provide specific feedback
4. Audit entry automatically created
```

**Approval Audit Entry:**
```
Event: report_approved
Tags: reporting, resource:report, outcome:success
Report ID: [report ID]
Report Type: [type]
Approval Time: [automated]
Admin: [current admin]
```

**Rejection Audit Entry:**
```
Event: report_revision_requested
Tags: reporting, resource:report, feedback:pending
Report ID: [report ID]
Feedback: [revision request message]
Admin: [current admin]
```

---

## SYSTEM MAINTENANCE

### Scheduled Maintenance

**Regular Maintenance Schedule:**

#### Daily Tasks (Automated)
- Database backups
- Log rotation
- Cache cleanup
- Session cleanup

#### Weekly Tasks (Review)
- Database optimization
- Backup verification
- Log review
- Performance check

#### Monthly Tasks (Manual)
- Database integrity check
- Audit log review
- User account audit
- Form approval review

#### Quarterly Tasks (Planning)
- Security assessment
- Compliance review
- Capacity planning
- Architecture review

### Updates & Patches

**Laravel Framework Updates:**

```bash
# Check for updates
composer update --dry-run

# Update dependencies (dev)
composer update

# Rebuild autoloader
composer dump-autoload -o

# Clear caches
php artisan cache:clear
php artisan config:clear

# Run tests
php artisan test

# Deploy to production
# [deployment process]
```

**Database Schema Updates:**

```bash
# Create migration
php artisan make:migration add_new_column_to_users

# Edit migration file
# Up method: add column
# Down method: drop column

# Run migration
php artisan migrate

# Rollback if needed
php artisan migrate:rollback
```

**Docker Updates:**

```bash
# Update Docker images
docker-compose pull

# Rebuild with new images
docker-compose up -d

# Verify services
docker-compose ps
```

### Maintenance Windows

**Planned Maintenance:**

- **Frequency:** Quarterly
- **Duration:** 2-4 hours
- **Notification:** 2 weeks advance notice
- **Communication:** Email to all users
- **Rollback Plan:** Always prepared
- **Testing:** Perform in staging first

**Maintenance Notification Template:**

```
Subject: Scheduled System Maintenance

The Health Data Bank system will be offline for scheduled maintenance:

Date: [Date]
Time: [Start Time] - [End Time] (UTC)
Duration: Approximately [X hours]
Impact: All services will be unavailable
Purpose: [Database optimization / Security updates / Infrastructure upgrades]

During this time:
- Users cannot login
- Forms are not available
- APIs are not accessible
- Data is not affected

We apologize for any inconvenience. For questions, contact support@example.com

Thank you for your patience,
Health Data Bank Administration Team
```

---

## SECURITY & ACCESS PROTOCOLS

### Password Security

**Password Requirements:**
- Minimum 8 characters
- Uppercase letters (A-Z)
- Lowercase letters (a-z)
- Numbers (0-9)
- Special characters (!@#$%^&*)

**Password Management:**
- Change password every 90 days
- Cannot reuse last 3 passwords
- Temporary passwords expire after 24 hours
- Failed login attempts locked after 5 tries

### Two-Factor Authentication

**2FA Enforcement:**
- Mandatory for all users
- Especially critical for admins
- Required on every login
- Cannot be bypassed

**2FA Backup Codes:**
- Generate 10 single-use codes
- Provide during 2FA setup
- Store securely (offline)
- Use only if authenticator app unavailable

### Admin Account Security

**Enhanced Admin Security:**
- Use unique username (not shared)
- Change password monthly
- Enable 2FA immediately
- Review login history weekly
- Disable when not in use
- Never share credentials

**Admin Session Management:**
- Session timeout: 15 minutes
- Automatic logout on inactivity
- Logout on browser close (configurable)
- Single concurrent session

**Sensitive Operations:**
- Require password confirmation
- Require 2FA code re-entry
- Log all sensitive operations
- Alert on unusual access patterns

### API Token Management

**Token Scopes:**
```
read   - Read-only access to resources
create - Create new resources
update - Modify existing resources
delete - Remove resources
```

**Token Security:**
- Generated by user (shown once)
- Store securely (never in code)
- Rotate quarterly
- Revoke if compromised
- Audit all token usage

**Token Revocation:**

```bash
Navigate to: Profile → API Tokens

For each token:
1. View token details (created, last used)
2. Click "Revoke" to disable
3. Token immediately invalid
4. Audit entry created
```

### Access Logging

All admin access is logged:

```
Event: admin_login
Tags: auth, admin, outcome:success
User: [admin ID]
IP Address: [IP]
User Agent: [browser info]
Timestamp: [automated]

Event: sensitive_operation
Tags: admin, action:[operation], outcome:success
User: [admin ID]
Operation: [what was changed]
Old Values: [before]
New Values: [after]
Timestamp: [automated]
```
## NOTIFICATIONS & SUGGESTIONS SECURITY

### Notification Access Control

**Access Rules:**
- Notifications are linked to individual user accounts
- Users can only view their own notifications
- Access to other users’ notifications is restricted
- All access is enforced through authentication and authorization middleware

**Authorized Actions:**
- View notification details
- Mark notifications as read
- Access related pages through notification links

**Restrictions:**
- Unauthorized access attempts are blocked
- Cross-user data access is not permitted
- All actions require authenticated sessions



### Privacy and Compliance

**Data Protection:**
- Notifications do not expose sensitive health data unnecessarily
- Only relevant and minimal information is included in notifications
- Sensitive values are not directly displayed in notification content

**Suggestion Data Handling:**
- Suggestions are generated using aggregated and analyzed data
- Individual sensitive data is not exposed in raw form
- Suggestions are only created when sufficient data is available

**Compliance Measures:**
- All notification and suggestion interactions are logged
- Audit logs capture access, actions, and system events
- Role-based access control (RBAC) is enforced across all features
- System follows established privacy and security policies



### Audit and Monitoring

**Audit Logging:**
- All notification creation events are logged
- User interactions (view, read, action) are recorded
- Suggestion generation events are tracked

**Monitoring:**
- Unauthorized access attempts are logged and monitored
- System activity related to notifications and suggestions is auditable
- Logs can be reviewed for compliance and debugging purposes


## DISASTER RECOVERY & BACKUPS

### Backup Strategy (Recommended Setup)

**3-2-1 Backup Rule:**
- **3** copies of data
- **2** different storage media
- **1** copy offsite

**Health Data Bank Backup:**

| Copy | Location | Frequency | Retention |
|------|----------|-----------|-----------|
| 1 | Local server | Daily | 30 days | (IMPLEMENTED)
| 2 | S3 (offsite) | Daily | 90 days | (NOT IMPLEMENTED)
| 3 | Tape archive | Weekly | 7 years | (NOT IMPLEMENTED)

### Disaster Recovery Plan

**Recovery Time Objective (RTO):** 4 hours
**Recovery Point Objective (RPO):** 24 hours

#### Scenario 1: Database Corruption

**Detection:**
```
Symptom: Users report errors or data inconsistency
Detection: Automated integrity check fails
Severity: High
```

**Recovery Steps:**

```
1. Notify stakeholders (15 min)
   - Send incident notification
   - Document incident time
   
2. Assess damage (15 min)
   - Identify affected tables
   - Determine data loss scope
   - Check backup availability
   
3. Restore from backup (30 min)
   - Restore from most recent clean backup
   - Verify data integrity
   - Check application functionality
   
4. Validate restoration (30 min)
   - Test all features
   - Verify user data
   - Confirm no data loss beyond expected
   
5. Resume operations (15 min)
   - Enable user access
   - Monitor for issues
   - Generate incident report
   
6. Post-incident (ongoing)
   - Analyze root cause
   - Implement preventive measures
   - Update documentation
```

**Expected Recovery Time:** 2 hours

#### Scenario 2: Server Failure

**Detection:**
```
Symptom: Application unavailable
Detection: Health check fails, 503 errors
Severity: Critical
```

**Recovery Steps:**

```
1. Incident response (10 min)
   - Declare incident
   - Activate disaster recovery team
   - Notify all stakeholders
   
2. Provision new infrastructure (30 min)
   - Spin up new server instance
   - Configure network settings
   - Install required software
   
3. Restore data (20 min)
   - Download latest backup from S3
   - Restore database
   - Restore application files
   
4. Validation (15 min)
   - Verify services operational
   - Test critical functions
   - Confirm data consistency
   
5. DNS failover (5 min)
   - Update DNS to point to new server
   - Wait for propagation (can be instant with correct setup)
   
6. Operations monitoring (ongoing)
   - Monitor system health
   - Verify user access restored
   - Generate incident report
```

**Expected Recovery Time:** 1-2 hours

#### Scenario 3: Data Breach

**Detection:**
```
Symptom: Unauthorized access detected
Detection: Audit log shows suspicious activity
Severity: Critical
```

**Response Steps:**

```
1. Immediate containment (5 min)
   - Identify compromised accounts
   - Revoke affected sessions
   - Disable compromised credentials
   
2. Investigation (ongoing)
   - Review audit logs for unauthorized access
   - Determine scope of breach
   - Identify affected records
   - Preserve evidence
   
3. Communication (15 min)
   - Notify affected users
   - Notify compliance officer
   - Contact legal counsel
   - Prepare breach notification
   
4. Remediation (1-2 hours)
   - Reset affected user passwords
   - Invalidate compromised tokens
   - Apply security patches
   - Update access controls
   
5. Notification (per legal requirements)
   - Send breach notification to affected parties
   - File incident report with regulators
   - Document in compliance file
   
6. Follow-up (ongoing)
   - Conduct security audit
   - Implement preventive measures
   - Monitor for continued suspicious activity
   - Provide user support/assistance
```

**Expected Response Time:** Depends on breach severity

### Backup Verification

**Monthly Verification:**

```bash
# Test restore process in staging environment

1. Download backup from S3
2. Restore to staging database
3. Verify data integrity
   - Check record counts match
   - Verify recent entries present
   - Test sample queries
4. Confirm application functionality
   - Login successful
   - Data accessible
   - Forms functional
5. Document verification
   - Date tested
   - Backup tested
   - Results recorded
```

**Verification Checklist:**

- [ ] Backup file accessible
- [ ] Backup file size reasonable
- [ ] Database restore successful
- [ ] Record counts match
- [ ] Recent data present
- [ ] Application starts
- [ ] Users can login
- [ ] Data queries work
- [ ] No corruption errors
- [ ] All tables present
- [ ] Indexes intact
- [ ] Audit logs complete

---

## ADMINISTRATOR WORKFLOWS

### Daily Workflow

**Morning (Start of Day):**

```
1. Review overnight logs (15 min)
   - Check for errors in application logs
   - Verify backup completion
   - Review security alerts
   
2. Check system health (10 min)
   - Verify database connectivity
   - Confirm all services running
   - Check disk space available
   
3. Review user accounts (10 min)
   - Check for new account requests
   - Verify no unauthorized access
   - Monitor failed login attempts
```

**Afternoon (Mid-Day):**

```
4. Review pending forms (30 min)
   - Check approval queue
   - Approve quality forms
   - Reject forms with issues
   - Provide feedback to creators
   
5. Monitor activities (15 min)
   - Check for error spikes
   - Monitor user activity patterns
   - Review API token usage
```

**End of Day:**

```
6. Final checks (10 min)
   - Confirm backup completed
   - Verify all services operational
   - Document any issues
   - Plan next day priorities
```

### Weekly Workflow

**Monday Morning:**

```
1. Audit log review (30 min)
   - Search for suspicious activity
   - Review user access patterns
   - Check for failed authentication attempts
   - Verify admin actions logged
```

**Wednesday:**

```
2. Database optimization (30 min)
   - Run table optimization
   - Review database size growth
   - Check for slow queries
   - Plan maintenance if needed
```

**Friday:**

```
3. Compliance review (45 min)
   - Verify 2FA enabled for all admins
   - Check for unused accounts
   - Review user role assignments
   - Confirm audit logging complete
   
4. Weekend preparation (15 min)
   - Verify backups will run
   - Confirm monitoring active
   - Document any known issues
   - Provide contact info if needed
```

### Monthly Workflow

```
1. Database integrity check (1 hour)
   - Run table consistency checks
   - Verify backup restorability
   - Check for orphaned records
   
2. Audit log analysis (1 hour)
   - Generate monthly compliance report
   - Identify access patterns
   - Review unusual activities
   - Document findings
   
3. User account audit (45 min)
   - Verify all accounts legitimate
   - Check for deactivated accounts
   - Review role assignments
   - Identify unused accounts
   
4. Security review (45 min)
   - Check for failed logins
   - Verify 2FA enabled
   - Review API token usage
   - Check for password age
   
5. Compliance documentation (30 min)
   - Update change log
   - Document system changes
   - Archive old logs
   - Prepare compliance artifacts
```

### Quarterly Workflow

```
1. Security assessment (2-4 hours)
   - Review access controls
   - Audit permissions
   - Test backup/restore
   - Verify encryption
   - Check SSL certificates
   
2. Compliance audit (2-3 hours)
   - Generate 3-month compliance report
   - Verify audit trail completeness
   - Check data retention policies
   - Review incident logs
   
3. Capacity planning (1-2 hours)
   - Analyze database growth rate
   - Project future storage needs
   - Review performance metrics
   - Plan infrastructure upgrades
   
4. System review (1-2 hours)
   - Review architecture
   - Plan major updates
   - Assess new features
   - Plan security improvements
```

---

## TROUBLESHOOTING & SUPPORT

### Common Admin Issues

#### Issue 1: User Cannot Login

**Symptoms:**
- User reports "Invalid credentials"
- User locked out after 5 attempts
- Email address not recognized

**Resolution:**

```bash
# Step 1: Verify user exists
php artisan tinker
>>> User::where('email', 'user@example.com')->first();
// Returns user record or null

# Step 2: If user doesn't exist, create account
>>> User::factory()->create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
]);

# Step 3: If user exists, check account status
>>> $user = User::where('email', 'user@example.com')->first();
>>> $user->account;  // Check associated account

# Step 4: If locked out, clear login attempts
>>> Cache::forget('login_attempts:' . $email);

# Step 5: Reset password if needed
>>> $user->forceFill(['password' => Hash::make('temporary_password')])->save();
// Send user the temporary password via secure channel
```

#### Issue 2: Form Not Appearing in User Dashboard

**Symptoms:**
- Form created but not visible to users
- Users can't access specific form
- Form seems to have disappeared

**Resolution:**

```bash
# Step 1: Check form status
>>> $form = FormTemplate::where('title', 'Form Name')->first();
>>> $form->approval_status;  // Should be "approved"

# Step 2: If pending, approve form
>>> $form->approval_status = 'approved';
>>> $form->save();

# Step 3: Clear form cache
>>> Cache::forget('forms:approved');

# Step 4: Verify form is published
>>> $form->published_at;  // Should not be null

# Step 5: Check user role permissions
>>> $user->hasPermissionTo('fill_forms');
```

#### Issue 3: Backup Not Running

**Symptoms:**
- No backup files created
- Backup time passes without completion
- Error in backup logs

**Resolution:**

```bash
# Step 1: Verify backup is enabled
>>> config('backup.backup.enabled');  // Should be true

# Step 2: Check backup directory permissions
chmod -R 755 storage/backups
chown -R www-data:www-data storage/backups

# Step 3: Run backup manually
php artisan backup:run

# Step 4: Check backup logs
tail -f storage/logs/laravel.log | grep backup

# Step 5: Verify disk space available
df -h | grep /var/www/html

# Step 6: If disk full, archive old backups
aws s3 mv storage/backups/ s3://backup-bucket/archive/
rm -rf storage/backups/*
```

#### Issue 4: Slow Database Performance

**Symptoms:**
- Queries taking longer than normal
- Users report lag when submitting forms
- API responses slow
- High server CPU/memory

**Resolution:**

```bash
# Step 1: Check slow query log
SHOW VARIABLES LIKE 'slow_query%';
SELECT * FROM mysql.slow_log;

# Step 2: Optimize tables
php artisan tinker
>>> DB::statement("OPTIMIZE TABLE users, form_templates, health_entries");

# Step 3: Rebuild indexes
>>> DB::statement("REPAIR TABLE users, form_templates, health_entries");

# Step 4: Check table statistics
>>> DB::statement("ANALYZE TABLE users, form_templates, health_entries");

# Step 5: Review running queries
>>> DB::select("SHOW PROCESSLIST");

# Step 6: Kill long-running queries if needed
>>> DB::statement("KILL QUERY [process_id]");

# Step 7: Clear application cache
php artisan cache:clear
php artisan config:clear
```

#### Issue 5: High Memory Usage

**Symptoms:**
- Server running out of memory
- PHP processes consuming excess RAM
- Application crashes
- OOM (Out of Memory) errors

**Resolution:**

```bash
# Step 1: Monitor memory
docker stats
# or
free -h
top

# Step 2: Check Laravel memory limit
>>> ini_get('memory_limit');  // Should be 512M or higher

# Step 3: Clear caches
php artisan cache:clear
php artisan view:clear
php artisan route:clear

# Step 4: Check for memory leaks in code
php artisan tinker
>>> ini_set('memory_limit', '2048M');  // Increase for analysis

# Step 5: Restart services
docker-compose restart laravel.test

# Step 6: Consider queue processing
# Move long-running operations to queues
php artisan queue:work --timeout=3600
```

### Getting Help

**Internal Support:**
- Contact IT team
- Submit support ticket
- Escalate to senior admin

**Documentation:**
- Review admin documentation
- Check system logs
- Search knowledge base

**Emergency Support:**
- 24/7 on-call admin
- Emergency contact list
- Incident response procedure

---

## CHANGE LOG & AUDITING

### Change Log

All system changes must be documented:

**Format for Change Log:**

```
Date: 2026-03-23
Type: User Management
Change: Approved form template "Weekly Health Check"
Admin: admin@example.com
Status: Completed
Notes: Form reviewed and approved for production use
Audit Entry: form_template_approved [ID]
Impact: Form now available to all users
```

**Change Log Categories:**

| Category | Example |
|----------|---------|
| User Management | Created/modified/deleted user account |
| Form Governance | Approved/rejected/rolled back form |
| Database | Backup/restore/optimization |
| Security | Password reset, 2FA configuration |
| System | Updates, patches, maintenance |
| Compliance | Audit, reports, policy changes |

### Audit Report Generation

**Generate Monthly Compliance Report:**

```bash
Navigate to: /admin/audit-log

1. Filter by date range (entire month)
2. Review all logged events
3. Export to CSV
4. Verify completeness
5. Archive report
6. Sign off on completeness
```

**Compliance Report Contents:**

```
Health Data Bank Compliance Report
Period: [Month, Year]
Generated: [Date]
Prepared By: [Admin Name]

Executive Summary
- Total audit entries: [number]
- Critical events: [number]
- User accounts created: [number]
- Forms approved: [number]
- Failed logins: [number]
- Data access attempts: [number]

Key Events
[List of significant events]

Security Assessment
- 2FA enabled for: [%] users
- Password compliance: [assessment]
- Access control: [assessment]
- Data protection: [assessment]

Findings & Recommendations
[Any concerns or recommendations]

Certification
I certify that this report accurately reflects system governance
during the reported period.

Signed: _________________
Name: [Admin Name]
Date: [Date]
```

---

## APPENDIX: QUICK REFERENCE

### Quick Links

| Task | Link | Role |
|------|------|------|
| Admin Dashboard | `/admin/dashboard` | Admin |
| User Management | `/admin/users` | Admin |
| Form Review | `/admin/forms` | Admin |
| Audit Log | `/admin/audit-log` | Admin |
| Database Management | `/admin/database` | Admin |
| Profile Settings | `/admin/profile` | Admin |

### Critical Commands

```bash
# Check system status
docker-compose ps
php artisan tinker

# View logs
tail -f storage/logs/laravel.log

# Create backup
php artisan backup:run

# Clear caches
php artisan cache:clear
php artisan config:clear

# Optimize database
php artisan tinker
>>> DB::statement("OPTIMIZE TABLE patients, users, form_templates");

# Restart services
docker-compose restart laravel.test
```

### Admin Checklist

Daily:
- [ ] Review logs
- [ ] Check system health
- [ ] Review pending forms
- [ ] Monitor errors

Weekly:
- [ ] Audit log review
- [ ] Database optimization
- [ ] Compliance check

Monthly:
- [ ] Database integrity check
- [ ] User account audit
- [ ] Security review
- [ ] Generate compliance report
