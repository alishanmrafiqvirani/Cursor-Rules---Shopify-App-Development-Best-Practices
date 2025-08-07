---
description: Shopify Point of Sale UI Extension Performance Guidelines
globs:
  [
    "**/*.js",
    "**/*.ts",
    "**/*.jsx",
    "**/*.tsx",
    "**/*.swift",
    "**/*.kt",
    "**/*.java",
    "**/*.dart",
    "**/*.css",
    "**/*.scss",
    "**/*.json",
    "**/*.yml",
    "**/*.yaml",
  ]
alwaysApply: false
---

# Shopify Point of Sale UI Extension Performance Guidelines

## Testing & Compatibility Requirements

### 1. Multi-Device Testing Strategy

- **TEST** across various device types for maximum compatibility:
  - **iOS and Android tablets** of different screen sizes
  - **Mobile phones** ranging from high-end to entry-level devices
  - **Different hardware generations** including older devices when possible
- **ENSURE** consistent experience regardless of merchant hardware choices
- **VERIFY** performance on entry-level devices and previous-generation hardware
- **WHY**: Merchants use diverse hardware configurations in retail environments

### 2. Realistic Data Volume Testing

- **SET UP** test stores with realistic product catalogs
- **INCLUDE** substantial numbers of orders and inventory items
- **TEST** complex product configurations (variants, options, etc.)
- **SIMULATE** operational workflows targeting < 2 second response times
- **AVOID** testing only with small test stores that don't reflect production loads
- **WHY**: Extensions must perform well with real-world data volumes

### 3. Multi-Extension Compatibility Testing

- **INSTALL** other POS extensions alongside yours during testing
- **TEST** various combinations of extensions running simultaneously
- **VERIFY** your extension maintains performance when others are active
- **MONITOR** resource sharing and potential conflicts
- **ENSURE** graceful degradation when resources are constrained
- **WHY**: Merchants often use multiple extensions simultaneously

## Performance Monitoring & Optimization

### 4. Resource Usage Monitoring

#### iOS Testing with Xcode Instruments

- **USE** Xcode's Instruments to monitor memory usage and performance
- **MONITOR** Memory Graph and Leaks instruments regularly
- **TRACK** CPU usage during different operations
- **IDENTIFY** memory leaks and performance bottlenecks

#### Android Testing with Android Studio

- **USE** Android Studio's Profiler for memory, CPU, and network monitoring
- **CHECK** for memory leaks using Memory Profiler
- **MONITOR** performance during user interactions
- **ANALYZE** resource consumption patterns

#### Chrome DevTools Monitoring

```javascript
// Monitor memory usage in Chrome DevTools
// Watch for automatic pauses indicating memory limits
console.log("Monitoring memory usage...");

// Check for memory warnings and graph trends
if (performance.memory) {
  console.log("Memory usage:", {
    used: performance.memory.usedJSHeapSize,
    total: performance.memory.totalJSHeapSize,
    limit: performance.memory.jsHeapSizeLimit,
  });
}
```

### 5. Efficient Resource Management

- **IMPLEMENT** lazy loading for components not immediately visible
- **RELEASE** resources when no longer needed
- **MINIMIZE** recurring operations (intervals, subscriptions, effect hooks)
- **OPTIMIZE** images for appropriate size and resolution
- **CACHE** expensive calculations and complex operations
- **WHY**: Efficient resource management ensures responsiveness on limited devices

#### Resource Management Pattern

```javascript
// Lazy loading implementation
const LazyComponent = React.lazy(() => import("./ExpensiveComponent"));

// Resource cleanup
useEffect(() => {
  const interval = setInterval(updateData, 1000);

  return () => {
    clearInterval(interval); // Clean up on unmount
  };
}, []);

// Memory-efficient image optimization
const optimizeImage = (src, maxWidth = 800) => {
  return `${src}?width=${maxWidth}&quality=80&format=webp`;
};
```

## Component Selection & UI Design

### 6. Optimal Component Selection

- **USE** List component instead of ScrollView for long lists
- **PREFER** List component for memory efficiency on all platforms
- **UNDERSTAND** ScrollView renders all items at once vs List renders only visible items
- **OPTIMIZE** expensive calculations through caching and memoization
- **MOVE** expensive operations out of rendering logic
- **WHY**: Component choice significantly impacts performance on resource-constrained devices

#### Efficient List Implementation

```javascript
// Use List component for memory efficiency
import { List } from "@shopify/pos-ui-extensions";

function ProductList({ products }) {
  return (
    <List>
      {products.map((product) => (
        <List.Item key={product.id}>
          <ProductItem product={product} />
        </List.Item>
      ))}
    </List>
  );
}

// Memoize expensive calculations
const memoizedCalculation = useMemo(() => {
  return expensiveCalculation(data);
}, [data]);
```

