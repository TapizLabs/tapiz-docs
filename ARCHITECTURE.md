# Architecture — tapiz-rest-api

## Overview

The project follows **Clean Architecture** with a strict inward dependency rule. No outer layer may be imported by an inner layer.

```
Presentation  →  Application  →  Domain  ←  Database
                                    ↑
                              Infrastructure
```

---

## Layers

### Domain
Pure TypeScript — no framework dependencies, no I/O.

- `Domain/models/` — TypeORM entity classes (used as DB schema source of truth)
- `Domain/repositories/` — Repository interfaces (`I*Repository`)
- `Domain/services/` — Use-case interfaces (`I*Service`)
- `Domain/dto/` — Data transfer objects for all domains (auth, attendance, subjects, forms, score sheets, reports)
- `Domain/ports/` — External service interfaces (email, raw-SQL query services)

### Application
Concrete implementations of use-case services. All business logic lives here. No Hono, no HTTP.

- `Application/facades/` — Thin facade services that delegate to one or more concrete services (e.g. `AttendanceService` delegates to `AttendanceCoreService` + `AttendanceStatisticsService`)
- `Application/services/` — Domain-specific service implementations:
  - `auth/` — Login, token, 2FA, account registration, seeding
  - `attendance/` — QR scanning, manual add/remove, statistics, matrix queries
  - `form/` — Form CRUD, public token access, response collection
  - `notification/` — Absence threshold warnings
  - `report/` — Report data assembly
  - `scoresheet/` — Column, row, and cell CRUD; formula evaluation
  - `assistant/` — Assistant profile management

### Database
TypeORM repository implementations and raw-SQL query services.

- `Database/repositories/` — TypeORM implementations of `I*Repository` interfaces
- `Database/query-services/AttendanceQueryService.ts` — Raw SQL for complex aggregation queries (stats, matrix, heatmap data) where ORM query building would be impractical

### Infrastructure
External adapters — anything that crosses a process boundary.

- `Infrastructure/email/HttpEmailService.ts` — Sends email via an external HTTP microservice (2FA codes)
- `Infrastructure/di/container.ts` — Manual dependency injection. Wires all repositories, services, and routers together. The single place where concrete implementations are chosen.
- `Infrastructure/config/database.ts` — TypeORM `DataSource` configuration

### Presentation
HTTP layer — Hono routers, middleware, parameter helpers.

- `Presentation/routes/` — One file per domain (`createXxxRouter` factory functions)
- `Presentation/middleware/auth.ts` — `authenticate` (validates JWT) and `authorize(role)` (checks role)
- `Presentation/middleware/errorHandler.ts` — Global Hono error handler + `sendError()` helper
- `Presentation/middleware/params.ts` — `intParam`, `intQuery`, `requireBody`, `requireIntQuery` helpers that validate and parse common request inputs

---

## Entry Points

| File | Purpose |
|------|---------|
| `src/index.ts` | Local development server (starts `@hono/node-server` on `PORT`) |
| `api/index.ts` | Vercel serverless handler — lazy-initializes DB + routers on first request |

The Vercel handler deduplicates concurrent cold-start calls using a single `initPromise`. If DB initialization fails, it resets the promise to allow a retry on the next request.

---

## Error Handling

Errors flow through `Result<T>` — a simple `Ok | Err` discriminated union defined in `core/Result.ts`. Services return `Result<T>` rather than throwing. Routes unwrap results and call `sendError(c, message, statusCode)` on failure.

Application errors use error codes from `core/ErrorCode.ts`, which maps each code to an HTTP status.

---

## Score Sheet Formula Engine

`helpers/scoresheetHelper.ts` implements a simple formula evaluator supporting:

- `SUM(col1, col2, ...)` — sum of named columns
- `IF(condition, trueVal, falseVal)` — conditional value
- Column cross-references by column name

Formulas are evaluated at read time; cells store raw values only.

---

## Rate Limiting

Login and 2FA verification endpoints use `hono-rate-limiter` (backed by Redis via `ioredis`) to limit brute-force attempts.

---

## Authentication Flow

```
POST /api/auth/login
  → { token, role, ... }          # 2FA disabled

  → { requiresTwoFactor: true }   # 2FA enabled
      ↓
POST /api/auth/verify-2fa
  → { token, role, ... }
```

All subsequent requests include `Authorization: Bearer <token>`.

Token refresh: `POST /api/auth/refresh` (requires a valid, non-expired token).
