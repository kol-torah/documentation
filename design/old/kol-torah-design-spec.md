# Kol Torah — Design Specification

**Version:** 1.0
**Direction:** "Stone and Light" (אבן ואור), Olive palette (זית), with minimal grain
**Applies to:** Main public site + Admin panel (one shared design system)
**Primary language:** Hebrew (RTL). English to be added later.

---

## 0. How to use this document

This is a design spec, not a component library. It defines tokens, rules, and rationale. Build the actual components in whatever framework the project uses, but derive every color, size, and spacing value from the tokens in §3 — do not introduce new hex values or arbitrary pixel spacing without a stated reason.

Sections marked **[Open]** are deliberately undecided. Those are good places to propose alternatives.

---

## 1. Why this direction was selected

This section exists so the choice can be challenged intelligently rather than treated as fixed.

### The brief

A Torah study site. It sells nothing and advertises nothing. The requested feeling was *a yeshiva library — spacious, not crowded with books*. The site must feel professional without feeling commercial, and must not feel like a template.

### What was rejected, and why

**Warm parchment with brass accents and book illustrations.** The first attempt. Rejected as "not deep and quiet enough." The diagnosis: too many warm accent colors competing for attention, decorative illustration doing work that restraint should do, card-based layout that reads as a product grid. Cards imply items for sale or consumption; a study site's content should read as a *list*, closer to a table of contents than a shop.

**Dark "night seder" direction.** Deep ink background with a pool of warm lamplight. Genuinely atmospheric and arguably the most emotionally resonant option. Rejected because it does not scale: a Torah archive gets dense fast — dozens or hundreds of shiurim, filters, search, pagination. Dark backgrounds with light text become fatiguing in long scannable lists, and the atmosphere that makes the homepage beautiful actively works against a functional archive page. Also awkward for an admin panel, where people work for extended periods.

**Techelet (blue-teal) palette.** Rejected in favor of olive. Blue is the default choice for Jewish and religious sites; it reads as conventional, and at scale — repeated across many topic labels — it starts to look corporate rather than contemplative.

### Why this direction was selected

**Light background, generous negative space.** The "spacious library" feeling is produced by emptiness, not by imagery of books. The hero is mostly air. Sections are separated by wide margins rather than by boxes and borders. This is the single most important property of the design — if a future change makes the page denser, it loses the brief.

**A list, not a grid.** Shiurim are presented as full-width rows separated by hairlines. This reads as a scholarly index. It also scales gracefully: 3 items or 300 items look equally correct, whereas a card grid becomes a wall at scale.

**Olive accent.** Earthy, warm, and uncommon on religious sites. It carries associations with land and cultivation without being literal or symbolic. It is also low-saturation enough to be repeated many times (topic labels, statuses, active states) without becoming loud — this was the specific failure mode of the blue option.

**The arch as the single signature element.** One geometric form, drawn in hairlines, holding a soft wash of color. It references beit midrash and synagogue architecture without depicting anything religious literally — no scrolls, no ritual objects, no Hebrew letters used as decoration. This matters: decorative use of sacred imagery can read as tacky or worse, depending on audience. An empty architectural opening is respectful and abstract.

**Restraint elsewhere.** Per the "spend your boldness in one place" principle: the arch is the memorable element, so everything around it stays quiet. No shadows, no rounded corners, no gradients other than the two soft washes, no illustrations, no icons except the single brand dot.

**Minimal grain.** Barely perceptible texture across the background, referencing the cut surface of Jerusalem stone. It removes the flat, sterile quality of pure digital white without ever being noticed consciously. Heavier stone treatments (visible ashlar coursing, chiseled masonry arch) were prototyped and rejected: at large viewport widths a repeating block grid reads as a pattern swatch, and it competes with content on dense pages.

### Known tradeoffs of this direction

- **Low contrast is the risk.** This design lives on subtlety, which means it can tip into "washed out" if implemented carelessly. Contrast requirements in §9 are not optional.
- **Bellefair is a light display face with a single weight.** There is no bold. Hierarchy must come from size and space, not weight. If a future page needs heavy emphasis in a heading, the type system is the thing that has to change, not the heading.
- **It is quiet by design, which means it is not attention-grabbing.** If the site ever needs to promote an event or drive urgent action, this system has no loud register to reach for. That is a deliberate cost.

---

## 2. Structure

Both sites share one design system. The admin panel is not a separate visual product — it uses the same background, type, accent, spacing, and hairlines. The only differences:

