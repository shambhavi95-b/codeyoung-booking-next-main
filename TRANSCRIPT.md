# TRANSCRIPT.md

# CodeYoung Trial Class Booking — AI Development Transcript

**Project:** CodeYoung Trial Class Booking System  
**Repository:** `codeyoung-booking-next-main`  
**AI Assistant:** ChatGPT

> **Accuracy note:** This file was prepared from the ChatGPT project context currently available. The original shared conversation titled “ChatGPT - Run Project In Cmd” is accessible only as a shared page title, not as its full message body. Therefore unavailable earlier messages are summarized rather than fabricated. The later debugging conversation is represented directly. No secrets or API keys are included.

---

## 1. Assignment Understanding

### User — Assignment

The CodeYoung Full-Stack Engineer assignment required a trial-class appointment booking system.

Required flow:

1. Parent chooses a convenient time slot.
2. An available mentor is assigned.
3. Parent and mentor receive a dummy live-class link.

Requirements included:

- 10 mentors available for trial classes.
- 20 interested parents per day.
- Parents and mentors may be in different time zones.
- Parents are usually in the US/UK and mentors are in India.
- Local times must be displayed correctly.
- Daylight Saving Time must be handled.
- Mentors have at most 2 demo classes per day.
- Appropriate error handling when no mentor is available.
- GitHub repository with `README.md`.
- Full AI session transcript as `TRANSCRIPT.md`.

### AI — Development Direction

The project was approached as a full-stack booking application with a frontend, server-side API logic, PostgreSQL/Supabase storage, timezone-aware scheduling, mentor availability, booking protection, and user-friendly error handling.

---

## 2. Project Architecture

### User

I wanted to build the assignment as a real working project rather than a static UI.

### AI

The project was structured around:

- Next.js
- React
- TypeScript
- Next.js API routes
- Supabase/PostgreSQL
- Server-side Supabase access
- Timezone-aware scheduling
- Mentor and booking records
- API endpoints for available slots and bookings

The project directory became:

`codeyoung-booking-next-main`

---

## 3. Project Setup

### User

I worked through project setup and ran the application from the VS Code terminal.

### AI

The development server was run with:

```powershell
npm run dev
```

The application was tested locally in the browser.

---

## 4. Environment Variables

### User

The application showed:

```text
Missing NEXT_PUBLIC_SUPABASE_URL or SUPABASE_SERVICE_ROLE_KEY.
Copy .env.local.example to .env.local and fill them in.
```

### AI

The required local environment variables were:

```env
NEXT_PUBLIC_SUPABASE_URL=...
SUPABASE_SERVICE_ROLE_KEY=...
```

The correct filename was:

```text
.env.local
```

and it had to be in the project root.

---

## 5. `.env.local` Debugging

### User

The application still returned HTTP 500 from:

```text
/api/slots
```

with a request such as:

```text
GET /api/slots?date=2026-09-27&timezone=America%2FNew_York 500
```

### AI

The environment file was inspected with PowerShell.

The important discovery was:

```text
Attributes     : Directory
PSIsContainer  : True
```

This showed that `.env.local` had accidentally been created as a directory rather than a file.

The incorrect directory was removed and a real `.env.local` file was created.

The correct verification was:

```text
PSIsContainer : False
```

### Security

A Supabase service-role secret was accidentally visible in a screenshot during debugging. That credential should be treated as compromised and rotated/revoked. The replacement key belongs only in `.env.local` and must never be committed to GitHub.

---

## 6. Supabase Connection

### User

After fixing `.env.local`, the application loaded, but clicking **Show available times** produced:

```text
Couldn't book that slot

Could not find the table 'public.mentors' in the schema cache
```

### AI

This showed that the environment configuration was now being read and the application was reaching Supabase.

The problem had moved from environment configuration to database setup.

---

## 7. Empty Supabase Database

### User

The Supabase Table Editor showed:

```text
No tables or views
```

in the `public` schema.

### AI

The application expected:

```text
public.mentors
```

but the database was empty.

Instead of manually creating only `mentors`, the project's database SQL files were searched so the intended schema could be used.

---

## 8. Finding the Database Schema

### User

I searched the project for:

```text
create table
```

The search found:

```text
supabase/schema.sql
supabase/seed.sql
```

`schema.sql` contained `CREATE TABLE` statements.

### AI

The correct order was:

```text
schema.sql
    ↓
create database structure
    ↓
seed.sql
    ↓
insert initial/sample data
```

---

## 9. Running `schema.sql`

### User

I copied `supabase/schema.sql` into Supabase SQL Editor and ran it.

The result was:

```text
Success. No rows returned
```

### AI

This is a normal successful result for a schema-creation script. `CREATE TABLE`, `CREATE INDEX`, `ALTER TABLE`, and similar statements normally do not return rows.

The next check was the Supabase Table Editor.

---

## 10. Database Booking Logic

The project schema included booking structures and constraints.

The visible SQL included:

```sql
create table if not exists bookings (
```

and a unique constraint protecting mentor/slot combinations.

It also included indexes and row-level security configuration.

This database-level protection helps prevent two requests from booking the same mentor and slot.

---

## 11. Seed Data

### User

The project contained:

```text
supabase/seed.sql
```

with initial mentor-related data.

### AI

The next step after successful schema creation was to execute the existing `seed.sql`.

The intended sequence was:

```text
1. Run schema.sql
2. Confirm tables
3. Run seed.sql
4. Confirm mentor records
5. Test available slots
6. Test booking
```

---

## 12. Timezone Handling

### User

The booking UI contained a timezone selector and date.

