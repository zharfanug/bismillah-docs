# Tech Stack Recommendation
## Wedding Announcement Website

**Status:** Decided
**Last updated:** 2026-08-11 (§7: illustrated asset pack moved from `web/public/assets/` to top-level `assets/wedding-illust/` — see CONTENT_DESIGN.md §31)

This doc covers architecture/infra: framework choices, backend/DB design, and the Docker Compose shape. For what's actually on the site (sections, feature behavior), see [CONTENT_DESIGN.md](./CONTENT_DESIGN.md). For the overall product goals, see [PRD.md](./PRD.md).

---

## 1. Frontend

**Decided: Next.js (App Router) + Tailwind CSS**

Since the backend is now a separate Django service rather than Next.js Route Handlers, Next.js here is used purely as the frontend — it calls the Django API instead of hosting its own API routes.

## 2. Backend

**Decided: Django + Django REST Framework (DRF)**

- Runs as its own container/service, separate from the Next.js frontend.
- `POST /api/comments/`, `GET /api/comments/` — submit / list guest wishes. Comments are inserted and shown **live immediately** — no approval queue. (No RSVP endpoint — the site is a public announcement, not an open invitation; see PRD §3, CONTENT_DESIGN.md §0.)
- `GET /api/health/` — public, unauthenticated, returns `{"status": "ok"}`. Exists purely so the backend's liveness can be checked directly (e.g. via a separate tunnel hostname pointed straight at the backend container), independent of the Next.js proxy — useful for isolating which layer broke when something's down.
- Serializer-level validation (DRF serializers) on every endpoint that accepts public input.
- **Denylist filter on comments:** a serializer `validate_message` check (word-boundary aware, case-insensitive, basic leetspeak normalization) rejects a submission containing a disallowed word/phrase with a 400 response and inline error — not silent censoring, not silent posting. List stored as a plain Python list/JSON file in the repo to start. (Product-level behavior/scope: CONTENT_DESIGN.md §3.)
- Basic anti-spam: a honeypot field on the form + DRF throttling classes (`AnonRateThrottle`) for simple per-IP rate limiting on the public write endpoint — no extra service (e.g., Redis) needed at this scale; DRF's default throttle backend uses the Django cache, which can just be local-memory here.
- **Django Admin, free bonus:** Django ships a built-in admin site. Registering the `Comment` model there gives the couple a real, auth-protected web page to browse/delete spam or inappropriate comments — no custom admin UI needs to be built, and it's a proper login-gated page (better fit for this than a DB GUI, since it doesn't require DB network access to use).

**Cross-origin note:** Next.js and Django are two separate origins in dev (different ports) and potentially in prod too. Simplest approach: configure Next.js `rewrites()` in `next.config.js` to proxy `/api/*` requests server-side to the Django service. From the browser's point of view everything is same-origin, so no CORS configuration or cookie/session cross-site headaches. (`django-cors-headers` is a fallback if you'd rather call Django directly from the browser instead.)

## 3. Database

**Decided: MariaDB**

- Official `mariadb:11` image in Docker Compose.
- Data persisted via a named Docker volume, so comments survive container restarts and rebuilds.
- Django connects via `django.db.backends.mysql` (MariaDB is wire-compatible) using the `mysqlclient` driver.

**ORM: Django ORM (built-in)**

- One model to start: `Comment` (name, message, created_at).
- Migrations via Django's own `makemigrations` / `migrate` — run as part of the backend container's start step.
- No extra ORM library needed — this is one of the advantages of Django over a Node backend: ORM, admin, migrations, and validation are all built in rather than assembled from separate packages (Prisma, Zod, a hand-rolled admin page).

## 4. Supporting libraries