| | Public site | Admin |
|---|---|---|
| Background | `--bg` | `--adm-bg` (slightly brighter, for data density) |
| Sidebar | none | `--adm-side` tinted panel |
| Density | generous (32px row padding) | compact (13px cell padding) |
| Display type | used at large sizes | used only for panel titles |

Rationale: staff who publish shiurim should feel they are inside the same institution, not in a generic CMS. It also halves the design and maintenance surface.

---

## 3. Design tokens

Define these once, in a single source of truth (CSS custom properties on `:root`, or a theme file). Everything else references them.

```css
:root {
  /* Surfaces */
  --bg:        #FAF8F2;  /* page background — warm off-white */
  --band:      #F1EEE3;  /* tinted section band, separates content zones */
  --adm-bg:    #FEFDF9;  /* admin content surface */
  --adm-side:  #EEEBDF;  /* admin sidebar */

  /* Ink */
  --head:      #1E2019;  /* headings */
  --text:      #34362E;  /* body text */
  --muted:     #8A8A7C;  /* secondary/metadata — see contrast note §9 */

  /* Accent */
  --accent:    #5A6B3C;  /* olive */

  /* Derived (do not hand-pick new values — derive from accent/head) */
  --hair:      rgba(30, 32, 25, 0.13);  /* all hairline borders */
  --hover:     rgba(90, 107, 60, 0.05); /* row hover tint */
  --wash:      rgba(90, 107, 60, 0.09); /* soft accent gradients */
  --stone:     rgba(30, 32, 25, 0.13);  /* arch outline */
  --adm-on:    rgba(90, 107, 60, 0.12); /* active nav item */
  --live-bg:   rgba(90, 107, 60, 0.12); /* published status chip */
}
```

**Rules:**
- No shadows anywhere. Separation is achieved with hairlines and space.
- No border-radius except on the arch and the brand dot. Everything else is square.
- `--hair` is the *only* border color. One hairline weight (1px) across both sites.

---

## 4. Typography

```
Display / headings:  Bellefair, serif        — 400 only (no other weight exists)
Body / UI:           Heebo, sans-serif       — 200, 300, 400, 500
```

Both are on Google Fonts with full Hebrew coverage.

### Why this pairing

Bellefair is a light, high-contrast serif with genuine Hebrew support — unusual, since most elegant serifs are Latin-only and most Hebrew serifs are heavy and institutional. Its lightness is what produces the "quiet" quality. Heebo is a neutral, highly legible Hebrew sans that stays out of the way in metadata and interface chrome.

### Scale

| Role | Face | Size | Weight | Notes |
|---|---|---|---|---|
| Hero heading | Bellefair | 52px / 34px mobile | 400 | line-height 1.3 |
| Section heading | Bellefair | 30px | 400 | |
| Shiur title | Bellefair | 24px | 400 | |
| Panel title (admin) | Bellefair | 18px | 400 | |
| Brand wordmark | Bellefair | 21px | 400 | letter-spacing 1px |
| Body / subhead | Heebo | 16px | 300 | line-height 1.9 |
| Metadata | Heebo | 13px | 300 | |
| Eyebrow / label | Heebo | 11px | 400 | letter-spacing 3–4px, uppercase not applicable in Hebrew |
| Table cell | Heebo | 13px | 300 | |

**Hard rule:** never set Bellefair below 18px. It is too light to hold at small sizes and will look broken. All small text is Heebo.

### RTL

- `<html lang="he" dir="rtl">`.
- Use logical CSS properties throughout (`padding-inline-start`, `border-inline-end`, `margin-inline`) rather than left/right. This is what makes the English version cheap to add later.
- Use Hebrew punctuation: geresh/gershayim (`״` `׳`) not ASCII quotes, in titles like `הרמב״ן`.
- The one exception: the grain overlay's split-comparison `clip-path` in the prototype uses physical directions. Not needed in production.

### **[Open]** English later

When English is added, Bellefair covers Latin natively, so the display face carries over. Heebo does not have great Latin — plan to swap the body face per language (Heebo for `:lang(he)`, something neutral like Inter for `:lang(en)`). Decide this when English work actually starts, not before.

---

## 5. Spacing

Base rhythm is generous. These are the values that produce the "spacious" feeling — treat reductions as design changes, not tweaks.

```
Page gutter:              48px desktop / 24px mobile
Max content width:        1080px
Admin max width:          1000px

Hero padding:             104px top / 88px bottom
Section band padding:     78px vertical
Space above admin intro:  84px
Footer padding:           60px

Shiur row padding:        32px vertical
Admin table cell:         13px vertical / 10px horizontal
Admin sidebar padding:    26px vertical / 18px horizontal
```

