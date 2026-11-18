# Employee Support Ticket Management Application

A secure, beginner-friendly, role-governed internal support ticket management system designed for logging, triaging, tracking, and resolving IT and operational workplace requests.

---

## 1. Project Overview

The **Employee Support Ticket Application** provides a centralized platform that bridges the communication gap between employees experiencing technical difficulties and IT support personnel.

### Key Capabilities
* **Role-Based Access Control (RBAC)**: Distinct permissions for **Employee** and **Support Staff** roles.
* **Scope Isolation**: Employees can strictly view, track, and comment on **only their own tickets**.
* **Global Support Queue**: Support staff have global visibility across all organization tickets with multi-attribute filtering (Status, Priority, Category, Employee).
* **Controlled Status Transitions**: Only Support Staff have permission to transition ticket statuses (`Open` &rarr; `In Progress` &rarr; `Resolved` &rarr; `Closed`).
* **Closed Ticket Immutability**: Once marked as `Closed`, core ticket attributes (`Title`, `Description`, `Category`, `Priority`) are strictly locked and read-only.
* **Persistent Discussion Trail**: Chronological commenting remains enabled for both Employees and Support Staff on tickets in any status, including `Closed`.

---

## 2. Technology Stack

* **Backend Runtime**: [Node.js](https://nodejs.org/) (v20+)
* **Application Framework**: [Express.js](https://expressjs.com/) (v4.21+)
* **Database**: [SQLite](https://www.sqlite.org/) via `sqlite3` driver with foreign key constraint enforcement
* **Authentication**: Cookie session-based authentication (`express-session`)
* **Security & Encryption**: Password hashing using Node.js native `crypto.scrypt` with a random 16-byte salt; verification using `crypto.timingSafeEqual`
* **Frontend**: Vanilla HTML5, modern responsive CSS3, and JavaScript (Fetch API) — zero bundler or build steps required

---

## 3. Project Directory Structure

```text
Employee_Support_Ticket_App/
├── .env.example                     # Environment configuration template
├── .gitignore                       # Ignored files (node_modules, SQLite databases, logs)
├── database.sqlite                  # SQLite persistent database file
├── package.json                     # Dependencies, metadata, and lifecycle scripts
├── PLAN.md                          # Approved step-by-step development plan
├── SPEC.md                          # Comprehensive software requirements specification
├── README.md                        # Documentation and production runbook
│
├── public/                          # Static frontend web application
│   ├── index.html                   # Landing page
│   ├── login.html                   # User authentication login portal
│   ├── register.html                # User registration portal with role picker
│   ├── employee-dashboard.html      # Employee personal tickets dashboard
│   ├── support-dashboard.html       # Support staff global queue & triage portal
│   ├── create-ticket.html           # New ticket submission form
│   ├── ticket-detail.html           # Ticket details, discussion trail & status controls
│   ├── css/
│   │   └── dashboard.css            # Unified stylesheet (badges, tables, forms, cards)
│   └── js/
│       ├── employeeDashboard.js     # Client logic for employee ticket monitoring
│       ├── supportDashboard.js      # Client logic for support staff queue triage
│       ├── createTicket.js          # Client logic for ticket submission
│       └── ticketDetail.js          # Client logic for ticket tracking, comments & status
│
├── src/                             # Backend application source code
│   ├── app.js                       # Express application configuration and entry point
│   ├── config/
│   │   └── config.js                # Environment configuration loader with fallbacks
│   ├── database/
│   │   ├── db.js                    # SQLite connection wrapper & promise helpers
│   │   └── schema.sql               # Relational DDL (Users, Tickets, Comments & Indexes)
│   ├── models/
│   │   ├── User.js                  # User data queries & role mappings
│   │   ├── Ticket.js                # Ticket queries, isolation rules & status updates
│   │   └── Comment.js               # Chronological comment queries
│   ├── controllers/
│   │   ├── authController.js        # Registration, login, logout, and session check
│   │   ├── ticketController.js      # Ticket creation, detail retrieval & status updates
│   │   └── commentController.js     # Comment authoring & retrieval
│   ├── routes/
│   │   ├── authRoutes.js            # Authentication routes (/api/auth)
│   │   └── ticketRoutes.js          # Ticket & commenting routes (/api/tickets)
│   ├── middleware/
│   │   └── authMiddleware.js        # Authentication & RBAC guard middleware
│   └── utils/
│       └── auth.js                  # Scrypt password hashing & verification utilities
│
└── tests/                           # Automated test suites (Zero external test runners)
    ├── verify_suite.js              # Database tables, models, and constraints verification
    ├── test_auth.js                 # Authentication, hashing, and session tests
    ├── test_employee_dashboard.js   # Employee dashboard & scope isolation tests
    ├── test_ticket_creation.js      # Ticket submission & mandatory validation tests
    ├── test_ticket_detail.js        # Detail view & scope isolation tests
    ├── test_support_dashboard.js    # Support staff global visibility & filter tests
    ├── test_ticket_status.js        # Status transitions & employee block tests
    ├── test_comments.js             # Discussion trail & closed ticket comment tests
    ├── test_validation_access_control.js # Closed immutability & PUT field tests
    └── test_e2e_regression.js       # SPEC.md Section 11 end-to-end regression suite
```

---

## 4. Getting Started

### Prerequisites
* Node.js v18.0.0 or higher
* npm v9.0.0 or higher

### Installation
1. Clone or navigate to the project directory:
   ```powershell
   cd c:\Users\USER\Documents\Employee_Support_Ticket_App
   ```

2. Install dependencies:
   ```powershell
   npm install
   ```

3. (Optional) Create environment configuration:
   ```powershell
   Copy-Item .env.example .env
   ```

4. Start the application:
   ```powershell
   npm start
   ```
   *(Or for auto-reloading during development: `npm run dev`)*

5. Open your browser:
   * **Home Portal**: [http://localhost:3000](http://localhost:3000)
   * **Login**: [http://localhost:3000/login.html](http://localhost:3000/login.html)
   * **Register**: [http://localhost:3000/register.html](http://localhost:3000/register.html)
   * **API Health Check**: [http://localhost:3000/api/health](http://localhost:3000/api/health)

---

## 5. API Reference

### Authentication Endpoints (`/api/auth`)
* `POST /api/auth/register`: Create account (`name`, `email`, `password`, `role`).
* `POST /api/auth/login`: Authenticate user and initiate session.
* `POST /api/auth/logout`: Invalidate session and clear cookie.
* `GET /api/auth/me`: Get active authenticated user session.

### Ticket Endpoints (`/api/tickets`)
* `POST /api/tickets`: Create support ticket (`title`, `description`, `category`, `priority`).
* `GET /api/tickets/my-tickets`: Get personal tickets for authenticated Employee (Scope Isolation).
* `GET /api/tickets/all`: Get global ticket queue (Support Staff only).
* `GET /api/tickets/:id`: Get ticket details and comments (Employees restricted to own tickets).
* `PUT /api/tickets/:id`: Update ticket fields (Rejected if ticket is `Closed` or user is not author).
* `PATCH /api/tickets/:id/status`: Transition ticket status (`Open`, `In Progress`, `Resolved`, `Closed`) — strictly restricted to Support Staff.

### Comment Endpoints (`/api/tickets/:id/comments`)
* `POST /api/tickets/:id/comments`: Add comment (Permitted on all tickets, including `Closed`).
* `GET /api/tickets/:id/comments`: Retrieve chronological comments for a ticket.

---

## 6. Testing & Quality Assurance

The application includes 10 test suites covering 142 discrete automated assertions. All tests use separate, temporary SQLite databases to guarantee zero interference with persistent data.

```powershell
# Run the unified SPEC.md Section 11 E2E regression suite
npm test

# Run all 10 test suites across the application (142 checks)
npm run test:all
```

---

## 7. Production Runbook

### Environment Variables
Configure the following in your production environment:
* `PORT`: Server port (default: `3000`).
* `NODE_ENV`: Set to `production` (enables secure cookies and production error masking).
* `SESSION_SECRET`: A long, cryptographically random secret string.
* `DB_PATH`: Custom path to persistent SQLite database file if mounted outside the app directory.

### Process Management (PM2 Recommended)
For production deployments, use a process manager like PM2:
```bash
npm install -g pm2
pm2 start src/app.js --name "ticket-app"
pm2 save
pm2 startup
```

### Database Backup & Maintenance
SQLite stores all relational data in a single file (`database.sqlite`).
* **Online Backup**: Use the SQLite online backup API or command line:
  ```powershell
  sqlite3 database.sqlite ".backup 'backup-$(Get-Date -Format 'yyyyMMdd-HHmmss').sqlite'"
  ```
* **Integrity Check**:
  ```powershell
  sqlite3 database.sqlite "PRAGMA integrity_check;"
  ```

### Security Considerations
* **HTTPS**: Run behind a reverse proxy (Nginx, Caddy, or Cloudflare) terminating TLS.
* **Session Cookie Flags**: In production (`NODE_ENV=production`), cookies are automatically flagged with `Secure; HttpOnly; SameSite=Lax`.
* **Rate Limiting**: Place rate-limiting middleware (such as `express-rate-limit`) in front of `/api/auth/login` to prevent brute force credential stuffing.
