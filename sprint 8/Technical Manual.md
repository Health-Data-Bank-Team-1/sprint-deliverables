# Technical Manual

## Health-Data Bank
## System Architecture and Workflows Documentation

---

## 1. Executive Summary

### 1.1 Overview of Health Data Bank Architecture

The Health Data Bank system is a comprehensive, role-based health information management platform built on modern web technologies. It enables patients (Users), healthcare providers (HCPs), researchers, and administrators to securely manage, access, and analyze health data while maintaining strict HIPAA compliance and privacy standards.

The architecture follows a layered, modular design pattern that separates concerns across multiple layers: Routes, Controllers, Services, Repositories, Models, and Database. This separation ensures maintainability, testability, and scalability.

### 1.2 Core System Principles

#### Layered Architecture Design
The Health Data Bank implements a clean, multi-layered architecture that promotes separation of concerns and code reusability:

- **Routes Layer**: Defines HTTP endpoints and applies middleware
- **Controller Layer**: Handles HTTP requests and responses
- **Service Layer**: Contains business logic and application workflows
- **Repository Layer**: Manages all database operations and queries
- **Model Layer**: Represents database entities and relationships
- **Database Layer**: Persistent data storage using MySQL

This design allows developers to modify business logic without affecting routes, and to test individual layers independently.

#### Separation of Concerns
Each layer has distinct responsibilities:
- Controllers never directly access the database; they delegate to Services
- Services orchestrate business logic but don't handle HTTP concerns
- Repositories abstract database queries for easy testing and modification
- Models define data structure and relationships only

#### HIPAA Compliance and Data Security
The system is designed with healthcare data protection at its core:
- Protected Health Information (PHI) is encrypted at rest using AES-256
- All sensitive data transmission occurs over HTTPS/TLS
- Role-based access controls prevent unauthorized data access
- Comprehensive audit logging tracks all data access and modifications
- User authentication includes two-factor authentication (2FA)
- Data de-identification and aggregation for researcher access
- K-anonymity enforcement ensures research datasets meet privacy thresholds

### 1.3 Key Technologies and Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Backend Framework | Laravel 11.x (PHP 8.2+) | Web application framework |
| Authentication | Laravel Jetstream & Sanctum | User authentication & API tokens |
| Frontend | Livewire + Blade + Tailwind CSS | Interactive UI components |
| Build Tool | Vite | Asset compilation and bundling |
| Database | MySQL 8.0+ | Relational data storage |
| ORM | Eloquent | Database abstraction layer |
| Permissions | Spatie Permission | Role-based access control |
| Containerization | Docker & Docker Compose | Development & deployment |
| API Authentication | Laravel Sanctum | Bearer token authentication |

### 1.4 System Goals and Objectives

1. **Data Security**: Protect patient health information with industry-standard encryption and access controls
2. **Interoperability**: Provide secure data access to authorized providers and researchers
3. **Usability**: Create intuitive interfaces for different user roles
4. **Compliance**: Meet HIPAA, HITECH, and other healthcare regulations
5. **Scalability**: Support growing user bases and data volumes
6. **Auditability**: Maintain comprehensive logs of all system activities
7. **Privacy**: Implement k-anonymity and data aggregation for research

---

## 2. Technical Stack Overview

### 2.1 Backend Framework

**Laravel** is a modern PHP web framework that provides:
- Elegant routing system with RESTful support
- Built-in ORM (Eloquent) for database operations
- Comprehensive middleware system for request processing
- Service container for dependency injection
- Migration system for version-controlled database schema

**Jetstream** provides:
- Authentication scaffolding (registration, login, password reset)
- Team management functionality
- Two-factor authentication (2FA) setup and verification
- Profile management and security settings

Key dependencies in `composer.json`:
- `laravel/framework` - Core framework
- `laravel/jetstream` - Authentication UI
- `laravel/sanctum` - API token authentication
- `spatie/laravel-permission` - Role and permission management
- `owen-it/laravel-auditing` - Audit logging

### 2.2 Frontend Technologies

**Livewire** provides real-time, reactive components:
- Server-driven UI without page reloads
- Two-way data binding between frontend and backend
- Built-in form validation feedback
- Event-driven component communication
- File upload support with progress tracking

**Blade** is Laravel's templating engine:
- Simple, expressive syntax for HTML generation
- Component system for reusable UI elements
- Conditional rendering and loops
- Template inheritance for layout management

**Tailwind CSS** is a utility-first CSS framework:
- Responsive design utilities
- Dark mode support
- Pre-built components library
- Small production bundle size

**Vite** is a next-generation build tool:
- Fast hot module replacement (HMR) for development
- Optimized production bundles
- CSS and JavaScript compilation
- Asset versioning for cache busting

### 2.3 Database and Data Storage

**MySQL 8.0+** serves as the primary relational database:
- ACID compliance for data consistency
- Full-text search capabilities
- JSON column support for flexible data
- Row-level security through application logic
- Binary logging for point-in-time recovery

Database design includes:
- UUID primary keys for security and privacy
- Encrypted JSON columns for sensitive health data
- Soft deletes for audit trail preservation
- Polymorphic relationships for flexible model associations

### 2.4 Infrastructure and Deployment

**Docker and Docker Compose** containerize the application:
- `laravel.dockerfile` - Application container
- `mysql.dockerfile` - Database container
- `compose.yaml` - Orchestration configuration

