# Handoff: Kol Torah — visual design system (direction 6b)

## Overview

Kol Torah is a Hebrew-first website indexing thousands of recorded Torah lessons. Lessons are
transcribed and processed into **minute-level summaries**, source references (verses, gemara
pages), and split Q&A units. Users find material two ways — semantic **search** over segments,
and a **grounded agent** that answers only by pointing at what a rabbi actually said, with a
timestamped link. **Transcripts are never shown to users** (transcription accuracy is not high
enough to publish); the recording is the authority and summaries are only a way into it.

This bundle documents the agreed **visual design system** — the one the whole site will use.
It is the output of a design exploration; direction **6b ("green ink, champagne, ruled ground")**
was selected. The bundle is NOT a spec for any particular page. The first site to be built with
it is an internal admin/experimentation tool, which does not need careful visual design — for
that, take the tokens, the type pairing, and the component patterns, and don't spend time on the
signature graphics.

## About the design files

`Kol Torah Directions.dc.html` in this bundle is a **design reference created in HTML** — a set
of side-by-side explorations, newest at the top of the file. It is a prototype of look and feel,
not production code to copy. It uses a component runtime that is irrelevant to your build; read
it for its inline style values only.

**The task is to recreate this visual language in the target codebase's own environment**
(React/Vue/Django templates/whatever is already there) using its established patterns. If no
codebase exists yet, pick the appropriate stack and implement there. `tokens.css` and
`base.css` are provided as a faithful, framework-agnostic reference implementation — port their
values; the class names are not contractual.

Two sections of that file are canonical: **Turn 6 → option 6b** for the page design, and
**Turn 10 → option 10a** for the logo. Everything else in the file is superseded exploration;
ignore it. Both decisions are already captured in the CSS and the `logo/` assets, so you should
rarely need to open the HTML at all.

## Fidelity

**High-fidelity for color, typography, and component treatment** — exact values are in
`tokens.css` and should be reproduced precisely. **Layout is illustrative**: the reference shows
one page (a lesson page) at 760px; real pages should apply the same rules at real breakpoints.
There is no per-screen pixel spec in this bundle, by design — only two screens have been designed
so far, and page-level design work is still pending.

---

## The design direction in one paragraph

Warm off-white ground. **Green is the ink** — headings, rules, the wordmark, filled buttons — and
it is the only strong color on the page. **Champagne gold is never structural**: it appears as a
wash behind the active/playing item, as the player's knob, and nowhere at large size. Because
green and gold never sit at the same visual weight, they don't compete. Type is a two-family
pairing: **Noto Serif Hebrew** for anything titular (it carries the "sefer" feeling) and
**Heebo** for body and UI chrome, with **JetBrains Mono** reserved for timestamps and small
eyebrow labels. Two signature graphics keep pages from feeling empty: a faint **ruled ground**
behind content, and one oversized **Hebrew watermark letter** behind each page title.

### What this system deliberately avoids
- No dark theme, no colored header band (it was tried and rejected), no tekhelet/blue.
- No gradients, no glassmorphism, no large drop shadows (exactly one shadow token exists).
- No illustration or icon-art. Ornament is **only** type, rule, texture, or a real photograph.
- No emoji anywhere.
- No large gold fills — gold at size reads cheap against this green.
- Minimal radii (3–6px). Pills are for topic chips only.

---

## Design tokens

All values live in `tokens.css` with a comment naming each one's role. Colors are authored in
`oklch()`; convert if the target platform needs hex, but keep the relationships (the green is a
desaturated deep green at hue ~148; the gold is warm at hue ~92–100 and always high-lightness
except as text).

Key ones:

