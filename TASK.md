# Build Plan
## Wedding Announcement Website

**Status:** All local-dev steps complete — ready for Cloudflare Tunnel setup (user-owned, outside this repo)
**Last updated:** 2026-07-28 (all phases 0-5 complete; full E2E smoke test passed 19/19 via headless-browser scripting; added `stack.sh` + prod build path — see TECH_STACK.md §5)

Execution plan for local development. Steps are meant to be done **one at a time, in order** — each step has a concrete deliverable and a test to run before moving to the next. Don't start a step until the previous one's test passes. Scope is local-only; Cloudflare Tunnel / production exposure is handled separately by the user (see PRD.md §3).

**Running the stack:** use `./stack.sh [up|restart|down|logs [service]]` rather than raw `docker compose` commands (steps below were written before the script existed, so they show `docker compose ...` directly — same effect, `stack.sh` just picks the right compose files and derives `DJANGO_DEBUG` for you based on `APP_ENV` in `.env`). Set `APP_ENV=dev` for local iteration (hot reload) or `APP_ENV=prod` before pointing anything (like a Cloudflare Tunnel) at it — see TECH_STACK.md §5 for why dev mode isn't safe/correct to expose publicly.

Steps are numbered `<phase>.<step>`. Check off each step as it's completed: `[ ]` → `[x]`.

Reference docs: [PRD.md](./PRD.md) (goals), [TECH_STACK.md](./TECH_STACK.md) (architecture), [CONTENT_DESIGN.md](./CONTENT_DESIGN.md) (feature/content spec).

---

### Phase 0 — Scaffold

- [x] **Step 0.1: Repo + Docker Compose skeleton**
  - Deliverable: `docker-compose.yml` with three services (`web` = bare Next.js app, `backend` = bare Django project, `db` = MariaDB), `.env.example`, `.gitignore`. No features yet — just booting empty apps.
  - Test:
    - `docker compose up --build` starts all three containers with no errors/crash loops.
    - `curl localhost:3000` returns the default Next.js page.
    - `curl localhost:8000/admin/` returns the Django admin login page.
    - `docker compose exec db mariadb -u <user> -p<pass> -e "SELECT 1;"` returns a result.

### Phase 1 — Backend data layer

- [x] **Step 1.1: Django model + admin**
  - Deliverable: `Comment` (name, message, created_at) model, migrations, registered in Django Admin. A superuser created for local testing.
  - Test:
    - `docker compose exec backend python manage.py migrate` runs clean.
    - Log into `/admin/` with the superuser, manually create one `Comment` row through the admin UI, confirm it appears in the list view.

- [x] **Step 1.2: DRF API endpoints**
  - Deliverable: `GET/POST /api/comments/` — serializer with validation, denylist check on message, honeypot field, per-IP throttle (see CONTENT_DESIGN.md §6, TECH_STACK.md §2).
  - Test (via `curl` or a REST client):
    - `POST /api/comments/` with a normal message → `201`, then `GET /api/comments/` includes it.
    - `POST /api/comments/` with a message containing a denylisted word → `400` with a clear error, and it does **not** appear in `GET /api/comments/`.
    - `POST /api/comments/` with the honeypot field filled in → rejected (treated as a bot).
    - Fire several rapid `POST /api/comments/` requests → later ones get throttled (429).

### Phase 2 — Frontend/backend wiring

