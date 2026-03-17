
# Controlled Database Restore Procedure for Admin/Governance

## Purpose

This document defines the controlled procedures for restoring the Health-Data Bank database from backups. The restore process is critical for disaster recovery, compliance requirements, and business continuity. All restore operations must be audited, documented, and performed by authorized administrators following strict governance controls.

---

## Overview

Database restore procedures are essential for:
- **Disaster Recovery**: Recovering from data loss or system failures
- **Compliance & Governance**: Meeting regulatory requirements (HIPAA, GDPR, CCPA, PHIPA)
- **Audit Trail Continuity**: Maintaining verifiable records of all restore operations
- **Data Integrity**: Ensuring restored data is consistent and trustworthy
- **PHI Protection**: Safeguarding sensitive health information during restore operations

---

## Authorization & Role Requirements

### Authorized Roles

Only administrators with the following roles are authorized to perform restore operations:

- **System Administrator** (role: `system_admin`): Full restore authority
- **Database Administrator** (role: `db_admin`): Database restore authority
- **Governance Officer** (role: `governance_officer`): Approval and oversight authority

### Authorization Controls

```php
// Route protection example
Route::middleware(['auth:sanctum', 'role:system_admin|db_admin'])->group(function () {
    Route::post('/admin/database/restore', [DatabaseRestoreController::class, 'restore']);
    Route::get('/admin/database/restore-status', [DatabaseRestoreController::class, 'status']);
    Route::get('/admin/database/restore-history', [DatabaseRestoreController::class, 'history']);
});
```

### Access Restrictions

- Non-administrators cannot initiate, approve, or execute restore operations
- Restore operations must not be performed in production without explicit approval
- All restore attempts (successful and failed) are logged in audit trails

---

## Backup Infrastructure

### Backup Configuration

**Location**: `config/backup.php`

**Key Settings**:
- **Database Connection**: MySQL/PostgreSQL via environment variable `DB_CONNECTION`
- **Backup Compression**: GZip compression with compression level 9
- **Backup Retention Strategy**:
  - Keep all backups for 7 days
  - Keep daily backups for 16 days
  - Keep weekly backups for 8 weeks
  - Keep monthly backups for 4 months
  - Keep yearly backups for 2 years

**Backup Locations**:
```
storage/app/backups/  (Primary local storage)
storage/app/backup-temp/  (Temporary working directory)
```

**Backup Archive**:
- Format: ZIP with AES-256 encryption
- Password: `env('BACKUP_ARCHIVE_PASSWORD')` from environment
- Filename pattern: `{database}_{timestamp}.zip`

**Health Monitoring**:
- Maximum age allowed: 1 day
- Maximum storage size: 5000 MB
- Alerts triggered if these thresholds are exceeded

### Database Dump Configuration

The system uses Spatie's database dumping library with the following configuration:

```php
'databases' => [
    env('DB_CONNECTION', 'mysql'),
],

'database_dump_compressor' => null,  // Compression handled at ZIP level
'database_dump_filename_base' => 'database',
'database_dump_file_extension' => 'sql',
```

**For MySQL Backups**:
- Uses `mysqldump` utility
- Can be configured with `useSingleTransaction: true` for InnoDB to avoid table locking
- Automatically excludes specified tables (if configured in `config/database.php`)

**Excluded Tables** (if configured):
- Session tables (handled by Laravel middleware)
- Cache tables (rebuilt from cache)
- Temporary working tables

---

## Pre-Restore Validation

### System Health Check

Before initiating any restore operation, administrators must verify:

1. **Backup File Integrity**
   - Backup file exists and is accessible
   - File size is within expected range
   - Checksum verification (if available)
   - Decryption key is available

2. **Database Status**
   - Current database is accessible
   - Sufficient disk space available for restore operations
   - No active migrations running
   - No long-running queries blocking restoration

3. **System Resources**
   - Memory available for database operations
   - CPU not under excessive load
   - Network connectivity stable
   - Backup storage accessible

### Pre-Restore Checklist

```
□ Backup file located and verified
□ Backup encryption key available
□ Database connection credentials verified
□ Backup timestamp documented
□ Current database state backed up (if restoring over live data)
□ Maintenance mode enabled (recommended)
□ All active users notified of downtime
□ Governance approval obtained
□ Audit trail ready (timestamp, admin ID recorded)
```

---

## Restore Procedures

### Procedure 1: Full Database Restore (Production)

**Approval Required**: Governance Officer + System Administrator

**Timeline**: 
- Approval: 24 hours minimum notice
- Execution: Off-peak hours only
- Estimated Duration: 30-60 minutes depending on database size

**Steps**:

1. **Request & Approval Phase**
   ```
   - Admin submits restore request via admin interface
   - Request includes: backup timestamp, reason, target environment
   - Governance Officer reviews and approves/rejects
   - Approval logged with timestamp and approver ID
   ```

2. **Pre-Restore Validation**
   ```
   docker-compose -f compose.yaml exec laravel.test php artisan backup:verify {backup-file}
   ```

3. **Maintenance Mode Activation**
   ```
   docker-compose -f compose.yaml exec laravel.test php artisan down --message="Database restore in progress"
   ```

4. **Current Database Backup** (Safety precaution)
   ```
   docker-compose -f compose.yaml exec laravel.test php artisan backup:run
   ```

5. **Decrypt & Extract Backup**
   ```bash
   # Extract encrypted backup (password from BACKUP_ARCHIVE_PASSWORD env)
   unzip -P $BACKUP_ARCHIVE_PASSWORD storage/app/backups/{backup-file}.zip -d storage/app/backup-temp/
   ```

6. **Database Restore**
   ```bash
   # For MySQL
   docker-compose -f compose.yaml exec -T mysql mysql -u $DB_USERNAME -p$DB_PASSWORD $DB_DATABASE < storage/app/backup-temp/database.sql
   ```

7. **Post-Restore Verification**
   ```
   docker-compose -f compose.yaml exec laravel.test php artisan migrate --force
   docker-compose -f compose.yaml exec laravel.test php artisan db:check
   ```

8. **Audit Logs Restoration Check**
   ```
   # Verify audit logs are intact
   docker-compose -f compose.yaml exec laravel.test php artisan db:query "SELECT COUNT(*) FROM audit_logs;"
   ```

9. **Maintenance Mode Deactivation**
   ```
   docker-compose -f compose.yaml exec laravel.test php artisan up
   ```

10. **Post-Restore Testing**
    ```
    - Run health check API endpoints
    - Verify user authentication
    - Test data access permissions
    - Confirm audit logging functionality
    ```

11. **Audit Logging**
    ```php
    AuditLogger::log(
        'database_restore_completed',
        ['admin', 'database', 'outcome:success'],
        null,
        ['backup_file' => $backupFile],
        ['restored_at' => now(), 'restored_by' => $adminId]
    );
    ```

### Procedure 2: Point-in-Time Recovery

**Approval Required**: Database Administrator + Governance Officer

**Use Cases**:
- Recovering from accidental data deletion (within backup window)
- Reverting to state before unintended changes
- Dispute resolution investigations

**Steps**:

1. **Identify Target Timestamp**
   - Review audit logs to determine exact time of incident
   - Verify backup exists for that timestamp
   - Calculate data loss window

2. **Backup Current State**
   - Perform full backup before proceeding
   - Store with incident reference label

3. **Execute Restore**
   - Follow steps 3-10 from Full Database Restore procedure
   - Use backup closest to target timestamp

4. **Verify Data State**
   - Query audit logs to confirm restore point
   - Spot-check critical data records
   - Verify user accounts and permissions

5. **Document Recovery**
   ```php
   AuditLogger::log(
        'database_point_in_time_recovery',
        ['admin', 'database', 'recovery', 'outcome:success'],
        null,
        ['incident_timestamp' => $incidentTime],
        ['restored_to_timestamp' => $restorePoint]
    );
    ```

### Procedure 3: Partial/Table-Specific Restore

**Approval Required**: Database Administrator

**Use Cases**:
- Corrupted single table
- Specific data category restore
- Targeted data reconstruction

**Steps**:

1. **Identify Affected Table**
   - Determine which table(s) need restoration
   - Assess impact scope
   - Verify backup contains clean copy

2. **Extract Backup**
   - Decrypt and extract backup file
   - Isolate table-specific SQL dump

3. **Pre-Restore Backup**
   - Backup current corrupted table
   - Store with reference ID

4. **Execute Selective Restore**
   ```bash
   # Drop corrupted table
   docker-compose -f compose.yaml exec -T mysql mysql -u $DB_USERNAME -p$DB_PASSWORD -e "DROP TABLE $DB_DATABASE.corrupted_table;"
   
   # Restore from backup SQL (selective)
   docker-compose -f compose.yaml exec -T mysql mysql -u $DB_USERNAME -p$DB_PASSWORD $DB_DATABASE < storage/app/backup-temp/database_table_extract.sql
   ```

5. **Verify Relationships**
   - Check foreign key constraints
   - Validate referential integrity
   - Confirm no orphaned records

6. **Update Audit Trail**
   ```php
   AuditLogger::log(
        'database_table_restore',
        ['admin', 'database', 'table_restore'],
        null,
        ['table' => $tableName],
        ['restored_rows' => $rowCount]
    );
   ```

---

## Disaster Recovery Scenarios

