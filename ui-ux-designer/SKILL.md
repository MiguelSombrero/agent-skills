---
name: ui-ux-designer
description: Designs exceptional user interfaces and experiences for web applications with expertise in usability, accessibility, performance, visual design, and component architecture. Use when designing UI components, layouts, user flows, design systems, or improving user experience. Focuses on React/TypeScript/Next.js web applications with emphasis on responsive design, accessibility, and performance.
---

# UI/UX Designer

Creates user-centered, accessible, and performant web interfaces that delight users while maintaining code quality and scalability. Combines design thinking with technical implementation knowledge to build exceptional web experiences.

## When to Use This Skill

Use this skill when the user:

- Wants to **design or improve UI components** or layouts
- Asks about **user experience**, **usability**, or **user flows**
- Needs help with **responsive design** or **mobile-first** approaches
- Wants to improve **accessibility** (WCAG, ARIA, keyboard navigation)
- Asks about **visual design** (colors, typography, spacing, hierarchy)
- Needs guidance on **component structure** or **design systems**
- Wants to optimize **UI performance** (rendering, animations, loading)
- Asks about **interaction design** (hover states, transitions, feedback)
- Needs help with **forms**, **validation UX**, or **error handling**
- Wants to improve **navigation** or **information architecture**
- Asks "how should this look/work" or "what's the best UX for..."
- Mentions **user research**, **personas**, or **user journeys**
- Wants to create a **design system** or **component library**

## Scope

**In Scope** (Web UI/UX):

- UI component design and structure
- User experience and usability principles
- Responsive web design
- Web accessibility (WCAG)
- Visual design (typography, color, spacing)
- Interaction design (animations, transitions)
- Performance optimization
- Design systems and component libraries
- User flows and information architecture

**Out of Scope**:

- Native mobile app design (iOS/Android specific)
- Desktop application UI (Electron, etc.)
- Game UI design
- 3D interfaces or VR/AR
- Marketing/branding strategy
- Business model design

**Note**: While mobile-specific examples are out of scope, many principles (accessibility, usability, performance) apply to both web and mobile.

---

## Core UX Principles

### User-Centered Design

**Always design for your users, not for yourself.**

#### User Research & Personas

```typescript
// Example: Define user personas to guide design decisions
interface UserPersona {
  name: string;
  role: string;
  goals: string[];
  painPoints: string[];
  techSavviness: "low" | "medium" | "high";
  devices: string[];
}

const primaryPersona: UserPersona = {
  name: "Sarah, Small Business Owner",
  role: "Non-technical user",
  goals: [
    "Quickly update inventory",
    "View sales reports",
    "Manage customer orders",
  ],
  painPoints: [
    "Complex interfaces are frustrating",
    "Limited time to learn new tools",
    "Needs mobile access on the go",
  ],
  techSavviness: "medium",
  devices: ["laptop", "tablet", "smartphone"],
};

// Design decisions based on persona:
// ✅ Simple, clear interface
// ✅ Mobile-first responsive design
// ✅ Clear visual hierarchy
// ✅ Minimal steps to complete tasks
// ✅ Contextual help and tooltips
```

#### User Journey Mapping

```typescript
// Map critical user journeys
interface UserJourney {
  step: number;
  action: string;
  touchpoint: string;
  emotion: "frustrated" | "neutral" | "satisfied" | "delighted";
  painPoints?: string[];
  opportunities?: string[];
}

const checkoutJourney: UserJourney[] = [
  {
    step: 1,
    action: "Add item to cart",
    touchpoint: "Product page",
    emotion: "satisfied",
    opportunities: ["Show cart preview on add"],
  },
  {
    step: 2,
    action: "Review cart",
    touchpoint: "Cart page",
    emotion: "neutral",
    painPoints: ["Can't edit quantity easily"],
    opportunities: ["Inline quantity editing", "Save for later option"],
  },
  {
    step: 3,
    action: "Enter shipping info",
    touchpoint: "Checkout form",
    emotion: "frustrated",
    painPoints: ["Long form", "No autofill", "Unclear required fields"],
    opportunities: [
      "Address autocomplete",
      "Save addresses",
      "Progressive disclosure",
    ],
  },
  {
    step: 4,
    action: "Complete payment",
    touchpoint: "Payment page",
    emotion: "delighted",
    opportunities: ["Multiple payment options", "Order summary visible"],
  },
];
```

### Usability Heuristics (Nielsen Norman)

#### 1. Visibility of System Status

**Keep users informed about what's happening.**

```tsx
// ❌ Bad - No feedback
function SubmitButton() {
  return <button onClick={handleSubmit}>Submit</button>;
}

// ✅ Good - Clear status feedback
function SubmitButton() {
  const [status, setStatus] = useState<
    "idle" | "loading" | "success" | "error"
  >("idle");

  return (
    <button
      onClick={handleSubmit}
      disabled={status === "loading"}
      className={cn(
        "px-4 py-2 rounded",
        status === "loading" && "opacity-50 cursor-not-allowed",
        status === "success" && "bg-green-500",
        status === "error" && "bg-red-500",
      )}
    >
      {status === "loading" && <Spinner className="mr-2" />}
      {status === "success" && <Check className="mr-2" />}
      {status === "error" && <X className="mr-2" />}
      {status === "loading" ? "Submitting..." : "Submit"}
    </button>
  );
}

// ✅ Good - Upload progress
function FileUpload() {
  const [progress, setProgress] = useState(0);

  return (
    <div className="space-y-2">
      <input type="file" onChange={handleUpload} />
      {progress > 0 && (
        <div className="w-full">
          <div className="flex justify-between text-sm mb-1">
            <span>Uploading...</span>
            <span>{progress}%</span>
          </div>
          <div className="w-full bg-gray-200 rounded-full h-2">
            <div
              className="bg-blue-500 h-2 rounded-full transition-all"
              style={{ width: `${progress}%` }}
            />
          </div>
        </div>
      )}
    </div>
  );
}
```

#### 2. Match Between System and Real World

**Speak the user's language, not technical jargon.**

```tsx
// ❌ Bad - Technical jargon
<button>Execute POST request</button>
<error>ERR_HTTP_429: Rate limit exceeded</error>

// ✅ Good - User-friendly language
<button>Save Changes</button>
<error>You're sending too many requests. Please wait a moment and try again.</error>

// ✅ Good - Real-world metaphors
function Trash({ onDelete }: { onDelete: () => void }) {
  return (
    <button onClick={onDelete} aria-label="Move to trash">
      <TrashIcon /> {/* Familiar icon */}
    </button>
  );
}
```

#### 3. User Control and Freedom

**Provide undo/redo, cancel, and easy exits.**

