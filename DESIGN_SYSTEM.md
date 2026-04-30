# Wizzy Portfolio — Design System Spec

> **⚠️ Source of Truth Hierarchy**
> 1. **Figma MCP** = primary source for token VALUES (colors, font sizes, spacing pixels, etc.)
> 2. **This document** = supplementary source for what's NOT in Figma (responsive grid, Korean handling, component max-widths, naming conventions, hard rules)
> 3. **If Figma value differs from this doc** → Figma wins. Report the diff.

---

## 1. Aesthetic Direction

**Editorial Magazine × Confident Restraint**
References: GH Social, Thea Washington Casting, Cereal Magazine.

- Dark mode native
- Cream + dark inverse rhythm for cards
- Hairline borders (0.5–1px) only — no heavier
- Background contrast > borders (cards distinguished by bg color)
- Sans-serif headlines (Manrope) with Playfair Italic *signature emphasis* on key words
- Generous whitespace, editorial pacing
- NO drop shadows, NO gradients

---

## 2. Token Source — Figma MCP

All tokens (colors, typography, spacing, radius, border weights) live in the Figma file:
- Page: `🎯 Final System`
- Frame: `01 Tokens`

**Figma Variable → CSS custom property mapping pattern**:
```
Figma:  bg/page         →  CSS:  --bg-page
Figma:  text/primary    →  CSS:  --text-primary
Figma:  accent/yellow   →  CSS:  --accent-yellow
Figma:  card/cream      →  CSS:  --card-cream
Figma:  spacing/8       →  CSS:  --space-8
Figma:  radius/card     →  CSS:  --radius-card
```

**Rules for CSS conversion**:
- Use exact Figma values, do not round or guess
- Preserve Figma naming hierarchy (slash → dash)
- If Figma has Variable that this doc doesn't mention, include it
- If this doc mentions a token that Figma doesn't have, report it (don't invent)

### Legacy name notes

| Token | Actual value | Note |
| --- | --- | --- |
| `--card-cream` | `#CAEAFF` (light blue) | Name retained for legacy compatibility — was `#F0EBE0` (beige) until content review on 2026-04-30. All "cream variant" cards now render in light blue. Do **not** rename the token (avoid churn across cards.css / ui.css / preview / pages); rename only if Figma source is later renamed. |

**Color family separation** (avoid conflict between two light-blue tokens):
- `--card-cream` (`#CAEAFF`) — *card backgrounds only*. Larger fill areas.
- `--accent-blue` (`#B1E0FF`) — *eyebrow text, hover underlines, em-dash markers, small accents*. Reserve for foreground accents on dark surfaces. Avoid placing accent-blue elements directly on a card-cream background — contrast is too low (both tones share R/G in the 220+ range).

---

## 3. Font Loading

Three font families used:

```css
/* In design-system.css */

/* Manrope — Latin sans, headlines + body */
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&display=swap');

/* Playfair Display — Latin serif italic, signature emphasis only */
@import url('https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@1,400&display=swap');

/* Pretendard — Korean, all weights */
@import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.css');

:root {
  --font-sans: 'Manrope', -apple-system, system-ui, sans-serif;
  --font-serif: 'Playfair Display', Georgia, serif;
  --font-kr: 'Pretendard', -apple-system, system-ui, sans-serif;
}
```

**Loading strategy**: `display=swap` — text appears in fallback until web font loads.

---

## 4. Typography Rules

### English text
- Base font: **Manrope** (body, headings)
- Signature emphasis (within headlines, counters, stat numbers): **Playfair Display Italic**
- Letter-spacing for Playfair Italic: **always -0.05em** (-5%)