| Role | Token | Value |
|---|---|---|
| Primary ink / brand | `--kt-green` | `oklch(0.36 0.06 148)` ≈ `#2f5c47` |
| Display headings | `--kt-green-deep` | `oklch(0.30 0.05 148)` |
| Accent (knob, thin rules) | `--kt-gold` | `oklch(0.80 0.075 92)` |
| Active-row wash | `--kt-gold-wash` | `oklch(0.945 0.055 96)` |
| Notice background | `--kt-gold-soft` | `oklch(0.955 0.030 96)` |
| Page ground | `--kt-bg` | `oklch(0.975 0.008 100)` |
| Cards / header | `--kt-surface` | `#ffffff` |
| Body text | `--kt-text` | `oklch(0.25 0.03 150)` |
| Card border | `--kt-border` | `oklch(0.87 0.010 100)` |
| Elevation (only one) | `--kt-shadow` | `0 24px 60px -30px rgba(45,60,45,.40)` |

Spacing is a 4px base, used as 4 / 7 / 10 / 14 / 18 / 22 / 26 / 32.

### Typography

| Use | Family | Size | Weight | Notes |
|---|---|---|---|---|
| Wordmark | Noto Serif Hebrew | 25px | 700 | green |
| Page title | Noto Serif Hebrew | 30px | 700 | line-height 1.3, `--kt-green-deep`, max-width ~70% on desktop |
| Section heading | Noto Serif Hebrew | 20px | 700 | |
| List-item heading | Noto Serif Hebrew | 17px | 700 | |
| Body / UI | Heebo | 15px | 400 | line-height 1.6 |
| Summary text | Heebo | 14.5px | 400 | line-height 1.65, `--kt-text-muted` |
| Meta | Heebo | 13–14px | 400 | `--kt-text-subtle` |
| Timestamps | JetBrains Mono | 12px | 400 | must render LTR — see RTL notes |
| Eyebrow label | JetBrains Mono | 11px | 400 | letter-spacing .14em |

Fonts are Google Fonts (Noto Serif Hebrew, Heebo, JetBrains Mono) — all have full Hebrew
coverage except JetBrains Mono, which is used for digits only. Self-host them for production.

---

## Signature graphics — implement these two, they are the identity

### 1. Ruled ground
A faint horizontal rule every **31px** (30px gap + 1px line) in `--kt-green-tint`, behind the
main content region only — **not** behind the header, cards, or the sidebar. It evokes the
scribal grid a page is written on. Implemented as a `repeating-linear-gradient`; see `.kt-ruled`.
Hide it in print.

### 2. Watermark letter
One oversized Hebrew letter (250px desktop, 170px mobile) in `--kt-green-tint`, positioned
behind the page title, bleeding off the **trailing** edge (left in RTL, right in LTR) and clipped
by the container — the title is aligned to the leading edge and capped at ~70% width so text never
crosses the letter. In CSS this is `inset-inline-end: -10px`. It gives every page a distinct mark for free, with no asset pipeline.

**Which letter:** the first letter of the page's primary subject — the topic, the tractate, the
series, or the rabbi's name (the reference page shows **ס** for *מלאכת סוחט*). Derive it
server-side and pass it down; it must be deterministic per page so a page always looks the same.
Never make it legible enough to read as content, never interactive (`pointer-events: none`,
`user-select: none`, `aria-hidden="true"`).

---

## Component patterns

Each has a reference implementation in `base.css`; the class names are illustrative.

**Header** — white, **3px solid green** bottom border (the border is doing the work a colored
band used to do). Wordmark right (RTL), dual-mode search field filling the middle, EN toggle and
account at the far end. Persistent on every page.

**Dual-mode search/ask field** — search and the agent are **peer modes**, toggled by a small
two-segment control inside the field itself; the selected segment is filled green. This is a
product-level decision, not decoration: the agent must be reachable from every page without
being the front door.

**Topic strip** — white row under the header, 1px rule beneath, holding pill chips
(`--kt-chip-border`, green text). Shows the current page's topic path or filters.

**Card** — white, 1px `--kt-border`, 6px radius, no shadow. The page container may carry the
single `--kt-shadow`; interior cards never do.

