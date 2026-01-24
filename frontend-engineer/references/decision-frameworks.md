# Decision Frameworks

Senior-level guidance for technology and architecture decisions.

## Table of Contents
1. [State Management](#state-management)
2. [Server vs Client Components](#server-vs-client-components)
3. [CSS Approach](#css-approach)
4. [Component Libraries](#component-libraries)
5. [Build vs Use Library](#build-vs-use-library)
6. [Animation Library](#animation-library)
7. [Form Handling](#form-handling)
8. [Data Fetching](#data-fetching)

---

## State Management

### Decision Tree

```
Is it server data (from API)?
├── Yes → TanStack Query (or SWR)
└── No → Is it needed across multiple unrelated components?
    ├── Yes → Is it complex with many actions?
    │   ├── Yes → Zustand with slices
    │   └── No → Zustand (simple store)
    └── No → Is it needed by nested children?
        ├── Yes (2-3 levels) → Props or composition
        ├── Yes (deep tree) → Context (read-heavy) or Zustand
        └── No → useState / useReducer
```

### Recommendations by Use Case

| Use Case | Solution | Why |
|----------|----------|-----|
| Server data (CRUD) | TanStack Query | Caching, refetching, optimistic updates |
| Global UI state (theme, sidebar) | Zustand | Simple, no boilerplate, works outside React |
| Form state | React Hook Form | Performance, validation, field arrays |
| URL state | nuqs or useSearchParams | Shareable, bookmarkable |
| Complex local state | useReducer | Predictable state transitions |
| Cross-component communication | Zustand or Event emitter | Decoupled, testable |

### When NOT to Use State Management Libraries

- Prop drilling through 2-3 levels → Just pass props
- Data that's only used in one component tree → useState
- Data that should be in URL → useSearchParams
- Server data → TanStack Query (it's not "state management")

---

## Server vs Client Components

### Default: Server Components

React Server Components should be your default. Add `'use client'` only when needed.

### Use Client Component When:

```tsx
'use client' // Required for:

// 1. Event handlers
<button onClick={() => {}}>Click</button>

// 2. State
const [count, setCount] = useState(0);

// 3. Effects
useEffect(() => { /* browser APIs */ }, []);

// 4. Browser-only APIs
localStorage.getItem('key');
window.addEventListener('resize', handler);

// 5. Custom hooks that use above
const { data } = useQuery(/* ... */);

// 6. Third-party components requiring client
import { motion } from 'framer-motion';
```

### Keep as Server Component When:

- Fetching data
- Accessing backend resources (database, file system)
- Keeping sensitive info server-side (API keys, tokens)
- Large dependencies that shouldn't ship to client
- Static content that doesn't need interactivity

### Composition Pattern

```tsx
// page.tsx (Server Component)
async function ProductPage({ id }) {
  const product = await db.products.findUnique({ where: { id } });
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      {/* Client island for interactivity */}
      <AddToCartButton productId={id} />
    </div>
  );
}

// AddToCartButton.tsx (Client Component)
'use client';
export function AddToCartButton({ productId }) {
  const [pending, startTransition] = useTransition();
  return (
    <button onClick={() => startTransition(() => addToCart(productId))}>
      {pending ? 'Adding...' : 'Add to Cart'}
    </button>
  );
}
```

---

## CSS Approach

### Decision Matrix

| Need | Approach |
|------|----------|
| Rapid development, utility-first | Tailwind CSS 4 |
| Component-scoped styles | CSS Modules |
| Complex animations | CSS + Motion library |
| Critical CSS, zero-runtime | Tailwind or vanilla CSS |
| Theme customization | CSS custom properties |
| Design system tokens | Tailwind @theme or CSS vars |

### Avoid in 2025

- **Runtime CSS-in-JS** (styled-components, Emotion): Performance overhead, SSR complexity
- **Sass/Less**: CSS has native nesting, variables, calc() now
- **BEM with global CSS**: Scoping issues at scale

### Tailwind CSS 4 Setup

```css
/* app.css */
@import "tailwindcss";

@theme {
  --color-brand: oklch(55% 0.25 250);
  --font-sans: "Inter Variable", system-ui, sans-serif;
  --radius-lg: 0.75rem;
}
```

### When to Use CSS Modules

- Working in a non-Tailwind codebase
- Need computed/dynamic styles beyond Tailwind's capabilities
- Team prefers traditional CSS authoring
- Component library that shouldn't impose Tailwind

```tsx
// Button.module.css
.button { /* styles */ }
.button[data-variant="primary"] { /* variant */ }

// Button.tsx
import styles from './Button.module.css';
<button className={styles.button} data-variant={variant} />
```

---

## Component Libraries

### Decision Tree

```
Need complete design system out of the box?
├── Yes → Ant Design (enterprise) or MUI (Material)
└── No → Want full styling control?
    ├── Yes → Need accessibility built-in?
    │   ├── Yes, long-term project → React Aria
    │   ├── Yes, quick start → shadcn/ui
    │   └── No → Build from scratch
    └── No → Prefer minimal bundle?
        ├── Yes → Headless UI
        └── No → shadcn/ui (copy-paste ownership)
```

### Comparison

| Library | Styled? | Accessible? | Bundle | Control | Best For |
|---------|---------|-------------|--------|---------|----------|
| shadcn/ui | Yes (Tailwind) | Yes (Radix) | 0 (copy) | Full | Most projects |
| React Aria | No | Excellent | Tree-shake | Full | Enterprise, long-term |
| Radix UI | No | Yes | Tree-shake | Full | Custom design systems |
| Headless UI | No | Yes | Small | Full | Tailwind projects |
| MUI | Yes | Good | Large | Medium | Material Design apps |
| Ant Design | Yes | Good | Large | Medium | Enterprise dashboards |

### Recommendation for 2025

**Default choice: shadcn/ui**
- Copy components into your codebase (no dependency)
- Built on Radix UI primitives
- Tailwind styling you fully control
- Active ecosystem, AI tools generate shadcn code

**For long-term/enterprise: React Aria**
- Adobe-backed, guaranteed maintenance
- Best-in-class accessibility
- More work to style, but rock-solid foundation

---

## Build vs Use Library

### Build Custom When:

- Core differentiator for your product
- Existing libraries don't fit your exact needs
- You have the time and expertise to maintain it
- Learning is a goal (side projects)

### Use Library When:

- Solved problem (forms, tables, date pickers)
- Security-sensitive (auth, payments)
- Accessibility-critical (modals, dropdowns, menus)
- Time-constrained
- Team doesn't have domain expertise

### Specific Recommendations

| Feature | Build? | Use |
|---------|--------|-----|
| Button, Input | ✅ Build | — |
| Modal, Dialog | ❌ | Radix/React Aria |
| Dropdown menu | ❌ | Radix/React Aria |
| Data table | ❌ | TanStack Table |
| Date picker | ❌ | React Aria DatePicker |
| Rich text editor | ❌ | TipTap or Lexical |
| Drag and drop | ❌ | dnd-kit |
| Command palette | ❌ | cmdk |
| Charts | ❌ | Recharts or Visx |
| Forms | ❌ | React Hook Form |
| Virtual lists | ❌ | TanStack Virtual |

---

## Animation Library

### Decision Tree

```
What type of animation?
├── Simple transitions (hover, enter/exit)
│   └── CSS transitions/animations OR Motion
├── Layout animations
│   └── Motion (layout prop, AnimatePresence)
├── Gesture-based (drag, swipe)
│   └── Motion
├── Complex timeline sequences
│   └── GSAP
├── Scroll-driven
│   ├── Simple parallax → CSS scroll-timeline
│   └── Complex scroll scenes → GSAP ScrollTrigger
├── SVG morphing
│   └── GSAP MorphSVG
└── 3D / WebGL
    └── Three.js + React Three Fiber
```

### Quick Reference

| Library | Size | Strength | Weakness |
|---------|------|----------|----------|
| CSS | 0 | Performance, simple | Limited sequencing |
| Motion | ~32KB | React integration, layout | Complex timelines |
| GSAP | ~23KB | Power, control | Imperative, cleanup needed |
| React Spring | ~20KB | Physics-based | Learning curve |

---

## Form Handling

### React Hook Form + Zod (Recommended)

```tsx
const schema = z.object({
  email: z.string().email(),
  age: z.number().min(18),
});

const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: zodResolver(schema),
});
```

### When to Use Native Forms

- Very simple forms (1-3 fields, no validation)
- Progressive enhancement required
- Server Actions (can use native `<form action={}>`)

### When to Use React Hook Form

- Complex validation
- Dynamic fields (field arrays)
- Performance matters (many fields)
- Multi-step wizards
- Need touched/dirty state

---

## Data Fetching

### Decision Tree

```
Where is the data fetched?
├── Server (SSR/RSC)
│   └── fetch() or ORM directly
└── Client
    └── Needs caching, refetching, optimistic updates?
        ├── Yes → TanStack Query
        └── No, simple one-off → fetch in useEffect
```

### TanStack Query Patterns

```tsx
// Read
const { data, isLoading, error } = useQuery({
  queryKey: ['products', filters],
  queryFn: () => fetchProducts(filters),
  staleTime: 5 * 60 * 1000, // 5 min
});

// Mutation with optimistic update
const mutation = useMutation({
  mutationFn: updateProduct,
  onMutate: async (newData) => {
    await queryClient.cancelQueries(['products']);
    const previous = queryClient.getQueryData(['products']);
    queryClient.setQueryData(['products'], (old) => /* optimistic */);
    return { previous };
  },
  onError: (err, _, context) => {
    queryClient.setQueryData(['products'], context.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries(['products']);
  },
});
```

### Server Actions (Next.js 14+)

```tsx
// actions.ts
'use server';
export async function createPost(formData: FormData) {
  const title = formData.get('title');
  await db.posts.create({ data: { title } });
  revalidatePath('/posts');
}

// Component
<form action={createPost}>
  <input name="title" />
  <button type="submit">Create</button>
</form>
```