### Korean text
- Base font: **Pretendard**
- Emphasis: **Pretendard Bold** (NEVER Playfair Italic — italic doesn't work for Korean)
- Numbers (e.g., "30%", "76") stay Playfair Italic regardless of language context

### `.ds-signature` — Korean override (HARD RULE)

`.ds-signature` is Playfair Italic only in EN context. In any Korean context (`.ds-kr` ancestor/self, or page in `data-lang="ko"`) it MUST switch to Pretendard normal-style. The override below is included in `cards.css`:

```css
/* EN default — Playfair Italic, ls -0.05em */
.ds-signature {
  font-family: var(--font-serif);
  font-style: italic;
  letter-spacing: -0.05em;
  font-weight: var(--text-sig-28-weight);
}

/* KR override — Pretendard, normal style, ls -0.02em */
.ds-kr .ds-signature,
.ds-kr.ds-signature,
.ds-signature.ds-kr,
html[data-lang="ko"] .ds-signature {
  font-family: var(--font-kr);
  font-style: normal;
  font-weight: 700;
  letter-spacing: -0.02em;
}
```

**Markup rule**: any element wrapping Korean signature words must either be `data-lang-aware` (JS-toggled `.ds-kr`) or sit inside `<html data-lang="ko">`. Number stats are unaffected (still Playfair Italic per design tokens).

> **Language code convention**: `en` / `ko` (NOT `kr`). `KR` is the country code (Republic of Korea), `KO` is the ISO 639-1 language code. CSS attribute selectors, JS data attributes, and toggle button labels all use `EN` / `KO`.

### Markup pattern

```html
<!-- English: Playfair Italic emphasis word inside Manrope headline -->
<h2 class="ds-h2">
  HVAC decisions repeat — but the
  <em class="ds-signature">criteria</em>
  never accumulate
</h2>

<!-- Korean: Pretendard Bold emphasis -->
<h2 class="ds-h2 ds-kr">
  이 문제는 단순히 <strong>온도</strong>를 조절하는 문제가 아니었다
</h2>

<!-- Numbers always Playfair Italic -->
<div class="ds-stat-massive">76</div>
<div class="ds-stat-massive">30<span class="ds-stat-suffix">%</span></div>
```

---

## 5. Responsive Grid System

### Breakpoints

```css
/* Mobile-first. Default = mobile. Scale up. */

/* Tablet+    */ @media (min-width: 768px)  { ... }
/* Desktop+   */ @media (min-width: 1440px) { ... }
/* Wide+      */ @media (min-width: 1920px) { ... }
```

### Container

```css
.ds-container {
  width: 100%;
  margin-inline: auto;
  padding-inline: 20px;       /* Mobile */
}

@media (min-width: 768px) {
  .ds-container {
    max-width: 720px;
    padding-inline: 32px;
  }
}

@media (min-width: 1440px) {
  .ds-container {
    max-width: 1280px;
    padding-inline: 80px;
  }
}

@media (min-width: 1920px) {
  .ds-container {
    max-width: 1440px;
    padding-inline: 80px;
  }
}
```

### Typography responsive scale

If Figma defines mobile typography, use those. Otherwise apply:

```css
@media (max-width: 767px) {
  :root {
    --fs-display:        56px;  /* desktop 96px */
    --fs-h1:             40px;  /* desktop 64px */
    --fs-h2:             28px;  /* desktop 40px */
    --fs-h3:             18px;  /* desktop 22px */
    --fs-body-lg:        17px;  /* desktop 18px */
    --fs-body:           15px;  /* desktop 16px */
    --fs-stat-massive:   72px;  /* desktop 120px */
    --fs-stat-large:     56px;  /* desktop 96px */
  }
}
```

### Korean heading downscale tokens

Hangul renders larger than Latin at the same px size. KR-specific font-size tokens auto-apply when an element has `.ds-kr` paired with a heading utility (`.ds-display` / `.ds-h1` / `.ds-h2` / `.ds-h3`), or for our existing card BEM classes via scoped overrides in `cards.css`.

```css
:root {
  --fs-display-kr: 80px;  /* EN 96 · ~83% */
  --fs-h1-kr:      54px;  /* EN 64 · ~84% */
  --fs-h2-kr:      34px;  /* EN 40 · ~85% */
  --fs-h3-kr:      20px;  /* EN 22 · ~91% */
}

@media (max-width: 767px) {
  :root {
    --fs-display-kr: 48px;
    --fs-h1-kr:      36px;
    --fs-h2-kr:      26px;
    --fs-h3-kr:      18px;
  }
}
```

**Auto-apply rules** (in `cards.css`):
```css
.ds-kr.ds-h1, .ds-h1.ds-kr, .ds-kr .ds-h1 { font-size: var(--fs-h1-kr); }
/* ... same pattern for display / h2 / h3 */
```

Plus BEM-scoped equivalents so `.ds-numbered-step.ds-kr .ds-numbered-step__title`, `.ds-feature.ds-kr .ds-feature__title`, etc. all auto-downscale without changing markup. Line-height stays at the EN token value (intentionally generous for hangul).

### Padding responsive pattern

```css
.ds-card {
  padding: 32px;                                       /* Mobile */
}
@media (min-width: 768px)  { .ds-card { padding: 40px; } }
@media (min-width: 1440px) { .ds-card { padding: 56px; } }
```

---

## 6. Card Components — Max-Width Spec

**Critical**: Figma card sizes are *visual reference only*. Real card widths are determined by content + container, not Figma frame width. Use these max-widths in `components.css`:

**Updated 2026-04-30**: Card max-widths now apply at **all breakpoints** (base CSS, not gated by desktop-only media query). On narrow viewports the natural `width: 100%` is unconstraining; on wider viewports the cap engages and `margin-inline: auto` centers the card. Outcome / Approach / Ideate / Define section containers also use `display: flex; flex-direction: column; gap: --space-20; align-items: center` to ensure vertical separation between centered cards.

| Card variant            | All breakpoints (max-width)    | Notes |
| ----------------------- | ------------------------------ | ----- |
| `numbered-step`         | **640px**                      | reduced from 720 — tighter line lengths |
| `pull-quote`            | 800px                          |       |
| `insight`               | 560px                          | reset to `none` inside `.ds-need-pain-grid` / `.ds-impact-grid` |
| `research-finding`      | 1080px                         |       |
| `criteria-list`         | 100% container                 |       |
| `feature`               | 720px                          |       |
| `before-after`          | 100% container                 |       |
| `split-feature`         | 100% container                 |       |
| `stat` Variant A (76)   | **480px**                      | applied at base, was tablet+ only |
| `stat` Variant B (grid) | 100% container                 |       |
| `media --full`          | 100%                           |       |
| `media --two-up`        | 100% mobile / 50% tablet+      |       |
| `media --inline`        | 720px                          |       |

All cards centered with `margin-inline: auto` when narrower than container.

---

## 7. Card Components — Detail Spec

> Each card's structure, hover behavior, and responsive transformation.
> Visual styling (colors, fonts) → from Figma tokens.
> Sizing/behavior → from this doc.

### 7.1 `numbered-step`

**Use**: Overview steps (01–05), Reflection points (01–05)

**Structure** (required):
- `__counter` — Playfair Italic, **48px** (`--text-sig-48-*` tokens). Switches to text-primary on hover (yellow→primary on dark, dark→yellow on cream).
- `__caption` ("— How It Started", eyebrow style, tertiary)
- `__divider` (short hairline, hover-extends 2×)
- `__title` h3 (with optional `<em class="ds-signature">` emphasis)
- `__body` paragraph (optional)

**Optional children** (composable, append below body in this order):
- `__quote` — blockquote-style emphasis. Left 2px accent-yellow stripe (`--border-thick` token), padding-left `--space-6`. Italic body-lg in EN; switches to upright Pretendard in `.ds-kr`.
- `__list` — `<ul>` with em-dash markers in accent-blue, body size, gap `--space-2`. Cream variant uses `currentColor` for marker.
- `__closing` — single body-lg line, text-primary, font-weight matches h3.

**Sizing & padding**:
- Internal padding: **`--space-6` mobile (24) / `--space-8` tablet (32) / `--space-10` desktop (40)**
- Max-width: 100% mobile / 100% tablet / 720px desktop+ (per Section 6 table)

**Vertical rhythm** (per-child `margin-top`, replaces flex gap):
- counter → caption: `--space-3` (12)
- caption → divider: `--space-4` (16)
- divider → title:   `--space-6` (24)
- title → body / quote / list: `--space-4` (16)
- body or quote → list: `--space-6` (24)
- * → closing: `--space-6` (24)

**Variants**: `--dark` (bg-surface) | `--cream` (card-cream)

**Optional layout variant — `--with-media`**

Adds a square media slot inset at the top-right of the card. Requires wrapping the existing children in `__content` and adding a sibling `__media` slot.

```html
<article class="ds-numbered-step ds-numbered-step--cream ds-numbered-step--with-media">
  <div class="ds-numbered-step__content">
    <p class="ds-numbered-step__counter">02</p>
    <p class="ds-numbered-step__caption">— THE PROBLEM</p>
    <div class="ds-numbered-step__divider"></div>
    <h3 class="ds-numbered-step__title">
      Three adjustments per day, <em class="ds-signature">zero</em> accumulated learning
    </h3>
    <p class="ds-numbered-step__body">…</p>
  </div>
  <div class="ds-numbered-step__media">
    <!-- <img src="..." alt="..." />  or placeholder content -->
  </div>
</article>
```

| Breakpoint | Media size | Position | Content `padding-right` |
| --- | --- | --- | --- |
| Mobile (<768px) | 100% width × 16:10 (full-width block above text via flex `order: -1`) | static, `margin-bottom: --space-6` | none |
| Tablet (768+) | **160px × 160px** (`--space-40`, 1:1) | absolute, top/right = `--space-8` | `calc(--space-40 + --space-6)` = 184 |
| Desktop (1440+) | **200px × 200px** (`--space-50`, 1:1) | absolute, top/right = `--space-10` | `calc(--space-50 + --space-6)` = 224 |

**Slot styling**: `bg-muted` background, `--radius-card` (16px), `overflow: hidden`. Any `<img>` / `<video>` child fills via `width: 100%; height: 100%; object-fit: cover`. Empty/placeholder content is centered (monospace caption) until image is plugged in.

**Image export guide**: 1:1 ratio. Display 200px → 2× export = **400×400** is enough; larger fine since downscaled. Even simple diagrams/placeholders at 1080×1080 work.

**Hover**:
- Counter color shift (yellow→primary on dark, dark→yellow on cream)
- Divider extends from short to ~2x length
- Transition: 250ms ease

**Mobile**: Same vertical layout, padding scaled down.

### 7.2 `pull-quote`

**Use**: Standalone emphasis quotes (▎ box quotes)

**Structure**:
- quote text (Manrope Medium 22px) with Playfair Italic emphasis words
- optional context line below (~70% opacity)

**Variants**: `--cream` (default, inverse against dark page) | `--dark`

**Border**: NONE — background contrast does the work.
**Radius**: card-lg.

**Hover**: subtle scale 1.01 (250ms editorial easing).

### 7.3 `insight`

**Use**: Standalone insight callouts; or paired in `.ds-need-pain-grid` for User Need / Pain Point comparison.

**Structure**:
- eyebrow ("INSIGHT", "USER NEED", "PAIN POINT", etc.)
- h3 title with Playfair Italic emphasis
- body paragraph (optional)

**Variants**:
- `--cream` (default) — light bg, dark text (USER NEED, positive)
- `--yellow` — filled accent yellow bg, dark text (signature moment)
- `--dark` — bg-surface, primary text (PAIN POINT, contrast / negative)

**Eyebrow color** (when using `<p class="ds-eyebrow">` utility-class markup inside insight):
- `--cream` / `--yellow`: `var(--text-on-cream)` opacity 0.6
- `--dark`: `var(--accent-blue)` opacity 1

**Hover**: subtle lift `translateY(-2px)`, 250ms.

**Need / Pain comparison pattern** — pair two `.ds-insight` cards (one default cream, one `--dark`) inside `.ds-need-pain-grid` for side-by-side User Need vs Pain Point:

```html
<div class="ds-need-pain-grid">
  <article class="ds-insight">
    <p class="ds-eyebrow">USER NEED</p>
    <p class="ds-h3">Stay <em class="ds-signature">comfortable</em> while cutting cost</p>
  </article>
  <article class="ds-insight ds-insight--dark">
    <p class="ds-eyebrow">PAIN POINT</p>
    <p class="ds-h3">Adjustments lack <em class="ds-signature">cost feedback</em> — anxiety follows</p>
  </article>
</div>
```

`.ds-need-pain-grid`: 1col mobile / 2col tablet+ (grid-template-columns: 1fr 1fr), gap `--space-4`, margin-block `--space-10`. Children's `max-width` is reset so each insight fills its grid cell. Cells stretch to equal height for visual parity.

**Impact grid pattern** — three insight cards side-by-side for User / Business / KPI comparison (Result & Impact section). Use `.ds-impact-grid`:

```html
<div class="ds-impact-grid">
  <article class="ds-insight">          <!-- USER IMPACT (cream) -->
  <article class="ds-insight ds-insight--dark">  <!-- BUSINESS IMPACT -->
  <article class="ds-insight ds-insight--yellow"> <!-- EXPECTED KPI -->
</div>
```

`.ds-impact-grid`: 1col mobile / 2col tablet+ (768) / **3col desktop** (1440), gap `--space-4`, margin-block `--space-20`. Children's `max-width` reset to fill cell. Em-dash bullet lists (`<ul class="ds-numbered-step__list">`) inside each insight are recolored automatically per variant: cream/yellow → text-on-cream marker, dark → accent-blue marker.

### 7.4 `research-finding`

**Use**: Secondary Research, Primary Research, Analysis sections

**Background**: bg-page (no card surface — borderless flow).

**Base structure** (simple variant):
- `__eyebrow` "SECONDARY RESEARCH" (accent-blue)
- `__title` h2 (with Playfair Italic emphasis)
- `__body` paragraph
- `__grid` — 2-column image grid expecting `__figure` children (each with `__media` + `__caption`), 16:10, gap 16px

**Mobile**: image grid → 1 column stack.
**Tablet+**: 2-column image grid.
**Hover (on images)**: scale 1.02, caption brightens.

**Compound structure** (when a single finding contains multiple labeled sub-blocks like "Pattern" / "Observation" plus a media grid):

```html
<article class="ds-research-finding">
  <p class="ds-eyebrow ds-eyebrow--accent-blue">[ 02.1 — SECONDARY RESEARCH ]</p>
  <h2 class="ds-h2">Heading with <em class="ds-signature">emphasis</em></h2>

  <div class="ds-research-finding__media-grid">
    <div class="ds-media-placeholder" data-aspect="16/10">…</div>
    <div class="ds-media-placeholder" data-aspect="16/10">…</div>
  </div>

  <h3 class="ds-research-finding__sub-heading">Pattern</h3>
  <ul class="ds-research-finding__list">
    <li>…</li>
  </ul>

  <h3 class="ds-research-finding__sub-heading">Observation</h3>
  <p class="ds-research-finding__paragraph">…</p>
</article>
```

| Child | Spec |
| --- | --- |
| `__media-grid` | `display: grid`, 1col mobile / 2col tablet+, gap `--space-3` mobile / `--space-4` tablet, `margin-block: --space-10` (40 above/below). Differs from `__grid` — children are direct media boxes, not `__figure` with caption. |
| `__media-grid--3col` | Modifier on `__media-grid`. Tablet+ becomes 3-column equal (`repeat(3, 1fr)`). Mobile stays 1-column stack. Use when 3 figures need equal weight (e.g. interview sessions + protocol). |
| `__sub-heading` | `--text-h3-*` (22/18 mobile), text-primary, `margin-top: --space-20` (80 — strong section break), `margin-bottom: --space-6` (24). KR: `--fs-h3-kr` auto when `.ds-kr` on element or parent. |
| `__list` | em-dash markers in accent-blue, body size + secondary, gap `--space-3` (12), padding-left `--space-6` for marker. KR: Pretendard via `li.ds-kr` or `.ds-kr` parent. |
| `__paragraph` | body size + secondary, max-width 720px. Consecutive paragraphs separated by `margin-top: --space-6` (24). KR: Pretendard via element-self or parent `.ds-kr`. |
| `__implication-list` | Emphasis variant of `__list` for conclusion / takeaway statements. body-lg size + text-primary (vs body + secondary in `__list`). Pair each `<li>` with leading `<span class="ds-implication-arrow">→</span>` instead of em-dash. Margin-block `--space-6`. Use after a multi-paragraph analysis to surface "this means →" results. |

**Bilingual markup pattern** (`data-lang-content` + element-level `.ds-kr` for KR):
- EN element: `<h3 class="ds-research-finding__sub-heading" data-lang-content="en">Pattern</h3>`
- KR element: `<h3 class="ds-research-finding__sub-heading ds-kr" data-lang-content="ko">패턴</h3>`

### 7.5 `criteria-list`

**Use**: Numbered short-item lists (01/02/03 patterns)

**Structure** (3 columns):
- 8px yellow dot top-left
- eyebrow ("01" / "02" / "03")
- main text (Manrope SemiBold 18px, 1-2 lines)
- vertical hairline dividers between columns

**Background**: bg-surface, radius card.

**Mobile**: 3 columns → vertical stack (3 rows), horizontal dividers between.
**Tablet+**: 3 columns horizontal.

**Hover (per item)**: yellow dot 8→12px, text brightens, 1px accent-blue underline appears.

### 7.6 `feature`

**Use**: Solution Feature 01/02/03

**Structure** (image-led):
- top: full-width image (16:10), radius card-lg, image flush
- below: eyebrow + h2 with Playfair Italic emphasis + body + bullet list (em-dash —, accent-blue) + hairline + outcome line

**Background**: bg-surface. Radius: card-lg.

**Hover**: image scales 1.03 (400ms editorial easing); eyebrow brightens.

### 7.7 `before-after`

**Use**: Flow Change, Behavioral Change

**Structure** (2 columns INVERSE backgrounds):
- LEFT: bg-surface (BEFORE) — eyebrow text-tertiary, muted flow text
- RIGHT: card-cream (AFTER) — eyebrow text-on-cream, flow text with Playfair Italic emphasis
- CENTER: large arrow → (Playfair Italic 80px, accent-yellow)

**Border**: NONE. Outer radius: card-lg.

**Mobile**: 2 columns → vertical stack (BEFORE on top, AFTER below). Arrow rotates to ↓.
**Tablet+**: 2 columns horizontal with → arrow centered.

**Hover**: arrow shifts +4px horizontal; BEFORE column dims to 0.7 opacity.

### 7.8 `split-feature`

**Use cases**:
- Iteration showcase (text-heavy + small image)
- Solution Feature large version (balanced text + image)
- Research finding large version (image-led)
- Any "text + media" full-width section

**Structure** (horizontal split):
- Two columns separated by configurable ratio
- One side: text block (eyebrow + heading + body, optional bullet/outcome)
- Other side: media slot (image/video, ratio depends on variant)

**Modifier classes**:

Split ratio (required):
- `.ds-split-feature--50-50` — equal split (high-impact, image-led use)
- `.ds-split-feature--60-40` — text-heavy (60% text / 40% media, iteration pattern)

Media position (required):
- `.ds-split-feature--media-left`
- `.ds-split-feature--media-right`

Background (required):
- `.ds-split-feature--dark` (bg-surface)
- `.ds-split-feature--cream` (card-cream)

**Sizing**:
- Width: 100% of container (full-width section)
- Internal padding: 16px mobile / 24px tablet / 32px desktop *(reduced to maximize media slot)*
- Gap between columns: 24px mobile / 40px tablet / 56px desktop
- Media aspect ratio:
  - --50-50: 4:3 (more square, balanced)
  - --60-40: 4:3 or 3:2

**Radius**: card-lg (24px)

**Mobile** (<768px):
- All variants → vertical stack (text on top, media below)
- Media aspect ratio: 16:10

**Hover**:
- Eyebrow brightens
- Image scales 1.02x (400ms editorial easing)

**Markup pattern**:
```html
<article class="ds-split-feature ds-split-feature--50-50 ds-split-feature--media-left ds-split-feature--dark">
  <div class="ds-split-feature__media">
    <!-- image or placeholder -->
  </div>
  <div class="ds-split-feature__content">
    <p class="ds-eyebrow">FEATURE 01</p>
    <h2 class="ds-h2">
      <em class="ds-signature">One</em> screen — judge and choose together
    </h2>
    <p class="ds-body">...</p>
    <ul class="ds-split-feature__list">
      <li>Real-time cost projection</li>
      <li>Historical comfort data</li>
      <li>One-tap selection</li>
    </ul>
    <div class="ds-split-feature__outcome">
      <span class="ds-eyebrow">결과 / OUTCOME</span>
      <p class="ds-body-lg">Decisions made in under 10 seconds</p>
    </div>
  </div>
</article>
```

**Use case mapping (Operator page)**:
- Iteration 01/02 → `--60-40 --media-right --dark/--cream`
- Solution Feature 01/02/03 large → `--50-50 --media-left --dark` (alternating media-right/left for rhythm)

### 7.9 `stat`

**Use**: KPIs (30/40/15), SUS Score (76)
**Number style**: Playfair Italic (token), letter-spacing -0.05em.

**Variant A — Standalone (SUS 76)**:
- background: card-cream
- big number Playfair Italic (massive size)
- hairline divider 40px (1px accent-yellow)
- label (eyebrow style)
- caption (max 2 lines, 70% opacity)
- radius: card-lg

**Variant B — Grid (KPI 30/40/15)**:
- 3 columns INVERSE rhythm:
  - Item 1: card-cream / text-on-cream
  - Item 2: accent-yellow / text-on-cream
  - Item 3: bg-surface / text-primary
- Each: number Playfair Italic + "%" suffix Manrope SemiBold inline
- Hairline divider (32px), label

**Mobile (Variant B)**: 3 columns → vertical stack (3 rows), gap 16px.
**Tablet+**: 3 columns horizontal.

**Hover**: 1px accent-blue underline animates 0 → 40px below number (200ms ease).

### 7.10 `media`

**Use**: Standalone images, video slots, diagrams

**Structure** (no card chrome):
- optional figure label "Fig. 01" (eyebrow accent-blue)
- media slot (default 16:9, radius card, bg-muted placeholder)
- caption below (caption style, text-tertiary)

**Width variants**: `--full` | `--two-up` | `--inline` (see max-width table).

**Mobile (two-up)**: 2 columns → 1 column stack.

**Hover**: subtle scale 1.015 (400ms editorial easing), caption brightens.

---

## 8. Image Placeholder Spec (for Operator page — Thursday meeting)

For the content-review meeting, images are placeholder. Make placeholders **informative**, not blank.

```html
<div class="ds-media-placeholder" data-aspect="16/10">
  <span class="ds-eyebrow">FIG. 03</span>
  <p class="ds-media-placeholder__label">Affinity Mapping</p>
  <p class="ds-media-placeholder__meta">operator/approach/affinity-mapping.jpg</p>
</div>
```

```css
.ds-media-placeholder {
  background: var(--bg-muted);
  border: 0.5px dashed var(--border-hairline);
  aspect-ratio: 16 / 10;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  color: var(--text-tertiary);
  border-radius: var(--radius-card);
}
.ds-media-placeholder__label {
  font-size: var(--fs-body);
  color: var(--text-secondary);
}
.ds-media-placeholder__meta {
  font-size: var(--fs-caption);
  font-family: monospace;
  opacity: 0.6;
}
```

→ Reviewer immediately sees what image will go there. After meeting, swap with real images.

---

## 9. UI Elements

### Navigation rules (HARD RULE)

The site nav is **language-agnostic** — chrome elements never translate. Only body content responds to the language toggle.

| Element | Translates? | Notes |
| --- | --- | --- |
| `.ds-nav__logo` ("wizzy") | ❌ Fixed English (lowercase brand mark) | No `data-lang-content` attribute |
| `.ds-nav__tagline` ("Product designer") | ❌ Fixed English | No `data-lang-content` attribute. Sub-line under logo. |
| `.ds-nav-link` (PROJECT / ABOUT / RESUME) | ❌ Fixed English (uppercase) | Section labels stay English in both modes |
| `.ds-lang-toggle__btn` (EN / KO) | ❌ Fixed labels | The toggle itself doesn't translate; clicking sets `<html data-lang>` |

Body content (`<section>` everything below the nav) uses `data-lang-content="en"` / `data-lang-content="ko"` paired siblings — JS toggle on `.ds-lang-toggle__btn` swaps `<html data-lang>` and CSS reveals the active locale only.

**Toggle JS pattern** (page-local in `<script>`):
```js
const buttons = document.querySelectorAll('.ds-lang-toggle__btn');
const setLang = (lang) => {
  if (lang !== 'en' && lang !== 'ko') return;
  document.documentElement.dataset.lang = lang;
  document.documentElement.setAttribute('lang', lang);
  buttons.forEach(b => b.classList.toggle('ds-lang-toggle__btn--active', b.dataset.lang === lang));
  document.querySelectorAll('[data-lang-aware]').forEach(el => el.classList.toggle('ds-kr', lang === 'ko'));
};
buttons.forEach(b => b.addEventListener('click', () => setLang(b.dataset.lang)));
setLang(document.documentElement.dataset.lang || 'ko');
```

**Visibility CSS** (page-local):
```css
[data-lang-content="en"], [data-lang-content="ko"] { display: none; }
html[data-lang="en"] [data-lang-content="en"],
html[data-lang="ko"] [data-lang-content="ko"] { display: revert; }
```

### Typography utility classes

For markup that doesn't sit inside a BEM-scoped card, apply these utility classes for consistent typography. Each pairs with `.ds-kr` to auto-swap font-family to Pretendard and downscale to the matching `--fs-*-kr` token (h1/h2/h3/display) — body sizes keep the same px in KR.

| Class | Token | KR variant |
| --- | --- | --- |
| `.ds-display` | `--text-display-*` (96/56) | `--fs-display-kr` (80/48) |
| `.ds-h1` | `--text-h1-*` (64/40) | `--fs-h1-kr` (54/36) |
| `.ds-h2` | `--text-h2-*` (40/28) | `--fs-h2-kr` (34/26) |
| `.ds-h3` | `--text-h3-*` (22/18) | `--fs-h3-kr` (20/18) |
| `.ds-body-lg` | `--text-body-lg-*` (18/17) | same px, Pretendard |
| `.ds-body` | `--text-body-*` (16/15) | same px, Pretendard |
| `.ds-caption` | `--text-caption-*` (13) | same px, Pretendard |

These live in `components/ui.css`. Auto-apply rules for `.ds-kr` live in `components/cards.css` (alongside the heading auto-downscale block).

### UI Elements

Beyond cards, the following UI elements are needed. Specs from Figma `02 Components — UI Elements` frame.

| Element        | Class                              | Notes                                |
| -------------- | ---------------------------------- | ------------------------------------ |
| Tag            | `.ds-tag`                          | pill, 4 states (default/hover/active/disabled) |
| Button primary | `.ds-btn .ds-btn--primary`         | yellow filled, 3 states              |
| Button ghost   | `.ds-btn .ds-btn--secondary`       | transparent + border                 |
| Text link      | `.ds-link`                         | underline on hover                   |
| Divider        | `.ds-divider` + variants           | 5 variants (hairline / accent / etc) |
| Eyebrow        | `.ds-eyebrow` + color modifier     | uppercase caps, tracking             |
| Form input     | `.ds-input`                        | bottom-border only                   |
| Nav link       | `.ds-nav-link`                     | uppercase, 3 states · responsive: 12px mobile (`--text-counter-size`) / 14px desktop (≥768px, literal — no token) · letter-spacing keeps `--text-eyebrow-letter-spacing` |
| Implication arrow | `.ds-implication-arrow`         | inline `→` marker, `accent-yellow`, weight 600, `margin-right: --space-3`. Used inside `__implication-list <li>` or inside a pull-quote body for "→ takeaway" emphasis. |
| Divider — accent thin | `.ds-divider .ds-divider--accent-thin` | 0.5px (`--border-thin`) accent-yellow stripe, 40px wide (`--space-10`). Lighter version of `--accent-yellow`. Used as the SUS-style stat divider between number and supporting text. |

---

## 10. Section Header Pattern (Main Page)

Reusable across main page sections.

```html
<header class="ds-section-header" data-lang-aware>
  <p class="ds-section-header__eyebrow">[ SELECTED WORK ]</p>
  <h2 class="ds-section-header__title ds-h1">
    Selected <em class="ds-signature">Projects</em>
  </h2>
  <p class="ds-body-lg">Brief description, max 600px, 2 lines.</p>
</header>
```

**Eyebrow size**: section-header uses the **larger** eyebrow tokens (`--text-eyebrow-lg-*`) — 14px desktop / 12px mobile — *not* the standard 11px. The default `.ds-eyebrow` utility (Section 9) stays at 11px; section headers earn the larger size for hierarchy. Letter-spacing and weight remain the same as base eyebrow.

```css
.ds-section-header__eyebrow {
  font-size: var(--text-eyebrow-lg-size);          /* 14 / mobile 12 */
  line-height: var(--text-eyebrow-lg-line-height); /* 21 / mobile 18 */
  letter-spacing: var(--text-eyebrow-letter-spacing); /* 1.32, unchanged */
  font-weight: var(--text-eyebrow-weight);         /* 500, unchanged */
  text-transform: uppercase;
}
```

> **Location note**: The `.ds-section-header__*` rules currently live page-local (e.g. `projects/operator.html` style block). Once the pattern stabilizes, extract to `design-system/components/ui.css`.

**KR mode**: add `data-lang-aware` to the `<header>` so JS toggles `.ds-kr` and the title (with `.ds-h1`) auto-downscales to `--fs-h1-kr`.

Spacing:
- top of section: 80px mobile / 120px desktop
- below header: 40px mobile / 56px desktop

---

## 11. Critical Implementation Rules

### Hard rules (Claude Code MUST follow)

1. **NEVER hard-code values** — `font-size`, `padding`, `margin`, `gap`, `color`, `border-radius`, `border-width` use `var(--*)` tokens only
2. **NO drop shadows**, **NO gradients** on UI elements
3. **Hairline borders only** (0.5px or 1px max for component borders)
4. **Cards distinguish via background color contrast**, not borders
5. **Playfair Italic only for**: signature emphasis words, counters, stat numbers
6. **Korean = Pretendard**, emphasis = Pretendard Bold (NEVER Playfair Italic)
7. **Auto Layout / flexbox / grid** everywhere, NO absolute positioning
8. **Mobile-first**: write base for mobile, scale up via `@media (min-width: ...)`
9. **All new classes use `.ds-` prefix** to avoid collision with existing main page classes
10. **DO NOT modify main page** until Operator page is complete — touch index.html only after Thursday meeting

### Transition defaults

```css
/* Default hover (color, lift, scale ≤1.02) */
transition: all 250ms cubic-bezier(0.4, 0, 0.2, 1);

/* Image hover scale (≥1.02) */
transition: transform 400ms cubic-bezier(0.65, 0, 0.35, 1);

/* Underline animations */
transition: width 200ms ease;
```

### Alignment rules (HARD RULE)

**Default = LEFT.** Center is the exception, used only for visual impact moments where the content is essentially a number or symbol.

**Always LEFT** (do not center horizontally — neither `text-align: center` nor `align-items: center` on text-bearing flex containers):
- All `numbered-step` children (counter, caption, divider, title, body, quote, list, closing)
- `pull-quote`, `insight`, `research-finding`, `criteria-list` columns, `feature`, `before-after` LEFT/RIGHT columns (text within), `split-feature` content, `iteration` text
- `ds-section-header` (eyebrow, h1/h2, body)
- Short decorative dividers under headings: `margin-left: 0` (no `margin: 0 auto`)
- Any `<ul>` bullet list, body paragraph, eyebrow + heading + body composition

**Always CENTER** (intentional impact moments only):
- `stat` Variant A standalone (the SUS-76 number) — `align-items: center; text-align: center` on the card container
- `stat` Variant B grid items — same pattern per cell
- `before-after` middle arrow `→` (or `↓` on mobile)

**Vertical centering** (`justify-content: center` on column flex, `align-items: center` on grid for cell-axis alignment) is layout, not text alignment — allowed where it improves balance between adjacent media+text columns (e.g., `split-feature`).

**Card horizontal positioning** (`margin-inline: auto` on the card itself when narrower than container) is independent of internal alignment — keep it for cards with desktop max-widths.

### Lottie compatibility

Main page uses Lottie JSON animations. The design system DOES NOT touch Lottie:
- Lottie containers can use `var(--*)` for `width`, `max-width`, `margin`, `padding`, `border-radius`
- Lottie's own JS-injected classes are NEVER touched
- Animation trigger classes (e.g., `data-animate`, `.fade-in`) are NEVER renamed

---

## 12. File Structure

```
/Wizzy/
├── index.html                          # main page — DO NOT TOUCH until after meeting
├── projects/
│   └── operator.html                   # build new with system from scratch
├── _content/
│   └── operator.md                     # content source (Korean primary, English secondary)
├── assets/
│   ├── css/
│   │   ├── design-system.css           # Phase 2: tokens + fonts only
│   │   └── components.css              # Phase 2: card 10 variants + UI elements
│   ├── js/                             # existing animations preserved
│   └── projects/operator/
│       ├── img/                        # to be added later
│       └── video/                      # to be added later
├── templates/
│   └── project-page.html               # Phase 5 (post-meeting)
├── _archive/                           # DO NOT READ (.claudeignore)
├── .claudeignore
├── CLAUDE.md                           # working rules
├── DESIGN_SYSTEM.md                    # this file
└── PLAN.md                             # phase plan (optional)
```

---

## 13. Component Build Priority

For Thursday meeting, build cards in this priority order.

### Tier 1 (essential — Operator page can't ship without these)
1. `numbered-step` (used 10+ times: Overview 5, Reflection 5)
2. `pull-quote` (scattered through every section)
3. `feature` (Solution 3 features — main visual moment)
4. `stat` (KPI grid + SUS 76 — impact moment)

### Tier 2 (high-priority — Approach section depends on these)
5. `research-finding` (4 instances)
6. `criteria-list` (Analysis 3-item patterns)
7. `before-after` (Flow Change, Behavioral Change)

### Tier 3 (supporting — finishing touches)
8. `split-feature` (Iteration 01/02 + large Solution Feature variants)
9. `insight` (standalone callouts)
10. `media` (image slots — placeholder for meeting)

Plus UI: `tag`, `button` (primary + secondary), `divider`, `eyebrow`, `nav-link`.

---

## 14. Build Sequence Reference

```
Phase 2 — Code system
  Step 1: design-system.css (tokens + fonts only) ← Figma MCP "01 Tokens"
  Step 2: components.css Tier 1                    ← Figma MCP "02 Components" + this doc
  Step 3: components.css Tier 2
  Step 4: components.css Tier 3 + UI Elements

Phase 3 — Operator page build (Thursday morning 10:00)
  - HTML skeleton + nav + sections
  - Hero + Overview (Tier 1 cards)
  - Approach (Tier 2 cards)
  - Define + Ideate
  - Solution (Tier 1 feature cards + Tier 1 stat grid)
  - Outcome + Iteration + Reflection
  - More Projects
  - Image placeholders (Section 8 spec)
  - Korean/English toggle (data-lang attribute)

Phase 4 — Main page token injection (POST-MEETING)
  - Add CSS link (no visual change)
  - Replace hard-coded values with var(--*)
  - Touch testimonials/skill tags (problem sections)
  - DO NOT touch Lottie animations or trigger classes

Phase 5 — Template extraction (later)
```

---

## 15. Verification Checklist

Claude Code self-verification:

**For design-system.css**:
- [ ] All Figma `01 Tokens` Variables included
- [ ] No hard-coded values
- [ ] Font @import statements present
- [ ] Responsive media queries (768/1440/1920) included
- [ ] Mobile typography downscale included
- [ ] CSS variable naming matches Figma hierarchy

**For components.css**:
- [ ] Every value uses `var(--*)`
- [ ] Each card has its max-width per Section 6 table
- [ ] Hover states use specified transitions
- [ ] Mobile transformations applied (vertical stacks where specified)
- [ ] All classes use `.ds-` prefix
- [ ] Korean variants use Pretendard, no Playfair Italic on Korean text

**For Operator page**:
- [ ] Both `design-system.css` and `components.css` linked
- [ ] All sections present (Hero, Overview, Approach, Define, Ideate, Solution, Outcome, Iteration, Reflection, More)
- [ ] Image placeholders informative (Section 8)
- [ ] Both English and Korean text present (with data-lang attributes)
- [ ] No hard-coded values in HTML inline styles
