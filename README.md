# VBDA Dashboard — AI‑powered invitation workflow for VBDA 2025

A Next.js (App Router) dashboard for generating invitation copy, uploading recipient lists via CSV, and sending invitations via email providers. Built with a lightweight UI shell (sidebar + pages) and serverless API routes for email delivery and CSV parsing.

> **Status:** The repository contains working UI routes + API handlers, but a few flows are still wired with hardcoded values and “coming soon” placeholders. This README documents **only what exists in the current codebase**.

---

## Links

- **Repository:** https://github.com/RiyaKaushik-tech/vbda_dashboard

---

## Key Features

- **Dashboard layout with navigation sidebar** (Welcome, Create Invitation, Upload, Generate Invitation)
- **CSV recipient upload** via `/upload` UI and a multipart API route (`/api/upload`) that parses CSV into JSON
- **Template-based invitation generator** (role-based default messages with edit + copy-to-clipboard)
- **AI email generation + sending** via `/api/generate-email` using OpenAI + Resend
- **SMTP email sending** via `/api/send-email` using Nodemailer (SMTP credentials via env vars)
- **Drizzle ORM schema + Drizzle Kit config** (PostgreSQL dialect) for a `users` table (schema present; app usage not yet wired)

---

## Technical Architecture Overview

This project uses the **Next.js App Router** with:

- **UI routes** under `src/app/**/page.tsx`
- **Serverless API routes** under `src/app/api/**/route.ts`
- **Global layout** (`src/app/layout.tsx`) that composes a persistent `Sidebar` and a page content area
- **Styling** via Tailwind CSS v4 imported through `src/app/globals.css`

### Request flow (high level)

| Flow | UI Route | API Route | Core Libraries |
|------|----------|----------|----------------|
| Upload recipients | `/upload` | `POST /api/upload` | `csv-parse/sync` |
| Generate AI + send email | (used by `/template`) | `POST /api/generate-email` | `openai`, `resend` |
| Send email (SMTP) | `/create-invitation` | `POST /api/send-email` | `nodemailer` |

---

## Tech Stack

### Frontend
- **Next.js 15.3.1** (App Router)
- **React 19**
- TypeScript (project configured)

### Backend
- Next.js **Route Handlers** (`src/app/api/*/route.ts`) running server-side

### State Management
- Local component state via **React hooks** (`useState`, `useEffect`)

### APIs
- **OpenAI** (`openai` SDK) for generating invitation email text
- **Resend** for sending generated emails
- **SMTP** via Nodemailer for sending emails through a configured SMTP server

### Authentication
- None implemented in the current codebase.

### Styling
- **Tailwind CSS v4** (`@import "tailwindcss";` in `globals.css`)
- PostCSS + Autoprefixer

### Tooling
- ESLint (`eslint`, `eslint-config-next`)
- Drizzle Kit (`drizzle-kit`) + Drizzle ORM
- `dotenv` (used by Drizzle config)

---

## Folder Structure

```txt
vbda_dashboard/
├── drizzle.config.ts
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── postcss.config.mjs
├── public/
├── src/
│   ├── app/
│   │   ├── analytics/
│   │   │   └── page.tsx
│   │   ├── api/
│   │   │   ├── generate-email/
│   │   │   │   └── route.ts
│   │   │   ├── send-email/
│   │   │   │   └── route.ts
│   │   │   └── upload/
│   │   │       └── route.ts
│   │   ├── create-invitation/
│   │   │   └── page.tsx
│   │   ├── generate-invitation/
│   │   │   └── page.tsx
│   │   ├── upload/
│   │   │   └── page.tsx
│   │   ├── template/
│   │   │   └── page.tsx
│   │   ├── favicon.ico
│   │   ├── globals.css
│   │   ├── layout.tsx
│   │   └── page.tsx
│   ├── components/
│   │   └── Sidebar.tsx
│   └── db/
│       └── schema.ts
└── tsconfig.json
```

---

## Installation & Setup

### Prerequisites
- Node.js (recommended: latest LTS)
- npm (repo includes `package-lock.json`)

### Install
```bash
npm install
```

### Run locally
```bash
npm run dev
```

The dev script runs:
- `next dev --turbopack`

Open: http://localhost:3000

---

## Environment Variables

No `.env.example` is present in the repository. These variables are referenced in code and should be provided (e.g., via `.env.local`):

### Required

