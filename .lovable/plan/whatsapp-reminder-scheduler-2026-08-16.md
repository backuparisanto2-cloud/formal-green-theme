# WhatsApp Reminder Scheduler

A responsive white-and-green WhatsApp-style dashboard (formal, elegant, exclusive) for scheduling reminder messages: one-time, recurring, contacts and groups, plus a delivery log.

## Important note on sending

whatsapp-web.js requires Puppeteer and a long-running Node process, which this hosting environment cannot run. So this build delivers the full product — UI, accounts, scheduling engine, and a due-message queue — with the actual send step stubbed behind a single adapter. Later you point that adapter at your own whatsapp-web.js server (or a hosted API) without changing the rest of the app.

## What gets built

**Auth**
- Email/password login and signup, each user only sees their own data.
- Protected dashboard area; unauthenticated visitors land on the login page.

**Contacts & groups**
- Add contacts (name, phone in international format, notes).
- Create groups and assign contacts to them.
- Reminders can target a contact or a whole group.

**Reminders**
- One-time: pick exact date and time, timezone-aware (Asia/Jakarta default).
- Recurring: daily, weekly (choose weekdays), monthly (choose day), or custom cron expression.
- Message body with simple placeholders like {name}.
- Optional end date / max occurrences, plus pause and resume.
- List view with next-run time, status badges, edit and delete.

**Delivery log**
- Every scheduled attempt recorded: queued, sent, failed, with timestamp and error text.
- Filter by reminder, status, and date range; manual retry on failures.

**Scheduler**
- A recurring job computes due occurrences and enqueues them.
- A public, secret-protected endpoint your WhatsApp worker can call to pull pending messages and post back delivery status — that is the bridge for whatsapp-web.js later.

**Design**
- White base, WhatsApp green accents (deep #075E54-family plus bright accent), generous whitespace, subtle depth, refined typography.
- Fully responsive: mobile-first cards with a bottom-safe layout, sidebar navigation on desktop.
- Screens: Login, Dashboard overview, Reminders, Contacts & Groups, Delivery Log, Settings (timezone, connection status).

## Technical notes

- Lovable Cloud (Postgres + auth) for storage; tables: `profiles`, `contacts`, `groups`, `group_members`, `reminders`, `reminder_occurrences`, `message_logs`. RLS on every table scoped to `auth.uid()`, plus explicit grants.
- Schedules stored as timezone + rule (one-time timestamp or recurrence rule); next-run computed server-side.
- Scheduling and queue logic in server functions; the worker bridge lives at `/api/public/whatsapp/*` with a shared-secret header check and Zod validation.
- Sending goes through one `sendWhatsAppMessage` adapter that currently marks messages as "pending external worker"; swapping in your endpoint URL is the only change needed to go live.

## Out of scope for now

- Running whatsapp-web.js itself, QR pairing inside this app, and media attachments. The QR/pairing screen can be added once your worker exists and exposes its session state.