### Scenario 1: Complete Data Loss

**Recovery Time Objective (RTO)**: 2 hours
**Recovery Point Objective (RPO)**: 24 hours

**Steps**:
1. Verify backup integrity
2. Follow Full Database Restore procedure
3. Run application tests to confirm functionality
4. Notify stakeholders of recovery completion

### Scenario 2: Corrupted Database

**Recovery Time Objective (RTO)**: 1 hour
**Recovery Point Objective (RPO)**: 24 hours

**Steps**:
1. Enable maintenance mode
2. Attempt database repair: `docker-compose -f compose.yaml exec laravel.test php artisan db:repair`
3. If repair fails, follow Full Database Restore procedure
4. Run integrity checks post-restore

### Scenario 3: Ransomware/Security Incident

**Recovery Time Objective (RTO)**: 4 hours
**Recovery Point Objective (RPO)**: Last verified clean backup (2+ days old)

**Critical Steps**:
1. Isolate affected systems immediately
2. Do NOT attempt to restore until incident is analyzed
3. Restore from clean backup verified before incident date
4. Run comprehensive security audit post-restoration
5. Implement incident response controls

---

## Audit & Compliance

### Audit Logging for Restore Operations

All restore operations must be audited with the following minimum information:

**Audit Event**: `database_restore_initiated`
- Actor: Administrator ID and role
- Action: Restore initiation
- Target: Backup file identifier
- Metadata: Restore reason, target timestamp
- Outcome: Pending/Initiated

**Audit Event**: `database_restore_completed`
- Actor: Administrator ID and role
- Action: Restore completion
- Target: Database identifier
- Metadata: Duration, rows affected, verification status
- Outcome: Success/Failure

**Audit Event**: `database_restore_failed`
- Actor: Administrator ID and role
- Action: Restore failure
- Target: Backup file identifier
- Metadata: Error details, failure reason
- Outcome: Failure

### Restore Operation Logging

```php
// Example audit logging for restore
class DatabaseRestoreService {
    
    public function restore($backupFile, $adminId) {
        // Log restore initiation
        AuditLogger::log(
            'database_restore_initiated',
            ['admin', 'database', 'restore'],
            null,
            ['backup_file' => $backupFile],
            ['initiated_by' => $adminId]
        );
        
        try {
            // Perform restore...
            $result = $this->performRestore($backupFile);
            
            // Log successful completion
            AuditLogger::log(
                'database_restore_completed',
                ['admin', 'database', 'restore', 'outcome:success'],
                null,
                ['backup_file' => $backupFile],
                [
                    'duration' => $result['duration'],
                    'rows_affected' => $result['rows'],
                    'completed_by' => $adminId
                ]
            );
            
        } catch (Exception $e) {
            // Log failure
            AuditLogger::log(
                'database_restore_failed',
                ['admin', 'database', 'restore', 'outcome:failure'],
                null,
                ['backup_file' => $backupFile],
                [
                    'error' => $e->getMessage(),
                    'failed_at' => now(),
                    'attempted_by' => $adminId
                ]
            );
            throw $e;
        }
    }
}
```

### Retention & Compliance

**Audit Log Retention**:
- Restore operation logs retained for 2 years minimum
- Required for compliance investigations
- Cannot be modified or deleted without audit trail

**Compliance Framework**:

**HIPAA Compliance**:
- 45 CFR § 164.308(a)(3)(ii)(B): Disaster Recovery Plan
- 45 CFR § 164.312(b): Audit controls for restore operations
- Restore procedures documented and tested annually

**GDPR Compliance**:
- Article 32: Security measures including backup and restore
- Article 33-34: Breach notification (if restore required due to incident)

**CCPA Compliance**:
- Section 1798.150: Data minimization during restore
- Right to delete: Honored before restore if requested

**PHIPA Compliance**:
- Section 11: Individual access to health information
- Restore operations support access request fulfillment

---

## Testing & Validation

### Annual Restore Testing

**Requirement**: At least one full restore test annually

**Test Checklist**:
```
□ Test environment prepared
□ Backup file extracted successfully
□ Database restored completely
□ All tables present with correct record counts
□ Foreign key relationships intact
□ Audit logs preserved
□ User authentication working
□ Authorization roles functional
□ API endpoints responding correctly
□ Performance benchmarks acceptable
□ Restore time documented
```

### Post-Restore Verification

```php
// Verification script
php artisan db:check  // Laravel built-in database check

// Manual verification
- SELECT COUNT(*) FROM accounts;  // Verify user count
- SELECT COUNT(*) FROM audit_logs;  // Verify audit trail
- SELECT MAX(created_at) FROM audit_logs;  // Latest audit entry
- SHOW TABLE STATUS;  // Table integrity check
```

