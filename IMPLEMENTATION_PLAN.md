# Idealtime Bookings — Implementation Plan

Source: `app doc/IDEALTIME BOOKINGS APP.md` (PRD v2.0, 46 sections, two user groups: Guests + Hosts).
Status: greenfield. This plan orders the work from design system → architecture → feature phases, each with concrete outputs and acceptance criteria.

---

## Architectural decisions (decide first, before Phase 1)

| # | Decision | Recommendation | Rationale |
|---|----------|----------------|-----------|
| AD-1 | Web / mobile strategy | **Mobile-first responsive web app first**, wrap as PWA; native apps later | Nigerian discovery happens on mobile web/WhatsApp; PWA ships fastest, one codebase |
| AD-2 | Frontend | **Next.js (App Router) + TypeScript + Tailwind CSS** | SEO for property discovery, server components for listing pages, large hiring pool |
| AD-3 | Backend | **Supabase (Postgres + Auth + Storage + Realtime)** | Auth, row-level security for guest/host separation, file storage for photos/ID, realtime for messaging/booking updates without ops overhead |
| AD-4 | Payments | **Paystack** (primary) + Flutterwave (fallback) | Paystack: cards, bank transfer, USSD — the methods Nigerian guests actually use; supports deposits/split logic via separate charge records |
| AD-5 | SMS/OTP + WhatsApp | **Termii** (OTP/SMS) + **WhatsApp Business Cloud API** (notifications) | Termii is Nigeria-native; WhatsApp is PRD §26 core channel. Booking/payment record stays in-app; WhatsApp is notify-only |
| AD-6 | Identity verification | Third-party KYC (Smile Identity / VerifyMe) behind a verification-provider interface | Don't build KYC; abstraction lets you swap vendors |
| AD-7 | Repo layout | Monorepo: `apps/web`, `packages/ui`, `packages/config`, `supabase/` (migrations), `docs/adr` | Design system ships as a real package from day one |
| AD-8 | Availability model | **Block-based calendar**: `availability_blocks` (available/pending/unavailable); confirmed bookings auto-write blocks in the same DB transaction | PRD §18: prevents double booking at the DB level, not just UI |
| AD-9 | Money handling | All amounts in **kobo (integers)**; separate `booking_charges` (nightly, cleaning, service — non-refundable) from `caution_fee` (refundable) | PRD §19: refundable vs non-refundable must never mix |

**Concrete outputs:** `docs/adr/AD-1 … AD-9.md`, scaffolded monorepo, CI (lint + typecheck + tests), staging + production Supabase projects, Paystack test keys wired.

**Acceptance:** `pnpm build` green, staging URL renders, test payment succeeds in sandbox.

---

## Phase 1 — Design system

PRD anchor: brand (§1: professional, trustworthy, modern) + trust badges (§13–16).

**Concrete outputs (`packages/ui` + Storybook):**
- Design tokens: color (incl. Verified green / Pending amber / Unavailable red), type scale, spacing, radius, shadows — as Tailwind theme + CSS variables.
- Components: Button, Input, PhoneInput (+234 default), Badge (`VERIFIED`, `READY`, `PENDING`), PropertyCard, PriceBreakdown (nightly × n + cleaning + service + caution = total), AvailabilityCalendar (month grid, 3 states), RatingStars, StayTimeline, BookingStatusChip, EmptyState, BottomSheet (mobile actions).
- Listing-page template + booking-summary template composed from the above.

**Acceptance:** every component has a Storybook story + mobile (360px) snapshot; PropertyCard and PriceBreakdown match PRD §12/§19 fields exactly.

---

## Phase 2 — Data model + API contracts

PRD anchor: everything (this is the foundation all phases build on).

**Concrete outputs:**
- ERD (`docs/erd.md`) + Supabase migrations. Core tables:
  - `users` (role: `guest` | `host`), `host_profiles` (subtype: owner/manager/agent + verification status), `business_profiles` (company + employees + spend limits)
  - `properties`, `property_photos`, `property_amenities`, `verification_records`, `inspections`
  - `availability_blocks`, `price_rules`, `bookings`, `booking_guests` (book-for-someone-else), `booking_charges`, `payments`, `receipts`
  - `reviews` (guest→property, host→guest), `messages`, `cleaning_tasks`, `maintenance_issues`, `incidents`, `promotions`, `services` (travel add-ons), `bundles`
- RLS policies: guests see only published properties + own bookings; hosts see only own properties/bookings; agent-hosts scoped to authorized properties.
- API contract doc (`docs/api.md`): REST/RPC shapes for search, availability, booking quote → reserve → pay → confirm.

**Acceptance:** migrations apply cleanly; RLS tests prove cross-host data isolation; seed script creates a demo host + verified property + booking.

---

## Phase 3 — Auth, guest verification, trust profiles (PRD §7–9)

**Concrete outputs:**
- Guest registration (phone OTP via Termii, email, Google), host application flow with subtype selection.
- KYC hookup (ID + selfie) behind provider interface; verification statuses (`unverified/basic/identity-verified`).
- Guest Trust Profile UI (phone ✓, ID ✓, stays, reviews) and Property Trust Profile UI (host verified, inspected, bookings, rating) — non-sensitive fields only.