**Audio player** — 44px round green play button; 6px green progress track on a light track. The
fill grows from the **leading** edge (right in RTL), i.e. `inset-inline-start: 0`, so elapsed time
extends away from the play button; the knob and the chapter ticks are likewise positioned by
`inset-inline-start` as a percentage of duration. It carries a **champagne knob** (the one place gold is a filled shape) and 2px **chapter ticks** marking
segment boundaries, so the map and the audio agree visually. Times below in mono, LTR. A control
row under a hairline separator holds skip-back-15, speed, and **"שיתוף מדקה זו"** (share from
this minute) — deep-linking to a timestamp is a core feature, so the affordance is in the player,
not buried in a menu. A **"מנגן: <chapter>"** label in green names the current chapter. On
mobile, a sticky mini-player docks to the bottom.

**Chapter map / list rows** — the core repeating pattern of the whole site, used for chapters,
search results, and series listings. Row = mono timestamp column (50px) + serif heading + sans
summary, separated by 1px rules (no cards, no boxes). **Active row**: champagne wash + 3px green
edge bar bleeding to the container edge, timestamp turns green/bold. Whole row is the hit target.

**Automatic-summary notice** — a `--kt-gold-soft` block, 12.5px, stating that summaries are
generated automatically and the ruling is what is in the recording. **Required on every page that
displays generated summaries.** Wording used in the reference:
> הסיכומים נכתבו אוטומטית כדי לעזור לכם לנווט. ההלכה למעשה היא מה שנאמר בהקלטה.

**Avatar** — circle, champagne background, serif Hebrew initials. Replace with a real portrait
photograph wherever one exists; a real face next to a lesson does more for trust than any
ornament.

**Buttons** — filled green primary, white/green-text secondary, 3px radius. No tertiary style.

---

## Hebrew / RTL / bilingual requirements

These are hard requirements, not polish:

1. **Hebrew is the primary language**; English is a secondary locale, not a fallback. Author
   copy in Hebrew.
2. Set `dir="rtl"` and `lang="he"` on `<html>`; the English locale flips to `dir="ltr"`
   `lang="en"`. **Use logical CSS properties everywhere** — `inset-inline-start`,
   `margin-inline-*`, `padding-inline-*`, `border-inline-*` — so one stylesheet serves both
   directions. Never hard-code `left`/`right`.
3. **Timestamps and any digit run inside Hebrew text must be isolated**:
   `direction: ltr; unicode-bidi: isolate`. Without this, `14:20` next to Hebrew renders
   wrong. This is the single most common bug in this kind of site.
4. Hebrew source references (`שבת קמ״ה.`, `שו״ע או״ח ש״כ`) use gershayim — keep them as
   authored text; do not "normalize" the punctuation.
5. The mono font is for **digits and Latin only**. Never set Hebrew in it.
6. Line-height for Hebrew body runs 1.6–1.65 (Hebrew needs more than Latin at the same size).
7. Minimum font size 12px anywhere; minimum tap target 44px on mobile.

## Accessibility

- Green on white and green-deep on champagne wash both clear WCAG AA for body text; **gold is
  never used as text on a light ground** (`--kt-gold-ink` at 0.42 lightness is the only
  gold-family text color, and only on the champagne avatar).
- Active state is signaled by wash **plus** edge bar **plus** weight — never by color alone.
- `:focus-visible` is a 2px green outline at 2px offset; do not remove it.
- The watermark letter is `aria-hidden`.
- Player needs real keyboard control (space, arrows) and `aria-valuenow` on the track.

## Responsive behavior

Single breakpoint at **860px**. Above it, content + 262px sidebar side by side; below, the
sidebar stacks under the content, the header wraps with the search field on its own row, the
title drops to 24px, the watermark to 170px, and the player becomes a sticky bottom mini-player.
Mobile-friendliness is a primary requirement — most listening will happen on a phone.

## Content rules that affect the UI

- **Never display a raw transcript.** Show generated summaries, chapter titles, and Q&A units.
- Every summary is an entry point: it must link to its exact timestamp in the recording.
- The **agent may not answer unsourced**. An answer is an arrangement of attributions — each one
  naming the rabbi, the lesson, and the minute, with a play affordance — and an honest "לא מצאתי"
  when nothing grounded is found. Because transcripts aren't published, an attribution card
  shows a *summary* of what was said plus a play button, not a verbatim quote.