**Laravel Sail** provides a pre-configured Docker development environment:
- Single command to start all services: `docker-compose -f compose.yaml up -d`
- Database included with automatic migration running
- Mail testing with Mailhog
- Database browser with Adminer

Production deployment uses:
- Kubernetes for orchestration (optional)
- GitHub Actions for CI/CD
- Environment-based configuration
- Health check endpoints for monitoring

### 2.5 Authentication and Security

**Laravel Sanctum** provides token-based API authentication:
- Bearer token authentication for API requests
- Token scoping for fine-grained permission control
- Ability to specify token expiration times
- No session cookies required for API clients

**Two-Factor Authentication (2FA)**:
- Time-based One-Time Password (TOTP) implementation
- QR code generation for authenticator app setup
- Backup codes for account recovery
- Mandatory for admin accounts, optional for users

**Role-Based Access Control (RBAC) via Spatie Permission**:
- Four core roles: `user`, `provider`, `researcher`, `admin`
- Granular permissions: `create`, `read`, `update`, `delete`
- Role and permission caching for performance
- Middleware enforcement at route and controller level
- Database pivot tables for flexible role-permission assignment

**Password Security**:
- Bcrypt hashing with configurable rounds
- Password reset tokens with expiration
- Session timeout enforcement
- CSRF protection on all state-changing requests

---

## 3. Layered Architecture Overview

### 3.1 Architecture Layers Diagram

```
HTTP Request
    ↓
[Route Layer] - HTTP endpoint definition
    ↓
[Controller Layer] - Request validation & HTTP handling
    ↓
[Service Layer] - Business logic & workflows
    ↓
[Repository Layer] - Database query abstraction
    ↓
[Model Layer] - Entity definitions & relationships
    ↓
[Database Layer] - MySQL persistent storage
    ↓
HTTP Response
```

### 3.2 Layer Descriptions

#### 3.2.1 Route Layer

**Location**: `routes/api.php`, `routes/web.php`

**Purpose**: Define HTTP request endpoints and apply middleware

**Key Routes Example**:
```php
// API routes with authentication
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('patients', PatientController::class);
    Route::get('patients/{patient}/summary', [PatientController::class, 'summary']);
    Route::post('forms/{form}/submit', [FormController::class, 'submit']);
});

// Admin routes with role middleware
Route::middleware(['auth:web', 'role:admin'])->group(function () {
    Route::get('admin/dashboard', AdminController::class . '@dashboard');
    Route::post('admin/forms/{form}/approve', FormApprovalController::class . '@approve');
});
```

**Middleware Stack**:
- `auth:sanctum` - Verify API token authentication
- `auth:web` - Verify session authentication
- `role:{name}` - Verify user has specific role
- `verified` - Verify email is verified
- `jetstream.auth_session` - Jetstream session verification
- `throttle:60,1` - Rate limiting

#### 3.2.2 Controller Layer

**Location**: `app/Http/Controllers/`

**Purpose**: Handle HTTP requests, validate input, coordinate business logic

**Example Controller Structure**:
```php
namespace App\Http\Controllers;

use App\Services\PatientService;
use Illuminate\Http\Request;

class PatientController extends Controller
{
    public function __construct(private PatientService $patientService) {}

    public function index(Request $request)
    {
        // Input validation via FormRequest
        $patients = $this->patientService->getPatients($request->validated());
        return response()->json(['data' => $patients]);
    }

    public function show($id)
    {
        // Authorize action
        $this->authorize('view', Patient::findOrFail($id));
        
        $patient = $this->patientService->getPatientDetail($id);
        return response()->json(['data' => $patient]);
    }
}
```

**Key Controllers**:
- `PatientController` - Patient CRUD operations and records
- `ProviderDashboardController` - Provider metrics and patient search
- `ResearcherCohortController` - Research cohort management
- `FormTemplateController` - Form template operations
- `AdminController` - System administration functions
- `AuthController` - Authentication and session management

**Controller Responsibilities**:
- Receive and parse HTTP requests
- Validate request data (via FormRequest classes)
- Delegate to appropriate services
- Authorize actions based on user role
- Format and return JSON responses
- Handle and log exceptions

#### 3.2.3 Service Layer

**Location**: `app/Services/`

**Purpose**: Encapsulate business logic and application workflows

**Example Service Structure**:
```php
namespace App\Services;

use App\Repositories\PatientRepository;
use App\Services\AuditLogger;

class PatientService
{
    public function __construct(
        private PatientRepository $patientRepository,
        private AuditLogger $auditLogger
    ) {}

    public function createPatient(array $data)
    {
        // Validate business rules
        if ($this->patientRepository->emailExists($data['email'])) {
            throw new PatientException('Email already registered');
        }

        // Create patient with encryption
        $patient = $this->patientRepository->create([
            'name' => $data['name'],
            'encrypted_values' => encrypt($data['sensitive_data']),
        ]);

        // Log action for audit trail
        $this->auditLogger->log('patient_created', $patient->id, [
            'name' => $patient->name,
            'email' => $patient->email
        ]);

        return $patient;
    }
}
```

