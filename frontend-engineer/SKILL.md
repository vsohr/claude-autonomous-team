---
name: frontend-engineer
description: Senior-level frontend engineering combining technical excellence with design sensibility. Use when building React/TypeScript applications requiring professional-grade architecture, performance optimization, and polished user experiences. Covers decision frameworks for technology choices, debugging/profiling, design quality principles, accessibility, i18n, and security. For React 19, Vite 7, Tailwind CSS 4, and modern CSS patterns.
---

# Frontend Engineer

Expert guidance for building production-grade, visually excellent web interfaces. This skill emphasizes *judgment* and *decision-making*—the senior-level skills that distinguish great engineers.

## Core Principle

Technical correctness is table stakes. Senior engineers deliver UIs that are:
- **Technically sound** — performant, accessible, maintainable
- **Visually polished** — thoughtful typography, spacing, motion
- **Delightful** — the small details that make users smile

## Technology Stack (2025)

| Layer | Technology | Version |
|-------|------------|---------|
| Framework | React | 19.2 |
| Language | TypeScript | 5.x (strict mode) |
| Build | Vite | 7.x |
| Styling | Tailwind CSS | 4.x |
| Components | shadcn/ui + React Aria | latest |
| Animation | Motion (Framer Motion) | 11.x |
| State | TanStack Query + Zustand | latest |
| Forms | React Hook Form + Zod | latest |
| Testing | Vitest + Playwright | latest |

## React 19 Key Features

**Use these—they're production-ready:**
- `useActionState` + `useFormStatus` for form mutations
- `useOptimistic` for instant UI feedback
- `use()` hook for reading promises/context in render
- `<Activity>` for controlling render priority
- Server Components for data fetching (framework-dependent)
- Ref cleanup functions and refs as props

**Server vs Client Components:**
```
Server Component (default)     Client Component ('use client')
├── Data fetching              ├── Interactivity (onClick, onChange)
├── Backend access             ├── Browser APIs (localStorage, etc.)
├── Heavy dependencies         ├── State (useState, useReducer)
├── Sensitive logic            └── Effects (useEffect)
└── Zero JS to client
```

## Design Quality Principles

**The difference between "works" and "exceptional."**

### Typography
- **Hierarchy**: 3-4 distinct levels max (display, heading, body, caption)
- **Scale**: Use a modular scale (1.25 or 1.333 ratio)
- **Measure**: 45-75 characters per line for readability
- **Weight contrast**: Bold headlines (600-700) + regular body (400)
- **Font pairing**: One display font + one workhorse font, max

```css
--text-xs: clamp(0.75rem, 0.7rem + 0.25vw, 0.875rem);
--text-sm: clamp(0.875rem, 0.8rem + 0.375vw, 1rem);
--text-base: clamp(1rem, 0.9rem + 0.5vw, 1.125rem);
--text-lg: clamp(1.125rem, 1rem + 0.625vw, 1.25rem);
--text-xl: clamp(1.25rem, 1.1rem + 0.75vw, 1.5rem);
--text-2xl: clamp(1.5rem, 1.2rem + 1.5vw, 2rem);
```

### Spacing
- **8px grid**: All spacing in multiples of 8 (4 for tight spaces)
- **Consistency**: Same padding/margin patterns across similar elements
- **Breathing room**: When in doubt, add more whitespace
- **Proximity**: Related elements closer, unrelated further apart

### Color
- **60-30-10 rule**: 60% dominant, 30% secondary, 10% accent
- **Semantic colors**: Success (green), warning (amber), error (red), info (blue)
- **Contrast**: 4.5:1 minimum for text, 3:1 for large text/UI elements
- **Dark mode**: Design both themes simultaneously, not as afterthought

