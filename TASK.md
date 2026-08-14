# Build Plan
## Wedding Announcement Website

**Status:** All phases (0-8) complete. Site live via Cloudflare Tunnel in prod mode.
**Last updated:** 2026-08-10 (Phase 8: split Cover arch into edge-pinned halves, bride-first Couple Profile order, fixed reopen-to-middle-page bug, scroll-snap page-change animation, `docs/OPERATIONS.md` + fixed `/admin/` 400, Wishes name field pre-filled from `?to=` without locking it. Phase 7: real names, split Couple Profile, `name/` + `ig_v2/` assets, Instagram link, dark-mode contrast bug fixed and verified, scroll lock verified with real wheel events, Cover loading gate, centered desktop frame. Final Lighthouse: Performance 99, Accessibility 100, Best Practices 96, SEO 100, 0 color-contrast failures)

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

### Phase 6 — Illustrated assets + animation

The user dropped a full illustrated watercolor asset pack in `web/public/assets/` (Indonesian-Muslim wedding styling — hijab/peci portraits, lily + blue-orchid florals, birds, foliage frames), organized per-section as layered PNGs (background / illustration / frame). This phase replaces the hand-drawn placeholder SVGs (Step 4.x) with these real assets and adds entrance animation. Full asset → section mapping and the "location option" repurposing decision are documented in CONTENT_DESIGN.md §12; library choice and image-optimization approach in TECH_STACK.md §7. Don't start a step until the previous one's test passes, same as every other phase.

- [x] **Step 6.1: Image/animation pipeline setup**
  - Deliverable:
    - Install `motion` (the current package name — `framer-motion` is now just an alias, same version); confirm `sharp` is available for Next's production image optimizer.
    - A small `MotionSection` or per-component pattern for scroll-triggered entrance animation that respects `prefers-reduced-motion` (motion's `useReducedMotion` hook — verify actual API name against the installed version before using it, per this project's running pattern of checking bundled docs rather than assuming).
    - The `public/assets/` → `public/img/` resize pipeline (TECH_STACK.md §7): ImageMagick (`magick`, now installed) commands per the sizing convention there (1080px backgrounds / 800px accents / 900px portraits, `-strip`), mirroring subfolder structure. Doesn't need to process every asset up front — resize each file as it's actually used in Steps 6.2–6.6.
  - Test: `npm run build` succeeds with `motion` installed; a throwaway component using it renders with no console errors in both normal and `prefers-reduced-motion: reduce` emulation (headless browser check, both modes); one test resize (e.g. `assets/cover/background_cover.png` → `img/cover/background_cover.png`) runs and produces a file at/under the target dimension.

- [x] **Step 6.2: Cover + post-tap hero ("opening")**
  - Deliverable: resize `cover/background_cover.png`, `landscape_cover.png`, `birds&banner_cover.png` and `opening/background_opening.png`, `arch_opening.png` into `public/img/cover/` and `public/img/opening/`; wire the `img/` copies into the pre-tap gate overlay and the revealed in-flow hero respectively — these are currently identical content shown twice (CONTENT_DESIGN.md §12 explains why splitting them across the two asset folders is the right call). Position layers per CONTENT_DESIGN.md §12's extracted `.psd` percentages (landscape bottom-anchored at 37–100% height; banner near-full-bleed 2–98%/9–95%; opening's arch spans the top 47%) rather than eyeballing it. Birds/banner get a subtle idle animation; tap-to-open transition softens (fade/scale instead of instant swap).
  - Test: both `img/` asset sets load (200 on each image path, served from `public/img/`, not `public/assets/`); tap-to-open still reveals the page and still calls `markOpened()` (music unlock unaffected — rerun that specific smoke-test check); animation doesn't block interaction (button clickable immediately, not gated behind animation finishing).

