# Fluid Design System - Claude Context v3.1

**Purpose**: Machine-readable design rules for AI assistants. For human-readable documentation, see the full ruleset.

---

## Core Directives

### Priority Order
1. **Fluidity first** - Use viewport units, clamp(), percentages. Let browser calculate.
2. **Accessibility always** - WCAG 2.2 AA is non-negotiable.
3. **Semantic HTML** - Use correct elements, not divs with classes.
4. **Progressive enhancement** - Start with working HTML, layer CSS.

### Absolute Rules (NEVER violate)

**NEVER:**
- Use fixed pixel breakpoints for layout (`@media (min-width: 768px)`)
- Use generic `<div>` when semantic HTML exists
- Omit `alt` text on images
- Trap keyboard focus
- Use color alone to convey information
- Set `line-height` in pixels (always unitless)
- Use `!important` (unless overriding third-party CSS)
- Quote or reproduce exact text (paraphrase for citations)
- Remove focus indicators without replacement (`outline: none`)
- Use `hanging-punctuation` on form fields

**ALWAYS:**
- Use cascade layers: `@layer reset, theme, layout, components, utilities`
- Scope variables globally (`:root`) with local overrides (`--_variable`)
- Make all interactive elements keyboard-accessible
- Set `box-sizing: border-box` globally
- Include `lang` attribute on `<html>`
- Associate `<label>` with form inputs
- Use semantic HTML before ARIA
- Test keyboard navigation (Tab, Enter, Space, Arrows)

---

## Decision Trees

### Choosing Layout Approach

```
Need fixed number of columns?
├─ YES → Use .repeating-grid or .repeating-flex
└─ NO → Should content determine columns?
    ├─ YES → Use .fluid-grid (DEFAULT CHOICE)
    └─ NO → Justify why fixed columns needed
```

### Choosing Between Grid and Flex

```
Need items to stretch and fill last row?
├─ YES → Use Flex variant (.fluid-flex or .repeating-flex)
└─ NO → Use Grid variant (.fluid-grid or .repeating-grid)

Need Subgrid alignment?
├─ YES → Must use Grid + .subgrid-rows
└─ NO → Either Grid or Flex works
```

### Typography Sizing

```
Text purpose?
├─ Long-form body → 45-75ch line length, line-height: 1.5-1.6
├─ Large headings → Shorter line-height: 1.1-1.3, -0.02em letter-spacing
├─ Small UI text → Taller line-height: 1.6-1.8
└─ All → Use clamp() for fluid sizing
```

---

## CSS Architecture

### Layer Structure (Mandatory)
```css
@layer reset, theme, layout, components, utilities;
```

**Specificity order**: `reset` (lowest) → `utilities` (highest)

### Variable Pattern

**Global scope** (`:root` in `@layer theme`):
```css
--layout-fluid-min: 35ch;
--space-default: clamp(1.5rem, 1.37rem + 0.65vw, 1.88rem);
--font-size-base: clamp(1rem, 0.95rem + 0.25vw, 1.25rem);
```

**Local scope** (component-level):
```css
.fluid-grid {
  --_fluid-min: var(--fluid-grid-min, var(--layout-fluid-min));
  /* Use --_fluid-min in calculations */
}
```

**Override** (semantic class):
```css
.product-grid {
  --fluid-grid-min: 250px;
}
```

---

## Layout Utilities

### .fluid-grid (DEFAULT CHOICE)

**Use when**: Don't know exact column count, want browser to calculate optimal layout.

```css
.fluid-grid {
  --_fluid-min: var(--fluid-grid-min, var(--layout-fluid-min));
  --_gap: var(--grid-gap, var(--layout-default-gap));
  
  display: grid;
  grid-template-columns: repeat(
    auto-fit,
    minmax(min(var(--_fluid-min), 100%), 1fr)
  );
  gap: var(--_gap);
  container: grid-item / inline-size;
}
```

**Override example**:
```html
<div class="fluid-grid" style="--fluid-grid-min: 250px">
```

