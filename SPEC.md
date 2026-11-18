# Software Requirements Specification (SRS)
## Employee Support Ticket Management Application

---

## 1. Project Overview
The **Employee Support Ticket Management Application** is an internal support tracking system designed to streamline the logging, triage, handling, and resolution of workplace IT and operational issues. The application bridges the gap between employees experiencing technical difficulties and support personnel tasked with resolving them, providing transparent lifecycle tracking and clear separation of duties.

---

## 2. Problem Statement
In organizational environments without dedicated ticketing systems, support requests are commonly submitted through ad-hoc channels (emails, chat messages, or verbal requests). This leads to:
* Requests getting lost, duplicated, or forgotten.
* Lack of visibility for employees regarding the progress of their issues.
* Difficulty for support staff in prioritizing urgent tasks.
* Inability to maintain an audit trail or discussion history per issue.
* Uncontrolled edits to tickets post-resolution.

This application provides a centralized, role-governed platform ensuring accountability, visibility, and controlled workflows.

---

## 3. User Roles

| Role | Description | Core Capabilities |
| :--- | :--- | :--- |
| **Employee** | Regular organization staff member submitting requests. | Create tickets, view personal tickets, track status, add comments. |
| **Support Staff** | Technical/administrative personnel resolving issues. | View all tickets, update ticket statuses, review history, add comments. |

---

## 4. Functional Requirements

### 4.1 Ticket Creation
* **FR-1.1**: Employees must be able to log a support ticket by providing a Title, Description, Category, and Priority.
* **FR-1.2**: New tickets must automatically be initialized with the status **Open**.
* **FR-1.3**: The system must automatically capture and persist the creator identity (`Created By`), creation timestamp (`Created Date`), and initial last updated timestamp (`Last Updated Date`).

### 4.2 Ticket Viewing & Queues
* **FR-2.1**: **Employee Scope Isolation**: When an Employee views tickets, the system must return **only** tickets where `Created By` matches the authenticated Employee's identity.
* **FR-2.2**: **Support Staff Global Scope**: Support Staff must have access to a global view displaying all tickets logged across the organization.
* **FR-2.3**: Both roles must be able to view full ticket details, including current status and complete comment history.

### 4.3 Status Management
* **FR-3.1**: The system must strictly restrict status update operations to users with the **Support Staff** role.
* **FR-3.2**: Employees must be prohibited from modifying ticket status at any point.
* **FR-3.3**: Supported status values are: `Open`, `In Progress`, `Resolved`, and `Closed`.
* **FR-3.4**: Updating a status must automatically update the `Last Updated Date` timestamp.

### 4.4 Commenting Subsystem
* **FR-4.1**: Both Employees and Support Staff can post comments on tickets.
* **FR-4.2**: Comments must be appended in chronological order with author identity and timestamp.
* **FR-4.3**: **Comments on Closed Tickets**: The system must allow users to post comments on tickets even when the status is **Closed**.

### 4.5 Ticket Editing & Immutability
* **FR-5.1**: Tickets with status `Open`, `In Progress`, or `Resolved` may have general fields edited if permitted by business policy.
* **FR-5.2**: **Closed Ticket Immutability**: Once a ticket's status is set to **Closed**, all core ticket attributes (`Title`, `Description`, `Category`, `Priority`) are strictly locked and read-only. No updates to these fields are permitted.

---

## 5. Non-Functional Requirements

* **NFR-1: Security & Data Isolation**: Role-based access control (RBAC) must be enforced at the business logic and API layer, not solely in the user interface.
* **NFR-2: Data Integrity**: Modification timestamps and author stamps must be system-generated to prevent tampering. Closed ticket immutability must be strictly maintained in persistent storage.
* **NFR-3: Usability**: Clear visual status indicators (e.g., color badges for Priority and Status) to enable quick assessment by both employees and support agents.
* **NFR-4: Auditability**: Every status transition and comment submission must be tracked with associated timestamps.
* **NFR-5: Performance**: The ticket list retrieval for both employees and support staff should respond within sub-second latency under standard operational volumes.

---

## 6. Ticket Data Fields

### 6.1 Ticket Entity

| Field Name | Data Type | Mandatory | Allowed / Default Values | Description |
| :--- | :--- | :--- | :--- | :--- |
| `id` | UUID / Integer | System Generated | Unique identifier | Primary key |
| `title` | String | **Yes** | 1 to 200 characters | Brief summary of the issue |
| `description` | Text | **Yes** | Non-empty text | Comprehensive details of the issue |
| `category` | Enum | **Yes** | `Hardware`, `Software`, `Network`, `Access`, `Other` | Type of problem reported |
| `priority` | Enum | **Yes** | `Low`, `Medium`, `High` | Urgency of the request |
| `status` | Enum | System Managed | `Open`, `In Progress`, `Resolved`, `Closed`<br>*(Default: `Open`)* | Current progress state |
| `created_by` | String / Reference | System Generated | Active user identifier | Employee who created the ticket |
| `created_date` | Timestamp | System Generated | ISO 8601 UTC timestamp | Creation time |
| `last_updated_date`| Timestamp | System Generated | ISO 8601 UTC timestamp | Most recent update time |

### 6.2 Comment Entity

