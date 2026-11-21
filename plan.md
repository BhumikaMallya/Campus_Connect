# CampusConnect implementation plan

## Product and implementation
CampusConnect is a persistent student opportunity and application-tracking platform. Use the initialized React 19 + TypeScript + Wouter client, Express + tRPC server, Drizzle ORM and managed MySQL database. Preserve the starter's real Manus OAuth identity/session validation and extend application-level role and profile handling; do not simulate a user or introduce password auth. Students complete a persistent profile after login. The project owner remains the initial administrator per the starter's owner-role behavior; admin-only procedures guard listing review/CRUD, announcement publication, student account management and statistics.

Define additive Drizzle tables for student profiles, opportunities, saved opportunity joins, applications with student-editable statuses, and announcements. Expose validated tRPC routers using protected/student/admin procedures and enforce row ownership on every student-specific read/write. Seed realistic, unmistakably DEMO opportunities and announcements only when the relevant tables are empty; do not claim demo content is verified. Student pages cover dashboard, searchable/filterable category discovery, saves, application tracker, announcements, profile, eligibility and deadline-based recommendations, and career-readiness insights. Admin pages cover opportunity/announcement CRUD, review and verification, student account management and aggregated application counts. Recommendations explain matching signals (skills, branch, interests, eligibility); reminders are shown from persisted upcoming deadlines without claiming email delivery.

## Project structure
- `client/src/App.tsx`: route shell and dashboard/discovery/admin routes.
- `client/src/pages/`: dashboard, opportunities, saved, applications, announcements, profile, admin and auth/profile setup pages.
- `client/src/components/`: responsive sidebar, header, reusable data/status cards, forms, charts, and the compact `ResumeAnalyzer.tsx`/`.css` profile panel.
- `client/src/index.css`: CampusConnect navy/white/electric-blue visual system and responsive dashboard styling.
- `server/routers.ts`: typed tRPC API mount points for auth, profile, opportunities, saved items, applications, announcements, admin operations, and the focused resume router.
- `server/db.ts`: relational query helpers, seed-on-empty demo dataset, recommendation and stats queries.
- `server/resume.ts`: active-student-only analyzer and opportunity-option procedures, structured result validation, and server-side recommendation scoping.
- `server/_core/resumePdf.ts`: bounded, in-memory PDF signature/page/text extraction.
- `server/_core/llm.ts`: existing Manus-managed LLM wrapper, with the minimal completion-token support used by the analyzer.
- `drizzle/schema.ts` and `drizzle/`: normalized persistent schema and checked-in migration.
- `shared/`: shared enums, validation/types when needed.
- `public/manus-routes.json`: synchronized set of client page routes.

## Design
Three styles considered: editorial campus journal **0.08**, Swiss grid-and-type system **0.06**, mission-control analytics **0.05**. Chosen: **mission-control analytics**, tempered with a calm editorial dashboard so student actions stay clear rather than dense.

- **Design Movement:** Contemporary Swiss information design for a career platform.
- **Core Principles:** Fast scanning; visible provenance; accessible contrast; action-first hierarchy.
- **Color Philosophy:** Deep navy grounds trust, bright electric blue signals a next step, soft cool surfaces keep data legible; amber/orange is reserved for deadlines.
- **Layout Paradigm:** Fixed-width collapsible left rail on desktop and compact mobile navigation, broad workspace with a welcome/action header and asymmetric dashboard columns.
- **Signature Elements:** Small blue opportunity-type glyphs; deadline chips with clear urgency; progress/status rails on application cards.
- **Interaction Philosophy:** Every control has an immediate, understandable result; forms validate inline; status changes are explicit and reversible.
- **Animation:** Restrained 150–200 ms hover/focus transitions only; no decorative movement; honor reduced-motion preferences.
- **Typography System:** Inter for UI and headings, clear tabular numerals for metrics, compact labels in medium weight.
- **Brand Essence:** The campus-to-career command center for students who want one reliable place to find and follow opportunities; **focused, credible, encouraging**.
- **Brand Voice:** Specific, supportive and action-oriented. Examples: “Your next opportunity is closer than you think.” “Four deadlines are coming up—start with the closest.”
- **Wordmark & Logo:** A distinct linked-path monogram (two small nodes connected by a rising electric-blue line) alongside a sturdy CampusConnect wordmark.
- **Signature Brand Color:** Electric blue `#2563EB`.

## Approved AI Resume Analyzer implementation
- Add a compact analyzer panel to the existing authenticated student profile view; do not add a new navigation redesign or change the existing UI system.
- Reuse Manus OAuth, the current authenticated tRPC context, the existing student profile and approved opportunity records, and `server/_core/llm.ts`. New analysis procedures must require an active `user` role; no auth changes or admin access.
- Accept PDF bytes as a bounded base64 tRPC request, validate MIME/signature, size, page count and extractable text server-side with a PDF text library. Process only in memory; do not write the original PDF or transcript to object storage, the filesystem, logs or the database. Scanned/image-only PDFs receive a clear unsupported/OCR-needed response.
- For a student-initiated Analyze action, send extracted resume text, minimal profile context (branch/year/current skills/interests only), and bounded approved, unexpired internship/job descriptions to the existing Manus-managed LLM. Use only the explicitly available, live-catalog-verified `gpt-6-luna` model: it supports strict JSON-schema output and is rated low cost per task. Use low reasoning effort, validate structured output, and bound the resume text, opportunity count and output tokens to control usage. Report no numerical fit score.
- Return factual education, skills, projects and experience extraction; resume suggestions; selected-role matched skills/gaps; and AI-selected relevant opportunity IDs with explanations. Validate every recommendation ID against the actual approved opportunities queried from the database. Optionally let a student explicitly merge extracted skills into their existing profile through the existing profile-save procedure; never overwrite silently.
- Do not persist analyses or add database schema in this first version. Keep results only in page memory, show a brief disclosure that resume text/profile context are sent to Manus AI, and clear the file/base64 after the request settles.
- Extend `server/_core/llm.ts` narrowly to support the model-required `max_completion_tokens` request field. Add unit/regression coverage for PDF validation, ownership/role checks, output validation and opportunity scoping. Test once end-to-end with a generated synthetic sample resume, without database writes.
- Project changes are not permission to publish. Preserve `publishing.auto_publish: false` and do not publish without a separate explicit request.

## Material constraints
Demo records are fictional and explicitly marked “Demo data”; demo opportunities remain unverified and cannot be marked verified. New, sourced listings may be verified only after administrator review. Admin permission derives only from the authenticated database role. Authenticated web sessions retain the exact starter cookie name and valid project JWT checks; cookie attributes support HTTPS embedded Preview. All existing user content and workflow state persist in the managed database. Resume PDFs, extracted transcript text and analyzer results in this initial implementation are transient and are not persisted. No publication requested; do not publish.
