Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

