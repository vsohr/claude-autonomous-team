# Design Quality Deep Dive

Visual design principles for engineers. The knowledge that makes UIs feel *designed* rather than *assembled*.

## Table of Contents
1. [Typography System](#typography-system)
2. [Spacing System](#spacing-system)
3. [Color System](#color-system)
4. [Motion Principles](#motion-principles)
5. [Visual Hierarchy](#visual-hierarchy)
6. [Polish Checklist](#polish-checklist)
7. [Common Design Mistakes](#common-design-mistakes)

---

## Typography System

### Type Scale

Use a modular scale for harmonious sizing. Common ratios:
- **1.25** (Major Third) — Subtle, professional
- **1.333** (Perfect Fourth) — Balanced, versatile
- **1.5** (Perfect Fifth) — Dramatic, editorial

```css
/* 1.25 ratio scale */
--text-xs: 0.64rem;    /* 10.24px */
--text-sm: 0.8rem;     /* 12.8px */
--text-base: 1rem;     /* 16px */
--text-lg: 1.25rem;    /* 20px */
--text-xl: 1.563rem;   /* 25px */
--text-2xl: 1.953rem;  /* 31.25px */
--text-3xl: 2.441rem;  /* 39px */
--text-4xl: 3.052rem;  /* 48.8px */
```

### Line Height

| Text Size | Line Height | Use Case |
|-----------|-------------|----------|
| Body text | 1.5 - 1.75 | Paragraphs, long-form |
| Headings | 1.1 - 1.3 | Titles, hero text |
| UI elements | 1.25 - 1.5 | Buttons, labels, nav |
| Tight captions | 1.2 - 1.4 | Small text, metadata |

### Line Length (Measure)

```css
/* Optimal reading: 45-75 characters */
.prose {
  max-width: 65ch; /* Character units! */
}

/* Short-form content */
.card-description {
  max-width: 45ch;
}
```

### Font Pairing

**Safe combinations:**
- **Inter + Serif**: Modern + readable body
- **Space Grotesk + System**: Technical + familiar
- **Outfit + Source Serif**: Geometric + literary

**Pairing rules:**
1. Contrast: Pair different classifications (sans + serif, geometric + humanist)
2. Similar x-heights work better together
3. One display font, one workhorse font

### Weight Usage

```css
/* Clear hierarchy through weight */
--font-weight-body: 400;      /* Regular paragraphs */
--font-weight-medium: 500;    /* Subtle emphasis, UI labels */
--font-weight-semibold: 600;  /* Subheadings, important UI */
--font-weight-bold: 700;      /* Headlines, strong emphasis */
```

### Responsive Typography

```css
/* Fluid type scale with clamp() */
h1 {
  font-size: clamp(2rem, 5vw + 1rem, 4rem);
  /* min: 32px, grows with viewport, max: 64px */
}

/* Or use container queries */
.card {
  container-type: inline-size;
  h2 { font-size: clamp(1rem, 5cqi, 1.5rem); }
}
```

---

## Spacing System

### Base Unit: 4px or 8px

All spacing as multiples of base unit creates visual rhythm.

```css
--space-0: 0;
--space-1: 0.25rem;  /* 4px */
--space-2: 0.5rem;   /* 8px */
--space-3: 0.75rem;  /* 12px */
--space-4: 1rem;     /* 16px */
--space-5: 1.25rem;  /* 20px */
--space-6: 1.5rem;   /* 24px */
--space-8: 2rem;     /* 32px */
--space-10: 2.5rem;  /* 40px */
--space-12: 3rem;    /* 48px */
--space-16: 4rem;    /* 64px */
--space-20: 5rem;    /* 80px */
--space-24: 6rem;    /* 96px */
```

### Spacing Principles

**Proximity**: Related items closer together
```
Header            ← space-4 → Subheader
─────────────────────────────────────────
                  ↑
             space-8 (bigger gap = separation)
                  ↓
Card 1    Card 2    Card 3   ← space-4 between cards
```

**Consistency**: Same elements, same spacing
```tsx
// ❌ Inconsistent
<Card className="p-4" />
<Card className="p-6" />
<Card className="px-4 py-5" />

// ✅ Consistent
<Card className="p-4" /> /* Always */
```

**Breathing room**: When in doubt, more space
```css
/* Dense UI: 4px base */
/* Balanced UI: 8px base */
/* Luxurious UI: 12-16px base */
```

### Container Padding

```css
/* Mobile-first responsive padding */
.container {
  padding-inline: var(--space-4);
}

@media (min-width: 768px) {
  .container {
    padding-inline: var(--space-8);
  }
}

@media (min-width: 1280px) {
  .container {
    padding-inline: var(--space-12);
  }
}
```

### Component Internal Spacing

```
┌─────────────────────────────────────┐
│ padding-4                           │
│  ┌─────────────────────────────┐    │
│  │ Title                       │    │
│  └─────────────────────────────┘    │
│           ↕ gap-2                   │
│  ┌─────────────────────────────┐    │
│  │ Description text that       │    │
│  │ might wrap to multiple...   │    │
│  └─────────────────────────────┘    │
│           ↕ gap-4                   │
│  ┌──────────┐  ┌──────────┐         │
│  │ Button 1 │  │ Button 2 │ ← gap-2 │
│  └──────────┘  └──────────┘         │
│                                     │
└─────────────────────────────────────┘
```

---

## Color System

### Color Roles

```css
/* Semantic color tokens */
--color-background: /* Page background */
--color-foreground: /* Primary text */
--color-muted: /* Secondary backgrounds */
--color-muted-foreground: /* Secondary text */
--color-border: /* Borders, dividers */
--color-ring: /* Focus rings */

--color-primary: /* Brand, main actions */
--color-primary-foreground: /* Text on primary */

--color-secondary: /* Secondary actions */
--color-accent: /* Highlights, badges */

--color-destructive: /* Errors, delete */
--color-warning: /* Warnings, caution */
--color-success: /* Success, confirm */
--color-info: /* Information, help */
```

### 60-30-10 Rule

| Proportion | Usage | Example |
|------------|-------|---------|
| 60% | Dominant | Background, large surfaces |
| 30% | Secondary | Cards, containers, headers |
| 10% | Accent | CTAs, highlights, active states |

### Contrast Requirements (WCAG 2.1)

| Element | Minimum | Enhanced |
|---------|---------|----------|
| Normal text | 4.5:1 | 7:1 |
| Large text (18px+) | 3:1 | 4.5:1 |
| UI components | 3:1 | — |
| Focus indicators | 3:1 | — |

**Tools:**
- Stark (Figma plugin)
- WebAIM Contrast Checker
- Chrome DevTools color picker

### Dark Mode

**Don't just invert colors:**

```css
/* ❌ Inverted (harsh) */
.dark {
  --background: #000000;
  --foreground: #ffffff;
}

/* ✅ Designed for dark (softer) */
.dark {
  --background: oklch(15% 0.01 270);  /* Dark blue-gray, not pure black */
  --foreground: oklch(95% 0 0);       /* Off-white, not pure white */
  --muted: oklch(22% 0.01 270);       /* Elevated surfaces slightly lighter */
  --border: oklch(30% 0.01 270);      /* Visible but subtle */
}
```

**Dark mode principles:**
- Reduce contrast slightly (pure white hurts)
- Elevate with lightness, not shadows
- Saturate colors slightly less
- Keep brand colors recognizable

### OKLCH Color Space

```css
/* oklch(lightness chroma hue) */

/* Lightness: 0% (black) to 100% (white) */
/* Chroma: 0 (gray) to 0.4 (vivid) */
/* Hue: 0-360 (color wheel) */

--blue-500: oklch(55% 0.25 250);
--green-500: oklch(65% 0.2 145);
--red-500: oklch(55% 0.22 25);

/* Perceptually uniform: same lightness = same perceived brightness */
```

---

## Motion Principles

### Timing

| Duration | Use Case |
|----------|----------|
| 100-150ms | Micro-interactions (button press, toggle) |
| 200-300ms | Revealing content (fade in, slide) |
| 300-500ms | Page transitions, modals |
| 500ms+ | Complex sequences, storytelling |

### Easing

```css
/* Natural motion curves */
--ease-out: cubic-bezier(0, 0, 0.2, 1);      /* Enter: fast then slow */
--ease-in: cubic-bezier(0.4, 0, 1, 1);       /* Exit: slow then fast */
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1); /* State change */
--ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1); /* Playful overshoot */

/* When to use */
/* Entering elements: ease-out (decelerate into view) */
/* Exiting elements: ease-in (accelerate out of view) */
/* State changes: ease-in-out (smooth transformation) */
```

### Motion Patterns

**Staggered reveal:**
```tsx
const containerVariants = {
  hidden: {},
  visible: {
    transition: { staggerChildren: 0.05 }
  }
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 }
};

<motion.ul variants={containerVariants} initial="hidden" animate="visible">
  {items.map(item => (
    <motion.li key={item.id} variants={itemVariants} />
  ))}
</motion.ul>
```

**Shared layout animation:**
```tsx
// Element smoothly animates position when layout changes
<motion.div layout layoutId="expandable-card">
  {isExpanded ? <ExpandedView /> : <CollapsedView />}
</motion.div>
```

**Scroll-triggered:**
```css
/* CSS scroll-driven animation */
@keyframes fade-slide-up {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.reveal {
  animation: fade-slide-up linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 30%;
}
```

### Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

```tsx
// Motion library
<motion.div
  initial={{ opacity: 0 }}
  animate={{ opacity: 1 }}
  transition={{ duration: prefersReducedMotion ? 0 : 0.3 }}
/>
```

---

## Visual Hierarchy

### Creating Hierarchy

Tools for directing attention (combine 2-3, not all):

| Tool | Example |
|------|---------|
| Size | Larger = more important |
| Weight | Bolder = more important |
| Color | Saturated/bright = more important |
| Contrast | Higher contrast = more important |
| Space | More space around = more important |
| Position | Top-left (LTR) gets seen first |

### Example: Card Hierarchy

```
┌─────────────────────────────────────┐
│  FEATURED                    ← Badge (small, colored)
│                                     │
│  Product Name               ← Primary: large, bold
│  Short description that     ← Secondary: smaller, muted
│  explains the value.                │
│                                     │
│  $99.00  $149.00           ← Price: bold + strikethrough
│                                     │
│  ┌─────────────────────┐           │
│  │   Add to Cart       │    ← CTA: largest interactive, primary color
│  └─────────────────────┘           │
│                                     │
│  ★★★★☆ 42 reviews        ← Tertiary: smallest, muted
└─────────────────────────────────────┘
```

### Squint Test

Blur your eyes or view at 50% zoom. If you can still identify:
- The most important element
- The CTA
- The section boundaries

...then your hierarchy is working.

---

## Polish Checklist

### States

- [ ] **Hover**: Every clickable element has hover feedback
- [ ] **Active/Pressed**: Buttons depress or react on click
- [ ] **Focus**: Visible focus ring (not just browser default)
- [ ] **Disabled**: Clear visual distinction, correct cursor
- [ ] **Loading**: Spinners, skeletons, or progress indicators
- [ ] **Empty**: Helpful empty states, not just "No data"
- [ ] **Error**: Clear error messages with recovery actions

### Micro-details

- [ ] **Icons**: Consistent size (16, 20, 24px), stroke width, style
- [ ] **Border radius**: Consistent across component types
- [ ] **Shadows**: Consistent elevation system
- [ ] **Transitions**: Smooth state changes (200-300ms)
- [ ] **Truncation**: Text overflow with ellipsis + title attr
- [ ] **Selection**: Custom ::selection color
- [ ] **Scrollbars**: Styled or hidden (where appropriate)
- [ ] **Favicon**: Present, multiple sizes

### Responsive

- [ ] **Touch targets**: 44x44px minimum on mobile
- [ ] **Text readable**: No tiny text on mobile
- [ ] **Images sized**: Right resolution for each breakpoint
- [ ] **No horizontal scroll**: Test all breakpoints

### Performance Perception

- [ ] **Skeleton screens**: Match actual content layout
- [ ] **Optimistic UI**: Instant feedback before server confirms
- [ ] **Progress indicators**: For operations > 1 second
- [ ] **Instant navigation**: Prefetch likely destinations

---

## Common Design Mistakes

### 1. Inconsistent Spacing

```
❌ p-4 p-5 p-6 p-8 px-3 py-7

✅ Define and use a scale:
   p-4 (small cards)
   p-6 (medium cards)  
   p-8 (large cards/sections)
```

### 2. Too Many Font Sizes

```
❌ 11px, 12px, 13px, 14px, 15px, 16px...

✅ Limited scale with clear purpose:
   text-sm (captions, metadata)
   text-base (body)
   text-lg (subheadings)
   text-xl (headings)
   text-2xl (page titles)
```

### 3. Low Contrast Gray Text

```
❌ color: #999 on white (2.8:1 - fails WCAG)
✅ color: #666 on white (5.7:1 - passes AA)
```

### 4. Missing Feedback

```tsx
// ❌ Silent button
<button onClick={save}>Save</button>

// ✅ Communicative button
<button onClick={save} disabled={isSaving}>
  {isSaving ? 'Saving...' : 'Save'}
</button>
```

### 5. Harsh Borders

```css
/* ❌ High-contrast border */
border: 1px solid #000;

/* ✅ Subtle border */
border: 1px solid oklch(85% 0 0);
/* Or */
box-shadow: 0 0 0 1px oklch(0% 0 0 / 0.1);
```

### 6. Generic Empty States

```tsx
// ❌ Useless
<p>No results</p>

// ✅ Helpful
<div className="text-center py-12">
  <SearchIcon className="mx-auto h-12 w-12 text-muted" />
  <h3 className="mt-4 text-lg font-medium">No results found</h3>
  <p className="mt-2 text-muted-foreground">
    Try adjusting your search or filters.
  </p>
  <Button variant="outline" className="mt-4" onClick={clearFilters}>
    Clear all filters
  </Button>
</div>
```

### 7. Ignoring Focus States

```css
/* ❌ Removes focus for "cleaner" look */
:focus { outline: none; }

/* ✅ Custom focus that's visible AND aesthetic */
:focus-visible {
  outline: 2px solid var(--color-ring);
  outline-offset: 2px;
}
```
