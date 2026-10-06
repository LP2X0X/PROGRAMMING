---
tags:
  - css
  - learning-path
  - roadmap
---

This roadmap organizes CSS topics into four progressive stages. Each builds on the previous -- skip ahead only if you can confidently explain everything in the prior stage.

**How to use this note:** Work through each stage top-to-bottom. Check off topics as you complete them. Items linked with `[[wikilinks]]` already have notes in this vault. Items marked with **(gap)** are topics worth creating notes for.

---

## Stage 1 -- Beginner (Foundations)

*Goal: Understand how CSS works before worrying about what it can do.*

The order here matters. You need to understand the cascade and specificity before writing any meaningful CSS, because without them you will constantly be confused about why your styles "aren't working."

### 1.1 How CSS Works

- [ ] [[The purpose of CSS]]
- [ ] [[CSS Rule]] -- selectors, properties, values
- [ ] [[1 - The Cascade|The Cascade]] -- origin, specificity, source order
- [ ] [[2 - Inheritance|Inheritance]] -- which properties pass down, which don't
- [ ] [[CSS Specificity]] -- how to calculate it, when it bites you
- [ ] [[Browser (User Agent) Style|User Agent Styles]] -- what the browser gives you for free

### 1.2 Selectors (Basics)

- [ ] [[Element Selector]], [[Class Selector]], [[Id Selector]]
- [ ] [[Universal Selector]]
- [ ] [[Attribute Selector]]
- [ ] [[Pseudo Selector]] -- overview of pseudo classes vs pseudo elements
- [ ] [[Compound Selector]], [[Listing Selector]]

### 1.3 The Box Model

- [ ] [[3 - Box Model|Box Model]] -- content, padding, border, margin
- [ ] [[box-sizing]] -- `content-box` vs `border-box`
- [ ] [[width]], [[height]], [[margin]], [[margin-inline]]
- [ ] [[max-height and max-width]], [[min-height and min-width]]
- [ ] [[Block Element]], [[Inline Element]], [[Inline Block Element]]

### 1.4 Common Properties

- [ ] [[color]], [[font-size]], [[line-height]], [[text-align]]
- [ ] [[letter-spacing]], [[white-space]]
- [ ] [[font]] (shorthand)
- [ ] [[outline]], [[overflow]]
- [ ] [[list-style]], [[list-style-type]], [[list-style-position]], [[list-style-image]]
- [ ] [[object-fit]], [[object-position]]

### 1.5 Units

- [ ] [[Length]] -- what a "length" means in CSS
- [ ] [[px]] -- absolute unit baseline
- [ ] [[rem and em]] -- relative to root vs parent font size
- [ ] [[vh and vw]], [[dvh]], [[svh and lvh]], [[vmin and vmax]]
- [ ] [[Unitless number]] -- when and why to omit the unit
- [ ] [[CSS Units Overview]]

### 1.6 Keywords and Values

- [ ] [[auto]]
- [ ] [[inherit]], [[initial]], [[unset]]
- [ ] [[!important]] -- why to avoid it and when you can't
- [ ] [[Shorthand Property]] -- how shorthand resets values you didn't set
- [ ] [[Cascaded Value]]
- [ ] [[Initial Values of CSS Properties]]

```ad-summary
title: Stage 1 Checkpoint
Before moving on, you should be able to:
- Predict which style wins when two rules conflict
- Explain the box model from memory
- Choose the right unit for a given situation (`rem` vs `px` vs `%`)
- Read any basic CSS and understand what it targets and why
```

---

## Stage 2 -- Intermediate (Layout and Control)

*Goal: Control where things go on the page. This is where CSS goes from "I can style text" to "I can build interfaces."*

Layout is the single largest skill gap between beginners and competent CSS developers. Invest heavily here.

### 2.1 Display and Normal Flow

- [ ] [[Normal Document Flow]] -- how block and inline elements behave by default
- [ ] [[display]] -- `block`, `inline`, `inline-block`, `none`, `flex`, `grid`
- [ ] [[Formatting Context]] -- BFC, IFC, and why they matter
- [ ] [[Intrinsic and Extrinsic]] sizing

### 2.2 Flexbox

- [ ] [[Flexbox]] -- the mental model (main axis vs cross axis)
- [ ] Container: [[flex-direction]], [[flex-wrap]], [[gap]]
- [ ] Items: [[flex-grow]], [[flex-shrink]], [[flex-basis]], [[flex]] shorthand
- [ ] Items: [[order]]
- [ ] Alignment: [[justify-content]], [[align-items]], [[align-self]]
- [ ] [[Flex Cheatsheet]]
- [ ] [[Different between align-items and justify-content]]
- [ ] [[min-height in Flex]] (pitfall)

### 2.3 Grid

- [ ] [[Grid]] -- the mental model (rows, columns, cells, areas)
- [ ] Container: [[grid-template-columns]], [[grid-template-rows]]
- [ ] Container: [[row-gap and column-gap]]
- [ ] Items: [[grid-column and grid-row]], [[justify-self]]
- [ ] [[Grid Property Overview]]
- [ ] Functions: [[repeat]]
- [ ] [[Shared between Grid and Flexbox|Shared Alignment Properties]] -- [[place-items]]
- [ ] [[Grid Cell Centering]] (technique)

