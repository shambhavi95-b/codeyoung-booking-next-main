<<<<<<< HEAD
# CodeYoung Trial Class Booking — Next.js + shadcn/ui + Supabase

Full-stack rebuild of the assignment on:

- **Next.js 14 (App Router)** — UI + API routes in one project
- **shadcn/ui + Tailwind CSS** — styling/components
- **Supabase (Postgres)** — persistent storage for mentors & bookings
- **Luxon** — all timezone/DST math

## 1. Set up Supabase

1. Create a project at [supabase.com](https://supabase.com).
2. Open the SQL editor and run `supabase/schema.sql`, then `supabase/seed.sql` (seeds the 10 mentors).
3. Go to **Project Settings → API** and copy the **Project URL** and the **`service_role` key** (not the anon key — writes go through the server only).

## 2. Configure the app

```bash
cp .env.local.example .env.local
# then fill in NEXT_PUBLIC_SUPABASE_URL and SUPABASE_SERVICE_ROLE_KEY
```

## 3. Run it

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

## Architecture

```
app/
  page.tsx                  # server component shell
  api/mentors/route.ts      # GET  - list mentors
  api/slots/route.ts        # GET  - candidate slots for a parent-local date + live availability
  api/bookings/route.ts     # GET  - list bookings; POST - book a slot
components/
  booking-app.tsx           # client toggle: form <-> confirmation
  booking-form.tsx          # the booking UI (shadcn Card/Input/Select/Alert)
  confirmation.tsx
  ui/                       # shadcn primitives (button, input, label, select, card, alert)
lib/
  timezone.ts                # Luxon/DST helpers (pure functions, unit-testable)
  booking-logic.ts            # mentor-availability logic, shared by /api/slots and /api/bookings
  supabase-server.ts          # server-only Supabase client (service role key)
  types.ts
supabase/
  schema.sql / seed.sql
```

All Supabase access happens **only** inside `app/api/**/route.ts` files, via `lib/supabase-server.ts`. That file throws if imported into a `"use client"` component, since the service-role key must never reach the browser — the frontend only ever talks to Supabase indirectly, through these API routes.

## How the requirements were handled

**Mentors (10) × capacity (20 parents/day).** Each mentor row has `start_hour=9`, `end_hour=18` (IST), `max_classes_per_day=2` → 10 × 2 = 20/day, matching the brief. This isn't hardcoded logic — `findAvailableMentor()` in `lib/booking-logic.ts` always checks live booking counts from Postgres, so it adapts automatically if mentor data changes.

**Timezones + DST.** Every instant is stored as `slot_utc timestamptz` in Postgres — a single canonical UTC value. Local time is purely a display concern, computed with Luxon using real IANA zone names (`America/New_York`, `Europe/London`, `Asia/Kolkata`, …). Because DST is a property of a place rather than a fixed offset, letting Luxon resolve the offset for the *specific instant* means DST transitions are handled correctly with zero special-cased logic — see the comment block at the top of `lib/timezone.ts`.

**"Which day" a mentor's cap applies to.** The 2-classes/day cap is enforced against the **mentor's own IST calendar day** (`istDayKey()`), not the parent's day or UTC's day — a class at 9:30 PM EST lands at ~7 AM IST the next day and correctly counts against that next IST day, since that's the day it falls on from the mentor's schedule's point of view. This is a deliberate judgment call the assignment leaves open, documented inline.

**Race conditions.** Two safety nets: (1) the availability check re-reads bookings from Postgres on every request, and (2) a DB-level `unique (mentor_id, slot_utc)` constraint means even a genuine simultaneous double-booking attempt fails atomically at the database — the API catches Postgres error `23505` and returns a friendly 409 instead of a 500.

**No mentors available.** `POST /api/bookings` returns `409` with a clear message plus up to 3 real, currently-free alternative slots on the same parent-local day, computed live rather than guessed.

**Dummy links.** `https://demo.codeyoung.example/live-class/<bookingId>` — no real video infra, per the assignment's stated assumption. "Emails" are logged server-side (see terminal output after a booking) rather than actually sent.

## What was deliberately not built

Auth, real email delivery, an admin dashboard beyond the raw `GET /api/bookings`, calendar sync, and a custom date-picker component (the date field uses a native `<input type="date">` styled as a shadcn `Input` rather than a full shadcn `Calendar`/`Popover`, to keep the surface area focused on the actual evaluation criteria — timezone correctness, capacity logic, and graceful failure — rather than UI polish that wasn't asked for).

## AI usage

Built with an AI coding assistant. Per the submission instructions, export the full prompt/response transcript of this session and include it in the repo as `TRANSCRIPT.md` before submitting.
=======
# codeyoung-booking-next
A full-stack booking application built with Next.js, React, Tailwind CSS, and Supabase for managing mentor bookings, time slots, and appointments.
>>>>>>> 012f92446932931c6ebb15586d62daaf771f61f3
