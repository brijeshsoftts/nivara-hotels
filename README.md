# Nivara Hotels

A full-stack hotel management and online room reservation system built with **NestJS**, **React**, **PostgreSQL**, and **Prisma ORM**. Nivara Hotels provides a modern public-facing portal for room discovery and online booking, alongside a comprehensive, role-guarded operations dashboard for hotel staff and administrators.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Architecture and Repository Structure](#architecture-and-repository-structure)
4. [Implemented Features](#implemented-features)
5. [User Roles and Permissions](#user-roles-and-permissions)
6. [Authentication and Authorization](#authentication-and-authorization)
7. [Core Application Workflows](#core-application-workflows)
8. [API Route Catalog](#api-route-catalog)
9. [Database Schema and Models](#database-schema-and-models)
10. [Environment Configuration](#environment-configuration)
11. [Installation and Local Setup](#installation-and-local-setup)
12. [Available Scripts and Commands](#available-scripts-and-commands)
13. [Build and Deployment](#build-and-deployment)
14. [Codebase Verification and Audit Notes](#codebase-verification-and-audit-notes)

---

## Project Overview

Nivara Hotels is designed as a modular monorepo containing two dedicated applications:
- **`apps/api`**: A RESTful backend service built on NestJS 11, Express, and Prisma 7, delivering API endpoints for authentication, reservations, payments, housekeeping, maintenance issues, staff management, audit logging, and reporting.
- **`apps/web`**: A responsive single-page application (SPA) built with React 19, Vite 8, Tailwind CSS v4, and TanStack Query v5, serving both public visitors and authenticated hotel employees across four distinct dashboard experiences.

The system emphasizes strict transactional integrity for hotel bookings (row-level database locking to avoid double-booking), automated status transitions between housekeeping tasks and room readiness, and end-to-end payment lifecycle handling (front-desk cash, UPI/QR, and online Razorpay orders, signatures, webhooks, and refunds).

---

## Technology Stack

### Backend (`apps/api`)

| Category | Technology | Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Framework** | NestJS | `^11.0.1` | Modular backend architecture, controllers, services, guards |
| **HTTP Engine** | Express (`@nestjs/platform-express`) | `^11.0.1` | Underlying HTTP server framework |
| **Language** | TypeScript | `^5.7.3` | Type safety and domain modeling |
| **Database ORM** | Prisma Client & CLI | `^7.10.0` | Relational ORM with Prisma 7 config (`prisma7.config.ts`) |
| **Database Adapter** | `@prisma/adapter-pg` / `pg` | `^7.10.0` / `^8.23.1` | Native PostgreSQL connection pooling adapter |
| **Authentication** | `@nestjs/jwt`, `argon2` | `^12.0.1` / `^0.45.1` | JWT signing, verification, and Argon2id password hashing |
| **Cookies** | `cookie-parser` | `^1.4.7` | HTTP-only cookie parsing for JWT access & refresh tokens |
| **Validation** | `zod`, `nestjs-zod` | `^4.6.2` / `^5.5.0` | Runtime schema validation pipes and DTO type generation |
| **File Handling** | `multer` | `^2.4.0` | Multipart/form-data upload interception |
| **Cloud Storage** | `node-appwrite` | `^29.0.0` | Appwrite storage integration for images, ID proofs, signatures |
| **Payment Gateway** | `razorpay` | `^2.9.8` | Order creation, payment verification, refunds, webhooks |
| **Email Service** | `nodemailer` | `^10.2.1` | SMTP transport for password resets and contact messages |
| **Security** | `helmet` | `^8.3.0` | HTTP security headers |
| **Testing** | Jest, `supertest` | `^30.0.0` / `^7.0.0` | Unit and e2e API testing |

### Frontend (`apps/web`)

| Category | Technology | Version | Purpose |
| :--- | :--- | :--- | :--- |
| **Library** | React | `^19.2.8` | Component-based UI library |
| **Build Tool** | Vite | `^8.0.0` | Fast development server and production bundler |
| **Routing** | React Router (`react-router-dom`) | `^7.18.4` | Declarative client-side routing and nested layouts |
| **Styling** | Tailwind CSS | `^4.0.0` | Utility-first CSS styling via `@tailwindcss/vite` |
| **Server State** | TanStack React Query | `^5.104.1` | Async server state, caching, refetching, and mutations |
| **HTTP Client** | Axios | `^1.20.0` | HTTP client with automatic 401 token refresh queue |
| **Forms** | React Hook Form, `@hookform/resolvers` | `^7.89.0` / `^5.9.1` | Uncontrolled/controlled form state with Zod validation |
| **Validation** | Zod | `^4.6.5` | Client-side schema validation matching API contracts |
| **UI Components** | shadcn/ui & Base UI (`@base-ui/react`) | `^1.8.0` / `^4.21.1` | Accessible dialogs, popovers, dropdowns, tables, sidebars |
| **Icons** | Lucide React | `^1.50.0` | Consistent iconography across views and badges |
| **Animations** | Motion (`motion`) | `^14.9.1` | Page and layout enter/exit transitions |
| **PDF Generation**| `@react-pdf/renderer` | `^4.9.0` | Client-side invoice and payment receipt PDF export |
| **Notifications** | Sonner | `^2.0.8` | Toast alerts for mutations and error reporting |
| **Charts** | Recharts | `^3.8.0` | Revenue trend and booking status visualizations |
| **Dates** | `date-fns`, `react-day-picker` | `^4.4.0` / `^10.0.2` | Date manipulation, range pickers, calendar widgets |
| **Payment SDK** | `react-razorpay` | `^3.0.1` | Razorpay modal integration for client-side checkout |
| **Theming** | `next-themes` | `^0.4.6` | Dark mode and light mode theme toggling |

### Workspace and Infrastructure

- **Package Manager**: `pnpm` (independent workspaces for `apps/api` and `apps/web`)
- **Process Orchestration**: `concurrently` (runs frontend and backend concurrently via root `pnpm run dev`)
- **Containerization**: `docker-compose.yaml` (PostgreSQL Alpine database service)
- **Deployment**: Vercel configuration for static SPA rewrites (`apps/web/vercel.json`) and Node production build output for NestJS API.

---

## Architecture and Repository Structure

```
nivara-hotels/
├── apps/
│   ├── api/                       # NestJS Backend Application
│   │   ├── prisma/                # Prisma schema & SQL migrations
│   │   │   ├── migrations/        # Sequential PostgreSQL migrations
│   │   │   └── schema.prisma      # Central Prisma schema definition
│   │   ├── src/
│   │   │   ├── common/            # Shared guards, decorators, helpers, pipes, constants
│   │   │   │   ├── constants/     # Cookie names, expiration, file limits, role keys
│   │   │   │   ├── decorators/    # @CurrentUser(), @Roles()
│   │   │   │   ├── guards/        # AuthGuard (JWT cookie), RoleGuard (RBAC)
│   │   │   │   ├── helpers/       # API response envelopes, password hash, cookies
│   │   │   │   └── pipes/         # ValidationPipe (Zod), FileValidationPipe
│   │   │   ├── config/            # Environment parsing (Zod), Appwrite, Nodemailer
│   │   │   ├── generated/         # Generated Prisma client output
│   │   │   ├── modules/           # Domain feature modules
│   │   │   │   ├── amenities/     # Hotel amenities management
│   │   │   │   ├── audit-logs/    # Global mutation interceptor & audit log reader
│   │   │   │   ├── auth/          # Register, login, refresh, logout, password reset
│   │   │   │   ├── bookings/      # Booking lifecycle, reservations, availability
│   │   │   │   ├── customers/     # Customer profiles, IDs, signatures, bookings
│   │   │   │   ├── dashboard/     # Role-specific dashboard aggregations & charts
│   │   │   │   ├── emails/        # Nodemailer integration & email templates
│   │   │   │   ├── invoices/      # Invoices, billing status, cancellation
│   │   │   │   ├── issues/        # Room & hotel maintenance issue tracking
│   │   │   │   ├── notifications/ # In-app notification delivery and read tracking
│   │   │   │   ├── payments/      # Cash, QR, Razorpay orders, verify, webhooks, refunds
│   │   │   │   ├── profile/       # User profile details, avatar upload, password change
│   │   │   │   ├── public/        # Public amenities, room browsing, inquiries/contact
│   │   │   │   ├── room-types/    # Room categories, pricing, specs, image uploads
│   │   │   │   ├── rooms/         # Physical rooms, room status lifecycle, row locking
│   │   │   │   ├── staff/         # Staff directory, ID proofs, categories, signatures
│   │   │   │   ├── tasks/         # Housekeeping task assignments & status sync
│   │   │   │   └── users/         # User accounts management (Admin only)
│   │   │   ├── prisma/            # PrismaService lifecycle connection provider
│   │   │   ├── types/             # API response contracts & Prisma export types
│   │   │   ├── app.controller.ts  # Root greeting & health check (/api/v1/health)
│   │   │   ├── app.module.ts      # Root NestJS application module
│   │   │   └── main.ts            # Entrypoint (global prefix, cookies, CORS, helmet)
│   │   ├── test/                  # End-to-end integration tests
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── prisma7.config.ts      # Prisma 7 configuration file
│   │   └── tsconfig.json
│   │
│   └── web/                       # React Frontend Application
│       ├── public/                # Static public assets
│       ├── src/
│       │   ├── app/               # Router, application providers, layouts
│       │   │   ├── layout/        # PublicLayout, AuthLayout, DashboardLayout
│       │   │   └── router.tsx     # React Router v7 configuration
│       │   ├── assets/            # Static images (hero, banner, logos)
│       │   ├── components/        # Shared presentation components
│       │   │   ├── common/        # DataTable, StatCard, Header, EnumBadges, Dialogs
│       │   │   ├── sidebar/       # AppSidebar, NavMain, navigation groupings
│       │   │   └── ui/            # shadcn/ui and Base UI primitives
│       │   ├── constants/         # Page navigation definitions, hotel metadata
│       │   ├── features/          # Feature-based domain logic (CRUD hooks, keys, schemas)
│       │   │   ├── auditLog/      # Audit log viewer components & hooks
│       │   │   ├── auth/          # Auth hooks (login, logout, reset password)
│       │   │   ├── bookings/      # Booking tables, status badges, filters, wizards
│       │   │   ├── customers/     # Customer management & creation dialogs
│       │   │   ├── dashboard/     # 4 dashboard views: Management, Receptionist, etc.
│       │   │   ├── invoices/      # Invoice list, detail dialog, PDF download
│       │   │   ├── issues/        # Issue reporting dialog, status tracking
│       │   │   ├── notifications/ # Unread count badge & popover
│       │   │   ├── payments/      # Payment tables, cash/QR dialogs, receipt download
│       │   │   ├── profile/       # Profile form, avatar upload, password dialog
│       │   │   ├── public/        # Public rooms, inquiries, contact form
│       │   │   ├── rooms/         # Room cards, status filters, room type dialogs
│       │   │   ├── staff/         # Staff directory & registration dialog
│       │   │   └── tasks/         # Housekeeping task grid, stats, creation dialog
│       │   ├── hooks/             # Utility hooks (useAuth, useDebounce, useMobile)
│       │   ├── lib/               # Axios instance (apiClient), formatters, notifications
│       │   ├── pages/
│       │   │   ├── auth/          # Login, Register, ResetPassword
│       │   │   ├── protect/       # Authenticated dashboard views (Bookings, Staff, etc.)
│       │   │   └── public/        # Landing, About, Contact, Rooms, BookingCheckoutFlow
│       │   ├── providers/         # AuthProvider & theme context
│       │   └── index.css          # Tailwind CSS v4 root stylesheet
│       ├── package.json
│       ├── vercel.json            # Vercel deployment rewrite rules
│       ├── vite.config.ts
│       └── tsconfig.json
│
├── docker-compose.yaml            # PostgreSQL service definition
├── package.json                   # Root orchestrator package.json
└── README.md                      # Primary project documentation
```

---

## Implemented Features

### 1. Public Visitor Experience
- **Landing Page**: Hotel presentation featuring hero room search widget, room type highlights, categorized amenity marquee, customer testimonials, and hotel introduction.
- **Room Discovery (`/rooms`)**: Filterable room catalog by guest count, bed configuration, price bounds, and textual search.
- **Room Details (`/rooms/:slug`)**: High-resolution image galleries, architectural specifications (area in sq. ft., bed count, allowed occupancy, smoking and pet rules), amenity categorization, and instant availability calculation.
- **Public Checkout Flow (`/rooms/:slug/book` & `/checkout`)**:
  - Step 1: Guest details (full name, email, phone number, address, optional auto-create customer account toggle, and accompanying guest list with ages and gender).
  - Step 2: Payment options (online payment checkout initiated through Razorpay with public checkout tokens).
  - Step 3: Instant booking confirmation screen displaying reference numbers, stay breakdown, and confirmation details.
- **Contact & Inquiry Page (`/contact`)**: Public inquiry form storing entries in the database (`Inquiry` model) while emailing the hotel desk via Nodemailer.

### 2. Multi-Role Dashboard
The application automatically renders a dedicated dashboard interface based on the logged-in user's role and staff category:
- **Management Dashboard (Admin & Manager)**: High-level KPI statistics (total revenue, occupancy rate, today's arrivals and departures, pending housekeeping), 30-day revenue trends chart, booking status breakdown pie chart, and live stays list.
- **Receptionist Dashboard (Staff - Receptionist)**: Stay metrics, occupancy distributions (vacant, occupied, reserved, out of order), payments requiring attention (pending payments), and rapid booking creation.
- **Housekeeper Dashboard (Staff - Housekeeper)**: Assigned room cleaning tasks (cleaning, deep clean, turndown), current room cleanliness statuses (clean, dirty, cleaning), and one-click task status updates.
- **Customer Dashboard (Customer)**: Upcoming reservation alert with stay countdown, booking history metrics, and quick shortcuts to report room maintenance issues or browse rooms.

### 3. Room & Inventory Operations
- **Room Types**: Specification of base prices, currency (INR), bed type, bed count, room dimensions, smoking/pets permissions, and multiple image uploads to Appwrite storage.
- **Rooms**: Individual room inventory mapped to room types, floor numbers, room numbers, occupancy status (`VACANT`, `RESERVED`, `OCCUPIED`, `OUT_OF_ORDER`), and housekeeping status (`CLEAN`, `DIRTY`, `CLEANING`).
- **Concurrency & Double-Booking Protection**: Room reservation validation performs PostgreSQL row-level locks (`SELECT "id" FROM "Room" WHERE "id" = $id FOR UPDATE`) to guarantee atomic room assignments during concurrent booking submissions.
- **Amenities Management**: Categorized amenity catalog (`ROOM`, `BATHROOM`, `FOOD`, `ENTERTAINMENT`, `CONNECTIVITY`, `COMFORT`, `SAFETY`, `OTHER`) with Lucide icon associations linked to room types.

### 4. Booking & Reservation Management
- **Centralized Booking Ledger**: Searchable, paginated booking directory filterable by status (`PENDING`, `CONFIRMED`, `CHECKED_IN`, `CHECKED_OUT`, `CANCELLED`, `NO_SHOW`), date ranges, and reference IDs.
- **Walk-In Booking Wizard**: 5-step internal booking flow for front desk staff: Customer selection -> Stay dates and room assignment -> Accompanying guests -> Payment recording -> Review & confirmation.
- **Booking State Machine**: Enforces valid status transitions; checking in a guest marks the room as `OCCUPIED`, while checking out automatically marks the room as `DIRTY` and triggers housekeeping requirements.

### 5. Housekeeping & Task Automation
- **Task Scheduling**: Creation and assignment of tasks (`CLEANING`, `DEEP_CLEAN`, `TURNDOWN`) to specific housekeepers.
- **Automated Room Cleanliness Synchronization**:
  - Setting a task to `IN_PROGRESS` automatically transitions the associated room to `CLEANING`.
  - Marking all active tasks on a room as `COMPLETED` automatically switches the room status to `CLEAN`.
  - Abandoning or cancelling tasks reverts the room status to `DIRTY`.

### 6. Maintenance & Issue Tracking
- **Issue Logging**: Issues can be logged by guests for their assigned room or by hotel staff, categorized by trade (`PLUMBING`, `ELECTRICAL`, `HVAC`, `FURNITURE`, `APPLIANCE`, `INTERNET`, `BATHROOM`, `DOOR_LOCK`, `LIGHTING`, `OTHER`) and priority (`LOW`, `MEDIUM`, `HIGH`, `URGENT`).
- **Workflow & Proofs**: Tracks issue lifecycle (`OPEN` -> `IN_PROGRESS` -> `COMPLETED` -> `RESOLVED` / `CANCELLED`), allows image uploads, records assigned maintenance staff, and logs timestamps for started, completed, and resolved events.

### 7. Invoicing & Billing
- **Automatic Invoice Generation**: Every confirmed booking automatically generates a unique `Invoice` record with subtotal, taxes, discounts, and total amount.
- **Invoice Status Lifecycle**: Dynamically reflects payment state (`UNPAID`, `PARTIALLY_PAID`, `PAID`, `REFUNDED`).
- **Client-Side PDF Generation**: Exports professional, print-ready PDF invoices directly in the browser via `@react-pdf/renderer`.

### 8. Payments & Financial Reconciliation
- **Multiple Payment Channels**:
  - **Cash**: Direct recording by front desk staff for in-person transactions.
  - **QR / UPI**: Offline or dynamic QR code transaction recording with reference IDs.
  - **Online (Razorpay)**: Automated order creation (`orders` API), client-side modal checkout, and cryptographic HMAC-SHA256 signature verification.
- **Webhook Reconciliation**: Secure webhook endpoint (`/api/v1/payments/webhook`) receiving raw bodies and validating `x-razorpay-signature` and `x-razorpay-event-id` to protect against double-processing.
- **Refund Management**: Admins and Managers can trigger payment refunds, automatically updating the payment status to `REFUNDED` and recalculating the associated invoice status.
- **Receipts**: In-browser client-side PDF download for payment receipts.

### 9. Auditing & Security
- **Automated Audit Logs**: A global NestJS interceptor (`AuditLogsInterceptor`) automatically intercepts all mutating operations (`POST`, `PUT`, `PATCH`, `DELETE`), filters payloads against allowed audited fields, and records user IDs, roles, actions, entity types, entity IDs, and before/after values into the database.
- **Audit Viewer**: Admin-only interactive table with search, entity filters, and a JSON inspector for data changes.

### 10. In-App Notifications
- Notification generation for booking confirmations, check-in reminders, payment receipts, task assignments, and issue updates.
- Real-time unread counter badge and popover dropdown with mark-as-read and mark-all-as-read capabilities.

---

## User Roles and Permissions

The system defines four principal user roles (`UserRole` enum) and four operational staff categories (`Category` enum):

```
User Roles:
├── ADMIN
├── MANAGER
├── STAFF
│   ├── RECEPTIONIST
│   ├── HOUSEKEEPER
│   ├── SECURITY_GUARD
│   └── WAITER
└── CUSTOMER
```

### Role Capabilities Matrix

| Functional Module | ADMIN | MANAGER | STAFF (Receptionist) | STAFF (Housekeeper) | CUSTOMER |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **System Dashboard** | Management | Management | Receptionist | Housekeeper | Customer |
| **User Accounts Management** | Full Access | No Access | No Access | No Access | No Access |
| **Staff Directory** | Full Access | View / Update | No Access | No Access | No Access |
| **Customer Records** | Full Access | Full Access | Full Access | No Access | Own Profile |
| **Room Inventory Management** | Full Access | Edit / Status | View Only | View / Cleanliness | No Access |
| **Room Types & Pricing** | Full Access | View Only | View Only | No Access | View Public |
| **Amenities** | Full Access | Edit Only | View Only | No Access | View Public |
| **Bookings & Reservations** | Full Access | Full Access | Full Access | No Access | Own Bookings |
| **Walk-In Booking Wizard** | Yes | Yes | Yes | No | No |
| **Housekeeping Tasks** | Full Access | Full Access | View Only | Assigned Tasks | No Access |
| **Issue Management** | Full Access | Full Access | View Only | View Only | Own Room Issues |
| **Invoices & Billing** | Full Access | Full Access | View Only | No Access | Own Invoices |
| **Record Cash / QR Payments** | Yes | Yes | Yes | No | No |
| **Online Payments** | Yes | Yes | Yes | No | Yes (Razorpay) |
| **Issue Refunds** | Yes | Yes | No | No | No |
| **Inquiries Management** | Full Access | Full Access | No Access | No Access | Submit Inquiries |
| **Audit Logs Viewer** | Full Access | No Access | No Access | No Access | No Access |
| **Profile Management** | Own Profile | Own Profile | Own Profile | Own Profile | Own Profile |

---

## Authentication and Authorization

### Dual-Token Cookie Strategy
Authentication uses JSON Web Tokens (JWT) stored exclusively in secure, HTTP-only cookies to mitigate XSS vulnerabilities:
- **`access_token`**: Short-lived token (1 hour) containing user ID, email, role, and staff category.
- **`refresh_token`**: Long-lived random token (7 days) stored hashed in the `RefreshToken` database table.

```
+-------------+                 +--------------+                 +---------------+
| React App   |                 | NestJS API   |                 | PostgreSQL DB |
+-------------+                 +--------------+                 +---------------+
       |                               |                                 |
       |--- POST /auth/login --------->|                                 |
       |    (email, password)          |--- Verify Argon2 hash --------->|
       |                               |<-- User record with role -------|
       |<-- Set-Cookie (HTTP-only) ----|                                 |
       |    access_token & refresh_tok |--- Save RefreshToken hash ----->|
       |                               |                                 |
       |--- API Request -------------->|                                 |
       |    (Browser sends cookies)    |--- AuthGuard validates JWT -----|
       |                               |--- RoleGuard checks RBAC -------|
       |<-- 200 OK with Data ----------|                                 |
       |                               |                                 |
       |--- 401 Unauthorized --------->|                                 |
       |    (access token expired)     |                                 |
       |--- POST /auth/refresh ------->|--- Verify RefreshToken -------->|
       |<-- Set new cookie ------------|<-- Issue new rotated tokens ----|
       |--- Retry original request --->|                                 |
```

### Automatic Token Refresh Queue
The frontend Axios client (`apps/web/src/lib/apiClient.ts`) implements a response interceptor with a mutex/promise lock:
1. When any API request fails with a `401 Unauthorized`, it halts subsequent requests.
2. An initial call to `POST /api/v1/auth/refresh` is dispatched.
3. Concurrent requests subscribe to that single in-flight refresh promise.
4. Once tokens are renewed, original requests are re-executed with the updated session cookie.

### Multi-Tier Authorization
- **Guard Layer**: Every protected endpoint applies `@UseGuards(AuthGuard, RoleGuard)` and evaluates the allowed roles declared via `@Roles('ADMIN', 'MANAGER', ...)`.
- **Service Layer**: Fine-grained business restrictions verify staff categories. For example, `BookingsService.assertBookingAccess()` enforces that `STAFF` users must have the category `RECEPTIONIST` to create or update bookings.

---

## Core Application Workflows

### 1. Public Guest Booking & Razorpay Payment

```mermaid
sequenceDiagram
    autonumber
    actor Guest as Public Guest
    participant Web as Web (React)
    participant API as API (NestJS)
    participant RZP as Razorpay Gateway
    participant DB as PostgreSQL

    Guest->>Web: Select dates, room type, enter guest list
    Web->>API: POST /api/v1/bookings/public
    API->>DB: Lock room, verify cleanliness, create Booking (PENDING)
    API-->>Web: Returns bookingReference, bookingId, checkoutToken
    Web->>API: POST /api/v1/payments/public/orders (bookingId, checkoutToken)
    API->>RZP: razorpay.orders.create(amount, currency)
    RZP-->>API: Returns order_id
    API->>DB: Record Payment attempt (PENDING, orderId)
    API-->>Web: Returns Razorpay order details & Key ID
    Web->>RZP: Open Razorpay checkout modal
    Guest->>RZP: Completes payment
    RZP-->>Web: Returns payment_id, order_id, signature
    Web->>API: POST /api/v1/payments/public/verify (signature, checkoutToken)
    API->>API: Verify HMAC-SHA256 signature
    API->>DB: Confirm booking, assign clean room, update Invoice to PAID
    API-->>Web: Verification confirmed
    Web-->>Guest: Displays booking confirmation and receipt download
```

### 2. Room State Lifecycle and Concurrency Locking

```mermaid
stateDiagram-v2
    [*] --> VACANT
    VACANT --> RESERVED: Confirmed reservation created
    RESERVED --> OCCUPIED: Guest check-in completed
    VACANT --> OCCUPIED: Direct walk-in check-in
    OCCUPIED --> VACANT: Check-out (if room remains CLEAN)
    OCCUPIED --> OUT_OF_ORDER: Maintenance issue logged
    VACANT --> OUT_OF_ORDER: Placed out of order
    OUT_OF_ORDER --> VACANT: Released out of order

    note right of VACANT
        Cleanliness states:
        - CLEAN (bookable)
        - DIRTY (requires cleaning)
        - CLEANING (task in progress)
    end note
```

---

## API Route Catalog

All backend routes are prefixed with `/api/v1`.

### Authentication (`/api/v1/auth`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/auth/register` | Public | Register a new customer account |
| `POST` | `/auth/login` | Public | Authenticate with email & password, sets cookies |
| `POST` | `/auth/refresh` | Public (Cookie) | Rotate refresh token and issue fresh access token |
| `POST` | `/auth/logout` | Authenticated | Revoke refresh token and clear cookies |
| `POST` | `/auth/forgot-password` | Public | Send a 30-minute password reset link via email |
| `POST` | `/auth/reset-password?token=` | Public | Reset password using the one-time token |

### Profile & Account (`/api/v1/profile`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/profile/me` | Authenticated | Retrieve current user profile summary |
| `GET` | `/profile/details` | Authenticated | Retrieve profile details with staff/customer records |
| `PATCH` | `/profile/me` | Authenticated | Update full name, phone number, and address |
| `PATCH` | `/profile/avatar` | Authenticated | Upload profile avatar image to Appwrite storage |
| `DELETE`| `/profile/avatar` | Authenticated | Remove current profile avatar |
| `PATCH` | `/profile/change-password` | Authenticated | Update account password (verifies current password) |

### System Users (`/api/v1/users`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/users/stats` | ADMIN | Aggregated user metrics |
| `GET` | `/users` | ADMIN | Paginated list of all system users |
| `GET` | `/users/:id` | ADMIN | Retrieve single user record by ID |
| `POST` | `/users` | ADMIN | Create new user account |
| `PATCH` | `/users/:id` | ADMIN | Update user account details |
| `PATCH` | `/users/:id/status` | ADMIN | Activate or deactivate user account |
| `DELETE`| `/users/:id` | ADMIN | Soft-delete user account |

### Staff Directory (`/api/v1/staff`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/staff/stats` | ADMIN, MANAGER | Staff count metrics by category |
| `GET` | `/staff` | ADMIN, MANAGER | Paginated staff members directory |
| `GET` | `/staff/lookup` | ADMIN, MANAGER | Searchable lookup list for assignments |
| `GET` | `/staff/:id` | ADMIN, MANAGER | Single staff record with ID proof & signature |
| `POST` | `/staff` | ADMIN, MANAGER | Create staff record with document uploads |
| `PATCH` | `/staff/:id` | ADMIN, MANAGER | Update staff metadata |
| `PATCH` | `/staff/:id/status`| ADMIN, MANAGER | Toggle staff active/inactive status |
| `DELETE`| `/staff/:id` | ADMIN | Soft-delete staff member |

### Customers (`/api/v1/customers`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/customers/stats` | ADMIN, MANAGER, STAFF | Customer total counts and metrics |
| `GET` | `/customers` | ADMIN, MANAGER, STAFF | Paginated list of hotel customers |
| `GET` | `/customers/search`| ADMIN, MANAGER, STAFF | Autocomplete search for front-desk booking |
| `GET` | `/customers/:id` | ADMIN, MANAGER, STAFF | Retrieve customer profile by ID |
| `GET` | `/customers/:id/bookings` | ADMIN, MANAGER, STAFF | Customer booking history |
| `GET` | `/customers/:id/payments` | ADMIN, MANAGER, STAFF | Customer payment transaction history |
| `POST` | `/customers` | ADMIN | Manually create customer record |
| `PATCH` | `/customers/:id` | ADMIN, MANAGER | Update customer profile details |
| `PATCH` | `/customers/:id/status` | ADMIN, MANAGER | Toggle customer status |
| `DELETE`| `/customers/:id` | ADMIN | Delete customer record |

### Room Types (`/api/v1/room-types`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/room-types` | Authenticated | Paginated room types list |
| `GET` | `/room-types/:slug` | ADMIN, MANAGER, STAFF | Detailed room type record by slug |
| `POST` | `/room-types` | ADMIN | Create room type with images and amenities |
| `PATCH` | `/room-types/:id` | ADMIN | Update room type details and image gallery |

### Physical Rooms (`/api/v1/rooms`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/rooms/stats` | ADMIN, MANAGER, STAFF | Room occupancy and cleanliness statistics |
| `GET` | `/rooms` | ADMIN, MANAGER, STAFF | Paginated room inventory list |
| `GET` | `/rooms/lookup` | ADMIN, MANAGER, STAFF | Filterable room lookup list |
| `GET` | `/rooms/:id` | ADMIN, MANAGER, STAFF | Room details with active assignment |
| `POST` | `/rooms` | ADMIN | Add physical room to inventory |
| `PATCH` | `/rooms/:id` | ADMIN, MANAGER | Update room number, floor, or status |
| `PATCH` | `/rooms/:id/release-out-of-order` | ADMIN, MANAGER | Release room from OUT_OF_ORDER status |
| `DELETE`| `/rooms/:id` | ADMIN | Remove room from inventory |

### Amenities (`/api/v1/amenities`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/amenities` | Public | Paginated list of active hotel amenities |
| `POST` | `/amenities` | ADMIN | Create new amenity item |
| `PATCH` | `/amenities/:id` | ADMIN, MANAGER | Update amenity details or category |
| `DELETE`| `/amenities/:id` | ADMIN | Delete amenity item |

### Bookings (`/api/v1/bookings`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/bookings/available-rooms` | Public | Query available rooms for specific dates |
| `POST` | `/bookings/public` | Public | Initiate guest reservation (pending checkout) |
| `GET` | `/bookings/stats` | All Roles | Booking counts by status |
| `GET` | `/bookings` | All Roles (Scoped) | Paginated list of bookings (customers see own) |
| `GET` | `/bookings/:id` | All Roles (Scoped) | Retrieve single booking details |
| `POST` | `/bookings` | All Roles (Scoped) | Create booking (staff wizard or customer) |
| `PATCH` | `/bookings/:id/status` | ADMIN, MANAGER, STAFF | Check-in, check-out, or cancel booking |

### Payments (`/api/v1/payments`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/payments/public/orders` | Public | Generate Razorpay order for guest checkout |
| `POST` | `/payments/public/verify` | Public | Verify Razorpay payment signature for guest |
| `POST` | `/payments/public/status` | Public | Retrieve public payment status by token |
| `POST` | `/payments/webhook` | Public (Signed) | Process Razorpay webhook notifications |
| `GET` | `/payments/stats` | ADMIN, MANAGER, STAFF | Payment revenue and collection metrics |
| `GET` | `/payments` | All Roles (Scoped) | Paginated payments list |
| `GET` | `/payments/:id` | All Roles (Scoped) | Single payment record details |
| `POST` | `/payments/cash` | ADMIN, MANAGER, STAFF | Record in-person cash payment |
| `POST` | `/payments/qr` | ADMIN, MANAGER, STAFF | Record offline QR/UPI payment |
| `POST` | `/payments/orders` | Authenticated | Create online payment order for booking |
| `POST` | `/payments/verify` | Authenticated | Verify Razorpay online payment signature |
| `POST` | `/payments/:id/refund` | ADMIN, MANAGER | Refund payment and update invoice status |

### Invoices (`/api/v1/invoices`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/invoices/stats` | ADMIN, MANAGER, STAFF | Invoice financial summaries |
| `GET` | `/invoices` | All Roles (Scoped) | Paginated list of invoices |
| `GET` | `/invoices/:id` | All Roles (Scoped) | Detailed invoice breakdown |
| `PATCH` | `/invoices/:id` | ADMIN, MANAGER | Adjust invoice discount or subtotal |
| `POST` | `/invoices/:id/cancel` | ADMIN, MANAGER | Void/cancel invoice |

### Housekeeping Tasks (`/api/v1/tasks`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/tasks/stats` | ADMIN, MANAGER, STAFF | Task metrics (pending, in-progress, done) |
| `GET` | `/tasks` | ADMIN, MANAGER, STAFF | Paginated housekeeping task list |
| `POST` | `/tasks` | ADMIN, MANAGER | Assign cleaning task to housekeeper |
| `PATCH` | `/tasks/:id` | ADMIN, MANAGER | Update task type, room, or assignee |
| `PATCH` | `/tasks/:id/status`| ADMIN, MANAGER, STAFF | Update task status (syncs room status) |
| `DELETE`| `/tasks/:id` | ADMIN, MANAGER | Delete scheduled task |

### Maintenance Issues (`/api/v1/issues`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/issues/stats` | ADMIN, MANAGER, CUSTOMER | Priority breakdown of issues |
| `GET` | `/issues` | ADMIN, MANAGER, CUSTOMER | Paginated list of reported issues |
| `GET` | `/issues/:id` | ADMIN, MANAGER, CUSTOMER | Issue details and status history |
| `POST` | `/issues` | ADMIN, MANAGER, CUSTOMER | Report new issue with image proof |
| `PATCH` | `/issues/:id` | ADMIN, MANAGER | Assign issue, update priority or status |

### Dashboard Metrics (`/api/v1/dashboard`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/dashboard/stats` | All Roles | Role-tailored KPI cards |
| `GET` | `/dashboard/room-status` | ADMIN, MANAGER, Receptionist | Occupancy and cleaning status distribution |
| `GET` | `/dashboard/next-booking`| CUSTOMER | Customer's next upcoming reservation |
| `GET` | `/dashboard/revenue-trend` | ADMIN, MANAGER | 30-day/custom range revenue trend data |
| `GET` | `/dashboard/booking-status` | ADMIN, MANAGER, Receptionist | Distribution of bookings by status |
| `GET` | `/dashboard/stays` | ADMIN, MANAGER, Receptionist | Today's arrivals, departures, and stayovers |
| `GET` | `/dashboard/tasks` | STAFF (Housekeeper) | Assigned housekeeping queue |
| `GET` | `/dashboard/payments` | STAFF (Receptionist) | Payments requiring attention |

### Inquiries & Public (`/api/v1/public`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/public/rooms` | Public | Available room types for public browsing |
| `GET` | `/public/rooms/:slug` | Public | Single available room type details |
| `GET` | `/public/amenities` | Public | List active amenities |
| `POST` | `/public/contact` | Public | Submit contact message (stores and emails) |
| `GET` | `/public/inquiries` | ADMIN, MANAGER | Review customer inquiries |
| `DELETE`| `/public/inquiries/:id` | ADMIN, MANAGER | Remove inquiry record |

### Notifications (`/api/v1/notifications`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/notifications/unread` | Authenticated | Retrieve unread notifications |
| `PATCH` | `/notifications/:id/read` | Authenticated | Mark individual notification as read |
| `PATCH` | `/notifications/read-all` | Authenticated | Mark all notifications as read |

### Audit Logs (`/api/v1/audit-logs`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/audit-logs` | ADMIN | Searchable audit trail of data mutations |

### System Health (`/api/v1`)

| Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/` | Public | Welcome message |
| `GET` | `/health` | Public | Service health verification |

---

## Database Schema and Models

The application database is modeled in `apps/api/prisma/schema.prisma` and executed via PostgreSQL.

### Core Relational Models

- **`User`**: Base identity storing `fullName`, `email`, `phone`, `passwordHash` (Argon2), `role` (`UserRole`), `isActive`, and soft deletion flags.
- **`Staff`**: 1-to-1 extension of `User` for hotel employees. Stores `fatherName`, `motherName`, `qualification`, `experience`, `category` (`Category`), `emergencyContact`, `address`, and Appwrite image links (`idProofImageId`, `signatureImageId`).
- **`Customer`**: 1-to-1 extension of `User` for hotel guests. Stores ID proof numbers, addresses, and document image references.
- **`RoomType`**: Defines room categories with `slug`, dimensions (`sizeSqFt`), capacity (`maxGuests`, `adults`, `children`), `basePrice`, `currency`, `bedType`, `bedCount`, `smokingAllowed`, and `petsAllowed`.
- **`Room`**: Physical room inventory referencing `RoomType`. Tracks `roomNumber`, `floor`, `occupancyStatus`, and `housekeepingStatus`.
- **`Amenity`**: Hotel amenities mapped many-to-many with `RoomType`.
- **`Booking`**: Reservation record with `bookingReference`, date bounds (`checkInDate`, `checkOutDate`), `totalGuests`, `totalAmount`, `paymentMethod`, `status` (`BookingStatus`), and guest checkout token hashes.
- **`BookingRoom`**: 1-to-1 bridge between `Booking`, `RoomType`, and assigned physical `Room`.
- **`BookingGuest`**: Accompanying guests linked to a booking (`fullName`, `age`, `gender`, `idProofNumber`).
- **`HousekeepingTask`**: Maintenance task assigning a `Room` to a housekeeper `User`. Tracks `taskType`, `status`, and scheduled/completed timestamps.
- **`HousekeeperRoomAssignment`**: Standing room-to-housekeeper mapping.
- **`Invoice`**: Billing ledger linked to `Booking`. Tracks `invoiceNumber`, `subtotal`, `taxAmount`, `discountAmount`, `totalAmount`, and `status`.
- **`Payment`**: Financial transaction linked to `Invoice`. Stores `paymentReference`, `transactionId`, `orderId`, `amount`, `paymentMethod`, `paymentStatus`, and recorder ID.
- **`PaymentWebhookEvent`**: Idempotency ledger for incoming Razorpay webhook events (`eventId`, `eventType`).
- **`Issue`**: Hotel maintenance ticket linked to `Room`, `reporter`, and optional `assignee`. Tracks `category`, `priority`, `status`, timestamps, and proof image.
- **`Notification` & `NotificationRead`**: In-app alert system with user-specific read state tracking.
- **`Image`**: Centralized image ledger storing Appwrite URLs, alt text, and relation to room types, avatars, or issue attachments.
- **`RefreshToken` & `PasswordResetToken`**: Cryptographic token ledgers for session rotation and password recovery.
- **`AuditLog`**: Mutation ledger storing actor, action, entity type, entity ID, and JSON snapshots of state changes.
- **`Inquiry`**: Public contact form submissions.
- **`Review`**: Customer booking reviews model (defined in Prisma schema).

### Primary Enums

- **`UserRole`**: `ADMIN`, `MANAGER`, `STAFF`, `CUSTOMER`
- **`Category`** (Staff): `RECEPTIONIST`, `HOUSEKEEPER`, `SECURITY_GUARD`, `WAITER`
- **`OccupancyStatus`**: `VACANT`, `RESERVED`, `OCCUPIED`, `OUT_OF_ORDER`
- **`HousekeepingStatus`**: `CLEAN`, `DIRTY`, `CLEANING`
- **`BookingStatus`**: `PENDING`, `CONFIRMED`, `CHECKED_IN`, `CHECKED_OUT`, `CANCELLED`, `NO_SHOW`
- **`TaskType`**: `CLEANING`, `DEEP_CLEAN`, `TURNDOWN`
- **`TaskStatus`**: `PENDING`, `IN_PROGRESS`, `COMPLETED`
- **`InvoiceStatus`**: `UNPAID`, `PARTIALLY_PAID`, `PAID`, `REFUNDED`
- **`PaymentMethod`**: `CASH`, `ONLINE`
- **`PaymentStatus`**: `PENDING`, `COMPLETED`, `FAILED`, `REFUNDED`
- **`IssueCategory`**: `PLUMBING`, `ELECTRICAL`, `HVAC`, `FURNITURE`, `APPLIANCE`, `INTERNET`, `BATHROOM`, `DOOR_LOCK`, `LIGHTING`, `OTHER`
- **`IssuePriority`**: `LOW`, `MEDIUM`, `HIGH`, `URGENT`
- **`IssueStatus`**: `OPEN`, `IN_PROGRESS`, `COMPLETED`, `RESOLVED`, `CANCELLED`
- **`BedType`**: `SINGLE`, `TWIN`, `DOUBLE`, `QUEEN`, `KING`, `BUNK`, `SOFA_BED`
- **`AmenityCategory`**: `ROOM`, `BATHROOM`, `FOOD`, `ENTERTAINMENT`, `CONNECTIVITY`, `COMFORT`, `SAFETY`, `OTHER`
- **`NotificationType`**: `BOOKING_CONFIRMED`, `CHECK_IN_REMINDER`, `PAYMENT_RECEIVED`, `ISSUE_CREATED`, `ISSUE_ASSIGNED`, `ISSUE_UPDATED`, `HOUSEKEEPING_ASSIGNED`, `BOOKING_CANCELLED`

---

## Environment Configuration

### Backend (`apps/api/.env`)

Validated via Zod schema in `apps/api/src/config/env.config.ts`:

| Variable Name | Type / Requirement | Default | Description |
| :--- | :--- | :--- | :--- |
| `NODE_ENV` | `development` \| `production` | `development` | Application runtime environment |
| `PORT` | Number | `3000` | HTTP listening port (e.g. `5000`) |
| `HOST_NAME` | String (Required) | None | Host interface binding (e.g. `0.0.0.0` or `localhost`) |
| `HOTEL_TIME_ZONE` | Valid IANA Timezone | `Asia/Kolkata` | Timezone for operational schedules and audit logs |
| `CORS_ORIGIN` | Valid URL String | None | Permitted client origin (e.g. `http://localhost:5173`) |
| `JWT_ACCESS_SECRET` | String (min 32 chars) | None | Cryptographic secret for signing JWT access tokens |
| `DATABASE_URL` | Valid PostgreSQL URL | None | Database connection string |
| `DATABASE_POOL_MAX`| Positive Integer | `10` | Maximum connections in the PostgreSQL client pool |
| `APPWRITE_ENDPOINT`| Valid URL String | None | Appwrite API endpoint URL |
| `APPWRITE_PROJECT_ID`| String (Required) | None | Appwrite project identifier |
| `APPWRITE_API_KEY` | String (Required) | None | Appwrite server API key with storage permissions |
| `APPWRITE_BUCKET_ID`| String (Required) | None | Target Appwrite storage bucket ID |
| `APPWRITE_BUCKET_NAME`| String (Required) | None | Target Appwrite bucket label |
| `EMAIL_USER` | String (Required) | None | Gmail/SMTP sender address |
| `EMAIL_PASS` | String (Required) | None | Gmail app password or SMTP credential |
| `RAZORPAY_KEY_ID` | String (Required) | None | Razorpay gateway public Key ID |
| `RAZORPAY_KEY_SECRET` | String (Required) | None | Razorpay gateway API secret |
| `RAZORPAY_WEBHOOK_SECRET` | String (Required) | None | Webhook secret for HMAC signature validation |

### Frontend (`apps/web/.env`)

Read via `import.meta.env` in `apps/web/src/constants/hotel.ts` and `apiClient.ts`:

| Variable Name | Requirement | Default | Description |
| :--- | :--- | :--- | :--- |
| `VITE_API_URL` | Required | `http://localhost:5000/api/v1` | Base URL pointing to the NestJS API |
| `VITE_HOTEL_NAME` | Optional | `Nivara` | Hotel display brand name |
| `VITE_HOTEL_LOCATION` | Optional | `Lucknow, Uttar Pradesh, India` | Hotel geographic location |
| `VITE_HOTEL_ADDRESS` | Optional | None | Physical hotel street address |
| `VITE_HOTEL_PHONE` | Optional | None | Front desk telephone contact |
| `VITE_HOTEL_EMAIL` | Optional | None | Hotel contact email address |
| `VITE_HOTEL_HOURS` | Optional | None | Reception and desk operational hours |

---

## Installation and Local Setup

### Prerequisites
- **Node.js**: `v20.x` or higher
- **pnpm**: `v9.x` or higher
- **Docker**: Docker Engine & Docker Compose (for local PostgreSQL database)

### Step 1: Clone and Install Dependencies

```bash
git clone https://github.com/teamiglobexinfotech-star/nivara-hotels
cd nivara-hotels

# Install dependencies for both applications
pnpm --dir apps/api install
pnpm --dir apps/web install
pnpm install
```

### Step 2: Start PostgreSQL Database

Start the bundled PostgreSQL 16 container:

```bash
docker compose up -d
```

This launches a PostgreSQL container named `postgres_nivara` exposed on port `5432` with database `nivara_hotels`.

### Step 3: Configure Environment Variables

Create `.env` files for both projects using the provided examples:

```bash
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env
```

Ensure `DATABASE_URL` in `apps/api/.env` matches your local database instance:
```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/nivara_hotels?schema=public"
```

### Step 4: Run Database Migrations and Generate Prisma Client

```bash
cd apps/api
pnpm prisma migrate dev
pnpm prisma generate
cd ../..
```

### Step 5: Start the Development Servers

From the project root, launch both applications concurrently:

```bash
pnpm run dev
```

Alternatively, run each service in separate terminal windows:

```bash
# Terminal 1: Backend API (runs on port 5000, or PORT specified in .env)
pnpm run dev:api

# Terminal 2: Frontend Web App (runs on http://localhost:5173)
pnpm run dev:web
```

---

## Available Scripts and Commands

### Root Workspace (`package.json`)

| Command | Action |
| :--- | :--- |
| `pnpm run dev` | Runs both API and Web concurrently using `concurrently` |
| `pnpm run dev:api` | Runs `pnpm --dir apps/api start:dev` |
| `pnpm run dev:web` | Runs `pnpm --dir apps/web dev` |

### Backend API (`apps/api/package.json`)

| Command | Action |
| :--- | :--- |
| `pnpm --dir apps/api start:dev` | Starts NestJS API in watch mode |
| `pnpm --dir apps/api build` | Compiles NestJS TypeScript application into `apps/api/dist` |
| `pnpm --dir apps/api start:prod` | Runs production build via `node dist/main` |
| `pnpm --dir apps/api start` | Starts NestJS without watch mode |
| `pnpm --dir apps/api postinstall` | Automatically executes `prisma generate` |
| `pnpm --dir apps/api lint` | Executes ESLint with autofix across `src/` and `test/` |
| `pnpm --dir apps/api format` | Formats code with Prettier |


### Frontend Web (`apps/web/package.json`)

| Command | Action |
| :--- | :--- |
| `pnpm --dir apps/web dev` | Starts Vite development server bound to `0.0.0.0` |
| `pnpm --dir apps/web build` | Executes TypeScript typecheck (`tsc -b`) and Vite production bundle |
| `pnpm --dir apps/web typecheck` | Checks TypeScript types without emitting code (`tsc --noEmit`) |
| `pnpm --dir apps/web preview` | Locally serves the production build from `apps/web/dist` |
| `pnpm --dir apps/web lint` | Lints TypeScript and JSX files |
| `pnpm --dir apps/web format` | Formats source files with Prettier |

---

## Build and Deployment

### Production Compilation
Both applications compile cleanly without type or bundling errors:

```bash
# Compile backend
pnpm --dir apps/api build

# Compile frontend
pnpm --dir apps/web build
```

### Deployment Strategy
1. **API Hosting**: The NestJS API compiles to standard CommonJS output in `apps/api/dist/main.js`. It can be run on any Node.js hosting platform (Docker, Railway, Render, EC2, VPS) via `node dist/main` with environment variables supplied.
2. **Web Hosting**: The frontend builds static assets into `apps/web/dist`. `apps/web/vercel.json` provides rewrite configuration for Single-Page Application (SPA) routing:
   ```json
   {
     "rewrites": [
       { "source": "/(.*)", "destination": "/index.html" }
     ]
   }
   ```
3. **Database & Storage**: PostgreSQL runs externally or in a managed database service; files are stored remotely in Appwrite Storage buckets.

---

## Codebase Verification and Audit Notes

During technical inspection of the repository, all claims in this document were verified directly against the source code:

1. **Role and Category Realities**: Staff categories (`RECEPTIONIST`, `HOUSEKEEPER`, `SECURITY_GUARD`, `WAITER`) are strictly enforced at the business logic layer. Receptionists handle front-desk bookings and payments, Housekeepers have dedicated task and room status access, while Admins/Managers retain administrative management.
2. **Client-Side PDF Generation**: Invoices and payment receipts are rendered client-side using `@react-pdf/renderer` inside React components (`InvoiceDownload.tsx` and `PaymentDownload.tsx`).
3. **Double-Booking Protection**: The reservation engine in `apps/api/src/modules/rooms/room-status-lifecycle.ts` uses explicit PostgreSQL row locks (`SELECT ... FOR UPDATE`) inside transactions to eliminate race conditions.
4. **Status Lifecycle Automation**: Task status transitions directly alter room cleanliness (`IN_PROGRESS` -> `CLEANING`, `COMPLETED` -> `CLEAN`, abandoned -> `DIRTY`).
