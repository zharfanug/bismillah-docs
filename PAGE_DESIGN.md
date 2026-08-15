# Page-by-Page Design
## Wedding Announcement Website

**Status:** Built (Phase 7 complete) — this doc now describes the shipped state, not a plan
**Last updated:** 2026-08-15 (Phase 10/11: page order changed again — Countdown and Couple Profile swapped, Couple Profile is now Page 3 and Countdown Page 4 (announcement, then who the couple is, then the countdown); new Main Announcement page inserted right after Cover — see Page 2; Cover's own revealed in-flow "second screen" removed entirely, so tapping "Buka" now lands directly on Main Announcement instead of a near-duplicate greeting first — see Page 1; A Note From Us moved to Page 7 (after Gallery); language toggle made guest-facing — see the floating-controls entries at the bottom; Countdown's date line replaced with a QS. Ar-Rum: 21 quote (meaning only) — see Page 4; Main Announcement's layout later switched to three independently-positioned blocks — see Page 2. Phase 9: Cover's wedding date removed from the pre-tap overlay; wheel scrolling converted to a 35%-threshold page-turn gesture — see CONTENT_DESIGN.md §24; Guest Wishes card capped to one viewport with an internal comment scroller. Each change is flagged in its page's entry below)

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
│      Dear {Name},              │  <- from ?to= param, falls back to
│                               │     "Dear Family & Friends,"
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
- **Assets (pre-tap overlay only, now):** `cover/background_cover.png` (gradient), `cover/landscape_cover.png` (watercolor scene), `cover/birds&banner_cover.png` (names/date banner, idle bird/banner motion), `name/ara-zharfan.png` (couple's names). The `[ background photo ]` placeholder in the wireframe above is these layered together.
- **Typography:** the greeting text block sits in a translucent white panel for contrast against the artwork.

### Page 2 — Main Announcement  `id: mainAnnouncement` · default: **on** (new, Phase 10, 2026-08-15)

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
└──────────────────────────────┘
```

- New section, inserted right after Cover — a formal Bismillah + wedding announcement, distinct from Cover's brief greeting and Couple Profile's per-person bios. Full spec: CONTENT_DESIGN.md §33.
- **Container is the content's own background**, not a decorative overlay — `components/letter_frame.png` is an opaque paper-card graphic (checked with ImageMagick: fully opaque cream center, transparent only at the rounded corners), unlike every other page's `bg-white/75` panel over a separate full-bleed background image.
- Bismillah shown in Arabic script (right-to-left), same text regardless of the site's language toggle — not translated. Uses a dedicated Arabic font (Noto Naskh Arabic) since none of the site's other fonts cover Arabic glyphs.
- Bride and groom each get their nickname signature art (reused from Couple Profile/Cover), their full legal name in a handwriting-style script font colored light blue (`#a1c1da`), and their parents line in small plain text (no handwriting, smaller than the name) beneath.
- A large handwriting-style "&" divides the two — rendered as styled text, not an image (no standalone ampersand asset exists in the source pack; checked before deciding this).
- Closing line ("yang akan dilaksanakan pada" / "which will take place on") followed by the event date in the same handwriting script, colored green (`#adcd9f`).
- **Layout revised (follow-up, 2026-08-15):** the Bismillah/intro block, the bride/&/groom block, and the date-intro/date block are each independently positioned — top and bottom blocks sit exactly 13.20% in from the frame's top/bottom edge (matching where the frame art's corner flowers end), the middle block is vertically centered in the space between them. Full reasoning: CONTENT_DESIGN.md §33.
- **Assets:** `cover/background_cover.png` (reused, no dedicated background of its own) + `components/letter_frame.png` (the card/container) + `name/ara.png` / `name/zharfan.png` (nickname signatures, reused from Couple Profile).

### Page 3 — Couple Profile  `id: coupleProfile` · default: **on**

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
- Same `bride & groom/Background_*` background reused for both (CONTENT_DESIGN.md §12); the paired `frame_bride&groom.png` isn't reused here (designed for two people side by side, would distort scaled to one — CONTENT_DESIGN.md §15).
- **Assets:** `bride & groom/bride_bride&groom.png` / `groom_bride&groom.png` (full illustrated portraits) + `Background_bride&groom.png`, plus `name/zharfan.png` and `name/ara.png`. Fade/slide-in per section as it scrolls into view.

### Page 4 — Countdown  `id: countdown` · default: **on**

```
┌──────────────────────────────┐
│                               │
│   Counting down to our day     │
│                               │
│   [ 12 ]  [ 08 ]  [ 45 ]  [ 30 ] │
│   Days    Hrs    Min    Sec     │
│                               │
│  "...He placed between you     │
│   affection and mercy..."      │
│      QS. Ar-Rum: 21            │
│                               │
└──────────────────────────────┘
```
- Live countdown, ticking every second.
- **Revised (Phase 10, 2026-08-15):** the static date line was removed (the date already shows on Main Announcement, Page 2) — replaced with a short Qur'an quote, QS. Ar-Rum: 21, about spouses finding tranquility, affection, and mercy together. Meaning only (standard published translation, no Arabic script) plus the citation beneath it, in the translucent card below the timer. Full spec: CONTENT_DESIGN.md §36.
- **Moved (2026-08-15):** previously Page 3, right after Main Announcement — now Page 4, after Couple Profile (the announcement, then who the couple is, then the countdown to the day). Position only, no content/asset change.
- **Assets:** `location option 2/` — `background_location2.png` + `silhouette_location2.png` (treeline overlay). Assigned here specifically (this folder was originally an unused spare — see CONTENT_DESIGN.md §12). Countdown numbers sit in a translucent white card for contrast against the artwork. **Revised (Phase 8):** the framing arch is now the same shared `components/arch_flower1_left.png`/`arch_flower1_right.png` halves Cover uses (retiring this folder's own `arch_location2.png`), same off-screen bleed + slow sway treatment — CONTENT_DESIGN.md §30.

### Page 5 — Love Story  `id: loveStory` · default: **off**

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
- **Assets:** background reused from `outro/Background_outro.png` (no dedicated folder of its own — see CONTENT_DESIGN.md §12) + small floral accents from `components/` (`lily.png`, `orchid.png`, `amaranthus.png`) beside milestones. Timeline sits in a translucent white panel for contrast.

### Page 6 — Gallery  `id: gallery` · default: **off**

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
- **Assets:** background reused from `cover/background_cover.png` (no dedicated folder of its own) + `components/mainbouquet.png` as a corner accent around the grid (decoration only — doesn't sit on top of the photos themselves, doesn't affect lightbox behavior). Grid sits in a translucent white panel for contrast.

### Page 7 — "A Note From Us"  `id: noteFromUs` · default: **on**

```
┌──────────────────────────────┐
│                               │
│        A Note From Us          │
│                               │
│  "Our celebration was           │
│   intentionally small — just    │
│   the two of us and our         │
│   closest family. But we've     │
│   felt so much love from every  │
│   prayer, kind thought, and     │
│   warm wish you've sent our     │
│   way. Thank you for being      │
│   part of this new chapter      │
│   with us."                     │
│                               │
│         — Raka & Dinda          │
│                               │
└──────────────────────────────┘
```
- Replaces the usual venue/location block (CONTENT_DESIGN.md §5) — no address, map, or "you're invited" language.
- Draft 1 shown; drafts 2/3 or your own edit are drop-in swaps in the content file.
- Small signature line under the note is a nice touch, optional.
- **Moved (Phase 10, 2026-08-15):** previously Page 4, right after Couple Profile — now Page 7, after Gallery, to make room for the new Main Announcement section right after Cover. No content or asset change, position only.
- **Assets:** `location option 1/` — `Background_location1.png` + `frame_location1.png` (floral corner frame) reused purely for its look (repurposing explained in CONTENT_DESIGN.md §12; nothing address/venue-related is added despite the folder's name). Fade-in animation. Text sits in a translucent white panel for contrast.

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
- **Assets:** background reused from `opening/background_opening.png` (no dedicated folder of its own) + `components/lace banner.png` as a small side accent — decoration only, doesn't sit over the form fields or wish list (would hurt usability/contrast if it did). Form + list sit in a translucent white panel for contrast; input fields keep an opaque white background regardless.

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
- **Assets:** background reused from `bride & groom/Background_bride&groom.png` (no dedicated folder of its own) + a small `components/orchid.png` accent near the heading. The QR image itself stays the hand-drawn placeholder (`public/gift/qr.svg`) — no real QR code asset exists yet. Content sits in a translucent white panel for contrast.

### Page 10 — Share / Closing  `id: share` · default: **on**

```
┌──────────────────────────────┐
│   [ ig_v2-1 background ]       │
│   [ ig_v2-2 floral frame ]     │
│                               │
│   Thank you for being part      │
│   of our story.                 │
│                               │
│      "Ara & Zharfan" (art)      │
│                               │
│   [ 📷 Watch Live on Instagram ] │
│                               │
│                               │
│   ┌──────────────────────┐    │
│   │   ig_v2-3 portrait,    │    │
│   │   pinned to the very   │    │
│   │   bottom edge (footer) │    │
│   └──────────────────────┘    │
└──────────────────────────────┘
```
- Short closing line.
- **Revised (Phase 8, CONTENT_DESIGN.md §27–§28):** rebuilt from the `ig_v2/` layers directly rather than the flattened composite used in the initial Phase 7 version — background + frame both full-bleed behind the text panel. Portrait pinned to the section's true bottom edge as a footer (not floating mid-section with dead space beneath it, which is what it looked like at first), enlarged. Closing signature is the hand-lettered `name/ara-zharfan.png` art (CONTENT_DESIGN.md §14) instead of script-font text.
- **Just one action now:** "Watch Live on Instagram" (with an inline Instagram glyph icon) to `instagram.com/hauradhiyaa`. Copy Link / Share-to-WhatsApp buttons were removed (§28) — no longer part of this page.
- Still doubles as the page's Open Graph preview target — when the link is pasted into a chat, the OG title/description/image (not this section specifically) is what shows.
- **Assets:** `ig_v2/ig_v2-1.png` (background) + `ig_v2-2.png` (frame) + `ig_v2-3.png` (portrait, bottom-pinned footer) + `name/ara-zharfan.png`. (`closing/`'s own `Background_closing.png`/`doodle_closing.png`/`frame_closing.png` are no longer used here. `outro/` was originally a spare unused folder — now reused for Love Story's background instead, see Page 5. `location option 2/` is likewise no longer unused — now Countdown's background, see Page 3.)

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
- Defaults revised (2026-08-10): Love Story, Gallery, and Digital Gift now default **off**; Cover, Countdown, Couple Profile, A Note From Us, Guest Wishes, and Share/Closing stay **on**. Each page's heading above shows its actual current default — anything can be toggled either way per CONTENT_DESIGN.md §2.
- Every page confirmed to have a full background image from the asset pack (some reused across pages without a dedicated composition) — see CONTENT_DESIGN.md §12 for the complete assignment.
- Phase 7 (2026-08-10): Couple Profile is now two pages (3a groom, 3b bride) instead of one combined layout; couple's names render as hand-lettered signature art (`name/`) instead of script-font text on Cover, Couple Profile, and Share; Share's hero illustration switched to the `ig_v2/` story card plus a new Instagram live link; site-wide fixes — forced light-only theme (contrast bug), scroll lock + hidden scrollbar pre-open, Cover's open button gated on asset loading, and a centered max-width column on wide viewports. See CONTENT_DESIGN.md §13–§20.
- Phase 9 (2026-08-15): Cover's wedding date removed from both the pre-tap overlay and the revealed hero (the date lives only on Countdown now); wheel scrolling converted to a page-turn gesture — one section per 35%-of-viewport wheel input, keyboard/touch still use the CSS snap (CONTENT_DESIGN.md §24); Guest Wishes card capped to one viewport with the comment list as an internal scroller. See CONTENT_DESIGN.md §24 and the Page 1/Page 7 entries above.
- Phase 10 (2026-08-15): new Main Announcement page inserted right after Cover (now Page 2 — Bismillah + formal announcement text in a `letter_frame.png` card); Cover's own revealed in-flow hero section removed, so "Tap to Open" now lands directly on Main Announcement instead of a second, near-duplicate greeting screen first; A Note From Us moved from Page 4 to Page 7 (after Gallery) to make room; every page from Countdown onward renumbered accordingly. Language toggle (id/en) made guest-facing — floating button, flag-emoji notation, usable from the Cover overlay before opening. See CONTENT_DESIGN.md §33–§35 and the Page 1/Page 2/Page 7/floating-controls entries above.
- Phase 10/11 (2026-08-15): Countdown and Couple Profile swapped — Countdown is now Page 4, Couple Profile Page 3 (announcement → couple intro → countdown). Page numbers in the entry headers above reflect this; Page 7's "Moved (Phase 10)" note and Page 4's "Moved (2026-08-15)" note record the history.