| Field Name | Data Type | Mandatory | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID / Integer | System Generated | Unique comment identifier |
| `ticket_id` | Foreign Key | System Generated | Associated ticket reference |
| `author` | String / Reference | System Generated | Identity of commenter (Employee or Support Staff) |
| `content` | Text | **Yes** | The comment text content |
| `created_date` | Timestamp | System Generated | ISO 8601 UTC timestamp |

---

## 7. Employee Workflow

```
[Employee Logs In]
         │
         ▼
[Employee Dashboard] ──► Views personal tickets only (with current status)
         │
         ├───► [Create New Ticket]
         │          │
         │          ├──► Enters Title* & Description*
         │          ├──► Selects Category & Priority
         │          └──► Submits ──► Saved as "Open" (Timestamp generated)
         │
         └───► [Selects Ticket Details]
                    │
                    ├──► Reviews status and staff responses
                    └──► [Add Comment] (Allowed anytime, including "Closed" tickets)
```

---

## 8. Support Staff Workflow

```
[Support Staff Logs In]
         │
         ▼
[Global Ticket Queue] ──► Views all tickets across all employees
         │
         ├───► [Filter / Sort] (By Status, Priority, Category, Employee)
         │
         └───► [Select Ticket to Review]
                    │
                    ├──► Read full issue description & comment history
                    ├──► [Add Comment] (Provide updates / request info)
                    │
                    └──► [Update Status]
                              │
                              ├──► Move to "In Progress"
                              ├──► Move to "Resolved"
                              └──► Move to "Closed" (Locks ticket fields)
```

---

## 9. Validation Rules

1. **Title Validation**:
   * Must not be null or empty.
   * Whitespace-only strings are rejected.
2. **Description Validation**:
   * Must not be null or empty.
   * Whitespace-only strings are rejected.
3. **Category Validation**:
   * Must strictly match one of: `Hardware`, `Software`, `Network`, `Access`, `Other`.
4. **Priority Validation**:
   * Must strictly match one of: `Low`, `Medium`, `High`.
5. **Status Update Authority**:
   * Any request to update `status` submitted by an Employee must be rejected with an `Access Denied / Forbidden` error.
6. **Closed Ticket Field Lock**:
   * Any request attempting to modify `title`, `description`, `category`, or `priority` on a ticket whose current status is `Closed` must be rejected with a `Validation / State Conflict` error.
7. **Comment Validation**:
   * Comment text must not be empty or whitespace-only.
   * Comment submission must succeed regardless of ticket status (even if `Closed`).

---

## 10. Access Permissions Matrix

| Action | Employee | Support Staff |
| :--- | :---: | :---: |
| **Create Ticket** | ✅ Yes | ❌ No (or as self) |
| **View Own Tickets** | ✅ Yes | ✅ Yes |
| **View All Tickets Across Org** | ❌ No | ✅ Yes |
| **Edit Ticket Fields (Open / In Progress / Resolved)** | ✅ Yes (Own only) | ✅ Yes |
| **Edit Ticket Fields (Closed)** | ❌ No (Locked) | ❌ No (Locked) |
| **Change Ticket Status** | ❌ **No (Strictly Forbidden)** | ✅ **Yes** |
| **Add Comment (Open / In Progress / Resolved)** | ✅ Yes | ✅ Yes |
| **Add Comment (Closed)** | ✅ **Yes** | ✅ **Yes** |

---

## 11. Acceptance Criteria

### Scenario 1: Successful Ticket Creation by Employee
* **Given** an authenticated Employee is on the "New Ticket" page,
* **When** they fill in a valid `Title`, `Description`, and select `Category` and `Priority`,
* **Then** the ticket is saved with status `Open`, author set to the Employee, and timestamps generated.

### Scenario 2: Validation Failure on Missing Mandatory Fields
* **Given** an Employee is creating a ticket,
* **When** the `Title` or `Description` is left blank or whitespace,
* **Then** the submission is blocked and a descriptive validation error is displayed.

### Scenario 3: Employee Scope Isolation
* **Given** multiple tickets created by different employees in the system,
* **When** Employee "Alice" views her ticket list,
* **Then** she sees only tickets where `Created By == Alice`, and none belonging to other employees.

### Scenario 4: Support Staff Global Visibility
* **Given** tickets submitted by multiple different employees,
* **When** a Support Staff member views the ticket list,
* **Then** all tickets across the entire organization are visible.

### Scenario 5: Status Change Permissions
* **Given** an existing ticket with status `Open`,
* **When** an Employee attempts to change the status to `In Progress` or `Closed`,
* **Then** the request is rejected with a forbidden status error.
* **When** a Support Staff member changes the status to `In Progress`,
* **Then** the status is updated and `Last Updated Date` is refreshed.

### Scenario 6: Closed Ticket Field Modification Lock
* **Given** a ticket currently marked as `Closed`,
* **When** any user attempts to edit the `Title`, `Description`, `Category`, or `Priority`,
* **Then** the system rejects the operation and retains existing data.

### Scenario 7: Commenting on Closed Tickets
* **Given** a ticket currently marked as `Closed`,
* **When** either an Employee or Support Staff submits a valid non-empty comment,
* **Then** the comment is successfully saved and appended to the ticket's comment trail.