### Motion
- **Duration**: 150-300ms for micro-interactions, 300-500ms for larger transitions
- **Easing**: ease-out for entrances, ease-in for exits, ease-in-out for state changes
- **Purpose**: Motion should communicate, not decorate
- **Reduce motion**: Always respect `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### The Last 10%
What separates good from great:
- Loading skeletons that match actual content layout
- Focus indicators that are visible AND aesthetic
- Empty states with helpful guidance, not just "No data"
- Error messages that explain what went wrong AND what to do
- Hover/active states on every interactive element
- Consistent icon sizes and stroke weights
- Proper text truncation with ellipsis and title attributes

## Decision Frameworks

See [references/decision-frameworks.md](references/decision-frameworks.md) for detailed guidance on:
- State management: when to use what
- Server vs Client Components
- CSS approach selection
- Component library choices
- When to build vs use a library

## Debugging & Performance

See [references/debugging-profiling.md](references/debugging-profiling.md) for:
- React DevTools Profiler usage
- Identifying unnecessary re-renders
- Bundle analysis and optimization
- Memory leak detection
- Core Web Vitals debugging

### Quick Performance Checklist
- [ ] Images: lazy loading, proper sizing, modern formats (WebP/AVIF)
- [ ] Code splitting: `lazy()` for route-level and heavy components
- [ ] Bundle: No massive dependencies (check with `npx vite-bundle-visualizer`)
- [ ] Fonts: `font-display: swap`, preload critical fonts
- [ ] CSS: No runtime CSS-in-JS, use Tailwind or CSS Modules
- [ ] Lists: Virtualize lists > 100 items (TanStack Virtual)

## Accessibility (a11y)

**Non-negotiable requirements:**

```tsx
// ✅ Semantic HTML first
<button onClick={handleClick}>Save</button>
// ❌ Not this
<div onClick={handleClick}>Save</div>

// ✅ Accessible icon button
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

// ✅ Form labels
<label htmlFor="email">Email</label>
<input id="email" type="email" aria-describedby="email-hint" />
<p id="email-hint">We'll never share your email.</p>

// ✅ Live regions for dynamic content
<div aria-live="polite" aria-atomic="true">
  {status && <p>{status}</p>}
</div>
```

**Testing:**
- Tab through entire app—everything reachable?
- Use screen reader (VoiceOver/NVDA) on key flows
- Check color contrast (Lighthouse or axe DevTools)
- Test with `prefers-reduced-motion` enabled

## Internationalization (i18n)

**Setup with next-intl (Next.js) or react-intl:**

```tsx
// Message extraction pattern
const t = useTranslations('ProductPage');
return <h1>{t('title')}</h1>;

// Pluralization
t('itemCount', { count: items.length })
// en: "{count, plural, =0 {No items} one {# item} other {# items}}"

// Date/number formatting
const format = useFormatter();
format.dateTime(date, { dateStyle: 'long' });
format.number(price, { style: 'currency', currency: 'USD' });
```

**RTL Support:**
```css
/* Use logical properties */
margin-inline-start: 1rem;  /* not margin-left */
padding-inline-end: 1rem;   /* not padding-right */
inset-inline-start: 0;      /* not left: 0 */
```

Tailwind 4 logical classes: `ms-4` (margin-start), `me-4` (margin-end), `ps-4`, `pe-4`

## Security

**Frontend security essentials:**

```tsx
// ❌ XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ If you must render HTML, sanitize it
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />

// ✅ Validate and sanitize all inputs
const schema = z.string().max(1000).regex(/^[a-zA-Z0-9\s]+$/);

// ✅ Use CSP headers (server-side)
// Content-Security-Policy: default-src 'self'; script-src 'self'
```

**Auth UI patterns:**
- Never store tokens in localStorage (use httpOnly cookies)
- Implement proper logout (clear all state, invalidate session)
- Handle token expiry gracefully (refresh or redirect)
- Protect routes at both client AND server level

## Modern CSS (2025)

```css
/* Container queries - components adapt to their container */
.card { container-type: inline-size; }
@container (min-width: 400px) { .card-body { display: flex; } }

/* :has() - parent selector */
.form-group:has(input:invalid) { --border-color: var(--color-error); }

/* Subgrid - nested grid alignment */
.grid { display: grid; grid-template-columns: 1fr 2fr 1fr; }
.card { display: grid; grid-template-columns: subgrid; grid-column: span 3; }

