# Content & Web Design
## Wedding Announcement Website

**Status:** Decided
**Last updated:** 2026-08-15 (Phase 10: new Main Announcement section inserted right after Cover — Bismillah + formal announcement text in a `letter_frame.png` card, Noto Naskh Arabic added for the Arabic line, bride/groom names + date colored per instruction; layout later replaced hand-tuned margins with three `top`/`bottom`-percentage-positioned zones (both now `13.20%`, unified) computed from the frame art's actual pixel geometry, "&" reduced one size (§33); Cover's own revealed in-flow hero removed entirely — "Tap to Open" now lands directly on Main Announcement instead of a second, near-duplicate greeting screen (§35); page order revised, A Note From Us moved later; language toggle (id/en) made guest-facing instead of dev-only — floating button, flag-emoji notation, raised above Cover's overlay so it's usable pre-tap, both floating buttons now 50%-opaque backgrounds (§34); Countdown's date line removed (the date now lives on Main Announcement only, §33) and replaced with a QS. Ar-Rum: 21 quote — meaning only, no Arabic script (§36). Phase 9: Cover's preload gate generalized from an image-only list to `PRELOAD_ASSETS`, now including the background music track alongside the Couple Profile images it already warmed — audio resolves on `canplaythrough`/`error` instead of `onload`/`onerror` (§19); Cover's wedding date removed from both the pre-tap overlay and the revealed hero (§35 superseded this — see above); wheel scrolling converted to a page-turn gesture — one section per 35%-of-viewport accumulated wheel input, keyboard/touch still use the CSS snap (§24); Guest Wishes card capped to one viewport with the comment list as an internal scroller, plus a bottom fade as the scrollability cue (§32). Phase 8: asset pack move + favicon (§31); shared arch halves + rotational sway (§30); real "stuck loading" root cause + progress bar + auto-refresh (§29); Closing rebuilt from `ig_v2`, uniform music icon, copy/share buttons removed (§27–§28); Cover arch split (§21), bride-first Couple Profile (§22), opening-to-mid-page fix (§23), scroll-snap page-change animation (§24), Wishes name pre-fill (§25), bride-first signature (§26). Phase 7: real names, split Couple Profile, `name/` signature assets, `ig_v2/` → Closing, Instagram live link, light-only theme, scroll lock + hidden scrollbar, Cover asset gate, centered desktop frame — see §12–§14)

This doc covers what's actually on the site: sections, page flow, and the behavior of each feature. For architecture/infra (frameworks, Docker Compose, hosting), see [TECH_STACK.md](./TECH_STACK.md). For high-level product goals, see [PRD.md](./PRD.md). For a visual per-page preview, see [PAGE_DESIGN.md](./PAGE_DESIGN.md).

---

## 0. Framing: announcement, not invitation

The ceremony itself is held at an official/civil building, small space, limited guest count. People the couple actually wants physically present are invited **separately, through other means** (WhatsApp, phone, etc.) — not through this site. This website's job is to **publicly announce** the marriage and let anyone who sees the link (family, friends, coworkers who aren't part of the small in-person guest list) share in the moment and send well-wishes, **not** to solicit attendance or collect an RSVP headcount. This shapes several decisions below:

- **No RSVP feature.** Since the site's audience isn't the attendee list, there's nothing to RSVP to on the site itself.
- **No venue/location disclosed at all.** Rather than a "here's how to find us" details block, the section where that would normally go is instead **a short note from the couple** — see §5.
- **No add-to-calendar.** That feature exists to help people attend an event; since the site's audience isn't attending, it doesn't apply here.
- Copy tone throughout should read as an announcement/share ("We're getting married" / "With joy, we share our wedding day with you") rather than a summons ("You are cordially invited...").

## 1. Site sections

The page is a single long-scroll experience. Every section is independently switched on/off via a config flag (§2) — nothing below is permanently locked in, so treat "default" as "what ships on unless you turn it off," not "mandatory."

Order and defaults revised in Phase 10 (2026-08-15, §33) — Main Announcement inserted right after Cover, A Note From Us moved later (after Gallery instead of right after Couple Profile).

| # | Section | Default | Notes |
|---|---|---|---|
| 1 | Cover / Landing screen | **On** | Couple's names, "tap to open," personalized greeting placeholder (§4). No wedding date on the Cover (removed 2026-08-15) — the date shows on Main Announcement (row 2) only, per §33/§36. Pre-tap overlay only — the revealed in-flow hero was removed (§35); tapping "Buka" now reveals Main Announcement directly. |
| 2 | Main announcement | **On** | Formal Bismillah + announcement text in a `letter_frame` card — see feature spec §33. |
| 3 | Countdown timer | **On** | Live countdown to the event date/time. |
| 4 | Couple profile | **On** | Two full-screen sections, bride then groom (revised — was one combined overlapping layout; split for readability, see §13). |
| 5 | Love story / timeline | **Off** | Optional short story or milestones — off by default (revised), toggle on any time. |
| 6 | Gallery | **Off** | Photo (and optionally video) gallery with lightbox — off by default (revised). |
| 7 | A Note From Us | **On** | Replaces the typical venue/location block — see feature spec §5. |
| 8 | Guest wishes/comments | **On** | See feature spec §6 — the site's main interactive element now that RSVP is out of scope. |
| 9 | Digital gift info | **Off** | Bank/e-wallet info, copy-to-clipboard, optional QR — off by default (revised). |
| 10 | Share / closing | **On** | Copy link / share-to-WhatsApp, OG meta tags. |

Background music is a floating control present across all sections, not a section itself — it has its own on/off flag too (§2).

## 2. Section toggle mechanism

Since removability was an explicit requirement: each section (and the music player) is controlled by a single boolean in the content data file, not scattered through component code.

**Two content files, not one — split by what kind of thing the value is:**
- **`content/wedding.ts`** — personal/event data: couple names, event date, "A Note From Us" body, gallery paths, gift/bank info, section toggles. Anything specific to *this* couple's wedding.
- **`content/labels.ts`** — generic UI copy: headings, button text, form placeholders, system messages (e.g. "Tap to Open," "Send," "Counting down to our day"). Nothing here is couple-specific — it's the same on every deployment of this template.
  - Every label is stored as `{ en: "...", id: "..." }`, and a single exported `locale` constant (currently `"id"`) picks which one renders, via a small `t(dict)` helper. Flipping `locale` to `"en"` re-renders the whole site in English with no other code changes — verified by actually flipping it and checking output before shipping.
  - A real in-browser language switcher (so visitors pick the language, not just the deploy-time constant) is still nice-to-have/not built — see §3.