**Key Services**:
- `PatientService` - Patient management and health data
- `ReportingAggregationService` - Aggregate metrics for accounts
- `AuditLogger` - Centralized audit logging (HIPAA-compliant)
- `CohortFilterBuilder` - Build researcher cohort queries
- `KThresholdService` - Enforce k-anonymity (minimum 10 individuals)
- `PersonalSummaryService` - Generate personal health summaries
- `TrendCalculationService` - Calculate health trend data
- `AggregatedMetricsService` - Aggregate research metrics
- `PersonalComparisonService` - Compare user metrics to cohorts

**Service Responsibilities:**

- `ReportingAggregationService`  
  - Aggregates reporting data for dashboards and summary views  
  - Supports report-oriented metric collection across accounts  

- `PersonalSummaryService`  
  - Produces user-specific summary data for personal health review  
  - Supports personal dashboard and summary endpoints  

- `TrendCalculationService`  
  - Calculates trend information over a selected time range  
  - Supports graphing and time-based reporting outputs  

- `AggregatedMetricsService`  
  - Computes aggregated metrics for grouped or cohort-based data access  
  - Supports researcher reporting workflows and aggregate analysis  

- `PersonalComparisonService`  
  - Compares a user’s personal data against broader aggregate values  
  - Supports user-facing comparison features while preserving privacy constraints  

- `AuditLogger`  
  - Centralizes audit logging across the system  
  - Records significant actions such as data access, report generation, approvals, and updates  
  - Supports accountability, traceability, and compliance requirements  

- `CohortFilterBuilder`  
  - Builds researcher cohort queries from selected filter criteria  
  - Supports cohort creation and filtered reporting workflows  

- `KThresholdService`  
  - Enforces minimum cohort-size requirements before aggregated data is returned  
  - Prevents reporting output that would violate privacy constraints  


**Reporting Services:**  
The reporting subsystem is implemented through coordinated service-layer components. These services retrieve data, calculate summaries and trends, apply aggregation rules, and return report-ready results for dashboards, personal summaries, and researcher analytics.  

**Audit Services:**  
Audit logging is handled centrally through `AuditLogger`, allowing multiple system workflows to record actions in a consistent format. This includes access events, approval events, report events, and other security-relevant operations.  

**RBAC Enforcement:**  
Role-based access control is enforced through Laravel middleware and the Spatie Permission package. At the service and controller levels, this ensures that users only access functionality and data appropriate to their role, such as User, Provider, Researcher, or Admin.  
#### 3.2.4 Repository Layer

**Location**: `app/Repositories/`

**Purpose**: Abstract database operations and query building

**Example Repository Structure**:
```php
namespace App\Repositories;

use App\Models\Patient;

class PatientRepository
{
    public function find($id)
    {
        return Patient::findOrFail($id);
    }

    public function create(array $attributes)
    {
        return Patient::create($attributes);
    }

    public function getActivePatients($accountId)
    {
        return Patient::where('account_id', $accountId)
            ->where('status', 'active')
            ->with(['healthEntries', 'healthGoals'])
            ->get();
    }
}
```

#### 3.2.5 Model Layer

**Location**: `app/Models/`

**Purpose**: Define entity structure and database relationships

**Key Models**:
- `User` - Authentication user with roles
- `Account` - Account records (User, Researcher, HCP, Admin)
- `HealthEntry` - Encrypted health data
- `FormTemplate` - Form definitions
- `Role` - Role definitions
- `Permission` - Permission definitions

#### 3.2.6 Database Layer

**Purpose**: Persistent storage using MySQL

**Key Tables**:
- `users` - Authentication users
- `accounts` - Account records
- `health_entries` - Encrypted health data
- `form_templates` - Form definitions
- `audits` - Audit trail logs
- `roles` - Role definitions
- `permissions` - Permission definitions
- `model_has_roles` - User-role associations

---

## 4. API Architecture

### 4.1 API Endpoints by Role

#### User/Patient Endpoints

GET    /api/patients                         - List patient records available to the authenticated user  
POST   /api/patients                         - Create a new patient record  
GET    /api/patients/{id}                    - Retrieve a specific patient record  
PUT    /api/patients/{id}                    - Update a specific patient record  
DELETE /api/patients/{id}                    - Delete a specific patient record  

GET    /api/goals                            - List health goals  
POST   /api/goals                            - Create a health goal  
GET    /api/goals/{goalId}                   - Retrieve a specific health goal  

GET    /api/me/summary                       - Retrieve personal health summary data  
GET    /api/reporting/trends                 - Retrieve personal trend data  
GET    /api/personal-comparison              - Retrieve comparison data against aggregate values  

GET    /api/reports/dashboard/trends         - Retrieve dashboard trend data  
GET    /api/reports/dashboard/trends/export.csv - Export dashboard trend data as CSV  


#### Provider Endpoints

GET    /api/provider/dashboard               - Retrieve provider dashboard metrics  
GET    /api/provider/patients/search         - Search for patient accounts  
GET    /api/provider/patients/{patient}      - Retrieve a specific patient record  
PUT    /api/provider/patients/{patient}      - Update provider-managed patient information  


#### Researcher Endpoints

POST   /api/researcher/cohorts               - Create a researcher-defined cohort  
GET    /api/researcher/cohorts               - List researcher cohorts  
GET    /api/research/reporting/aggregate     - Retrieve aggregated research reporting data  
POST   /api/researcher/reports/aggregated    - Generate an aggregated report  
POST   /api/researcher/reports/aggregated/export.csv - Export aggregated report data as CSV  


#### Admin Endpoints