### 7. Responsive UI Design Requirements

- **USE** Figma UI kit for extension design
- **DESIGN** for different screen sizes from the start
- **USE** relative measurements rather than fixed pixel values
- **TEST** layouts on both small and large screens
- **ENSURE** touch targets meet minimum size requirements:
  - **44×44 pixels minimum** for all interactive elements
  - **8 pixels minimum** space between touch targets
  - **Use Button component** for critical actions

#### Responsive Design Pattern

```css
/* Use relative measurements for responsive design */
.container {
  width: 100%;
  max-width: 800px;
  padding: 1rem;
  margin: 0 auto;
}

.touch-target {
  min-width: 44px;
  min-height: 44px;
  margin: 8px;
}

/* Responsive breakpoints */
@media (max-width: 768px) {
  .container {
    padding: 0.5rem;
  }
}
```

## Memory & Navigation Management

### 8. Memory Optimization Strategies

- **IMPLEMENT** pagination for large data sets
- **CLEAR** cached data when no longer needed
- **MONITOR** memory usage during development and testing
- **USE** efficient data structures for large collections
- **PREVENT** memory leaks through proper cleanup
- **WHY**: Excessive memory usage causes crashes and poor performance

#### Memory Management Implementation

```javascript
// Pagination for large datasets
const usePagination = (data, itemsPerPage = 20) => {
  const [currentPage, setCurrentPage] = useState(1);

  const paginatedData = useMemo(() => {
    const startIndex = (currentPage - 1) * itemsPerPage;
    return data.slice(startIndex, startIndex + itemsPerPage);
  }, [data, currentPage, itemsPerPage]);

  return { paginatedData, currentPage, setCurrentPage };
};

// Clear cached data
const clearCache = useCallback(() => {
  // Clear any cached data that's no longer needed
  cache.clear();
  localStorage.removeItem("tempData");
}, []);
```

### 9. Navigation Best Practices

- **USE** Navigation API efficiently for predictable screen management
- **UNDERSTAND** navigation lifecycle to prevent memory issues
- **MANAGE** navigation state carefully based on use case
- **CONSIDER** component-based approaches for simple UI changes
- **BE MINDFUL** that each screen consumes additional memory
- **WHY**: Proper navigation prevents memory issues and maintains performance

#### Navigation Implementation

```javascript
// Efficient navigation management
import { useNavigation } from "@shopify/pos-ui-extensions";

function NavigationExample() {
  const navigation = useNavigation();

  const handleNavigate = useCallback(
    (screen, params) => {
      // Use appropriate navigation method
      navigation.push(screen, params);
    },
    [navigation]
  );

  // Component-based approach for simple changes
  const [showDetails, setShowDetails] = useState(false);

  return (
    <View>
      {showDetails ? (
        <DetailComponent onClose={() => setShowDetails(false)} />
      ) : (
        <Button onPress={() => setShowDetails(true)}>Show Details</Button>
      )}
    </View>
  );
}
```

## API Optimization & Data Management

### 10. Efficient API Operations

- **BATCH** operations when possible to reduce network overhead
- **USE** bulk operations APIs (e.g., bulkSetLineItemDiscounts)
- **AVOID** chains of dependent network requests
- **IMPLEMENT** queue systems for sequential operations
- **CACHE** responses appropriately to avoid redundant requests
- **WHY**: Efficient network operations maintain responsive performance

#### API Optimization Patterns

```javascript
// Batch operations instead of individual calls
const batchUpdateLineItems = async (updates) => {
  try {
    // Single bulk operation instead of multiple individual calls
    const result = await api.bulkSetLineItemDiscounts(updates);
    return result;
  } catch (error) {
    console.error("Batch update failed:", error);
    throw error;
  }
};

// Implement caching with stale-while-revalidate pattern
const useCachedData = (key, fetcher) => {
  const [data, setData] = useState(cache.get(key));

  useEffect(() => {
    // Return cached data immediately, then revalidate
    if (data) return;

    fetcher().then((result) => {
      cache.set(key, result);
      setData(result);
    });
  }, [key, fetcher]);

  return data;
};
```

### 11. Network Operation Best Practices

- **IMPLEMENT** proper loading states for user feedback
- **ADD** comprehensive error handling for all network requests
- **SET** appropriate timeouts to prevent hanging operations
- **INCLUDE** fallback mechanisms when operations fail
- **USE** pagination for large datasets
- **WHY**: Robust network handling ensures good experience in varying conditions

#### Network Handling Implementation