**Translatability isn't limited to `labels.ts`.** Within `wedding.ts` itself, the same `{ en, id }` shape (exported as the `Bilingual` type from `labels.ts`, resolved with the same `t()`) is used for every field that's actual prose — even though it's personal/event content, not generic UI copy:
- Bilingual: event `displayDate` (month names differ — "August 28, 2026" vs "28 Agustus 2026"), `greeting.fallback`, couple `parents` lines, "A Note From Us" `title`/`body`, Love Story `title`/`body` per milestone, gallery `alt` text, Share `closingLine`.
- **Not** bilingual (plain values, because there's nothing to translate): couple/gift proper nouns (names, bank name), the account number, the ISO date/time, all file paths (photos, gallery images, QR, audio), and the note's `signature` (just names).
- Both directions verified by actually flipping `locale` and checking rendered output — not assumed from the code alone.

- The content file (`content/wedding.ts` — see §11.1) exports an ordered list, e.g.:
  ```ts
  export const sections = [
    { id: "cover", enabled: true },
    { id: "countdown", enabled: true },
    { id: "coupleProfile", enabled: true },
    { id: "noteFromUs", enabled: true },
    { id: "loveStory", enabled: true },
    { id: "gallery", enabled: true },
    { id: "wishes", enabled: true },
    { id: "gift", enabled: true },
    { id: "share", enabled: true },
  ] as const;

  export const musicEnabled = true;
  ```
- The page component maps over this list once, looking up each `id` in a small component registry and skipping anything with `enabled: false`. The list's order also controls the section order — reordering the array reorders the page, no JSX changes needed.
- **To remove a page:** flip its `enabled` to `false` and rebuild (`docker compose up --build` picks it up, per the existing "edit one data file and rebuild" maintainability requirement). **To bring it back:** flip it to `true`. No component code, routing, or layout files need touching either way.
- This applies uniformly — including sections that started as "off by default" (Love Story) and ones that started "on" (everything else). There's no special-cased "core" section that can't be turned off.
- Turning off the Comments section also hides its form/list from the page, but does **not** delete existing comment data in the database — toggling is purely a display decision, not a data one. (Turning it back on later shows the previously submitted comments again.)

## 3. Nice-to-have (not in first build)

- Simple animations/transitions between sections.
- Light/dark or theme variant selection.
- **In-browser language switcher UI.** The ID/EN label infrastructure itself is done (§2) — a visitor-facing toggle to switch `locale` at runtime (rather than at build time) is the remaining nice-to-have.

## 4. Feature spec: Personalized greeting placeholder

- **Decided:** the cover greeting shows a name from a URL query param, e.g. `?to=Jane` → "Dear Jane,". This is **not** a guest-list/lookup feature — it's a plain placeholder in the URL.
- **No guest list, no backend involvement, no validation.** The couple crafts a personalized link per recipient by hand-editing the `to` value before sharing it (e.g. copy the base link, change `?to=` for each person/family they send it to). Any string works; nothing is checked against a database.
- **Default when the param is absent:** falls back to a generic greeting (e.g. "Dear Family & Friends,") rather than showing a blank or broken placeholder — this covers the general public link that isn't personalized to anyone.
- Purely client-side (read the query param, render it) — no API involved.
- **Also used to pre-fill the Guest Wishes name field** (§25) — same `to` value, one query param serving both.

## 5. Feature spec: "A Note From Us"

Replaces the typical venue/location block entirely — this section carries the emotional weight of explaining a small, private ceremony instead.

- **Content:** a section title + a short paragraph, both editable in the content data file like everything else. Draft copy options (pick one, edit freely, or ask for more):
  1. *"Our celebration was intentionally small — just the two of us and our closest family. But we've felt so much love from every prayer, kind thought, and warm wish you've sent our way. Thank you for being part of this new chapter with us."*
  2. *"We chose to keep our wedding day small and quiet. Even so, your prayers and well-wishes have reached us and mean more than words can say. Thank you for celebrating this moment with us — even from afar."*
  3. *"Some celebrations are meant to be small, so the love within them can feel infinite. Though we couldn't share this day with everyone in person, every message and prayer you've sent has been a gift. Thank you for walking this new chapter with us."*
- **Suggested section title:** "A Note From Us" (alternatives: "With Gratitude", "Thank You For Being With Us — In Spirit").
- No location, address, map, or "you're invited" language anywhere in this section — that's the point of it.
- Presentation: plain centered text block, similar visual weight to the Love Story section — no map embed, no calendar button, no address chip.

## 6. Feature spec: Guest wishes/comments

- **Decided:** database-backed via the Django API, shown as a public list on the site (most recent first).
- Fields: guest name, message.
- **Displayed live immediately** — no approval queue. Instead, trust/safety is handled at submission time:
  - **Denylist check:** the message is checked against a list of disallowed words/phrases (profanity, slurs, spam terms) before saving. A match **rejects the submission** with an inline error shown to the guest — never silently posted, never silently auto-censored.
    - **Scope:** covers both **Indonesian and English**.
    - **Source:** a small hand-curated list checked into the repo (not a large external dependency) — easy for the couple to extend if something slips through.
    - Matching should be case-insensitive, word-boundary aware, with basic leetspeak normalization (e.g. `4` → `a`) to catch simple evasion.
  - **Anti-spam:** a honeypot field on the form + basic per-IP rate limiting on the submit endpoint.
- If something inappropriate still gets through, the couple can delete it via Django Admin (no separate "hide/report" UI needed).
- See §2 for how hiding this section (without deleting data) works.

## 7. Feature spec: Digital gift info

- Static content: bank account name/number and/or e-wallet details, shown with a copy-to-clipboard button.
- Optionally a QR code image (e.g., for e-wallet transfer) — just an image asset, no dynamic QR generation needed.
- No payment processing — purely informational, well-wisher sends manually outside the site.

## 8. Feature spec: Background music

- A single audio track, autoplay-on-first-interaction (to respect browser autoplay policies) with a visible mute/unmute toggle.
- Controlled by its own `musicEnabled` flag (§2) — if off, no player renders at all and no audio file loads.

## 9. Feature spec: Share / link preview

- Open Graph + Twitter Card meta tags (title, description, image) so the link looks good when pasted into a chat app.
- ~~A "copy link" and/or "share to WhatsApp" button~~ — built in Phase 4, **removed in Phase 8** (§28) in favor of the "Watch Live on Instagram" link as the section's one action; the couple shares the link directly themselves rather than visitors re-sharing it further.

## 10. Content-facing non-functional requirements

- **Performance:** optimized images, minimal JS shipped for static sections. Target Lighthouse performance score 90+.
- **Responsiveness:** mobile-first; must look correct on common phone widths (360–430px) as well as desktop.
- **SEO / Link previews:** proper `<title>`, meta description, and Open Graph/Twitter card tags.
- **Accessibility:** reasonable color contrast, alt text on images, keyboard-navigable gallery/lightbox and forms.
- **Maintainability:** personal/event data lives in one file (`content/wedding.ts`), generic bilingual UI copy in another (`content/labels.ts`) — both editable without touching component code; sections removable the same way (§2).

## 11. Open decisions (non-blocking)

1. ~~Content source format~~ — **Decided:** a typed data file, split in two by kind of content — `content/wedding.ts` (personal/event data + section toggles) and `content/labels.ts` (bilingual UI copy). No markdown parser needed for what's a small amount of text.
2. **Domain/name:** affects the Open Graph `url` field and share links. Not needed to build/run locally — can be filled in once the domain is known (user is handling hosting/domain via Cloudflare, per PRD §3).
3. **Note copy/title:** which of the three drafts in §5 (or an edited/new version), and which section title — defaults to draft 1 + "A Note From Us" if not specified before building.

## 12. Illustrated asset pack (`assets/wedding-illust/`)

The user supplied a full watercolor illustration set — Indonesian-Muslim wedding styling (hijab/peci portraits), lily + blue-orchid florals, birds, foliage frames, consistent dusty-blue/cream/sage/blush palette. Organized per-section as layered PNGs (`Background_*` / illustration / `frame_*`), plus `.psd` source files (not used directly — PNG exports only) and a shared `components/` folder of standalone decorative elements. This replaces the hand-drawn placeholder SVGs from Step 4.x (`public/couple/*.svg`, `public/gallery/*.svg`) — those were always documented as swap-out placeholders (CONTENT_DESIGN.md history, TASK.md Step 4.x), and now there's something real to swap in. Build plan: TASK.md Phase 6.

**Two folders, two roles:** `assets/wedding-illust/` (below) is the source pack — kept untouched, not referenced by app code. `web/public/img/` holds the resized/cleaned copies that components actually use, produced with ImageMagick per a defined sizing convention. See TECH_STACK.md §7 for the pipeline; this section is about *which* source asset goes where, not the file path it ends up served from.

**Revised (Phase 8, §31):** originally lived at `web/public/assets/` — moved to top-level `assets/wedding-illust/` (repo root, a sibling of `web/`, not inside it) since it was never referenced by app code anyway and, sitting inside `web/`, was being pulled into every `web` Docker build context/image for no reason (336MB, no `.dockerignore` excluding it). Every path below written as `assets/wedding-illust/...` reflects the current location; historical entries elsewhere in these docs (e.g. TASK.md's Phase 6/7 step log) still say `web/public/assets/` or `public/assets/` because that was accurate *at the time* — journal entries aren't rewritten after the fact, see TASK.md's own note on this.

**Mapping decision — asset folder → site section.** Every section now has a full-bleed background from this pack (revised — originally several sections had only a small accent, no background; audited and filled in the gaps so no section is left on a flat color). Folders without a dedicated composition reuse another folder's `Background_*`/`background_*` layer:

| Asset folder | Site section | How it's used |
|---|---|---|
| `cover/` | Cover, pre-tap overlay | `background_cover.png` (gradient) + `landscape_cover.png` (watercolor scene) + `birds&banner_cover.png` (names/date banner). Background also reused for **Gallery** (no dedicated folder). |
| `opening/` | ~~Cover, revealed in-flow hero~~ **Wishes only, as of Phase 10 (§35)** | `background_opening.png`, now reused for **Wishes** (no dedicated folder). Originally also Cover's revealed-hero background alongside `arch_opening.png` (superseded by the shared `components/arch_flower1_*` halves in Phase 8, §30) — that whole hero section was removed in Phase 10, so this folder's only live use today is Wishes. |
| `bride & groom/` | Couple Profile | `bride_bride&groom.png` / `groom_bride&groom.png` (full illustrated portraits, replacing the stickman SVGs) + `Background_bride&groom.png` + `frame_bride&groom.png`. Background also reused for **Gift** (no dedicated folder). |
| `location option 1/` | **A Note From Us** | `Background_location1.png` + `frame_location1.png` (floral corner frame) reused purely for its look. **This is a deliberate repurposing, not a reversal of §0/§5** — the folder is named "location" because the source template assumes an open invitation with a venue block; this project doesn't show one. Only the artwork is reused; no address/map/venue text is added anywhere. |
| `location option 2/` | **Countdown** (revised — previously unused) | `background_location2.png` + `silhouette_location2.png` (treeline overlay). Assigned here specifically at the user's direction. Its `arch_location2.png` (floral arch framing the countdown digits) was retired in Phase 8 (§30) in favor of the shared `components/arch_flower1_left.png`/`_right.png` halves (same ones Cover uses). |
| `closing/` | Share / Closing | `doodle_closing.png` (the couple peeking around a curtain with a heart — the section's hero illustration) + `Background_closing.png` + `frame_closing.png` |
| `outro/` | **Love Story** (revised — previously unused) | `Background_outro.png` reused as Love Story's background (no dedicated folder of its own). |
| `components/` | Love Story, Gallery, Wishes, Gift (accents); Cover + Countdown (`arch_flower1_left/right.png`, §30); Main Announcement (`letter_frame.png`, §33) | Standalone flowers (`lily.png`, `orchid.png`, `amaranthus.png`, `mainbouquet.png`), `birds.png`, `lace banner.png` (a reusable translucent panel — same shape as the cover banner but without birds, good as a heading backdrop), two silhouette assets, `arch_flower1_left.png`/`_right.png` — the split arch halves, moved here from `opening/` since they're shared between Cover and Countdown (Phase 8) — and (Phase 10) `letter_frame.png`, an ornate floral card used whole as Main Announcement's content container. Used as corner/heading decoration only (except `letter_frame`, which is the content's own background) — no other functional changes to these sections. |
| `couple/*.svg` (hand-drawn placeholders) | *(retired)* | Removed once `bride & groom/` assets are wired in (TASK.md Step 6.3) — grep for references first, don't delete blind. |
| `gallery/*.svg` (hand-drawn placeholders) | *(kept)* | The gallery's actual photo content is still couple-supplied (CONTENT_DESIGN.md doesn't change on that front) — component florals dress up the section chrome (TASK.md Step 6.5), but the gallery images themselves stay as whatever the couple provides. These 3 files had gone missing from disk at some point before Phase 6 started (found broken/404 while testing Step 6.5) — recreated from the original Step 4.x source since nothing in this asset pack replaces them. |
| `gift/qr.svg` (hand-drawn placeholder) | *(stays)* | No real asset exists for this — still a placeholder until the couple supplies an actual QR code. |

**Contrast:** now that every section sits on a busy illustrated background instead of a flat color, all text content is wrapped in a `bg-white/75 backdrop-blur-sm rounded-2xl` panel rather than sitting directly on the artwork. Verified empirically with Lighthouse's `color-contrast` audit (0 failing elements, Accessibility 100), not just eyeballed.

**Typography:** Playfair Display (headings, `h1`/`h2` globally), Great Vibes script (couple's names specifically, via a `.font-script` class), Cormorant Garamond (body text, replacing the scaffold's default Geist Sans) — all self-hosted via `next/font/google` (TECH_STACK.md decision, now actually implemented). Geist Mono kept only for the countdown digits.

**Layer positions (from source `.psd` files).** Each section's `.psd` encodes the designer's actual intended composition — where every layer sits on a shared canvas — which the individually exported PNGs lose once cropped to their own bounding box in isolation. Extracted with `identify -verbose "file.psd[N]"` (ImageMagick reads PSD layers directly, no Photoshop needed); each layer's "Page geometry" (`WxH+X+Y`) converts to an x/y range as a percentage of the canvas, so it holds regardless of final resolution — cross-checked against the actual exported PNG dimensions, same aspect ratio in every case. The `Background` layer (always full-canvas) is the base gradient/color, already covered by `Background_*.png` — omitted below since it needs no positioning decision. Use these ranges directly for Step 6.2–6.6 CSS positioning (absolute-positioned within a relative container sized to the section) instead of guessing layout from the flat previews.

- **Cover** (canvas 1913×3400, portrait):
  - `landscape_cover`: x 0–100%, y 37–100% — bottom-anchored, full width.
  - `birds&banner_cover`: x 2–98%, y 9–95% — near full-bleed, overlaps the landscape's upper portion.
- **Opening** (canvas 1913×3400):
  - `background_opening`: x 0–100%, y 0–100% — full canvas (the transparency in the PNG itself does the framing, not the layer bounds).
  - `arch_opening`: x 0–100%, y 0–47% — hangs from the top.
- **Bride & Groom** (canvas 1913×3400):
  - `frame_bride&groom`: x 13.5–86.5%, y 3–97% — full-height border, ~13.5% side margins. **z-order matters:** despite the name, this isn't a transparent border — it has an opaque cream fill (a photo-card backdrop), so it has to render *behind* both portraits, not in front, or it covers them. Confirmed by actually rendering it wrong first, then fixing (TASK.md Step 6.3).
  - `bride_bride&groom`: x 0–62%, y 41–100% — left half, bottom-anchored.
  - `groom_bride&groom`: x 55–98%, y 41–100% — right half, bottom-anchored. Overlaps bride at 55–62% (standing close together) — z-order matters here, check which renders on top visually before implementing.
- **A Note From Us** (`location option 1`, canvas 1440×3288):
  - `frame_location1`: x 2–98%, y 1–99% — near-full-bleed 4-corner frame.
  - `frame2_location1`: x 0–100%, y 43–57% — a separate horizontal band, likely where the note text is meant to sit.
- **Closing** (canvas 2160×2070):
  - `frame_closing`: x 0–28%, y 0–100% — left-edge foliage border.
  - `doodle_closing`: x 0–45%, y 0–100% — left ~45% width, full height, layered in front of the frame.
- **Location option 2** *(unused — reference only)* (canvas 1440×3288): `arch_location2` x 0–100%/y 2–54% (top half); `silhouette_location2` full-bleed (tree-canopy-top/foliage-bottom transparency).
- **Outro** *(unused — reference only)* (canvas 2160×1278 — the one landscape-orientation PSD in the set, unlike every other portrait one): `frame_outro` x 0–67%/y 0–100%.

**Animation:** entrance/idle animation using `motion` (see TECH_STACK.md §7) — birds/banner idle motion on Cover, staggered slide-in on Couple Profile, fade-in on Note From Us and Share, subtle heart-pulse on the closing doodle. Kept to *entrance and idle* motion, not constant/looping distraction — and must respect `prefers-reduced-motion` (TASK.md Step 6.1/6.7 test this explicitly, not just assume the library handles it).

## 13. Couple identity: nickname vs. full name

Revised (Phase 7): each of `couple.bride`/`couple.groom` in `content/wedding.ts` now has both a `nickname` (short, casual — "Ara", "Zharfan") and a `fullName` (formal — "Haura Dhiya Amaninida", "Muhammad Zharfan Nugroho"), not a single `name` field.

- **Nickname** is what's visually prominent — Cover and Share/Closing show it (as the hand-lettered `name/` signature image, §14, not plain text), and the page `<title>`/OG tags use it too (shorter, matches what's on-screen).
- **Full name** appears once, in the Couple Profile section, alongside the parents line — the one place on the site where formal identification matters.
- **Parents' names stay placeholder** (`Bpk. A & Ibu B` / `Bpk. C & Ibu D`) — real names weren't supplied, don't invent them.

## 14. `name/` signature assets — couple's names as hand-lettered art, not just a font

The user supplied a small set of transparent PNGs (`assets/wedding-illust/name/`) — a calligraphic signature reading "Ara & Zharfan", plus solo versions ("Ara" alone, "Zharfan" alone) and a standalone ampersand. These render as actual illustrated art (a hand-drawn signature look), which reads noticeably more polished than the `.font-script` (Great Vibes) text previously used for the couple's names — so they replace it everywhere the *couple's* names are the visual focus:

| File (`assets/name/`) | Resized to (`img/name/`) | Used in |
|---|---|---|
| `ara & zharfan.png` (combined) | `ara-zharfan.png` | Cover (pre-tap overlay + revealed hero), Share/Closing signature |
| `Untitled_Artwork-3.png` ("Ara" solo) | `ara.png` | Couple Profile — bride's solo section |
| `Untitled_Artwork-2.png` ("Zharfan" solo) | `zharfan.png` | Couple Profile — groom's solo section |
| `Untitled_Artwork-1.png` (ampersand only) | *(not used)* | Kept in source pack only — no section needs a bare "&" on its own. |

Each image is treated as content, not decoration — its `alt` text is the actual nickname(s) it spells out (e.g. `alt="Ara & Zharfan"`), the standard accessible pattern for an image that *is* a name rendered as art, so screen readers get the right name without needing a separate visually-hidden text duplicate. Couple Profile additionally shows the full legal name as plain visible text right below each solo signature (§15) — that's real, indexable text regardless of the image.

Sizing convention: treated as an illustration/accent layer per TECH_STACK.md §7 (max 800px longest edge), trimmed first (`-trim +repage`) to drop the large transparent margins in the 2048×2048 source canvas before resizing — otherwise the rendered image would be mostly empty space.

## 15. Couple Profile split into two solo sections (mobile readability)

Revised (Phase 7): the combined bride+groom layout (both portraits overlapping in one `min-h-screen` section, per the original `.psd` composition — §12 layer percentages above) was too cramped on phone widths — both illustrated portraits, the frame, and two name/parents blocks fighting for space in one viewport. Split into **two separate full-screen sections**, each with:
- The shared `bride & groom/Background_*` background (reused for both, unchanged from before).
- That person's portrait alone, larger and centered/bottom-anchored — no longer sharing width with the other person.
- Their solo `name/` signature image (§14) as the heading, with their full name (§13) as real text beneath it, then the placeholder parents line.

The paired `frame_bride&groom.png` (designed for the two-person composition, §12's layer percentages) isn't reused here — scaling a frame built for two side-by-side portraits down to one would distort the source design, so each solo section instead uses the same `bg-white/75 backdrop-blur-sm` text-panel treatment as every other section, for visual consistency across the page rather than a mismatched one-off frame.

## 16. `ig_v2/` — Instagram-story layers for Closing, + Instagram live link

The user supplied a second illustration set, `assets/wedding-illust/ig_v2/` — a portrait-oriented (1840×3271, Instagram Story proportions) composition: a floral corner frame (matching the rest of the pack's palette) around an illustrated portrait of the couple in wedding dress. Shipped both as a flattened composite (`ig_v2.png`) and its three individual layers (`ig_v2-1.png` background gradient, `ig_v2-2.png` frame, `ig_v2-3.png` portrait).

**Revised (Phase 8, §27): uses the three separate layers, not the flattened composite** — background and frame as independent full-bleed elements behind the section's own text content, portrait as a separate in-flow image below the text panel, rather than one small rounded "card" showing the whole flattened design. See §27 for the full layout.

- **Instagram live link:** the closing section also gets a "Watch Live on Instagram" / "Saksikan Live di Instagram" link (with an Instagram glyph icon, §27) to `instagram.com/hauradhiyaa` (the bride's handle) — the couple's ceremony will be shared live there. `content.instagram = { handle, url }` field in `content/wedding.ts`; label text in `labels.ts` (`share.watchLive`) per the usual bilingual pattern.
- `ig_v1/` (an earlier draft of the same composition, also supplied) is **not used** — superseded by `ig_v2/` per explicit instruction.

## 17. Forced light-only theme (contrast bug fix)

**Root cause of the reported "white text on white background":** `globals.css` still had a `@media (prefers-color-scheme: dark)` block flipping `--foreground` to a near-white color for visitors with a dark-mode OS/browser preference — left over from the original scaffold, never removed even after the site committed to a light-only illustrated design (the "all black" dark-mode issue, resolved earlier by adding backgrounds, never actually removed this rule). Several sections have plain `<p>` tags with no explicit text color (inheriting `body`'s `color: var(--foreground)`), sitting inside the `bg-white/75` contrast panels (§12/Step 6.8) — under a dark-mode OS preference this rendered as near-white text on a translucent white panel, exactly the bug reported.

**Fix:** removed the `@media (prefers-color-scheme: dark)` block entirely — `--foreground`/`--background` are now fixed light-mode values regardless of OS preference, matching the fact that no part of this design was ever actually dark-mode-aware (illustrated backgrounds, translucent white panels, and dark-gray text are all light-theme-only choices). Also removed the one remaining `dark:` Tailwind variant pair on `MusicPlayer`'s toggle button, for the same reason. Verified via Lighthouse `color-contrast` (0 failures) under both the system's light *and* dark preference emulation — previously only checked under light.

## 18. Scroll lock + hidden scrollbar

- **Scroll lock:** while the Cover's pre-tap overlay is showing (`!opened` in `OpenedContext`), `document.documentElement`'s `overflow` is set to `hidden` via a small `ScrollLock` client component — without this, the overlay is `position: fixed` and visually blocks the page, but wheel/touch scroll events still bubble to `<html>`/`<body>` and scroll the content underneath it. Restored to normal once the visitor taps to open.
- **Hidden scrollbar:** `scrollbar-width: none` (Firefox) + `::-webkit-scrollbar { display: none }` (Chromium/WebKit) applied globally in `globals.css` — purely visual, scrolling still works via wheel/touch/keyboard once unlocked.

## 19. Cover asset-loading gate

The pre-tap "Tap to Open" / "Buka" button only renders once every entry in `PRELOAD_ASSETS` (`Cover.tsx`) has settled — previously the button appeared immediately on mount regardless of whether anything had actually finished loading, which could show a working button before the next section's assets were ready. A small loading indicator (spinner + "Memuat..."/"Loading..." label, `labels.cover.loading`) shows in the button's place until then, its progress bar driven by `loadedAssets.length / PRELOAD_ASSETS.length`.

**What's gated is not what's on screen.** `PRELOAD_ASSETS` is a background preload list, not the pre-tap overlay's own artwork — the overlay renders `background_cover`, `landscape_cover`, `birds_banner_cover`, and the `name/` combined signature directly, from their own hardcoded `src`s, independent of this gate. `PRELOAD_ASSETS` instead warms the cache for what the visitor sees *next*: the Couple Profile portraits/nickname art (`bride.photo`, `bride.nameImage`, `groom.photo`, `groom.nameImage`, `coupleSignatureImage`) plus, since Phase 9 (2026-08-15), the background music track (`content.music.src`, only when `musicEnabled`) — so by the time the visitor taps through, those assets are already cached instead of popping in, and the music is already buffered instead of stalling on the same click that has to also satisfy the browser's autoplay gesture requirement.

**Mixed asset types:** the preload effect (`Cover.tsx`) branches per entry — images use `new window.Image()` (`window.Image` explicit, since the file's own `import Image from "next/image"` shadows the global constructor) and resolve on `onload`/`onerror`; the audio entry uses `new Audio()` and resolves on `canplaythrough` (enough buffered to play without stalling) or `error`. Either outcome counts toward the gate — a failed asset can't wedge it open forever (same reasoning as the 30s auto-refresh fallback, §29).

## 20. Centered desktop layout frame

Revised (Phase 7, in response to "layout looks messy" on wider viewports): the whole page's scrolling content (`<main>`) is now wrapped in a `max-w-md mx-auto bg-white` column, matching the common convention for this genre of site (single-column, phone-proportioned, centered with visible margins on desktop rather than every section's background stretching full-bleed edge-to-edge on a wide monitor). `--background` (globals.css) is a soft off-white distinct from the column's own white, so the column reads as a visible card against the page backdrop on wide viewports.

**Deliberately not** using a `transform` on that wrapper to give it a CSS containing block for `position: fixed` descendants — that trick was tried first, but it would also re-target Cover's full-viewport pre-tap overlay and MusicPlayer's toggle (both `position: fixed`) to be positioned relative to the wrapper's own box instead of the real viewport. Since the wrapper is a normal-flow block spanning the *entire* page's content height (many stacked `min-h-screen` sections), that breaks both: MusicPlayer would scroll away with the page instead of staying pinned (its whole point), and Cover's `inset-0` overlay would stretch to the wrapper's full multi-screen height instead of exactly one viewport.

**Actual fix:** Cover's overlay and MusicPlayer's toggle each center themselves independently to match the column's width, with no ancestor transform involved — `fixed` + `left-1/2 -translate-x-1/2` + `w-full max-w-md` — a standard fixed-element horizontal-centering technique that stays genuinely pinned to the real viewport (so MusicPlayer still persists across scroll, and Cover's overlay still covers exactly one screen), just width-capped and centered to match the content column beneath it.

## 21. Cover's floral arch split into edge-pinned halves

Revised (Phase 8, follow-up report after §20 shipped): `opening/arch_opening.png` (used in Cover's revealed, in-flow hero) is a single image containing *both* floral clusters — one hanging from the top-left, one from the top-right, with empty space between them for the greeting text. Rendered as one `object-contain` image inside a container, the whole composition (both clusters *and* the gap) scales and centers as a unit — so on any container wider than the image's own aspect ratio, both clusters visibly drift toward the middle with dead space at the true edges, instead of staying anchored to them. This is what the "image in the middle even on PC" report was describing.

**Fix:** the source PNG (720×608) was cropped at the exact midpoint into two independent files — `img/opening/arch_opening_left.png` (354×608, trimmed) and `arch_opening_right.png` (355×608, trimmed), each still bleeding to its own crop edge (the original composition already had each cluster extending to the image's true left/right edge, confirmed by inspecting the trim bounds — no content was cut off by the split). Cover.tsx now renders them as two independent absolutely-positioned elements: left half at `left-0`, right half at `right-0`, each `w-1/2` of the section with a fixed height (`h-[47%]`, unchanged from before) and `object-contain` + `object-left-top`/`object-right-top`.

**Superseded (Phase 10, §35):** Cover's revealed, in-flow hero — the section this whole entry describes — was removed entirely; the arch halves (by then already the *shared* `components/arch_flower1_left/right.png` assets, §30) are Countdown's alone now. Left as-is below since it was accurate history at the time; not rewritten after the fact (same policy as TASK.md's own note on this).

Since each half's box is always wide enough to comfortably fit its content at the height-constrained size (the per-half aspect ratio is roughly half as wide as the combined image's, so a `w-1/2` box has ample headroom), each cluster renders at a **constant size** set by the section's height, not by the container's width — and stays flush against its own edge. The gap between the two clusters grows as the viewport widens instead of both clusters shrinking and sliding toward the center. Verified visually at mobile (390px, near-identical to the pre-split rendering) and desktop (1440px, clusters now pinned to the card's left/right edges with a wide gap between, instead of one small centered composition).

The combined `img/opening/arch_opening.png` is no longer referenced anywhere and was deleted from `public/img/` (the untouched original stays in `assets/wedding-illust/opening/` per the standard two-tier pipeline, §12).

## 22. Couple Profile order: bride first

Revised (Phase 8, follow-up): Couple Profile's two split sections (§15) now render bride, then groom — reversed from the initial split, which had matched the old combined layout's groom-first ordering. Purely a `content/wedding.ts`-consumption order change in `CoupleProfile.tsx` (which `PersonSection` call comes first); Cover/Share's combined signature wasn't touched at the time — fixed separately in §26.

## 23. Fixed: opening sometimes landed mid-page instead of at Cover

**Bug report:** after scrolling around the site, reopening (or reloading) sometimes revealed the page already scrolled down to a middle section (e.g. Couple Profile) instead of starting at Cover.

**Root cause:** Chrome (and most other browsers) restore the previous scroll position automatically on a manual reload (`history.scrollRestoration` defaults to `"auto"`). `opened` state itself always resets to `false` on a fresh page load (it's plain React state, not persisted), so the pre-tap overlay correctly shows again — but nothing was resetting the actual `window.scrollY`, which the browser had already restored to wherever the visitor had scrolled to in a previous visit/session. The overlay's own visibility doesn't depend on scroll position, so it *looked* fine at first glance, but the content sitting underneath it (revealed the instant scroll unlocked) was wherever the browser had put it — not necessarily the top.

**Fix**, two layers:
1. `ScrollLock` now sets `window.history.scrollRestoration = "manual"` once on mount (opts out of the browser's automatic restore) and force-scrolls to `(0, 0)` immediately, before the lock even engages — so the pre-tap overlay always covers a page that's genuinely at the top underneath it, not just visually appears to.
2. `Cover.tsx`'s `handleOpen` *also* force-scrolls to `(0, 0)` right when the visitor taps "Buka," as a second, independent guarantee — defense in depth, since this is a "the couple's guests see something broken" class of bug or if either layer alone doesn't apply.

Both use `behavior: "instant"` explicitly (not the CSS default) — see §24, where `scroll-behavior: smooth` is turned on globally for the new scroll-snap page transitions; without forcing `instant` here, these invisible resets would inherit that and animate, which isn't the intent for a pre-reveal reset.

**Verified:** scrolled deep into the page (`scrollY` 2532px), reloaded — confirmed `scrollY` was `0` immediately after reload (not restored), the pre-tap overlay was showing, and tapping it open landed exactly on Cover content, not the previously-scrolled section.

## 24. Scroll-snap: section transitions as animated "page changes"

Every section is already a full `min-h-screen` block with its own entrance animation (`MotionSection`'s fade + slide-up, triggered `whileInView`) — but on a freeform continuous scroll, a visitor could stop partway between two sections, so transitions didn't reliably read as distinct "page changes." CSS scroll-snap (`scroll-snap-type: y mandatory` on `html`, `scroll-snap-align: start` on every top-level `<section>` under `<main>`) makes the scroll itself settle on section boundaries, with `scroll-behavior: smooth` for a glide rather than an abrupt jump — the combination is what reads as "changing pages," carrying each section's existing entrance animation along with it rather than needing a separate new animation system. `prefers-reduced-motion: reduce` turns both off (`scroll-snap-type: none`, `scroll-behavior: auto`), consistent with how every other animation in this project already respects that preference.

On top of the CSS snap, a small `SectionScroll` client component (mounted in `page.tsx` next to `ScrollLock`) makes wheel scrolling feel like page-turn navigation. Wheel events are intercepted (so the mandatory snap can't yank every tick to a boundary) and accumulated; once the visitor has wheeled the equivalent of **35% of the viewport height** (`SCROLL_THRESHOLD_RATIO` in `components/SectionScroll.tsx`) in one direction, the page smooth-glides exactly one section in that direction (`window.scrollBy` ± one viewport, riding the global `scroll-behavior: smooth` from above) and the accumulator resets. Reversing direction resets the accumulator. Keyboard and touch scrolling are untouched — they still use the CSS scroll-snap's own snap-to-section behavior. Exceptions: events over a `textarea` or an element marked `data-no-scroll-jump` (the Guest Wishes comment list, which scrolls internally — §32) are left untouched so inner scrolling still works; the handler is inert while the Cover overlay is showing (`!opened`) since scroll is locked anyway, and `ctrl`/`cmd`-wheel (zoom) passes through. This mirrors the "page-turn" pattern common in this genre of site, driven by an explicit viewport-fraction threshold, and respects `prefers-reduced-motion` via the shared CSS `scroll-behavior` toggle above.

**Design note (2026-08-15):** the SectionScroll wheel-jump was added on top of the earlier CSS-only snap. The mandatory snap alone made *any* wheel tick from the top of a section snap to the next boundary — that's the behavior a visitor would read as "scrolls to next page" but it couldn't express "only after 35% of the viewport," so the JS accumulator was layered on to realize that threshold explicitly. Verified in headless Firefox (wheel over the section, wheel over the form, keyboard, and clamp-at-bottom cases); wheel-over-inner-scrollables was confirmed via synthetic dispatch — see §32 for why real-gesture simulation wasn't trustworthy there.

**Verification note:** the CSS itself was confirmed correctly applied (`getComputedStyle` on `html` reports `scroll-snap-type: y mandatory`; all 7 top-level sections report `scroll-snap-align: start`), and this is a mature, near-universally-supported CSS feature — but headless-Chromium testing via Puppeteer's synthetic wheel/touch event dispatch did **not** reliably reproduce exact snap-settling in this environment (scroll position after a simulated gesture came to rest a bit short of the true section boundary rather than exactly on it, across multiple attempts including a raw CDP touch-swipe simulation). This looks like a synthetic-input limitation of headless automation rather than a CSS defect — real touch/wheel/trackpad input goes through the browser's native gesture-recognition pipeline that headless-injected events don't fully replicate — but it means this specific behavior wasn't empirically confirmed pixel-exact the way other changes in this project have been. Worth a real-device check.

## 25. Guest Wishes name field pre-filled from `?to=`, not locked

The same `to` query param that personalizes Cover's greeting (§4) now also seeds the Guest Wishes name field's *initial* value — `?to=Jane` shows "Dear Jane," on Cover **and** starts the Wishes form's name input with "Jane" already typed in. It's a plain initial value on the existing controlled `<input>` (`useState(greetingName ?? "")`), not a `readOnly`/`disabled` field — the visitor can freely edit or clear it before submitting, same as if they'd typed it themselves. This matches the existing placeholder's own rule (§4): "the couple crafts a personalized link... any string works, nothing is checked" — pre-filling is a convenience, not a lock, and the guest is always free to correct it (e.g. sign as "Jane & family" instead, or a nickname).

`Wishes` now reads the same `greetingName` prop `Cover` does, passed down from the `?to=` param in `page.tsx` (previously only `Cover` received it).

## 26. Signature text order: bride first ("Ara & Zharfan")

The `name/ara-zharfan.png` signature image itself (§14) has always read "Ara & Zharfan" — bride's name on top, groom's below the "&" — but the *text-based* renderings of the same couple-name pair (page `<title>`, the image's own `alt` text on Cover and Share) were built from `${groom.nickname} & ${bride.nickname}`, backwards from what the art actually shows. Fixed to `${bride.nickname} & ${groom.nickname}` in `layout.tsx` (site title), `Cover.tsx` (`coupleNames`, used for both states' `alt` text), and `Share.tsx` (`alt` text) — now consistent with the image and with Couple Profile's bride-first order (§22). `noteFromUs.signature` in `content/wedding.ts` was already `"Ara & Zharfan"` (unaffected by this fix).

## 27. Closing section rebuilt from `ig_v2/` layers (Phase 8)

Revised: the Share/Closing hero, previously a small rounded "card" showing the flattened `ig_v2.png` composite, is now built from the three individual `ig_v2/` layers (§16) — matching the layered-asset approach used everywhere else in this project rather than being the one section relying on a pre-flattened image:

- **Background:** `ig_v2-1.png` → `img/closing/ig-v2-background.png` (max 1080px, background convention) — full-bleed (`fill object-cover`), directly behind everything else in the section.
- **Frame:** `ig_v2-2.png` → `img/closing/ig-v2-frame.png` — also sized at max 1080px, *not* the usual 800px accent cap, because its trimmed content (1840×3230 within the 1840×3271 canvas) spans almost the entire canvas as a full corner-to-corner border, not a small decorative element — the 800px accent convention is for things like `components/lily.png` that render small in a corner; this one is as dominant as the background it sits over. Also full-bleed (`fill object-cover`), stacked directly on top of the background — since both source layers share the same canvas dimensions, independently `object-cover`-fit elements stay visually aligned with each other automatically (no manual position matching needed).
- **Portrait:** `ig_v2-3.png` → `img/closing/ig-v2-portrait.png` (trimmed to its content bounds — 1256×1184 within the canvas — then resized to max 900px, portrait convention) — originally a separate in-flow element below the text panel; **repositioned to a bottom-pinned footer in a follow-up, §28.**
- **Content, top to bottom inside the usual `bg-white/75` text panel:** closing line (`content.share.closingLine`) → couple signature image (`content.coupleSignatureImage`, §14) → "Watch Live on Instagram" button with an inline Instagram glyph icon. (Copy Link / Share-to-WhatsApp buttons were kept at first, then removed in the §28 follow-up.)
- **Instagram icon:** a small hand-written inline SVG (`src/components/icons/InstagramIcon.tsx`) — a generic rounded-square/camera-lens-circle/corner-dot outline (the common open-source icon-set shape, not Meta's trademarked wordmark asset) — rather than adding an icon-font/library dependency (Font Awesome, etc.) for a single glyph. `currentColor`-based so it inherits the button's text color; `aria-hidden` since the button's visible text already reads "Watch Live on Instagram"/"Saksikan Live di Instagram".

The old `closing/Background_closing.png`, `doodle_closing.png`, `frame_closing.png`-derived `img/closing/{background,doodle,frame}.png` and the flattened `img/closing/ig-v2.png` are no longer referenced anywhere and were deleted (confirmed via `grep` first) — the source files stay untouched in `assets/wedding-illust/` either way, per the standard two-tier pipeline (§12).

## 28. Uniform music icon + Closing follow-up: remove copy/share buttons, portrait as bottom footer

**Music icon:** the floating music toggle used mismatched emoji (🎵 playing / 🔇 muted) — different subjects (a note vs. a muted speaker), inconsistent rendering across OS/browser emoji fonts, and not a matched pair. Replaced with `src/components/icons/MusicIcon.tsx` — one inline SVG component, `muted` prop just overlays a diagonal slash on the *exact same* note-glyph path, so the two states are guaranteed to be the same icon family rather than two unrelated symbols. Same approach as `InstagramIcon` (§27) — no icon-font/library dependency for a couple of glyphs, `currentColor`-based, `aria-hidden` (the button already has a text `aria-label`).

**Closing section, follow-up to §27:**
- **Copy Link / Share-to-WhatsApp buttons removed** — the section's one action is now "Watch Live on Instagram." Removed the now-dead `copied`/`shareUrl` state, `handleCopyLink`, and the `whatsappHref` computation from `Share.tsx` entirely (not just hidden) — with no other client-side state or effects left, `Share.tsx` no longer needs `"use client"` at all, a nice side benefit (one less client bundle). Cleaned up the now-fully-unused `labels.share.copyLink`/`shareWhatsApp` entries from `labels.ts` (§9 updated — feature removed, not just this one section's UI).
- **Portrait repositioned to a bottom-pinned "footer,"** not an in-flow element anymore. Root cause of the reported "weird space below it, looks cut off": the section centered its flow content (text panel + portrait) as a group via `justify-center`; whenever that group's total height came in under one viewport, the leftover space split above *and below* it — landing a visible gap of bare background between the (upper-body-only, not full-figure) portrait and the section's true bottom edge, which read as the portrait being awkwardly cropped and floating rather than intentionally placed. Fixed by making the portrait `absolute inset-x-0 bottom-0` (ignoring flex flow entirely) and removing the section's bottom padding (`px-8 pt-8`, no `pb-`) so it sits flush against the literal bottom edge — a deliberate "footer," no gap possible by construction. Also enlarged (`w-56` → `w-72`, ~28% wider) per instruction, `object-bottom` so it stays anchored low within its own box.

## 29. Cover loading: progress bar + 30s auto-refresh (and the real "stuck loading" root cause)

**What was actually wrong:** the user reported the pre-tap "Buka" button sometimes never appearing — stuck on the loading state indefinitely. Investigated by reproducing against the real public tunnel URL (not just `localhost`, which never showed the problem) with Puppeteer, cache disabled, full network-request timing. Found the requests for all 4 gating images actually completed successfully in under a second — so the loading-gate *logic* (Step 7.3, `loadedCount`/`onLoad`/`onError`) was never the bug. Then checked what was actually running inside the live `web` container: **`npm run dev` (Turbopack dev server + HMR), not the prod build** — despite `.env` saying `APP_ENV=prod`. My own prior `./stack.sh restart` (switching back to prod at the end of the previous session's work) had been interrupted between turns and never actually completed; a follow-up health check that only asserted `curl` returned `200` didn't catch it, since `next dev` also serves 200s. Dev mode is exactly the thing TECH_STACK.md §5 already documents as unsafe to expose publicly — its HMR websocket and general instability under real (non-localhost, non-Puppeteer-loopback) network conditions is the actual, most likely explanation for the intermittent stalls. Fixed by re-running `./stack.sh restart` and this time verifying the actual **process** inside the container (`docker exec ... ps aux` showing `next-server`/`npm run start`, not `next dev`), not just an HTTP status code — confirmed the button now appears in ~1.1s through the real tunnel, reproducing the exact scenario that was stuck before.

**Still worth building regardless of that specific incident** — the requested progress bar and auto-refresh are good defensive UX for *any* class of stuck-loading cause (a flaky mobile connection, an ad-blocker silently dropping one request, a future regression), not just the one found this time:

- **Progress bar:** replaced the spinner with a real `loadedCount / PRETAP_IMAGE_COUNT` progress bar (a plain `<div>` width percentage, `transition-[width] duration-300`) — visibly fills as each of the 4 required images settles, instead of an indeterminate spinner that gives no sense of whether anything is actually happening.
- **30-second auto-refresh:** a one-time `setTimeout` (mount-once effect) that calls `window.location.reload()` if `assetsReady` still hasn't become true after `AUTO_REFRESH_MS` (30,000ms). Reads the latest `assetsReady` via a ref (`assetsReadyRef`, kept in sync by a separate effect) rather than the stale value closed over when the timer was scheduled, so it correctly no-ops if loading actually finishes any time before the deadline. A last-resort recovery, not a fix for any specific cause — if the underlying condition causing the stall persists (e.g. a truly broken deployment), the reload will just get stuck again and the visitor is no worse off than before; if it was transient, the reload gives them a working page without needing to manually refresh themselves.

## 30. Cover's arch halves promoted to shared `components/` assets, reused in Countdown

The two arch halves from §21 (`img/opening/arch_opening_left.png`/`_right.png`) are renamed and moved to `img/components/arch_flower1_left.png`/`_right.png` — no longer Cover-specific, now a shared decorative asset alongside the other `components/` florals (§12). **Countdown's own single `countdown/arch.png`** (the `location option 2/` arch, CONTENT_DESIGN.md §12) **is retired and replaced with these same two halves**, matching Cover's split-halves treatment instead of Countdown having its own distinct arch composition. `img/countdown/arch.png` deleted (confirmed unreferenced via `grep` first); `countdown/background.png`/`silhouette.png` (still `location option 2/`-derived) are unaffected.

**Positioning, in both Cover and Countdown:** revised from edge-flush (`left-0`/`right-0`, §21) to intentionally bleeding off-screen — `left: -5%` / `right: -5%` instead of `0`, so roughly the outer 5% of each half's box (and the artwork nearest that edge) renders past the section's boundary and gets clipped by `overflow-hidden`, rather than stopping precisely at it. Deliberate, not a bug — makes the florals feel like they extend beyond the frame rather than being neatly bounded by it.

**Animation:** a slow rotational sway (`origin-top` — pivoting from the top like a real hanging branch/vine rather than rotating around its own center) instead of Cover's previous static positioning for these halves. Left and right halves are deliberately *not* mirrors of each other — tuned by hand to asymmetric values rather than a symmetric formula, which reads less like one mechanical animation reflected twice:
- Left: `rotate: [0, -7, 0, 3, 0]`, 7s cycle.
- Right: `rotate: [0, 10, 0, -3, 0]`, 10s cycle, 0.5s delayed relative to the left.

Respects `prefers-reduced-motion` (`rotate: 0`, no animation) via the same `useReducedMotion` pattern used everywhere else in this project — verified empirically both ways: computed `transform` matrix sampled once a second showed continuous change under normal conditions, and stayed exactly `"none"` across multiple samples under reduced-motion emulation. Countdown didn't previously import from `motion/react` at all (no per-section idle animation before) — added `motion`/`useReducedMotion` there for this. Identical values used in both Cover and Countdown (Countdown's were tuned first in Cover, then synced over — kept in lockstep since these are the same shared asset appearing twice, §14/§27's "treat as content" principle applied to motion too, not just the art itself).

**Layering:** both halves sit at `z-20` in both sections — above their `bg-white/75` content panels (`z-10`), the highest layer in either section. Deliberate, per instruction: the florals read as sitting in front of everything else rather than tucked behind the card, closer to a layered-paper-cutout look. Doesn't cause any actual visual collision with the text panels in practice — the arch halves are confined to each section's outer edges and don't spatially overlap the centered panel content.

## 31. Asset pack moved out of `web/public/`; favicon added

**Asset pack relocated:** `web/public/assets/` (the delivered illustration pack, §12 — 336MB, never referenced directly from app code) moved to top-level `assets/wedding-illust/` (repo root, sibling of `web/`, not inside it). It was only ever a *source* for the resize pipeline (§12's two-tier explanation) — sitting inside `web/public/` meant the entire pack was part of the `web` service's Docker build context (no `.dockerignore` excluded it), bloating every image build for no reason since none of it ships to the browser directly. Every current-state reference in these docs was updated to the new path; TASK.md's historical Phase 6/7 step log still says `web/public/assets/`/`public/assets/` in places — left as-is, since that was accurate at the time those entries were written (a build journal, not living reference material).

**Favicon added**, from a standard realfavicongenerator-style bundle supplied at `assets/favicon/` (kept untouched there as the source, per instruction — same "keep the original, only copy resized/derived output" pattern as `assets/wedding-illust/` → `web/public/img/`, §12):

| File | Copied to | Purpose |
|---|---|---|
| `favicon.ico` | `web/public/favicon.ico` | Classic favicon, `sizes="any"` |
| `favicon-16x16.png` / `favicon-32x32.png` | `web/public/` | Modern browser tab icon at specific sizes |
| `apple-touch-icon.png` | `web/public/` | iOS home-screen icon |
| `android-chrome-192x192.png` / `-512x512.png` | `web/public/` | Android home-screen/PWA icons, referenced from `site.webmanifest` |
| `site.webmanifest` | `web/public/site.webmanifest` | PWA manifest |

`src/app/favicon.ico` (Next's App Router auto-convention placeholder — the default Next.js logo icon shipped by `create-next-app`, never replaced until now) was **removed**, not just overwritten — Next.js treats `app/favicon.ico` as a special route that generates its own `<link rel="icon">`, and a same-named `public/favicon.ico` alongside it is a conflict. `layout.tsx`'s `metadata` export now declares `icons`/`manifest` explicitly instead, pointing at the `public/` files.

**One exception to the "everything from `content/wedding.ts`" rule:** `site.webmanifest`'s `name`/`short_name` (shipped empty by the generator) are filled in as static plain text ("Ara & Zharfan") directly in the JSON file, not pulled from `content.ts` dynamically. This is deliberate — it's PWA/home-screen metadata, not visible page content, and converting it to a generated route (`app/manifest.ts`) to keep it in sync with `content.ts` would be more machinery than a rarely-touched file like this warrants. If the couple's names in `content/wedding.ts` ever change, `public/site.webmanifest` needs a manual one-line edit to match — documented here specifically so that's not a surprise later.

## 32. Guest Wishes: card capped to one viewport, comments scroll internally (2026-08-15)

The Guest Wishes page grew unbounded as wishes accumulated — dozens of comments made the section taller than one screen, which both violated the site's "each page is one viewport" feel and broke the §24 page-turn scroll's assumption that one `scrollBy` = exactly one section. Now:

- The card is capped with `max-h-[calc(100dvh-4rem)]` — always fits inside one viewport, with the form and heading fixed and only the comment list shrinking.
- The comment list (`<ul data-no-scroll-jump>`) is the internal scroller: `min-h-0 flex-1 overflow-y-auto overscroll-contain` inside a `flex-1` wrapper. A soft gradient fade sits at the list's bottom edge as the scrollability cue — the site hides scrollbars globally (§18), so without it the list would look clipped with no hint it scrolls. `aria-hidden` and `pointer-events-none`, purely decorative.
- **Why `data-no-scroll-jump` matters here:** §24's wheel handler lets events over `[data-no-scroll-jump]` pass through untouched — so wheel over the comment list scrolls the list itself instead of page-turning, while wheel over the form (outside the list) still advances the page.
- **Verification note:** the layout was verified in headless Firefox with 23 comments — card `getBoundingClientRect()` stays within the viewport, list `scrollHeight` (1943px) > `clientHeight` (452px) with `overflow-y: auto`, and wheel-over-list vs wheel-over-form behavior confirmed. Real-gesture wheel simulation in headless Firefox was unreliable (marionette synthesizes wheel at the viewport center / on `body` rather than the pointer's hover target, so hover-dependent behavior can't be reproduced headlessly) — the event-target check itself was verified via synthetic `WheelEvent` dispatch at the element, and the layout logic is plain CSS/flexbox. Worth a real-device scroll-through to eyeball the feel.

## 33. Feature spec: Main Announcement (Phase 10, 2026-08-15)

New section, `id: mainAnnouncement`, inserted right after Cover (§1) — a formal Bismillah + wedding announcement, distinct from Cover's greeting and Couple Profile's per-person bios. Content lives in `content.mainAnnouncement` (`wedding.ts`); component is `MainAnnouncement.tsx`.

**Container — `letter_frame.png`, not a hollow border.** Checked with ImageMagick before using it (`-format "%[fx:mean.a]"` on a center crop returned fully opaque, `#FFFBF5` cream, vs. fully transparent at the corners) — unlike `note/frame.png` (§4/NoteFromUs, hollow center, artwork sits *behind* it), `letter_frame` is itself an opaque paper-card graphic with transparent rounded corners. So it doubles as the content's own background; no separate `bg-white/75` panel layered on top, unlike every other section. Rendered as a `next/image fill` inside an `aspect-[351/800]` box (the resized asset's own ratio) sized `h-[94vh] max-h-[840px]` (revised up from `h-[85vh] max-h-[720px]` — per-instruction "make the container bigger" — a **viewport-relative** unit, not `h-[92%]` — percentage height doesn't resolve inside a `min-h-screen flex flex-col` section, since `min-height` isn't a definite height for percentage children to reference; caught by an actual `getBoundingClientRect()` measurement showing 0×0 before the fix, not by inspection).

**Layout — three independently-positioned zones, not flex flow.** Originally a single flex column with `justify-center` + hand-tuned `mt-*`/`gap-*` margins between blocks; replaced with three `absolute` blocks positioned by the frame art's actual geometry, since trial-and-error margins couldn't guarantee text stayed clear of the frame's top/bottom corner-flower decoration:
- **Top zone** (Bismillah + `intro`) and **bottom zone** (`dateIntro` + date): both `13.20%` (`top: 13.20%` / `bottom: 13.20%`) — unified to one value per instruction. Originally measured independently from `letter_frame.png`'s corner-flower extent (421px top / 420px bottom of the source PNG's 3182px height ≈ 13.23%/13.19%), close enough that using one shared value for both reads as intentional symmetry rather than a near-miss. Bottom zone uses `bottom` positioning (not `top` + a computed height) so its content naturally stacks *upward* from that anchor regardless of its own height.
- **Middle zone** (bride, "&", groom): `top-1/2 -translate-y-1/2` — vertically centered in whatever space remains between the other two zones, not pinned to a fixed offset.
- **Why `top`/`bottom` percentages, not `padding-top`/`margin-top`:** CSS resolves vertical `padding`/`margin` percentages against the containing block's *width*, not height — always, regardless of direction (a real, easy-to-miss spec quirk). `top`/`bottom` offsets on an absolutely-positioned element resolve against *height* instead, which is what "so many px down this 3182px-tall canvas" actually means. Same convention already used for the pre-tap overlay's PSD-derived layer positions (Cover.tsx, §12) — just not previously applied inside this section.
- **Verified precisely, not just visually:** `getBoundingClientRect()` on a live render measured both zones at exactly 101px off their respective edges on a 765px-tall box — `101/765 = 13.20%` both sides, matching exactly (not just approximately) after the unification. No zone overlaps another.

**Content** (all inside the frame, `t()`/`useT()` for bilingual strings):
- `bismillahArabic` — Arabic Bismillah, plain string (not `Bilingual`, same text regardless of locale, per instruction), `.font-arabic` (Noto Naskh Arabic, `layout.tsx` — none of the site's other fonts cover Arabic script), `dir="rtl"`.
- `intro` — bilingual announcement line ("Dengan memohon rahmat dan ridho Allah Azza wa Jalla..." / English rewrite, not a literal translation, since the source is only Indonesian). `text-[13px]` (up one step from the original `text-[11px]`, per an earlier instruction — "normal text" one size bigger).
- Bride: `nameImage` (nickname signature art, reused from Couple Profile) + `fullName` in `.font-script` (Great Vibes) colored `#a1c1da` (light blue, per instruction) + `parents` in small plain body text (`text-xs`, up from `text-[10px]`, no handwriting, smaller than the name).
- A large `.font-script` "&", `text-3xl` (down one step from `text-4xl`, per instruction) — **no dedicated ampersand asset exists** in `assets/wedding-illust/` (checked; the closest is the already-composed `name/ara-zharfan.png` combined signature, which isn't a standalone glyph) — rendered as styled text instead of sourcing/fabricating a new image.
- Groom: same treatment as bride.
- `dateIntro` — bilingual ("yang akan dilaksanakan pada" / "which will take place on"), `text-[13px]` (same bump as `intro`).
- `content.event.displayDate`, `.font-script`, colored `#adcd9f` (green, per instruction). Still shown here — the date was only removed from Countdown (§36), not here.

**Verified in headless Firefox:** section order via rendered background images (cover → main-announcement → countdown → couple profile ×2 → note from us → wishes → share, matching the reordered `sections` array with Love Story/Gallery/Gift correctly absent while off); computed `color` on the name/date elements exactly `rgb(161, 193, 218)` / `rgb(173, 205, 159)`; frame image `naturalWidth/Height` > 0 and `complete: true`; content box `scrollHeight === clientHeight` at every revision (no overflow); the three-zone `top`/`bottom`/middle offsets measured as described above; computed `font-size` on the "&" confirmed `30px` (`text-3xl`); language toggle confirmed still updates all three zones. A rendered screenshot was also visually reviewed each time — the frame's floral border, Bismillah, both names, and the date all sit within the card with comfortable margins and no overlap with the corner artwork.

**Page order, revised:** cover → **mainAnnouncement** → countdown → coupleProfile → loveStory (off) → gallery (off) → **noteFromUs** (moved later — was right after coupleProfile) → wishes → gift (off) → share. `SectionId` (`wedding.ts`) and `sectionComponents` (`registry.tsx`) both reordered to match; `PAGE_DESIGN.md`'s page numbers renumbered accordingly.

## 34. Guest-facing language toggle (id/en)

`labels.ts`'s `locale` was previously a module-level constant — the whole site's language was a build-time choice, not something a guest could change. It's now `defaultLocale` (kept only as the starting value and as `t()`'s fallback for contexts that can't react to client state, e.g. `layout.tsx`'s server-rendered `<title>`/OG metadata) plus a real client-side toggle:

- **`context/LocaleContext.tsx`** — `LocaleProvider` (React state, mirrors `OpenedContext`) wraps the whole page in `page.tsx`; `useLocale()` exposes `{ locale, toggleLocale }`; `useT()` is a drop-in replacement for the old static `t` import, bound to the live locale instead of the fixed default.
- Every section that shows bilingual text now calls `useT()` instead of importing `t` directly. Four components that displayed static-per-request text (`NoteFromUs`, `Share`, `CoupleProfile`, `LoveStory`) weren't client components before this and had to become one, since reacting to the toggle requires client-side state.
- **`components/LanguageToggle.tsx`** — floating button, same fixed-footer pattern as `MusicPlayer`, anchored bottom-left (music stays bottom-right — verified no collision). Hideable via `languageToggleEnabled` in `wedding.ts` (same pattern as `musicEnabled`) for a dev who wants the site fixed at one language.
- **On top of Cover, not just the revealed page:** the toggle's wrapper is `z-[60]`, above Cover's pre-tap overlay (`z-50`, `Cover.tsx`) — deliberately, so a guest can switch language before ever tapping "Buka"/"Tap to Open," not just after. `MusicPlayer` stays at its original `z-40` (unchanged, not asked for).
- **Notation:** shows the flag emoji of the *current* locale (🇮🇩 / 🇬🇧) rather than "ID"/"EN" text — tap toggles to the other. The `aria-label` still spells out the target language in words for screen readers, independent of the visual notation.
- **Both floating buttons** (`MusicPlayer`, `LanguageToggle`) use `bg-white/50` (was `bg-white/80`) — a more see-through circle over the artwork behind them.
- **Verified in headless Firefox:** toggling flips visible text between locales immediately, including on sections not yet scrolled into view (no reload); flips back correctly; `languageToggleEnabled = false` hides only the language button, leaving the music button untouched; toggling while the Cover overlay is still showing works and doesn't accidentally trigger "Tap to Open"; computed `background-color` alpha ≈ 0.5 on both buttons; button `getBoundingClientRect()`s confirmed non-overlapping.

## 35. Cover's revealed in-flow hero removed — "Tap to Open" now lands on Main Announcement (Phase 10, 2026-08-15)

Before Main Announcement existed (§33), Cover rendered two near-identical greeting screens: the pre-tap fixed overlay, then — once tapped — an in-flow `<section>` underneath with the same "Wedding Of / [names] / Dear [name]," content again, over `opening/background_opening.png` with the animated `arch_flower1_left/right.png` halves (§21, §30). With Main Announcement now the very next section after Cover, that second greeting screen was redundant — a guest would tap "Buka" and land on a near-duplicate of what they'd just seen, one scroll-turn *before* reaching the actual announcement.

**Removed entirely** (not hidden behind a flag — deleted from `Cover.tsx`): the whole in-flow `<section>`, its two `motion.div` arch halves, and the `MotionSection` greeting panel. Cover now renders only the pre-tap overlay (`<AnimatePresence>` + the fixed `motion.div`) — `handleOpen`'s existing `window.scrollTo({ top: 0 })` (§23) is unchanged and now lands exactly on Main Announcement, since that's the first `<section>` in `<main>`'s document flow once Cover contributes none.

**No orphaned assets** — both `opening/background_opening.png` (still Wishes' reused background) and the arch halves (still Countdown's framing arch, shared since Phase 8 per §30) stay in active use elsewhere; only Cover's own use of them was removed. `main > section` count dropped from 8 to 7 with the default section config.

**Verified in headless Firefox:** post-open `scrollY === 0` lands on Main Announcement's Bismillah text (not a second greeting); total `main > section` count is 7; one scroll-turn wheel gesture from Main Announcement lands on Countdown with its arch still animating; language toggle still functions post-open. A screenshot taken immediately after clicking "Buka" was also visually reviewed — the letter-frame card is the first thing shown, full-viewport, no intermediate screen.

## 36. Countdown: date replaced with a Qur'an quote (QS. Ar-Rum: 21)

The date used to repeat here (`content.event.displayDate`, same value already shown on Main Announcement) — removed per instruction, replaced with a short devotional quote directly beneath the timer.

- **`content.countdown.quote`** (`wedding.ts`) — `reference: "QS. Ar-Rum: 21"` (plain string, kept as-is regardless of locale — the "QS." abbreviation convention is Indonesian but functions as a citation label either way, not prose to translate) + `text: Bilingual`.
- **Meaning only, no Arabic script — per instruction.** Both language versions are the standard published translations (Sahih International for English, Kemenag/Indonesian Ministry of Religious Affairs for Indonesian), not a paraphrase — the same source-fidelity standard as `bismillahArabic` on Main Announcement (§33), which is why this isn't wrapped in a `Bilingual` shape that includes an Arabic key at all.
- **Rendered as a `<blockquote>`**, italic, quotation marks (`&ldquo;`/`&rdquo;`), with the reference on its own line beneath in smaller muted text — visually distinct from the countdown digits above it, reads as a citation rather than more body copy.
- `content.event.displayDate` is not orphaned — still shown on Main Announcement (§33); only Countdown's copy of it was removed.

**Verified in headless Firefox:** the check was scoped specifically to Countdown's own `<section>` (`document.querySelectorAll('main > section')[1].innerText`), not the whole page body — a body-wide substring check would have false-positived on Main Announcement's own (unrelated, unchanged) date text still being present elsewhere on the page. Scoped correctly, Countdown's section text contains the quote and reference and no longer contains "Agustus 2026". Language toggle confirmed still translates the quote (checked for "tranquility" appearing after toggling to English).
