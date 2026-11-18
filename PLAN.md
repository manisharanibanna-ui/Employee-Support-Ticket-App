# Development Plan: Employee Support Ticket Application

This document outlines the step-by-step implementation plan for the **Employee Support Ticket Management Application** based on the specifications in [SPEC.md](file:///c:/Users/USER/Documents/Employee_Support_Ticket_App/SPEC.md).

---

## Technical Stack Architecture (Recommended Foundation)
* **Backend**: Node.js with Express (or Python FastAPI / Flask)
* **Database**: SQLite (relational, zero-config, ACID-compliant for ticket and comment persistence)
* **Frontend**: HTML5, CSS3 (Modern, responsive layout), JavaScript (Fetch API / UI interactions)
* **Authentication & Authorization**: Session/token-based auth with Role-Based Access Control (RBAC) middleware

---

## Task Breakdown

### Task 1: Project Setup and Configuration
* **Description**: Initialize the project environment, package manager, runtime configuration, directory layout, environment variables, and essential dependencies (server framework, database driver, session/auth libraries, testing harness).
* **Files Created / Modified**:
  * `package.json` / `requirements.txt`
  * `.env.example`
  * `.gitignore`
  * `src/config/config.js` (or `.py`)
  * `src/app.js` (or `app.py`)
* **Expected Outcome**: A runnable baseline server listening on a local port with environment variables loaded and health-check endpoint responding.
* **Acceptance Criteria from SPEC.md**:
  * **None directly linked**: This is an infrastructure and foundational setup task required for all downstream deliverables.

---

### Task 2: Database Design and Ticket Structure
* **Description**: Create the database schemas, migration scripts, and model layer for the `Users`, `Tickets`, and `Comments` tables according to Section 6 of `SPEC.md`. Define constraints, enums (`Hardware`, `Software`, `Network`, `Access`, `Other`; `Low`, `Medium`, `High`; `Open`, `In Progress`, `Resolved`, `Closed`), default values, foreign keys, and indexes.
* **Files Created / Modified**:
  * `src/database/db.js` (Database connection and initialization)
  * `src/database/schema.sql` (DDL for Users, Tickets, Comments)
  * `src/models/Ticket.js`
  * `src/models/Comment.js`
  * `src/models/User.js`
* **Expected Outcome**: Automated database initialization on boot, creating structured tables with all constraints and relational integrity checks.
* **Acceptance Criteria from SPEC.md**:
  * **None directly linked**: This is an architectural persistence task that underpins all functional scenarios but does not constitute an end-user acceptance scenario on its own.

---

### Task 3: Employee Registration and Login
* **Description**: Implement authentication and session management. Allow employees and support staff to register/login with role assignment (`Employee` vs `Support Staff`). Implement authentication middleware to inject user context (`id`, `name`, `role`) into subsequent requests.
* **Files Created / Modified**:
  * `src/routes/authRoutes.js`
  * `src/controllers/authController.js`
  * `src/middleware/authMiddleware.js`
  * `public/login.html`
  * `public/register.html`
  * `public/js/auth.js`
* **Expected Outcome**: Users can register and log in; the application securely distinguishes between an Employee and Support Staff member across sessions.
* **Acceptance Criteria from SPEC.md**:
  * **None directly linked**: SPEC.md Section 11 assumes an authenticated user as a precondition (`Given an authenticated Employee...`), but user account creation/authentication flow itself is not an explicit functional scenario in the acceptance criteria.

---

### Task 4: Employee Dashboard
* **Description**: Build the main UI dashboard for logged-in employees. Displays a summary of the employee's tickets with status badges (`Open`, `In Progress`, `Resolved`, `Closed`), priority badges, a quick link to create a new ticket, and filtering by status.
* **Files Created / Modified**:
  * `public/employee-dashboard.html`
  * `public/css/dashboard.css`
  * `public/js/employeeDashboard.js`
  * `src/routes/ticketRoutes.js` (Dashboard data endpoint)
* **Expected Outcome**: Employees land on a clear, personalized dashboard showing real-time progress of their submitted requests.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 3: Employee Scope Isolation**: Ensures the employee sees their tickets with status badges and no unauthorized tickets.

---

### Task 5: Create and Submit Support Tickets
* **Description**: Implement the ticket creation form and backend endpoint. The form captures `Title` (mandatory), `Description` (mandatory), `Category` dropdown, and `Priority` dropdown. The backend auto-populates `status = 'Open'`, `created_by = currentUser.id`, `created_date`, and `last_updated_date`.
* **Files Created / Modified**:
  * `public/create-ticket.html`
  * `public/js/createTicket.js`
  * `src/controllers/ticketController.js` (`createTicket` handler)
  * `src/routes/ticketRoutes.js` (`POST /api/tickets`)
* **Expected Outcome**: Employees can submit tickets; inputs are validated; newly created tickets persist with status `Open` and return a success confirmation.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 1: Successful Ticket Creation by Employee**
  * **Scenario 2: Validation Failure on Missing Mandatory Fields**

---

### Task 6: View and Track Personal Tickets
* **Description**: Implement personal ticket list and detailed view for employees. The backend query enforces strict isolation (`WHERE created_by = :userId`). The detail view shows full descriptions, metadata timestamps, current status, and comment history.
* **Files Created / Modified**:
  * `public/ticket-detail.html`
  * `public/js/ticketDetail.js`
  * `src/controllers/ticketController.js` (`getEmployeeTickets`, `getTicketById`)
  * `src/routes/ticketRoutes.js` (`GET /api/tickets/my-tickets`, `GET /api/tickets/:id`)
* **Expected Outcome**: An employee can click into any of their tickets to monitor status and inspect comments; attempting to open a ticket belonging to another employee returns a 403/404.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 3: Employee Scope Isolation**

---

### Task 7: Support Staff Dashboard
* **Description**: Build the centralized management dashboard and backend query for Support Staff. Displays all tickets from all employees across the organization. Provides sorting and filtering by Status, Priority, Category, and Employee name.
* **Files Created / Modified**:
  * `public/support-dashboard.html`
  * `public/js/supportDashboard.js`
  * `src/controllers/ticketController.js` (`getAllTickets`)
  * `src/routes/ticketRoutes.js` (`GET /api/tickets/all`)
  * `src/middleware/roleMiddleware.js` (Restricts route to Support Staff)
* **Expected Outcome**: Support staff can view, filter, sort, and search tickets across the entire organization in a consolidated triage interface.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 4: Support Staff Global Visibility**

---

### Task 8: Update Ticket Status
* **Description**: Implement status transition controls on the ticket detail view for Support Staff. Support staff can update status (`Open` -> `In Progress` -> `Resolved` -> `Closed`). The backend updates `status` and `last_updated_date`. Employees are strictly blocked from triggering this endpoint.
* **Files Created / Modified**:
  * `src/controllers/ticketController.js` (`updateTicketStatus`)
  * `src/routes/ticketRoutes.js` (`PATCH /api/tickets/:id/status`)
  * `public/js/ticketDetail.js` (Conditional rendering of status controls based on user role)
* **Expected Outcome**: Support staff can seamlessly change ticket statuses; status updates reflect immediately across dashboards; employee status change requests are rejected with 403 Forbidden.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 5: Status Change Permissions**

---

### Task 9: Add Comments to Tickets
* **Description**: Implement a chronological commenting system. Both Employees and Support Staff can post comments on tickets they have access to. Crucially, commenting must remain active and functional even when the ticket status is `Closed`.
* **Files Created / Modified**:
  * `src/controllers/commentController.js` (`addComment`, `getCommentsByTicket`)
  * `src/routes/commentRoutes.js` (`POST /api/tickets/:id/comments`, `GET /api/tickets/:id/comments`)
  * `public/js/ticketDetail.js` (Comment list rendering and submission logic)
* **Expected Outcome**: Chronological discussion trail on tickets with author and timestamp; comments can be added to tickets in any state, including `Closed`.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 7: Commenting on Closed Tickets**

---

### Task 10: Input Validation and Access Control Enforcement
* **Description**: Implement centralized validation and security guards:
  1. Mandatory field validation (`title` and `description` must not be blank).
  2. Category and Priority enum enforcement.
  3. Closed Ticket Lock: Reject any edits to `title`, `description`, `category`, or `priority` if `status == 'Closed'`.
  4. Role-based endpoint guards (preventing Employees from accessing support queues or status change actions).
* **Files Created / Modified**:
  * `src/middleware/validationMiddleware.js`
  * `src/middleware/roleMiddleware.js`
  * `src/controllers/ticketController.js` (Enforce closed immutability check)
* **Expected Outcome**: Robust server-side protection preventing unauthorized access, malformed inputs, and illicit mutations to closed tickets.
* **Acceptance Criteria from SPEC.md**:
  * **Scenario 2: Validation Failure on Missing Mandatory Fields**
  * **Scenario 3: Employee Scope Isolation**
  * **Scenario 5: Status Change Permissions**
  * **Scenario 6: Closed Ticket Field Modification Lock**

---

### Task 11: Testing and Bug Fixing
* **Description**: Write automated unit and integration tests covering all 7 scenarios in Section 11 of `SPEC.md`. Perform cross-role end-to-end verification and resolve any identified defects.
* **Files Created / Modified**:
  * `tests/ticketCreation.test.js`
  * `tests/accessControl.test.js`
  * `tests/statusTransition.test.js`
  * `tests/comments.test.js`
  * `tests/closedTicketLock.test.js`
* **Expected Outcome**: Complete test suite passing with 100% verification against the acceptance criteria.
* **Acceptance Criteria from SPEC.md**:
  * **Covers all Acceptance Scenarios (Scenario 1 through Scenario 7)**

---

### Task 12: Final Code Review and Documentation
* **Description**: Perform static analysis, code cleanup, security review (SQL injection prevention, XSS sanitation, authorization bypass checks), and assemble a run-book/README with startup and usage instructions.
* **Files Created / Modified**:
  * `README.md`
  * Codebase refactoring and formatting across all files
* **Expected Outcome**: Production-ready, maintainable, secure codebase with comprehensive execution documentation.
* **Acceptance Criteria from SPEC.md**:
  * **None directly linked**: Quality assurance, security auditing, and documentation governance tasks do not link to a specific functional acceptance criterion.

---

## Tasks Not Directly Linked to an Acceptance Criterion

As requested, the following tasks from the development plan do not map directly to a single functional acceptance scenario in Section 11 of `SPEC.md`:

| Task # | Task Name | Reason Not Directly Linked to an AC |
| :--- | :--- | :--- |
| **Task 1** | **Project setup and configuration** | Foundational scaffolding (package management, server config, directory structures). It enables the system to run but does not represent a user-facing business feature. |
| **Task 2** | **Database design and ticket structure** | Data architecture and persistence layer. Essential for fulfilling all requirements, but internal schema creation has no direct standalone acceptance scenario. |
| **Task 3** | **Employee registration and login** | Authentication mechanism. In Section 11, authenticated users are defined as a precondition (`Given an authenticated Employee...`), but user registration and credential handling were not written as dedicated acceptance scenarios in SPEC.md. |
| **Task 12** | **Final code review** | Quality assurance, security sanitization, and documentation activity. Verifies structural and non-functional health rather than exercising a specific functional acceptance criterion. |

---

## Next Steps
This plan is ready for execution. **No code has been written.** Please review this plan and give your approval or specify any desired adjustments before implementation begins.
