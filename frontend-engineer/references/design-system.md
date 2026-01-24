# Design System Implementation

Building and maintaining scalable design systems.

## Table of Contents
1. [Design Token Architecture](#design-token-architecture)
2. [Component API Principles](#component-api-principles)
3. [Variant System with CVA](#variant-system-with-cva)
4. [Theme Implementation](#theme-implementation)
5. [Component Documentation](#component-documentation)
6. [Testing Components](#testing-components)

---

## Design Token Architecture

### Token Hierarchy

```
Primitive Tokens  →  Semantic Tokens  →  Component Tokens
    (raw values)      (purpose-based)     (component-specific)
    
    blue-500     →    color-primary   →    button-bg-primary
    space-4      →    spacing-md      →    card-padding
    radius-md    →    radius-default  →    input-radius
```

### Implementation in CSS

```css
@import "tailwindcss";

@theme {
  /* ═══════════════════════════════════════════════════════════
     PRIMITIVE TOKENS
     Raw values with no semantic meaning
     ═══════════════════════════════════════════════════════════ */
  
  /* Colors - OKLCH for perceptual uniformity */
  --color-gray-50: oklch(98% 0 0);
  --color-gray-100: oklch(95% 0 0);
  --color-gray-200: oklch(90% 0 0);
  --color-gray-300: oklch(82% 0 0);
  --color-gray-400: oklch(70% 0 0);
  --color-gray-500: oklch(55% 0 0);
  --color-gray-600: oklch(45% 0 0);
  --color-gray-700: oklch(35% 0 0);
  --color-gray-800: oklch(25% 0 0);
  --color-gray-900: oklch(15% 0 0);
  --color-gray-950: oklch(10% 0 0);
  
  --color-blue-500: oklch(55% 0.25 250);
  --color-blue-600: oklch(48% 0.25 250);
  --color-green-500: oklch(65% 0.2 145);
  --color-red-500: oklch(55% 0.22 25);
  --color-amber-500: oklch(75% 0.18 85);
  
  /* Spacing */
  --spacing-px: 1px;
  --spacing-0: 0;
  --spacing-0-5: 0.125rem;
  --spacing-1: 0.25rem;
  --spacing-2: 0.5rem;
  --spacing-3: 0.75rem;
  --spacing-4: 1rem;
  --spacing-5: 1.25rem;
  --spacing-6: 1.5rem;
  --spacing-8: 2rem;
  --spacing-10: 2.5rem;
  --spacing-12: 3rem;
  --spacing-16: 4rem;
  
  /* Radii */
  --radius-none: 0;
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  --radius-xl: 1rem;
  --radius-full: 9999px;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  
  /* ═══════════════════════════════════════════════════════════
     SEMANTIC TOKENS
     Purpose-based, theme-aware
     ═══════════════════════════════════════════════════════════ */
  
  /* Surfaces */
  --color-background: var(--color-gray-50);
  --color-foreground: var(--color-gray-900);
  --color-card: white;
  --color-card-foreground: var(--color-gray-900);
  --color-popover: white;
  --color-popover-foreground: var(--color-gray-900);
  
  /* Muted */
  --color-muted: var(--color-gray-100);
  --color-muted-foreground: var(--color-gray-500);
  
  /* Borders & Rings */
  --color-border: var(--color-gray-200);
  --color-input: var(--color-gray-200);
  --color-ring: var(--color-blue-500);
  
  /* Primary */
  --color-primary: var(--color-blue-500);
  --color-primary-hover: var(--color-blue-600);
  --color-primary-foreground: white;
  
  /* Secondary */
  --color-secondary: var(--color-gray-100);
  --color-secondary-hover: var(--color-gray-200);
  --color-secondary-foreground: var(--color-gray-900);
  
  /* Status */
  --color-destructive: var(--color-red-500);
  --color-destructive-foreground: white;
  --color-success: var(--color-green-500);
  --color-warning: var(--color-amber-500);
}

/* ═══════════════════════════════════════════════════════════
   DARK MODE
   ═══════════════════════════════════════════════════════════ */

.dark {
  --color-background: var(--color-gray-950);
  --color-foreground: var(--color-gray-50);
  --color-card: var(--color-gray-900);
  --color-card-foreground: var(--color-gray-50);
  --color-muted: var(--color-gray-800);
  --color-muted-foreground: var(--color-gray-400);
  --color-border: var(--color-gray-800);
  --color-input: var(--color-gray-800);
  --color-secondary: var(--color-gray-800);
  --color-secondary-hover: var(--color-gray-700);
}
```

---

## Component API Principles

### 1. Composition Over Configuration

```tsx
// ❌ Prop-heavy component
<Card
  title="Welcome"
  subtitle="Get started"
  icon={<StarIcon />}
  actions={[{ label: 'Learn More', onClick }]}
  footer={<Footer />}
/>

// ✅ Composable component
<Card>
  <Card.Header>
    <Card.Icon><StarIcon /></Card.Icon>
    <Card.Title>Welcome</Card.Title>
    <Card.Description>Get started</Card.Description>
  </Card.Header>
  <Card.Content>{/* ... */}</Card.Content>
  <Card.Footer>
    <Button>Learn More</Button>
  </Card.Footer>
</Card>
```

### 2. Forward Refs and Spread Props

```tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'default' | 'outline' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  asChild?: boolean;
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'default', size = 'md', asChild, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';
    return (
      <Comp
        ref={ref}
        className={cn(buttonVariants({ variant, size }), className)}
        {...props}
      />
    );
  }
);
Button.displayName = 'Button';
```

### 3. Sensible Defaults

```tsx
// Every prop has a reasonable default
<Button>Click me</Button>
// Equivalent to:
<Button variant="default" size="md" type="button">Click me</Button>
```

### 4. Escape Hatches

```tsx
// Allow className override for customization
<Button className="custom-class">Custom</Button>

// Allow style override for one-offs
<Button style={{ minWidth: 200 }}>Wide</Button>

// Allow asChild for polymorphism
<Button asChild>
  <Link href="/about">About</Link>
</Button>
```

---

## Variant System with CVA

### Setup

```bash
npm install class-variance-authority clsx tailwind-merge
```

```typescript
// lib/utils.ts
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Button Example

```tsx
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
  // Base styles (always applied)
  [
    'inline-flex items-center justify-center gap-2',
    'whitespace-nowrap rounded-md font-medium',
    'transition-colors duration-200',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2',
    'disabled:pointer-events-none disabled:opacity-50',
  ],
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary-hover',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-secondary',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary-hover',
        ghost: 'hover:bg-secondary hover:text-secondary-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        sm: 'h-8 px-3 text-xs',
        md: 'h-10 px-4 text-sm',
        lg: 'h-12 px-6 text-base',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'md',
    },
  }
);

interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  }
);
```

### Input Example

```tsx
const inputVariants = cva(
  [
    'flex w-full rounded-md border bg-transparent',
    'px-3 py-2 text-sm',
    'transition-colors duration-200',
    'file:border-0 file:bg-transparent file:text-sm file:font-medium',
    'placeholder:text-muted-foreground',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
    'disabled:cursor-not-allowed disabled:opacity-50',
  ],
  {
    variants: {
      variant: {
        default: 'border-input',
        error: 'border-destructive focus-visible:ring-destructive',
      },
      inputSize: {
        sm: 'h-8 text-xs',
        md: 'h-10',
        lg: 'h-12 text-base',
      },
    },
    defaultVariants: {
      variant: 'default',
      inputSize: 'md',
    },
  }
);
```

---

## Theme Implementation

### Theme Provider

```tsx
'use client';

import { createContext, useContext, useEffect, useState } from 'react';

type Theme = 'light' | 'dark' | 'system';

interface ThemeContextType {
  theme: Theme;
  setTheme: (theme: Theme) => void;
  resolvedTheme: 'light' | 'dark';
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('system');
  const [resolvedTheme, setResolvedTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    const stored = localStorage.getItem('theme') as Theme | null;
    if (stored) setTheme(stored);
  }, []);

  useEffect(() => {
    const root = document.documentElement;
    
    if (theme === 'system') {
      const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark'
        : 'light';
      setResolvedTheme(systemTheme);
      root.classList.toggle('dark', systemTheme === 'dark');
    } else {
      setResolvedTheme(theme);
      root.classList.toggle('dark', theme === 'dark');
    }
    
    localStorage.setItem('theme', theme);
  }, [theme]);

  // Listen for system theme changes
  useEffect(() => {
    if (theme !== 'system') return;
    
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    const handler = (e: MediaQueryListEvent) => {
      setResolvedTheme(e.matches ? 'dark' : 'light');
      document.documentElement.classList.toggle('dark', e.matches);
    };
    
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [theme]);

  return (
    <ThemeContext.Provider value={{ theme, setTheme, resolvedTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) throw new Error('useTheme must be used within ThemeProvider');
  return context;
}
```

### Theme Toggle Component

```tsx
function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <button
      onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      aria-label={`Switch to ${theme === 'light' ? 'dark' : 'light'} mode`}
    >
      {theme === 'light' ? <MoonIcon /> : <SunIcon />}
    </button>
  );
}
```

---

## Component Documentation

### Storybook Setup

```bash
npx storybook@latest init
```

### Story Template

```tsx
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'Primary UI button component with multiple variants and sizes.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['default', 'destructive', 'outline', 'secondary', 'ghost', 'link'],
      description: 'The visual style of the button',
    },
    size: {
      control: 'select',
      options: ['sm', 'md', 'lg', 'icon'],
      description: 'The size of the button',
    },
    disabled: {
      control: 'boolean',
      description: 'Whether the button is disabled',
    },
    asChild: {
      control: 'boolean',
      description: 'Merge props onto child element instead of rendering a button',
    },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Default: Story = {
  args: {
    children: 'Button',
  },
};

