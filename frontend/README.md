# 🏢 SmartSociety — Complete Full-Stack PropTech & Society Management Platform

> **A production-ready, enterprise-grade Housing Society & Gated Community Operating System.**  
> Built with **React 19**, **Tailwind CSS v4**, **GSAP 3.15**, **Node.js/Express**, and **MongoDB**.

---

## 📑 Table of Contents

1. [Executive Overview & Vision](#1-executive-overview--vision)
2. [High-Level System Architecture & ERD](#2-high-level-system-architecture--erd)
3. [RBAC Access Control Matrix](#3-rbac-access-control-matrix)
4. [Frontend Architecture & UI/UX Stack](#4-frontend-architecture--uiux-stack)
5. [Backend Micro-Modules & Database Models](#5-backend-micro-modules--database-models)
6. [Complete REST API Specification](#6-complete-rest-api-specification)
7. [Marketing Landing Page Modules](#7-marketing-landing-page-modules)
8. [Form Validation & RFC Regex Engine](#8-form-validation--rfc-regex-engine)
9. [Export, Printing & Media Handling](#9-export-printing--media-handling)
10. [Step-by-Step Installation & Setup](#10-step-by-step-installation--setup)
11. [Environment Variables Reference](#11-environment-variables-reference)
12. [Default Seed Credentials](#12-default-seed-credentials)
13. [Project Directory Tree](#13-project-directory-tree)

---

## 1. Executive Overview & Vision

Managing a modern residential complex or housing society involves multiple stakeholders working simultaneously:
* **Residents** who demand transparent dues billing, quick complaint resolutions, and frictionless guest entry.
* **Security Guards** who need split-second visitor QR code verification without paper registers.
* **Maintenance Staff** who need clear ticket assignments, priority SLA timers, and proof-of-work tracking.
* **Society Administrators** who need 360° financial visibility, flat inventory occupancy maps, automated monthly invoicing, and emergency broadcast tools.

**SmartSociety** unifies all these operations into a cohesive, high-performance platform featuring sub-second API responses, offline-ready camera QR scanning, GSAP parallax animations, and strict RFC data validations.

---

## 2. High-Level System Architecture & ERD

### 2.1 Layered Architecture Flow

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      Client Browser (React 19 SPA)                     │
│  ┌───────────────────────┬─────────────────────────┬──────────────────┐ │
│  │ GSAP Marketing Pages  │  Role Dashboards (RBAC) │ Live Camera QR   │ │
│  │ Dues Lookup Widget    │  Tailwind CSS v4 Engine │ Confetti Modals  │ │
│  └───────────────────────┴─────────────────────────┴──────────────────┘ │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ HTTPS / REST (JSON)
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                     Node.js & Express REST Backend                      │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │ Middlewares: JWT Auth Guard · Role Guard (RBAC) · Multer Uploads  │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┬────────┐ │
│  │ /auth        │ /users       │ /flats       │ /visitors    │ /bills │ │
│  ├──────────────┼──────────────┼──────────────┼──────────────┼────────┤ │
│  │ /complaints  │ /notices     │ /amenities   │ /emergency   │ /audit │ │
│  └──────────────┴──────────────┴──────────────┴──────────────┴────────┘ │
└────────────────────────────────────┬────────────────────────────────────┘
                                     │ Mongoose ODM
                                     ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      MongoDB Database Layer                             │
│  [Users] ──1:N── [Flats] ──1:N── [Bills] ──1:N── [Complaints]          │
│  [Users] ──1:N── [VisitorPasses] ──1:N── [GateLogs]                     │
│  [Users] ──1:N── [AmenityBookings] ──1:N── [PollVotes]                  │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Entity Relationship Diagram (Mongoose Models)

```
 +-------------------------------------------------------------------------+
 |                                  USER                                   |
 |  _id (PK) · name · email · passwordHash · phone · role · avatar · isActive|
 |  vehicles: [{ type, vehicleNumber }] · emergencyContacts: [{ name... }] |
 +-------------------------------------------------------------------------+
       │ 1                                      │ 1
       │ owns / resides in                      │ raises / assigns
       ▼ N                                      ▼ N
 +---------------------------+            +-------------------------------+
 |           FLAT            |            |           COMPLAINT           |
 |  _id (PK) · blockName     |            |  _id (PK) · title · category  |
 |  flatNumber (Unique)      |            |  description · priority (SLA) |
 |  floor · maintenanceRate  |            |  status · residentId (FK)     |
 |  residentId (FK) · type   |            |  assignedStaffId (FK) · proof |
 +---------------------------+            +-------------------------------+
       │ 1
       │ generates monthly
       ▼ N
 +-------------------------------------------------------------------------+
 |                                  BILL                                   |
 |  _id (PK) · flatId (FK) · residentId (FK) · billingMonth (YYYY-MM)      |
 |  baseAmount · utilityAmount · penaltyAmount · totalDue · paymentStatus  |
 |  dueDate · paidAt · transactionId · receiptUrl                          |
 +-------------------------------------------------------------------------+
       │
       ▼
 +---------------------------+            +-------------------------------+
 |       VISITOR_PASS        |            |        AMENITY_BOOKING        |
 |  _id (PK) · visitorName   |            |  _id (PK) · amenityId (FK)    |
 |  phone · vehicleNumber    |            |  residentId (FK) · date       |
 |  passCode (6-digit)       |            |  startTime · endTime · guests |
 |  qrToken · validUntil     |            |  status · paymentStatus · fee |
 |  residentId (FK) · status |            +-------------------------------+
 +---------------------------+
```

---

## 3. RBAC Access Control Matrix

The system enforces strict route protection both at the Frontend Router (`src/routes/ProtectedRoute.jsx`) and the Backend Middleware (`src/middleware/rbac.js`).

| Feature / Subsystem | Admin (👑) | Resident (👤) | Security Guard (🛡️) | Staff (🔧) |
|---|:---:|:---:|:---:|:---:|
| **User Role Assignment & Account Reset** | ✅ Full | ❌ | ❌ | ❌ |
| **Flat Creation & Occupancy Allocation** | ✅ Full | ❌ | ❌ | ❌ |
| **Bulk Billing & Maintenance Fee Engine** | ✅ Generate/Audit | 👁️ View/Pay Own | ❌ | ❌ |
| **Generate QR Visitor Passes** | ✅ Any Flat | ✅ Own Flat | ❌ | ❌ |
| **Camera QR Pass Scanner & Gate Keypad** | 👁️ Audit | ❌ | ✅ Full Gate Control | ❌ |
| **Walk-in Visitor Logging & Exit Stamps** | 👁️ Audit | ❌ | ✅ Full Gate Control | ❌ |
| **File Complaint / Work Order** | ✅ Any | ✅ Own Flat | ❌ | ❌ |
| **Accept, Progress & Resolve Tickets** | ✅ Assign | ❌ | ❌ | ✅ Assigned Tickets |
| **Shared Amenity Slot Reservations** | ✅ Full/Approve | ✅ Book & Pay | ❌ | ❌ |
| **Publish Notices & Community Polls** | ✅ Full Create | 🗳️ Vote Only | 👁️ View Only | 👁️ View Only |
| **Emergency SOS Siren Broadcast** | ✅ Global Alarm | 🚨 Trigger SOS | 🔔 Alert Receiver | 🔔 Alert Receiver |
| **PDF Ledger & Excel Data Exports** | ✅ System Wide | 📄 Personal Bills | ❌ | ❌ |

---

## 4. Frontend Architecture & UI/UX Stack

```
frontend/src/
├── assets/images/           # High-res hero imagery, icons, background artwork
├── components/
│   ├── layout/              # Sidebar navigation, Top Header, User Avatar Dropdown
│   └── ui/                  # Reusable Design System:
│       ├── Button.jsx       # Primary, Outline, Danger, Ghost variants
│       ├── TextField.jsx    # Input wrapper with dynamic red validation labels
│       ├── TextareaField.jsx
│       ├── SelectField.jsx  # Styled dropdowns
│       ├── Modal.jsx        # Glassmorphic backdrop modals with ESC/outside click
│       ├── Table.jsx        # Responsive data table with column sorting
│       ├── Badge.jsx        # Status pills (Pending, Approved, Paid, Overdue, etc.)
│       ├── Loader.jsx       # Animated loading spinners
│       ├── EmptyState.jsx   # Fallback placeholders with clean iconography
│       ├── HolographicScanner.jsx # Laser overlay with sound synthesis
│       └── ConfettiCelebration.jsx # Festive canvas burst on pass/bill approval
├── context/
│   ├── AuthContext.jsx      # Global JWT state, user payload, role decoding
│   └── ThemeContext.jsx     # Dark / Light luxury theme persistence
├── lib/
│   ├── currency.js          # Format Rs / PKR currency with commas
│   └── date.js              # Relative date formatting and standard timestamps
├── marketing/               # High-Performance Public Landing Experience:
│   ├── Navbar.jsx           # Sticky glassmorphic navbar (dynamic Login/Dashboard)
│   ├── Footer.jsx           # Executive 4-column directory footer
│   └── sections/
│       ├── Hero.jsx         # 3-Layer GSAP Scroll parallax + Live Dues search widget
│       ├── EliteAmenities.jsx # Facility showcase carousel
│       ├── CommunityNotices.jsx # Interactive community voting carousel
│       ├── UpcomingEvents.jsx
│       ├── EstateGuidelines.jsx
│       ├── SitemapFlowchart.jsx # Interactive animated system architecture tree
│       └── EmergencySOS.jsx # Direct guard SOS directory
├── pages/                   # RBAC Dashboard Panels:
│   ├── Dashboard.jsx        # Analytics widgets, stats cards, revenue charts
│   ├── Users.jsx            # User management, staff trade tagger, reset password
│   ├── Flats.jsx            # Building block units, occupancy status, tenant mapping
│   ├── Visitors.jsx         # Timed QR passes, gate camera scanner, walk-in logger
│   ├── Billing.jsx          # Monthly invoices, payment verification, PDF receipts
│   ├── Complaints.jsx       # Service ticket dispatcher, SLA priority, remarks
│   ├── Notices.jsx          # Society announcements & live voting poll manager
│   ├── Amenities.jsx        # Slot availability checker, booking scheduler
│   ├── Emergency.jsx        # SOS siren trigger, society security directory
│   ├── Guidelines.jsx       # Rules and estate policy viewer/editor
│   ├── Profile.jsx          # Personal info, vehicles list, password reset
│   └── Login.jsx            # Animated dual Login & Resident Register portal
├── services/                # Modular Axios API clients with auto token injection
└── utils/
    └── validators.js        # Centralized RFC Regex & validation utility engine
```

---

## 5. Backend Micro-Modules & Database Models

### 5.1 Database Models Schema Breakdown

#### `User` Model
```javascript
{
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true }, // bcrypt hashed (cost 10)
  phone: { type: String, required: true },
  role: { type: String, enum: ['Admin', 'Resident', 'Guard', 'Staff'], default: 'Resident' },
  avatar: { type: String },
  staffProfession: { type: String, enum: ['Electrician', 'Plumber', 'HVAC', 'General', 'LiftTechnician'] },
  vehicles: [{ type: { type: String }, vehicleNumber: String }],
  emergencyContacts: [{ name: String, phone: String, relation: String }],
  familyMembers: [{ name: String, relation: String, age: Number }],
  isActive: { type: Boolean, default: true },
  createdAt: { type: Date, default: Date.now }
}
```

#### `Flat` Model
```javascript
{
  blockName: { type: String, required: true },
  flatNumber: { type: String, required: true, unique: true },
  floor: { type: Number, required: true },
  maintenanceRate: { type: Number, required: true, default: 3000 },
  resident: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  occupancyType: { type: String, enum: ['Owner', 'Tenant', 'Vacant'], default: 'Vacant' }
}
```

#### `VisitorPass` Model
```javascript
{
  visitorName: { type: String, required: true },
  phone: { type: String, required: true },
  vehicleNumber: { type: String },
  purpose: { type: String, required: true },
  passCode: { type: String, required: true, unique: true }, // 6-digit numeric OTP
  qrToken: { type: String, required: true, unique: true },   // Cryptographic signature
  flat: { type: mongoose.Schema.Types.ObjectId, ref: 'Flat', required: true },
  resident: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  validUntil: { type: Date, required: true },
  status: { type: String, enum: ['PENDING', 'APPROVED', 'CHECKED_IN', 'CHECKED_OUT', 'EXPIRED'], default: 'APPROVED' },
  entryTime: { type: Date },
  exitTime: { type: Date }
}
```

#### `Bill` Model
```javascript
{
  flat: { type: mongoose.Schema.Types.ObjectId, ref: 'Flat', required: true },
  resident: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  billingMonth: { type: String, required: true }, // Format: YYYY-MM
  baseAmount: { type: Number, required: true },
  utilityAmount: { type: Number, default: 0 },
  penaltyAmount: { type: Number, default: 0 },
  totalDue: { type: Number, required: true },
  dueDate: { type: Date, required: true },
  paymentStatus: { type: String, enum: ['UNPAID', 'PAID', 'OVERDUE'], default: 'UNPAID' },
  paidAt: { type: Date },
  transactionReference: { type: String }
}
```

#### `Complaint` Model
```javascript
{
  title: { type: String, required: true },
  description: { type: String, required: true },
  category: { type: String, enum: ['Plumbing', 'Electrical', 'Elevator', 'Security', 'General', 'Cleanliness'], required: true },
  priority: { type: String, enum: ['LOW', 'MEDIUM', 'HIGH', 'EMERGENCY'], default: 'MEDIUM' },
  resident: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  flat: { type: mongoose.Schema.Types.ObjectId, ref: 'Flat' },
  assignedStaff: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  status: { type: String, enum: ['OPEN', 'ASSIGNED', 'IN_PROGRESS', 'RESOLVED', 'CLOSED'], default: 'OPEN' },
  resolutionNotes: { type: String },
  resolvedAt: { type: Date },
  slaDueDate: { type: Date }
}
```

---

## 6. Complete REST API Specification

Base URL: `http://localhost:5000/api/v1`

### 6.1 Authentication (`/auth`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/auth/register` | Public | Register a new resident account with multipart avatar |
| `POST` | `/auth/login` | Public | Authenticate user, returns Access Token & Refresh Token |
| `POST` | `/auth/refresh-token` | Public | Exchanges refresh token for a fresh short-lived access token |
| `GET` | `/auth/me` | Authenticated | Retrieve currently authenticated user payload |
| `PUT` | `/auth/update-profile` | Authenticated | Update name, phone, vehicles list, and emergency contacts |
| `PUT` | `/auth/change-password` | Authenticated | Change account password with old password verification |
| `POST` | `/auth/logout` | Authenticated | Revoke refresh token and terminate session |

### 6.2 Users (`/users`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `GET` | `/users` | Admin | List all society members with filtering by role and status |
| `POST` | `/users` | Admin | Create user directly (Admin, Guard, Staff, Resident) |
| `GET` | `/users/:id` | Admin | Retrieve individual user record by MongoDB ID |
| `PUT` | `/users/:id/status` | Admin | Update user role, trade, or active status |
| `PUT` | `/users/:id/reset-password`| Admin | Administratively reset user password |
| `DELETE` | `/users/:id` | Admin | Soft/Hard delete user from society records |

### 6.3 Flats & Inventory (`/flats`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/flats` | Admin | Register new building flat with floor & base rate |
| `GET` | `/flats` | Authenticated | List all society flats and occupancy details |
| `POST` | `/flats/:id/assign` | Admin | Assign resident owner/tenant to flat |
| `GET` | `/flats/occupancy-map` | Admin | Fetch aggregate building occupancy breakdown |

### 6.4 Visitors & Gate Control (`/visitors`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/visitors/generate-pass`| Resident / Admin | Create single-use timed QR pass & 6-digit code |
| `GET` | `/visitors/my-passes` | Resident | Fetch all active and expired visitor passes for flat |
| `POST` | `/visitors/verify-qr` | Guard / Admin | Decrypt and validate holographic QR pass scan |
| `POST` | `/visitors/verify-code`| Guard / Admin | Verify 6-digit numeric keypad entry code |
| `POST` | `/visitors/walk-in` | Guard / Admin | Log walk-in visitor entry without prior invitation |
| `PUT` | `/visitors/:id/exit` | Guard / Admin | Record visitor check-out timestamp |
| `GET` | `/visitors/gate-logs` | Guard / Admin | Live chronological gate entry/exit logs |

### 6.5 Billing & Invoicing (`/bills`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/bills/generate-monthly`| Admin | Bulk generate maintenance invoices for current month |
| `GET` | `/bills/my-bills` | Resident | View personal billing history and pending invoices |
| `GET` | `/bills/all` | Admin | View society-wide collection master ledger |
| `GET` | `/bills/public-dues/:flatNumber`| Public | Lookup outstanding balance for public landing hero |
| `PUT` | `/bills/:id/pay` | Resident / Admin | Mark invoice as paid with transaction ID |
| `GET` | `/bills/metrics` | Admin | Total collected, pending, and overdue financials |

### 6.6 Complaints & Helpdesk (`/complaints`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/complaints` | Resident | Lodge maintenance complaint with priority & trade |
| `GET` | `/complaints/my` | Resident | List resident's personal filed tickets |
| `GET` | `/complaints/assigned` | Staff | List service tickets assigned to logged-in technician |
| `GET` | `/complaints/all` | Admin | View all society complaints with SLA tracking |
| `PUT` | `/complaints/:id/assign`| Admin | Dispatch complaint to specific staff member |
| `PUT` | `/complaints/:id/status`| Staff / Admin | Advance status (`IN_PROGRESS`, `RESOLVED`) with remarks |

### 6.7 Notices & Digital Polls (`/notices`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/notices` | Admin | Publish society circular or democratic poll |
| `GET` | `/notices` | Authenticated / Public | Get active society announcements & polls |
| `POST` | `/notices/:id/vote` | Resident | Cast vote for a poll option (enforces single vote per user) |
| `DELETE` | `/notices/:id` | Admin | Remove notice or poll |

### 6.8 Shared Amenities (`/amenities`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/amenities` | Admin | Register new facility (Clubhouse, Pool, Court) |
| `GET` | `/amenities` | Authenticated / Public | Fetch amenity list with capacity & booking fees |
| `GET` | `/amenities/:id/availability` | Authenticated | Check booked slots for a chosen date |
| `POST` | `/amenities/:id/book` | Resident | Reserve facility slot for specific guest count |
| `GET` | `/amenities/my-bookings`| Resident | View upcoming reservation passes |
| `PUT` | `/amenities/bookings/:id/cancel` | Resident / Admin | Cancel reservation |

### 6.9 Emergency SOS (`/emergency`)
| Method | Endpoint | Access | Description |
|---|---|---|---|
| `POST` | `/emergency/trigger` | Authenticated | Broadcast urgent SOS alarm with location details |
| `GET` | `/emergency/active` | Authenticated | List all unresolved live emergency siren alarms |
| `PUT` | `/emergency/:id/resolve`| Admin / Guard | Resolve emergency incident |
| `GET` | `/emergency/contacts` | Authenticated / Public | Get estate emergency phone directory |
| `POST` | `/emergency/contacts` | Admin | Add emergency phone contact |
| `DELETE` | `/emergency/contacts/:id`| Admin | Delete contact |

---

## 7. Marketing Landing Page Modules

The public marketing interface is built for high conversion and architectural transparency:

1. **GSAP 3.15 Hero Parallax:** Features an expanding architectural cover frame and the **Live Dues Search Widget** where any resident can type their flat unit (`A-101`) to inspect their current bill and overdue status.
2. **Interactive Community Poll Carousel:** Allows visitors and residents to see live voting percentages and active society ballots.
3. **Elite Amenities Grid:** Dynamic image galleries of sports courts, swimming facilities, and party halls.
4. **Sitemap Flowchart Section:** Visual tree flowchart illustrating the exact path from public entry to the 4 RBAC role portals.
5. **Emergency SOS Directory:** Instant one-click phone numbers for Society Security, Ambulance, Fire Brigade, Police, and Society Secretary.
6. **Smart Dynamic Navbar:** Automatically switches between `"Log in"` and `"Go to Dashboard"` depending on whether the user is logged in.

---

## 8. Form Validation & RFC Regex Engine

All inputs in the system are bound to strict validation rules defined in [`src/utils/validators.js`](frontend/src/utils/validators.js):

* **Email Standard:** Strict RFC 5322 regex: `/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/`
* **Phone Standard:** Validates 10 to 15 digits supporting `+92`, `03xx`, dashes, and spaces: `/^(\+?[0-9]{1,4}[-.\s]?)?(\(?\d{3,4}\)?[-.\s]?)?[\d\s.-]{7,12}$/`
* **Name Standard:** Letters, spaces, hyphens, and dots only (2 to 50 characters): `/^[a-zA-Z\s.-]{2,50}$/`
* **Vehicle Plate Standard:** Multi-format license plate verification: `/^[A-Z0-9]{2,4}[-\s]?[0-9]{1,4}$/i`
* **Password Policy:** Minimum 6 characters with instant confirm-password matching.
* **Time Slot Logic:** Enforces `endTime > startTime` and prevents booking dates in the past.
* **Real-time Error Recovery:** As soon as the user starts typing inside an invalid field, the red error border and error label clear immediately.

---

## 9. Export, Printing & Media Handling

* **PDF Invoicing:** Direct PDF generation of maintenance dues bills using `jspdf` and `jspdf-autotable`.
* **Excel Data Exports:** One-click spreadsheet export (`.xlsx`) for user rosters, visitor gate logs, and financial records using `xlsx`.
* **QR Code Generation:** Generates high-density vector QR codes for visitor passes (`qrcode.react`).
* **Live Camera Scanning:** Real-time video frame QR scanning with continuous auto-focus (`html5-qrcode`).
* **Image Uploads:** Avatars and maintenance defect photos are processed via Multer.

---

## 10. Step-by-Step Installation & Setup

### Prerequisites
* **Node.js**: v18.0.0 or later
* **MongoDB**: Community Server running on `localhost:27017` or MongoDB Atlas URI
* **npm** or **yarn**

---

### Step 1: Clone Repository
```bash
git clone https://github.com/your-username/ContestAzm-26.git
cd ContestAzm-26
```

---

### Step 2: Backend Setup
```bash
# Navigate to backend
cd backend

# Install dependencies
npm install

# Create environment file
cp .env.example .env

# Seed initial admin and demo data (optional if seed script provided)
npm run seed

# Start backend server
npm run dev
```
> Backend runs at `http://localhost:5000`

---

### Step 3: Frontend Setup
```bash
# Navigate to frontend (in a separate terminal)
cd ../frontend

# Install dependencies
npm install

# Create environment file
cp .env.example .env

# Start Vite development server
npm run dev
```
> Frontend runs at `http://localhost:5173`

---

## 11. Environment Variables Reference

### Backend (`backend/.env`)
```env
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb://localhost:27017/smartsociety
JWT_SECRET=smartsociety_super_secret_jwt_access_key_2026!
JWT_REFRESH_SECRET=smartsociety_super_secret_jwt_refresh_key_2026!
JWT_EXPIRES_IN=1d
JWT_REFRESH_EXPIRES_IN=7d
CORS_ORIGIN=http://localhost:5173
```

### Frontend (`frontend/.env`)
```env
VITE_API_URL=http://localhost:5000/api/v1
```

---

## 12. Default Seed Credentials

Use these credentials to log in and test each role panel immediately:

| Role | Email | Password | Assigned Permissions |
|---|---|---|---|
| **Executive Admin** | `admin@smartsociety.com` | `Admin123!` | Complete society management, billing, audits |
| **Resident Owner** | `resident@smartsociety.com` | `Resident123!` | Passes, bill payments, amenities, polls |
| **Security Guard** | `guard@smartsociety.com` | `Guard123!` | Holographic QR scanner, gate logs, walk-ins |
| **Maintenance Staff** | `staff@smartsociety.com` | `Staff123!` | Work orders queue, SLA timers, remarks |

---

## 13. Project Directory Tree

```
ContestAzm-26/
├── backend/
│   ├── src/
│   │   ├── config/           # Database connection & JWT config
│   │   ├── controllers/      # authController, flatController, billController, etc.
│   │   ├── middleware/       # auth.js, rbac.js, upload.js, errorHandler.js
│   │   ├── models/           # User, Flat, Bill, VisitorPass, Complaint, Notice, Amenity
│   │   ├── routes/           # Express router endpoints
│   │   ├── utils/            # jwtHelpers.js, seedData.js
│   │   └── server.js         # Express app entry point
│   ├── .env.example
│   └── package.json
│
├── frontend/
│   ├── public/               # Static icons, logos, favicons
│   ├── src/
│   │   ├── assets/           # Parallax hero artwork, sitemap background
│   │   ├── components/       # UI design system components
│   │   ├── context/          # AuthContext, ThemeContext
│   │   ├── lib/              # currency.js, date.js
│   │   ├── marketing/        # Landing page sections (Hero, Sitemap, Footer, etc.)
│   │   ├── pages/            # 11 RBAC dashboard pages
│   │   ├── routes/           # AppRoutes.jsx, ProtectedRoute.jsx
│   │   ├── services/         # Axios API clients for all backend modules
│   │   ├── utils/            # validators.js (RFC Regex engine)
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   └── index.css
│   ├── .env.example
│   ├── package.json
│   ├── vite.config.js
│   └── README.md
│
└── README.md
```

---

## 📄 License
This project is open-source and released under the **MIT License**.