---

## Disaster Recovery Plan

### Recovery Time Objectives (RTO)

| Scenario | RTO | Notes |
|----------|-----|-------|
| Full data loss | 2 hours | Complete database restore |
| Table corruption | 1 hour | Selective table restore |
| Ransomware/Incident | 4 hours | Additional security review required |
| Backup file corruption | 6 hours | Attempt restore from multiple backups |

### Recovery Point Objectives (RPO)

| Scenario | RPO | Backup Frequency |
|----------|-----|-------------------|
| Normal Operations | 24 hours | Daily automated backups |
| Critical Infrastructure | 6 hours | Consider increased frequency |
| Post-Incident | Last verified clean | Varies by incident date |

### Backup Verification Schedule

- **Weekly**: Verify backup file integrity (5% sampling)
- **Monthly**: Test restore procedure on non-production environment
- **Quarterly**: Full disaster recovery drill
- **Annually**: Complete restore test with stakeholder sign-off

---

## Incident Response

### Data Breach Restore Scenario

If restore is needed due to data breach:

1. **Immediate Actions**
   - Enable maintenance mode
   - Notify Governance Officer
   - Isolate affected systems
   - Preserve evidence (do not overwrite logs)

2. **Investigation Phase**
   - Review audit logs for breach timeline
   - Determine data exposure scope
   - Document timeline of events

3. **Restore Execution**
   - Choose restore point before breach detection
   - Perform restore with additional verification
   - Run forensic checks post-restore

4. **Post-Restore Obligations**
   - Notify affected individuals (GDPR/CCPA)
   - Report to regulatory bodies (HIPAA)
   - Document incident response
   - Update security controls

---

## Documentation Requirements

### For Each Restore Operation

**Must Document**:
- Date and time of restore
- Administrator performing restore
- Backup file used (timestamp, filename)
- Reason for restore
- Pre-restore system state
- Duration of restore operation
- Post-restore verification results
- Any issues encountered and resolutions
- Stakeholder notifications sent

**Documentation Template**:
```
RESTORE OPERATION LOG
====================
Date/Time: [timestamp]
Restoring Administrator: [name, ID, role]
Backup File: [filename, size, checksum]
Restore Duration: [X minutes]
Records Affected: [count]
Reason: [business reason]
Pre-Restore Backup: [backup file reference]
Verification Status: [PASSED/FAILED]
Issues Encountered: [none or detailed list]
Incident Reference: [ticket/case ID if applicable]
Governance Approval: [approver ID, approval timestamp]
```

---

## Security Considerations

### Backup Encryption

- All backups encrypted with AES-256
- Encryption password stored in environment variables
- Password rotated quarterly
- Access to `BACKUP_ARCHIVE_PASSWORD` restricted to system administrators

### Access Control

- Restore operations restricted to authorized roles
- Multi-factor authentication required for production restores
- Restore requests logged with full accountability chain
- Session recording enabled during restore operations

### Data Integrity

- Checksums verified before and after restore
- Foreign key constraints checked post-restore
- Data consistency checks run automatically
- Audit logs verified for tampering

---

## Troubleshooting Guide

### Issue: Backup File Corrupted

**Symptoms**: Unzip fails, checksum mismatch

**Resolution**:
1. Attempt restore from previous day's backup
2. If multiple backups corrupted, assess backup process
3. Check backup storage disk health
4. Escalate to infrastructure team

### Issue: Insufficient Disk Space

**Symptoms**: Restore fails partway through

**Resolution**:
1. Free up disk space (remove old backups, clear temp files)
2. Retry restore operation
3. Monitor disk usage patterns
4. Increase storage capacity if needed

### Issue: Database Locking

**Symptoms**: Restore operation hangs

**Resolution**:
1. Enable single-transaction mode: `useSingleTransaction: true`
2. Kill long-running queries (if safe)
3. Restart MySQL service (as last resort)
4. Retry restore

### Issue: Audit Log Mismatch

**Symptoms**: Audit logs don't align with restored data

**Resolution**:
1. Verify audit logs restored correctly
2. Check for data consistency
3. Document discrepancy
4. Escalate to governance for review

---

## Glossary

**RTO**: Recovery Time Objective - Maximum acceptable downtime
**RPO**: Recovery Point Objective - Maximum acceptable data loss window
**PHI**: Protected Health Information
**Maintenance Mode**: Read-only state preventing write operations during restore
**Audit Trail**: Immutable record of all system actions
**Checksum**: Hash verification for data integrity
**AES-256**: Advanced Encryption Standard with 256-bit key strength

---

## Related Documentation

- Data Retention & Audit Requirements
- Provider & Admin Workflow Documentation
- Deployment and Hosting Options
- Database Architecture Guide
