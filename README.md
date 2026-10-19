# LeaveFlow — Leave Request & Approval System

LeaveFlow is a Microsoft Power Apps Canvas application designed to simplify employee leave management through a structured, multi-stage approval workflow. It enables employees to submit and track leave requests, managers to review team requests, and HR to perform second-stage approvals.

The application uses Microsoft Dataverse to store employee information, leave requests, leave balances, approval decisions, and demonstration login records.

## 1. Project Overview

**Project Name:** LeaveFlow  
**Application Type:** Microsoft Power Apps Canvas App  
**Platform:** Microsoft Power Platform  
**Database:** Microsoft Dataverse  
**Primary Purpose:** Employee leave management and approval automation

### Key Objectives

- Digitize the employee leave application process.
- Provide role-specific dashboards for employees, managers, and HR.
- Implement a sequential, two-stage leave approval workflow.
- Track leave allocations, usage, and remaining balances.
- Provide visibility into request statuses and approved leave.
- Support a demonstration login flow for showcasing different user roles.

## 2. User Roles

| Role | Responsibilities |
|---|---|
| Employee | Apply for leave, view personal requests, cancel eligible requests, and view leave balances and calendar information. |
| Manager | Review team leave requests, approve or reject requests at the first stage, and perform bulk decisions on selected requests. |
| HR | Review requests approved by managers, record second-stage decisions, and view approved leave within the authorized scope. |

The application provides role-specific navigation and pages according to the logged-in user's role.

## 3. Application Pages

### Employee Pages
- **Employee Dashboard** — Overview of leave activity and balances.
- **Apply for Leave** — Submit a leave request with the leave type, dates, and reason.
- **My Requests** — View the status and details of personal leave applications.
- **Calendar** — View relevant leave dates.

### Manager Pages
- **Manager Dashboard** — Summary of pending approvals and team leave activity.
- **Team Requests** — Review team requests and filter them by status, leave type, or date.
- **Manager Approvals** — Process requests at the first approval stage.
- **Bulk Decision** — Approve or reject multiple selected pending requests together.
- **Team Calendar** — View team leave information within the manager's authorized scope.

### HR Pages
- **HR Dashboard** — Overview of second-stage pending approvals and leave activity.
- **HR Approvals** — Review requests that have passed manager approval and record the final decision.
- **HR Calendar** — View approved leave within HR's authorized scope.

### Login
- **scrLogin** — Login screen for the demonstration workflow.
- **Logout** — Returns the current user to the login screen.

## 4. Microsoft Dataverse Data Model

LeaveFlow uses five Dataverse tables.

### 4.1 Employees

**Purpose:** Store employee details, role, department, and reporting manager.

| Column | Data type | Required |
|---|---|---|
| Employee Name | Text (Primary Name) | Yes |
| Employee Code | Text | Yes |
| Email | Email or Text | Yes |
| Department | Choice | Yes |
| Manager | Lookup → Employees | Conditional |
| Role | Choice | Yes |
| Active | Yes/No | Yes |
| Created On | Date and time (system) | Automatic |

**Role choices:** Employee, Manager, HR

**Department choices:** Engineering, Finance, Operations, or the values configured in the environment.

The Manager column is a self-referencing lookup to another Employees record. It stores a relationship to the manager's employee record rather than only storing a manager's name as text. The Manager field may be optional for HR users who do not report to another employee.

### 4.2 LeaveRequests

**Purpose:** Store each leave application and its current workflow status.

| Column | Data type | Required |
|---|---|---|
| Request Name | Text (Primary Name) | Yes |
| Request Code | Text | Yes |
| Employee | Lookup → Employees | Yes |
| Leave Type | Choice | Yes |
| Start Date | Date only | Yes |
| End Date | Date only | Yes |
| Reason | Multiple lines of text or Text | Yes |
| Total Days | Whole number | Yes |
| Status | Choice | Yes |
| Created On | Date and time (system) | Automatic |

**Leave Type choices:** Annual, Sick, Other

**Status choices:**
- Pending Manager Approval
- Pending Second Approval
- Approved
- Rejected
- Cancelled

The existing custom Status column uses schema name `new_Status`. It should be retained instead of creating a duplicate Status column.

For this MVP, Total Days is a whole number representing the number of leave days requested.