---

## 6. Components

### 6.1 Header

Full-width, hairline bottom border, 24px vertical padding. Brand on the start side (right in RTL): a 7px olive dot followed by `קול תורה` in Bellefair. Nav links on the end side, Heebo 300 at 14px in `--muted`, 32px gap.

Nav hover: text goes to `--accent` and a 1px `--accent` underline appears beneath the link (3px offset). Transition 250ms.

**Not sticky.** The prototype used a sticky header only to host the demo controls. A quiet page should let the header scroll away.

### 6.2 Hero

Centered. Vertical order: arch → eyebrow → h1 → subhead → actions.

**The arch.** 148×206px, centered, 46px bottom margin.
- Outer shape: 1px `--stone` border, `border-radius: 74px 74px 0 0`.
- Inner shape: a pseudo-element inset 26px on start/end/top, `border-radius: 48px 48px 0 0`, same border, filled with a vertical gradient from `--wash` to transparent at 85%.

**The glow.** A non-interactive radial gradient positioned above and behind the hero content: 900×400px, centered, offset -140px from the top, `radial-gradient(ellipse at center, var(--wash) 0%, transparent 68%)`. The hero needs `overflow: hidden` and the content needs a higher stacking context.

**Buttons.** Not filled. 1px `--accent` border, `--accent` text, 10px/24px padding, square corners. On hover the fill inverts: background `--accent`, text `--bg`. Secondary variant uses `--muted` text and `--hair` border, and on hover only darkens the text and border — it never fills.

### 6.3 Shiurim list

Wrapped in a `--band` section with hairline top and bottom borders — this band is what separates content zones instead of cards.

Each row: hairline top border, 32px vertical padding, flex with title block at the start and metadata at the end, baseline-aligned. Last row also gets a bottom border.

Row contents:
- Topic label — 11px Heebo, `--accent`, letter-spacing 2px, 7px bottom margin
- Title — 24px Bellefair, `--head`, line-height 1.5
- Metadata — 13px Heebo 300, `--muted`, `white-space: nowrap` (rabbi name · duration)

**Hover:** background tints to `--hover`, and a 1px `--accent` line draws across the row's top edge from the start side, animating width 0 → 100% over 400ms. This is the one piece of motion on the page that is purely expressive; it should feel like a line being ruled, not like a UI response.

**Mobile:** below 820px the row becomes a column with 8px gap.

### 6.4 Admin

Sidebar 200px fixed width, `--adm-side` background, hairline border on the inline-end edge. Panel title in Bellefair 16px. Nav items are 13px Heebo 300 in `--muted`, 9px/11px padding, with a transparent 2px border on the inline-end edge. Active item: `--adm-on` background, `--accent` text, `--accent` border on that edge.

Table: full width, collapsed borders. Headers are 11px Heebo 400 in `--muted` with 1.5px letter-spacing and a hairline bottom border. Cells are 13px Heebo 300 with hairline bottom borders.

Status chips: 11px, 3px/11px padding, square, 1px letter-spacing. Published uses `--live-bg` background with `--accent` text; draft uses `--hover` background with `--muted` text.

**[Open]** The admin currently only demonstrates a list view. Forms, uploads, the audio/video player, and validation states are unspecified. When designing them: no shadows, no rounded inputs, hairline borders, focus state uses `--accent`.

---

## 7. The grain

A fine noise texture across the page background. Confirmed at very low intensity — it should be invisible unless someone zooms in looking for it.

```css
.grain {
  position: fixed;          /* NOT absolute — see note below */
  inset: 0;
  z-index: 1;
  pointer-events: none;
  opacity: 0.55;
  mix-blend-mode: multiply;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='180' height='180'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3CfeColorMatrix type='saturate' values='0'/%3E%3C/filter%3E%3Crect width='180' height='180' filter='url(%23n)' opacity='0.35'/%3E%3C/svg%3E");
}
```

**Critical implementation notes:**

