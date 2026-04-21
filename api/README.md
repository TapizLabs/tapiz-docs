# tapiz-rest-api

Backend for **Tapiz** — an attendance tracking and academic management platform for university courses. Built with [Hono](https://hono.dev/) on Node.js, TypeScript, TypeORM, and PostgreSQL.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Development](#development)
- [Production Build](#production-build)
- [Environment Variables](#environment-variables)
- [Database Setup](#database-setup)
- [First Assistant Account](#first-assistant-account)
- [Architecture](./ARCHITECTURE.md)
- [API Reference](./API_REFERENCE.md)
- [Database Schema](./DATABASE.md)

---

## Prerequisites

- Node.js ≥ 18
- PostgreSQL ≥ 14
- npm ≥ 9
- Redis (used by `hono-rate-limiter` via `ioredis`)

---

## Installation

```bash
git clone https://github.com/your-org/tapiz-rest-api.git
cd tapiz-rest-api
npm install
```

---

## Development

```bash
cp .env.example .env
# Fill in your DB credentials and secrets (see Environment Variables below)
npm run dev
```

The server starts on `http://localhost:3001` by default.

To run **type-checking only** (no emit):

```bash
npm run typecheck
```

---

## Docker (PostgreSQL)

Spin up a local PostgreSQL instance quickly with Docker:

```bash
docker run -d \
  --name tapiz-pg \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=root1234 \
  -e POSTGRES_DB=odp_db \
  -p 5432:5432 \
  -v tapiz_pg_data:/var/lib/postgresql/data \
  postgres:latest
```

---

## Production Build

```bash
npm run build   # compiles TypeScript → dist/
npm start       # runs dist/index.js
```

---

## Environment Variables

Copy `.env.example` to `.env` and fill in the required values.

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PORT` | | `3001` | HTTP server port |
| `CLIENT_URL` | | `*` | CORS allowed origin (set to your frontend URL in production) |
| `DB_HOST` | ✓ | | PostgreSQL host |
| `DB_PORT` | | `5432` | PostgreSQL port |
| `DB_USER` | ✓ | | Database username |
| `DB_PASSWORD` | ✓ | | Database password |
| `DB_NAME` | ✓ | | Database name |
| `DB_SSL` | | `false` | Set `true` to enable SSL (required for hosted databases like Aiven) |
| `JWT_SECRET` | ✓ | | Secret for signing JWTs — use a long random string |
| `SALT_ROUNDS` | | `10` | bcrypt salt rounds |
| `QR_EXPIRY_SECONDS` | | `300` | QR code validity window in seconds |
| `SEED_SECRET_KEY` | ✓ | | One-time secret to create the first assistant account |
| `MAIL_SERVICE_URL` | | | Base URL for the external email microservice (used for 2FA codes) |
| `ENV_TYPE` | | | Set to `DEV` to enable TypeORM query logging |
| `DEFAULT_STUDENT_PASSWORD` | | | Default password assigned when an assistant resets a student's password |
| `NPM_PACKAGE_VERSION` | | | Injected at build time for `/api/health` version reporting |

---

## Database Setup

Run the SQL migration files in order from the `sql/` directory:

```bash
psql -U <user> -d <database> -f sql/01_create.sql
psql -U <user> -d <database> -f sql/02_indexes.sql
```

`01_create.sql` creates all tables and constraints. `02_indexes.sql` adds performance indexes.

To reset the database and start over, drop all tables manually or recreate the database:

```bash
dropdb <database> && createdb <database>
psql -U <user> -d <database> -f sql/01_create.sql
psql -U <user> -d <database> -f sql/02_indexes.sql
```

See [DATABASE.md](./DATABASE.md) for the full schema reference.

---

## First Assistant Account

After the database is set up, seed the first assistant account using the `SEED_SECRET_KEY` you defined in `.env`:

```bash
curl -X POST http://localhost:3001/api/auth/seed-assistant \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@university.edu",
    "password": "yourpassword",
    "firstName": "First",
    "lastName": "Last",
    "secretKey": "<SEED_SECRET_KEY>"
  }'
```

This endpoint can only be called once per secret key and is designed for bootstrapping only.

---

## Health Check

```
GET /api/health
→ { "status": "ok", "ts": "2025-01-01T00:00:00.000Z" }
```