### 4.3 LeaveBalances

**Purpose:** Track allocated leave, used leave, and remaining balance for each employee and leave type.

| Column | Data type | Required |
|---|---|---|
| Balance Name | Text (Primary Name) | Yes |
| Balance Code | Text | Yes |
| Employee | Lookup → Employees | Yes |
| Leave Type | Choice | Yes |
| Allocated Days | Whole number | Yes |
| Used Days | Whole number | Yes |
| Remaining Days | Whole number | Yes |

Each employee should normally have a separate balance record for Annual, Sick, and Other leave.

The application should prevent approval when the employee does not have sufficient available leave balance. Balance updates should occur only according to the defined approval and cancellation rules, without deducting leave multiple times.

### 4.4 ApprovalRecords

**Purpose:** Record the manager and HR decisions associated with each leave request.

| Column | Data type | Required |
|---|---|---|
| Approval Name | Text (Primary Name) | Yes |
| Approval Code | Text | Yes |
| Leave Request | Lookup → LeaveRequests | Yes |
| Stage | Choice | Yes |
| Approver | Lookup → Employees | Yes |
| Decision | Choice | Yes |
| Comments | Multiple lines of text | Optional |
| Decision Date | Date and time | Optional until a decision is made |

**Stage choices:** Manager, Second Approver

**Decision choices:** Pending, Approved, Rejected

Each leave request should have separate approval records for the manager and second-approver stages. HR must only process a request after the manager has approved it.

### 4.5 DemoCredentials

**Purpose:** Support demonstration login by linking dummy login records to employee records.

| Column | Data type | Required |
|---|---|---|
| Credential Name | Text (Primary Name) | Yes |
| Email | Text | Yes |
| Demo Password | Text | Yes for the current demo design |
| Employee | Lookup → Employees | Yes |
| Active | Yes/No | Yes |

The Employee lookup connects each demo login to an Employees record. The application reads the associated employee's role to determine which dashboard and navigation to display.

**Important security note:** DemoCredentials is intended exclusively for a demonstration. Storing plaintext passwords in ordinary Dataverse columns is not secure authentication. Use only fictitious accounts and dummy data. This mechanism must not be used for real employee credentials or production authentication.

## 5. Demonstration Login Accounts

The following are example demonstration accounts, not real user credentials.

| Role | Email | Demo password | Employee |
|---|---|---|---|
| Employee | `bishwajit@leaveflow.demo` | `EmpDemo@123` | Bishwajit Satapathy |
| Manager | `rohit@leaveflow.demo` | `MgrDemo@123` | Rohit Sahu |
| HR | `arjun@leaveflow.demo` | `HrDemo@123` | Arjun Rao |

Only use these values if the corresponding dummy records have been configured in Dataverse. Do not use these passwords for personal or production accounts.

### Login Flow

1. The user enters an email address and password on `scrLogin`.
2. The app checks the demonstration credentials against active DemoCredentials records.
3. The app retrieves the associated employee record and role.
4. The user is routed to the relevant dashboard:
   - Bishwajit Satapathy → Employee Dashboard
   - Rohit Sahu → Manager Dashboard
   - Arjun Rao → HR Dashboard
5. The application displays role-specific pages and relevant information.
6. Logging out returns the user to the login screen and clears the app's current login state.

This is a demonstration login flow, not a substitute for Microsoft Entra ID or another secure authentication provider.

## 6. Leave Approval Workflow

LeaveFlow is designed around a sequential, two-stage approval process.

### Stage 1: Employee Submission
1. The employee submits a leave request.
2. The request is saved in LeaveRequests.
3. Its status becomes **Pending Manager Approval**.
4. The manager approval record is created or updated as pending.

### Stage 2: Manager Review
1. The manager reviews requests from their authorized team.
2. The manager approves or rejects a request.
3. If rejected, the request becomes **Rejected**.
4. If approved, it moves to **Pending Second Approval** and becomes eligible for HR review.

### Stage 3: HR Review
1. HR reviews requests that have passed manager approval.
2. HR records the second-stage decision and optional comments.
3. If HR rejects the request, its status becomes **Rejected**.
4. If HR approves it, its status becomes **Approved**.

### Stage 4: Balance and Calendar
- Validate leave balance before final approval.
- Update used and remaining leave balances consistently.
- Show approved leave in the relevant calendar.
- Prevent duplicate balance deductions when approval actions are repeated.

