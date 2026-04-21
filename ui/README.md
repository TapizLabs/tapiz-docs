# tapiz-reactjs-ui

Frontend client for **Tapiz** — built with React 19, Vite, TypeScript, and Tailwind CSS v4.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Development](#development)
- [Production Build](#production-build)
- [Environment Variables](#environment-variables)
- [Project Structure](#project-structure)
- [Routing](#routing)
- [Features](#features)
- [External Services](#external-services)

---

## Prerequisites

- Node.js ≥ 18
- npm ≥ 9
- A running instance of `tapiz-rest-api` (see [api/README.md](../api/README.md))

---

## Installation

```bash
git clone https://github.com/your-org/tapiz-reactjs-ui.git
cd tapiz-reactjs-ui
npm install
```

---

## Development

```bash
cp .env.example .env
# Set VITE_API_URL to your running API instance
npm run dev
```

Vite dev server starts at `http://localhost:5173`.

---

## Production Build

```bash
npm run build     # type-check + Vite build → dist/
npm run preview   # preview the production build locally
```

The `dist/` directory is a static SPA — deploy to any static host (Vercel, Netlify, S3 + CloudFront, etc.).

---

## Environment Variables

Copy `.env.example` to `.env`.

| Variable | Required | Description |
|----------|----------|-------------|
| `VITE_API_URL` | ✓ | Base URL of the REST API, e.g. `http://localhost:3001/api` |
| `VITE_EXCEL_API_URL` | | URL of the Excel report generation microservice |
| `VITE_PDF_API_URL` | | URL of the PDF report generation microservice |
| `VITE_DEFAULT_STUDENT_PASSWORD` | | Default password shown to assistants after a student password reset |

All `VITE_*` variables are inlined at build time by Vite and visible in the browser bundle. Do not store secrets here.

---

## Project Structure

```
src/
├── App.tsx                  # Root router, lazy-loaded routes, auth redirect
├── main.tsx                 # React entry point
├── index.css                # Tailwind CSS v4 import
│
├── pages/
│   ├── auth/                # LoginPage, RegisterPage
│   ├── student/             # StudentDashboard, ScanPage, GradesPage, StudentCalendarPage
│   ├── assistant/           # StudentsPage, SubjectsPage, SessionsPage, AttendancesPage,
│   │                        #   AnalyticsPage, ScoreSheetsPage, QRPage, FormsPage, CalendarPage
│   └── public/              # PublicFormPage (accessible by token, no auth)
│
├── components/
│   ├── analytics/           # KPI row, charts (trend, heatmap, distribution, cumulative, spikes),
│   │                        #   student stats tables
│   ├── attendance/          # Session attendance panel + table, heatmap, radial progress,
│   │                        #   session selector, bulk add
│   ├── auth/                # RegisterSteps (multi-step registration form)
│   ├── calendar/            # SessionCalendar, DayCell
│   ├── forms/               # FormBuilder, FormCard, DeleteFormDialog
│   ├── grades/              # Grade display components
│   ├── layout/              # Shell/sidebar layout
│   ├── qr/                  # QR code display and rotation
│   ├── report/              # Report generation UI
│   ├── scoresheet/          # Spreadsheet-like score sheet editor
│   ├── sessions/            # Session list and management
│   ├── settings/            # Account settings
│   ├── shared/              # Shared UI components
│   ├── students/            # Student list, enrollment
│   ├── subjects/            # Subject list, creation
│   └── ui/                  # Design system: buttons, cards, modals, skeletons, feedback
│
├── hooks/                   # Custom React hooks (useAuth, data-fetching hooks, etc.)
├── services/                # Axios API service modules (one per domain)
├── contexts/                # React context providers (AuthContext, etc.)
├── models/                  # TypeScript interfaces matching API response shapes
├── types/                   # Shared TypeScript types
├── config/                  # App-wide config (API base URL, constants)
├── constants/               # Static constants
├── helpers/                 # Pure utility functions
├── utils/                   # Additional utilities
├── lib/                     # Third-party library wrappers
└── features/                # Feature-level state or logic (if any)
```

---

## Routing

All routes are lazy-loaded. Authentication state is read from `AuthContext` via `useAuth()`. `ProtectedRoute` redirects unauthenticated users to `/login` and checks role authorization.

### Public routes

| Path | Page |
|------|------|
| `/login` | Login form (with 2FA step if required) |
| `/register` | Multi-step student registration |
| `/forms/:token` | Public form submission page |

### Student routes (role: `student`)

| Path | Page |
|------|------|
| `/dashboard` | Student home — attendance overview |
| `/dashboard/scan` | QR code scanner |
| `/dashboard/grades` | Score sheets / grades |
| `/dashboard/calendar` | Session calendar |

### Assistant routes (role: `assistant`)

| Path | Page |
|------|------|
| `/assistant` | Students list |
| `/assistant/subjects` | Subject management |
| `/assistant/sessions` | Session management + QR generation |
| `/assistant/calendar` | Session calendar |
| `/assistant/attendances` | Attendance table and manual entry |
| `/assistant/analytics` | Analytics dashboard |
| `/assistant/score-sheets` | Score sheet editor |
| `/assistant/qr` | Live QR code display (for projecting in class) |
| `/assistant/forms` | Form builder and responses |

The `/` root redirects to the appropriate home route based on authenticated role.

---

## Features

### QR Attendance Scanning
Students navigate to `/dashboard/scan` and scan the QR code displayed by the assistant. The `jsqr` library decodes QR frames from the camera feed in real time. The decoded token is sent to `POST /api/attendance/scan`.

### Session Calendar
A monthly calendar view showing all sessions. DayCells are colour-coded by session type. Assistants can create and delete sessions directly from the calendar.

### Analytics Dashboard
Available to assistants at `/assistant/analytics`. Includes:
- KPI row (total students, total sessions, average attendance rate, students above threshold)
- Cumulative attendance chart (attendance count per session, per student)
- Trend chart (attendance rate over time)
- Absence spikes chart (sessions with unusually high absence)
- Distribution chart (student attendance rate buckets)
- Student stats table with search, sortable columns, and mobile-friendly row expansion

### Score Sheet Editor
Spreadsheet-like grid with:
- Dynamic columns (type: `number`, `text`, `formula`)
- Formula support: `SUM`, `IF`, column cross-references
- Cell editing inline
- Version history: save snapshots, view diff, restore to prior state
- Hidden columns for internal calculations

### Form Builder
Assistants can create custom forms linked to a subject. Each form gets a public access token URL (`/forms/:token`) that can be shared with students. Supports text and multiple-choice questions. Responses are collected and viewable in the assistant panel.

### Reports
Assistants can export attendance and score sheet data as PDF or Excel. Report data is fetched from the API and sent to one of two Vercel microservices (`VITE_EXCEL_API_URL`, `VITE_PDF_API_URL`) which handle document generation and return a downloadable file.

### Notifications
Students receive in-app notifications (e.g. absence threshold warnings). Assistants can trigger warning sends per subject from the attendance page.

### 2FA
Both roles support email-based two-factor authentication, managed from the account settings page.

---

## External Services

| Variable | Purpose |
|----------|---------|
| `VITE_EXCEL_API_URL` | Vercel microservice that accepts report data and returns `.xlsx` |
| `VITE_PDF_API_URL` | Vercel microservice that accepts report data and returns `.pdf` |

These are optional. If not configured, report export buttons will not function.