export const Variants: Story = {
  render: () => (
    <div className="flex flex-wrap gap-4">
      <Button variant="default">Default</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="destructive">Destructive</Button>
      <Button variant="outline">Outline</Button>
      <Button variant="ghost">Ghost</Button>
      <Button variant="link">Link</Button>
    </div>
  ),
};

export const Sizes: Story = {
  render: () => (
    <div className="flex items-center gap-4">
      <Button size="sm">Small</Button>
      <Button size="md">Medium</Button>
      <Button size="lg">Large</Button>
      <Button size="icon"><PlusIcon /></Button>
    </div>
  ),
};

export const WithIcon: Story = {
  render: () => (
    <div className="flex gap-4">
      <Button><MailIcon className="h-4 w-4" /> Send Email</Button>
      <Button variant="outline">Download <DownloadIcon className="h-4 w-4" /></Button>
    </div>
  ),
};

export const Loading: Story = {
  render: () => (
    <Button disabled>
      <Spinner className="h-4 w-4 animate-spin" />
      Loading...
    </Button>
  ),
};

export const AsLink: Story = {
  render: () => (
    <Button asChild>
      <a href="/about">Go to About</a>
    </Button>
  ),
};
```

---

## Testing Components

### Vitest + React Testing Library

```tsx
// Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders children correctly', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await userEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledOnce();
  });

  it('does not call onClick when disabled', async () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick} disabled>Click me</Button>);
    
    await userEvent.click(screen.getByRole('button'));
    expect(handleClick).not.toHaveBeenCalled();
  });

  it('applies variant classes correctly', () => {
    render(<Button variant="destructive">Delete</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-destructive');
  });

  it('forwards ref correctly', () => {
    const ref = { current: null };
    render(<Button ref={ref}>Click me</Button>);
    expect(ref.current).toBeInstanceOf(HTMLButtonElement);
  });

  it('renders as child element when asChild is true', () => {
    render(
      <Button asChild>
        <a href="/about">About</a>
      </Button>
    );
    
    const link = screen.getByRole('link', { name: /about/i });
    expect(link).toBeInTheDocument();
    expect(link).toHaveAttribute('href', '/about');
  });
});
```

### Accessibility Testing

```tsx
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

it('has no accessibility violations', async () => {
  const { container } = render(<Button>Accessible Button</Button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

### Visual Regression with Playwright

```tsx
// button.spec.ts
import { test, expect } from '@playwright/test';

test('button variants match snapshots', async ({ page }) => {
  await page.goto('/storybook/iframe.html?id=components-button--variants');
  await expect(page).toHaveScreenshot('button-variants.png');
});
```
