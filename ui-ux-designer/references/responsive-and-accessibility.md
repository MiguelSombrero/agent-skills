Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