```tsx
// ✅ Good - Undo functionality
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [deletedTodo, setDeletedTodo] = useState<Todo | null>(null);

  const deleteTodo = (id: string) => {
    const todo = todos.find(t => t.id === id);
    if (todo) {
      setTodos(todos.filter(t => t.id !== id));
      setDeletedTodo(todo);

      // Show toast with undo option
      toast({
        title: 'Item deleted',
        action: (
          <button onClick={() => {
            setTodos([...todos, todo]);
            setDeletedTodo(null);
          }}>
            Undo
          </button>
        ),
        duration: 5000
      });
    }
  };

  return (/* ... */);
}

// ✅ Good - Easy modal exit
function Modal({ isOpen, onClose, children }: ModalProps) {
  return (
    <Dialog open={isOpen} onOpenChange={onClose}>
      <DialogContent>
        {/* Multiple ways to close */}
        <DialogClose asChild>
          <button className="absolute right-4 top-4">
            <X className="h-4 w-4" />
          </button>
        </DialogClose>
        {children}
        <DialogFooter>
          <button onClick={onClose}>Cancel</button>
          {/* ... */}
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

#### 4. Consistency and Standards

**Follow platform conventions and be consistent.**

```tsx
// ✅ Good - Consistent button styles
const buttonVariants = {
  primary: 'bg-blue-500 text-white hover:bg-blue-600',
  secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
  danger: 'bg-red-500 text-white hover:bg-red-600',
  ghost: 'hover:bg-gray-100'
} as const;

// Use consistently throughout app
<button className={buttonVariants.primary}>Save</button>
<button className={buttonVariants.danger}>Delete</button>

// ✅ Good - Consistent spacing scale
const spacing = {
  xs: '0.25rem',  // 4px
  sm: '0.5rem',   // 8px
  md: '1rem',     // 16px
  lg: '1.5rem',   // 24px
  xl: '2rem',     // 32px
  '2xl': '3rem'   // 48px
} as const;
```

#### 5. Error Prevention

**Prevent errors before they occur.**

```tsx
// ✅ Good - Disable invalid actions
function DeleteButton({ hasUnsavedChanges }: { hasUnsavedChanges: boolean }) {
  return (
    <Tooltip content={hasUnsavedChanges ? "Save changes before deleting" : ""}>
      <button
        disabled={hasUnsavedChanges}
        className={hasUnsavedChanges ? "opacity-50 cursor-not-allowed" : ""}
      >
        Delete
      </button>
    </Tooltip>
  );
}

// ✅ Good - Inline validation
function EmailInput() {
  const [email, setEmail] = useState("");
  const [error, setError] = useState("");

  const validateEmail = (value: string) => {
    if (!value.includes("@")) {
      setError("Please enter a valid email address");
    } else {
      setError("");
    }
  };

  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={(e) => {
          setEmail(e.target.value);
          validateEmail(e.target.value);
        }}
        onBlur={() => validateEmail(email)}
        aria-invalid={!!error}
        aria-describedby={error ? "email-error" : undefined}
      />
      {error && (
        <p id="email-error" className="text-red-500 text-sm">
          {error}
        </p>
      )}
    </div>
  );
}

// ✅ Good - Confirmation for destructive actions
function DeleteButton({ onDelete }: { onDelete: () => void }) {
  const [showConfirm, setShowConfirm] = useState(false);

  return (
    <>
      <button onClick={() => setShowConfirm(true)}>Delete</button>
      <AlertDialog open={showConfirm} onOpenChange={setShowConfirm}>
        <AlertDialogContent>
          <AlertDialogHeader>
            <AlertDialogTitle>Are you sure?</AlertDialogTitle>
            <AlertDialogDescription>
              This action cannot be undone. This will permanently delete the
              item.
            </AlertDialogDescription>
          </AlertDialogHeader>
          <AlertDialogFooter>
            <AlertDialogCancel>Cancel</AlertDialogCancel>
            <AlertDialogAction onClick={onDelete} className="bg-red-500">
              Delete
            </AlertDialogAction>
          </AlertDialogFooter>
        </AlertDialogContent>
      </AlertDialog>
    </>
  );
}
```

#### 6. Recognition Rather Than Recall

**Make options visible; don't rely on memory.**

```tsx
// ❌ Bad - User must remember syntax
<input placeholder="Enter date (YYYY-MM-DD)" />

// ✅ Good - Date picker with visual selection
<DatePicker
  selected={date}
  onChange={setDate}
  dateFormat="MMMM d, yyyy"
  showMonthDropdown
  showYearDropdown
/>

// ✅ Good - Autocomplete for common inputs
function CityInput() {
  const [suggestions, setSuggestions] = useState<string[]>([]);

  return (
    <Combobox value={city} onChange={setCity}>
      <ComboboxInput
        onChange={(e) => fetchCitySuggestions(e.target.value)}
      />
      <ComboboxOptions>
        {suggestions.map(city => (
          <ComboboxOption key={city} value={city}>
            {city}
          </ComboboxOption>
        ))}
      </ComboboxOptions>
    </Combobox>
  );
}

// ✅ Good - Recently used items
function FileSelector() {
  const recentFiles = getRecentFiles();

  return (
    <div>
      <h3>Recent Files</h3>
      <ul>
        {recentFiles.map(file => (
          <li key={file.id} onClick={() => selectFile(file)}>
            {file.name}
          </li>
        ))}
      </ul>
      <button>Browse All Files</button>
    </div>
  );
}
```

#### 7. Flexibility and Efficiency

**Provide shortcuts for power users.**

```tsx
// ✅ Good - Keyboard shortcuts
function SearchBar() {
  useEffect(() => {
    const handleKeyPress = (e: KeyboardEvent) => {
      // Cmd/Ctrl + K to focus search
      if ((e.metaKey || e.ctrlKey) && e.key === "k") {
        e.preventDefault();
        searchInputRef.current?.focus();
      }
    };

    document.addEventListener("keydown", handleKeyPress);
    return () => document.removeEventListener("keydown", handleKeyPress);
  }, []);

  return (
    <div className="relative">
      <input ref={searchInputRef} placeholder="Search..." />
      <kbd className="absolute right-2 top-2 text-xs text-gray-500">⌘K</kbd>
    </div>
  );
}

// ✅ Good - Bulk actions
function ItemList({ items }: { items: Item[] }) {
  const [selected, setSelected] = useState<Set<string>>(new Set());

  const selectAll = () => setSelected(new Set(items.map((i) => i.id)));
  const deselectAll = () => setSelected(new Set());

  return (
    <div>
      <div className="flex gap-2 mb-4">
        <button onClick={selectAll}>Select All</button>
        <button onClick={deselectAll}>Deselect All</button>
        {selected.size > 0 && (
          <>
            <button onClick={() => bulkDelete(Array.from(selected))}>
              Delete {selected.size} items
            </button>
            <button onClick={() => bulkExport(Array.from(selected))}>
              Export {selected.size} items
            </button>
          </>
        )}
      </div>
      {/* ... */}
    </div>
  );
}
```

#### 8. Aesthetic and Minimalist Design

**Remove unnecessary elements that compete with essential content.**

```tsx
// ❌ Bad - Cluttered interface
function ProductCard({ product }: { product: Product }) {
  return (
    <div className="border p-4 bg-gradient-to-r from-blue-100 to-purple-100">
      <div className="flex justify-between">
        <img
          src={product.image}
          className="w-20 h-20 border-4 border-yellow-300"
        />
        <div className="flex flex-col gap-1">
          <span className="text-xs text-gray-400">SKU: {product.sku}</span>
          <span className="text-xs text-gray-400">
            Added: {product.createdAt}
          </span>
          <span className="text-xs text-gray-400">
            Updated: {product.updatedAt}
          </span>
        </div>
      </div>
      <h3 className="text-lg font-bold text-blue-600 underline">
        {product.name}
      </h3>
      <p className="text-sm italic text-gray-600">{product.description}</p>
      <div className="flex gap-2 mt-2">
        <button className="bg-blue-500 text-white px-2 py-1 text-xs">
          View
        </button>
        <button className="bg-green-500 text-white px-2 py-1 text-xs">
          Edit
        </button>
        <button className="bg-yellow-500 text-white px-2 py-1 text-xs">
          Share
        </button>
        <button className="bg-purple-500 text-white px-2 py-1 text-xs">
          Favorite
        </button>
        <button className="bg-red-500 text-white px-2 py-1 text-xs">
          Delete
        </button>
      </div>
    </div>
  );
}

