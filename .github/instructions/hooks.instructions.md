---
description: "Guidelines for custom React hooks that encapsulate reusable stateful logic"
applyTo: "src/hooks/**"
---

## 🎯 What Are Custom Hooks?

Custom hooks are **reusable functions that encapsulate stateful logic**.  
Think: `useAuth`, `useDebounce`, `useLocalStorage`, `useMediaQuery`, `useFetch`.

### Core Principles:

- ✅ **Start with "use"** - Required by React (linting rules)
- ✅ **DRY (don't repeat your self)** - Used across multiple components
- ✅ **Single responsibility** - Each hook does one thing well
- ✅ **Return consistent shape** - Predictable return values
- ✅ **Type-safe** - Full TypeScript types for requests and responses
- ✅ **Separation of concerns** - Keep the logic out of components

---

## ✅ Hook Best Practices

### 1. Always Start with "use"

```tsx
// ✅ GOOD
export const useAuth = () => {
  /* ... */
};
export const useDebounce = () => {
  /* ... */
};

// ❌ BAD - Not a valid hook name
export const getAuth = () => {
  /* ... */
};
export const debounce = () => {
  /* ... */
};
```

### 2. Return Consistent Shapes

```tsx
// ✅ GOOD - Object with named properties
export const useAuth = () => {
  return {
    user,
    login,
    logout,
    isAuthenticated,
    isLoading,
  };
};

// ✅ ALSO GOOD - Tuple for simple cases
export const useToggle = (initial: boolean) => {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue(!value);
  return [value, toggle] as const;
};

// ❌ BAD - Inconsistent return
export const useAuth = () => {
  if (isLoading) return null; // ❌ Different return type
  return { user, login, logout };
};
```

### 3. Document with JSDoc

```tsx
/**
 * Debounce a value - delays updating until user stops changing it
 *
 * @param value - The value to debounce
 * @param delay - Delay in milliseconds (default: 500ms)
 * @returns Debounced value
 *
 * @example
 * const debouncedSearch = useDebounce(search, 500);
 */
export const useDebounce = <T,>(value: T, delay = 500): T => {
  // Implementation
};
```

### 4. Use TypeScript Generics for Flexibility

```tsx
// ✅ GOOD - Generic hook works with any type
export const useLocalStorage = <T,>(key: string, initialValue: T) => {
  // Type-safe for any T
};

// Usage:
const [user, setUser] = useLocalStorage<User | undefined>("user", undefined);
const [theme, setTheme] = useLocalStorage<"light" | "dark">("theme", "light");
```

### 5. Cleanup Side Effects

```tsx
// ✅ GOOD - Always cleanup
export const useEventListener = (event: string, handler: () => void) => {
  useEffect(() => {
    window.addEventListener(event, handler);

    return () => {
      window.removeEventListener(event, handler); // ✅ Cleanup
    };
  }, [event, handler]);
};

// ❌ BAD - Memory leak!
export const useEventListener = (event: string, handler: () => void) => {
  useEffect(() => {
    window.addEventListener(event, handler); // ❌ No cleanup
  }, [event, handler]);
};
```

### 6. Stabilize Callbacks with useCallback

```tsx
// ✅ GOOD - Memoized functions in dependencies
export const useClickOutside = (handler: () => void) => {
  const ref = useRef(null);

  // Stabilize handler to prevent effect re-running unnecessarily
  const stableHandler = useCallback(handler, []);

  useEffect(() => {
    const handleClick = (e: MouseEvent) => {
      if (ref.current && !ref.current.contains(e.target)) {
        stableHandler();
      }
    };

    document.addEventListener("mousedown", handleClick);
    return () => document.removeEventListener("mousedown", handleClick);
  }, [stableHandler]);

  return ref;
};
```

### 7. Handle SSR (Server-Side Rendering)

```tsx
// ✅ GOOD - SSR-safe
export const useWindowSize = () => {
  const [size, setSize] = useState({
    width: typeof window !== "undefined" ? window.innerWidth : 0,
    height: typeof window !== "undefined" ? window.innerHeight : 0,
  });

  useEffect(() => {
    // This only runs on client
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
};
```

---

## ❌ Common Mistakes

### Mistake 1: Not Following Rules of Hooks

```tsx
// ❌ BAD - Conditional hook call
export const useConditionalData = (shouldFetch: boolean) => {
  if (shouldFetch) {
    const { data } = useQuery(/* ... */); // ❌ Conditional hook!
    return data;
  }
  return null;
};

// ✅ GOOD - Hook always called
export const useConditionalData = (shouldFetch: boolean) => {
  const { data } = useQuery({
    queryKey: ["data"],
    queryFn: fetchData,
    enabled: shouldFetch, // ✅ Use enabled option
  });

  return data;
};
```

### Mistake 2: Missing Dependencies

```tsx
// ❌ BAD - Missing dependency
export const useFetch = (url: string) => {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then(setData);
  }, []); // ❌ Missing 'url' dependency!

  return data;
};

// ✅ GOOD - Complete dependencies
export const useFetch = (url: string) => {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then(setData);
  }, [url]); // ✅ Includes 'url'

  return data;
};
```

### Mistake 3: Not Memoizing Returned Functions

```tsx
// ❌ BAD - New function on every render
export const useCounter = () => {
  const [count, setCount] = useState(0);

  const increment = () => setCount((c) => c + 1); // ❌ New function each time

  return { count, increment };
};

// ✅ GOOD - Memoized function
export const useCounter = () => {
  const [count, setCount] = useState(0);

  const increment = useCallback(() => setCount((c) => c + 1), []); // ✅ Stable

  return { count, increment };
};
```

### Mistake 4: Doing Too Much in One Hook

```tsx
// ❌ BAD - Hook doing everything
export const useUserDashboard = (userId: string) => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [friends, setFriends] = useState([]);
  const [notifications, setNotifications] = useState([]);

  // 100 lines of logic...

  return { user, posts, friends, notifications /* ... */ };
};

// ✅ GOOD - Separate focused hooks
export const useUser = (userId: string) => {
  /* ... */
};
export const usePosts = (userId: string) => {
  /* ... */
};
export const useFriends = (userId: string) => {
  /* ... */
};
export const useNotifications = (userId: string) => {
  /* ... */
};

// Compose in component:
const Dashboard = ({ userId }: { userId: string }) => {
  const { data: user } = useUser(userId);
  const { data: posts } = usePosts(userId);
  const { data: friends } = useFriends(userId);
  const { data: notifications } = useNotifications(userId);

  // ...
};
```