GET    /api/admin/dashboard                  - Retrieve administrative dashboard metrics  
GET    /api/admin/audit-log                  - View audit log entries  

GET    /api/admin/forms                      - List form templates for administrative review  
POST   /api/admin/forms/{template}/approve   - Approve a form template  
POST   /api/admin/forms/{template}/reject    - Reject a form template  
POST   /api/admin/forms/{template}/submit    - Submit a form template into the approval workflow  

GET    /api/form-templates/{template}/versions            - View form version history  
POST   /api/form-templates/{template}/rollback/{version}  - Roll back to a previous form version  


**Endpoint Characteristics:**

- Endpoints are grouped by user role and protected by middleware  
- API access is authenticated using Laravel Sanctum  
- Role restrictions are applied using role-based middleware  
- Research endpoints return aggregated or filtered data rather than direct access to sensitive individual data  
- Administrative endpoints are restricted to users with elevated permissions  
```



### 4.2 API Authentication

**Method**: Bearer Token (Laravel Sanctum)

**Header**:
```
Authorization: Bearer {token}
```

**Token Scopes**:
- `read` - Read-only access to resources
- `create` - Create new resources
- `update` - Update existing resources
- `delete` - Delete resources

### 4.3 API Response Format

**Successful Response (200 OK)**:
```json
{
    "data": {
        "id": "uuid-string",
        "name": "John Doe",
        "email": "john@example.com",
        "created_at": "2026-03-24T10:30:00Z"
    }
}
```

**Error Response (400 Bad Request)**:
```json
{
    "message": "Validation failed",
    "errors": {
        "email": ["Email must be unique"]
    }
}
```

---

## 5. Frontend Architecture

### 5.1 Livewire Components

**Location**: `app/Livewire/`

**Key Components**:
- `UserDashboard` - User health overview
- `ProviderDashboard` - Healthcare provider metrics
- `ResearcherDashboard` - Research project overview
- `AdminDashboard` - System administration
- `FormRenderer` - Dynamic form display and submission
- `HealthSummary` - Personal health summary
- `HealthGoals` - Health goal management

### 5.2 Blade Templates

**Location**: `resources/views/`

**Template Structure**:
- `layouts/app.blade.php` - Main application layout
- `layouts/guest.blade.php` - Guest/unauthenticated layout
- `components/` - Reusable UI components
- `livewire/` - Livewire component views
- `auth/` - Authentication views
- `dashboard/` - Role-specific dashboards

### 5.3 Frontend Build and Assets

**Build Tool**: Vite

**CSS Framework**: Tailwind CSS
- Utility-first CSS approach
- Responsive design with breakpoints
- Dark mode support

---

## 6. Authentication and Authorization Workflows

### 6.1 Authentication Flow

```
User Login Form
    ↓
Validate Credentials (Fortify)
    ↓
If Valid:
    ├─→ Check if 2FA Enabled
    │   ├─→ Yes: Prompt for 2FA Code
    │   │   └─→ Verify TOTP
    │   └─→ No: Continue
    └─→ Create Session
        └─→ Redirect to Role Dashboard
        
If Invalid:
    └─→ Return Error Message
```

### 6.2 Authorization and RBAC

**System**: Spatie Laravel Permission

**Roles**:
- `user` - Regular patients
- `provider` - Healthcare providers
- `researcher` - Research investigators
- `admin` - System administrators

**Middleware Enforcement**:
```php
// Route-level role checking
Route::middleware('role:admin')->group(function () {...});

// Permission-based routes
Route::middleware('permission:approve-forms')->group(function () {...});
```

### 6.3 Session Management

**Session Driver**: Database or file-based

**Configuration**:
- Session timeout: 120 minutes
- Secure HTTP-only cookies
- CSRF protection enabled

---

## 7. Data Flow and Workflows

### 7.1 User Registration and Onboarding

```
Registration Form Submission
    ↓
Fortify Validation
    ├─ Email unique check
    ├─ Password strength check
    └─ Required fields validation
    ↓
Create User Model
    ├─ Hashed password (bcrypt)
    ├─ Generated UUID
    └─ Email verification token
    ↓
Create Account Model
    ├─ account_type = 'User'
    ├─ status = 'ACTIVE'
    └─ Link to User
    ↓
Assign 'user' Role
    ↓
Prompt 2FA Setup
    ├─ Generate TOTP secret
    ├─ Display QR code
    └─ Save secret (encrypted)
    ↓
Onboarding Complete
    └─ Redirect to User Dashboard
```

### 7.2 Health Data Submission Workflow

```
User Selects Form
    ↓
FormRenderer Component Loads
    ├─ Fetch form template
    ├─ Display form fields dynamically
    └─ Show field-level validations
    ↓
User Fills Out Form
    ├─ Livewire reactive validation
    ├─ Real-time error messages
    └─ Client-side calculations
    ↓
User Submits Form
    ↓
Server-Side Validation
    ├─ Validate each field
    ├─ Check required fields
    └─ Apply business rules
    ↓
Create HealthEntry
    ├─ Encrypt sensitive data (AES-256)
    ├─ Store encrypted_values column
    └─ Record timestamp
    ↓
Log Audit Event
    ├─ Event: 'health_entry_created'
    ├─ User ID
    ├─ Non-sensitive metadata only
    └─ Timestamp & IP
    ↓
Return Confirmation
    └─ User sees success message