1. **Use `position: fixed`, not `absolute`.** The prototype used absolute positioning, which is fine on a short page but means the overlay spans the entire document height on a long archive page — wasteful and can cause repaint cost. Fixed to the viewport gives identical visual results at constant cost.
2. All page content must sit above it. Give content containers `position: relative; z-index: 2`.
3. `mix-blend-mode: multiply` is what keeps it subtle. Do not substitute opacity-only compositing.
4. Because it is a data-URI SVG, no network request and no image asset to manage.
5. **Verify it renders** on Safari and on mobile Chrome before considering it done — `feTurbulence` in a data URI combined with `mix-blend-mode` is the single most fragile thing in this spec. If it fails on a target browser, ship without it. The design works fine without grain; it is a refinement, not a foundation.
6. Skip it entirely behind `prefers-reduced-transparency` if you want to be thorough, though it is not motion so `prefers-reduced-motion` does not apply.

---

## 8. Motion

Deliberately sparse. Total inventory:

| Element | Effect | Duration |
|---|---|---|
| Nav link | color + underline width | 250ms |
| Button | background/text inversion | 250ms |
| Shiur row | rule draws across top edge | 400ms ease |
| Shiur row | background tint | 300ms |
| Admin nav item | background/color | 200ms |

No scroll-triggered animation. No page-load sequence. No parallax. Additional motion should be argued for, not added by default — the quiet is the product.

Respect `prefers-reduced-motion: reduce` by disabling transitions (keep the end states).

---

## 9. Accessibility and quality floor

**Contrast — read this carefully, this design's main risk.**

| Pair | Ratio | Verdict |
|---|---|---|
| `--text` on `--bg` | ~11:1 | Passes AAA |
| `--head` on `--bg` | ~14:1 | Passes AAA |
| `--accent` on `--bg` | ~5.3:1 | Passes AA for normal text |
| `--muted` on `--bg` | ~3.1:1 | **Fails AA for body text** |

`--muted` is acceptable for large text, decorative labels, and non-essential metadata. It is **not** acceptable for anything a user must read to accomplish a task — form labels, error messages, table headers conveying required meaning. If information matters, use `--text`. If a designer later asks to make more things muted, this is the reason to say no.

Verify these ratios yourself during implementation rather than trusting the table.

**Other requirements:**
- Visible keyboard focus on every interactive element. Use a 2px `--accent` outline with 2px offset — do not remove outlines without replacing them.
- Full responsive behavior down to 360px.
- The arch and grain are decorative: `aria-hidden="true"`, no alt text.
- Hebrew screen reader support depends on correct `lang` and `dir` attributes — set them on `<html>`, and on any English fragments embedded in Hebrew pages.
- Semantic headings in order. The visual scale is not the document outline.

---

## 10. Content and copy

Written content is design material here, since there is almost no imagery to carry tone.

- Plain, calm, unhurried. No marketing register, no exclamation, no urgency.
- Buttons name the destination or action literally: `לשיעורים`, `לוח שבועי`. Not `לחצו כאן`.
- Empty states are invitations, not apologies: an empty topic says what would appear there, not "no results found."
- Errors state what happened and what to do, in the interface's voice.
- Vocabulary stays consistent between the public site and admin: if the public site says `שיעור`, admin says `שיעור`, not `פריט` or `תוכן`.

---

## 11. Open questions

Bring alternatives on any of these — none are settled:

1. **Archive and search pages.** Only the homepage list is designed. A full archive needs filtering by topic and by rabbi, plus pagination or infinite scroll. This is where the design will be stress-tested.
2. **Audio/video player.** Probably the most-used component on the site, and entirely unspecified. It needs to be quiet and not break the restraint — a standard branded player widget would be jarring here.
3. **Rabbi/maggid shiur pages.** Photographs would be the obvious approach, but photographs will fight this design badly. Consider whether they are needed at all.
4. **Topic color-coding.** Currently every topic label is olive. Multiple hues would aid scanning in a large archive but risk the "loud" failure mode. If pursued, use tints of a narrow range rather than distinct hues.
5. **Logo/wordmark.** Currently the wordmark is just Bellefair text with a dot. A real mark could be derived from the arch geometry.
6. **Dark mode.** The rejected "night seder" direction could return here as an opt-in theme rather than the default — arguably the best home for it.
7. **Whether the grain survives.** If it fails browser testing or nobody can perceive it in production, dropping it costs nothing.

---

## 12. Reference prototypes

The working prototypes these tokens were extracted from:

- `kol-torah-grain-compare.html` — the selected design, with palette and grain controls
- `kol-torah-color-variants.html` — olive vs wine vs techelet comparison
- `kol-torah-three-directions.html` — the three rejected/considered directions
- `kol-torah-stone-touches.html` — the three stone treatments

Values in this document supersede the prototypes where they disagree. The prototypes were built for comparison, not production — they contain demo scaffolding (switchers, sliders, split-view logic) that should not be carried over.
