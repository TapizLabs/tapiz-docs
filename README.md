# Tapiz Documentation

> **Tapiz** is an attendance tracking and academic management platform for university courses. It consists of two repositories: a REST API backend and a React/TypeScript frontend.

---

## Repositories

| Repo | Description | Tech Stack |
|------|-------------|------------|
| [`tapiz-rest-api`](./api/README.md) | Backend REST API | Node.js, Hono, TypeORM, PostgreSQL |
| [`tapiz-reactjs-ui`](./ui/README.md) | Frontend client | React 19, Vite, Tailwind CSS v4 |

---

## Platform Overview

Tapiz supports two user roles:

- **Assistant** — manages subjects, sessions, score sheets, forms, and reports; generates QR codes for attendance
- **Student** — scans QR codes to record attendance, views grades, score sheets, and calendar

### Core Features

- QR-code based attendance scanning with configurable expiry
- Session management (lectures, lab exercises, computer exercises, auditory exercises)
- Analytics dashboard with heatmaps, trend charts, and cumulative attendance graphs
- Score sheets with formula support (SUM, IF, column references)
- Version history for score sheets (full snapshots + diffs)
- Custom form builder with public token-based access
- 2FA via email for both roles
- PDF and Excel report export via Vercel microservices
- Absence threshold notifications per subject

---

## Getting Started

1. **Set up the API** → [api/README.md](./api/README.md)
2. **Set up the UI** → [ui/README.md](./ui/README.md)
3. **API Reference** → [api/API_REFERENCE.md](./api/API_REFERENCE.md)
4. **Database schema** → [api/DATABASE.md](./api/DATABASE.md)
5. **Architecture** → [api/ARCHITECTURE.md](./api/ARCHITECTURE.md)

---

## Quick Links

- [Environment Variables — API](./api/README.md#environment-variables)
- [Environment Variables — UI](./ui/README.md#environment-variables)
- [Authentication Flow](./api/API_REFERENCE.md#authentication)
- [Roles & Permissions](./api/API_REFERENCE.md#roles--permissions)