```javascript
// Comprehensive network request handling
const useApiCall = (apiFunction, options = {}) => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);

  const execute = useCallback(
    async (...args) => {
      setLoading(true);
      setError(null);

      try {
        // Set timeout to prevent hanging
        const timeoutPromise = new Promise((_, reject) =>
          setTimeout(
            () => reject(new Error("Request timeout")),
            options.timeout || 10000
          )
        );

        const result = await Promise.race([
          apiFunction(...args),
          timeoutPromise,
        ]);

        setData(result);
        return result;
      } catch (err) {
        setError(err);
        // Implement fallback mechanism
        if (options.fallback) {
          return options.fallback();
        }
        throw err;
      } finally {
        setLoading(false);
      }
    },
    [apiFunction, options]
  );

  return { execute, loading, error, data };
};
```

## Component Lifecycle & State Management

### 12. Subscription and Effect Management

- **BE CAUTIOUS** with data subscriptions that trigger frequent UI updates
- **IMPLEMENT** debouncing or throttling for subscription updates
- **USE** appropriate caching techniques to prevent unnecessary recalculations
- **KEEP** side effects tightly scoped to prevent cascade updates
- **AVOID** complex state management patterns that trigger cascading effects
- **WHY**: Proper subscription management prevents performance degradation

#### Subscription Management Pattern

```javascript
// Debounced subscription updates
const useDebouncedSubscription = (subscription, delay = 300) => {
  const [value, setValue] = useState(subscription.current);

  useEffect(() => {
    const debouncedUpdate = debounce((newValue) => {
      setValue(newValue);
    }, delay);

    const unsubscribe = subscription.subscribe(debouncedUpdate);

    return () => {
      unsubscribe();
      debouncedUpdate.cancel();
    };
  }, [subscription, delay]);

  return value;
};

// Prevent race conditions
const useAsyncOperation = () => {
  const [state, setState] = useState({
    loading: false,
    data: null,
    error: null,
  });
  const cancelRef = useRef();

  const execute = useCallback(async (asyncFn) => {
    // Cancel previous operation
    if (cancelRef.current) {
      cancelRef.current();
    }

    let cancelled = false;
    cancelRef.current = () => {
      cancelled = true;
    };

    setState({ loading: true, data: null, error: null });

    try {
      const result = await asyncFn();
      if (!cancelled) {
        setState({ loading: false, data: result, error: null });
      }
    } catch (error) {
      if (!cancelled) {
        setState({ loading: false, data: null, error });
      }
    }
  }, []);

  useEffect(() => {
    return () => {
      if (cancelRef.current) {
        cancelRef.current();
      }
    };
  }, []);

  return { ...state, execute };
};
```

## Performance Troubleshooting

### 13. Common Performance Issues & Solutions

#### Slow Initial Load

- **DEFER** non-essential initialization
- **IMPLEMENT** progressive loading strategies
- **MINIMIZE** dependencies and bundle size
- **USE** code splitting for large applications

#### UI Lag During Interaction

- **MOVE** heavy processing off main thread where possible
- **OPTIMIZE** render cycles and reduce unnecessary updates
- **SIMPLIFY** complex UI elements
- **IMPLEMENT** virtual scrolling for large lists

#### High Memory Usage

- **RELEASE** unused resources promptly
- **USE** more efficient data structures
- **IMPLEMENT** pagination for large datasets
- **MONITOR** memory usage continuously

### 14. Entry-Level Device Optimization

- **UNDERSTAND** processing speeds and memory limitations
- **OPTIMIZE** for minimal resource usage
- **ENSURE** graceful degradation of enhanced features
- **MAINTAIN** responsive interactions (< 2 second response times)
- **TEST** core functionality on older/slower devices
- **WHY**: Ensures accessibility to widest possible merchant base

#### Performance Optimization Example

```javascript
// Progressive loading for entry-level devices
const ProgressiveLoader = ({ children, fallback }) => {
  const [isLowEnd, setIsLowEnd] = useState(false);

  useEffect(() => {
    // Detect low-end devices
    const memoryInfo = navigator.deviceMemory;
    const connectionInfo = navigator.connection;

    if (memoryInfo && memoryInfo <= 2) {
      setIsLowEnd(true);
    }

    if (connectionInfo && connectionInfo.effectiveType === "slow-2g") {
      setIsLowEnd(true);
    }
  }, []);

  return isLowEnd ? fallback : children;
};

// Efficient component for low-end devices
const OptimizedComponent = memo(({ data }) => {
  // Simplified rendering for performance
  return (
    <View>
      <Text>{data.title}</Text>
      {/* Only essential elements on low-end devices */}
    </View>
  );
});
```