### .repeating-grid

**Use when**: Need exactly N equal columns.

```css
.repeating-grid {
  --_repeat: var(--grid-repeat, var(--layout-default-repeat));
  --_gap: var(--grid-gap, var(--layout-default-gap));
  
  display: grid;
  grid-template-columns: repeat(var(--_repeat), 1fr);
  gap: var(--_gap);
}
```

**Override example**:
```html
<div class="repeating-grid" style="--grid-repeat: 4">
```

### .fluid-flex

**Use when**: Want last row items to stretch and fill space.

```css
.fluid-flex {
  --_fluid-min: var(--fluid-flex-min, var(--layout-fluid-min));
  --_gap: var(--flex-gap, var(--layout-default-gap));
  
  display: flex;
  flex-wrap: wrap;
  gap: var(--_gap);
}

.fluid-flex > * {
  flex: 1 1 var(--_fluid-min);
}
```

### .repeating-flex

**Use when**: Need exactly N columns but want last row to stretch.

```css
.repeating-flex {
  --_repeat: var(--flex-repeat, var(--layout-default-repeat));
  --_gap: var(--flex-gap, var(--layout-default-gap));
  --_gap-count: calc(var(--_repeat) - 1);
  --_gap-calc: calc(var(--_gap) / var(--_repeat) * var(--_gap-count));
  
  display: flex;
  flex-wrap: wrap;
  gap: var(--_gap);
}

.repeating-flex > * {
  flex: 1 1 calc((100% / var(--_repeat)) - var(--_gap-calc));
}
```

### .stack (Vertical spacing)

**Use when**: Need consistent vertical rhythm.

```css
.stack {
  display: flex;
  flex-direction: column;
  gap: var(--_stack-gap, var(--layout-default-gap));
}
```

### .subgrid-rows (Optional, Grid only)

**Use when**: Need nested grid aligned to parent grid tracks.

```css
.subgrid-rows > * {
  display: grid;
  grid-row: auto / span var(--subgrid-rows, 4);
  grid-template-rows: subgrid;
  gap: var(--subgrid-gap, 0);
}
```

---

## Typography Rules

### Fluid Scales (Use Utopia calculator)

