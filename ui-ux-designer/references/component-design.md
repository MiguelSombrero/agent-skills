Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

