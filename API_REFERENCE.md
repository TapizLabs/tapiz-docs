# API Reference — tapiz-rest-api

All endpoints are prefixed with `/api`. The server defaults to `http://localhost:3001`.

---

## Authentication

All protected routes require a `Bearer` token in the `Authorization` header:

```
Authorization: Bearer <jwt>
```

Tokens are issued by `POST /api/auth/login`. If 2FA is enabled on the account, the login response returns `{ requiresTwoFactor: true }` — call `POST /api/auth/verify-2fa` to receive the actual JWT.

---

## Roles & Permissions

| Role | Description |
|------|-------------|
| `assistant` | Manages subjects, sessions, attendance, score sheets, forms, and reports |
| `student` | Scans QR codes, views own attendance, score sheets, grades, and notifications |

Route tables below use these symbols:
- 🔓 Public (no auth required)
- 🎓 `student` role required
- 🧑‍🏫 `assistant` role required
- 🔐 Any authenticated user

---

## `/api/auth`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/register` | 🔓 | Register a new student account |
| `POST` | `/login` | 🔓 | Log in (returns JWT or 2FA challenge) |
| `POST` | `/verify-2fa` | 🔓 | Complete 2FA login, returns JWT |
| `POST` | `/refresh` | 🔐 | Refresh JWT token |
| `GET`  | `/2fa-status` | 🔐 | Get current 2FA enabled status |
| `POST` | `/toggle-2fa` | 🔐 | Enable or disable 2FA |
| `POST` | `/seed-assistant` | 🔓 | Bootstrap the first assistant account (requires `SEED_SECRET_KEY`) |
| `GET`  | `/assistant-profile` | 🧑‍🏫 | Get authenticated assistant's profile |
| `GET`  | `/student-profile` | 🎓 | Get authenticated student's profile |
| `POST` | `/change-password` | 🎓 | Student changes own password |
| `POST` | `/change-password-assistant` | 🧑‍🏫 | Assistant changes own password |
| `POST` | `/reset-student-password` | 🧑‍🏫 | Reset a student's password to the default |
| `POST` | `/deactivate-account` | 🧑‍🏫 | Deactivate an assistant account |
| `POST` | `/deactivate-student-account` | 🎓 | Deactivate a student account |

### `POST /api/auth/register`
```json
{
  "smer": "PR",
  "indexNumber": 42,
  "enrollmentYear": 2023,
  "lastName": "Petrović",
  "firstName": "Marko",
  "email": "marko@university.edu",
  "password": "securepassword"
}
```

### `POST /api/auth/login`
```json
{ "email": "user@university.edu", "password": "password" }
```
**Response (no 2FA):**
```json
{ "token": "<jwt>", "role": "student", "id": 1, ... }
```
**Response (2FA enabled):**
```json
{ "requiresTwoFactor": true }
```

### `POST /api/auth/verify-2fa`
```json
{ "email": "user@university.edu", "code": "123456" }
```

### `POST /api/auth/toggle-2fa`
```json
{ "enabled": true }
```

---

## `/api/sessions`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/generate` | 🧑‍🏫 | Create a new session and generate its QR token |
| `POST` | `/rotate/:id` | 🧑‍🏫 | Rotate (refresh) the QR token for an existing session |
| `POST` | `/invalidate` | 🧑‍🏫 | Deactivate a session |
| `GET`  | `/` | 🧑‍🏫 | List all sessions for a subject (`?subjectId=N`) |
| `GET`  | `/active` | 🧑‍🏫 | Get the active session for a subject (`?subjectId=N`) |
| `DELETE` | `/:id` | 🧑‍🏫 | Delete a session |

### `POST /api/sessions/generate`
```json
{
  "subjectId": 1,
  "sessionNumber": 5,
  "sessionType": "Računarske vežbe"
}
```
Valid `sessionType` values: `"Predavanja"`, `"Računarske vežbe"`, `"Auditorne vežbe"`, `"Labaratorijske vežbe"`

---

## `/api/attendance`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `POST` | `/scan` | 🎓 | Student scans QR code to record attendance |
| `GET`  | `/my` | 🎓 | Student's own attendance for a subject (`?subjectId=N`) |
| `GET`  | `/all` | 🧑‍🏫 | All attendance records for a subject (`?subjectId=N`) |
| `GET`  | `/stats` | 🧑‍🏫 | Attendance statistics for a subject (`?subjectId=N`) |
| `GET`  | `/matrix` | 🧑‍🏫 | Attendance matrix (student × session) (`?subjectId=N`) |
| `POST` | `/manual` | 🧑‍🏫 | Manually add a single attendance record |
| `POST` | `/manual-bulk` | 🧑‍🏫 | Bulk-add attendance for multiple students |
| `DELETE` | `/:id` | 🧑‍🏫 | Remove an attendance record (`?subjectId=N`) |

