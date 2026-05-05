# Design system

The instructions for how the site looks and how to extend it. If you're adding
or changing visual style, this is the contract — follow it, don't invent next
to it.

## Font

**Google Sans Flex** — a single variable font file with six axes:

| Axis | Range | What it controls | How to set it |
| --- | --- | --- | --- |
| `wght` | 1 → 1000 | Weight (thin → black) | `font-weight: <n>` |
| `wdth` | 25% → 151% | Width (condensed → expanded) | `font-stretch: <pct>` |
| `slnt` | -10° → 0° | Slant (oblique italic) | `font-style: oblique <deg>deg` |
| `opsz` | 6 → 144 | Optical size (refines stroke for size) | `font-optical-sizing: auto` (handled globally) |
| `GRAD` | 0 → 100 | Grade (weight without changing width) | `--axis-grad` |
| `ROND` | 0 → 100 | Roundness (geometric → soft) | `--axis-rond` |

**Use the standard CSS properties for `wght`/`wdth`/`slnt`/`opsz`.** Don't put
those axes in `font-variation-settings` — doing so silently overrides the
shorthands and breaks composition. Only `GRAD` and `ROND` go through
`font-variation-settings`, driven by `--axis-grad` and `--axis-rond`.

Optical sizing is **automatic**: `font-optical-sizing: auto` is set on `:root`,
so the browser tracks `opsz` to the current font-size. You don't set it per
step.

## Type scale

Seven steps. Sizes scale fluidly with viewport width — no media queries.

