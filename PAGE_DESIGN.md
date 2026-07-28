# Page-by-Page Design
## Wedding Announcement Website

**Status:** Draft — for review
**Last updated:** 2026-07-28

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
│      28 . 07 . 2026           │
│                               │
│        [ Tap to Open ]        │
│                               │
└──────────────────────────────┘
```
- Couple's names (large, primary visual focus).
- Personalized greeting placeholder (CONTENT_DESIGN.md §4).
- Wedding date.
- "Tap to open" interaction — this is also the user gesture that unlocks the background music autoplay (browsers block autoplay without one).

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

### Page 3 — Couple Profile  `id: coupleProfile` · default: **on**

```
┌──────────────────────────────┐
│                               │
│   [ photo ]         [ photo ]   │
│                               │
│   Raka Pratama    &   Dinda Ayu │
│   Putra dari         Putri dari  │
│   Bpk. A & Ibu B     Bpk. C & Ibu D│
│                               │
└──────────────────────────────┘
```
- Photo + name + short bio/parents' names for each side (common convention).
- Stacks vertically on mobile.

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

### Page 5 — Love Story  `id: loveStory` · default: **on**

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

### Page 6 — Gallery  `id: gallery` · default: **on**

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

### Page 8 — Digital Gift  `id: gift` · default: **on**

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

### Page 9 — Share / Closing  `id: share` · default: **on**

```
┌──────────────────────────────┐
│                               │
│   Thank you for being part      │
│   of our story.                 │
│                               │
│   [ Copy Link ]  [ Share to WA ] │
│                               │
│         Raka & Dinda            │
│         28 . 07 . 2026          │
│                               │
└──────────────────────────────┘
```
- Short closing line.
- Copy-link / share-to-WhatsApp buttons.
- Doubles as the page's Open Graph preview target — when this whole link is pasted into a chat, the OG title/description/image (not this section specifically) is what shows.

### Floating, on every page (not a page itself)  `musicEnabled` flag · default: **on**

```
                              ┌────┐
                              │ ♪  │  <- mute/unmute toggle,
                              └────┘     bottom corner, persists
                                          across scroll
```
- Background music toggle, always visible, doesn't scroll away.

---

## Resolved

- Page order confirmed as above.
- All sections shipping **on** for now (including Love Story) — anything can be toggled off later per CONTENT_DESIGN.md §2.