```

### 7.3 Provider Patient Lookup Workflow

```
Provider Searches for Patient
    ↓
PatientSearchController::search()
    ├─ Receive search criteria
    └─ Get authenticated provider
    ↓
Validate Search Input
    ├─ Check search term length
    ├─ Sanitize input
    └─ Rate limit queries
    ↓
Query Accounts Table
    ├─ account_type = 'User'
    ├─ status = 'ACTIVE'
    ├─ Match name LIKE or email LIKE
    └─ Order by relevance
    ↓
Log Audit Event
    ├─ Event: 'patient_searched'
    ├─ Provider ID
    ├─ Search term hash
    └─ Result count
    ↓
Return Patient List
    └─ JSON response with patient summary
```

### 7.4 Researcher Cohort Creation Workflow

```
Researcher Creates Cohort
    ↓
ResearcherCohortController::store()
    ├─ Receive filter criteria
    ├─ Get authenticated researcher
    └─ Get research account
    ↓
Validate Cohort Definition
    ├─ Check filter parameters
    ├─ Ensure filters are defined
    └─ Validate date ranges
    ↓
CohortFilterBuilder Service
    ├─ Build WHERE clause from filters
    ├─ Join tables as needed
    └─ Identify matching accounts
    ↓
Count Matching Accounts
    └─ Get account_id list
    ↓
KThresholdService - Privacy Check
    ├─ If count >= 10: Proceed to store
    ├─ If count < 10: Suppress result
    └─ Return error: "Cohort too small"
    ↓
Store Cohort Definition
    ├─ Create researcher_cohorts record
    ├─ Save filter criteria JSON
    ├─ Set status = 'active'
    └─ Link to researcher
    ↓
Log Audit Event
    ├─ Event: 'cohort_created'
    ├─ Researcher ID
    ├─ Cohort size
    └─ Filter summary
    ↓
Return Cohort Details
    └─ Cohort ID, name, size
```

### 7.5 Researcher Data Access Workflow

```
Researcher Requests Aggregated Data
    ↓
ResearcherAggregateController::aggregate()
    ├─ Receive cohort ID and metric requests
    └─ Get authenticated researcher
    ↓
Validate Request
    ├─ Check cohort belongs to researcher
    ├─ Validate requested metrics
    └─ Check date range
    ↓
CohortFilterBuilder
    ├─ Retrieve cohort filter criteria
    ├─ Execute query to get account IDs
    └─ Build full account list
    ↓
AggregatedMetricsService
    ├─ For each metric (BP, HR, Weight, etc):
    │   ├─ Query health_entries for accounts
    │   ├─ Calculate MIN, MAX, AVG, COUNT
    │   ├─ Group by time period if requested
    │   └─ Apply aggregation
    └─ Return aggregate-only results
    ↓
KThresholdService - Verification
    ├─ Re-verify cohort size >= 10
    ├─ If suppressed: return error
    └─ If valid: continue
    ↓
De-identification Check
    ├─ Ensure NO individual records
    ├─ Only aggregated statistics
    ├─ NO demographic identifiers
    └─ NO timestamps for individuals
    ↓
Log Audit Event
    ├─ Event: 'cohort_data_accessed'
    ├─ Researcher ID
    ├─ Cohort ID
    ├─ Metrics requested
    └─ Record count returned
    ↓
Return Aggregated Data
    ├─ JSON with summary statistics
    ├─ Example: {"BP_avg": 120, "BP_min": 100, "BP_max": 150, "count": 150}
    └─ NO individual-level information
```

### 7.6 Admin Form Approval Workflow

```
Form Creator Submits Template
    ↓
FormTemplateController::store()
    ├─ Validate form structure
    ├─ Validate fields
    └─ Set status = 'pending'
    ↓
Store Form Template
    ├─ Create form_templates record
    ├─ Save form_fields
    └─ Link creator user
    ↓
Admin Notification
    └─ Admin sees pending form in dashboard
    ↓
Admin Reviews Form
    ↓
FormTemplateApprovalController
    ├─ Admin clicks approve or reject
    └─ Provide feedback (if rejecting)
    ↓
If Approved:
    ├─ Set status = 'approved'
    ├─ Set published_at = now()
    ├─ Form becomes available to users
    └─ Notify creator
    ↓
If Rejected:
    ├─ Set status = 'rejected'
    ├─ Store rejection reason
    └─ Notify creator with feedback
    ↓
Log Audit Event
    ├─ Event: 'form_template_approved' or 'rejected'
    ├─ Admin ID
    ├─ Form ID
    ├─ Approval status
    └─ Admin comments
    ↓
Update User Permissions
    ├─ If approved: users can now use form
    └─ If rejected: form unavailable
```

### 7.7 Audit Logging Workflow

```
User Action Triggered
    └─ E.g., create patient, access health data
    ↓
Service/Controller calls AuditLogger::log()
    ├─ Provide: event name, user ID, metadata
    └─ Pass request object for IP/URL
    ↓
AuditLogger Validates Sensitive Data
    ├─ NEVER log raw health values
    ├─ NEVER log passwords or tokens
    ├─ NEVER log direct identifiers (SSN, etc.)
    ├─ OK to log: IDs, names, actions, timestamps
    └─ Hash sensitive search terms
    ↓