| Step | Use |
| --- | --- |
| `display` | Hero on the homepage. The single biggest piece of type on the page. One per page max. |
| `h1` | Page title. Appears once per page, near the top. |
| `h2` | Major section heading (the parts of a page you'd link to). |
| `h3` | Subsection heading inside an `h2` section. |
| `body-lg` | Lede paragraph or pulled quote — body text that wants emphasis without being a heading. |
| `body` | Default paragraph and inline copy. Almost everything is this. |
| `small` | Captions, metadata, post dates, footnotes. |

Steps live in [src/styles/tokens.css](src/styles/tokens.css) as `--fs-<name>`.
Element rules and utility classes (`.display`, `.body-lg`, `.small`) live in
[src/styles/typography.css](src/styles/typography.css).

### How to choose a step

- Reach for **semantic HTML first** (`<h1>`, `<h2>`, `<p>`, `<small>`). The
  element rules apply the right step automatically.
- Use a **utility class** only when the visual weight needs to differ from the
  semantic level — e.g., a homepage hero `<h1>` styled as `.display`, or a
  footer `<p>` styled as `.small`.
- Don't introduce a new step casually. Seven covers nearly every case. If you
  genuinely need one (e.g., a `caption` between `body` and `small`), add it to
  `tokens.css` with min/max values, then add a class in `typography.css`.

### Color by step

Each step has a fixed color tier. This is **baked into the element rules and
utility classes** — semantic HTML gets it for free, and utilities are
self-contained so they carry their color when applied anywhere.

| Step | Color token | Tier |
| --- | --- | --- |
| `display` | `--color-text` | primary |
| `h1`–`h6` | `--color-text` | primary |
| `body-lg` | `--color-text-muted` | secondary |
| `body` | `--color-text-muted` | secondary |
| `small` | `--color-text-muted` | secondary |

The rule is simple: **every heading uses primary, every body step uses
secondary.** Headings stand out from running text by being darker, not just
larger or heavier.

To override locally — e.g., a meta line that should pop as primary — set
`color: var(--color-text)` on that one element.

## How the type is responsive

Every step is a `clamp(min, fluid, max)`:

- **At viewport width ≤ 360px** → size is at its minimum.
- **At viewport width ≥ 1440px** → size is at its maximum.
- **In between** → size scales linearly with viewport width.

The preferred value is a browser-compatible linear `calc()` expression derived
from the 360px and 1440px anchors. Every step is:

```css
clamp(min, calc(intercept + slope * 100vw), max)
```

Don't write breakpoint media queries for type — the system handles every width
continuously.

To change the responsive range, edit `--vw-min` and `--vw-max` in
[src/styles/tokens.css](src/styles/tokens.css). To change a step's sizes,
edit its `--fs-<name>-min`, `--fs-<name>-max`, and preferred `calc()`.

## Spacing

Six steps. Spacing scales fluidly across the same viewport range as type, so
layout rhythm and typography breathe together.

| Step | 360px → 1440px | Use |
| --- | --- | --- |
| `--space-2xs` | 4px → 8px | Hairline gaps, shell insets, micro-adjustments. |
| `--space-xs` | 8px → 16px | Tight gaps, icon/text spacing, compact padding. |
| `--space-sm` | 16px → 24px | Compact padding, tight stacks, small component gaps. |
| `--space-md` | 24px → 40px | Default page gutter, card padding, paragraph rhythm. |
| `--space-lg` | 48px → 80px | Component-internal sections and medium page breaks. |
| `--space-xl` | 64px → 128px | Hero padding and major page-section rhythm. |

Every step is a `clamp(min, fluid, max)` with the same linear preferred-value
pattern as type:

```css
clamp(min, calc(intercept + slope * 100vw), max)
```

Adding a spacing step requires two numbers (`--space-<name>-min` and
`--space-<name>-max`), a preferred `calc()`, and a documented use case. Don't
add breakpoint media queries for spacing.

### How to choose a step

- Use `xs` for tiny gaps that should still breathe a little across viewports.
- Use `sm` for compact local spacing inside controls or tight groups.
- Use `md` as the default practical spacing step, including container gutters.
- Use `lg` for larger component sections or meaningful breaks within a page.
- Use `xl` for the biggest vertical rhythm: heroes and major page sections.

## Layout

Use `.container` for constrained page content:

```html
<section class="container">…</section>
```

It sets the max width to `--container-max` (1000px), applies centered inline
margins, and subtracts `--container-gutter` (`--space-md`) from both viewport
edges. Full-width sections can still wrap an inner `.container`.

### How to build a layout

A layout component is a page section pattern. It should own the outer
`<section>`, the `.container`, and the vertical spacing for the whole block.
Pages should compose layout components directly instead of wrapping them in
one-off page-level sections.

Use this default shape:

```astro
---
import SectionTitle from "./SectionTitle.astro";
---

<section class="container example-layout">
  <SectionTitle title="Section title" />

  <div class="example-layout__content">
    ...
  </div>
</section>

<style>
  .example-layout {
    display: grid;
    gap: var(--space-md);
    padding-block: var(--space-lg);
  }

  .example-layout__content {
    ...
  }
</style>
```

The rule: **section owns spacing, content owns arrangement.**

- The outer `<section>` gets `class="container <layout-name>"`.
- The outer section owns vertical rhythm with `padding-block`.
- The outer section owns spacing between the section title and content with
  `gap`.
- The inner content element owns layout details: grid columns, flex direction,
  card gaps, max-widths, alignment, etc.
- Do not add extra wrappers around layout components in page files just to add
  margins or padding. Put that spacing inside the layout component.
- Use `--space-lg` for normal section padding unless the section has a clear
  reason to be tighter or larger.
- Use `--space-md` between a `SectionTitle` and the section content by default.

Pages should stay simple:

```astro
<BaseLayout title="Home" heroTitle="Carter Shades">
  <Text title="Intro" text="..." />
  <CardGrid title="Work">...</CardGrid>
</BaseLayout>
```

Avoid this pattern:

```astro
<section class="container custom-page-spacing">
  <CardGrid title="Work">...</CardGrid>
</section>
```

That splits layout responsibility between the page and component, which makes
spacing drift.

### Hero

Every page gets a Hero component automatically — it's rendered by
[BaseLayout](src/layouts/BaseLayout.astro) between `<Header>` and `<main>`,
driven by the optional `heroTitle` prop that BaseLayout takes for the visible
page heading. Pages do not include a Hero or `<h1>` themselves.

The Hero is a centered `.container` + `.display` styled `<h1>` with
`--space-xl` padding above and below. The `<h1>` is semantic so the page still
has exactly one top-level heading; `.display` provides the larger visual size,
and `.container` keeps the title aligned to the same content width as the rest
of the page.

Hero titles use the cursor-responsive variable font effect. The component
splits the title into individual character spans, keeps the original title as
the accessible label, and animates each character from the Display resting
state, `font-weight: 600` / `font-stretch: 100%`, up to `font-weight: 800` /
`font-stretch: 130%` as the cursor gets nearby. The swell should feel broad and
soft across a few characters, not like a hard single-letter spotlight.

Nearby characters also get a smooth RGB glitch shadow behind the primary text:
red, blue, and yellow `text-shadow` layers fade in and spread outward based on
the same cursor proximity value. This is proximity-based, not random flicker,
so it should feel like a reactive chromatic echo.

Each Hero title line stays on one line while it animates so variable-width
changes do not reflow the heading. Cursor distance is calculated from each
character's resting center point, not its live animated position, to avoid
jitter from the letters chasing their own layout changes.

To change the visible page heading without changing the browser tab title,
pass `heroTitle` to BaseLayout:

```astro
<BaseLayout title="Work" heroTitle="Carter Shades">
  ...
</BaseLayout>
```

For intentional line breaks, pass an array. Each item becomes one stable line:

```astro
<BaseLayout
  title="Home"
  heroTitle={[
    "Carter Daniel Shades.",
    "Product designer,",
    "gardener, builder.",
  ]}
>
  ...
</BaseLayout>
```

If `heroTitle` is omitted, BaseLayout falls back to `title`. Don't add a second
`<h1>` to the page body — the hero owns it.

### Page transitions

BaseLayout uses Astro's `<ClientRouter>` for same-origin page navigation.
Primary nav links use `data-astro-prefetch="load"` so the main pages are warmed
up as soon as the nav renders.

Page content is wrapped in `.page-transition-content`. `<PageTransition>` starts
the fade immediately from a capture-phase same-site link click, then also
listens to Astro's navigation lifecycle as a fallback. Before Astro swaps in
the next document, it marks the incoming body as entering; after the swap, the
class is removed so the new content fades in. Header and the background remain
outside the faded content wrapper.

### Section title

`<SectionTitle>` renders an `<h3>` left-aligned with no margin. Use it as the
heading at the top of any section that lives inside a layout (a card grid, a
list of writing posts, a column of bio copy, etc.).

```astro
<SectionTitle title="Selected work" />
```

It does not own spacing. The parent layout section owns spacing with `gap`, so
title-to-content rhythm is visible in one place and stays consistent across
layouts.

### Card grid

`<CardGrid>` is a two-column grid that collapses to one column at `≤640px`.
`<Card>` is each tile: a 4:5 image with a glass label in the top-left
containing a heading title and a `<p>` subtitle. Cards link somewhere when
given an `href`, or render as a static `<article>` when not.

CardGrid also takes an optional `title` prop. When set, it renders a
`<SectionTitle>` above the grid. The outer section uses `padding-block:
var(--space-lg)` and `gap: var(--space-md)`, so the title and grid follow the
standard layout pattern.

```astro
---
import CardGrid from "../components/CardGrid.astro";
import Card from "../components/Card.astro";
import atempo from "../assets/atempo.jpg";
import workingSets from "../assets/working-sets.jpg";
---

<CardGrid title="Selected work">
  <Card
    title="Atempo"
    subtitle="Your digital notebook for daily planning"
    image={atempo}
    imageAlt="Atempo app on a stone surface"
    href="/work/atempo"
  />
  <Card
    title="Working Sets"
    subtitle="Track your working sets, simply"
    image={workingSets}
    imageAlt="Working Sets timer running on a phone"
    href="/work/working-sets"
  />
</CardGrid>
```

**Images live in `src/assets/`** and are imported as ES modules so Astro's
build-time `<Image>` component generates responsive `srcset`, modern formats
(AVIF / WebP with JPEG fallback), and lazy loading automatically. Don't put
project images in `public/` — that bypasses optimization.

**Hover behavior.** On hover-capable devices above the mobile breakpoint
(`@media (hover: hover) and (min-width: 641px)`), the label is hidden by
default and fades in (300ms) when the card is hovered or keyboard-focused
— the image stays clean until the visitor expresses interest. Touch devices
*and* small viewports (`≤640px`) keep the label visible always, so a small
laptop window or a phone never hides the title behind an invisible interaction.
The image also subtly zooms (`scale(1.04)`, 400ms) on hover. Both transitions
are removed when `prefers-reduced-motion: reduce` is set.

**The breakpoint** (`≤640px`) is the same value used for the bottom-nav rule.
Sizing within the card (label padding, type, gutters) remains fluid.

## Line-height, balance, and wrapping

- Line-height tightens at larger sizes. Display: `1.05`. Body: `1.55`. The
  values live in `tokens.css` as `--lh-<name>`.
- Headings (`h1`–`h6`, `.display`) use `text-wrap: balance` — distributes
  words evenly across lines so headlines don't end with one orphaned word.
- Body text (`body`, `p`) uses `text-wrap: pretty` — avoids single-word last
  lines without rebalancing the whole paragraph.

## Weight, width, slant — defaults and overrides

- Default body weight: `400`.
- Default heading weight: `600`.
- Width and slant default to neutral (`100%`, `0deg`).
- Override locally with the standard CSS properties:
  ```css
  .label { font-weight: 500; font-stretch: 90%; }
  em { font-style: oblique -8deg; }
  ```
- **Don't** set these via `font-variation-settings` — use the shorthand
  properties so they compose with the rest of the system.

## Grade and roundness — the design levers

`GRAD` and `ROND` are the two axes that don't have standard CSS shorthands,
so they live as custom properties on `:root`:

```css
--axis-grad: 0;  /* 0 = neutral, 100 = heavier without widening */
--axis-rond: 0;  /* 0 = geometric, 100 = soft and rounded */
```

Both default to `0`. Tune them globally in `tokens.css` once you've seen the
type rendered, or override locally on a selector for a specific accent.

## Color

Two-tier OKLCH token system.

### Tier 1 — primitives

The small palette of underlying values. **Components must not reference these
directly.**

| Token | Source | Role |
| --- | --- | --- |
| `--ink` | `#191919` | Primary — headlines and core text. Maximum readability and a sense of permanence. |
| `--canvas` | `#F8F5EE` | Neutral — page background. Softer, more organic than pure white. |
| `--accent` | `var(--ink)` *(placeholder)* | Tertiary — the sole driver for interaction. Currently matches the primary color; links rely on the default underline for affordance. Will be replaced with a dedicated color later. |
| `--accent-hover` | derived (`accent` mixed 35% toward `canvas`) | Hover state for interactive elements. |
| `--accent-active` | derived (`accent` mixed 60% toward `canvas`) | Pressed / active state. |
| `--pop-red` | OKLCH (theme-aware) | Decorative red accent for the colored stars. Lighter in light mode, darker in dark mode. |
| `--pop-blue` | OKLCH (theme-aware) | Decorative blue accent (same uses as `--pop-red`). |
| `--pop-yellow` | OKLCH (theme-aware) | Decorative yellow accent (same uses as `--pop-red`). |

OKLCH is the source of truth. Hex values above are documentation only —
sRGB→OKLCH conversion happens once in `tokens.css`.

### Tier 2 — semantic

The only color layer components are allowed to reference. Names describe
**role**, not appearance, so the same name resolves correctly in light and
dark mode.

| Token | Resolves to | Use for |
| --- | --- | --- |
| `--color-bg` | `--canvas` | Page background. |
| `--color-text` | `--ink` | Body and heading text. Default `color` on `<body>`. |
| `--color-text-muted` | `--ink` @ 80% α | Captions, metadata, post dates, footnotes — utilitarian text. |
| `--color-border` | `--ink` @ 80% α | Hairlines, dividers, input borders. |
| `--color-link` | `--accent` | `<a>` color and any primary action. |
| `--color-link-hover` | `--accent-hover` | `:hover` on links and buttons. |
| `--color-link-active` | `--accent-active` | `:active` on links and buttons. |
| `--color-focus` | `--accent` | Outline color for `:focus-visible`. |
| `--color-selection-bg` | `--accent` @ 20% α | `::selection` background. |
| `--color-selection-fg` | `--ink` | `::selection` text. |
| `--color-pop-red` | `--pop-red` | Decorative pop color. **Not for text or interactive elements** — contrast is not guaranteed. Used by the colored stars. |
| `--color-pop-blue` | `--pop-blue` | Decorative pop color (same constraint as red). |
| `--color-pop-yellow` | `--pop-yellow` | Decorative pop color (same constraint as red). |

`--color-text-muted` and `--color-border` resolve to the same value by design
(your "secondary" color is one primitive serving multiple semantic roles).
Keeping them as separate names lets you split them later without changing
every callsite.

### Dark mode

Activated by `prefers-color-scheme: dark`. The semantic layer (Tier 2) does
not change — it references the primitives, so swapping primitives swaps the
whole theme. The dark-mode override is just `ink ↔ canvas`:

```css
@media (prefers-color-scheme: dark) {
  :root {
    --ink:    oklch(0.9705 0.0098 87.47); /* was canvas */
    --canvas: oklch(0.2134 0 0);          /* was ink */

    --pop-red:    oklch(0.55 0.20 25);
    --pop-blue:   oklch(0.50 0.18 250);
    --pop-yellow: oklch(0.70 0.18 95);
  }
}
```

The pop colors invert in lightness (lighter in light mode, darker in dark
mode) so they stay visible without overpowering the canvas. The star canvas
reads them with `getComputedStyle(...).getPropertyValue(...)` and re-reads
on `prefers-color-scheme` change so a system theme flip updates colored
stars without a reload.

`--accent` currently follows `--ink` (placeholder), so it inherits the swap
automatically and contrast is preserved (16.15:1 in both modes).

When the real accent is introduced, this block will need a dedicated `--accent`
override and likely inverted hover/active deltas (in dark mode, *brighter* =
more prominent).

### Rules

- **Components reference Tier 2 only.** Never `var(--ink)`, never `oklch(…)`
  inline, never a hex code.
- **Don't add a new primitive without a reason.** The whole palette is three
  base colors plus two derived states.
- **Don't bypass the semantic layer.** If you need a new role (e.g., a
  warning color), add it as a Tier 2 token in `tokens.css` and document it
  here.

