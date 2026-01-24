# Debugging & Performance Profiling

Senior-level debugging techniques and performance optimization patterns.

## Table of Contents
1. [React DevTools Profiler](#react-devtools-profiler)
2. [Identifying Re-renders](#identifying-re-renders)
3. [Bundle Analysis](#bundle-analysis)
4. [Memory Leak Detection](#memory-leak-detection)
5. [Core Web Vitals](#core-web-vitals)
6. [Network Debugging](#network-debugging)
7. [Common Performance Issues](#common-performance-issues)

---

## React DevTools Profiler

### Setup
1. Install React DevTools browser extension
2. Open DevTools → Profiler tab
3. Click record, interact with app, stop recording

### Reading the Flamegraph

```
Component render times:
├── Gray = Did not render
├── Blue/Teal = Fast render (< 1ms)
├── Yellow = Medium (1-16ms)
└── Red/Orange = Slow (> 16ms) ← Investigate these
```

### Key Metrics to Watch

- **Render duration**: Time spent in render phase
- **Commit count**: How many times component re-rendered
- **What caused render**: Props changed? Parent rendered? State?

### Profiler Settings

Enable these for better debugging:
- ✅ Record why each component rendered
- ✅ Highlight updates when components render

---

## Identifying Re-renders

### Quick Debug: Highlight Updates

React DevTools → Settings → Components → ✅ Highlight updates

Blue flash = component rendered. Frequent flashing = problem.

### Why Did You Render (Library)

```bash
npm install @welldone-software/why-did-you-render --save-dev
```

```tsx
// wdyr.ts (import first in app)
import React from 'react';
if (process.env.NODE_ENV === 'development') {
  const whyDidYouRender = require('@welldone-software/why-did-you-render');
  whyDidYouRender(React, { trackAllPureComponents: true });
}

// On specific component
MyComponent.whyDidYouRender = true;
```

### Common Re-render Causes

| Cause | Solution |
|-------|----------|
| Parent re-renders | Move state down, lift component up |
| New object/array in props | `useMemo` the value |
| New function in props | `useCallback` or move outside |
| Context value changes | Split contexts, memoize value |
| Inline object styles | Extract to constant or useMemo |

### Example: Fixing Unnecessary Re-renders

```tsx
// ❌ Bad: New object every render
<UserCard user={{ name, email }} />

// ✅ Good: Stable reference
const user = useMemo(() => ({ name, email }), [name, email]);
<UserCard user={user} />

// ❌ Bad: New function every render
<Button onClick={() => handleClick(id)} />

// ✅ Good: Stable function (if Button is memoized)
const handleClickMemo = useCallback(() => handleClick(id), [id]);
<Button onClick={handleClickMemo} />

// ✅ Better: Question if memo is even needed
// Often, re-renders are fine! Profile first.
```

---

## Bundle Analysis

### Vite Bundle Visualizer

```bash
npx vite-bundle-visualizer
```

Opens interactive treemap showing what's in your bundle.

### What to Look For

| Problem | Sign | Solution |
|---------|------|----------|
| Massive dependency | Large block in visualizer | Find lighter alternative |
| Duplicate packages | Same lib multiple times | Check package.json, dedupe |
| Unused code | Large chunks rarely loaded | Code split with lazy() |
| Full library import | lodash (70KB) | Import specific: lodash-es/debounce |
| Dev dependencies in prod | Test utils in bundle | Check imports |

### Code Splitting

```tsx
// Route-level splitting
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

// Feature-level splitting
const HeavyChart = lazy(() => import('./components/HeavyChart'));

// Conditional loading
const AdminPanel = lazy(() => 
  user.isAdmin ? import('./AdminPanel') : Promise.resolve({ default: () => null })
);
```

### Import Cost (VS Code Extension)

Shows size of imports inline. Install "Import Cost" extension.

```tsx
import { format } from 'date-fns'; // 15.2KB (gzipped: 5.3KB) ← shown inline
```

---

## Memory Leak Detection

### Chrome DevTools Memory Tab

1. Open DevTools → Memory
2. Take heap snapshot
3. Perform action (navigate, open modal, etc.)
4. Take another snapshot
5. Compare snapshots for growing objects

### Common Leak Patterns

```tsx
// ❌ Leak: Missing cleanup
useEffect(() => {
  const handler = () => { /* ... */ };
  window.addEventListener('resize', handler);
  // Missing cleanup!
}, []);

// ✅ Fixed
useEffect(() => {
  const handler = () => { /* ... */ };
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, []);

// ❌ Leak: Subscriptions not cleaned
useEffect(() => {
  const subscription = someObservable.subscribe(callback);
  // Missing unsubscribe!
}, []);

// ✅ Fixed
useEffect(() => {
  const subscription = someObservable.subscribe(callback);
  return () => subscription.unsubscribe();
}, []);

// ❌ Leak: Timers not cleared
useEffect(() => {
  setInterval(() => { /* ... */ }, 1000);
}, []);

// ✅ Fixed
useEffect(() => {
  const id = setInterval(() => { /* ... */ }, 1000);
  return () => clearInterval(id);
}, []);
```

### AbortController for Fetch

```tsx
useEffect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .then(setData)
    .catch(err => {
      if (err.name !== 'AbortError') throw err;
    });

  return () => controller.abort();
}, []);
```

---

## Core Web Vitals

### Target Metrics

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| LCP (Largest Contentful Paint) | ≤ 2.5s | 2.5s - 4s | > 4s |
| INP (Interaction to Next Paint) | ≤ 200ms | 200ms - 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | 0.1 - 0.25 | > 0.25 |

### Measuring

```bash
# Lighthouse CLI
npx lighthouse https://yoursite.com --view

# Or use web.dev/measure
```

### web-vitals Library

```tsx
import { onLCP, onINP, onCLS } from 'web-vitals';

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

### Fixing LCP

```tsx
// Preload critical images
<link rel="preload" as="image" href="/hero.webp" />

// Preconnect to external origins
<link rel="preconnect" href="https://fonts.googleapis.com" />

// Priority hints
<img src="/hero.webp" fetchPriority="high" />

// Avoid render-blocking resources
<script defer src="/analytics.js"></script>
```

### Fixing INP

```tsx
// ❌ Blocking interaction
function handleClick() {
  expensiveCalculation(); // Blocks for 300ms
  setResult(value);
}

// ✅ Non-blocking with scheduler
function handleClick() {
  // Show immediate feedback
  setIsProcessing(true);
  
  // Defer expensive work
  requestIdleCallback(() => {
    expensiveCalculation();
    setResult(value);
    setIsProcessing(false);
  });
}

// ✅ Using transition for non-urgent updates
function handleSearch(query) {
  setInputValue(query); // Urgent: update input immediately
  startTransition(() => {
    setSearchResults(search(query)); // Non-urgent: can be interrupted
  });
}
```

### Fixing CLS

```tsx
// ❌ Layout shift: no dimensions
<img src="/photo.jpg" />

// ✅ Reserve space
<img src="/photo.jpg" width={800} height={600} />

// ❌ Layout shift: dynamic content
{isLoaded && <Banner />}

// ✅ Reserve space with skeleton
{isLoaded ? <Banner /> : <BannerSkeleton />}

// ❌ Layout shift: fonts
// FOUT (Flash of Unstyled Text)

// ✅ Font-display and preload
<link rel="preload" href="/font.woff2" as="font" crossOrigin="" />
// + font-display: optional (prevents any shift)
```

---

## Network Debugging

### Chrome DevTools Network Tab

**Filters:**
- `larger-than:100kb` — Find large requests
- `is:from-cache` — See what's cached
- `-domain:googleapis.com` — Exclude domain
- `status-code:404` — Find broken requests

**Throttling:**
Test on slow connections: Fast 3G, Slow 3G, Offline

### Waterfall Analysis

```
Long green bar (TTFB)    → Server slow, optimize backend
Long blue bar (download) → Asset too large, compress
Many sequential requests → Parallelize or combine
Requests after DOMContentLoaded → Lazy load these
```

### Request Timing Breakdown

| Phase | Meaning | Fix |
|-------|---------|-----|
| Queueing | Waiting for connection | HTTP/2, fewer domains |
| Stalled | Browser limit | Fewer requests, domain sharding |
| DNS Lookup | Resolving domain | dns-prefetch |
| Initial Connection | TCP handshake | preconnect |
| SSL | TLS negotiation | Session resumption |
| TTFB | Server processing | Backend optimization |
| Content Download | Transfer time | Compression, CDN |

---

## Common Performance Issues

### 1. Expensive Render Calculations

```tsx
// ❌ Recalculates on every render
function ProductList({ products, filter }) {
  const filtered = products.filter(p => p.category === filter); // Every render!
  const sorted = filtered.sort((a, b) => a.price - b.price); // Every render!
  return sorted.map(p => <Product key={p.id} {...p} />);
}

// ✅ Memoize expensive calculations
function ProductList({ products, filter }) {
  const processedProducts = useMemo(() => {
    return products
      .filter(p => p.category === filter)
      .sort((a, b) => a.price - b.price);
  }, [products, filter]);
  
  return processedProducts.map(p => <Product key={p.id} {...p} />);
}
```

### 2. Large Lists

```tsx
// ❌ Rendering 10,000 items
<ul>
  {items.map(item => <li key={item.id}>{item.name}</li>)}
</ul>

// ✅ Virtualize with TanStack Virtual
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef<HTMLDivElement>(null);
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: 400, overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualItem => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: virtualItem.start,
              height: virtualItem.size,
            }}
          >
            {items[virtualItem.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 3. Unoptimized Images

```tsx
// ❌ Original 4MB image
<img src="/photo.jpg" />

// ✅ Optimized
<img
  src="/photo.webp"
  srcSet="/photo-400.webp 400w, /photo-800.webp 800w"
  sizes="(max-width: 600px) 400px, 800px"
  loading="lazy"
  decoding="async"
  width={800}
  height={600}
  alt="Description"
/>
```

### 4. Layout Thrashing

```tsx
// ❌ Reads and writes interleaved (forces layout recalc)
elements.forEach(el => {
  const height = el.offsetHeight; // Read
  el.style.height = height + 10 + 'px'; // Write
});

// ✅ Batch reads, then batch writes
const heights = elements.map(el => el.offsetHeight); // All reads
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px'; // All writes
});
```

### 5. Missing Error Boundaries

```tsx
// Wrap sections that might fail
<ErrorBoundary fallback={<ErrorMessage />}>
  <RiskyComponent />
</ErrorBoundary>

// Error boundary component
class ErrorBoundary extends Component {
  state = { hasError: false };
  
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    logErrorToService(error, info);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```