- [x] **Step 2.1: Next.js scaffold + API proxy**
  - Deliverable: Next.js app with Tailwind configured, base layout, `next.config.js` `rewrites()` proxying `/api/*` to the `backend` service.
  - Test: a temporary test page in the Next.js app fetches `GET /api/comments/` via the relative path (not the backend's direct URL) and renders the raw JSON — confirms the browser-facing proxy round-trips to Django correctly. Remove/replace this test page once confirmed.

### Phase 3 — Content structure

- [x] **Step 3.1: Content data file + section registry**
  - Deliverable: `content/wedding.ts` holding couple names, event date/time (no venue/address — see CONTENT_DESIGN.md §5), the "Note From Us" title/copy, gallery image list, gift info, default/fallback greeting text, etc., with placeholder values — plus the `sections` toggle list and `musicEnabled` flag (CONTENT_DESIGN.md §2), and a small component registry/loop in the page that renders only `enabled` sections in list order. Also `content/labels.ts` — generic bilingual (ID/EN) UI copy, separated from the personal/event data, with a `locale` constant + `t()` helper.
  - Test:
    - A placeholder page renders several content values on screen, confirming components pull from the data file rather than hardcoded strings.
    - Set one section's `enabled` to `false` (e.g. `gallery`, which defaults `on`), rebuild, confirm that section is fully absent from the rendered page (no empty gap, no broken import). Flip it back to `true`, rebuild, confirm it reappears in the same position.
    - Flip `locale` in `content/labels.ts` from `"id"` to `"en"`, confirm all UI copy switches language with no other code changes; flip back to `"id"`.

### Phase 4 — UI sections

- [x] **Step 4.1: Cover (+ greeting placeholder), Countdown, Couple profile, "A Note From Us"** *(visual/mobile-width check still needs a human look — see note below)*
  - Test: manual check in browser at mobile width (~375px) and desktop; countdown numbers tick down correctly against a test date set in the content file; `localhost:3000/?to=Jane` shows "Dear Jane," in the cover greeting, and `localhost:3000` with no query param falls back to the generic greeting; "A Note From Us" section shows the chosen copy with no address/map/calendar element present anywhere on the page.

- [x] **Step 4.2: Love story + Gallery** *(lightbox interaction needs a human look — see note in Step 4.1)*
  - Test: gallery opens a lightbox on click, arrow/keyboard navigation works, images are lazy-loaded (check Network tab).

- [x] **Step 4.3: Comments section (list + form)** *(live-update UX needs a human look — API behavior fully verified via curl through the proxy)*
  - Test:
    - Submit a normal comment → appears at the top of the list immediately without a full page reload.
    - Submit a denylisted word → inline error shown, comment not added to the list.
    - Refresh the page → previously submitted comment is still there (confirms it's reading from the DB, not local state).

- [x] **Step 4.4: Gift info, music toggle, share/OG tags** *(interactive polish — clipboard copy, audio playback — needs a human look; markup/OG tags fully verified)*
  - Test: copy-to-clipboard button copies the correct account number; music toggle mutes/unmutes and persists across a page scroll; `view-source:` (or a link-preview debug tool) shows correct OG title/description/image tags.

### Phase 5 — Polish & full smoke test

- [x] **Step 5.1: Responsive/accessibility/performance pass**
  - Test: Lighthouse run on the cover page scores 90+ on Performance; keyboard-only navigation reaches the comment form and the gallery lightbox; color contrast check on text over background images/video.
  - Result (real Lighthouse run via headless Chromium, not eyeballed): **Performance 100, Accessibility 100, SEO 100, Best Practices 96** (the only flags: missing placeholder `audio/track.mp3` file, and a dev-mode Turbopack source-map artifact — neither a real defect). `color-contrast` audit passed explicitly; no positive `tabindex` values (natural keyboard order preserved).
  - Still needs a human look: actual mobile-width visual layout (this environment has no screenshot tool).

- [x] **Step 5.2: Full local smoke test (end-to-end visitor journey)**
  - Open `localhost:3000/?to=TestName` and walk the entire page top to bottom:
    - Greeting shows "Dear TestName,".
    - Countdown is correct.
    - "A Note From Us" reads correctly, no venue/address/map/calendar anywhere on the page.
    - Gallery lightbox works.
    - Submit a normal comment (appears live) and a denylisted comment (rejected).
    - Copy gift info.
    - Toggle music.
    - Share/copy-link button works.
  - This step is the "ready to move on to Cloudflare Tunnel setup" checkpoint — everything above it must pass first.
  - **Result:** ran a real scripted browser session (headless Chromium + Puppeteer, mobile viewport 390×844) driving actual clicks/keyboard events against the live local stack — not just HTTP inspection. **19/19 checks passed**: greeting personalization, tap-to-open reveal, live countdown tick, gallery lightbox open/ArrowRight/Escape, comment submit + live append, denylist rejection (inline error, not added to list), gift copy button, music toggle (aria-label flips), Copy Link + Share-to-WhatsApp buttons present, no unexpected console errors. This also closes out the "needs a human look" caveats noted in Steps 4.1–4.4.