**Acceptance:** new user books a test flow phone-only; KYC-verified badge appears; no ID document URL is reachable by other users (test).

---

## Phase 4 — Property listing + discovery (PRD §10–12, §17)

**Concrete outputs:**
- Host property CRUD (details, photos/video upload, amenities incl. Nigeria-specific: generator/inverter/solar, house rules, price rules, cancellation policy).
- Public search (city/area/estate/landmark/airport/name), filters (type, bedrooms, price, amenities, purpose), listing page (§12 field list), side-by-side comparison (§17, up to 3).
- Verification workflow UI (internal): checklist → `VERIFIED` + last-inspected date (§13–14); `READY` flag workflow (§15).

**Acceptance:** seeded Lagos properties searchable by "Ikeja"/"Near Lagos Airport"; unpublished properties invisible; comparison shows price/cancellation/verification deltas.

---

## Phase 5 — Availability, pricing, booking, payments (PRD §18–24) — *MVP core*

**Concrete outputs:**
- Availability calendar with transactional blocking (§18).
- Price quote engine → booking summary (§19 format) → six booking modes (§20: Instant, Request-to-Book, Pay Now, Deposit, Pay on Arrival, Corporate Billing).
- Paystack integration (inline + transfer/USSD), webhook handler, booking confirmation (ITB-number), digital receipt (§23), My Bookings tabs (§24), book-for-someone-else (§21).

**Acceptance (MVP demo script):** search → verified property → available dates → quote matches receipt to the kobo → pay test card → confirmation + WhatsApp-style notification logged → dates blocked → second concurrent booking of same dates rejected.

---

## Phase 6 — Stay management (PRD §25–27, §31)

**Concrete outputs:** Stay Timeline screen (§25), check-in pack + check-out flow incl. caution status (§31), in-app guest↔host messaging (§27, payments blocked from chat), WhatsApp notification service (confirmation, payment, check-in/out reminders, driver info — §26).

**Acceptance:** timeline renders airport-pickup → check-in → cleaning → check-out → review for a test booking; all notifications also persisted in-app.

---

## Phase 7 — Host operations (PRD §33–39)

**Concrete outputs:** Host dashboard (today/upcoming, revenue, occupancy, requests, reviews), multi-property switcher, cleaning task pipeline (assign → start → photo proof → approve → `READY`), maintenance log (status/assignee/cost), revenue dashboard (gross, fees, net, payouts, occupancy, ABV), agent-host view (clients, bookings, commissions, transaction history).

**Acceptance:** checkout → cleaning task → photos → approval flips property back to `READY`; revenue math reconciles with payments table.

---

## Phase 8 — Reviews, incidents, protection, support (PRD §28–30, §32)

**Concrete outputs:** dual reviews (guest→property on 7 dimensions, host→guest), booking-linked eligibility check; incident tickets (IT-number, photo/video, status); Stay Protection policy page + relocation/refund workflow stubs; Help Centre + live chat + WhatsApp/phone handoff.

**Acceptance:** review only submittable post-checkout; incident with photos creates trackable ticket; policy terms page exists pre-launch (legal sign-off flagged).

---

## Phase 9 — Corporate, travel services, growth (PRD §40–46)

**Concrete outputs:** business accounts (employees, authorized bookers, spend limits, invoices), corporate booking record; bookable add-ons (transfers, car, driver, cleaning, chef, events, holiday, relocation) attachable to bookings; stay bundles; promotions engine (weekend/long-stay/early/last-minute/corporate/5+1); recommendations (previous bookings, location, budget — transparent, no dark patterns).

**Acceptance:** corporate booker books within spend limit and downloads invoice; bundle books apartment + pickup + cleaning as one confirmation.

---

## Phase 10 — Hardening + pilot launch

**Concrete outputs:** security pass (RLS audit, rate limits, webhook signature verification), performance pass (listing page LCP, image optimization), analytics events (search→view→book funnel), pilot runbook: 10–20 inspected Lagos properties, manual verification SOP, support rota, launch checklist.

**Acceptance:** pilot conversion funnel measurable; dispute drill (unavailable property → relocation/refund) completed.

---

## Suggested MVP cut (launchable end of Phase 5 + slices of 6–8)

Search → verified listing → availability → book → Paystack → confirmation/receipt → host dashboard → messaging → reviews → basic incidents. Corporate/services/bundles/promotions (Phase 9) stay post-MVP.

## Top risks

1. **Verification ops don't scale** — mitigate: inspection SOP + photo-evidence requirements from day one (§14).
2. **Double bookings** — mitigate: transactional blocks (AD-8), never trust client-side availability.
3. **Payment reconciliation** — mitigate: kobo integers, webhook idempotency, separate caution ledger (AD-9).
4. **WhatsApp API approval delays** — mitigate: in-app notifications first, WhatsApp as transport upgrade.
