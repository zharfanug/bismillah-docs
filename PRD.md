# Product Requirements Document (PRD)
## Wedding Announcement Website

**Status:** Draft
**Last updated:** 2026-07-28

This is the top-level product doc: what we're building and why. Detailed docs live alongside it:

- **[TECH_STACK.md](./TECH_STACK.md)** — infra/architecture design: frontend/backend/DB choices, Docker Compose shape, hosting approach.
- **[CONTENT_DESIGN.md](./CONTENT_DESIGN.md)** — web/content design: site sections, feature specs (comments, denylist, gift info, etc.), content-facing non-functional requirements.
- **[PAGE_DESIGN.md](./PAGE_DESIGN.md)** — page-by-page text wireframes, one per section, for a quick visual preview of what's on each page.
- **[TASK.md](./TASK.md)** — the step-by-step build plan, run one step at a time with a test/verification at each step.

---

## 1. Overview

The ceremony is a small, official/civil registration held at a limited-space venue — not an open event. People the couple wants physically present are invited separately, outside this site. So this is a **wedding announcement website**, not an invitation: a link shared publicly (WhatsApp, social media, etc.) so anyone — not just the small in-person guest list — can learn the couple is married, see photos, and leave a well-wish. Most of the site is static content, plus a real backend + database (see TECH_STACK.md) to support guest comments directly.

## 2. Goals

- Publicly announce the marriage: who and when — no location is disclosed (see CONTENT_DESIGN.md §5 for why: a note from the couple stands in for venue details).
- Feel personal and memorable — reflect the couple's story and style.
- Give the public an easy way to send well-wishes without needing to attend anything.
- Load fast and look great on mobile (most visitors will open the link on their phone from a chat app).
- Be easy to self-host and update (single Docker Compose command to run locally).

## 3. Non-Goals / Out of Scope

- **No RSVP / attendance collection.** This site's audience is the general public, not the attendee list — actual guests are invited to the ceremony separately, through other means, outside this website.
- No guest authentication/login (comments are submitted with just a name, no account system).
- No custom-built admin dashboard UI — moderation happens via **Django's built-in admin site** (auth-protected) rather than a page built from scratch.
- No payment processing (digital gift/envelope is just displayed account info / QR code, well-wisher transfers manually).
- No reverse-proxy/TLS setup in this codebase — public exposure is via Cloudflare Tunnel, set up and owned by the user outside this project. This project's job is to run correctly on `localhost`.

## 4. Target Users

| User | Needs |
|---|---|
| Wedding couple (site owner) | Easy to customize with their own content, cheap/simple to host, looks modern |
| Public visitors (family, friends, coworkers — not necessarily the in-person attendee list) | Learn the couple got married, see photos, easy way to send a well-wish |

## 5. Success Criteria

- A visitor can open the link on a phone and understand who got married and when, within seconds, and come away feeling the couple's gratitude even without knowing where the ceremony was held.
- Guest comments are saved reliably and survive a container restart.
- Couple can update wedding content (names/date/photos) by editing one data file and rebuilding, and can view/moderate comments without needing a custom admin page.
- `docker compose up` brings up frontend + backend + database locally with no manual extra steps beyond running migrations once.

## 6. Key Decisions Log

Quick reference — full detail lives in the linked docs.

| Decision | Answer | Detail |
|---|---|---|
| Site purpose | Public announcement, not an open invitation — actual attendees invited separately, outside the site | This doc §1, CONTENT_DESIGN.md §0 |
| Frontend | Next.js (App Router) + Tailwind CSS | TECH_STACK.md §1 |
| Backend | Django + Django REST Framework, own service | TECH_STACK.md §2 |
| Database | MariaDB | TECH_STACK.md §3 |
| RSVP | Not included — out of scope (see Non-Goals) | This doc §3 |
| Venue/location | Not disclosed — replaced by "A Note From Us" | CONTENT_DESIGN.md §5 |
| Add-to-calendar | Not included — no attendance to plan for | CONTENT_DESIGN.md §0 |
| Personalized greeting | `?to=Name` URL placeholder, hand-edited per recipient, no guest-list lookup | CONTENT_DESIGN.md §4 |
| Comment moderation | Live immediately + denylist check on submit | CONTENT_DESIGN.md §6 |
| Denylist scope | Indonesian + English, small curated list in-repo | CONTENT_DESIGN.md §6 |
| Section removability | Every section (+ music) toggled via one config flag, no code changes needed | CONTENT_DESIGN.md §2, PAGE_DESIGN.md |
| Content architecture | Two files: `wedding.ts` (personal/event data) and `labels.ts` (bilingual ID/EN UI copy, deploy-time `locale` switch) | CONTENT_DESIGN.md §2 |
| Hosting | Self-hosted via Docker Compose + Cloudflare Tunnel (user-managed) | TECH_STACK.md §6 |
| Build scope | Local-first — get it fully working on `localhost` before worrying about the tunnel/domain | This doc, §3 |

Still open (non-blocking, default and revisit later): custom domain/name (affects OG meta), content source format for love-story copy, exact wording/title for the "Note From Us" section (drafts in CONTENT_DESIGN.md §5).
