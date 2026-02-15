Reference for ui-ux-designer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