The precise balance-update point must be implemented consistently with the configured workflow to prevent duplicate deductions or inconsistent records.

## 7. Bulk Approval and Rejection

The Manager Approvals and Bulk Decision pages support processing multiple selected requests in a single operation.

Expected behavior:
- Display eligible pending requests.
- Allow individual selection and select-all.
- Show the number of selected requests.
- Provide **Approve Selected** and **Reject Selected** actions.
- Confirm the decision before applying it.
- Process only requests that remain eligible for manager approval.
- Display success or failure results for individual requests.
- Record approval decisions in ApprovalRecords.

Bulk actions must not bypass the second-approval stage or process requests that are already approved, rejected, or cancelled.

## 8. Validation and Business Rules

The application should enforce the following rules:

- Employee, leave type, start date, end date, and reason are required.
- The end date must not be earlier than the start date.
- Total Days must be calculated and validated consistently.
- Overlapping leave requests should be identified and warned about.
- Only authorized managers can process requests within their reporting scope.
- HR can process only requests that have passed manager approval.
- Cancelled or rejected requests must not be treated as approved leave.
- Sufficient leave balance must be available before approval.
- Leave balances must not be deducted more than once for the same request.
- Inactive employees and inactive demo credentials should not be used for new login sessions.

## 9. Technology Stack

- **Microsoft Power Apps Canvas App** — User interface and application screens.
- **Microsoft Dataverse** — Relational data storage for employees, leave requests, balances, approvals, and demo credentials.
- **Power Fx** — Formulas, validations, navigation, and app logic.
- **Microsoft Power Platform** — Environment and solution management.

Power Automate can be used for additional workflow automation, such as notifications and calendar integration, when configured in the target environment.

## 10. Solution Package and Installation

The exported Power Apps solution package is named:

`LeaveFlow_1_0_0_1_managed.zip`

This is a managed solution package intended for import into a Power Platform environment. It is not a standalone application that runs directly from GitHub.

### Import Instructions

1. Open the [Power Apps Maker Portal](https://make.powerapps.com/).
2. Select the target environment.
3. Open **Solutions**.
4. Select **Import solution**.
5. Upload the managed solution ZIP.
6. Follow the import wizard.
7. Configure connection references and environment variables if required.
8. Verify the required Dataverse tables, app, and configured dependencies.
9. Open the imported app and test the login, request submission, manager approval, HR approval, and balance workflow.

The target environment must have the required permissions, licensing, data configuration, and dependencies for the solution to function correctly.

## 11. Repository Structure

The repository is intended to contain the exported solution package and project documentation.

Example structure:

```text
LeaveFlow/
├── README.md
├── LeaveFlow_1_0_0_1_managed.zip
└── docs/
    └── project-documentation.md
```

The exact files may differ depending on which artifacts have been uploaded. The managed solution ZIP should be replaced or versioned when a newer export is published.

## 12. Project Scope

The MVP focuses on:
- Employee leave submission and request tracking.
- Manager approval and rejection.
- Second-stage HR approval and rejection.
- Role-specific dashboards and navigation.
- Leave balance tracking.
- Calendar visibility for approved leave.
- Bulk manager decisions.
- Demonstration login for employee, manager, and HR roles.

## 13. Limitations

- DemoCredentials is not a production authentication system.
- The managed solution package requires a compatible Power Platform environment.
- Connection references, environment settings, permissions, and demo records may need configuration after import.
- The availability of advanced automation depends on the components and licenses configured in the target environment.

## 14. Future Enhancements

Potential future enhancements include:
- Microsoft Entra ID authentication.
- Automated email and Teams notifications.
- Calendar integration for approved leave.
- Detailed audit reporting.
- Advanced leave analytics and team availability reporting.
- Stronger server-side enforcement of approval permissions and balance validation.

## 15. Project Status

LeaveFlow is an MVP focused on demonstrating the employee leave request lifecycle, sequential manager and HR approvals, and leave balance management using Microsoft Power Platform.

**Repository:** https://github.com/Bishwajit-Satapathy/LeaveFlow

**Platform:** Microsoft Power Platform  
**Application:** Power Apps Canvas App  
**Data Layer:** Microsoft Dataverse