### 2.4 Positioning

- [ ] [[position Property]] -- `static`, `relative`, `absolute`, `fixed`, `sticky`
- [ ] [[Absolute Position Properties]] -- `top`, `right`, `bottom`, `left`
- [ ] [[The Containing Block]] -- what `absolute` positions relative to
- [ ] [[position fixed vs position absolute]]
- [ ] [[offset Properties]]
- [ ] Stacking context **(gap)** -- `z-index` behavior and stacking order

### 2.5 Float (Legacy Knowledge)

- [ ] [[float]], [[clear]]
- [ ] [[Clearfix Hack]]

### 2.6 Selectors (Advanced)

- [ ] Combinators: [[Descendant Combinator]], [[Child Combinator]]
- [ ] Combinators: [[Adjacent Sibling Combinator]], [[General Sibling Combinator]]
- [ ] [[Combinator Overview]]
- [ ] [[Nesting Selector]]

### 2.7 Pseudo Classes and Elements

- [ ] [[Pseudo Class]] -- `:hover`, `:focus`, `:active`, `:visited`
- [ ] [[active]], [[link]], [[visited]]
- [ ] [[first-child]], [[root]]
- [ ] [[Chaining pseudo classes]]
- [ ] [[Psuedo Class Overview]]
- [ ] [[before]], [[after]] -- `::before`, `::after`
- [ ] [[marker]] -- `::marker` for lists
- [ ] [[Pseudo-classes vs Pseudo-elements]] (question)

### 2.8 Functions

- [ ] [[var]] -- CSS custom properties / variables
- [ ] [[Custom Properties (CSS variables)]]
- [ ] [[calc]] -- mixing units
- [ ] [[clamp]] -- responsive sizing without media queries
- [ ] [[CSS Value Function Overview]]

### 2.9 At-Rules

- [ ] [[@supports]] -- feature detection
- [ ] [[media query]] -- responsive breakpoints
- [ ] `@font-face` **(gap)** -- custom font loading
- [ ] `@import` **(gap)** -- and why you usually should avoid it

### 2.10 Common Techniques

- [ ] [[Magic Centering]] and the [[Centering]] folder
- [ ] [[Margin Tricks]]
- [ ] [[Double container pattern]]
- [ ] [[Styling List]]
- [ ] [[Fallback Font]]
- [ ] [[Multiple Classes on a Single HTML Element]]
- [ ] [[BEM]] naming convention
- [ ] [[Should everything be in rem]]

```ad-summary
title: Stage 2 Checkpoint
Before moving on, you should be able to:
- Build any common page layout with Flexbox, Grid, or a combination
- Know when to use Flexbox vs Grid (one-dimensional vs two-dimensional)
- Position overlays, modals, and sticky headers correctly
- Write responsive layouts that adapt without JavaScript
```

---

## Stage 3 -- Advanced (Polish and Architecture)

*Goal: Write CSS that is maintainable, performant, and handles real-world complexity.*

### 3.1 Responsive Design

- [ ] Media queries -- breakpoints, `prefers-color-scheme`, `prefers-reduced-motion` **(gap)**
- [ ] Container queries -- `@container` for component-level responsiveness **(gap)**
- [ ] Fluid typography with `clamp()` **(gap - technique note)**
- [ ] Responsive images -- `srcset`, `sizes`, and CSS approaches **(gap)**
- [ ] Mobile-first vs desktop-first strategy **(gap)**

### 3.2 Animations and Transitions

- [ ] [[transition]] shorthand
- [ ] [[transition-property]], [[transition-duration]], [[transition-timing-function]]
- [ ] [[transitionend Event]]
- [ ] [[CSS vs JS Animations]]
- [ ] `@keyframes` and the `animation` shorthand **(gap)**
- [ ] `transform` -- translate, rotate, scale, skew **(gap)**
- [ ] Hardware acceleration and `will-change` **(gap)**
- [ ] Scroll-driven animations **(gap)**

### 3.3 Advanced Selectors

- [ ] `:is()`, `:where()`, `:not()`, `:has()` **(gap)** -- modern pseudo-class functions
- [ ] `:nth-child()`, `:nth-of-type()` **(gap)** -- pattern matching
- [ ] `:focus-visible` vs `:focus` **(gap)**
- [ ] `:empty`, `:checked`, `:disabled`, `:required` **(gap)** -- form-related

### 3.4 Custom Properties (Advanced Usage)

- [ ] Scoping variables to components **(gap)**
- [ ] Theming with custom properties **(gap)**
- [ ] Fallback values and inheritance tricks **(gap)**
- [ ] Dynamic values via JavaScript `setProperty()` **(gap)**

### 3.5 Typography and Color

- [ ] `font-display` for web font loading **(gap)**
- [ ] Color functions -- `hsl()`, `oklch()`, `color-mix()` **(gap)**
- [ ] System font stacks **(gap)**
- [ ] `text-overflow`, `word-break`, `overflow-wrap` **(gap)**

### 3.6 CSS Architecture

