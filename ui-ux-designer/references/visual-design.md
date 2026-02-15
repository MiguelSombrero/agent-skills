Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

