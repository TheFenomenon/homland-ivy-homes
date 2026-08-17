# Homland Ivy Homes — Milestone 3: Remove Land Scheme + Build Training System

## Build/validation status
- `npx tsc --noEmit` -> 0 errors.
- `npm run build` -> compiles clean, 6 routes generated: `/`, `/contact`, `/ivy-city`,
  `/training`, `/training/[slug]` (SSG'd for `brokers-training-jul-2026`), `/_not-found`.
  (Google Fonts fetch was stubbed only to validate compilation in this sandbox, which has
  no internet access to fonts.googleapis.com — Vercel's real build servers fetch fonts
  normally, no code change needed.)
- One real bug caught and fixed by the build: Next.js 15 made dynamic route `params` an
  async Promise in the App Router. `app/training/[slug]/page.tsx` and its
  `generateMetadata` were updated to `await params` accordingly — this would have been a
  runtime type error otherwise.

## Part 1 — Land Scheme removal (complete)
Full codebase grep for "land scheme" / "land_scheme" / "/land-scheme" before and after
changes. Confirmed zero remaining references in active application code after cleanup.

Removed:
- `app/land-scheme/` route directory — deleted entirely (not hidden, not redirected).
- Land Scheme nav item — `components/SiteHeader.tsx`.
- Land Scheme homepage section — `app/page.tsx` (section deleted, not CSS-hidden; homepage
  rebalanced so Training now takes the alternating sand-background slot Land Scheme used to
  occupy, keeping the section rhythm intact rather than leaving a gap).
- Land Scheme's payment-plan mention that had leaked into the homepage's value-prop copy
  ("spread ownership over 10 to 24 months") — replaced with a non-Land-Scheme point about
  land-only vs. land+home options.
- `land_scheme` WhatsApp context and message — `lib/whatsapp.ts`.
- Stale reference in this README's own prior revision.

**Database:** inspected the `payment_plans` table (Supabase project `eazlsmnocmawfcwinjre`).
Found 3 rows (10/12/24-month plans) — all `status = 'draft'`, never published, and the
`leads` table has zero rows total (nothing ever referenced them). Per the "local/static
seed records, remove if safe" rule, these were deleted. No production/customer data was
touched — there wasn't any.

**Schema note (flagging per "STOP and report" rule, not acted on):** `leads.interest` still
has `'land_scheme'` as an allowed value in its check constraint. Removing it is a schema
change I did not make without approval — it's harmless to leave (nothing in the UI can
generate that value anymore), but let me know if you want it dropped from the constraint
too.

**Analytics:** no dedicated Land Scheme analytics events existed to disable (the
event-tracking layer itself hasn't been wired to a provider yet, per the original plan).

## Part 2 — Training system (complete)
- `/training` — directory/hub page: intro to the Wednesday masterclass programme (using
  the verified "Become a Real Estate Professional" flyer content — host, free access,
  Wednesdays 11am), plus a list of actual dated training sessions below it.
- `/training/[slug]` — detail page: date, time, venue, host, trainers, status. Completed
  sessions are visually and textually marked as completed (no registration CTA shown), not
  presented as upcoming.
- `lib/training-data.ts` — single source of truth for training content, static for now
  (mirrors the `training`/`trainers` tables already seeded in Supabase), documented in-file
  as a placeholder for a future live Supabase query once env vars/CMS are wired. No new
  database schema was created — reused the existing `training`/`trainers` tables.

**Content honesty note:** only one training record has a client-confirmed date — the
Brokers' Training on 29 Jul 2026 (already in the past relative to today, correctly shown as
Completed). The other supplied flyer ("Become a Real Estate Professional," Wednesdays
11am) has no specific date, so it's used only as general programme description on the hub
page, not as an invented dated session — consistent with "do not invent missing
information."

## Assets used
No new images this milestone — training pages are currently text/data-only (no training
photo was supplied that isn't already the flyer graphic itself, which would repeat the
promotional-material-as-photography issue already flagged earlier in this project).

## WhatsApp
Training CTA message updated to match this milestone's exact spec: "Hello Homland Ivy
Homes, I am interested in the upcoming training. Please send me the details." Centralized
in `lib/whatsapp.ts`, no hardcoded numbers/messages elsewhere.

## SEO
`/training` and `/training/[slug]` both have title/description via the Metadata API;
`/training/[slug]` generates metadata per-training from `training-data.ts`. No sitemap
generation exists yet in this codebase (not built in any prior milestone), so nothing to
update there — flagging as still outstanding, not silently skipped.

## Not touched this milestone
Events, Insights/Blog, CMS/admin — per explicit instruction not to proceed to them.

## Push instructions (unchanged — I still have no GitHub write access)
Commit message: `feat: remove discontinued land scheme and add training`
Push to `Homland`. Vercel deploys automatically from the push.