Create Audit Record
    ├─ user_id - Who performed action
    ├─ event - Action performed
    ├─ auditable_id - Resource ID
    ├─ auditable_type - Resource type
    ├─ old_values - Previous state (if update)
    ├─ new_values - New state (sanitized)
    ├─ url - Request URL
    ├─ ip_address - Request IP
    ├─ user_agent - Browser/client info
    └─ created_at - Timestamp
    ↓
Store in Audits Table
    └─ Via owen-it/laravel-auditing package
    ↓
Admin Can Review
    ├─ View in Admin Dashboard
    ├─ Filter by user, event, date range
    ├─ Generate compliance reports
    └─ Export audit trail
```

### 7.8 Notification and Reminder Workflow

```

System Runs Scheduled Reminder Check
    ↓
Load Active Reminder Rules
    ↓
Evaluate Reminder Conditions
    ├─ Check inactivity conditions
    ├─ Check missing data submissions
    └─ Check scheduled reminder timing
    ↓
Identify Users Meeting Reminder Conditions
    ↓
Apply RBAC / Privacy Validation
    ├─ Ensure user is authorized
    └─ Ensure data belongs to user
    ↓
Generate Reminder Event
    ↓
Create Notification Record
    ├─ Assign user_id
    ├─ Assign notification type (reminder / alert)
    ├─ Set status = unread
    └─ Attach message content
    ↓
Store Notification in Database
    ↓
Check Notification Preferences
    ├─ In-app notification (dashboard)
    └─ Email (if enabled)
    ↓
Deliver Notification
    ↓
Log Notification Event (Audit Log)
    ↓
User Opens Notification
    ↓
Mark Notification as Read
    ↓
End Workflow

```
### 7.9 Suggestion System Workflow

```

System Trigger
    ├─ Periodic analysis
    └─ New data submission
    ↓
Load User Data and Reporting Metrics
    ↓
Check Preconditions
    ├─ Sufficient historical data available?
    └─ Suggestion feature enabled?
    ↓
Apply RBAC / Privacy Validation
    └─ Ensure user access allowed
    ↓
Analyze Data
    ├─ Evaluate trends
    └─ Evaluate aggregated metrics
    ↓
Evaluate Suggestion Rules
    ├─ Threshold conditions
    ├─ Trend-based conditions
    └─Missing data conditions
    ↓
Condition Met?
    ├─ No → End Workflow
    └─ Yes → Continue
    ↓
Generate Suggestion
    ├─ Assign suggestion type
    ├─ Create suggestion message
    └─ Attach user reference
    ↓
Store Suggestion Record
    ↓
Display Suggestion on Dashboard
    ↓
Optional: Send Notification for High Priority Suggestion
    ↓
Log Suggestion Event (Audit Log)
    ↓
End Workflow

```

### 7.10 Stretch Feature Integration

```

Reminder System
↓
Triggers Notification System
↓
Notification Stored and Delivered
↓
User Interacts via Dashboard

AND

User Data + Reporting System
↓
Processed by Suggestion System
↓
Suggestions Generated
↓
Displayed on Dashboard

All Components Enforce:
├─ RBAC (Role-Based Access Control)
├─ Privacy Constraints
└─  Audit Logging

```
---

## 8. Security Architecture

### 8.1 Authentication Security

**Password Hashing**: Laravel bcrypt with configurable cost factor
- Default cost: 12 iterations
- Automatic verification on login
- Password never stored in plain text

**Two-Factor Authentication (2FA)**:
- TOTP (Time-based One-Time Password) using RFC 6238
- 6-digit codes refreshing every 30 seconds
- Authenticator app compatible
- Backup codes for account recovery
- Mandatory for admin accounts

**Session Security**:
- Session tokens verified on every request
- Secure HTTP-only cookies (no JavaScript access)
- CSRF tokens on all state-changing requests
- Session timeout: 120 minutes of inactivity
- Session fixed on login to prevent fixation attacks

**API Token Security**:
- Bearer tokens in Authorization header
- Tokens stored as hashed values in database
- Token scoping restricts available operations
- Optional token expiration dates
- No cookies for API authentication

### 8.2 Data Protection

**Encryption at Rest**:
- Health data encrypted using AES-256-CBC
- Encryption key stored in .env (never in repository)
- Encrypted columns: `encrypted_values` in health_entries
- Database-level encryption possible (optional)

**Encryption in Transit**:
- HTTPS/TLS for all communication
- Minimum TLS 1.2 (1.3 preferred)
- Security headers configured:
  - `Strict-Transport-Security` - Force HTTPS
  - `X-Frame-Options` - Prevent clickjacking
  - `X-Content-Type-Options` - Prevent MIME sniffing
  - `X-XSS-Protection` - Enable browser XSS protection
  - `Content-Security-Policy` - Prevent script injection

**Data Anonymization**:
- De-identified datasets for researchers
- Cohort suppression if size < 10 (k-anonymity)
- Aggregated metrics only (no individual records)
- Demographic data removed from research exports
- Dates shifted by random offset (optional)

### 8.3 Role-Based Access Control

**Middleware Enforcement**:
```php
// Route-level role checking
Route::middleware('role:admin')->group(function () {...});

// Permission-based routes
Route::middleware('permission:approve-forms')->group(function () {...});

