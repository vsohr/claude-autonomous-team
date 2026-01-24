# Advanced Component Patterns

Implementation patterns for complex UI components.

## Table of Contents
1. [Compound Components](#compound-components)
2. [Custom Hooks](#custom-hooks)
3. [Form Patterns](#form-patterns)
4. [Modal & Dialog](#modal--dialog)
5. [Data Tables](#data-tables)
6. [Infinite Scroll & Pagination](#infinite-scroll--pagination)
7. [Drag and Drop](#drag-and-drop)
8. [Command Palette](#command-palette)
9. [File Upload](#file-upload)
10. [Error Boundaries](#error-boundaries)
11. [Multi-Step Forms](#multi-step-forms)
12. [Optimistic Updates](#optimistic-updates)

---

## Compound Components

Components that share implicit state.

```tsx
// Usage
<Tabs defaultValue="tab1">
  <Tabs.List>
    <Tabs.Trigger value="tab1">Account</Tabs.Trigger>
    <Tabs.Trigger value="tab2">Settings</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Content value="tab1">Account content</Tabs.Content>
  <Tabs.Content value="tab2">Settings content</Tabs.Content>
</Tabs>

// Implementation
const TabsContext = createContext<TabsContextType | null>(null);

function Tabs({ defaultValue, children }: TabsProps) {
  const [value, setValue] = useState(defaultValue);
  return (
    <TabsContext.Provider value={{ value, setValue }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

Tabs.List = function TabsList({ children }: { children: ReactNode }) {
  return <div role="tablist" className="flex border-b">{children}</div>;
};

Tabs.Trigger = function TabsTrigger({ value, children }: TriggerProps) {
  const ctx = useContext(TabsContext)!;
  return (
    <button
      role="tab"
      aria-selected={ctx.value === value}
      onClick={() => ctx.setValue(value)}
      className={cn(ctx.value === value && 'border-b-2 border-primary')}
    >
      {children}
    </button>
  );
};

Tabs.Content = function TabsContent({ value, children }: ContentProps) {
  const ctx = useContext(TabsContext)!;
  if (ctx.value !== value) return null;
  return <div role="tabpanel">{children}</div>;
};
```

---

## Custom Hooks

### useDebounce
```tsx
function useDebounce<T>(value: T, delay = 300): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
const debouncedSearch = useDebounce(searchTerm, 300);
useEffect(() => {
  if (debouncedSearch) fetchResults(debouncedSearch);
}, [debouncedSearch]);
```

### useLocalStorage
```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = useCallback((value: T | ((prev: T) => T)) => {
    setStoredValue(prev => {
      const valueToStore = value instanceof Function ? value(prev) : value;
      localStorage.setItem(key, JSON.stringify(valueToStore));
      return valueToStore;
    });
  }, [key]);

  return [storedValue, setValue] as const;
}
```

### useMediaQuery
```tsx
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() => {
    if (typeof window === 'undefined') return false;
    return window.matchMedia(query).matches;
  });

  useEffect(() => {
    const media = window.matchMedia(query);
    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);

  return matches;
}

// Usage
const isMobile = useMediaQuery('(max-width: 768px)');
const prefersReducedMotion = useMediaQuery('(prefers-reduced-motion: reduce)');
```

### useClickOutside
```tsx
function useClickOutside<T extends HTMLElement>(
  handler: () => void
): RefObject<T> {
  const ref = useRef<T>(null);

  useEffect(() => {
    const listener = (event: MouseEvent | TouchEvent) => {
      if (!ref.current || ref.current.contains(event.target as Node)) return;
      handler();
    };

    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [handler]);

  return ref;
}
```

### useCopyToClipboard
```tsx
function useCopyToClipboard() {
  const [copiedText, setCopiedText] = useState<string | null>(null);

  const copy = useCallback(async (text: string) => {
    try {
      await navigator.clipboard.writeText(text);
      setCopiedText(text);
      setTimeout(() => setCopiedText(null), 2000);
      return true;
    } catch {
      setCopiedText(null);
      return false;
    }
  }, []);

  return { copiedText, copy };
}
```

---

## Form Patterns

### React Hook Form + Zod
```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Minimum 8 characters'),
  confirmPassword: z.string(),
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ['confirmPassword'],
});

type FormData = z.infer<typeof schema>;

function SignupForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<FormData>({
    resolver: zodResolver(schema),
  });

  const onSubmit = async (data: FormData) => {
    await signUp(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} noValidate>
      <div>
        <label htmlFor="email">Email</label>
        <input id="email" type="email" {...register('email')} />
        {errors.email && <span role="alert">{errors.email.message}</span>}
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input id="password" type="password" {...register('password')} />
        {errors.password && <span role="alert">{errors.password.message}</span>}
      </div>
      
      <div>
        <label htmlFor="confirmPassword">Confirm Password</label>
        <input id="confirmPassword" type="password" {...register('confirmPassword')} />
        {errors.confirmPassword && <span role="alert">{errors.confirmPassword.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Signing up...' : 'Sign Up'}
      </button>
    </form>
  );
}
```

### Field Array (Dynamic Fields)
```tsx
import { useFieldArray } from 'react-hook-form';

function TeamMembersForm() {
  const { control, register } = useForm({
    defaultValues: { members: [{ name: '', email: '' }] },
  });

  const { fields, append, remove } = useFieldArray({
    control,
    name: 'members',
  });

  return (
    <form>
      {fields.map((field, index) => (
        <div key={field.id}>
          <input {...register(`members.${index}.name`)} placeholder="Name" />
          <input {...register(`members.${index}.email`)} placeholder="Email" />
          <button type="button" onClick={() => remove(index)}>Remove</button>
        </div>
      ))}
      <button type="button" onClick={() => append({ name: '', email: '' })}>
        Add Member
      </button>
    </form>
  );
}
```

---

## Modal & Dialog

### Accessible Modal with Focus Trap
```tsx
import { createPortal } from 'react-dom';
import { useEffect, useRef, useCallback } from 'react';

interface ModalProps {
  open: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
}

function Modal({ open, onClose, title, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousActiveElement = useRef<Element | null>(null);

  // Store and restore focus
  useEffect(() => {
    if (open) {
      previousActiveElement.current = document.activeElement;
      modalRef.current?.focus();
    } else {
      (previousActiveElement.current as HTMLElement)?.focus();
    }
  }, [open]);

  // Lock body scroll
  useEffect(() => {
    if (open) {
      const originalStyle = window.getComputedStyle(document.body).overflow;
      document.body.style.overflow = 'hidden';
      return () => { document.body.style.overflow = originalStyle; };
    }
  }, [open]);

  // Handle escape key
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
    };
    if (open) {
      document.addEventListener('keydown', handleKeyDown);
      return () => document.removeEventListener('keydown', handleKeyDown);
    }
  }, [open, onClose]);

  // Focus trap
  const handleKeyDown = useCallback((e: React.KeyboardEvent) => {
    if (e.key !== 'Tab') return;
    
    const focusable = modalRef.current?.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    if (!focusable?.length) return;

    const first = focusable[0] as HTMLElement;
    const last = focusable[focusable.length - 1] as HTMLElement;

    if (e.shiftKey && document.activeElement === first) {
      e.preventDefault();
      last.focus();
    } else if (!e.shiftKey && document.activeElement === last) {
      e.preventDefault();
      first.focus();
    }
  }, []);

  if (!open) return null;

  return createPortal(
    <div
      className="fixed inset-0 z-50 flex items-center justify-center"
      onClick={(e) => e.target === e.currentTarget && onClose()}
    >
      <div className="fixed inset-0 bg-black/50" aria-hidden="true" />
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
        onKeyDown={handleKeyDown}
        className="relative bg-white rounded-lg shadow-xl max-w-md w-full mx-4 p-6"
      >
        <h2 id="modal-title" className="text-lg font-semibold">{title}</h2>
        <button
          onClick={onClose}
          className="absolute top-4 right-4"
          aria-label="Close"
        >
          ✕
        </button>
        {children}
      </div>
    </div>,
    document.body
  );
}
```

---

## Data Tables

### TanStack Table with Sorting & Filtering
```tsx
import {
  useReactTable,
  getCoreRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  flexRender,
  type ColumnDef,
  type SortingState,
} from '@tanstack/react-table';

interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
}

function DataTable<T>({ data, columns }: DataTableProps<T>) {
  const [sorting, setSorting] = useState<SortingState>([]);
  const [globalFilter, setGlobalFilter] = useState('');

  const table = useReactTable({
    data,
    columns,
    state: { sorting, globalFilter },
    onSortingChange: setSorting,
    onGlobalFilterChange: setGlobalFilter,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
  });

  return (
    <div>
      <input
        value={globalFilter}
        onChange={e => setGlobalFilter(e.target.value)}
        placeholder="Search..."
        className="mb-4 p-2 border rounded"
      />
      
      <table className="w-full">
        <thead>
          {table.getHeaderGroups().map(headerGroup => (
            <tr key={headerGroup.id}>
              {headerGroup.headers.map(header => (
                <th
                  key={header.id}
                  onClick={header.column.getToggleSortingHandler()}
                  className="cursor-pointer select-none p-2 text-left"
                >
                  {flexRender(header.column.columnDef.header, header.getContext())}
                  {{ asc: ' ↑', desc: ' ↓' }[header.column.getIsSorted() as string] ?? null}
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody>
          {table.getRowModel().rows.map(row => (
            <tr key={row.id} className="border-t">
              {row.getVisibleCells().map(cell => (
                <td key={cell.id} className="p-2">
                  {flexRender(cell.column.columnDef.cell, cell.getContext())}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## Infinite Scroll & Pagination

### Infinite Scroll with TanStack Query
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
import { useInView } from 'react-intersection-observer';

function InfiniteList() {
  const { ref, inView } = useInView();

  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    isLoading,
  } = useInfiniteQuery({
    queryKey: ['items'],
    queryFn: ({ pageParam = 0 }) => fetchItems({ cursor: pageParam, limit: 20 }),
    getNextPageParam: (lastPage) => lastPage.nextCursor,
  });

  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage();
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isLoading) return <Skeleton />;

  return (
    <div>
      {data?.pages.flatMap(page => page.items).map(item => (
        <ItemCard key={item.id} item={item} />
      ))}
      
      <div ref={ref} className="h-10">
        {isFetchingNextPage && <Spinner />}
      </div>
    </div>
  );
}
```

---

## Drag and Drop

### dnd-kit Sortable List
```bash
npm install @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities
```

```tsx
import {
  DndContext,
  closestCenter,
  KeyboardSensor,
  PointerSensor,
  useSensor,
  useSensors,
} from '@dnd-kit/core';
import {
  arrayMove,
  SortableContext,
  sortableKeyboardCoordinates,
  useSortable,
  verticalListSortingStrategy,
} from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';

function SortableItem({ id, children }: { id: string; children: ReactNode }) {
  const { attributes, listeners, setNodeRef, transform, transition } = useSortable({ id });

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
  };

  return (
    <div ref={setNodeRef} style={style} {...attributes} {...listeners}>
      {children}
    </div>
  );
}

function SortableList() {
  const [items, setItems] = useState(['1', '2', '3', '4']);
  
  const sensors = useSensors(
    useSensor(PointerSensor),
    useSensor(KeyboardSensor, { coordinateGetter: sortableKeyboardCoordinates })
  );

  function handleDragEnd(event: DragEndEvent) {
    const { active, over } = event;
    if (over && active.id !== over.id) {
      setItems((items) => {
        const oldIndex = items.indexOf(active.id as string);
        const newIndex = items.indexOf(over.id as string);
        return arrayMove(items, oldIndex, newIndex);
      });
    }
  }

  return (
    <DndContext sensors={sensors} collisionDetection={closestCenter} onDragEnd={handleDragEnd}>
      <SortableContext items={items} strategy={verticalListSortingStrategy}>
        {items.map(id => (
          <SortableItem key={id} id={id}>
            Item {id}
          </SortableItem>
        ))}
      </SortableContext>
    </DndContext>
  );
}
```

---

## Command Palette

### Using cmdk
```bash
npm install cmdk
```

```tsx
import { Command } from 'cmdk';

function CommandPalette() {
  const [open, setOpen] = useState(false);

  useEffect(() => {
    const down = (e: KeyboardEvent) => {
      if (e.key === 'k' && (e.metaKey || e.ctrlKey)) {
        e.preventDefault();
        setOpen(open => !open);
      }
    };
    document.addEventListener('keydown', down);
    return () => document.removeEventListener('keydown', down);
  }, []);

  return (
    <Command.Dialog open={open} onOpenChange={setOpen} label="Command Menu">
      <Command.Input placeholder="Type a command or search..." />
      <Command.List>
        <Command.Empty>No results found.</Command.Empty>
        
        <Command.Group heading="Actions">
          <Command.Item onSelect={() => console.log('New file')}>
            New File
          </Command.Item>
          <Command.Item onSelect={() => console.log('New folder')}>
            New Folder
          </Command.Item>
        </Command.Group>
        
        <Command.Group heading="Navigation">
          <Command.Item onSelect={() => navigate('/dashboard')}>
            Go to Dashboard
          </Command.Item>
          <Command.Item onSelect={() => navigate('/settings')}>
            Go to Settings
          </Command.Item>
        </Command.Group>
      </Command.List>
    </Command.Dialog>
  );
}
```

---

## File Upload

### Drag & Drop Upload with Preview
```tsx
function FileUpload({ onUpload }: { onUpload: (files: File[]) => void }) {
  const [isDragging, setIsDragging] = useState(false);
  const [files, setFiles] = useState<File[]>([]);

  const handleDrag = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
  };

  const handleDragIn = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(true);
  };

  const handleDragOut = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
  };

  const handleDrop = (e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setIsDragging(false);
    
    const droppedFiles = Array.from(e.dataTransfer.files);
    setFiles(prev => [...prev, ...droppedFiles]);
    onUpload(droppedFiles);
  };

  const handleFileInput = (e: React.ChangeEvent<HTMLInputElement>) => {
    const selectedFiles = Array.from(e.target.files || []);
    setFiles(prev => [...prev, ...selectedFiles]);
    onUpload(selectedFiles);
  };

  return (
    <div
      onDragEnter={handleDragIn}
      onDragLeave={handleDragOut}
      onDragOver={handleDrag}
      onDrop={handleDrop}
      className={cn(
        'border-2 border-dashed rounded-lg p-8 text-center transition-colors',
        isDragging ? 'border-primary bg-primary/5' : 'border-muted'
      )}
    >
      <input
        type="file"
        multiple
        onChange={handleFileInput}
        className="hidden"
        id="file-input"
      />
      <label htmlFor="file-input" className="cursor-pointer">
        <p>Drag files here or click to upload</p>
      </label>
      
      {files.length > 0 && (
        <ul className="mt-4 text-left">
          {files.map((file, i) => (
            <li key={i} className="flex items-center gap-2">
              <span>{file.name}</span>
              <span className="text-muted-foreground">
                ({(file.size / 1024).toFixed(1)} KB)
              </span>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## Error Boundaries

### Class-based Error Boundary
```tsx
interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = { hasError: false };

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.props.onError?.(error, errorInfo);
    // Log to error reporting service
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="p-4 border border-destructive rounded bg-destructive/10">
          <h2 className="font-semibold text-destructive">Something went wrong</h2>
          <p className="text-sm text-muted-foreground mt-1">
            {this.state.error?.message}
          </p>
          <button
            onClick={() => this.setState({ hasError: false, error: undefined })}
            className="mt-2 text-sm underline"
          >
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorFallback />} onError={logToSentry}>
  <RiskyComponent />
</ErrorBoundary>
```

---

## Multi-Step Forms

### Wizard Pattern
```tsx
interface WizardStep {
  id: string;
  title: string;
  component: ComponentType<{ next: () => void; prev: () => void; data: any }>;
  validation?: z.ZodSchema;
}

function Wizard({ steps }: { steps: WizardStep[] }) {
  const [currentStep, setCurrentStep] = useState(0);
  const [data, setData] = useState({});

  const next = () => setCurrentStep(prev => Math.min(prev + 1, steps.length - 1));
  const prev = () => setCurrentStep(prev => Math.max(prev - 1, 0));
  const updateData = (newData: object) => setData(prev => ({ ...prev, ...newData }));

  const CurrentComponent = steps[currentStep].component;

  return (
    <div>
      {/* Progress indicator */}
      <div className="flex mb-8">
        {steps.map((step, index) => (
          <div
            key={step.id}
            className={cn(
              'flex-1 h-2 mx-1 rounded',
              index <= currentStep ? 'bg-primary' : 'bg-muted'
            )}
          />
        ))}
      </div>

      {/* Step title */}
      <h2 className="text-xl font-semibold mb-4">
        Step {currentStep + 1}: {steps[currentStep].title}
      </h2>

      {/* Current step component */}
      <CurrentComponent next={next} prev={prev} data={data} updateData={updateData} />

      {/* Navigation */}
      <div className="flex justify-between mt-6">
        <button onClick={prev} disabled={currentStep === 0}>
          Back
        </button>
        <span>{currentStep + 1} / {steps.length}</span>
      </div>
    </div>
  );
}

// Example step component
function PersonalInfoStep({ next, data, updateData }: StepProps) {
  const { register, handleSubmit } = useForm({ defaultValues: data });

  const onSubmit = (formData: any) => {
    updateData(formData);
    next();
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('firstName')} placeholder="First Name" />
      <input {...register('lastName')} placeholder="Last Name" />
      <button type="submit">Continue</button>
    </form>
  );
}
```

---

## Optimistic Updates

### TanStack Query Optimistic Pattern
```tsx
function TodoItem({ todo }: { todo: Todo }) {
  const queryClient = useQueryClient();

  const toggleMutation = useMutation({
    mutationFn: (completed: boolean) => updateTodo(todo.id, { completed }),
    
    // Optimistic update
    onMutate: async (completed) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: ['todos'] });
      
      // Snapshot previous value
      const previousTodos = queryClient.getQueryData<Todo[]>(['todos']);
      
      // Optimistically update cache
      queryClient.setQueryData<Todo[]>(['todos'], (old) =>
        old?.map(t => t.id === todo.id ? { ...t, completed } : t) ?? []
      );
      
      // Return context for rollback
      return { previousTodos };
    },
    
    // Rollback on error
    onError: (err, _, context) => {
      if (context?.previousTodos) {
        queryClient.setQueryData(['todos'], context.previousTodos);
      }
      toast.error('Failed to update todo');
    },
    
    // Always refetch after error or success
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <div
      onClick={() => toggleMutation.mutate(!todo.completed)}
      className={cn(toggleMutation.isPending && 'opacity-50')}
    >
      <input type="checkbox" checked={todo.completed} readOnly />
      <span className={cn(todo.completed && 'line-through')}>{todo.title}</span>
    </div>
  );
}
```
