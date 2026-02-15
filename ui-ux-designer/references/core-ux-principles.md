Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.

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

