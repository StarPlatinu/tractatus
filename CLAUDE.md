# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-page marketing website for **Tractatus** — a bar in Thao Dien, Ho Chi Minh City. The entire site is one static file: [index.html](index.html). There is no build step, no package manager, no backend.

## Running it

Open [index.html](index.html) directly in a browser. For live reload during edits, any static server works, e.g.:

```sh
python3 -m http.server 8000      # then visit http://localhost:8000
```

## Architecture

The site is intentionally a **single self-contained HTML file** (~1000 lines). All styles, scripts, and content live in `index.html`. When editing, preserve this constraint unless the user explicitly asks to split files.

Three things are wired together inside the file and are easy to break if edited in isolation:

1. **Tailwind config (inline `<script>` near the top)** defines the custom palette (`ink`, `coal`, `burgundy`, `wine`, `gold`, `goldsoft`, `cream`, `mist`) and font families (`serif` → Cormorant Garamond, `sans` → Inter). New utility classes like `bg-burgundy` or `font-serif` only work because of this config — Tailwind is loaded via CDN (`cdn.tailwindcss.com`) which reads the inline config at runtime. Adding a new color requires extending this block, not just using arbitrary hex values.

2. **Custom CSS (`<style>` block)** owns things Tailwind can't express cleanly: the hero `linear-gradient` overlay on the Unsplash background, the dotted-leader menu rows (`.menu-row` + `.leader` grid trick), the `.reveal` fade-in animation, the gold `.divider` rules, the `.grain` texture overlay, and the gold pulse on the floating CTA. The reveal animation depends on the JS observer below — adding a new section that should fade in needs **both** a `.reveal` class and being a child the observer can see.

3. **JS (`<script>` at the bottom)** runs six things: mobile menu open/close, navbar background-on-scroll, scroll-progress bar width update, IntersectionObserver for `.reveal` elements (with a small stagger when several enter together), the live "Open now" pip in the nav (`updateOpenStatus` recomputes every minute against `Asia/Ho_Chi_Minh` time and the hardcoded weekly schedule), and the reservation handler — see "Reservation flow" below.

The "Open now" schedule is duplicated: it lives in `updateOpenStatus()` **and** in the Visit section's hours list. If hours change, update both.

## Section structure

Sections in order, each anchored by `id` and linked from the nav: `#home` (hero) → `#about` → quote banner → `#menu` → `#events` → `#gallery` → `#visit` (Google Maps embed of `13 Đặng Tiến Đông, An Khánh, Thao Dien`) → `#contact` (reservation form) → footer. The floating "Book a Table" button and nav both link to `#contact`.

## Images

All photography is local in [images/](images/) — no CDN fallbacks. Eight files, each with a specific role:

| File | Where it appears |
|---|---|
| `exterior.jpg` | Hero background **and** gallery feature tile (the iconic Lambrosso + neon street shot) |
| `bar-interior.jpg` | About section image + gallery |
| `bottles.jpg` | Parallax background of the quote banner + gallery |
| `bartender.jpg` | Gallery |
| `chess.jpg` | Gallery |
| `piano.jpg` | Gallery |
| `courtyard.jpg` | Gallery |
| `books.jpg` | Gallery full-width bottom row (menus on table, red ambient light) |

`exterior.jpg` and `bottles.jpg` are referenced from CSS `background-image: url(...)` rules in the `<style>` block (hero-bg, quote-bg). The other five are `<img src>` references in the about + gallery sections. If a file is renamed or replaced, update both the file on disk **and** the references in the HTML.

## Reservation flow → Facebook Messenger

There is no backend. The form (`#reservationForm`) uses `handleReservation` to:
1. Format the booking fields into a clean text message
2. Copy that message to clipboard via `navigator.clipboard.writeText`
3. Open `https://m.me/<page>` in a new tab — the page username is read from the form's `data-fb-page` attribute

The Tractatus FB page username is set in **two places** that must stay in sync if it changes:
- `data-fb-page="..."` on the `<form id="reservationForm">` element
- The hard-coded `m.me/...` and `facebook.com/...` URLs in the contact cards + footer

The current placeholder is `tractatus.thaodien` — replace globally if the real page handle is different.

## Known placeholders to replace before launch

- Phone number `+84 (0)28 0000 0000` in the Visit section
- Facebook page username `tractatus.thaodien` (see Reservation flow above)
- Instagram handle `tractatus.bar`

## External dependencies (all CDN, no install)

- Tailwind CSS — `cdn.tailwindcss.com`
- Google Fonts — Cormorant Garamond + Inter
- Font Awesome 6.5.1 — `cdnjs.cloudflare.com`
- Google Maps embed — uses the public `/maps?q=...&output=embed` form (no API key)

---

## Behavioral Guidelines

**Think before coding.** State assumptions explicitly. If something is unclear, ask before implementing.

**Minimum viable change.** No features beyond what was asked. No abstractions for single-use code. No error handling for impossible scenarios.

**Surgical edits.** Don't improve adjacent code. Match existing style. Remove only what your own change made unused.

**Verify.** For multi-step tasks, state a plan first:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
```

