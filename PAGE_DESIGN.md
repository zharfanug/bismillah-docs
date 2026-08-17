# Page-by-Page Design
## Wedding Announcement Website

**Status:** Built (Phase 7 complete) — this doc now describes the shipped state, not a plan
**Last updated:** 2026-08-17 (Page 1 Cover's greeting wireframe corrected — trailing comma dropped from "Dear {Name}," per a direct `Cover.tsx` edit; see CONTENT_DESIGN.md §55). Previously, same day: A Note From Us (old Page 8) deleted outright, not just disabled; Test1/Test2 deleted too. Every page from old-Page-9 onward renumbered down by one — Guest Wishes 9→8, Digital Gift 10→9, Closing 11→10. `components/`'s last four images moved to `wedding/`, `components/` folder now gone; `InstagramIcon.tsx` and several dead asset files also removed. See CONTENT_DESIGN.md §52–§53 and the Resolved section's three newest entries below). Previously, same day: Page 11 renamed Share → Closing (`id: closing`) end-to-end; its `ig-v2-frame.png` swapped for three positioned flower corner images; dead "Watch Live on Instagram" button/label removed. See CONTENT_DESIGN.md §51 and the Page 11/Resolved entries below). Previously, same day: Backfilled several rounds of Countdown/A Note From Us changes this doc had missed — Page 5 (Countdown) rewritten to match its current two-card layout (note content + "Akad Nikah" card), bouquet-spray corner art, and bottom-only foliage silhouette; Page 8 (A Note From Us) flagged **off** by default, its content now duplicated into Page 5. See the Resolved section's 2026-08-16/2026-08-17 entries and CONTENT_DESIGN.md §47–§50). Previously, 2026-08-16 (New Page 2, Prolog (`id: prolog`) — two birds flying together + the QS. Ar-Rum: 21 quote, moved off Countdown; every subsequent page renumbered up by one — see CONTENT_DESIGN.md §41). Also 2026-08-16, earlier the same day: every page's own background image removed — replaced site-wide by one shared `BackgroundLayer`, `position: fixed` so it no longer scrolls with the page at all, same gradient + grain-crop look as Cover's overlay; each page's Assets bullet above updated to flag its background as superseded — see CONTENT_DESIGN.md §39. Also 2026-08-16, earlier still: Page 1 Cover's background image fetch replaced with a CSS gradient + real grain-crop overlay, landscape scene and birds/banner moved to a new `wedding/` folder — see CONTENT_DESIGN.md §38 (now itself superseded by §39, above). Previously 2026-08-15 (Phase 10/11: Couple Profile (Page 3) switched off by default — its bride/groom portraits moved onto Main Announcement's (Page 2) bottom corners instead, groom sized taller than bride, both bleeding past the section edge on purpose; the date's own bottom-anchored zone on Page 2 retired, `dateIntro`/date now sit two lines below the groom's name in the normal flow (CONTENT_DESIGN.md §37); page order changed again — Countdown and Couple Profile swapped, Couple Profile is now Page 3 and Countdown Page 4 (announcement, then who the couple is, then the countdown); new Main Announcement page inserted right after Cover — see Page 2; Cover's own revealed in-flow "second screen" removed entirely, so tapping "Buka" now lands directly on Main Announcement instead of a near-duplicate greeting first — see Page 1; A Note From Us moved to Page 7 (after Gallery); language toggle made guest-facing — see the floating-controls entries at the bottom; Countdown's date line replaced with a QS. Ar-Rum: 21 quote (meaning only) — see Page 4; Main Announcement's layout later switched to three independently-positioned blocks — see Page 2. Phase 9: Cover's wedding date removed from the pre-tap overlay; wheel scrolling converted to a 35%-threshold page-turn gesture — see CONTENT_DESIGN.md §24; Guest Wishes card capped to one viewport with an internal comment scroller. Each change is flagged in its page's entry below)

The site is still a **single long-scroll page**, not separate routes (per CONTENT_DESIGN.md §1). "Pages" below just means each full-viewport section, in scroll order — laid out here as individual text wireframes so you can quickly preview what's in each one before any real UI is built. Background music (§ at the bottom) is a floating control present on every page, not a page of its own.

Content shown is placeholder/draft text from CONTENT_DESIGN.md — swap freely.

**Every page below is independently removable/hideable** without touching component code — see CONTENT_DESIGN.md §2. Each heading lists its config `id` and default state; flip `enabled: false` for that `id` in the content file and rebuild to hide it (flip back to `true` to restore it — nothing is deleted).

