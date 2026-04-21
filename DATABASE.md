# Database Schema — tapiz-rest-api

PostgreSQL ≥ 14. All migration files are in `sql/`. Run `01_create.sql` first, then `02_indexes.sql`.

---

## Entity Relationship Overview

```
assistants ──< subject_assistants >── subjects ──< sessions ──< attendances
                                           │                         │
                                     student_subjects           students
                                           │                         │
                                        students              notifications
                                                                     │
                                                               subjects

subjects ──< score_sheets ──< score_columns
                         └──< score_rows ──< score_cells
                         └──< sheet_versions

subjects ──< forms ──< form_questions ──< form_options
                  └──< form_responses ──< form_answers
```

---

## Tables

### `assistants`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `email` | `VARCHAR(150)` | Unique |
| `passwordHash` | `TEXT` | bcrypt hash |
| `firstName` | `VARCHAR(100)` | |
| `lastName` | `VARCHAR(100)` | |
| `isActive` | `BOOLEAN` | Default `true` |
| `twoFactorEnabled` | `BOOLEAN` | Default `false` |
| `lastLoginAt` | `TIMESTAMPTZ` | Nullable |
| `createdAt` | `TIMESTAMPTZ` | Default `NOW()` |

---

### `subjects`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `name` | `VARCHAR(200)` | |
| `code` | `VARCHAR(20)` | Unique |
| `absenceThreshold` | `INTEGER` | Default `30` — percentage threshold for absence warnings |
| `isActive` | `BOOLEAN` | Default `true` |
| `createdAt` | `TIMESTAMPTZ` | |
| `assistant_id` | `INTEGER` FK → `assistants.id` | Owning assistant |

---

### `subject_assistants`

Junction table for assigning multiple assistants to a subject.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `subject_id` | `INTEGER` FK → `subjects.id` | Cascade delete |
| `assistant_id` | `INTEGER` FK → `assistants.id` | Cascade delete |
| `assignedAt` | `TIMESTAMPTZ` | |

Unique constraint on `(subject_id, assistant_id)`.

---

### `students`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `smer` | `VARCHAR(10)` | Study programme code (e.g. `"PR"`) |
| `indexNumber` | `INTEGER` | |
| `enrollmentYear` | `INTEGER` | |
| `lastName` | `VARCHAR(100)` | |
| `firstName` | `VARCHAR(100)` | |
| `email` | `VARCHAR(150)` | Unique |
| `passwordHash` | `TEXT` | |
| `isActive` | `BOOLEAN` | |
| `twoFactorEnabled` | `BOOLEAN` | |
| `createdAt` | `TIMESTAMPTZ` | |

Unique constraint on `(smer, indexNumber, enrollmentYear)`.

---

### `sessions`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `sessionNumber` | `INTEGER` | Sequential number within a subject |
| `qrToken` | `VARCHAR(255)` | Current active QR token |
| `qrExpiresAt` | `TIMESTAMPTZ` | Token expiry (controlled by `QR_EXPIRY_SECONDS`) |
| `isActive` | `BOOLEAN` | Default `true` |
| `sessionType` | `VARCHAR(50)` | Default `"Računarske vežbe"` |
| `createdAt` | `TIMESTAMPTZ` | |
| `subject_id` | `INTEGER` FK → `subjects.id` | |

---

### `attendances`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `student_id` | `INTEGER` FK → `students.id` | |
| `session_id` | `INTEGER` FK → `sessions.id` | Cascade delete |
| `recordedAt` | `TIMESTAMPTZ` | Default `NOW()` |

Unique constraint on `(student_id, session_id)` — prevents duplicate scan.

---

### `student_subjects`

Enrollment junction.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `student_id` | `INTEGER` FK → `students.id` | |
| `subject_id` | `INTEGER` FK → `subjects.id` | |
| `enrolledAt` | `TIMESTAMPTZ` | |

Unique on `(student_id, subject_id)`.

---

### `notifications`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `student_id` | `INTEGER` FK → `students.id` | Cascade delete |
| `subject_id` | `INTEGER` FK → `subjects.id` | Cascade delete, nullable |
| `type` | `VARCHAR(50)` | Default `"ABSENCE_WARNING"` |
| `message` | `TEXT` | |
| `isRead` | `BOOLEAN` | Default `false` |
| `createdAt` | `TIMESTAMPTZ` | |

---

### `score_sheets`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `name` | `VARCHAR(200)` | |
| `academicYear` | `VARCHAR(20)` | e.g. `"2024/2025"` |
| `isActive` | `BOOLEAN` | |
| `createdAt` | `TIMESTAMPTZ` | |
| `updatedAt` | `TIMESTAMPTZ` | |
| `subject_id` | `INTEGER` FK → `subjects.id` | Cascade delete |
| `assistant_id` | `INTEGER` FK → `assistants.id` | |

---

### `score_columns`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `name` | `VARCHAR(100)` | |
| `type` | `VARCHAR(20)` | Default `"number"` |
| `formula` | `VARCHAR(500)` | Optional formula (e.g. `SUM(col1, col2)`) |
| `position` | `INTEGER` | Display order |
| `maxPoints` | `DOUBLE PRECISION` | Nullable |
| `isHidden` | `BOOLEAN` | Default `false` |
| `sheet_id` | `INTEGER` FK → `score_sheets.id` | Cascade delete |

---

### `score_rows`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `position` | `INTEGER` | Row display order |
| `studentId` | `INTEGER` | Nullable — linked student |
| `studentName` | `VARCHAR(150)` | Nullable |
| `indexNumber` | `VARCHAR(30)` | Nullable |
| `transferredFromYear` | `VARCHAR(20)` | Nullable — for students from prior years |
| `sheet_id` | `INTEGER` FK → `score_sheets.id` | Cascade delete |

---

### `score_cells`

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `value` | `VARCHAR(500)` | Nullable |
| `row_id` | `INTEGER` FK → `score_rows.id` | Cascade delete |
| `column_id` | `INTEGER` FK → `score_columns.id` | Cascade delete |

Unique on `(row_id, column_id)`.

---

### `sheet_versions`

Snapshot-based version history for score sheets.

| Column | Type | Notes |
|--------|------|-------|
| `id` | `SERIAL` PK | |
| `sheet_id` | `INTEGER` FK → `score_sheets.id` | Cascade delete |
| `assistant_id` | `INTEGER` FK → `assistants.id` | Cascade delete |
| `label` | `VARCHAR(200)` | Version label / description |
| `snapshot` | `JSONB` | Full or partial sheet snapshot |
| `parentVersionId` | `INTEGER` | Nullable — for incremental snapshots |
| `isFullSnapshot` | `BOOLEAN` | Default `true` |
| `createdAt` | `TIMESTAMPTZ` | |

---

### `forms` / `form_questions` / `form_options` / `form_responses` / `form_answers`

Custom form builder tables.

**`forms`** — form definitions linked to a subject, with a public access token.

**`form_questions`** — questions within a form (text, multiple choice, etc.).

**`form_options`** — answer options for multiple-choice questions.

**`form_responses`** — one row per form submission (stores submitter identity if provided).

**`form_answers`** — one row per answered question within a response.