| Concern | Recommendation |
|---|---|
| Frontend styling | **Tailwind CSS** (confirmed) |
| Frontend forms | **React Hook Form** for form state; validation errors surfaced from DRF's serializer error responses |
| Frontend animations | `motion` (see §7) for elaborate bits (entrance/idle animation); plain CSS transitions for simple hover/fade effects |
| Frontend icons | **lucide-react** |
| Fonts | Self-hosted via `next/font/google` — Playfair Display (headings), Great Vibes (couple names), Cormorant Garamond (body), Geist Mono (countdown digits only). See §7. |
| Image handling | Next.js built-in `<Image />` |
| Countdown | Small dependency-free custom component |
| Gallery/Lightbox | **yet-another-react-lightbox** |
| Backend framework | **Django + Django REST Framework** |
| Backend DB driver | **mysqlclient** (MariaDB via Django's MySQL backend) |
| Backend server | **gunicorn** behind the container, for anything beyond `runserver` in dev |

## 5. Docker Compose shape

**Built and verified** (both modes actually run end-to-end, not just written). Three services, one mode switch:

- **`db`** — `mariadb:11`, named volume (`db_data`) for data, healthcheck (`healthcheck.sh --connect --innodb_initialized`), env vars for root password/db name/user via `.env`. Identical in dev and prod.
- **`backend`** — Django + DRF. Single image for both modes; `backend/entrypoint.sh` runs `manage.py migrate` on every start, then branches on `$APP_ENV`:
  - `dev`: `manage.py runserver 0.0.0.0:8000`.
  - `prod`: `gunicorn config.wsgi:application --workers 3 --no-control-socket` (the `--no-control-socket` flag matters — gunicorn 26's control-socket feature defaults to a path at the filesystem root, `/.gunicorn/`, which a non-root container user can't write to).
  - `collectstatic` + `whitenoise` (added to `MIDDLEWARE`) so Django Admin's CSS/JS are actually served in prod — `collectstatic` runs at **image build time** (as root, via a `RUN` step in the Dockerfile), not at container start, since the runtime user (`1000:1000`) can't write into the root-owned `/app` the way collectstatic needs to on first run.
- **`web`** — Next.js frontend. `web/Dockerfile` is multi-stage with a `dev` target and a `prod` target (built from a shared `builder` stage that runs `npm run build`); `docker-compose.yml` picks the target via `target: ${APP_ENV:-dev}`, so the build target always matches the mode.
  - Dev target: source bind-mounted (`docker-compose.dev.yml`), `npm run dev`, hot reload.
  - Prod target: no bind mount, runs the compiled `.next` output via `npm run start` (`next start`) — no HMR websocket, no dev-only overhead. This directly avoids the `wss://.../_next/webpack-hmr` connection errors that show up when a Cloudflare Tunnel (or any reverse proxy) points at the dev server instead of a real build.
  - Proxies `/api/*` to the `backend` service via Next.js rewrites (see cross-origin note above).
- **File layout:** `docker-compose.yml` (base — prod-safe by default: no bind mounts, `DJANGO_DEBUG` defaults to `False`) + `docker-compose.dev.yml` (adds bind mounts + a `web_node_modules` named volume so the container's own installed packages aren't shadowed by the host bind mount). `.env` / `.env.example` hold `APP_ENV` (`dev`/`prod`), DB credentials, Django secret key, `DJANGO_ALLOWED_HOSTS`, and `NEXT_PUBLIC_SITE_URL` (used for OG image URLs).
- **`stack.sh`** — the actual run/restart interface. Reads `APP_ENV` from `.env`, picks the right `-f` compose file combination, derives `DJANGO_DEBUG` from `APP_ENV` automatically (script-controlled, not user-set — avoids the real risk of `DEBUG=True` accidentally shipping to something publicly exposed). Subcommands: `up` (default), `restart` (rebuild + recreate, picks up code/.env changes), `down`, `logs [service]`.
- Superuser creation is **not** automated (no default admin/password baked in) — still a one-time manual `docker compose exec backend python manage.py createsuperuser`, deliberately, since auto-creating a known-credential superuser is a real risk for anything that might end up publicly reachable.

## 6. Hosting & scope note

Hosting is self-hosted via this same Docker Compose stack, exposed publicly through a **Cloudflare Tunnel** the user sets up and owns outside this project (see PRD §3, §6).

**Implication for scope:** since Cloudflare Tunnel handles public ingress/TLS, this project does not need an Nginx/reverse-proxy service or certificate handling — Docker Compose only needs `web`, `backend`, `db`, wired up correctly for **local development**. Ports are published to `localhost` directly (e.g., `3000` for Next.js, `8000` for Django) for local testing; how they're fronted in production (tunnel config, hostnames) is out of scope for this repo.

**Real gotcha hit during actual deployment (worth remembering):** it's tempting to give the tunnel its own hostname for the backend (e.g. `bismillah-api.yourdomain.com` → `localhost:8000`) and then point `BACKEND_INTERNAL_URL` at that public hostname instead of `http://backend:8000`. Don't — Next's rewrite proxy runs *inside* the `web` container and already reaches `backend` directly over the private Docker network; routing it back out through the public tunnel instead is unnecessary **and breaks it**, because the Host header Django then sees won't match `DJANGO_ALLOWED_HOSTS` (whatever value ends up on that request needs to be allow-listed, and it's easy to allow-list the wrong hostname while debugging). Symptom looks exactly like a Cloudflare/tunnel routing problem — a 400 on every `/api/*` request, page itself loads fine — but the actual cause is entirely inside `.env`. A separate public hostname pointed straight at the backend is still genuinely useful (e.g. for hitting `/api/health/` or `/admin/` directly, bypassing the Next proxy for diagnosis) — just don't wire the app's own `BACKEND_INTERNAL_URL` through it.

Nothing left open that blocks scaffolding. Build plan with step-by-step tests: [TASK.md](./TASK.md).

## 7. Illustrated assets + animation

Real asset pack landed in `assets/wedding-illust/` (per-section layered PNGs, repo-root-level — moved out of `web/public/` in Phase 8 since it was never referenced by app code and was needlessly bloating the `web` Docker build context, see CONTENT_DESIGN.md §12) — mapping decisions and which assets go where live in CONTENT_DESIGN.md §12. This section covers the technical side.

- **Animation library: `motion`** (not `framer-motion` — checked both against npm before deciding: same maintainer, same version (13.0.0), `framer-motion` is now just a compatibility alias for the current `motion` package). Used for entrance/idle animation only (Cover birds/banner, Couple Profile slide-in, Note From Us/Share fade-in, closing-doodle heart-pulse) — not for anything functional, so if it were ever removed the site would still work, just without the motion.
  - Must respect `prefers-reduced-motion` — verify the actual reduced-motion API against the installed version's own docs before wiring it up (this project's running pattern all session: Next.js 16, Django 6, gunicorn 26 all had surprises past what training data assumed; no reason `motion` 13 would be different).
- **Image delivery: `next/image`, not raw `<img>`.** The placeholder SVGs (Step 4.x) were served as plain `<img>` deliberately — they were small, hand-drawn, vector, optimization wouldn't have done much. These new assets are large source PNGs (hundreds of KB to a few MB each), so `next/image`'s responsive `srcset` generation and lazy-loading actually matter here for the Lighthouse Performance score to hold at ≥90 (TASK.md Step 6.7 tests this explicitly rather than assuming). Needs `sharp` available in the `web` image for Next's production image optimizer — confirm it's present (Next installs it automatically as an optional dependency in most setups, but verify rather than assume, same reasoning as above).
- **Two-tier asset pipeline — `assets/wedding-illust/` (source, repo root) → `web/public/img/` (served).** `assets/wedding-illust/` is the original delivered pack — kept as-is, untouched, **never referenced directly from app code**, and deliberately outside `web/` so it's not part of the Docker build context. Anything actually used gets resized/cleaned with ImageMagick (`magick`/`convert`, confirmed installed) into `web/public/img/`, mirroring the same subfolder structure for traceability (e.g. `assets/wedding-illust/cover/background_cover.png` → `web/public/img/cover/background_cover.png`) — that `public/img/` copy is what components actually reference, and `next/image` optimizes further on top of it at request time.
  - Why resize manually instead of just pointing `next/image` at the originals: several source files are far larger than anything the site needs — decorative accents like `lace banner.png` (1109×2467, 3.7MB) or `mainbouquet.png` (1845×2940) are meant to render as small corner/heading elements, not full-bleed images, so shipping them unresized wastes decode time and bandwidth even before `next/image`'s own optimizer touches them.
  - Sizing convention (longest edge, `-resize 'NNNx NNN>'` so it only ever shrinks, never upscales, plus `-strip` to drop metadata):
    - Full-bleed background layers (`Background_*`, `background_*`): max **1080px**.
    - Illustration/frame/decorative accent layers (arches, frames, doodles, component florals): max **800px**.
    - Portraits (bride/groom): max **900px** on the tall edge.
  - Keep PNG (need alpha transparency on nearly everything here) rather than converting to WebP by hand — `next/image` already negotiates WebP/AVIF automatically at serve time from a PNG source, so hand-converting format would be redundant work.
- **Source files not used directly:** `.psd` files in each asset folder are design sources, not shipped — only the exported `.png` layers get resized into `web/public/img/` and referenced from code. `vector assets.eps` in `components/` likewise isn't directly usable in a browser; skip it unless something needs re-exporting from it later.
- **Retiring the placeholders:** `public/couple/*.svg` and `public/gallery/*.svg` (the hand-drawn stickman/doodle SVGs from Step 4.x) get removed once real assets are wired in for those sections (TASK.md Step 6.3) — grep for references before deleting, don't assume. `public/gift/qr.svg` stays; no real QR asset exists yet.
- **Full background coverage:** every section (not just the ones with a dedicated asset folder) has a full-bleed background from this pack — folders without their own composition (Countdown, Love Story, Gallery, Wishes, Gift) reuse another folder's `Background_*` layer rather than shipping with a flat color. Full assignment: CONTENT_DESIGN.md §12.
- **Contrast, made systematic:** once every section has a busy illustrated background, plain text directly on the artwork risks failing contrast depending on which part of the image sits behind it. Standard treatment applied everywhere: wrap text content in `rounded-2xl bg-white/75 backdrop-blur-sm` — a translucent panel, not a solid block, so the illustration still reads through. Verified with Lighthouse's `color-contrast` audit (0 failing elements) rather than assumed from the visual design alone.
- **Fonts, upgraded from the scaffold default:** the original TECH_STACK.md decision ("self-hosted via `next/font`") was implemented with the scaffold's placeholder Geist Sans/Mono; now actually wedding-appropriate — **Playfair Display** for headings (`h1`/`h2`, applied globally via CSS rather than a class on every heading), **Great Vibes** (script) specifically for the couple's names via a `.font-script` utility class, **Cormorant Garamond** as the default body font. Geist Mono kept only for the countdown digits (tabular numbers read cleanly in a monospace face, deliberate contrast against the serif surroundings). All three new fonts are variable Google Fonts loaded through `next/font/google` — same self-hosting mechanism as before, confirmed by an actual successful build (font names aren't statically validated by TypeScript; a typo'd Google Font name only surfaces as a build/fetch failure).
