# Page-by-Page Design
## Wedding Announcement Website

**Status:** Built (Phase 7 complete) — this doc now describes the shipped state, not a plan
**Last updated:** 2026-08-15 (Phase 9: Cover's wedding date removed from the pre-tap overlay and the revealed hero; wheel scrolling converted to a 35%-threshold page-turn gesture — see CONTENT_DESIGN.md §24; Guest Wishes card capped to one viewport with an internal comment scroller — see Page 7; each change is flagged in its page's entry below)

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
- **No wedding date on the Cover** — the date was removed from both the pre-tap overlay and the revealed hero (2026-08-15); the date now appears only on the Countdown page, right beneath the timer.
- "Tap to open" interaction — only appears once the overlay's artwork has actually finished loading (a progress bar shows load progress until then, and a 30s auto-refresh recovers if it never finishes — CONTENT_DESIGN.md §19, §29); also the user gesture that unlocks the background music autoplay (browsers block autoplay without one).
- Page scroll is locked and the scrollbar hidden while this overlay is showing (CONTENT_DESIGN.md §18) — unlocked the moment "Tap to Open" is pressed.
- **Assets (pre-tap overlay):** `cover/background_cover.png` (gradient), `cover/landscape_cover.png` (watercolor scene), `cover/birds&banner_cover.png` (names/date banner, idle bird/banner motion), `name/ara-zharfan.png` (couple's names). The `[ background photo ]` placeholder in the wireframe above is these layered together.
- **Assets (revealed, in-flow, after tapping):** same content, different asset folder — `opening/background_opening.png` + `components/arch_flower1_left.png`/`arch_flower1_right.png` (the floral arch, split into two independently animated halves — moved from `opening/` to `components/` in Phase 8 since they're now shared with Countdown too, CONTENT_DESIGN.md §21/§30) + `name/ara-zharfan.png` again for the names. Deliberately distinct from the overlay above so "before" and "after" tapping feel different, not just an instant swap (CONTENT_DESIGN.md §12). The arch halves bleed 5% off-screen on their outer edges and sway slowly (rotating from the top, like a hanging branch) — CONTENT_DESIGN.md §30.
- **Typography:** both greeting text blocks sit in a translucent white panel for contrast against the artwork.

### Page 2 — Countdown  `id: countdown` · default: **on**

```
┌──────────────────────────────┐
│                               │
│   Counting down to our day     │
│                               │
│   [ 12 ]  [ 08 ]  [ 45 ]  [ 30 ] │
│   Days    Hrs    Min    Sec     │
│                               │
│      28 July 2026              │
│                               │
└──────────────────────────────┘
```
- Live countdown, ticking every second.
- Static date/time line beneath for anyone who doesn't want to do the math.
- **Assets:** `location option 2/` — `background_location2.png` + `silhouette_location2.png` (treeline overlay). Assigned here specifically (this folder was originally an unused spare — see CONTENT_DESIGN.md §12). Countdown numbers sit in a translucent white card for contrast against the artwork. **Revised (Phase 8):** the framing arch is now the same shared `components/arch_flower1_left.png`/`arch_flower1_right.png` halves Cover uses (retiring this folder's own `arch_location2.png`), same off-screen bleed + slow sway treatment — CONTENT_DESIGN.md §30.

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

### Page 4 — "A Note From Us"  `id: noteFromUs` · default: **on**

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
- **Assets:** `location option 1/` — `Background_location1.png` + `frame_location1.png` (floral corner frame) reused purely for its look (repurposing explained in CONTENT_DESIGN.md §12; nothing address/venue-related is added despite the folder's name). Fade-in animation. Text sits in a translucent white panel for contrast.

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

### Page 7 — Guest Wishes / Comments  `id: wishes` · default: **on**

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

### Page 8 — Digital Gift  `id: gift` · default: **off**

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

### Page 9 — Share / Closing  `id: share` · default: **on**

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
- **Assets:** `ig_v2/ig_v2-1.png` (background) + `ig_v2-2.png` (frame) + `ig_v2-3.png` (portrait, bottom-pinned footer) + `name/ara-zharfan.png`. (`closing/`'s own `Background_closing.png`/`doodle_closing.png`/`frame_closing.png` are no longer used here. `outro/` was originally a spare unused folder — now reused for Love Story's background instead, see Page 5. `location option 2/` is likewise no longer unused — now Countdown's background, see Page 2.)

### Floating, on every page (not a page itself)  `musicEnabled` flag · default: **on**

```
                              ┌────┐
                              │ ♪  │  <- mute/unmute toggle,
                              └────┘     bottom corner, persists
                                          across scroll
```
- Background music toggle, always visible, doesn't scroll away.
- **Revised (Phase 8, CONTENT_DESIGN.md §28):** icon changed from mismatched 🎵/🔇 emoji to a uniform inline-SVG `MusicIcon` — same note glyph in both states, muted just adds a diagonal slash, so playing/muted read as one consistent icon rather than two unrelated symbols.

---

## Resolved

- Page order confirmed as above.
- Defaults revised (2026-08-10): Love Story, Gallery, and Digital Gift now default **off**; Cover, Countdown, Couple Profile, A Note From Us, Guest Wishes, and Share/Closing stay **on**. Each page's heading above shows its actual current default — anything can be toggled either way per CONTENT_DESIGN.md §2.
- Every page confirmed to have a full background image from the asset pack (some reused across pages without a dedicated composition) — see CONTENT_DESIGN.md §12 for the complete assignment.
- Phase 7 (2026-08-10): Couple Profile is now two pages (3a groom, 3b bride) instead of one combined layout; couple's names render as hand-lettered signature art (`name/`) instead of script-font text on Cover, Couple Profile, and Share; Share's hero illustration switched to the `ig_v2/` story card plus a new Instagram live link; site-wide fixes — forced light-only theme (contrast bug), scroll lock + hidden scrollbar pre-open, Cover's open button gated on asset loading, and a centered max-width column on wide viewports. See CONTENT_DESIGN.md §13–§20.
- Phase 9 (2026-08-15): Cover's wedding date removed from both the pre-tap overlay and the revealed hero (the date lives only on Countdown now); wheel scrolling converted to a page-turn gesture — one section per 35%-of-viewport wheel input, keyboard/touch still use the CSS snap (CONTENT_DESIGN.md §24); Guest Wishes card capped to one viewport with the comment list as an internal scroller. See CONTENT_DESIGN.md §24 and the Page 1/Page 7 entries above.
