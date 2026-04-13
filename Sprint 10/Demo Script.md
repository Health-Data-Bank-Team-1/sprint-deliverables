Demo Script for Project Presentation

Roles with Features/Functionality:
1. User (Patient) – Data entry, dashboard, progress, tasks, sharing, notifications, suggestions
2. Provider – Patient lookup/list, patient record, notes, reports, export
3. Researcher – Projects, request access (if present), cohort creation, aggregated reporting + export
4. Admin – User management, form approval workflow, audit log review, governance

Demo Scenario:
Patient tracking vitals and questionnaire data in HDB; 
provider uses it for care; 
researcher uses de-identified aggregates; 
admin governs forms and auditing.

Demo data props:

- Accounts to log in with:
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
      - Login as User
      - Land on User Dashboard (/user/dashboard)
  - Point 2: Forms + health data submission workflow
      - Go to Forms
      - Pick an available form (“Weekly Vitals & Symptoms”)
      - Start/Complete form
      - Fill a few fields (blood pressure, weight, symptoms—whatever your form has)
      - Submit 
  - Point 3: Health Summary + Recent Entries
      - Return to Dashboard or Health Summary
      - Show the newly created entry appearing in Recent Entries
      - Open Health Summary and click a metric for detail 
  - Point 4: Progress Tracking + export
      - Go to Progress
      - Switch time range (week/month/year)
      - Show chart hover points (if available)
      - Export progress report as PDF 
  - Point 5: Tasks/To-Do + Notifications/Reminders + Suggestions
      - Open Tasks/To-Do and mark one complete (if exists)
      - Open Notifications: show unread vs read, open one and mark read
      - Show Suggestions panel (even if empty, show where it appears and explain triggers)
  - Point 6: Profile + Security
      - Go to Profile Settings
      - Show 2FA / security settings / active sessions 

Part 2 - PROVIDER (HealthCare Provider)
  - Point 6: HCP dashboard + patient list
      - Logout → login as HCP
      - Show HCP Dashboard widgets (patient list, recent activity, alerts, tasks, reports)
  - Point 7: Access patient record
      - Go to Patients
      - Search/select the demo patient
      - Within patient record click through tabs
  - Point 8: Clinical notes
      - Add a new clinical note
  - Point 9: Provider reports + export
      - Generate a patient-specific report 7) Export as PDF

Part 3 - RESEARCHER
  - Point 10: Researcher dashboard + compliance framing
      - Logout → login as Researcher
      - Show Researcher Dashboard: Active Projects, Data Requests, Cohorts, Reports, Compliance/IRB status
  - Point 11: Create cohort (k-threshold)
      - Go to Cohorts
      - Create a new cohort using simple criteria that you know returns >= 10 users
  - Point 12: Aggregated reporting
      -  Go to Aggregated Reporting / Research Reporting
      -  Select cohort + select metrics (BP avg/min/max, count, trends, etc.)
      -  Generate the aggregated output
  - Point 13: Export aggregated results
      - Export aggregated report as CSV

Part 4 - ADMIN
  - Point 14: Admin dashboard + governance
      - Logout → login as Admin
      - Show Admin Dashboard: system metrics, pending items (forms, requests), audit summary
  - Point 15: Form approval workflow
      - Go to Forms / Form Templates
      - Open a pending template (or create one beforehand for demo)
      - Approve it (or show reject option briefly)
  - Point 16: Audit logs
      - Open Audit Log
      - Filter by
      - Open a few entries to show “who did what and when”