- [x] **Step 6.3: Couple Profile ("bride & groom")**
  - Deliverable: resize `bride & groom/bride_bride&groom.png`, `groom_bride&groom.png`, `Background_bride&groom.png`, `frame_bride&groom.png` into `public/img/bride-groom/`; swap `couple/bride.svg` + `couple/groom.svg` placeholders for these. Position per CONTENT_DESIGN.md §12: bride left half (0–62% width), groom right half (55–98% width), both bottom-anchored — they overlap 55–62%, so get the z-order right (check which one the source has in front before picking a stacking order). Staggered slide/fade-in (bride and groom enter from opposite sides).
  - Test: `img/` images load; old placeholder SVG files removed from `public/couple/` once nothing references them (grep the codebase first — don't delete blind); content file paths updated to `img/` locations.

- [x] **Step 6.4: "A Note From Us"**
  - Deliverable: resize `location option 1/frame_location1.png` into `public/img/note/`, use as a decorative border around the note text — **not** used to show any venue/address (CONTENT_DESIGN.md §0/§5 still stands; this is picking one folder's artwork for its looks, nothing more). Per CONTENT_DESIGN.md §12, the frame is a near-full-bleed 4-corner border (~2% margin) with a separate horizontal band (`frame2_location1`, 43–57% height) — that band is where the source design puts its text, worth matching for where the note copy sits inside the frame. `location option 2/` stays unused in `assets/`, not copied into `img/`.
  - Test: same denylist-adjacent check as Step 4.1 — confirm no address/map/calendar text anywhere on the page after this change (still nothing there, just decoration added); frame image loads from `img/`; fade-in animation present.

- [x] **Step 6.5: Love Story, Gallery, Wishes, Gift accents** *(also recreated `public/gallery/*.svg` — found missing/404 during this step's testing, unrelated to this asset pack; see CONTENT_DESIGN.md §12)*
  - Deliverable: resize the needed `components/` florals (lily, orchid, amaranthus, mainbouquet) and `lace banner.png` into `public/img/components/`, use as small corner accents / heading backdrops across these four sections — decoration only, no functional changes.
  - Test: rerun the relevant slice of the Step 5.2 smoke test (lightbox open/close, comment submit + denylist rejection, gift copy button) to confirm nothing broke functionally; new `img/` images load; no layout shift covering interactive elements (buttons/inputs still clickable — verify via the same click-driven smoke test, not just visual inspection).

- [x] **Step 6.6: Share / Closing** *(adapted as a sized hero block rather than the source's full-height percentages — landscape-shaped composition doesn't map directly onto a tall mobile section; see CONTENT_DESIGN.md §12)*
  - Deliverable: resize `closing/doodle_closing.png`, `Background_closing.png`, `frame_closing.png` into `public/img/closing/`; doodle becomes the section's hero image. Per CONTENT_DESIGN.md §12: `frame_closing` sits at the left edge (0–28% width, full height), `doodle_closing` layered in front of it, wider (0–45% width) — both left-anchored, not centered. Small heart-pulse animation. `outro/` was unused at this point in the plan — later reused for Love Story's background (Step 6.8).
  - Test: `img/` image loads; Copy Link / Share-to-WhatsApp buttons still present and functional (rerun that smoke-test check).

- [x] **Step 6.7: Full regression pass**
  - Deliverable: nothing new — this step is purely verification after all the asset/animation changes land.
  - Test: rerun the full Step 5.2 E2E smoke test (all 19 checks) against the changed UI; rerun the Step 5.1 Lighthouse audit — Performance must stay ≥90 despite the much heavier images (this is the real risk of this phase; if it drops, go back and tune `next/image` `sizes`/priority/lazy-loading before calling this phase done, don't just accept a regression); confirm `prefers-reduced-motion: reduce` genuinely disables/minimizes the new animations (not just that the library supports it in theory); grep the codebase to confirm nothing under `public/assets/` is referenced directly — every image path in code should point at `public/img/`.
  - **Result:** 19/19 E2E checks passed against dev mode, then again against the real **prod build** (`next start`, not dev server). Lighthouse (prod build): **Performance 99** (down 1 from Phase 5's 100 — LCP 0.96/Speed Index 0.99, negligible given the much richer visuals now shipped, no tuning needed), **Accessibility 100, SEO 100, Best Practices 96** (same known missing-audio-placeholder flag as before, not new). Reduced-motion checked holistically by scrolling through every section under `prefers-reduced-motion: reduce` emulation — every heading/content block resolved to `opacity: 1`, no hydration errors. `grep` confirmed zero references to `public/assets/` anywhere in `web/src` — every image path used in code points at `public/img/`.

- [x] **Step 6.8: Fonts, contrast fix, full background coverage, Countdown → `location option 2`, revised section defaults**
  - Deliverable, at the user's direction:
    - **Fonts:** replaced the scaffold's Geist Sans/Mono with Playfair Display (headings), Great Vibes script (couple's names, `.font-script`), Cormorant Garamond (body). Geist Mono kept only for countdown digits.
    - **Countdown → `location option 2/`:** previously-unused folder now backs Countdown (`background_location2.png` + `silhouette_location2.png` + `arch_location2.png`), per explicit instruction.
    - **Full background coverage audit:** found Countdown, A Note From Us (frame only, no background), Love Story, Gallery, Wishes, and Gift all had no background image. Fixed all six — Note From Us gained `Background_location1.png`; Love Story reused `outro/Background_outro.png` (previously unused); Gallery reused `cover/background_cover.png`; Wishes reused `opening/background_opening.png`; Gift reused `bride & groom/Background_bride&groom.png`. Every section now has one — see CONTENT_DESIGN.md §12.
    - **Contrast fix:** since every section now sits on a busy illustrated background, wrapped all text content across every section in a `rounded-2xl bg-white/75 backdrop-blur-sm` translucent panel (form inputs additionally get an explicit opaque `bg-white` — browser default styling isn't guaranteed against a themed background).
    - **Section defaults revised** to match the user's own edit to PAGE_DESIGN.md: Love Story, Gallery, Digital Gift now `enabled: false` in `content/wedding.ts` (were `true`).
  - Test:
    - Type-check + lint clean across all changed files.
    - Full browser check (12 assertions): toggled-off sections (Love Story/Gallery/Gift) confirmed absent from the DOM; toggled-on sections confirmed present; `h1`/`h2`/`body` computed `font-family` confirmed to actually resolve to Great Vibes/Playfair Display/Cormorant Garamond (not just imported); Countdown's 3 `location option 2` images confirmed loaded (`naturalWidth > 0`); zero unexpected console/network errors.
    - **Lighthouse `color-contrast` audit: 0 failing elements, Accessibility 100** — contrast fix verified empirically, not just visually. Performance 97 (still well above 90, given even more background images now loaded than Step 6.7's baseline).
    - Reran the full Step 5.2 smoke test — 14/16 automated checks passed; the 2 "failures" (Gallery thumbnail, Gift copy button) are the smoke-test script itself being stale about the new defaults, not real regressions — those sections are supposed to be absent now.

### Phase 7 — Real content, contrast bug fix, layout/UX cleanup

User feedback after seeing the Phase 6 build live: leftover dark-mode CSS caused genuine white-on-white text in some visitor conditions, the page scrolled behind the still-closed Cover overlay, the "Tap to Open" button could appear before its artwork had loaded, Couple Profile was cramped on phone widths, and two new asset folders (`name/`, `ig_v2/`) plus real names/an Instagram handle needed wiring in. Full rationale for every change: CONTENT_DESIGN.md §13–§20.

- [x] **Step 7.1: Force light-only theme, fix white-on-white contrast**
  - Deliverable: removed the `@media (prefers-color-scheme: dark)` block in `globals.css` (root cause — see CONTENT_DESIGN.md §17) and the one remaining `dark:` Tailwind variant pair in `MusicPlayer.tsx`.
  - **Result:** verified via Puppeteer with `prefers-color-scheme: dark` emulated — `body`'s computed color stayed `rgb(23,23,23)` (dark) against the light `rgb(243,239,232)` background, confirmed with real screenshots at Countdown and Share showing no white-on-white anywhere. Lighthouse `color-contrast`: 0 failing elements, Accessibility 100.

- [x] **Step 7.2: Hide scrollbar + lock scroll until opened**
  - Deliverable: global CSS hiding the scrollbar; a `ScrollLock` client component using `OpenedContext` to set `document.documentElement`'s overflow to `hidden` while `!opened`.
  - **Result:** initial test with `window.scrollBy()` (a JS API call) misleadingly still moved `scrollY` — Chromium allows some programmatic scroll calls to bypass `overflow:hidden`. Retested with a real simulated wheel gesture (`page.mouse.wheel`), which is what the feature actually needs to block: `scrollY` stayed `0` pre-open, moved to `800` after tapping "Buka" — scroll lock confirmed working for actual user input.

- [x] **Step 7.3: Cover asset-loading gate before Tap to Open**
  - Deliverable: `onLoad`/`onError` tracked across the pre-tap overlay's 4 images; "Buka" only renders once all have settled; a spinner + "Memuat..." shows until then.
  - **Result:** confirmed via screenshot — button renders correctly once assets settle in normal conditions; `onError` fallback added so a broken/missing image can't permanently strand a visitor on the loading state.

- [x] **Step 7.4: Update wedding.ts with real names, nicknames, Instagram**
  - Deliverable: `couple.bride`/`couple.groom` restructured to `{ nickname, fullName, parents, photo, nameImage }` (CONTENT_DESIGN.md §13) — bride "Ara" / "Haura Dhiya Amaninida", groom "Zharfan" / "Muhammad Zharfan Nugroho", parents left as placeholder; new `content.instagram` and `content.coupleSignatureImage`; gift `accountName` and note `signature` updated too (found while sweeping for stale references to the old placeholder names).
  - **Result:** `grep` confirmed zero remaining references to the old `.name` field. Page `<title>` verified as "Pernikahan: Zharfan & Ara" in a live browser check.

- [x] **Step 7.5: Wire name signature PNGs into Cover + Couple Profile + Share**
  - Deliverable: resized `assets/name/*` into `img/name/` (trimmed transparent margins first); wired via new `content.couple.*.nameImage`/`content.coupleSignatureImage` fields (kept in `wedding.ts`, not hardcoded in components, per the project's content-architecture convention) into Cover (both states), Couple Profile (solo per person), and Share.
  - **Result:** confirmed via screenshots at both mobile and desktop widths — signature art renders correctly in all three places; `alt` text carries the actual nickname(s) (standard accessible pattern for an image that functions as a heading, not decoration).

- [x] **Step 7.6: Split Couple Profile into separate groom/bride sections**
  - Deliverable: `CoupleProfile.tsx` reworked into two full-viewport sections via a shared `PersonSection` component, groom then bride — portrait, solo name signature, full name, parents line each.
  - **Result:** confirmed via desktop screenshots — each person now has an uncramped dedicated screen; reduced-motion/entrance-animation behavior inherited unchanged from `MotionSection`.

- [x] **Step 7.7: Map ig_v2 assets + Instagram live link into Share/Closing**
  - Deliverable: resized `assets/ig_v2/ig_v2.png` (the flattened composite) into `img/closing/ig-v2.png`; replaced the old doodle+frame hero block in `Share.tsx` with this card; added a "Saksikan Live di Instagram" link to `instagram.com/hauradhiyaa`.
  - **Result:** confirmed via screenshot and a direct `href` check in a live browser session — link resolves to `https://instagram.com/hauradhiyaa`; Copy Link/Share-to-WhatsApp buttons re-verified working alongside it.

- [x] **Step 7.8: Desktop layout cleanup (centered mobile-card frame)**
  - Deliverable: wrapped `<main>` in `mx-auto max-w-md bg-white`; gave `--background` a distinct off-white so the column reads as a card on wide viewports. **Revised mid-implementation:** the originally planned `transform: translateZ(0)` containing-block trick was dropped after reasoning through its side effects — it would re-target `position: fixed` descendants (Cover's overlay, MusicPlayer's toggle) to be positioned relative to the wrapper's own (very tall, whole-page) box instead of the real viewport, breaking MusicPlayer's persist-across-scroll behavior and stretching Cover's overlay to the page's full multi-screen height instead of one viewport. Used `fixed` + `left-1/2 -translate-x-1/2` + `w-full max-w-md` on Cover's overlay and MusicPlayer's toggle instead — centers/width-caps them to match the column while staying genuinely viewport-fixed.
  - **Result:** confirmed via 1440px-wide screenshots, pre-tap and opened — page reads as a centered card against the off-white backdrop; MusicPlayer's toggle stays pinned bottom-right *of the card*, confirmed still visible/functional after scrolling (not just at load).

- [x] **Step 7.9: Update docs**
  - Deliverable: this Phase 7 entry, CONTENT_DESIGN.md §13–§20, and the corresponding PAGE_DESIGN.md page updates — written ahead of implementation, then §14 and §20 revised afterward to match the actual (simpler/safer) implementation decisions made during Steps 7.5 and 7.8.
  - **Result:** docs cross-checked against shipped code during this pass — accurate.

- [x] **Step 7.10: Full regression pass**
  - Deliverable: nothing new — verification only.
  - **Result:** `tsc --noEmit` clean, `eslint` clean, `next build` (prod) succeeds. Puppeteer E2E: scroll lock (real wheel events) verified pre/post-open, Cover loading gate, Instagram link href, page `<title>`, comment submit-and-appear all passed; only console/network issue was the pre-existing documented `audio/track.mp3` 404 (no real track supplied). Lighthouse (desktop preset): **Performance 99, Accessibility 100, Best Practices 96, SEO 100**, `color-contrast` 0 failures. Dark-OS-preference emulation re-verified the contrast fix directly (§7.1). Screenshots reviewed at mobile (390px) and desktop (1440px), both pre-tap and opened states, plus both Couple Profile sections and Share. Stack was switched to `APP_ENV=dev` for iteration and switched back to `APP_ENV=prod` afterward, matching the live Cloudflare Tunnel deployment as found.

### Phase 8 — Cover arch responsiveness follow-up

User feedback after Phase 7 shipped: the floral arch on Cover's revealed hero (`arch_opening.png`) is a single image with both clusters baked in, so `object-contain` shrinks and centers the whole thing on wide viewports instead of keeping each cluster pinned to its edge. Full rationale: CONTENT_DESIGN.md §21.

- [x] **Step 8.1: Split arch_opening into edge-pinned left/right halves**
  - Deliverable: cropped `assets/opening/arch_opening.png` (720×608) at the midpoint into `img/opening/arch_opening_left.png`/`arch_opening_right.png` (trimmed to content bounds); `Cover.tsx` now renders them as two independent `absolute` elements (`left-0`/`right-0`, `w-1/2`, `h-[47%]`, `object-contain` + `object-left-top`/`object-right-top`) instead of one full-width image; deleted the now-unreferenced combined `img/opening/arch_opening.png`.
  - Test: `grep` confirmed no remaining references to the combined file; visual check at mobile (390px) confirmed near-identical rendering to before the split; visual check at desktop (1440px, inside the Phase 7 centered card) confirmed both clusters now stay pinned to the card's left/right edges with a growing gap between them, instead of one small centered composition.

- [x] **Step 8.2: Swap Couple Profile order to bride first**
  - Deliverable: `CoupleProfile.tsx` now renders bride's `PersonSection` before groom's (was groom-first, matching the original combined layout's order) — CONTENT_DESIGN.md §22.
  - Test: visual check confirms bride's section (portrait, "Ara" signature, full name, parents) now scrolls into view first, groom's second; no other section's ordering (Cover, Share signatures) affected — those weren't part of this request.

- [x] **Step 8.3: Fix reopen-to-middle-page bug + scroll-snap page-change animation**
  - Deliverable: `ScrollLock` opts out of browser scroll restoration (`history.scrollRestoration = "manual"`) and force-scrolls to `(0,0)` on mount; `Cover.tsx`'s `handleOpen` does the same as a second guarantee (CONTENT_DESIGN.md §23). Added `scroll-snap-type: y mandatory` (html) + `scroll-snap-align: start` (every top-level section) + `scroll-behavior: smooth`, disabled under `prefers-reduced-motion: reduce` (CONTENT_DESIGN.md §24).
  - **Result:** scroll-reset bug reproduced and confirmed fixed — scrolled to `scrollY=2532`, reloaded, confirmed `scrollY=0` immediately (not restored by the browser) and tapping open landed on Cover, not the previously-scrolled section. Scroll-snap CSS confirmed correctly applied via `getComputedStyle` (`scroll-snap-type: y mandatory` on `html`, `scroll-snap-align: start` on all 7 top-level sections) — **but** headless-Chromium synthetic wheel/touch dispatch (tried both, including raw CDP touch-swipe) did not reliably reproduce exact snap-settling in this test environment, landing short of the true section boundary rather than exactly on it. Assessed as a synthetic-input/headless-automation limitation rather than a CSS defect (standard, widely-supported feature; correctly declared and scoped) — flagged as worth a real-device check rather than claimed as fully empirically verified. `tsc --noEmit`, `eslint`, and `next build` (prod) all clean; broader regression (comment submit-and-appear, no new console/network errors beyond the known audio placeholder 404) rerun and passed.

- [x] **Step 8.4: Operations doc + fixed `/admin/` "Bad Request"**
  - Deliverable: new `docs/OPERATIONS.md` — database access/backup/moderation/reset procedures for the live site. While verifying it, found `/admin/` returned a bare 400 for `Host: localhost:8000` (`DJANGO_ALLOWED_HOSTS` only listed the tunnel domain and `backend`, not `localhost`/`127.0.0.1`) — fixed in `.env` and `.env.example`.
  - **Result:** every command in `OPERATIONS.md` tested against the live database (backup dump, `SELECT` query, comment counts) before being written down as fact, not assumed. `/admin/` confirmed fixed (400 → 302 to login) after recreating just the `backend` container (`docker compose up -d --force-recreate backend`) — `web`/`db` and the public `/api/*` proxy path untouched/reverified healthy. Also surfaced: all 20 comments in the live database turned out to be development test artifacts (E2E test runs from this project's build-out), not real guest messages — flagged to the user, not deleted without confirmation (destructive/irreversible without a backup).

- [x] **Step 8.5: Pre-fill Wishes name from `?to=`, keep it editable**
  - Deliverable: `page.tsx` now passes the `?to=` `greetingName` prop to `Wishes` too (previously Cover-only); `Wishes.tsx` seeds its existing controlled name `<input>`'s initial state with it (`useState(greetingName ?? "")`) — a plain pre-fill, not `readOnly`/`disabled` (CONTENT_DESIGN.md §25).
  - Test: `?to=Jane` → Wishes' name field shows "Jane" pre-filled on load; field remains fully editable (typing/clearing works normally); submitting without editing posts "Jane" as the name; no `to` param → field starts empty, unchanged from before.

- [x] **Step 8.6: Swap signature text order to "Ara & Zharfan"**
  - Deliverable: the `name/ara-zharfan.png` image itself always read bride-first, but `layout.tsx`'s site title, `Cover.tsx`'s `coupleNames` (both states' `alt` text), and `Share.tsx`'s `alt` text were built groom-first — swapped all three to `bride.nickname` + `groom.nickname` (CONTENT_DESIGN.md §26).
  - Test: `curl` confirmed `<title>Pernikahan: Ara &amp; Zharfan</title>`; browser check confirmed both Cover and Share `<img>` elements' `alt` text reads "Ara & Zharfan"; `tsc`/`eslint`/`next build` clean.

- [x] **Step 8.7: Rebuild Closing section from `ig_v2` layers + Instagram icon**
  - Deliverable: resized `assets/ig_v2/ig_v2-1.png`/`-2.png`/`-3.png` separately into `img/closing/ig-v2-{background,frame,portrait}.png`; `Share.tsx` rebuilt to use background+frame as full-bleed layers behind the text panel and the portrait as its own image below the panel, replacing the previous flattened-composite "card" (CONTENT_DESIGN.md §27); added `src/components/icons/InstagramIcon.tsx` (inline SVG, no new dependency) to the "Watch Live on Instagram" button. Kept Copy Link/Share-to-WhatsApp buttons (not mentioned as removed). Deleted the now-unreferenced old `img/closing/{background,doodle,frame,ig-v2}.png`.
  - Test: real browser check at mobile (390px) and desktop (1440px, centered card) — background/frame/portrait all load and align correctly (independently `object-cover`-fit layers sharing the same source aspect ratio stay visually registered with each other); Instagram icon renders inline in the button; `instagram.com/hauradhiyaa` href confirmed; no new console/network errors beyond the known audio placeholder 404; `tsc --noEmit`, `eslint`, `next build` all clean.

- [x] **Step 8.8: Uniform music icon + Closing footer portrait + remove share buttons**
  - Deliverable: `src/components/icons/MusicIcon.tsx` (inline SVG, one note-glyph path shared by both states, `muted` prop just adds a slash line) replacing the mismatched 🎵/🔇 emoji in `MusicPlayer.tsx`. In `Share.tsx`: removed Copy Link/Share-to-WhatsApp entirely (state, handlers, and the now-unused `labels.share.copyLink`/`shareWhatsApp` — component no longer needs `"use client"` at all as a result); repositioned the `ig_v2-3` portrait from an in-flow element to `absolute inset-x-0 bottom-0` (section's bottom padding removed so it sits flush at the true edge), enlarged `w-56` → `w-72` (CONTENT_DESIGN.md §28).
  - Test: real browser check confirmed the music toggle renders the new SVG icon (not emoji) in both muted/unmuted states; Share section screenshot confirmed the portrait sits flush against the section's bottom edge with no gap beneath it, larger than before, and only the Watch Live on Instagram button remains (no Copy Link/WhatsApp); `tsc --noEmit`, `eslint`, `next build` all clean.

- [x] **Step 8.9: Fix real "stuck loading" cause + progress bar + 30s auto-refresh**
  - Deliverable: investigated the user's live "stuck loading" report by reproducing against the actual public tunnel URL (Puppeteer, cache disabled, full network timing) — found all 4 gating images loaded successfully in under a second, ruling out the Step 7.3 loading-gate logic. Checked the running container directly (`docker exec ... ps aux`) and found `npm run dev` (Turbopack + HMR) running instead of the prod build, despite `.env` saying `APP_ENV=prod` — the previous session's closing `./stack.sh restart` had been interrupted between turns and a follow-up health check that only asserted `curl` returned 200 didn't catch it (dev mode also serves 200s). Fixed by re-running `./stack.sh restart` and this time verifying the actual **process**, not just HTTP status. Added a real progress bar (`loadedCount`/`PRETAP_IMAGE_COUNT`, replacing the indeterminate spinner) and a one-time 30s `setTimeout` → `window.location.reload()` fallback (ref-mirrored `assetsReady` check to avoid a stale closure) to `Cover.tsx` as defensive UX for any future stuck-loading cause (CONTENT_DESIGN.md §29).
  - Test: `docker exec bismillah-web-1 ps aux` confirmed `next-server`/`npm run start` (not `next dev`) after the fix; re-ran the exact reproduction against the public tunnel URL — button now appears in ~1.1s (was stuck 10s+ before); progress bar visually confirmed filling as each image settles; `tsc --noEmit`, `eslint`, `next build` all clean.

- [x] **Step 8.10: Move arch halves to `components/`, reuse in Countdown, off-screen sway**
  - Deliverable: renamed/moved `img/opening/arch_opening_left.png`/`_right.png` → `img/components/arch_flower1_left.png`/`_right.png`; updated `Cover.tsx`'s `src` references. Retired `Countdown.tsx`'s own `img/countdown/arch.png`, replaced with the same two shared halves (deleted the now-unreferenced file, confirmed via `grep` first). Both sections: repositioned from edge-flush (`left-0`/`right-0`) to `left: "-5%"`/`right: "-5%"` (deliberately bleeding off-screen); added a slow rotational sway (`rotate` keyframes, 7s, `origin-top`, mirrored + 0.5s-delayed between the two halves, `useReducedMotion`-gated) — `Countdown.tsx` didn't import from `motion/react` before this, added it. CONTENT_DESIGN.md §30.
  - Test: real browser check confirmed no broken image paths (only the known audio-placeholder 404); screenshots of both Cover and Countdown confirmed the shared floral halves render correctly, bleeding past both edges; sampled the motion element's computed `transform` matrix once/second — confirmed continuously changing under normal conditions and frozen at `"none"` under `prefers-reduced-motion: reduce` emulation; `tsc --noEmit`, `eslint`, `next build` all clean.

- [x] **Step 8.11: `go-prod.sh` / `run-dev.sh` — self-verifying mode-switch scripts**
  - Deliverable: two thin wrappers around `./stack.sh restart` that set `APP_ENV` in `.env`, restart, wait for `web` to respond, then assert the actual running process (`npm run start` for `go-prod.sh`, `next dev` for `run-dev.sh`) via `docker compose exec web ps aux` — **exit non-zero if it doesn't match**, directly closing the gap from Step 8.9 (a curl-only health check that couldn't distinguish dev from prod). OPERATIONS.md §6 updated to point at these instead of raw `stack.sh restart`.
  - Test: ran both for real against the live stack — `./go-prod.sh` (from already-prod) reported "web is running 'npm run start' (prod)"; `./run-dev.sh` correctly switched and reported "web is running 'next dev'"; `./go-prod.sh` again correctly switched back and reverified prod. All three runs' HTTP checks (web/api/admin) passed. Left the stack in prod mode, self-confirmed by the script itself rather than a separate manual check.

- [x] **Step 8.12: Sync arch sway animation + z-index top layering**
  - Deliverable: Cover.tsx's arch halves had already been hand-tuned (outside this session's edits) to asymmetric keyframes — left `rotate: [0, -7, 0, 3, 0]`/7s, right `rotate: [0, 10, 0, -3, 0]`/10s/0.5s delay; synced Countdown.tsx's (still on the original symmetric `[0, ∓3, 0, ±3, 0]`/7s values) to match exactly. Added `z-20` to all four arch `motion.div`s (both halves, both sections) — above the `z-10` content panels, the top layer in either section (CONTENT_DESIGN.md §30).
  - Test: `./run-dev.sh` (self-verified dev mode); confirmed via `getComputedStyle` that all four arch elements report `zIndex: "20"`; sampled both sections' left-arch computed `transform` in parallel — identical values at each sample point, confirming Cover and Countdown are now running the exact same animation in lockstep; screenshots showed no visual regression (florals still confined to the outer edges, no collision with panel content despite now being the top layer); `tsc --noEmit`, `eslint`, `next build` all clean; `./go-prod.sh` switched back to prod, self-verified.

- [x] **Step 8.13: Move `assets/` out of `web/public/`, add favicon**
  - Deliverable:
    - Moved `web/public/assets/` (336MB, never referenced by app code — CONTENT_DESIGN.md §12/§31) to top-level `assets/wedding-illust/` (repo root, sibling of `web/`) — was needlessly part of the `web` Docker build context with no `.dockerignore` excluding it. Updated every doc reference to the new path (`CONTENT_DESIGN.md`, `TECH_STACK.md`); left TASK.md's own historical Phase 6/7 entries alone (accurate at the time they were written, journal entries aren't retroactively rewritten).
    - Copied `assets/favicon/*` (a standard realfavicongenerator-style bundle — `favicon.ico`, `favicon-16x16.png`, `favicon-32x32.png`, `apple-touch-icon.png`, `android-chrome-192x192.png`, `android-chrome-512x512.png`, `site.webmanifest`) into `web/public/`, leaving `assets/favicon/` completely untouched as the source ("og material," per instruction). Removed `src/app/favicon.ico` (Next's default placeholder, using the App Router auto-convention) since it would conflict with a same-named `public/favicon.ico`; wired up explicit `metadata.icons`/`metadata.manifest` in `layout.tsx` pointing at the `public/` files instead. Filled in `site.webmanifest`'s previously-empty `name`/`short_name` with "Ara & Zharfan" (CONTENT_DESIGN.md §31 notes this is a static exception to the usual content.ts-driven pattern — a PWA manifest, not visible page content).
  - Test: `./run-dev.sh` (self-verified dev mode, confirming the move didn't break the build). `curl` confirmed `/favicon.ico`, `/favicon-32x32.png`, `/apple-touch-icon.png`, `/android-chrome-192x192.png`, `/site.webmanifest` all `200`; `<head>` output showed correct `<link rel="icon">`/`<link rel="apple-touch-icon">`/`<link rel="manifest">` tags; no build conflict from the removed `app/favicon.ico`. Full page load + open flow re-verified with no new failed requests. `tsc --noEmit`, `eslint`, `next build` all clean. `./go-prod.sh` switched back to prod, self-verified.