| Variable | Used In | Purpose |
|---------|---------|---------|
| `DATABASE_URL` | `drizzle.config.ts` | Drizzle Kit DB connection URL (PostgreSQL dialect) |
| `OPENAI_API_KEY` | `src/app/api/generate-email/route.ts` | OpenAI API access |
| `RESEND_API_KEY` | `src/app/api/generate-email/route.ts` | Resend API access |
| `SMTP_HOST` | `src/app/api/send-email/route.ts` | SMTP server host |
| `SMTP_PORT` | `src/app/api/send-email/route.ts` | SMTP server port |
| `SMTP_USER` | `src/app/api/send-email/route.ts` | SMTP username |
| `SMTP_PASS` | `src/app/api/send-email/route.ts` | SMTP password |

Example (create **`.env.local`**):
```bash
DATABASE_URL="postgres://user:password@host:5432/dbname"

OPENAI_API_KEY="..."
RESEND_API_KEY="..."

SMTP_HOST="smtp.example.com"
SMTP_PORT="587"
SMTP_USER="..."
SMTP_PASS="..."
```

---

## Usage Guide

### 1) Landing page
- Visit `/` to access the welcome screen and navigate to **Create Invitation**.

### 2) Upload recipients (CSV → JSON)
- Go to `/upload`
- Upload a CSV file via the UI
- The app calls `POST /api/upload` and renders parsed records as JSON

**CSV parsing behavior**
- Uses `csv-parse/sync`
- `columns: true` (expects a header row)
- `skip_empty_lines: true`

### 3) Generate a role-based invitation message (client-side templates)
- Go to `/generate-invitation`
- Choose a recipient type (Judge/Student/Guest/Faculty)
- Edit the generated message
- Copy to clipboard

### 4) Generate an AI email (OpenAI) and send via Resend
- Go to `/template`
- Click “Generate Sample Email”
- UI calls `POST /api/generate-email` and displays returned `email` text

> Note: The API route currently expects `recipients` in the request body, but the `/template` page does not send it. This is a current wiring gap in the repository.

### 5) Send email via SMTP (Nodemailer)
- Go to `/create-invitation`
- Submits a request to `POST /api/send-email`

> Note: Current implementation includes hardcoded sender/recipient and message text in both UI and API. The form fields are present but not yet used for generating/sending dynamic content.

---

## Engineering Highlights

- **Clear separation of concerns (UI vs API):** UI pages call internal Next.js route handlers (`/api/*`) for server-side operations (email sending, CSV parsing).
- **Multipart upload handling:** CSV upload uses `FormData` on the client and `req.formData()` on the server route.
- **Provider flexibility for email:** Two distinct sending paths exist:
  - SMTP via Nodemailer (`/api/send-email`)
  - API-based delivery via Resend (`/api/generate-email`)
- **Database groundwork:** Drizzle ORM schema and Drizzle Kit configuration are present (PostgreSQL dialect), setting the stage for persistence (e.g., storing recipients, send logs, users).

---

## Performance / Optimization Notes

- Next dev mode uses **Turbopack** (`next dev --turbopack`).
- CSV parsing is synchronous (`csv-parse/sync`), which is simple and predictable for small-to-medium files; for large uploads, a streaming parser would be preferable.

---

## Security Considerations

- **Secrets in environment variables:** SMTP credentials, OpenAI key, Resend key, and DB URL are read from `process.env`. Do not commit them.
- **PII handling:** Recipient emails and uploaded CSV data may contain personal data; consider retention policies and encryption at rest if persisted later.
- **Input validation:** Current API routes accept JSON/form-data with minimal validation. If exposed publicly, add schema validation (e.g., `zod`, already in dependencies) and rate limiting.
- **Hardcoded email addresses:** `/api/send-email` currently uses hardcoded `from`/`to`. Replace with validated inputs and allowlist/authorization before production use.

---

## Future Improvements

- Wire `/create-invitation` form fields into:
  - AI generation (`/api/generate-email`)
  - actual recipient list sending (batch delivery)
- Add `zod` validation for:
  - API request bodies (`generate-email`, `send-email`)
  - CSV shape (required headers like `email`)
- Persist recipients, send history, and analytics using **Drizzle + Postgres**
- Implement authentication/authorization for dashboard access
- Add real analytics (open/click tracking and RSVP capture) to replace the “Coming soon” page
- Add `.env.example` and deployment docs (Vercel, etc.)

---

## Author

**Riya Kaushik**
