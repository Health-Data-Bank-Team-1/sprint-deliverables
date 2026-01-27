**1. Data Retention & Deletion Rules**

The system manages personal and health-related data using the principle of "retain only as long as necessary." This implies that data is only retained for as long as it is necessary to meet the operational goals of the system, such as health tracking, reporting, and care assistance, or to guarantee accountability and dispute resolution. This strategy is in line with standard privacy guidelines, which place a strong emphasis on reducing data retention wherever feasible.

Retention and deletion policies can only be started or overridden by administrators through controlled system activities; they are automatically implemented by the system through specified workflows and background operations.


**Retention Categories**

- **Account and profile data (all roles)**

Information about the account and profile is kept for as long as the account is active.
An account is soft-deactivated, that is, its status is converted to inactive when a user deactivates it.
This instantly stops additional access while retaining system integrity, including historical accountability and audit trail continuity.

- **User-submitted health data (forms, submissions, entries)**

Health data submitted by users is retained while the user's account is active in order to support dashboards, reporting, and healthcare provider review.
When a user deactivates their account, no more data collection or provider access is allowed, and access to their health data is prohibited.
After deactivation, the data is put into an inactive state and kept for up to a year in order to fulfill operational requirements like handling disputes or access requests.
Following this time frame, the data is either anonymized or safely erased in compliance with the retention policy.
This approach complies with PHIPA recommendations, which recommend maintaining records long enough for individuals to pursue suitable legal remedies, even though there isn't a set retention period.

- **Aggregated and Anonymized Outputs (Cohort Summaries and Community Reports)**

If all privacy regulations, such as minimum cohort size and suppression of minor counts are followed, aggregated and anonymized outputs may be kept longer than raw health data.

- **Research Artifacts (Reports, Versions, and Exports)**

To facilitate traceability and repeatability, created reports and report versions are kept, enabling the system to describe what was generated, when it was generated, and under what circumstances.
Exports in CSV format are regarded as sensitive outputs. The audit log is continually updated with export events. Instead of depending on long-term stored files, researchers must renew exports when necessary because CSV files are generated on demand and are not kept permanently.

- **Governance Records (Audit Logs)**

Audit logs are retained longer than standard operational data because they support system security, accountability, and investigation activities. Audit records are retained for two years, after which they may be archived or securely purged if they are no longer required for administrative, legal, or audit purposes.


**Deletion and Disposal Rules**

**Secure Deletion**

Records are securely deleted so that reconstruction is not reasonably foreseeable when they approach the end of their retention period or when test or invalid data must be eliminated. This complies with PHIPA regulations regarding the safe disposal of personal health data.

**Deletion Triggers (Examples)**

- User account deactivation: The account is immediately set to inactive, access is halted, and the retention countdown begins for any scheduled deletion or anonymization actions.
- Administrator deletion of a problematic report: The report is removed from active access while preserving a minimal audit record documenting the action.
- Expired retention period: Eligible data is securely deleted or anonymized according to the retention policy.

Where appropriate, anonymization may be used instead of deletion to preserve statistical value while eliminating the risk of individual identification.


**2. Audit Logging Requirements and Retention Rules**

To guarantee accountability, openness, and security across the system, audit logging is used. Administrators can monitor usage, look into events, and show that sensitive data is handled responsibly thanks to the audit log, which offers a trustworthy record of important system actions.

Audit logging is automatically enforced by the system and cannot be disabled by users. Because all audit records are append-only and unchangeable, historical records cannot be changed or deleted without detection.

**Auditable Events**

For operations involving sensitive data access, data manipulation, reporting, or system configuration, the system keeps track of audit entries. The following occurrences are at the very least recorded:

- Authentication events, including successful logins, failed login attempts, and logout actions
- Account lifecycle events, such as account creation, role assignment, and account deactivation
- Submission of user health data through forms
- Generation, viewing, and export of aggregated reports
- Creation, modification, or deletion of cohorts
- Administrative actions, including report moderation, backup and restore operations, and configuration changes

**Audit Log Contents**

Each audit log entry records the following information:

- Timestamp of the event
- Actor identifier (account ID and role)
- Action performed
- Target resource (e.g., report ID, cohort ID, account ID)
- Outcome of the action (success or failure)

Audit logs only record metadata to reduce privacy threats while upholding accountability; they do not keep sensitive content or raw health data.

**Retention and Disposal of Audit Logs**

For the purpose of supporting security investigations, compliance reviews, and dispute settlement, audit logs are kept for two years. This retention term is in line with security best practices, which advise maintaining audit records long enough to meet operational, legal, and administrative requirements.

In compliance with system governance policies, audit logs may be safely archived or removed when the retention time has passed. Reconstruction of audit records is not reasonably foreseeable thanks to secure disposal.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Reasonings:**

**Inactive user raw data retention: 1 year**

- It allows time for:
  - Account reactivation
  - Access requests
  - Dispute resolution
    
- It aligns with PHIPA guidance, which:
  - Requires "long enough" retention
  - Does not mandate a universal duration
    
- Short enough to respect data minimization
  
- Long enough to be operationally realistic

- Easy to enforce and explain


**Audit log retention: 2 years**

- Audit logs are governance and security records, not health data
  
- They are typically kept longer than operational data
  
- 2 years is:
  - Long enough for investigations and reviews
  - Short enough to avoid unnecessary accumulation
  - Common in academic, institutional, and mid-sized systems
  
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

**References**:

**PIPEDA Fair Information Principle 5 - Limiting Use, Disclosure, and Retention, (2020)**

<https://www.priv.gc.ca/en/privacy-topics/privacy-laws-in-canada/the-personal-information-protection-and-electronic-documents-act-pipeda/p_principle/principles/p_use/>

**CPSO, Medical Records Management**

<https://www.cpso.on.ca/physicians/policies-guidance/policies/medical-records-management>

**NIST Special Publication 800-53 (Audit Logging and Retention Controls)**

**AU-2 (Audit Events) and AU-11 (Audit Record Retention)**

<https://csf.tools/reference/nist-sp-800-53/r5/au/>