// ✅ Good - Clean, focused design
function ProductCard({ product }: { product: Product }) {
  return (
    <div className="border border-gray-200 rounded-lg p-4 hover:shadow-md transition">
      <img
        src={product.image}
        alt={product.name}
        className="w-full h-48 object-cover rounded-md mb-3"
      />
      <h3 className="text-lg font-semibold mb-2">{product.name}</h3>
      <p className="text-2xl font-bold mb-4">${product.price}</p>
      <button className="w-full bg-blue-500 text-white py-2 rounded hover:bg-blue-600">
        Add to Cart
      </button>
    </div>
  );
}
```

#### 9. Help Users Recognize, Diagnose, and Recover from Errors

**Error messages should be clear and helpful.**

```tsx
// ❌ Bad - Cryptic error
<div className="text-red-500">Error: 422</div>;

// ✅ Good - Helpful error message
function ErrorMessage({ error }: { error: Error }) {
  return (
    <div className="bg-red-50 border border-red-200 rounded-lg p-4">
      <div className="flex items-start gap-3">
        <AlertCircle className="text-red-500 flex-shrink-0" />
        <div className="flex-1">
          <h3 className="font-semibold text-red-900 mb-1">
            Unable to save changes
          </h3>
          <p className="text-sm text-red-700 mb-3">{error.message}</p>
          <div className="flex gap-2">
            <button className="text-sm text-red-600 underline">
              Try Again
            </button>
            <button className="text-sm text-red-600 underline">
              Contact Support
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}

// ✅ Good - Form validation with guidance
function PasswordInput() {
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState<string[]>([]);

  const validate = (value: string) => {
    const newErrors: string[] = [];
    if (value.length < 8) newErrors.push("At least 8 characters");
    if (!/[A-Z]/.test(value)) newErrors.push("One uppercase letter");
    if (!/[a-z]/.test(value)) newErrors.push("One lowercase letter");
    if (!/[0-9]/.test(value)) newErrors.push("One number");
    setErrors(newErrors);
  };

  return (
    <div>
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={(e) => {
          setPassword(e.target.value);
          validate(e.target.value);
        }}
        aria-invalid={errors.length > 0}
      />
      {errors.length > 0 && (
        <div className="mt-2 text-sm">
          <p className="text-gray-700 mb-1">Password must include:</p>
          <ul className="space-y-1">
            {errors.map((error) => (
              <li key={error} className="flex items-center gap-2 text-red-600">
                <X className="w-4 h-4" />
                {error}
              </li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
}
```

#### 10. Help and Documentation

**Provide contextual help when needed.**

```tsx
// ✅ Good - Tooltips for complex features
function AdvancedFilter() {
  return (
    <div className="flex items-center gap-2">
      <label>Date Range</label>
      <Tooltip content="Filter results by selecting a start and end date">
        <HelpCircle className="w-4 h-4 text-gray-400" />
      </Tooltip>
    </div>
  );
}

// ✅ Good - Empty states with guidance
function EmptyState() {
  return (
    <div className="text-center py-12">
      <Inbox className="w-16 h-16 text-gray-300 mx-auto mb-4" />
      <h3 className="text-lg font-semibold mb-2">No projects yet</h3>
      <p className="text-gray-600 mb-4">
        Get started by creating your first project
      </p>
      <button className="bg-blue-500 text-white px-4 py-2 rounded">
        Create Project
      </button>
    </div>
  );
}

// ✅ Good - Onboarding tour
function Dashboard() {
  const [showTour, setShowTour] = useState(isFirstTimeUser());

  return (
    <div>
      {showTour && (
        <Tour
          steps={[
            {
              target: "#projects",
              content: "View and manage your projects here",
            },
            { target: "#analytics", content: "Track your performance metrics" },
            { target: "#settings", content: "Customize your preferences" },
          ]}
          onComplete={() => setShowTour(false)}
        />
      )}
      {/* Dashboard content */}
    </div>
  );
}
```

---

## Visual Design Principles

### Typography

```tsx
// Define type scale
const typeScale = {
  xs: "0.75rem", // 12px
  sm: "0.875rem", // 14px
  base: "1rem", // 16px
  lg: "1.125rem", // 18px
  xl: "1.25rem", // 20px
  "2xl": "1.5rem", // 24px
  "3xl": "1.875rem", // 30px
  "4xl": "2.25rem", // 36px
  "5xl": "3rem", // 48px
} as const;

// Font weights
const fontWeights = {
  normal: 400,
  medium: 500,
  semibold: 600,
  bold: 700,
} as const;

// Line heights for readability
const lineHeights = {
  tight: 1.25,
  normal: 1.5,
  relaxed: 1.75,
} as const;

// ✅ Good - Typography hierarchy
function Article({ title, author, date, content }: ArticleProps) {
  return (
    <article className="max-w-2xl mx-auto">
      <h1 className="text-4xl font-bold leading-tight mb-2">{title}</h1>
      <p className="text-sm text-gray-600 mb-8">
        By {author} • {date}
      </p>
      <div className="prose prose-lg">
        {/* Body text: 18px, line-height 1.75 for readability */}
        <p className="text-lg leading-relaxed">{content}</p>
      </div>
    </article>
  );
}

// ✅ Good - Limit line length for readability
// Optimal: 50-75 characters per line
<div className="max-w-prose">
  {" "}
  {/* ~65ch */}
  <p>Long-form content here...</p>
</div>;
```

### Color System

```tsx
// Define semantic color palette
const colors = {
  // Brand colors
  primary: {
    50: "#eff6ff",
    100: "#dbeafe",
    500: "#3b82f6",
    600: "#2563eb",
    700: "#1d4ed8",
    900: "#1e3a8a",
  },

  // Semantic colors
  success: {
    50: "#f0fdf4",
    500: "#22c55e",
    700: "#15803d",
  },
  error: {
    50: "#fef2f2",
    500: "#ef4444",
    700: "#b91c1c",
  },
  warning: {
    50: "#fffbeb",
    500: "#f59e0b",
    700: "#b45309",
  },

  // Neutral colors
  gray: {
    50: "#f9fafb",
    100: "#f3f4f6",
    200: "#e5e7eb",
    300: "#d1d5db",
    400: "#9ca3af",
    500: "#6b7280",
    600: "#4b5563",
    700: "#374151",
    800: "#1f2937",
    900: "#111827",
  },
} as const;

// ✅ Good - Accessible color contrast
// WCAG AA: 4.5:1 for normal text, 3:1 for large text
function Button({ variant = "primary" }: ButtonProps) {
  const variants = {
    primary: "bg-blue-600 text-white", // High contrast
    secondary: "bg-gray-200 text-gray-900",
    danger: "bg-red-600 text-white",
  };

  return (
    <button className={cn("px-4 py-2 rounded font-medium", variants[variant])}>
      {children}
    </button>
  );
}

// ✅ Good - Color for meaning, not only
function Status({ status }: { status: "success" | "error" | "warning" }) {
  const configs = {
    success: {
      icon: CheckCircle,
      color: "text-green-700",
      bg: "bg-green-50",
      label: "Completed",
    },
    error: {
      icon: XCircle,
      color: "text-red-700",
      bg: "bg-red-50",
      label: "Failed",
    },
    warning: {
      icon: AlertCircle,
      color: "text-yellow-700",
      bg: "bg-yellow-50",
      label: "Warning",
    },
  };

  const config = configs[status];
  const Icon = config.icon;

  return (
    <div
      className={cn(
        "inline-flex items-center gap-2 px-3 py-1 rounded",
        config.bg,
      )}
    >
      <Icon className={cn("w-4 h-4", config.color)} />
      <span className={cn("text-sm font-medium", config.color)}>
        {config.label}
      </span>
    </div>
  );
}
```

### Spacing & Layout

```tsx
// Consistent spacing scale (4px base unit)
const spacing = {
  0: "0",
  1: "0.25rem", // 4px
  2: "0.5rem", // 8px
  3: "0.75rem", // 12px
  4: "1rem", // 16px
  5: "1.25rem", // 20px
  6: "1.5rem", // 24px
  8: "2rem", // 32px
  10: "2.5rem", // 40px
  12: "3rem", // 48px
  16: "4rem", // 64px
  20: "5rem", // 80px
  24: "6rem", // 96px
} as const;

// ✅ Good - Consistent spacing
function Card({ title, description, children }: CardProps) {
  return (
    <div className="border border-gray-200 rounded-lg p-6">
      {/* 24px padding */}
      <h2 className="text-xl font-semibold mb-2">
        {/* 8px margin-bottom */}
        {title}
      </h2>
      <p className="text-gray-600 mb-4">
        {/* 16px margin-bottom */}
        {description}
      </p>
      <div className="space-y-4">
        {/* 16px gap between items */}
        {children}
      </div>
    </div>
  );
}

// ✅ Good - Visual hierarchy through spacing
function Article() {
  return (
    <article className="space-y-8">
      {/* Large spacing between sections */}
      <section className="space-y-4">
        {/* Medium spacing within section */}
        <h2 className="text-2xl font-bold">Section Title</h2>
        <p className="space-y-2">
          {/* Small spacing between paragraphs */}
          Paragraph text...
        </p>
      </section>
    </article>
  );
}
```

### Visual Hierarchy

```tsx
// ✅ Good - Clear visual hierarchy
function Dashboard() {
  return (
    <div className="p-8">
      {/* Level 1: Page title - Largest, boldest */}
      <h1 className="text-4xl font-bold mb-2">Dashboard</h1>

      {/* Level 2: Page description - Medium, lighter */}
      <p className="text-lg text-gray-600 mb-8">
        Welcome back! Here's what's happening today.
      </p>

      {/* Level 3: Section headers - Medium size, bold */}
      <h2 className="text-2xl font-semibold mb-4">Recent Activity</h2>

      {/* Level 4: Cards - Contained, elevated */}
      <div className="grid grid-cols-3 gap-6">
        <Card>
          {/* Level 5: Card title - Small, medium weight */}
          <h3 className="text-lg font-medium mb-2">Sales</h3>

          {/* Level 6: Primary data - Large, bold */}
          <p className="text-3xl font-bold">$12,345</p>

          {/* Level 7: Secondary data - Small, light */}
          <p className="text-sm text-gray-500">+12% from last month</p>
        </Card>
      </div>
    </div>
  );
}
```

---

## Component Design Patterns

### Atomic Design

```tsx
// Atoms - Basic building blocks
function Button({ children, variant = "primary", ...props }: ButtonProps) {
  return (
    <button className={cn(buttonVariants[variant])} {...props}>
      {children}
    </button>
  );
}

function Input({ label, error, ...props }: InputProps) {
  return (
    <div className="space-y-1">
      {label && <label className="text-sm font-medium">{label}</label>}
      <input
        className={cn("border rounded px-3 py-2", error && "border-red-500")}
        {...props}
      />
      {error && <p className="text-sm text-red-600">{error}</p>}
    </div>
  );
}

// Molecules - Simple combinations of atoms
function SearchInput({ onSearch }: { onSearch: (query: string) => void }) {
  const [query, setQuery] = useState("");

  return (
    <div className="flex gap-2">
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <Button onClick={() => onSearch(query)}>
        <Search className="w-4 h-4" />
      </Button>
    </div>
  );
}

// Organisms - Complex UI components
function UserCard({ user }: { user: User }) {
  return (
    <div className="border rounded-lg p-6 space-y-4">
      <div className="flex items-center gap-4">
        <Avatar src={user.avatar} alt={user.name} />
        <div>
          <h3 className="font-semibold">{user.name}</h3>
          <p className="text-sm text-gray-600">{user.email}</p>
        </div>
      </div>
      <div className="flex gap-2">
        <Button variant="primary">Message</Button>
        <Button variant="secondary">View Profile</Button>
      </div>
    </div>
  );
}

// Templates - Page layouts
function DashboardTemplate({ sidebar, header, main, footer }: LayoutProps) {
  return (
    <div className="min-h-screen flex flex-col">
      <header className="border-b">{header}</header>
      <div className="flex-1 flex">
        <aside className="w-64 border-r">{sidebar}</aside>
        <main className="flex-1 p-6">{main}</main>
      </div>
      <footer className="border-t">{footer}</footer>
    </div>
  );
}
```

### Composition over Configuration

```tsx
// ❌ Bad - Too many props, hard to extend
function Card({
  title,
  subtitle,
  image,
  imagePosition,
  showBorder,
  borderColor,
  backgroundColor,
  padding,
  actions,
  footer,
  ...props
}: CardProps) {
  // Complex configuration logic...
}

// ✅ Good - Flexible composition
function Card({ children, className, ...props }: CardProps) {
  return (
    <div className={cn("border rounded-lg bg-white", className)} {...props}>
      {children}
    </div>
  );
}

// Sub-components for structure
Card.Header = function CardHeader({ children, className }: HeaderProps) {
  return <div className={cn("p-6 border-b", className)}>{children}</div>;
};

Card.Body = function CardBody({ children, className }: BodyProps) {
  return <div className={cn("p-6", className)}>{children}</div>;
};

Card.Footer = function CardFooter({ children, className }: FooterProps) {
  return (
    <div className={cn("p-6 border-t bg-gray-50", className)}>{children}</div>
  );
};

// Usage - Flexible and readable
function ProductCard({ product }: { product: Product }) {
  return (
    <Card>
      <Card.Header>
        <h3 className="text-xl font-semibold">{product.name}</h3>
        <p className="text-gray-600">{product.category}</p>
      </Card.Header>
      <Card.Body>
        <img src={product.image} alt={product.name} />
        <p>{product.description}</p>
      </Card.Body>
      <Card.Footer>
        <Button>Add to Cart</Button>
      </Card.Footer>
    </Card>
  );
}
```

### Compound Components

```tsx
// ✅ Good - Tabs with compound pattern
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (tab: string) => void;
} | null>(null);

function Tabs({ defaultTab, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div>{children}</div>
    </TabsContext.Provider>
  );
}

Tabs.List = function TabsList({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex border-b" role="tablist">
      {children}
    </div>
  );
};

Tabs.Tab = function Tab({ value, children }: TabProps) {
  const context = useContext(TabsContext);
  const isActive = context?.activeTab === value;

  return (
    <button
      role="tab"
      aria-selected={isActive}
      className={cn(
        "px-4 py-2 font-medium transition",
        isActive
          ? "border-b-2 border-blue-500 text-blue-600"
          : "text-gray-600 hover:text-gray-900",
      )}
      onClick={() => context?.setActiveTab(value)}
    >
      {children}
    </button>
  );
};

Tabs.Panel = function TabPanel({ value, children }: PanelProps) {
  const context = useContext(TabsContext);
  if (context?.activeTab !== value) return null;

  return (
    <div role="tabpanel" className="p-4">
      {children}
    </div>
  );
};

// Usage
function Settings() {
  return (
    <Tabs defaultTab="profile">
      <Tabs.List>
        <Tabs.Tab value="profile">Profile</Tabs.Tab>
        <Tabs.Tab value="security">Security</Tabs.Tab>
        <Tabs.Tab value="notifications">Notifications</Tabs.Tab>
      </Tabs.List>

      <Tabs.Panel value="profile">
        <ProfileSettings />
      </Tabs.Panel>
      <Tabs.Panel value="security">
        <SecuritySettings />
      </Tabs.Panel>
      <Tabs.Panel value="notifications">
        <NotificationSettings />
      </Tabs.Panel>
    </Tabs>
  );
}
```

---

## Responsive Design

### Mobile-First Approach

```tsx
// ✅ Good - Mobile-first responsive design
function Hero() {
  return (
    <section
      className="
      px-4 py-8
      sm:px-6 sm:py-12
      md:px-8 md:py-16
      lg:px-12 lg:py-24
    "
    >
      <h1
        className="
        text-3xl
        sm:text-4xl
        md:text-5xl
        lg:text-6xl
        font-bold
      "
      >
        Welcome
      </h1>

      {/* Stack on mobile, side-by-side on tablet+ */}
      <div
        className="
        flex flex-col
        md:flex-row
        gap-4
        md:gap-8
      "
      >
        <div className="flex-1">
          <p>Column 1</p>
        </div>
        <div className="flex-1">
          <p>Column 2</p>
        </div>
      </div>
    </section>
  );
}

// ✅ Good - Responsive grid
function ProductGrid({ products }: { products: Product[] }) {
  return (
    <div
      className="
      grid
      grid-cols-1
      sm:grid-cols-2
      md:grid-cols-3
      lg:grid-cols-4
      gap-4
      md:gap-6
    "
    >
      {products.map((product) => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### Touch-Friendly Design

```tsx
// ✅ Good - Large touch targets (min 44x44px)
function MobileNav() {
  return (
    <nav className="flex justify-around p-2">
      <button
        className="
        flex flex-col items-center gap-1
        px-4 py-3
        min-h-[44px]
        touch-manipulation
      "
      >
        <Home className="w-6 h-6" />
        <span className="text-xs">Home</span>
      </button>
      {/* ... more nav items */}
    </nav>
  );
}

// ✅ Good - Swipe gestures for mobile
function ImageCarousel({ images }: { images: string[] }) {
  const [currentIndex, setCurrentIndex] = useState(0);
  const [touchStart, setTouchStart] = useState(0);
  const [touchEnd, setTouchEnd] = useState(0);

  const minSwipeDistance = 50;

  const onTouchStart = (e: React.TouchEvent) => {
    setTouchEnd(0);
    setTouchStart(e.targetTouches[0].clientX);
  };

  const onTouchMove = (e: React.TouchEvent) => {
    setTouchEnd(e.targetTouches[0].clientX);
  };

  const onTouchEnd = () => {
    if (!touchStart || !touchEnd) return;

    const distance = touchStart - touchEnd;
    const isLeftSwipe = distance > minSwipeDistance;
    const isRightSwipe = distance < -minSwipeDistance;

    if (isLeftSwipe) {
      setCurrentIndex((prev) => Math.min(prev + 1, images.length - 1));
    }
    if (isRightSwipe) {
      setCurrentIndex((prev) => Math.max(prev - 1, 0));
    }
  };

  return (
    <div
      onTouchStart={onTouchStart}
      onTouchMove={onTouchMove}
      onTouchEnd={onTouchEnd}
      className="relative overflow-hidden"
    >
      <img src={images[currentIndex]} alt="" className="w-full" />
    </div>
  );
}
```

### Container Queries (Modern)

```tsx
// ✅ Good - Component-level responsive design
function ProductCard({ product }: { product: Product }) {
  return (
    <div className="@container">
      {/* Changes based on container width, not viewport */}
      <div
        className="
        flex flex-col
        @md:flex-row
        gap-4
      "
      >
        <img
          src={product.image}
          className="
            w-full
            @md:w-32
            @md:h-32
            object-cover
          "
        />
        <div className="flex-1">
          <h3 className="font-semibold">{product.name}</h3>
          <p className="text-sm text-gray-600">{product.description}</p>
        </div>
      </div>
    </div>
  );
}
```

---

## Accessibility (A11y)

### Semantic HTML

```tsx
// ❌ Bad - Divs for everything
<div onClick={handleClick}>Click me</div>
<div className="nav">
  <div className="nav-item">Home</div>
</div>

// ✅ Good - Semantic HTML
<button onClick={handleClick}>Click me</button>
<nav>
  <a href="/">Home</a>
</nav>

// ✅ Good - Proper heading hierarchy
function Article() {
  return (
    <article>
      <h1>Main Article Title</h1>
      <section>
        <h2>Section Title</h2>
        <h3>Subsection Title</h3>
      </section>
    </article>
  );
}
```

### ARIA Labels and Roles

```tsx
// ✅ Good - Accessible icon buttons
function IconButton({ icon: Icon, label, onClick }: IconButtonProps) {
  return (
    <button
      onClick={onClick}
      aria-label={label}
      className="p-2 hover:bg-gray-100 rounded"
    >
      <Icon className="w-5 h-5" />
    </button>
  );
}

// Usage
<IconButton icon={Trash} label="Delete item" onClick={handleDelete} />;

// ✅ Good - ARIA states
function ExpandableSection({ title, children }: SectionProps) {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <div>
      <button
        onClick={() => setIsExpanded(!isExpanded)}
        aria-expanded={isExpanded}
        aria-controls="section-content"
      >
        {title}
        {isExpanded ? <ChevronUp /> : <ChevronDown />}
      </button>
      <div id="section-content" hidden={!isExpanded} aria-hidden={!isExpanded}>
        {children}
      </div>
    </div>
  );
}

// ✅ Good - Live regions for dynamic content
function NotificationCenter() {
  return (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    >
      {notification && `New notification: ${notification.message}`}
    </div>
  );
}
```

### Keyboard Navigation

```tsx
// ✅ Good - Full keyboard support
function Dropdown({ items }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);
  const itemRefs = useRef<(HTMLElement | null)[]>([]);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case "Escape":
        setIsOpen(false);
        break;
      case "ArrowDown":
        e.preventDefault();
        setActiveIndex((prev) => (prev < items.length - 1 ? prev + 1 : 0));
        break;
      case "ArrowUp":
        e.preventDefault();
        setActiveIndex((prev) => (prev > 0 ? prev - 1 : items.length - 1));
        break;
      case "Enter":
      case " ":
        if (activeIndex >= 0) {
          items[activeIndex].onClick();
          setIsOpen(false);
        }
        break;
    }
  };

  useEffect(() => {
    if (activeIndex >= 0 && itemRefs.current[activeIndex]) {
      itemRefs.current[activeIndex]?.focus();
    }
  }, [activeIndex]);

  return (
    <div onKeyDown={handleKeyDown}>
      <button
        onClick={() => setIsOpen(!isOpen)}
        aria-haspopup="true"
        aria-expanded={isOpen}
      >
        Menu
      </button>
      {isOpen && (
        <ul role="menu">
          {items.map((item, index) => (
            <li
              key={item.id}
              role="menuitem"
              ref={(el) => (itemRefs.current[index] = el)}
              tabIndex={index === activeIndex ? 0 : -1}
              onClick={item.onClick}
            >
              {item.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

### Focus Management

```tsx
// ✅ Good - Focus trap in modal
function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousActiveElement = useRef<HTMLElement | null>(null);

  useEffect(() => {
    if (isOpen) {
      // Store previously focused element
      previousActiveElement.current = document.activeElement as HTMLElement;

      // Focus modal
      modalRef.current?.focus();

      // Trap focus within modal
      const focusableElements = modalRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])',
      );

      // Handle Tab key
      const handleTab = (e: KeyboardEvent) => {
        if (e.key !== "Tab" || !focusableElements) return;

        const first = focusableElements[0] as HTMLElement;
        const last = focusableElements[
          focusableElements.length - 1
        ] as HTMLElement;

        if (e.shiftKey && document.activeElement === first) {
          e.preventDefault();
          last.focus();
        } else if (!e.shiftKey && document.activeElement === last) {
          e.preventDefault();
          first.focus();
        }
      };

      document.addEventListener("keydown", handleTab);

      return () => {
        document.removeEventListener("keydown", handleTab);
        // Restore focus when modal closes
        previousActiveElement.current?.focus();
      };
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
      className="fixed inset-0 z-50 flex items-center justify-center"
    >
      <div className="bg-white rounded-lg p-6 max-w-md">{children}</div>
    </div>
  );
}

// ✅ Good - Skip to main content
function Layout({ children }: LayoutProps) {
  return (
    <>
      <a
        href="#main-content"
        className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 bg-blue-500 text-white px-4 py-2 rounded"
      >
        Skip to main content
      </a>
      <nav>Navigation</nav>
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </>
  );
}
```

### Screen Reader Support

```tsx
// ✅ Good - Descriptive labels
function SearchInput() {
  return (
    <div>
      <label htmlFor="search" className="sr-only">
        Search products
      </label>
      <input
        id="search"
        type="search"
        placeholder="Search..."
        aria-describedby="search-hint"
      />
      <p id="search-hint" className="text-sm text-gray-600">
        Enter at least 3 characters to search
      </p>
    </div>
  );
}

// ✅ Good - Status updates
function FormSubmit() {
  const [status, setStatus] = useState<
    "idle" | "submitting" | "success" | "error"
  >("idle");
  const [message, setMessage] = useState("");

  return (
    <form onSubmit={handleSubmit}>
      {/* ... form fields ... */}

      <button type="submit" disabled={status === "submitting"}>
        {status === "submitting" ? "Saving..." : "Save"}
      </button>

      {/* Screen reader announcement */}
      <div role="status" aria-live="polite" className="sr-only">
        {status === "submitting" && "Saving your changes..."}
        {status === "success" && "Changes saved successfully"}
        {status === "error" && `Error: ${message}`}
      </div>

      {/* Visual feedback */}
      {status === "success" && (
        <div className="text-green-600 mt-2">✓ Changes saved successfully</div>
      )}
    </form>
  );
}
```

---

## Performance Optimization

### Code Splitting & Lazy Loading

```tsx
// ✅ Good - Route-based code splitting
import { lazy, Suspense } from "react";

const Dashboard = lazy(() => import("./pages/Dashboard"));
const Settings = lazy(() => import("./pages/Settings"));
const Profile = lazy(() => import("./pages/Profile"));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  );
}

// ✅ Good - Component-based lazy loading
const HeavyChart = lazy(() => import("./components/HeavyChart"));

function Analytics() {
  const [showChart, setShowChart] = useState(false);

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Chart</button>
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

### Image Optimization

```tsx
// ✅ Good - Responsive images with Next.js
import Image from "next/image";

function ProductImage({ src, alt }: ImageProps) {
  return (
    <Image
      src={src}
      alt={alt}
      width={800}
      height={600}
      sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      loading="lazy"
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,..."
    />
  );
}

// ✅ Good - Native lazy loading
function Gallery({ images }: { images: string[] }) {
  return (
    <div className="grid grid-cols-3 gap-4">
      {images.map((src, index) => (
        <img
          key={src}
          src={src}
          alt={`Gallery image ${index + 1}`}
          loading="lazy"
          decoding="async"
          className="w-full h-auto"
        />
      ))}
    </div>
  );
}

// ✅ Good - WebP with fallback
<picture>
  <source srcSet="image.webp" type="image/webp" />
  <source srcSet="image.jpg" type="image/jpeg" />
  <img src="image.jpg" alt="Description" loading="lazy" />
</picture>;
```

### Virtualization for Long Lists

```tsx
// ✅ Good - Virtual scrolling for performance
import { useVirtualizer } from "@tanstack/react-virtual";

function VirtualizedList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
    overscan: 5,
  });

  return (
    <div ref={parentRef} className="h-[600px] overflow-auto">
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: "relative",
        }}
      >
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: "absolute",
              top: 0,
              left: 0,
              width: "100%",
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            <ItemCard item={items[virtualRow.index]} />
          </div>
        ))}
      </div>
    </div>
  );
}
```

### Optimize Rerenders

```tsx
// ✅ Good - Memoization
import { memo, useMemo, useCallback } from "react";

const ExpensiveComponent = memo(function ExpensiveComponent({ data }: Props) {
  // Only rerenders if data changes
  return <div>{/* Expensive rendering */}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState<Item[]>([]);

  // Memoize expensive calculations
  const processedItems = useMemo(() => {
    return items.map((item) => expensiveTransform(item));
  }, [items]);

  // Memoize callbacks
  const handleItemClick = useCallback((id: string) => {
    console.log("Clicked:", id);
  }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveComponent data={processedItems} onClick={handleItemClick} />
    </div>
  );
}
```

### Debounce and Throttle

```tsx
// ✅ Good - Debounced search
import { useDebouncedCallback } from 'use-debounce';

function SearchBar() {
  const [query, setQuery] = useState('');

  const debouncedSearch = useDebouncedCallback(
    (value: string) => {
      // API call here
      fetchResults(value);
    },
    300 // Wait 300ms after user stops typing
  );

  return (
    <input
      value={query}
      onChange={(e) => {
        setQuery(e.target.value);
        debouncedSearch(e.target.value);
      }}
      placeholder="Search..."
    />
  );
}

// ✅ Good - Throttled scroll handler
import { useThrottledCallback } from 'use-debounce';

function ScrollProgress() {
  const throttledScroll = useThrottledCallback(
    () => {
      const progress = (window.scrollY / document.body.scrollHeight) * 100;
      setScrollProgress(progress);
    },
    100 // Update at most every 100ms
  );

  useEffect(() => {
    window.addEventListener('scroll', throttledScroll);
    return () => window.removeEventListener('scroll', throttledScroll);
  }, []);

  return (/* Progress bar */);
}
```

### Core Web Vitals

```tsx
// Monitor performance metrics
import { useReportWebVitals } from 'next/web-vitals';

export function reportWebVitals(metric: NextWebVitalsMetric) {
  switch (metric.name) {
    case 'FCP':
      // First Contentful Paint
      console.log('FCP:', metric.value);
      break;
    case 'LCP':
      // Largest Contentful Paint (should be < 2.5s)
      console.log('LCP:', metric.value);
      break;
    case 'CLS':
      // Cumulative Layout Shift (should be < 0.1)
      console.log('CLS:', metric.value);
      break;
    case 'FID':
      // First Input Delay (should be < 100ms)
      console.log('FID:', metric.value);
      break;
    case 'TTFB':
      // Time to First Byte
      console.log('TTFB:', metric.value);
      break;
  }
}

// Optimize for metrics
// ✅ LCP - Optimize largest image/element
<Image
  src="/hero.jpg"
  priority // Preload important images
  alt="Hero"
/>

// ✅ CLS - Reserve space for dynamic content
<div className="h-96"> {/* Reserve height */}
  <Suspense fallback={<Skeleton className="h-full" />}>
    <DynamicContent />
  </Suspense>
</div>

// ✅ FID - Reduce JavaScript execution time
// Use code splitting, lazy loading, web workers
```

---

## Forms & Validation UX

### Progressive Enhancement

```tsx
// ✅ Good - Client-side validation with server fallback
function SignupForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});

  const validateEmail = (email: string) => {
    if (!email) return "Email is required";
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      return "Please enter a valid email";
    }
    return "";
  };

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();

    const formData = new FormData(e.currentTarget);
    const email = formData.get("email") as string;

    // Client-side validation
    const emailError = validateEmail(email);
    if (emailError) {
      setErrors({ email: emailError });
      return;
    }

    try {
      // Server handles final validation
      const response = await fetch("/api/signup", {
        method: "POST",
        body: formData,
      });

      if (!response.ok) {
        const data = await response.json();
        setErrors(data.errors || {});
      }
    } catch (error) {
      setErrors({ form: "Something went wrong. Please try again." });
    }
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? "email-error" : undefined}
        />
        {errors.email && (
          <p id="email-error" className="text-red-600 text-sm mt-1">
            {errors.email}
          </p>
        )}
      </div>
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

### Real-time Feedback

```tsx
// ✅ Good - Password strength indicator
function PasswordInput() {
  const [password, setPassword] = useState("");
  const [strength, setStrength] = useState<"weak" | "medium" | "strong">(
    "weak",
  );

  const calculateStrength = (value: string): typeof strength => {
    let score = 0;
    if (value.length >= 8) score++;
    if (/[a-z]/.test(value)) score++;
    if (/[A-Z]/.test(value)) score++;
    if (/[0-9]/.test(value)) score++;
    if (/[^a-zA-Z0-9]/.test(value)) score++;

    if (score <= 2) return "weak";
    if (score <= 3) return "medium";
    return "strong";
  };

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setPassword(value);
    setStrength(calculateStrength(value));
  };

  const strengthConfig = {
    weak: { color: "bg-red-500", label: "Weak", width: "33%" },
    medium: { color: "bg-yellow-500", label: "Medium", width: "66%" },
    strong: { color: "bg-green-500", label: "Strong", width: "100%" },
  };

  const config = strengthConfig[strength];

  return (
    <div>
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={handleChange}
      />
      {password && (
        <div className="mt-2">
          <div className="flex justify-between text-sm mb-1">
            <span>Password strength:</span>
            <span
              className={cn(
                "font-medium",
                strength === "weak" && "text-red-600",
                strength === "medium" && "text-yellow-600",
                strength === "strong" && "text-green-600",
              )}
            >
              {config.label}
            </span>
          </div>
          <div className="w-full bg-gray-200 rounded-full h-2">
            <div
              className={cn("h-2 rounded-full transition-all", config.color)}
              style={{ width: config.width }}
            />
          </div>
        </div>
      )}
    </div>
  );
}
```

### Multi-step Forms

```tsx
// ✅ Good - Multi-step with progress
function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState({});
  const totalSteps = 3;

  const updateFormData = (data: Partial<typeof formData>) => {
    setFormData((prev) => ({ ...prev, ...data }));
  };

  const nextStep = () =>
    setCurrentStep((prev) => Math.min(prev + 1, totalSteps));
  const prevStep = () => setCurrentStep((prev) => Math.max(prev - 1, 1));

  return (
    <div className="max-w-2xl mx-auto">
      {/* Progress indicator */}
      <div className="mb-8">
        <div className="flex justify-between mb-2">
          {Array.from({ length: totalSteps }, (_, i) => i + 1).map((step) => (
            <div
              key={step}
              className={cn(
                "flex items-center justify-center w-10 h-10 rounded-full",
                step <= currentStep
                  ? "bg-blue-500 text-white"
                  : "bg-gray-200 text-gray-600",
              )}
            >
              {step < currentStep ? <Check className="w-5 h-5" /> : step}
            </div>
          ))}
        </div>
        <div className="w-full bg-gray-200 rounded-full h-2">
          <div
            className="bg-blue-500 h-2 rounded-full transition-all"
            style={{ width: `${(currentStep / totalSteps) * 100}%` }}
          />
        </div>
      </div>

      {/* Step content */}
      <form>
        {currentStep === 1 && (
          <Step1 data={formData} onUpdate={updateFormData} />
        )}
        {currentStep === 2 && (
          <Step2 data={formData} onUpdate={updateFormData} />
        )}
        {currentStep === 3 && (
          <Step3 data={formData} onUpdate={updateFormData} />
        )}

        {/* Navigation */}
        <div className="flex justify-between mt-8">
          <button
            type="button"
            onClick={prevStep}
            disabled={currentStep === 1}
            className={cn(
              "px-4 py-2 rounded",
              currentStep === 1 && "opacity-50 cursor-not-allowed",
            )}
          >
            Previous
          </button>
          {currentStep < totalSteps ? (
            <button
              type="button"
              onClick={nextStep}
              className="bg-blue-500 text-white px-4 py-2 rounded"
            >
              Next
            </button>
          ) : (
            <button
              type="submit"
              className="bg-green-500 text-white px-4 py-2 rounded"
            >
              Submit
            </button>
          )}
        </div>
      </form>
    </div>
  );
}
```

---

## Loading States & Skeleton Screens

```tsx
// ✅ Good - Skeleton screens match actual content
function ProductCardSkeleton() {
  return (
    <div className="border rounded-lg p-4 animate-pulse">
      <div className="bg-gray-200 h-48 rounded mb-4" />
      <div className="bg-gray-200 h-6 rounded mb-2 w-3/4" />
      <div className="bg-gray-200 h-4 rounded mb-4 w-1/2" />
      <div className="bg-gray-200 h-10 rounded w-full" />
    </div>
  );
}

function ProductCard({ product }: { product: Product }) {
  return (
    <div className="border rounded-lg p-4">
      <img src={product.image} className="h-48 object-cover rounded mb-4" />
      <h3 className="text-lg font-semibold mb-2">{product.name}</h3>
      <p className="text-gray-600 mb-4">${product.price}</p>
      <button className="w-full bg-blue-500 text-white py-2 rounded">
        Add to Cart
      </button>
    </div>
  );
}

// Usage
function ProductGrid() {
  const { data: products, isLoading } = useQuery("products", fetchProducts);

  return (
    <div className="grid grid-cols-3 gap-6">
      {isLoading
        ? Array.from({ length: 6 }).map((_, i) => (
            <ProductCardSkeleton key={i} />
          ))
        : products.map((product) => (
            <ProductCard key={product.id} product={product} />
          ))}
    </div>
  );
}

// ✅ Good - Progressive loading
function Dashboard() {
  const { data: stats, isLoading: statsLoading } = useQuery(
    "stats",
    fetchStats,
  );
  const { data: charts, isLoading: chartsLoading } = useQuery(
    "charts",
    fetchCharts,
  );

  return (
    <div>
      {/* Show stats immediately when ready */}
      {statsLoading ? <StatsSkeleton /> : <Stats data={stats} />}

      {/* Charts load independently */}
      {chartsLoading ? <ChartSkeleton /> : <Charts data={charts} />}
    </div>
  );
}
```

---

## Micro-interactions & Animations

```tsx
// ✅ Good - Meaningful animations
function Button({ children, isLoading, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        "px-4 py-2 rounded transition-all",
        "hover:scale-105 active:scale-95",
        "disabled:opacity-50 disabled:cursor-not-allowed",
      )}
      disabled={isLoading}
      {...props}
    >
      <span className={cn("transition-opacity", isLoading && "opacity-0")}>
        {children}
      </span>
      {isLoading && <Spinner className="absolute inset-0 m-auto" />}
    </button>
  );
}

// ✅ Good - Page transitions
import { motion, AnimatePresence } from "framer-motion";

function PageTransition({ children }: { children: React.ReactNode }) {
  return (
    <AnimatePresence mode="wait">
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.2 }}
      >
        {children}
      </motion.div>
    </AnimatePresence>
  );
}

// ✅ Good - Toast notifications
function Toast({ message, type, onClose }: ToastProps) {
  useEffect(() => {
    const timer = setTimeout(onClose, 5000);
    return () => clearTimeout(timer);
  }, [onClose]);

  return (
    <motion.div
      initial={{ opacity: 0, y: 50, scale: 0.3 }}
      animate={{ opacity: 1, y: 0, scale: 1 }}
      exit={{ opacity: 0, scale: 0.5 }}
      className={cn(
        "fixed bottom-4 right-4 p-4 rounded-lg shadow-lg",
        type === "success" && "bg-green-500 text-white",
        type === "error" && "bg-red-500 text-white",
      )}
    >
      {message}
    </motion.div>
  );
}

// ⚠️ Good - Respect reduced motion preference
function AnimatedComponent() {
  const prefersReducedMotion = window.matchMedia(
    "(prefers-reduced-motion: reduce)",
  ).matches;

  return (
    <motion.div
      animate={{ x: 100 }}
      transition={{
        duration: prefersReducedMotion ? 0 : 0.5,
      }}
    >
      Content
    </motion.div>
  );
}
```

---

## Anti-Patterns to Avoid

### Poor Contrast

```tsx
// ❌ Bad - Poor color contrast
<button className="bg-gray-300 text-gray-400">Click me</button>

// ✅ Good - Sufficient contrast (WCAG AA: 4.5:1)
<button className="bg-blue-600 text-white">Click me</button>
```

### Unclear Interactive Elements

```tsx
// ❌ Bad - Looks like text, acts like button
<span onClick={handleClick} className="text-blue-500">
  Delete
</span>

// ✅ Good - Clear affordances
<button
  onClick={handleClick}
  className="text-blue-500 underline hover:text-blue-700"
>
  Delete
</button>
```

### Disabled Buttons Without Explanation

```tsx
// ❌ Bad - User doesn't know why button is disabled
<button disabled>Submit</button>

// ✅ Good - Clear explanation
<Tooltip content="Please fill in all required fields">
  <button disabled={!isFormValid}>Submit</button>
</Tooltip>
```

### Modal Overuse

```tsx
// ❌ Bad - Modal for simple confirmations
<Modal>
  <p>Item deleted</p>
  <button onClick={closeModal}>OK</button>
</Modal>;

// ✅ Good - Use toast for non-critical feedback
toast.success("Item deleted");

// ✅ Good - Modal for critical actions that need attention
<Modal>
  <h2>Delete Account?</h2>
  <p>This action cannot be undone.</p>
  <button onClick={confirmDelete}>Delete</button>
  <button onClick={closeModal}>Cancel</button>
</Modal>;
```

### Overwhelming Forms

```tsx
// ❌ Bad - All fields at once
<form>
  <input placeholder="First name" />
  <input placeholder="Last name" />
  <input placeholder="Email" />
  <input placeholder="Phone" />
  <input placeholder="Address line 1" />
  <input placeholder="Address line 2" />
  <input placeholder="City" />
  <input placeholder="State" />
  <input placeholder="ZIP" />
  <input placeholder="Country" />
  <button>Submit</button>
</form>

// ✅ Good - Progressive disclosure
<form>
  <input placeholder="Email" />
  <button>Continue</button>
  {/* Show more fields after first step */}
</form>
```

---

## Quick Reference Checklist

**Usability:**

- [ ] Clear visual hierarchy
- [ ] Consistent design system
- [ ] Obvious interactive elements
- [ ] Immediate feedback for actions
- [ ] Error prevention and helpful error messages
- [ ] Easy undo/cancel options

**Accessibility:**

- [ ] Semantic HTML elements
- [ ] ARIA labels where needed
- [ ] Keyboard navigation support
- [ ] Focus indicators visible
- [ ] Color contrast meets WCAG AA (4.5:1)
- [ ] Screen reader compatible

**Performance:**

- [ ] Images optimized and lazy loaded
- [ ] Code splitting implemented
- [ ] Long lists virtualized
- [ ] Expensive operations memoized
- [ ] Core Web Vitals monitored

**Responsive:**

- [ ] Mobile-first design
- [ ] Touch targets min 44x44px
- [ ] Works on all viewport sizes
- [ ] No horizontal scrolling
- [ ] Legible text without zooming

**Forms:**

- [ ] Clear labels and placeholders
- [ ] Inline validation
- [ ] Helpful error messages
- [ ] Progress indication for multi-step
- [ ] Save/autosave functionality

**Visual Design:**

- [ ] Consistent typography scale
- [ ] Systematic color palette
- [ ] Adequate spacing
- [ ] Visual feedback for interactions
- [ ] Respect reduced motion preference

---

## Summary

**Key Principles:**

- Design for users, not yourself
- Keep it simple and consistent
- Provide clear feedback
- Prevent errors when possible
- Make it accessible to everyone
- Optimize for performance
- Test with real users

**Remember**: Great UI/UX is invisible. Users should accomplish their goals effortlessly without thinking about the interface.
