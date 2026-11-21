# CampusConnect

CampusConnect is a full-stack campus opportunity and student-career portal. It brings opportunity discovery, student profiles, application tracking, campus announcements, administrator tools, and an AI Resume Analyzer into one workspace.

## Main features

- Manus OAuth sign-in and role-based access for students and administrators.
- Student profile management, including branch, year, skills, interests, and bio.
- Searchable opportunity discovery across internships, jobs, hackathons, technical events, workshops, and competitions.
- Category, deadline, and profile-based eligibility filters; saved opportunities and application tracking.
- Administrator tools for opportunities, announcements, and student accounts.
- AI Resume Analyzer for PDF resume extraction, analysis, skill-gap insights, and opportunity recommendations.
- MySQL persistence through Drizzle ORM, with checked-in schema and migrations.

## Technology stack

React 19, TypeScript, Vite, Tailwind CSS, Express, tRPC, Drizzle ORM, MySQL, Wouter, and Manus OAuth/platform services. Package management uses pnpm.

## Requirements

- Node.js 20 or later.
- pnpm 10 (the repository pins `pnpm@10.18.0`).
- A MySQL-compatible database for persistent application data.
- Valid Manus project OAuth and service configuration for authenticated and platform-backed features. No credentials are included in this archive.

## Local installation and setup

1. Install dependencies:

   ```bash
   pnpm install
   ```

2. Copy `.env.example` to `.env` and fill in the values from your own development setup. Never commit `.env` or put real credentials into `.env.example`.

3. Create a dedicated local/development MySQL database and set `DATABASE_URL` to its connection string. Do not point local migrations at production data.

4. Apply the checked-in migrations:

   ```bash
   pnpm db:migrate
   ```

   On an empty database, the application may seed clearly labeled demo opportunities and announcements.

5. Configure the Manus OAuth project and API values listed in `.env.example` for sign-in and platform-backed services. Some features will not work until valid project credentials and callback settings are configured.

## Run locally

Start the development server:

```bash
pnpm dev
```

The app listens on `http://localhost:3000` by default; set `PORT` to use a different port. Production-style build and start commands are:

```bash
pnpm build
pnpm start
```

## Validation commands

```bash
pnpm check
pnpm test
```

## Database and security notes

The schema is in `drizzle/schema.ts` and migrations are in `drizzle/`. Use a disposable development database when applying migrations or testing workflows. Keep all credentials server-side; do not publish `.env`, database dumps, uploaded resumes, or real student data. `OWNER_OPEN_ID` is an optional account identifier used for administrator bootstrapping in a trusted deployment; do not set it to an untrusted account.
