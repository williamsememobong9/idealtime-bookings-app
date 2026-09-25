# Idealtime Bookings App

**Find. Verify. Book. Stay.**

Idealtime Bookings is a trusted accommodation marketplace and stay-management platform for Nigeria, with expansion across Africa in view. It connects customers with verified hotels, serviced apartments, short-let apartments, villas, and other stays, while giving owners, managers, agents, and corporate clients tools to manage listings, availability, bookings, guests, and revenue.

Full product spec: `app doc/IDEALTIME BOOKINGS APP.md` (PRD v2.0, 50 sections).

## Vision

Make accommodation booking in Nigeria more transparent, trustworthy, convenient, and professionally managed.

Built around five principles: Trust, Transparency, Availability, Convenience, Support.

## Who it serves

- **Guests:** business travellers, families, tourists, returning Nigerians, expatriates, contractors, students, relocating individuals, event attendees. Hotels, serviced/short-let apartments, vacation, business, family, and extended stays.
- **Property owners:** list properties, receive bookings, manage availability/pricing, communicate with guests, track revenue, collect reviews.
- **Property managers:** multi-property calendars, bookings, guests, cleaners, maintenance, revenue reports.
- **Verified agents:** list authorized properties, manage inquiries, book for clients, track commissions.
- **Corporate customers:** business accounts for employees, executives, consultants, project teams, visitors.

## Customer journey

Discover -> Verify -> Compare -> Book -> Pay -> Confirm -> Prepare -> Check-in -> Stay -> Support -> Check-out -> Review

## Key features (PRD v2.0)

- **Search & discovery:** by city, area, estate, landmark, airport, property name/type. Filters for property type, bedrooms, price, amenities, purpose.
- **Trust system:**
  - Guest verification (phone/email + optional government ID) and Guest Trust Profile
  - IDEALTIME VERIFIED (host/property/location inspected, photos/amenities/pricing checked, last-inspected date)
  - IDEALTIME READY (cleaned, checked, prepared for check-in)
  - Property Reality Check (exterior, rooms, power, internet, security, condition) and Property Trust Profile
- **Comparison, calendar, pricing:** side-by-side compare; availability calendar (Available / Pending / Unavailable, auto-block on booking); transparent total before pay with separate refundable caution fee. Example: 5 nights + cleaning + service + caution = total payable.
- **Booking options:** Instant Booking, Request to Book, Pay Now, Deposit Booking, Pay on Arrival, Corporate Billing. Book for someone else. Formal confirmation with reference, receipts, My Bookings (Upcoming / Current / Completed / Cancelled), Stay Timeline.
- **Stay support:** WhatsApp integration, customer-property messaging, help centre / live chat / WhatsApp / phone, incident reporting, Stay Protection, structured check-in/check-out, reviews.
- **Owner/manager tools:** dashboard (today/upcoming, revenue, occupancy, requests, reviews), property CRUD with photos/video, amenities, house rules, prices, availability, multi-property management, cleaning tasks (assign -> photos -> approve -> READY), maintenance tracking, revenue dashboard (gross, fees, net, payouts, occupancy, ABV).
- **Agent & corporate:** agent dashboard (clients, bookings, commissions), Idealtime Business accounts, corporate booking, monthly billing.
- **Extensions:** airport transfers, car rental, drivers, cleaning, cook/chef, events, holiday packages, relocation, stay bundles (e.g. Complete Lagos Stay), airport-to-apartment journey, promotions, personalized recommendations.

## Repo contents

- `README.md` — this overview
- `app doc/IDEALTIME BOOKINGS APP.md` — full PRD v2.0
- `.gitignore` — standard ignores

## Status

v0.1.0 initial version: docs-only / greenfield. No app stack locked yet.

## Next steps

1. MVP phase breakdown from the 50-section PRD
2. Data model + API design (users, properties, availability, bookings, payments, reviews, cleaning/maintenance)
3. Scaffold app (e.g. Next.js + Supabase/Postgres + Paystack/Flutterwave) and CI deploy