/* Native nesting */
.card {
  padding: 1rem;
  &:hover { background: var(--hover-bg); }
  & .title { font-weight: 600; }
}

/* Scroll-driven animations */
@keyframes fade-in { from { opacity: 0; } }
.element {
  animation: fade-in linear;
  animation-timeline: view();
  animation-range: entry 0% cover 40%;
}
```

## Project Structure

```
src/
├── app/                    # Routes (Next.js/Remix)
├── components/
│   ├── ui/                 # Primitive components (Button, Input, Dialog)
│   └── [feature]/          # Feature-specific components
├── hooks/                  # Custom hooks
├── lib/                    # Utilities, API clients, constants
├── styles/                 # Global CSS, Tailwind theme
└── types/                  # Shared TypeScript types
```

## Reference Files

- [decision-frameworks.md](references/decision-frameworks.md) — When to use what
- [debugging-profiling.md](references/debugging-profiling.md) — Performance & debugging
- [design-quality.md](references/design-quality.md) — Visual design deep-dive
- [advanced-patterns.md](references/advanced-patterns.md) — Component implementation patterns
- [design-system.md](references/design-system.md) — Building design systems

## Testing Patterns

**Follow TDD — see test-driven-development skill for the red-green-refactor methodology.**

### Component Testing Strategy

```tsx
// Test user behavior, not implementation details
// ✅ Good: Tests what user sees and does
test('shows error when email is invalid', async () => {
  render(<LoginForm />);
  await userEvent.type(screen.getByLabelText('Email'), 'invalid');
  await userEvent.click(screen.getByRole('button', { name: 'Submit' }));
  expect(screen.getByRole('alert')).toHaveTextContent('Invalid email');
});

// ❌ Bad: Tests implementation details
test('sets error state', () => {
  const { result } = renderHook(() => useLoginForm());
  act(() => result.current.validateEmail('invalid'));
  expect(result.current.error).toBe('Invalid email');
});
```

### Testing Library Best Practices

| Query Priority | Use For |
|---------------|---------|
| `getByRole` | Most elements (buttons, links, headings) |
| `getByLabelText` | Form inputs |
| `getByText` | Non-interactive content |
| `getByTestId` | Last resort only |

### Async Testing

```tsx
// ✅ Wait for conditions, not arbitrary time
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});

// ✅ Use findBy for elements that appear async
const button = await screen.findByRole('button', { name: 'Save' });

// ❌ Never use arbitrary delays
await new Promise(r => setTimeout(r, 1000));
```

### E2E with Playwright

```typescript
// Use page objects for maintainability
class LoginPage {
  constructor(private page: Page) {}

  async login(email: string, password: string) {
    await this.page.getByLabel('Email').fill(email);
    await this.page.getByLabel('Password').fill(password);
    await this.page.getByRole('button', { name: 'Sign in' }).click();
  }
}

// Test critical user journeys
test('user can complete checkout', async ({ page }) => {
  const loginPage = new LoginPage(page);
  await loginPage.login('user@test.com', 'password');
  // ...continue journey
});
```

### What to Test

| Test Type | Coverage Target | Focus On |
|-----------|-----------------|----------|
| Unit (Vitest) | Utility functions, hooks | Logic, edge cases |
| Component (Testing Library) | UI components | User interactions |
| Integration | Connected features | Data flow |
| E2E (Playwright) | Critical paths only | Full user journeys |

---

## Anti-Patterns

| Don't | Do Instead |
|-------|------------|
| `any` in TypeScript | `unknown` + type guards |
| Context for global state | Zustand/Jotai or composition |
| CSS-in-JS runtime (styled-components) | Tailwind or CSS Modules |
| Premature `useMemo`/`useCallback` | Profile first, then optimize |
| Giant components (500+ lines) | Extract, compose, simplify |
| Accessibility "later" | Build it in from the start |
| Inline styles everywhere | Design tokens + utility classes |
| Ignoring loading/error states | Handle all states explicitly |