### `POST /api/attendance/scan`
```json
{ "sessionId": 10, "token": "<qr-token>" }
```

### `POST /api/attendance/manual`
```json
{ "subjectId": 1, "studentId": 5, "sessionId": 10 }
```

### `POST /api/attendance/manual-bulk`
```json
{ "subjectId": 1, "sessionId": 10, "studentIds": [1, 2, 3, 4] }
```
**Response:**
```json
{ "added": [...], "failed": 0, "message": "Dodato 4 prisustva" }
```

---

## `/api/subjects`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🧑‍🏫 | List all subjects belonging to the authenticated assistant |
| `GET`  | `/student` | 🎓 | List subjects the authenticated student is enrolled in |
| `GET`  | `/:id` | 🧑‍🏫 | Get subject details |
| `POST` | `/` | 🧑‍🏫 | Create a new subject |
| `PATCH` | `/:id` | 🧑‍🏫 | Update subject (name, code, absenceThreshold) |
| `DELETE` | `/:id` | 🧑‍🏫 | Delete a subject |
| `POST` | `/:id/assistants` | 🧑‍🏫 | Assign another assistant to a subject |
| `DELETE` | `/:id/assistants/:assistantId` | 🧑‍🏫 | Remove an assistant from a subject |

---

## `/api/students`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🧑‍🏫 | List all students enrolled in a subject (`?subjectId=N`) |
| `GET`  | `/:id` | 🧑‍🏫 | Get a student's details |
| `POST` | `/enroll` | 🧑‍🏫 | Enroll a student in a subject |
| `DELETE` | `/enroll` | 🧑‍🏫 | Remove a student from a subject |
| `POST` | `/bulk-enroll` | 🧑‍🏫 | Bulk-enroll multiple students |

---

## `/api/assistants`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🧑‍🏫 | Search assistants by email or name |
| `GET`  | `/profile` | 🧑‍🏫 | Get authenticated assistant's full profile |
| `PATCH` | `/profile` | 🧑‍🏫 | Update profile (firstName, lastName) |

---

## `/api/notifications`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🎓 | Get all notifications for the authenticated student |
| `POST` | `/read/:id` | 🎓 | Mark a notification as read |
| `POST` | `/read-all` | 🎓 | Mark all notifications as read |
| `POST` | `/send-warnings` | 🧑‍🏫 | Send absence warnings for students over threshold (`?subjectId=N`) |

---

## `/api/score-sheets`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🧑‍🏫 | List score sheets for a subject (`?subjectId=N`) |
| `GET`  | `/:id` | 🧑‍🏫 | Get a score sheet with all rows, columns, and cells |
| `GET`  | `/student/:sheetId` | 🎓 | Get student's own row in a score sheet |
| `POST` | `/` | 🧑‍🏫 | Create a new score sheet |
| `DELETE` | `/:id` | 🧑‍🏫 | Delete a score sheet |
| `POST` | `/:id/columns` | 🧑‍🏫 | Add a column |
| `PATCH` | `/:id/columns/:colId` | 🧑‍🏫 | Update a column |
| `DELETE` | `/:id/columns/:colId` | 🧑‍🏫 | Delete a column |
| `POST` | `/:id/rows` | 🧑‍🏫 | Add a row |
| `DELETE` | `/:id/rows/:rowId` | 🧑‍🏫 | Delete a row |
| `PATCH` | `/:id/cells` | 🧑‍🏫 | Update one or more cells (batch) |

---

## `/api/versions`

Score sheet version history.

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/:sheetId` | 🧑‍🏫 | List versions for a score sheet |
| `GET`  | `/:sheetId/:versionId` | 🧑‍🏫 | Get a specific version snapshot |
| `POST` | `/:sheetId/save` | 🧑‍🏫 | Save current state as a new version |
| `POST` | `/:sheetId/restore/:versionId` | 🧑‍🏫 | Restore a score sheet to a past version |

---

## `/api/forms`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🧑‍🏫 | List forms for a subject (`?subjectId=N`) |
| `GET`  | `/:id` | 🧑‍🏫 | Get form details and responses |
| `POST` | `/` | 🧑‍🏫 | Create a new form |
| `PATCH` | `/:id` | 🧑‍🏫 | Update a form |
| `DELETE` | `/:id` | 🧑‍🏫 | Delete a form |
| `GET`  | `/public/:token` | 🔓 | Get public form by access token (for student submission) |
| `POST` | `/public/:token/submit` | 🔓 | Submit a response to a public form |

---

## `/api/reports`

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| `GET`  | `/` | 🧑‍🏫 | Get report data for a subject (`?subjectId=N`) |

Report data is typically consumed by the frontend to generate PDF/Excel files via the Vercel microservices (`VITE_EXCEL_API_URL`, `VITE_PDF_API_URL`).

---

## `/api/health`

```
GET /api/health
→ 200 { "status": "ok", "ts": "2025-01-01T00:00:00.000Z" }
```