// Combined middleware
Route::middleware(['auth:sanctum', 'role:provider'])->group(function () {...});
```

**Granular Permissions**:
- Create (POST) - ability to create resources
- Read (GET) - ability to view resources
- Update (PUT/PATCH) - ability to modify resources
- Delete (DELETE) - ability to remove resources
- Custom permissions: approve-forms, export-data, manage-cohorts

**Account Type Validation**:
- Routes check user account type
- Account types: User, Researcher, HealthcareProvider, Admin
- Prevents unauthorized role access

### 8.4 Audit and Compliance

**Audit Logging**:
- Centralized AuditLogger service
- Never logs passwords, tokens, raw PHI
- Logs: user action, timestamp, IP, user agent, resource
- Audit trail immutable (append-only in database)
- 7-year retention policy (configurable)

**HIPAA Compliance**:
- No unencrypted PHI in logs or transit
- Access controls prevent unauthorized viewing
- Audit trails enable accountability
- Data retention policies enforced
- Breach notification procedures documented

### 8.5 Input Validation and Sanitization

**Request Validation**:
- Laravel FormRequest classes for validation
- Rules: required, string, email, unique, integer, date, etc.
- Custom validation rules for business logic
- Server-side validation always (never trust client)

**Output Escaping**:
- Blade templates auto-escape output (`{{ }}`)
- Raw output only when intentional (`{!! !!}`)
- JSON responses prevent injection
- API responses properly formatted

**SQL Injection Prevention**:
- Eloquent ORM parameterized queries
- No raw SQL with string concatenation
- Prepared statements used automatically
- Input never directly in WHERE clauses

---

## 9. Database Schema Overview

### 9.1 Core Tables

**Users Table**:
- `id` (UUID) - Primary key
- `name` - User name
- `email` - Email address
- `password` - Hashed password
- `two_factor_secret` - 2FA secret
- `two_factor_confirmed_at` - 2FA confirmation timestamp
- `account_id` (FK) - Link to Account
- `timestamps` - created_at, updated_at

**Accounts Table**:
- `id` (UUID) - Primary key
- `user_id` (FK) - User reference
- `name` - Account name
- `email` - Email address
- `account_type` - ENUM: User, Researcher, HealthcareProvider, Admin
- `status` - ENUM: ACTIVE, DEACTIVATED
- `timestamps` - created_at, updated_at

**Roles Table**:
- `id` (UUID) - Primary key
- `name` - Role name (user, provider, researcher, admin)
- `guard_name` - Guard (web, api)
- `description` - Role description
- `timestamps` - created_at, updated_at

**Model Has Roles Table**:
- `role_id` (FK) - Role ID
- `model_id` (FK) - User ID
- `model_type` - Model class
- Primary key: (role_id, model_id, model_type)

**Health Entries Table**:
- `id` (UUID) - Primary key
- `account_id` (FK) - Account ID
- `timestamp` - Entry timestamp
- `encrypted_values` - JSON encrypted health data
- `timestamps` - created_at, updated_at
- Index: (account_id, timestamp)

**Form Templates Table**:
- `id` (UUID) - Primary key
- `name` - Form name
- `description` - Form description
- `approval_status` - pending, approved, rejected
- `version` - Form version
- `created_by` (FK) - User ID
- `published_at` - Publication timestamp
- `timestamps` - created_at, updated_at
- `deleted_at` - Soft delete timestamp

**Audits Table**:
- `id` - Primary key (bigint)
- `user_type` - User model class
- `user_id` - User ID
- `event` - Event name
- `auditable_type` - Auditable model class
- `auditable_id` - Auditable model ID
- `old_values` - Before values (JSON)
- `new_values` - After values (JSON)
- `url` - Request URL
- `ip_address` - IP address
- `user_agent` - User agent
- `tags` - Tag string
- `created_at` - Timestamp

### 9.2 Table Relationships

- `User` → `Account` (one-to-one or many-to-one)
- `User` → `Roles` (many-to-many via model_has_roles)
- `Role` → `Permissions` (many-to-many via role_has_permissions)
- `Account` → `HealthEntries` (one-to-many)
- `Account` → `FormResponses` (one-to-many)
- `FormTemplate` → `FormFields` (one-to-many)

---

## 10. Key Services and Utilities

### 10.1 Core Services Reference

| Service | Responsibility | Key Methods |
|---------|-----------------|------------|
| PatientService | Patient CRUD and data retrieval | create(), update(), getPatientDetail(), getSummary() |
| ReportingAggregationService | Aggregate metrics across cohorts | aggregateMetrics(), calculateTrends() |
| AuditLogger | HIPAA-safe audit trail logging | log(), getAuditsByUser() |
| CohortFilterBuilder | Build complex researcher queries | buildQuery(), getMatchingAccounts() |
| KThresholdService | Enforce k-anonymity (min 10) | validateCohortSize(), suppressIfSmall() |
| PersonalSummaryService | Generate personal health summaries | getSummary(), calculateScore() |
| TrendCalculationService | Calculate health trends over time | getTrends(), calculateVelocity() |
| AggregatedMetricsService | Aggregate research-level metrics | getAggregates(), groupByPeriod() |
| PersonalComparisonService | Compare user metrics to cohorts | compare(), getPercentile() |

---

## 11. Middleware Stack

### 11.1 Route Middleware

| Middleware | Purpose |
|-----------|---------|
| `auth:sanctum` | Verify API token authentication |
| `auth:web` | Verify session authentication |
| `role:{name}` | Verify user has specific role |
| `permission:{name}` | Verify user has specific permission |
| `verified` | Verify email is verified |
| `jetstream.auth_session` | Verify Jetstream session |

### 11.2 HTTP Middleware Stack

Order of execution in `app/Http/Middleware/`:
1. TrustProxies - Trust reverse proxy headers
2. HandleCors - Cross-origin resource sharing
3. PreventRequestsDuringMaintenance - Maintenance mode
4. ValidatePostSize - Prevent oversized requests
5. TrimStrings - Trim whitespace from input
6. ConvertEmptyStringsToNull - Convert empty to null
7. VerifyCsrfToken - CSRF protection
8. Authenticate - Session/token authentication
9. Authorize - Role and permission checks

---

## 12. Configuration and Environment

### 12.1 Key Configuration Files

**config/permission.php**:
```php
return [
    'models' => [
        'permission' => \Spatie\Permission\Models\Permission::class,
        'role' => \Spatie\Permission\Models\Role::class,
    ],
    'table_names' => [
        'roles' => 'roles',
        'permissions' => 'permissions',
        'model_has_permissions' => 'model_has_permissions',
        'model_has_roles' => 'model_has_roles',
        'role_has_permissions' => 'role_has_permissions',
    ],
    'cache_expiration_time' => 24 * 60, // 24 hours
];
```

### 12.2 Environment Variables

Essential variables in `.env`:
```
APP_NAME="Health-Data-Bank"
APP_ENV=production
APP_KEY=base64:...
APP_DEBUG=false
APP_URL=https://health-data-bank.example.com

DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=hdb
DB_USERNAME=hdb_user
DB_PASSWORD=secure_password

CACHE_DRIVER=redis
QUEUE_CONNECTION=redis

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_USERNAME=...
MAIL_PASSWORD=...

SESSION_LIFETIME=120
SESSION_DRIVER=database
```

---

## 13. Development Workflow

### 13.1 Setting Up Development Environment

```bash
# 1. Clone repository
git clone https://github.com/Health-Data-Bank-Team-1/health-data-bank.git
cd health-data-bank

# 2. Copy environment file
cp .env.example .env

# 3. Generate app encryption key
php artisan key:generate

# 4. Start Docker containers
docker-compose -f compose.yaml up -d

# 5. Run database migrations
php artisan migrate

# 6. Seed initial data
php artisan db:seed

# 7. Install NPM dependencies
npm install

# 8. Build frontend assets
npm run dev

# 9. Access application
# http://localhost
```

### 13.2 Adding a New Feature

1. **Create Database Migration**:
   ```bash
   php artisan make:migration create_new_table
   ```

2. **Create Model**:
   ```bash
   php artisan make:model NewModel -m
   ```

3. **Create Repository**:
   ```bash
   mkdir -p app/Repositories
   # Create NewRepository.php
   ```

4. **Create Service**:
   ```bash
   php artisan make:class Services/NewService
   ```

5. **Create Controller**:
   ```bash
   php artisan make:controller NewController --resource
   ```

6. **Define Routes** in `routes/api.php`:
   ```php
   Route::apiResource('new', NewController::class);
   ```

7. **Test Implementation**:
   ```bash
   php artisan test
   ```

---

## 14. Deployment Architecture

### 14.1 Docker Setup

Services defined in `compose.yaml`:
- `laravel.app` - Laravel application
- `mysql` - MySQL 8.0 database
- `redis` - Redis cache (optional)
- `mailhog` - Email testing (development only)

### 14.2 Production Deployment

1. Build Docker images
2. Push to container registry
3. Run migrations: `php artisan migrate --force`
4. Clear caches: `php artisan cache:clear`
5. Build frontend: `npm run build`
6. Start services with orchestrator (Docker Compose or Kubernetes)

---

## 15. Performance Optimization

### 15.1 Caching Strategy

- Query caching for frequently accessed data
- Configuration caching in production
- Route caching in production
- View caching enabled

### 15.2 Database Optimization

- Indexes on foreign keys and frequently searched columns
- Lazy loading vs eager loading optimization
- Pagination for large result sets
- Query monitoring and optimization

### 15.3 Frontend Optimization

- Vite bundles and minifies assets
- CSS tree-shaking removes unused styles
- Code splitting for large components
- Asset versioning prevents cache issues

---

## 16. Monitoring and Logging

### 16.1 Application Logging

- Log level: DEBUG (development), WARNING (production)
- Logs stored in `storage/logs/`
- Structured logging with context
- No sensitive data in logs

### 16.2 Error Handling

- Exceptions caught and logged
- User-friendly error messages
- Detailed logs for debugging
- Email alerts for critical errors

### 16.3 Health Checks

- Application health endpoint: `/up`
- Database connectivity verification
- Cache validation
- Scheduled health monitoring

---

## 17. Glossary

- **CRUD** - Create, Read, Update, Delete operations
- **DTO** - Data Transfer Object
- **ORM** - Object-Relational Mapping
- **HIPAA** - Health Insurance Portability and Accountability Act
- **2FA** - Two-Factor Authentication
- **RBAC** - Role-Based Access Control
- **API** - Application Programming Interface
- **JWT** - JSON Web Token
- **UUID** - Universally Unique Identifier
- **PHI** - Protected Health Information
- **K-anonymity** - Privacy protection mechanism requiring minimum group size
- **TOTP** - Time-based One-Time Password
- **CSRF** - Cross-Site Request Forgery protection