API requests included values such as:

```text
/api/slots?date=2026-09-27&timezone=America%2FNew_York
```

and:

```text
/api/slots?date=2026-09-28&timezone=America%2FNew_York
```

### AI

Timezone handling is a core requirement.

The application should:

- accept the parent's timezone,
- interpret the requested date in that timezone,
- compare mentor availability consistently,
- display the appropriate local time,
- use IANA timezone identifiers,
- account for daylight-saving changes.

Examples:

```text
America/New_York
Europe/London
Asia/Kolkata
```

---

## 13. Frontend Booking Flow

The working UI displayed:

- CodeYoung Trial Class Booking
- Your name
- Your email
- Your timezone
- Date
- Show available times

The intended flow was:

```text
Parent opens booking page
        ↓
Enters name and email
        ↓
Selects timezone
        ↓
Selects date
        ↓
Requests available times
        ↓
Backend checks mentor availability
        ↓
Available slots are displayed
        ↓
Parent selects a slot
        ↓
Booking is created
        ↓
Mentor is assigned
        ↓
Dummy class link is returned
```

---

## 14. Error Handling

The browser displayed:

```text
Couldn't book that slot
Could not find the table 'public.mentors' in the schema cache
```

This was used to identify the database configuration problem.

The application should provide appropriate states for:

- no mentors available,
- slot already booked,
- invalid date,
- invalid timezone,
- database failure,
- booking failure.

---

## 15. GitHub Repository

### User

I asked for a GitHub repository description.

### AI

Suggested description:

> Full-stack trial class booking platform built with Next.js, TypeScript, and Supabase featuring timezone-aware scheduling and real-time mentor availability.

Suggested repository name:

```text
codeyoung-trial-class-booking
```

---

## 16. README

The repository should contain a `README.md` covering:

- project overview,
- features,
- technology stack,
- prerequisites,
- installation,
- environment variables,
- Supabase setup,
- database schema,
- seed data,
- local development,
- testing,
- deployment information where applicable.

The real `.env.local` must never be committed.

---

## 17. AI Transcript Requirement

### User

The assignment states that the solution must include a full transcript of AI sessions containing both prompts and agent responses, submitted as:

```text
TRANSCRIPT.md
```

### AI

`TRANSCRIPT.md` should preserve the actual AI-assisted development history, including:

- developer prompts,
- AI responses,
- architecture discussions,
- implementation discussions,
- debugging,
- database setup,
- testing,
- fixes and decisions.

The actual transcript should not contain invented conversations.

---

## 18. Current Project Status

### Completed / verified

- Next.js application runs locally.
- Booking UI loads.
- `.env.local` directory problem was identified and fixed.
- Supabase environment configuration was corrected.
- Application reached Supabase successfully.
- Empty Supabase database was identified.
- `supabase/schema.sql` was found.
- `schema.sql` executed successfully.
- Supabase reported:
  `Success. No rows returned`.

### Remaining verification

- Confirm all required tables are visible.
- Run `supabase/seed.sql`.
- Confirm mentor records exist.
- Test `/api/slots`.
- Verify timezone conversion.
- Test DST-sensitive dates.
- Verify maximum two demo classes per mentor per day.
- Verify double-booking protection.
- Verify no-mentor error handling.
- Verify successful booking.
- Verify dummy meeting link.
- Complete README.
- Complete the final verbatim AI transcript using available exports.

---

## 19. Final Testing Checklist

### Functional

- [ ] Parent can enter name.
- [ ] Parent can enter email.
- [ ] Parent can select timezone.
- [ ] Parent can select date.
- [ ] Available mentor slots are returned.
- [ ] Parent can select a slot.
- [ ] Booking succeeds.
- [ ] Mentor is assigned.
- [ ] Dummy meeting link is generated/returned.
- [ ] Parent and mentor local times are handled correctly.

### Scheduling

- [ ] Mentor cannot be double-booked.
- [ ] Mentor has no more than two demo classes per day.
- [ ] Different parent timezones are handled.
- [ ] India mentor timezone is handled.
- [ ] US/UK parent timezones are handled.
- [ ] DST-sensitive dates are tested.
- [ ] No-availability state works.

### Database

- [ ] `schema.sql` executes successfully.
- [ ] Required tables exist.
- [ ] Seed data exists.
- [ ] Booking constraints work.
- [ ] Database errors are handled.

### Security

- [ ] `.env.local` is not committed.
- [ ] `.env.local` is listed in `.gitignore`.
- [ ] Service-role credentials are server-only.
- [ ] Any accidentally exposed service-role key has been rotated.
- [ ] No API secrets appear in README or transcript.

### Submission

- [ ] GitHub repository is accessible.
- [ ] `README.md` exists.
- [ ] `TRANSCRIPT.md` exists.
- [ ] Setup instructions work from a clean environment.
- [ ] No secrets are committed.

---

## 20. Recommended Repository Structure

```text
codeyoung-booking-next-main/
│
├── app/
├── components/
├── lib/
├── public/
│
├── supabase/
│   ├── schema.sql
│   └── seed.sql
│
├── .env.local.example
├── .gitignore
├── package.json
├── README.md
└── TRANSCRIPT.md
```

`.env.local` must remain local and must not be committed.

---

## 21. Transcript Integrity Note

This document intentionally distinguishes between:

1. **Available conversation material**, which can be represented directly; and
2. **Earlier project conversation material that is only available as project context/summary**, which is summarized.

For the assignment's strict requirement of a **full verbatim transcript**, the original exported AI sessions should be added to this file where available. Unavailable historical prompts and responses should not be fabricated.