**Generate at**: [utopia.fyi/type/calculator](https://utopia.fyi/type/calculator/)

```css
--font-size-xs: clamp(0.75rem, 0.7rem + 0.26vw, 0.94rem);
--font-size-sm: clamp(0.89rem, 0.83rem + 0.28vw, 1.06rem);
--font-size-base: clamp(1rem, 0.95rem + 0.25vw, 1.25rem);
--font-size-md: clamp(1.13rem, 1.06rem + 0.33vw, 1.38rem);
--font-size-lg: clamp(1.27rem, 1.17rem + 0.47vw, 1.63rem);
--font-size-xl: clamp(1.42rem, 1.29rem + 0.65vw, 1.91rem);
--font-size-2xl: clamp(1.6rem, 1.42rem + 0.87vw, 2.25rem);
--font-size-3xl: clamp(1.8rem, 1.57rem + 1.15vw, 2.66rem);
```

### Line Height Formula

```
Small text (12-16px): 1.6-1.8
Body text (16-20px): 1.5
Large headings (32px+): 1.1-1.3
```

**Always unitless** (not px or em)

### Measure (Line Length)

```css
.prose {
  max-width: 70ch; /* Never exceed */
  margin-inline: auto;
}

/* Short lines for emphasis */
.lead {
  max-width: 45ch;
}
```

### Hanging Punctuation

```css
/* Progressive enhancement - Safari only */
.prose {
  hanging-punctuation: first last;
}

/* CRITICAL: Fix form fields */
input, textarea {
  hanging-punctuation: none;
}
```

---

## Spacing Rules

### Fluid Spacing (Use Utopia calculator)

**Generate at**: [utopia.fyi/space/calculator](https://utopia.fyi/space/calculator/)

```css
--space-3xs: clamp(0.25rem, 0.23rem + 0.11vw, 0.31rem);
--space-2xs: clamp(0.5rem, 0.46rem + 0.22vw, 0.63rem);
--space-xs: clamp(0.75rem, 0.68rem + 0.33vw, 0.94rem);
--space-s: clamp(1rem, 0.91rem + 0.43vw, 1.25rem);
--space-m: clamp(1.5rem, 1.37rem + 0.65vw, 1.88rem);
--space-l: clamp(2rem, 1.83rem + 0.87vw, 2.5rem);
--space-xl: clamp(3rem, 2.74rem + 1.3vw, 3.75rem);
--space-2xl: clamp(4rem, 3.65rem + 1.74vw, 5rem);
--space-3xl: clamp(6rem, 5.48rem + 2.61vw, 7.5rem);

/* One-up pairs for fluid relationships */
--space-s-m: clamp(1rem, 0.74rem + 1.3vw, 1.88rem);
--space-m-l: clamp(1.5rem, 1.2rem + 1.52vw, 2.5rem);
--space-l-xl: clamp(2rem, 1.48rem + 2.61vw, 3.75rem);
```

### Spacing Hierarchy

```
More space = More importance
```

```css
.section-major { margin-block: var(--space-xl-2xl); }
.section-minor { margin-block: var(--space-m-l); }
.related-content { margin-block: var(--space-s); }
```

---

## Visual Hierarchy

### Button Hierarchy

```css
/* Primary - highest contrast, solid */
.btn-primary {
  background: var(--color-primary);
  color: white;
  padding: 1em 2em;
  font-weight: 600;
}

/* Secondary - outlined, less prominent */
.btn-secondary {
  border: 2px solid currentColor;
  background: transparent;
  padding: 1em 2em;
}

/* Tertiary - link-styled */
.btn-tertiary {
  text-decoration: underline;
  background: none;
  padding: 0.5em 1em;
}
```

### De-emphasis Pattern

**Instead of only making primary content bold:**
```css
/* Make secondary content softer */
.metadata {
  color: hsl(from var(--color-text) h s calc(l + 40%));
  font-size: 0.875em;
}
```

---

## Accessibility Quick Reference

### WCAG 2.2 AA Requirements

**Images**:
```html
<!-- GOOD -->
<img src="chart.png" alt="Sales increased 23% in Q3">

<!-- BAD -->
<img src="chart.png" alt="chart">
<img src="chart.png"> <!-- Missing alt -->
```

**Forms**:
```html
<!-- GOOD -->
<label for="email">Email</label>
<input type="email" id="email" name="email">

<!-- BAD -->
<input type="email" placeholder="Email"> <!-- Placeholder ≠ label -->
```

**Focus**:
```css
/* GOOD - visible indicator */
button:focus-visible {
  outline: 3px solid var(--color-primary);
  outline-offset: 2px;
}

/* BAD - removes without replacement */
button:focus { outline: none; }
```

**Semantic HTML**:
```html
<!-- GOOD -->
<button type="button">Click</button>
<nav><a href="...">Link</a></nav>

<!-- BAD -->
<div onclick="...">Click</div> <!-- Missing role, tabindex, keyboard handler -->
```

### Color Contrast

- **Text**: 4.5:1 minimum
- **Large text** (18px+ or 14px+ bold): 3:1 minimum
- **UI components**: 3:1 minimum

### Keyboard Navigation

All interactive elements must respond to:
- `Tab` / `Shift+Tab` (focus movement)
- `Enter` (activation)
- `Space` (activation for buttons/checkboxes)
- Arrow keys (for select/radio/tabs)

---

## Container Queries

### Pattern for Component Responsiveness

```css
.card {
  container: card / inline-size;
}

/* Responds to card width, not viewport */
@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}

@container card (min-width: 600px) {
  .card-title {
    font-size: var(--font-size-2xl);
  }
}
```

### Auto-containerize Grid Items

```css
.fluid-grid > * {
  container: grid-item / inline-size;
}

/* Now all grid items are queryable */
@container grid-item (min-width: 350px) {
  .card { /* styles */ }
}
```

---

## Common Pitfalls

### aspect-ratio Breaks When:

1. **Both dimensions set**:
```css
/* BAD */
.box {
  width: 300px;
  height: 200px;
  aspect-ratio: 16/9; /* Ignored */
}

/* GOOD */
.box {
  width: 300px;
  aspect-ratio: 16/9; /* Height auto-calculated */
}
```

2. **Flex stretch active**:
```css
/* BAD */
.flex-item {
  aspect-ratio: 16/9; /* Gets stretched by default */
}

/* GOOD */
.flex-item {
  aspect-ratio: 16/9;
  align-self: flex-start;
}
```

3. **Content forces height**:
```css
/* GOOD */
.box {
  aspect-ratio: 1;
  overflow: auto; /* Constrain content */
}
```

### Avoid These Patterns

```css
/* BAD - Fixed breakpoints */
@media (min-width: 768px) { /* ... */ }

/* GOOD - Fluid or container queries */
@container (min-width: 400px) { /* ... */ }

/* BAD - Pixel spacing */
margin: 24px;

/* GOOD - Fluid spacing */
margin: var(--space-m);

/* BAD - Hardcoded column count */
grid-template-columns: 1fr 1fr 1fr;

/* GOOD - Auto-calculated */
grid-template-columns: repeat(auto-fit, minmax(min(250px, 100%), 1fr));
```

---

## Testing Checklist

**Before considering complete**:
- [ ] Resize: 320px → 3840px (no breaks)
- [ ] Keyboard: Tab through all interactive elements
- [ ] Screen reader: Test with NVDA/JAWS/VoiceOver
- [ ] Contrast: Check with DevTools (4.5:1 text, 3:1 UI)
- [ ] Zoom: 200% browser zoom (no horizontal scroll)
- [ ] No CSS: Content order still logical
- [ ] Accessibility scan: Run axe DevTools (zero violations)

---

## Quick Reference: What To Use When

**Card grid with unknown item count** → `.fluid-grid`  
**Exact 3-column layout** → `.repeating-grid` with `--grid-repeat: 3`  
**Gallery where last row stretches** → `.fluid-flex`  
**Vertical content flow** → `.stack`  
**Dashboard with fixed panels** → `.repeating-grid`  
**Article with images** → `.stack` for sections + `.prose` for text blocks  
**Product grid with varying card widths** → `.fluid-grid` with min width override  
**Nested grid aligned to parent** → `.fluid-grid` + `.subgrid-rows`

---

## Color System

**Use HSL for adjustability**:
```css
--color-primary: hsl(220 90% 56%);
--color-text: hsl(0 0% 10%);

/* Generate lighter variant */
--color-primary-light: hsl(from var(--color-primary) h s calc(l + 20%));

/* Generate de-emphasized text */
--color-text-secondary: hsl(from var(--color-text) h s calc(l + 40%));
```

---

## Tools & Generators

- **Type scales**: [utopia.fyi/type/calculator](https://utopia.fyi/type/calculator/)
- **Space scales**: [utopia.fyi/space/calculator](https://utopia.fyi/space/calculator/)
- **Contrast checker**: Browser DevTools or [webaim.org/resources/contrastchecker](https://webaim.org/resources/contrastchecker/)
- **Accessibility**: axe DevTools browser extension

---

## When To Push Back

**If asked to**:
- Use fixed breakpoints → Explain fluid approach is better
- Remove focus indicators → Cite accessibility requirements
- Use divs instead of semantic HTML → Suggest correct element
- Omit alt text → Explain legal/ethical requirement
- Add `!important` → Suggest cascade layer solution
- Set both width and height with aspect-ratio → Explain the conflict

**Provide alternatives, don't just refuse.**

---

**Version**: 3.1 (2025-09-30)  
**Full documentation**: Fluid-Design-Ruleset-v3.1.md  
**Author**: Tim (@yakbrother)