## Radius

Four fixed radius tokens. Values are not fluid; they describe reusable shape
roles and should be referenced directly from components.

| Token | Value | Use |
| --- | --- | --- |
| `--radius-sm` | `0.5rem` / 8px | Compact UI, buttons, tags. |
| `--radius-md` | `1rem` / 16px | Standard panels and grouped controls. |
| `--radius-lg` | `1.5rem` / 24px | Larger surfaces, cards, and glass panels. |
| `--radius-full` | `62.5rem` / 1000px | Pills, avatars, and fully rounded controls. |

Don't hardcode `border-radius` values in components. Use a `--radius-*` token
so the site's shape language stays consistent.

## Surfaces

### `.surface-glass`

Frosted-glass utility — a CSS-only approximation of iOS Liquid Glass. Apply to
any element to give it a translucent, blurred backdrop with a soft top
highlight and drop shadow.

```html
<div class="surface-glass">…</div>
```

**What it gets you:** translucent canvas-tinted fill, ~20px backdrop blur,
saturation boost (so colors behind read more vividly through the glass), a
hairline border that defines the surface, an inset top-edge highlight, and a
soft drop shadow.

**Theme-aware.** Tint and border derive from `--canvas` and `--ink`, so the
glass automatically inverts in dark mode. The highlight and shadow are fixed
light/dark — they simulate physical light reflection, not theme color.

