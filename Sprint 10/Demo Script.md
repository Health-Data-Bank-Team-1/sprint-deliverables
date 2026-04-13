Demo Script for Project Presentation

Roles with Features/Functionality:
1. User (Patient) – Data entry, dashboard, progress, tasks, sharing, notifications, suggestions
2. Provider – Patient lookup/list, patient record, notes, reports, export
3. Researcher – Projects, request access (if present), cohort creation, aggregated reporting + export
4. Admin – User management, form approval workflow, audit log review, governance

Demo Scenario:
“Patient tracking vitals and questionnaire data in HDB; provider uses it for care; researcher uses de-identified aggregates; admin governs forms and auditing.”

Demo data props:

- Accounts to log in with: user_demo, provider_demo, researcher_demo, admin_demo
    - user@demo.com, Password
    - provider@demo.com, Password
    - researcher@demo.com, Password
    - admin@demo.com, Password
  
- One approved form already exists
  -  Demo form: Weekly Vitals & Symptoms

- One pending form template exists for admin approval (to show workflow)
- User has either no entries yet or just a few (so your new entry is visible)
- Research cohort size should be >= 10 to avoid the “cohort too small” suppression (k-threshold)

Part 1 - USER (Patient)
  - Point 1: Login + RBAC
  - Point 2: Forms + health data submission workflow
  - Point 3: Health Summary + Recent Entries
  - Point 4: Progress Tracking + export
  - Point 5: Tasks/To-Do + Notifications/Reminders + Suggestions
  - Point 6: Profile + Security

Part 2 - PROVIDER (HealthCare Provider)
  - Point 6: HCP dashboard + patient list
  - Point 7: Access patient record
  - Point 8: Clinical notes
  - Point 9: Provider reports + export

Part 3 - RESEARCHER
  - Point 10: Researcher dashboard + compliance framing
  - Point 11: Create cohort (k-threshold)
  - Point 12: Aggregated reporting
  - Point 13: Export aggregated results

Part 4 - ADMIN
  - Point 14: Admin dashboard + governance
  - Point 15: Form approval workflow
  - Point 16: Audit logs
  - Point 17: System governance / configuration