- Search returns **segments, not lessons**: a match inside minutes 12–15 of an unrelated lesson
  is a first-class result and must show which lesson it came from and at what minute.

## Logo

The mark is the **arch of an aron kodesh** — read equally as the doorway into a beit midrash —
filled with the ruled lines of the site's own ground. **The third line is gold: the minute that is
playing.** It is the same idea as the player's champagne knob and the active chapter row, reduced
to a mark, and it is drawn only from an arc and four rounded rectangles, so it holds from a
printed cover down to 16px.

Ship the SVGs in `logo/` as-is; do not redraw the mark in CSS, icon fonts, or code.

| File | Use |
|---|---|
| `logo/lockup-he.svg` | primary Hebrew lockup, mark + wordmark + tagline |
| `logo/lockup-en.svg` | English lockup |
| `logo/mark.svg` | mark alone, full colour on a light ground |
| `logo/mark-reverse.svg` | on green or any dark ground (outlined arch, white rules) |
| `logo/mark-mono.svg` | one colour, **inline use only** — the rules are knocked out of the arch and the whole mark takes `currentColor`, so it inherits text colour. `currentColor` cannot cross an `<img>` boundary; inline the SVG or use the two files below |
| `logo/mark-black.svg` | one colour, static — for `<img>`, `background-image`, email, print |
| `logo/mark-white.svg` | one colour white, static — same, on dark grounds |
| `logo/favicon.svg` | 16–32px only: a **square** green tile with an outlined arch and two rules — the detailed mark fills in at that size. Square so it composites cleanly as a favicon, apple-touch-icon, or maskable PWA icon |
| `logo/logo.html` | usage sheet: lockups, sizes, clear space, and the don'ts |

**Rules.** Clear space on every side = the width of one ruled line inside the mark (≈18% of mark
height). Minimum sizes: mark 20px tall on screen / 6mm in print; lockup 120px wide; below 20px
switch to `favicon.svg`. In the site header the mark is **28px tall, before the wordmark**, with
14px between them, and the whole lockup is one link to the homepage.

**Don't:** recolour the arch; add a second gold element (the mark has exactly one); outline,
shadow, or gradient it; stretch or reproportion the arch; place the solid mark on a mid-tone
photograph (use the reverse); or set the wordmark in anything but Noto Serif Hebrew 700.

In the one-colour marks the ruled lines are **knocked out** of the arch rather than filled white,
so a single ink works on any ground. Never fill them white in a mono context.

The two lockup SVGs set the wordmark as live `<text>` in Noto Serif Hebrew — correct on the web
where the font is loaded, but **convert the text to outlines** before using them in print or
anywhere the font may be missing.

## Assets

The logo files in `logo/` are the only required assets. No other images, icons, or illustrations
are needed — that is deliberate, so the system scales to thousands of lessons without an art
pipeline. The one image the design wants is a real **portrait photograph per rabbi**, used in
place of the initials avatar. Font files come from Google Fonts.

## Files in this bundle

- `README.md` — this document, self-sufficient.
- `tokens.css` — every design value, each with its role in a comment. **Start here.**
- `base.css` — reference implementation of the base styles and each component pattern.
- `logo/` — the final logo assets plus `logo.html`, the usage sheet. See **Logo** above.
- `example-page.html` — plain HTML using only those two stylesheets: the designed lesson page plus a
  second page shape (content + sidebar), so you can see the system applied twice. Open it in a
  browser to check your port against it.
- `Kol Torah Directions.dc.html` — the original design exploration. Read **Turn 6 → 6b** (top of
  the file, second option) as canonical; everything below is superseded.

## Still open (not decided, do not invent)

Page-level design exists for the **lesson page** only. The homepage, search results, agent
conversation, onboarding, series/rabbi pages, and saved library have been discussed but **not
designed**. Build the admin tool on the tokens and components above; do not extrapolate a
user-facing page layout from this bundle without going back to design. The logo **is** settled —
see **Logo** above.