**Where it works.** `backdrop-filter` only blurs content actually behind the
element, so this is intended for surfaces that overlap other content:

- Sticky nav over scrolled page content.
- Cards floating over a hero image, gradient, or scenic background.
- Modal backdrops.

Over a flat background it renders as a translucent canvas-tinted card with a
hairline border — still pleasant, but no "glass" effect because there's
nothing to refract.

**What's intentionally omitted.** True edge refraction (the bend at the
curve, like Apple's Liquid Glass) is not done here. It requires SVG
displacement filters or WebGL and doesn't ship reliably across browsers
through `backdrop-filter`. If you ever need it, treat it as a separate,
heavier feature.

## Atmospheric Background

`<StarBackground>` is mounted once in BaseLayout, before the page content. It
uses an absolute top-of-page canvas with `pointer-events: none`, so it never
takes layout space or blocks interaction. It scrolls away with the top of the
page rather than staying fixed to the viewport.

The star layer uses `--color-text-muted`, fades out toward the bottom of the
viewport with a mask, and stays behind content through the global body stacking
rules in `global.css`.

Stars are generated once in a wide fixed virtual sky (`4096px` × `1200px`).
Window resize only changes the centered viewport crop, so the pattern does not
randomly regenerate or jump around while resizing.

Stars are mostly tiny circles, with roughly 10% four-point sparks. About 30%
of stars are randomly assigned red, blue, or yellow; the rest use
`--color-text-muted`. They twinkle gently between 10% and 30% opacity. Nearby
stars softly react to the cursor within an approximately 180px radius by
growing up to 1.8x and brightening up to 30% opacity. Stars brighten/grow
quickly, then dim/shrink more slowly to leave a soft cursor trail.
Reduced-motion mode renders a static star field.