---

### Page 1 — Cover  `id: cover` · default: **on**

```
┌──────────────────────────────┐
│                               │
│        [ background photo ]  │
│                               │
│      The Wedding Of           │
│                               │
│      Raka  &  Dinda           │
│                               │
│      Dear {Name}               │  <- from ?to= param, falls back to
│                               │     "Dear Family & Friends" (trailing
│                               │     comma dropped 2026-08-17)
│        [ Tap to Open ]        │
│                               │
└──────────────────────────────┘
```
- Couple's names (large, primary visual focus) — rendered as the hand-lettered `name/ara-zharfan.png` signature art, not font text (CONTENT_DESIGN.md §14).
- Personalized greeting placeholder (CONTENT_DESIGN.md §4).
- **No wedding date on the Cover** — the date was removed from both the pre-tap overlay and the (now-removed) revealed hero (2026-08-15); the date now appears only on the Countdown page, right beneath the timer.
- "Tap to open" interaction — only appears once the overlay's artwork has actually finished loading (a progress bar shows load progress until then, and a 30s auto-refresh recovers if it never finishes — CONTENT_DESIGN.md §19, §29); also the user gesture that unlocks the background music autoplay (browsers block autoplay without one).
- Page scroll is locked and the scrollbar hidden while this overlay is showing (CONTENT_DESIGN.md §18) — unlocked the moment "Tap to Open" is pressed.
- **Removed (Phase 10, 2026-08-15): the revealed, in-flow "second Cover screen."** Cover used to render its own in-flow `<section>` after the overlay closed — same greeting repeated over `opening/background_opening.png` with the animated `arch_flower1_left/right.png` halves — so tapping "Buka" landed on a near-duplicate of what the visitor had just seen, *before* reaching Main Announcement. That section is now gone entirely; Cover only ever renders the pre-tap overlay. Tapping "Buka" now reveals Main Announcement (Page 2) directly — verified `scrollY === 0` right after opening lands on Main Announcement's Bismillah, not a second greeting. `opening/background_opening.png` is unaffected (still Wishes' reused background) and the arch halves are unaffected (still Countdown's framing arch, CONTENT_DESIGN.md §30) — only Cover's own use of them was removed.
- **Assets (pre-tap overlay only, now):** background is a CSS gradient (`#B9D3DE` → `#F8F1E2`, colors sampled from the old `cover/background_cover.png`) plus `wedding/ltblue_white_transition.png`, a small real grain crop over the color transition band — replaces the old `background_cover.png` image fetch entirely (CONTENT_DESIGN.md §38). `wedding/house.png` (watercolor scene, formerly `cover/landscape_cover.png`), `wedding/birds_lace_banner.png` (names/date banner, idle bird/banner motion, formerly `cover/birds&banner_cover.png`), `name/ara-zharfan.png` (couple's names). The `[ background photo ]` placeholder in the wireframe above is these layered together.
- **Typography:** the greeting text block sits in a translucent white panel for contrast against the artwork.

### Page 2 — Prolog  `id: prolog` · default: **on** (new, 2026-08-16)

```
┌──────────────────────────────┐
│                               │
│         (bird)  (bird)        │
│                               │
│   "And of His signs is that   │
│    He created for you..."     │
│                               │
│        QS. Ar-Rum: 21         │
│                               │
└──────────────────────────────┘
```
- Two birds, pure CSS (no image files) — a sprite-sheet wing-flap animation (`wedding/bird_flap_sprite.svg`, self-hosted from a [CodePen technique](https://codepen.io/imdoug/pen/vYZNoYr), recolored white) combined with a `left`/`transform` keyframe animation that flies both birds across the section together, same direction, tightly spaced (short delay + small vertical gap only). Respects `prefers-reduced-motion` via the CSS Module directly. Full implementation notes: CONTENT_DESIGN.md §41.
- The QS. Ar-Rum: 21 quote (meaning only, no Arabic script) — moved here from Countdown (Page 5 below), which used to show it under the timer (§36, now superseded by §41).
- **Assets:** no background image of its own — the shared, fixed `BackgroundLayer` (CONTENT_DESIGN.md §39) shows through behind it. Quote sits in the usual `bg-white/75 backdrop-blur-sm` translucent panel for contrast; birds are outside that panel, directly on the backdrop.
- Full spec: CONTENT_DESIGN.md §41.

### Page 3 — Main Announcement  `id: mainAnnouncement` · default: **on** (new, Phase 10, 2026-08-15)

```
┌──────────────────────────────┐
│  ╭──────────────────────╮    │
│  │  بِسْمِ اللَّهِ الرَّحْمَٰنِ     │    │
│  │  الرَّحِيمِ                │    │
│  │                        │    │
│  │  Dengan memohon rahmat  │    │
│  │  dan ridho Allah Azza   │    │
│  │  wa Jalla dan dengan    │    │
│  │  kerendahan hati, kami  │    │
│  │  bermaksud menyeleng-   │    │
│  │  garakan pernikahan     │    │
│  │                        │    │
│  │    "Ara" (art)          │    │
│  │  Nama Lengkap Ara        │    │
│  │  Putri dari Bpk. Y &    │    │
│  │  Ibu L                  │    │
│  │                        │    │
│  │         &              │    │
│  │                        │    │
│  │  "Zharfan" (art)        │    │
│  │  Nama Lengkap Zharfan    │    │
│  │  Putra dari Bpk. Z &    │    │
│  │  Ibu S                  │    │
│  │                        │    │
│  │  yang akan dilaksanakan │    │
│  │  pada                   │    │
│  │  28 Agustus 2026         │    │
│  ╰──────────────────────╯    │
│ [bride]              [groom] │
└──────────────────────────────┘
```

- New section, inserted right after Cover — a formal Bismillah + wedding announcement, distinct from Cover's brief greeting and Couple Profile's per-person bios. Full spec: CONTENT_DESIGN.md §33.
- **Container is the content's own background**, not a decorative overlay — `content.letterFrameImage` (top-level, not nested under `mainAnnouncement` — moved there so Cover's `PRELOAD_ASSETS` can reference it directly; `wedding/letter_frame.png` as of 2026-08-16, moved from `components/`) is an opaque paper-card graphic (checked with ImageMagick: fully opaque cream center, transparent only at the rounded corners), unlike every other page's `bg-white/75` panel over a separate full-bleed background image.
- Bismillah shown in Arabic script (right-to-left), same text regardless of the site's language toggle — not translated. Uses a dedicated Arabic font (Noto Naskh Arabic) since none of the site's other fonts cover Arabic glyphs.
- Bride and groom each get their nickname signature art (reused from Couple Profile/Cover), their full legal name in a handwriting-style script font colored light blue (`#a1c1da`), and their parents line in small plain text (no handwriting, smaller than the name) beneath.
- A large handwriting-style "&" divides the two — rendered as styled text, not an image (no standalone ampersand asset exists in the source pack; checked before deciding this).
- `dateIntro` ("yang akan dilaksanakan pada" / "which will take place on") followed by the event date in the same handwriting script, colored green (`#adcd9f`). **Moved (2026-08-15):** used to be its own bottom-anchored zone (`bottom: 13.20%`, briefly `20%`); now just the last two lines of the name block itself, two lines below the groom's name — see CONTENT_DESIGN.md §37.
- **Bride and groom portraits now stand in this page's bottom-left/bottom-right corners** (added 2026-08-15, took over the role Couple Profile — now off, see Page 4 — used to serve). Layered *above* the letter_frame card, allowed to cover its corner artwork (and, to a lesser extent, the card's own text at their outer edges, judged acceptable by eye against real screenshots — see CONTENT_DESIGN.md §37 for why a strict zero-overlap rule was abandoned in favor of visual review). Both bleed intentionally past the section's own left/right edge (`-10%` offset, cropped by `overflow-hidden`) so they can render bigger without crowding the card's centered text column; groom is deliberately sized taller than bride. Full reasoning and the sizing iteration: CONTENT_DESIGN.md §37.
- **Layout:** the Bismillah/intro block sits in a top zone (`top: 13.20%`, matching where the frame art's corner flowers end); the bride/&/groom/dateIntro/date block is vertically centered (`top-1/2 -translate-y-1/2`) in the remaining space. There is no separate bottom zone anymore (retired along with the date's old position, §37). Full reasoning: CONTENT_DESIGN.md §33/§37.
- **Assets:** ~~`cover/background_cover.png` (reused, no dedicated background of its own)~~ **superseded (2026-08-16, CONTENT_DESIGN.md §39): background image removed, shared fixed `BackgroundLayer` shows through instead** + `letterFrameImage` (the card/container, `wedding/letter_frame.png` as of 2026-08-16) + `name/ara.png` / `name/zharfan.png` (nickname signatures) + `bride.photo` / `groom.photo` (the corner portraits, same full illustrated portraits Couple Profile used to show — `bride.png` re-edited 2026-08-15 to trim excess transparent canvas margin).

### Page 4 — Couple Profile  `id: coupleProfile` · default: **off** (was on — switched off 2026-08-15, §37)

**Off by default since 2026-08-15** — the bride/groom portraits it used to show now live on Main Announcement's bottom corners instead (Page 3), so there's no separate profile page in the default flow. The section still exists and works if re-enabled (`coupleProfile: true` in `wedding.ts`); everything below describes it as it renders when on.

Revised (Phase 7) — **two separate full-screen sections** instead of one combined overlapping layout (was too cramped on phone widths — CONTENT_DESIGN.md §15); order reversed to bride-first in Phase 8 (§22):

```
┌──────────────────────────────┐   ┌──────────────────────────────┐
│                               │   │                               │
│         [ bride photo ]        │   │         [ groom photo ]        │
│                               │   │                               │
│           "Ara" (art)          │   │         "Zharfan" (art)        │
│      Haura Dhiya Amaninida      │   │      Muhammad Zharfan Nugroho   │
│      Putri dari Bpk. C & Ibu D  │   │      Putra dari Bpk. A & Ibu B  │
│                               │   │                               │
└──────────────────────────────┘   └──────────────────────────────┘
       Section 3a — bride                  Section 3b — groom
```
- Each side gets its own full viewport: portrait, hand-lettered nickname signature (`name/zharfan.png` / `name/ara.png`, CONTENT_DESIGN.md §14), full legal name, and the placeholder parents line — all in the usual translucent white panel.
- No background image of its own — the shared, fixed `BackgroundLayer` (§39) shows through behind it, same as every other section.
- **Correction (2026-08-17):** this page's Assets bullet previously described the `bride & groom/` illustrated-pack portraits (`bride_bride&groom.png`/`groom_bride&groom.png`) and their paired frame/background — that stopped being accurate on 2026-08-16 (commit `364e9ec`) when `content.couple.{bride,groom}.photo` were quietly repointed to `wedding/bride.png`/`wedding/groom.png` instead, without a doc update at the time. See CONTENT_DESIGN.md §12's `bride & groom/` row for the full correction.
- **Assets:** `wedding/bride.png` / `wedding/groom.png` (portrait, via `content.couple.{bride,groom}.photo`) + `name/ara.png` / `name/zharfan.png` (nickname signature). Fade/slide-in per section as it scrolls into view.

### Page 5 — Countdown  `id: countdown` · default: **on**

```
┌──────────────────────────────┐
│  [bouquet]           [bouquet]  │
│                               │
│  "Our celebration was           │
│   intentionally small..."       │  <- card 1: note title/
│         — Raka & Dinda          │     body/signature (originally
│                               │     "A Note From Us," §48/§53)
│        Akad Nikah              │  <- card 2:
│  28 Agustus 2026 | pukul 9 WIB  │
│   [ 12 ]  [ 08 ]  [ 45 ]  [ 30 ] │
│   Days    Hrs    Min    Sec     │
│  📷 Watch live on Instagram     │
│                               │
│     [ foliage silhouette ]      │
└──────────────────────────────┘
```
- Live countdown, ticking every second, inside its own card — see card 2 below.
- **Two stacked cards, not one (2026-08-16, §48–§49):** **card 1** renders "A Note From Us"'s content — title, body, signature. Originally folded in here as a first step toward retiring that standalone page, content temporarily duplicated on both (§48); the standalone page was actually deleted 2026-08-17 (§53), so this card is now the *only* place that content renders — no more duplication. **Card 2** — a new "Akad Nikah"/"Akad Ceremony" heading, then the date + time line, then the countdown digits (no separate title above them anymore — the heading serves that role), then the live-stream note + Instagram link last (§49). No Qur'an quote here anymore — that moved to the new Prolog page (Page 2) back in §41.
- **Hanging bouquet sprays** (`wedding/bouquet_left.png`/`bouquet_right.png`) frame the section's top-left/top-right corners with a slow rotational sway, replacing the shared arch halves that used to sit here (§47) — `arch_flower1_left/right.png` sat unused after that swap and was deleted from disk 2026-08-17 (§52).
- **Foliage silhouette along the bottom edge** (`wedding/leaves1_bottom.png`, full width) — behind the bouquets/card, subtle framing. A matching top-edge silhouette (`leaves1TopImage`) existed briefly (§46) but is currently disabled — commented out in code, not deleted, as of 2026-08-17 (CONTENT_DESIGN.md §50); nothing renders in that slot right now.
- **Moved (2026-08-15):** previously Page 3, right after Main Announcement — then Page 4, after Couple Profile (the announcement, then who the couple is, then the countdown to the day). **Now Page 5**, after Prolog was inserted (2026-08-16, §41) — position only, no further content/asset change.
- **Assets:** `location option 2/`'s own `background_location2.png`/`silhouette_location2.png`/`arch_location2.png` are all superseded — the shared fixed `BackgroundLayer` (§39) shows through instead, and the framing arch was replaced twice over (retired for the shared `components/arch_flower1_*` halves in Phase 8, §30, since deleted from disk — §52; those in turn swapped for the bouquet sprays above, §47). Current art: `wedding/bouquet_left.png`/`bouquet_right.png` (corner sprays) + `wedding/leaves1_bottom.png` (bottom silhouette). Countdown/note content sits in translucent white cards for contrast against the shared background.

### Page 6 — Love Story  `id: loveStory` · default: **off**

```
┌──────────────────────────────┐
│                               │
│          Our Story             │
│                               │
│   ● 2021 — How We Met            │
│   │  short paragraph...        │
│   ●                           │
│   │  2023 — The Proposal       │
│   │  short paragraph...        │
│   ●                           │
│                               │
└──────────────────────────────┘
```
- Vertical timeline, a few milestones with a short line of text each.
- Shipping on with placeholder milestones for now — toggle off via `loveStory` if you'd rather leave it out (CONTENT_DESIGN.md §1).
- **Assets:** ~~background reused from `outro/Background_outro.png` (no dedicated folder of its own — see CONTENT_DESIGN.md §12)~~ **superseded (2026-08-16, CONTENT_DESIGN.md §39): background image removed, shared fixed `BackgroundLayer` shows through instead** + small floral accents (`lily.png`, `orchid.png`, `amaranthus.png`) beside milestones — moved from `img/components/` to `img/wedding/` 2026-08-17 (CONTENT_DESIGN.md §53), path only, no visual change. Timeline sits in a translucent white panel for contrast.

### Page 7 — Gallery  `id: gallery` · default: **off**

```
┌──────────────────────────────┐
│                               │
│         Our Moments            │
│                               │
│   [img] [img] [img]            │
│   [img] [img] [img]            │
│                               │
└──────────────────────────────┘
```
- Responsive photo grid; click/tap opens a lightbox (arrow + swipe + keyboard navigation).
- Lazy-loaded images.
- **Assets:** ~~background reused from `cover/background_cover.png` (no dedicated folder of its own)~~ **superseded (2026-08-16, CONTENT_DESIGN.md §39): background image removed, shared fixed `BackgroundLayer` shows through instead** + `wedding/mainbouquet.png` (moved from `img/components/` 2026-08-17, CONTENT_DESIGN.md §53) as a corner accent around the grid (decoration only — doesn't sit on top of the photos themselves, doesn't affect lightbox behavior). Grid sits in a translucent white panel for contrast.

### Page 8 — Guest Wishes / Comments  `id: wishes` · default: **on**

```
┌──────────────────────────────┐
│         Send Your Wishes       │
│                               │
│   Name:    [______________]    │
│   Message: [_____________]     │
│            [_____________]     │
│   [ ] (hidden honeypot field)  │
│            [ Send ]            │
│                               │
│   ────────────────────────     │
│   Sarah — "Congratulations!.." │
│   2 minutes ago                │
│                               │
│   Budi — "Selamat menempuh..." │
│   1 hour ago                   │
│                               │
└──────────────────────────────┘
```
- Form: name + message, submits instantly, appears at top of the list live (no page reload).
- If the message hits the denylist, an inline error shows instead of the wish appearing (CONTENT_DESIGN.md §6).
- List is public, newest first, reads from the database (persists across reloads).
- **The card is capped to one viewport** (`max-h-[calc(100dvh-4rem)]`) no matter how many wishes accumulate — the page never grows past one screen here. The comment list scrolls internally (`overflow-y-auto`) instead, with a soft fade at its bottom hinting more wishes below (the site hides scrollbars globally, CONTENT_DESIGN.md §18, so the fade is the scrollability cue). **Revised (2026-08-15):** previously the list grew the page unbounded; it was capped + given an internal scroller so every section stays one viewport tall (which the §24 page-turn scroll relies on).
- **Assets:** ~~background reused from `opening/background_opening.png` (no dedicated folder of its own)~~ **superseded (2026-08-16, CONTENT_DESIGN.md §39): background image removed, shared fixed `BackgroundLayer` shows through instead**. **Correction (2026-08-17):** this bullet previously claimed a `components/lace banner.png` side accent that doesn't actually match the code — `Wishes.tsx` renders `leaves1TopImage` (`wedding/leaves1_top.png`) as a top-edge foliage silhouette instead (added §46, swapped to this specific asset §50); `lace-banner.png` was never referenced anywhere and has since been deleted (§52). Form + list sit in a translucent white panel for contrast; input fields keep an opaque white background regardless.

### Page 9 — Digital Gift  `id: gift` · default: **off**

```
┌──────────────────────────────┐
│         Wedding Gift            │
│                               │
│   Bank X — a/n Raka Pratama    │
│   1234 5678 9099   [ Copy ]     │
│                               │
│   [ QR code image, optional ]   │
│                               │
└──────────────────────────────┘
```
- Account name/number with a copy-to-clipboard button.
- Optional QR image for e-wallet transfer.
- Purely informational — no payment flow on-site.
- **Assets:** ~~background reused from `bride & groom/Background_bride&groom.png` (no dedicated folder of its own)~~ **superseded (2026-08-16, CONTENT_DESIGN.md §39): background image removed, shared fixed `BackgroundLayer` shows through instead** + a small `wedding/orchid.png` accent near the heading (moved from `img/components/` 2026-08-17, CONTENT_DESIGN.md §53). The QR image itself stays the hand-drawn placeholder (`public/gift/qr.svg`) — no real QR code asset exists yet. Content sits in a translucent white panel for contrast.

### Page 10 — Closing  `id: closing` (renamed from `share`, 2026-08-17, CONTENT_DESIGN.md §51) · default: **on**

```
┌──────────────────────────────┐
│ [flower1]                      │
│                               │
│   Thank you for being part      │
│   of our story.                 │
│                               │
│      "Ara & Zharfan" (art)      │
│                               │
│                       [flower2] │
│   ┌──────────────────────┐    │
│   │   couple photo,        │    │
│   │   pinned to the very   │    │
│   │[flower3] bottom edge   │    │
│   └──────────────────────┘    │
└──────────────────────────────┘
```
- Short closing line.
- **Revised (Phase 8, CONTENT_DESIGN.md §27–§28):** rebuilt from the `ig_v2/` layers directly rather than the flattened composite used in the initial Phase 7 version. Portrait pinned to the section's true bottom edge as a footer (not floating mid-section with dead space beneath it, which is what it looked like at first), enlarged. Closing signature is the hand-lettered `name/ara-zharfan.png` art (CONTENT_DESIGN.md §14) instead of script-font text. **Superseded (2026-08-17, §51) — see below.**
- **No action button anymore.** "Watch Live on Instagram" (Phase 8's replacement for Copy Link/Share-to-WhatsApp) is itself gone as of 2026-08-17 — this page is just the closing line, signature, and portrait footer now. The Instagram live-stream pointer still exists on the site, just relocated to Countdown's card (Page 5) as a plain note + handle link instead of a button here (CONTENT_DESIGN.md §48).
- **Frame art revised (2026-08-17, CONTENT_DESIGN.md §51):** the single full-bleed `ig-v2-frame.png` (stretched/cropped to cover the section) is replaced with three separate corner floral pieces sized to their own art — `flower1TopLeftImage` (top-left), `flower2BottomRightImage` (bottom-right), `flower3BottomLeftImage` (bottom-left, tucked near the portrait footer). `ig-v2-frame.png` deleted from disk.
- Still doubles as the page's Open Graph preview target — when the link is pasted into a chat, the OG title/description/image (not this section specifically) is what shows.
- **Assets:** `wedding/flower1_top_left.png`/`flower2_bottom_right.png`/`flower3_bottom_left.png` (corner accents, §51) + `wedding/couple1.png` (portrait footer — `content.couple.photo`, not the old `ig_v2-3.png`/`ig-v2-portrait.png`, which had already fallen out of use before 2026-08-17 without a doc update at the time) + `name/ara-zharfan.png` (signature). `ig_v2-1.png`/`ig_v2-2.png` (background/frame) are both gone now — background superseded by the shared fixed `BackgroundLayer` back on 2026-08-16 (CONTENT_DESIGN.md §39), frame replaced by the three flower pieces above. `ig-v2-background.png`/`ig-v2-portrait.png` are still on disk, unreferenced, not cleaned up (same policy as other superseded assets). (`closing/`'s own `Background_closing.png`/`doodle_closing.png`/`frame_closing.png` are no longer used here either. `outro/` was originally a spare unused folder — now reused for Love Story's background instead, see Page 6 — Love Story's own background layer has since also been superseded, per above. `location option 2/` is likewise no longer unused — now Countdown's background, see Page 5 — also since superseded.)

### Floating, on every page (not a page itself)  `musicEnabled` flag · default: **on**

```
                              ┌────┐
                              │ ♪  │  <- mute/unmute toggle,
                              └────┘     bottom corner, persists
                                          across scroll
```
- Background music toggle, always visible, doesn't scroll away.
- **Revised (Phase 8, CONTENT_DESIGN.md §28):** icon changed from mismatched 🎵/🔇 emoji to a uniform inline-SVG `MusicIcon` — same note glyph in both states, muted just adds a diagonal slash, so playing/muted read as one consistent icon rather than two unrelated symbols.

### Floating, on every page (not a page itself)  `languageToggleEnabled` flag · default: **on**

```
┌────┐
│ 🇮🇩 │  <- language toggle, opposite
└────┘     corner from music, tap to
           switch id ⇄ en
```
- Guest-facing id/en switch (Phase 10, 2026-08-15) — previously the site's language was a fixed build-time choice with no in-page control. Bottom-left, opposite corner from the music toggle so the two floating controls don't collide. Full spec: CONTENT_DESIGN.md §34.
- Shows the flag of the *current* language (🇮🇩 Indonesian / 🇬🇧 English) — tap switches to the other, and every bilingual string on the page updates immediately, including on sections not yet scrolled into view.
- **Usable from the Cover overlay, before "Tap to Open"** — deliberately layered above Cover's pre-tap overlay, not just the revealed page underneath it.
- Both floating buttons' background changed from 80% to 50% opacity in the same pass (a more see-through circle over the artwork behind them).

---

## Resolved

- Page order confirmed as above.
- Defaults revised (2026-08-10): Love Story, Gallery, and Digital Gift now default **off**; Cover, Countdown, Couple Profile, A Note From Us, Guest Wishes, and Share/Closing stay **on**. Each page's heading above shows its actual current default — anything can be toggled either way per CONTENT_DESIGN.md §2. **Superseded 2026-08-15:** Couple Profile also switched to **off** by default — see the Phase 10/11 entry below. **Superseded again 2026-08-16:** A Note From Us also switched to **off** by default — see the entry below. **Superseded again 2026-08-17: A Note From Us isn't a togglable page anymore at all — deleted outright, see the entry below.**
- Every page confirmed to have a full background image from the asset pack (some reused across pages without a dedicated composition) — see CONTENT_DESIGN.md §12 for the complete assignment.
- Phase 7 (2026-08-10): Couple Profile is now two pages (3a groom, 3b bride) instead of one combined layout; couple's names render as hand-lettered signature art (`name/`) instead of script-font text on Cover, Couple Profile, and Share; Share's hero illustration switched to the `ig_v2/` story card plus a new Instagram live link; site-wide fixes — forced light-only theme (contrast bug), scroll lock + hidden scrollbar pre-open, Cover's open button gated on asset loading, and a centered max-width column on wide viewports. See CONTENT_DESIGN.md §13–§20.
- Phase 9 (2026-08-15): Cover's wedding date removed from both the pre-tap overlay and the revealed hero (the date lives only on Countdown now); wheel scrolling converted to a page-turn gesture — one section per 35%-of-viewport wheel input, keyboard/touch still use the CSS snap (CONTENT_DESIGN.md §24); Guest Wishes card capped to one viewport with the comment list as an internal scroller. See CONTENT_DESIGN.md §24 and the Page 1/Page 7 entries above.
- Phase 10 (2026-08-15): new Main Announcement page inserted right after Cover (now Page 2 — Bismillah + formal announcement text in a `letter_frame.png` card); Cover's own revealed in-flow hero section removed, so "Tap to Open" now lands directly on Main Announcement instead of a second, near-duplicate greeting screen first; A Note From Us moved from Page 4 to Page 7 (after Gallery) to make room; every page from Countdown onward renumbered accordingly. Language toggle (id/en) made guest-facing — floating button, flag-emoji notation, usable from the Cover overlay before opening. See CONTENT_DESIGN.md §33–§35 and the Page 1/Page 2/Page 7/floating-controls entries above.
- Phase 10/11 (2026-08-15): Countdown and Couple Profile swapped — Countdown is now Page 4, Couple Profile Page 3 (announcement → couple intro → countdown). Page numbers in the entry headers above reflect this; Page 7's "Moved (Phase 10)" note and Page 4's "Moved (2026-08-15)" note record the history.
- Phase 10/11, later same day (2026-08-15): Couple Profile (Page 3) switched off by default — its bride/groom portraits now stand in Main Announcement's (Page 2) bottom-left/bottom-right corners instead, sized by screenshot review rather than pure geometry (groom deliberately taller than bride, both intentionally bleeding past the section's edge via `overflow-hidden` cropping). Page 2's date (`dateIntro` + date) moved out of its own bottom-anchored zone into the name block's normal flow, two lines below the groom's name. See CONTENT_DESIGN.md §37 and the Page 2/Page 3 entries above.
- 2026-08-16: new Prolog page (`id: prolog`) inserted right after Cover — two birds flying together + the QS. Ar-Rum: 21 quote, moved off Countdown (which is timer-only again). Every page from Main Announcement onward renumbered up by one (Main Announcement 2→3, Couple Profile 3→4, Countdown 4→5, Love Story 5→6, Gallery 6→7, A Note From Us 7→8, Guest Wishes 8→9, Digital Gift 9→10, Share/Closing 10→11). See CONTENT_DESIGN.md §41 and the Page 2/Page 5 entries above.
- 2026-08-16, later the same day: Countdown's corner arches replaced with hanging bouquet sprays (§47); A Note From Us's content (title/body/signature/live-stream note) folded into Countdown as a first step toward retiring that standalone page, which is now switched **off** by default (§48); Countdown's own card restructured into two — note content, then a new "Akad Nikah" heading → date/time → countdown digits → live-stream note (§49). See the Page 5/Page 8 entries above.
- 2026-08-17: further Countdown/Wishes asset tweaks made directly in the working tree — Countdown's top foliage silhouette disabled (commented out, not deleted), its bottom silhouette and Wishes' top silhouette swapped to a new `leaves1_bottom.png`/`leaves1TopImage` pairing; `leaves2TopImage`/`leaves2BottomImage` now unused in any rendered JSX; the unused flower placeholder images renamed (`flower1/2/3TopLeft/BottomRight/BottomLeftImage`, no layout/behavior change — still preload-only); `layout.tsx`'s page `<title>` dropped its "Pernikahan:" prefix, now just the couple's nicknames. See CONTENT_DESIGN.md §50 and the Page 5 entry above.
- 2026-08-17, later the same day: Page 11 renamed Share → **Closing**, `id: share` → `id: closing` end-to-end (component file, section config, labels key). Its old single full-bleed `ig-v2-frame.png` swapped for the three flower corner images from earlier today's entry, now actually rendered instead of sitting preload-only; dead `labels.closing.watchLive` label and the "Watch Live on Instagram" button it powered are both gone (that button had already stopped rendering at some earlier, undocumented point — Countdown's live-stream note/link, §48, is what visitors actually see now). See CONTENT_DESIGN.md §51 and the Page 11 entry above (now Page 10 — see the next two entries).
- 2026-08-17, later still: `InstagramIcon.tsx` deleted (unused after the previous entry). Eight `web/public/img/` directories checked file-by-file for live references and trimmed or removed entirely where dead — `bride-groom/`, `closing/`'s remaining two files, `countdown/`, `lovestory/`, `opening/` deleted outright; `components/`, `cover/`, `note/` had some files deleted, some kept (the kept ones — `cover/background_cover.png`, `note/frame.png` — only survived because `Test2.tsx`/`NoteFromUs.tsx` still referenced them at that point). See CONTENT_DESIGN.md §52.
- 2026-08-17, later still: **A Note From Us (formerly Page 8) deleted outright** — not just disabled, the page/component is gone; its content lives on via Countdown's card only (§48). `Test1`/`Test2` (dev-only blank pages, never part of this numbered list) deleted too, along with their now-orphaned exclusive assets `cover/background_cover.png`/`note/frame.png` — both folders now fully empty and removed. **Every page from the old Page 9 onward renumbered down by one:** Guest Wishes 9→8, Digital Gift 10→9, Closing 11→10. `components/`'s last four files (`lily`/`orchid`/`amaranthus`/`mainbouquet`) moved to `wedding/`, `components/` folder now fully gone. See CONTENT_DESIGN.md §53 and the Page 5/Page 6/Page 7/Page 8/Page 9 entries above.
