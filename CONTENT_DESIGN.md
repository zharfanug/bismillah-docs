# Content & Web Design
## Wedding Announcement Website

**Status:** Decided
**Last updated:** 2026-07-28 (revised: every section is independently toggleable/removable — see §2)

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

| # | Section | Default | Notes |
|---|---|---|---|
| 1 | Cover / Landing screen | **On** | Couple's names, wedding date, "tap to open," personalized greeting placeholder (§4). |
| 2 | Countdown timer | **On** | Live countdown to the event date/time. |
| 3 | Couple profile | **On** | Names, short bio/photo for bride & groom. |
| 4 | A Note From Us | **On** | Replaces the typical venue/location block — see feature spec §5. |
| 5 | Love story / timeline | **On** | Optional short story or milestones — on for now with placeholder content, toggle off any time. |
| 6 | Gallery | **On** | Photo (and optionally video) gallery with lightbox. |
| 7 | Guest wishes/comments | **On** | See feature spec §6 — the site's main interactive element now that RSVP is out of scope. |
| 8 | Digital gift info | **On** | Bank/e-wallet info, copy-to-clipboard, optional QR. |
| 9 | Share / closing | **On** | Copy link / share-to-WhatsApp, OG meta tags. |

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

- A "copy link" and/or "share to WhatsApp" button.
- Open Graph + Twitter Card meta tags (title, description, image) so the link looks good when pasted into a chat app.

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