## File map

```
public/fonts/GoogleSansFlex.woff2   ← the variable font, Latin subset, ~1.4MB
src/styles/
├── fonts.css       ← @font-face only
├── tokens.css      ← --vw-*, --fs-*, --space-*, --container-*, --lh-*, --axis-*, --radius-*, --color-*
├── typography.css  ← type element styles + utility classes
└── global.css      ← reset + element color styles + imports the rest
```

Imported once in [src/layouts/BaseLayout.astro](src/layouts/BaseLayout.astro).
The font file is also `<link rel="preload">`-ed in the same layout so it
starts downloading before the CSS is parsed.

## Rules

- Don't hardcode `font-size`, `line-height`, font axis values, or color
  values in components. Use the tokens.
- Don't hardcode `margin`, `padding`, or `gap` values in components. Use
  `--space-*` tokens.
- Don't hardcode `border-radius` values in components. Use `--radius-*`
  tokens.
- Don't add CSS breakpoint media queries for type or spacing — they're already
  fluid.
- Don't introduce new spacing steps without documenting the use case.
- Don't put `wght` / `wdth` / `slnt` / `opsz` inside `font-variation-settings`.
- Don't introduce a second font family without updating this doc first.
- Don't reference Tier 1 color primitives from components — Tier 2 only.