- [ ] [[CSS Styling Approaches]] -- overview of approaches
- [ ] [[Atomic CSS]]
- [ ] ITCSS, SMACSS methodologies **(gap)**
- [ ] When and why to pick: [[Tailwind Overview|Tailwind]], [[Styled Components Overview|Styled Components]], [[CSS Module|CSS Modules]]
- [ ] CSS layers -- `@layer` for managing cascade priority **(gap)**

### 3.7 Visual Effects

- [ ] [[box-shadow]]
- [ ] Gradients -- `linear-gradient()`, `radial-gradient()`, `conic-gradient()` **(gap)**
- [ ] Filters -- `filter` and `backdrop-filter` **(gap)**
- [ ] `clip-path` **(gap)**
- [ ] `mix-blend-mode`, `isolation` **(gap)**
- [ ] `aspect-ratio` **(gap)**

```ad-summary
title: Stage 3 Checkpoint
Before moving on, you should be able to:
- Build fully responsive, animated interfaces
- Architect CSS for a large codebase without specificity wars
- Use modern selectors like `:has()` and `:is()` to reduce code
- Implement dark mode toggling with custom properties
- Choose the right CSS architecture for a given project
```

---

## Stage 4 -- Mastery (Modern CSS and Deep Knowledge)

*Goal: Know the edge cases, the "why" behind browser behavior, and leverage the newest CSS capabilities.*

### 4.1 Layout (Advanced)

- [ ] Subgrid **(gap)** -- nested grids that align to parent tracks
- [ ] `grid-template-areas` for named layouts **(gap)**
- [ ] Multi-column layout -- `columns`, `column-gap`, `break-inside` **(gap)**
- [ ] Writing modes and logical properties **(gap)** -- `inline-size`, `block-size`
- [ ] Content-visibility and `contain` for rendering performance **(gap)**

### 4.2 Modern CSS Features

- [ ] CSS Nesting (native, no preprocessor) **(gap)**
- [ ] `@scope` -- scoping styles to DOM subtrees **(gap)**
- [ ] `@property` -- typed custom properties with defaults **(gap)**
- [ ] View Transitions API (CSS portion) **(gap)**
- [ ] Anchor Positioning **(gap)**
- [ ] Scroll snap -- `scroll-snap-type`, `scroll-snap-align` **(gap)**
- [ ] `@starting-style` for entry animations **(gap)**
- [ ] `color-scheme` and system-aware styling **(gap)**

### 4.3 Performance

- [ ] What triggers layout, paint, and composite **(gap)**
- [ ] Reducing reflows -- batch DOM reads, avoid layout thrashing **(gap)**
- [ ] Critical CSS and extraction strategies **(gap)**
- [ ] CSS containment -- `contain: layout style paint` **(gap)**
- [ ] `content-visibility: auto` for off-screen rendering **(gap)**

### 4.4 Accessibility

- [ ] `prefers-reduced-motion` and animation safety **(gap)**
- [ ] `prefers-contrast`, `forced-colors` **(gap)**
- [ ] Focus indicators -- visible, styled, accessible **(gap)**
- [ ] Screen reader considerations -- `visually-hidden` pattern **(gap)**
- [ ] Color contrast requirements (WCAG) **(gap)**

### 4.5 Debugging and Tooling

- [ ] Chrome DevTools -- computed styles, layout overlays, animation inspector **(gap)**
- [ ] Understanding the "Computed" tab vs "Styles" tab **(gap)**
- [ ] Debugging specificity conflicts **(gap)**
- [ ] CSS preprocessors -- Sass/SCSS basics for legacy codebases **(gap)**
- [ ] PostCSS and build-time transforms **(gap)**

```ad-summary
title: Stage 4 Checkpoint
At this level, you should be able to:
- Diagnose and fix any layout or rendering issue from DevTools alone
- Use modern CSS features to eliminate JavaScript workarounds
- Architect accessible, performant styles for production
- Explain *why* a particular CSS behavior occurs (spec-level understanding)
- Evaluate new CSS proposals and know when to adopt them
```

---

## Vault Coverage Summary

A quick view of what this vault covers well vs where gaps remain.

| Stage | Covered | Gaps |
|---|---|---|
| 1 -- Beginner | Strong. Box model, selectors, units, cascade all have detailed notes | Minimal |
| 2 -- Intermediate | Strong. Flexbox, Grid, positioning, functions, combinators covered | Stacking context, some at-rules |
| 3 -- Advanced | Partial. Transitions exist, styling approaches covered | Responsive design, keyframes, modern selectors, typography, visual effects |
| 4 -- Mastery | Minimal. A few concepts touched on | Nearly everything -- this is your growth frontier |

```ad-tip
title: Suggested Next Notes to Write
The highest-impact gaps to fill next, in priority order:
1. Stacking context and `z-index` -- you will hit this constantly
2. `@keyframes` and `animation` -- the other half of CSS motion
3. `:has()`, `:is()`, `:where()` -- modern selectors that change how you write CSS
4. Container queries -- the future of component-based responsive design
5. CSS Nesting (native) -- already shipped in all browsers, replaces preprocessor nesting
```
