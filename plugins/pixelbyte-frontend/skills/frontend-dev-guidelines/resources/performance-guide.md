# Performance Guide

45 performance rules from Vercel Engineering, organized by priority and impact.

---

## Rule Priority Overview

| Priority | Category | Impact | Focus |
|----------|----------|--------|-------|
| 1 | Eliminating Waterfalls | CRITICAL | Network/data fetching |
| 2 | Bundle Size | CRITICAL | Initial load time |
| 3 | Server-Side | HIGH | RSC optimization |
| 4 | Client-Side Fetching | MEDIUM-HIGH | SWR patterns |
| 5 | Re-render | MEDIUM | React optimization |
| 6 | Rendering | MEDIUM | DOM/CSS |
| 7 | JavaScript | LOW-MEDIUM | JS micro-optimizations |
| 8 | Advanced | LOW | Edge cases |

---

## 1. Eliminating Waterfalls (CRITICAL)

### Rule: Defer await until needed

```typescript
// ❌ WRONG - Blocks entire function
async function handler(shouldLog: boolean) {
    const data = await fetchData();
    if (shouldLog) {
        console.log(data);
    }
    return data;
}

// ✅ CORRECT - Only await when needed
async function handler(shouldLog: boolean) {
    const dataPromise = fetchData();
    if (shouldLog) {
        console.log(await dataPromise);
    }
    return dataPromise;
}
```

### Rule: Use Promise.all() for independent operations

```typescript
// ❌ WRONG - Sequential (waterfall)
const users = await getUsers();
const posts = await getPosts();
const comments = await getComments();
// Total: 300ms + 200ms + 150ms = 650ms

// ✅ CORRECT - Parallel
const [users, posts, comments] = await Promise.all([
    getUsers(),
    getPosts(),
    getComments(),
]);
// Total: max(300ms, 200ms, 150ms) = 300ms
```

### Rule: Handle partial dependencies with better-all

```typescript
// ❌ WRONG - Users blocks posts that need userId
const users = await getUsers();
const posts = await getPosts(users[0].id);

// ✅ CORRECT - Use promise chaining or better-all
import { all } from 'better-all';

const { users, posts } = await all({
    users: getUsers(),
    posts: async (ctx) => {
        const users = await ctx.users;
        return getPosts(users[0].id);
    },
});
```

### Rule: Strategic Suspense boundaries

```typescript
// ❌ WRONG - Single boundary blocks everything
<Suspense fallback={<Loading />}>
    <SlowComponent />
    <FastComponent />
</Suspense>

// ✅ CORRECT - Granular boundaries
<div className="grid grid-cols-2">
    <Suspense fallback={<Skeleton />}>
        <SlowComponent />
    </Suspense>
    <Suspense fallback={<Skeleton />}>
        <FastComponent />
    </Suspense>
</div>
```

### Rule: Parallel fetching with component composition

```typescript
// ❌ WRONG - Parent fetches then passes to children
async function Dashboard() {
    const user = await getUser();
    const posts = await getPosts(user.id);
    return (
        <div>
            <UserCard user={user} />
            <PostList posts={posts} />
        </div>
    );
}

// ✅ CORRECT - Each component fetches independently
async function Dashboard() {
    return (
        <div>
            <Suspense fallback={<UserSkeleton />}>
                <UserCard />
            </Suspense>
            <Suspense fallback={<PostSkeleton />}>
                <PostList />
            </Suspense>
        </div>
    );
}

async function UserCard() {
    const user = await getUser();
    return <Card>{user.name}</Card>;
}

async function PostList() {
    const posts = await getPosts();
    return <List items={posts} />;
}
```

---

## 2. Bundle Size (CRITICAL)

### Rule: Avoid barrel file imports

```typescript
// ❌ WRONG - Barrel import pulls entire module
import { Button, Card } from '@/shared/components/ui';

// ✅ CORRECT - Direct imports
import { Button } from '@/shared/components/ui/button';
import { Card } from '@/shared/components/ui/card';
```

### Rule: Dynamic imports for heavy components

```typescript
// ❌ WRONG - Included in main bundle
import { DataGrid } from '@/components/DataGrid';
import { Chart } from '@/components/Chart';

// ✅ CORRECT - Loaded on demand
import dynamic from 'next/dynamic';

const DataGrid = dynamic(() => import('@/components/DataGrid'), {
    loading: () => <Skeleton className="h-96" />,
});

const Chart = dynamic(() => import('@/components/Chart'), {
    ssr: false, // Disable SSR for client-only libs
});
```

### Rule: Defer third-party scripts

```typescript
// ❌ WRONG - Blocks hydration
import { Analytics } from '@vercel/analytics';

// ✅ CORRECT - Load after hydration
import dynamic from 'next/dynamic';

const Analytics = dynamic(
    () => import('@vercel/analytics').then((mod) => mod.Analytics),
    { ssr: false }
);

// Or use next/script
import Script from 'next/script';

<Script
    src="https://analytics.example.com/script.js"
    strategy="afterInteractive"
/>
```

### Rule: Conditional module loading

```typescript
// ❌ WRONG - Always loaded
import { AdvancedEditor } from '@/components/AdvancedEditor';

function Editor({ mode }) {
    if (mode === 'advanced') {
        return <AdvancedEditor />;
    }
    return <SimpleEditor />;
}

// ✅ CORRECT - Load only when needed
import dynamic from 'next/dynamic';

const AdvancedEditor = dynamic(() => import('@/components/AdvancedEditor'));

function Editor({ mode }) {
    if (mode === 'advanced') {
        return <AdvancedEditor />;
    }
    return <SimpleEditor />;
}
```

### Rule: Preload on hover/focus

```typescript
import dynamic from 'next/dynamic';

const HeavyModal = dynamic(() => import('./HeavyModal'));

function Button() {
    const [showModal, setShowModal] = useState(false);

    // Preload when user hovers
    const handleMouseEnter = () => {
        import('./HeavyModal');
    };

    return (
        <>
            <button
                onMouseEnter={handleMouseEnter}
                onClick={() => setShowModal(true)}
            >
                Open
            </button>
            {showModal && <HeavyModal />}
        </>
    );
}
```

---

## 3. Server-Side Performance (HIGH)

### Rule: React.cache() for per-request deduplication

```typescript
import { cache } from 'react';

// Deduplicated within same request
export const getUser = cache(async (id: string) => {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
});

// Called multiple times but only fetches once per request
async function Header() {
    const user = await getUser('123');
    return <nav>{user.name}</nav>;
}

async function Sidebar() {
    const user = await getUser('123'); // Same request, reuses cache
    return <aside>{user.email}</aside>;
}
```

### Rule: LRU cache for cross-request caching

```typescript
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, any>({
    max: 500,
    ttl: 1000 * 60 * 5, // 5 minutes
});

async function getCachedData(key: string) {
    const cached = cache.get(key);
    if (cached) return cached;

    const data = await fetchExpensiveData(key);
    cache.set(key, data);
    return data;
}
```

### Rule: Minimize serialization at RSC boundaries

```typescript
// ❌ WRONG - Passes entire object to client
async function Page() {
    const user = await getUser(); // { id, name, email, settings, permissions, ... }
    return <ClientComponent user={user} />;
}

// ✅ CORRECT - Pass only needed data
async function Page() {
    const user = await getUser();
    return (
        <ClientComponent
            userName={user.name}
            userEmail={user.email}
        />
    );
}
```

### Rule: Use after() for non-blocking operations

```typescript
import { after } from 'next/server';

async function Page() {
    const data = await getData();

    // Don't block response for analytics
    after(async () => {
        await trackPageView();
        await logAnalytics(data);
    });

    return <Content data={data} />;
}
```

---

## 4. Client-Side Data Fetching (MEDIUM-HIGH)

### Rule: Use SWR for automatic deduplication

```typescript
// ❌ WRONG - Multiple components fetch same data
function ComponentA() {
    const [user, setUser] = useState(null);
    useEffect(() => {
        fetch('/api/user').then(r => r.json()).then(setUser);
    }, []);
}

function ComponentB() {
    const [user, setUser] = useState(null);
    useEffect(() => {
        fetch('/api/user').then(r => r.json()).then(setUser);
    }, []);
}

// ✅ CORRECT - SWR deduplicates automatically
function ComponentA() {
    const { data: user } = useSWR('user', fetcher, { suspense: true });
}

function ComponentB() {
    const { data: user } = useSWR('user', fetcher, { suspense: true });
    // Same key = same request, no duplication
}
```

### Rule: Deduplicate global event listeners

```typescript
// ❌ WRONG - Multiple listeners
function useWindowSize() {
    const [size, setSize] = useState({ width: 0, height: 0 });

    useEffect(() => {
        const handler = () => setSize({
            width: window.innerWidth,
            height: window.innerHeight,
        });
        window.addEventListener('resize', handler);
        return () => window.removeEventListener('resize', handler);
    }, []);

    return size;
}

// ✅ CORRECT - Shared listener with useSyncExternalStore
import { useSyncExternalStore } from 'react';

const windowSizeStore = {
    subscribe: (callback: () => void) => {
        window.addEventListener('resize', callback);
        return () => window.removeEventListener('resize', callback);
    },
    getSnapshot: () => ({
        width: window.innerWidth,
        height: window.innerHeight,
    }),
};

function useWindowSize() {
    return useSyncExternalStore(
        windowSizeStore.subscribe,
        windowSizeStore.getSnapshot
    );
}
```

---

## 5. Re-render Optimization (MEDIUM)

### Rule: Don't subscribe to state only used in callbacks

```typescript
// ❌ WRONG - Re-renders on every count change
function Counter() {
    const count = useStore((state) => state.count);
    const increment = useStore((state) => state.increment);

    const handleClick = () => {
        console.log('Current:', count);
        increment();
    };

    return <button onClick={handleClick}>+</button>;
}

// ✅ CORRECT - Only subscribe to what renders
function Counter() {
    const increment = useStore((state) => state.increment);
    const getCount = useStore((state) => state.getCount);

    const handleClick = () => {
        console.log('Current:', getCount());
        increment();
    };

    return <button onClick={handleClick}>+</button>;
}
```

### Rule: Extract expensive work into memoized components

```typescript
// ❌ WRONG - ExpensiveList re-renders on every parent update
function Parent() {
    const [filter, setFilter] = useState('');
    const [count, setCount] = useState(0);

    return (
        <div>
            <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
            <ExpensiveList filter={filter} />
        </div>
    );
}

// ✅ CORRECT - Memoize expensive component
const MemoizedList = React.memo(ExpensiveList);

function Parent() {
    const [filter, setFilter] = useState('');
    const [count, setCount] = useState(0);

    return (
        <div>
            <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
            <MemoizedList filter={filter} />
        </div>
    );
}
```

### Rule: Use primitive dependencies

```typescript
// ❌ WRONG - Effect runs on every render (new object each time)
useEffect(() => {
    fetchData(options);
}, [options]); // { page: 1, limit: 10 }

// ✅ CORRECT - Primitive values
useEffect(() => {
    fetchData({ page, limit });
}, [page, limit]);
```

### Rule: Functional setState for stable callbacks

```typescript
// ❌ WRONG - New function on every render
function Counter() {
    const [count, setCount] = useState(0);

    const increment = () => setCount(count + 1);

    return <ExpensiveChild onClick={increment} />;
}

// ✅ CORRECT - Stable reference with useCallback + functional update
function Counter() {
    const [count, setCount] = useState(0);

    const increment = useCallback(() => {
        setCount(c => c + 1);
    }, []); // Empty deps - function never changes

    return <ExpensiveChild onClick={increment} />;
}
```

### Rule: Lazy state initialization

```typescript
// ❌ WRONG - Expensive calculation runs every render
const [data, setData] = useState(expensiveCalculation());

// ✅ CORRECT - Only runs once
const [data, setData] = useState(() => expensiveCalculation());
```

### Rule: Use startTransition for non-urgent updates

```typescript
import { useState, startTransition } from 'react';

function SearchResults() {
    const [query, setQuery] = useState('');
    const [results, setResults] = useState([]);

    const handleChange = (e) => {
        setQuery(e.target.value); // Urgent - update input immediately

        startTransition(() => {
            setResults(filterResults(e.target.value)); // Non-urgent - can be deferred
        });
    };

    return (
        <div>
            <input value={query} onChange={handleChange} />
            <ResultList results={results} />
        </div>
    );
}
```

---

## 6. Rendering Performance (MEDIUM)

### Rule: CSS content-visibility for long lists

```css
.list-item {
    content-visibility: auto;
    contain-intrinsic-size: 0 100px; /* Estimated height */
}
```

```typescript
function LongList({ items }) {
    return (
        <ul>
            {items.map((item) => (
                <li key={item.id} className="content-visibility-auto">
                    <Item data={item} />
                </li>
            ))}
        </ul>
    );
}
```

### Rule: Animate wrapper div, not SVG

```typescript
// ❌ WRONG - Animating SVG directly causes repaints
<motion.svg animate={{ rotate: 360 }}>
    <path d="..." />
</motion.svg>

// ✅ CORRECT - Animate wrapper, SVG stays static
<motion.div animate={{ rotate: 360 }}>
    <svg>
        <path d="..." />
    </svg>
</motion.div>
```

### Rule: Use ternary, not && for conditionals

```typescript
// ❌ WRONG - Can render "0" or "false"
{count && <Badge count={count} />}
{isLoading && <Spinner />}

// ✅ CORRECT - Explicit ternary
{count > 0 ? <Badge count={count} /> : null}
{isLoading ? <Spinner /> : null}
```

---

## 7. JavaScript Performance (LOW-MEDIUM)

### Rule: Build index maps for repeated lookups

```typescript
// ❌ WRONG - O(n) lookup each time
function findUser(users, id) {
    return users.find(u => u.id === id);
}

// ✅ CORRECT - O(1) lookup with Map
const userMap = new Map(users.map(u => [u.id, u]));

function findUser(id) {
    return userMap.get(id);
}
```

### Rule: Use Set/Map for O(1) lookups

```typescript
// ❌ WRONG - O(n) array includes
const selectedIds = [1, 2, 3, 4, 5];
items.filter(item => selectedIds.includes(item.id));

// ✅ CORRECT - O(1) Set has
const selectedIds = new Set([1, 2, 3, 4, 5]);
items.filter(item => selectedIds.has(item.id));
```

### Rule: Combine iterations

```typescript
// ❌ WRONG - 3 iterations
const result = items
    .filter(item => item.active)
    .map(item => item.value)
    .filter(value => value > 0);

// ✅ CORRECT - 1 iteration
const result = items.reduce((acc, item) => {
    if (item.active && item.value > 0) {
        acc.push(item.value);
    }
    return acc;
}, []);
```

### Rule: Check length before expensive comparison

```typescript
// ❌ WRONG - Deep comparison even for different lengths
if (JSON.stringify(arr1) === JSON.stringify(arr2)) { ... }

// ✅ CORRECT - Quick length check first
if (arr1.length === arr2.length && JSON.stringify(arr1) === JSON.stringify(arr2)) { ... }
```

### Rule: Use toSorted() for immutability

```typescript
// ❌ WRONG - Mutates original array
const sorted = items.sort((a, b) => a.name.localeCompare(b.name));

// ✅ CORRECT - Returns new sorted array
const sorted = items.toSorted((a, b) => a.name.localeCompare(b.name));
```

### Rule: Early return from functions

```typescript
// ❌ WRONG - Nested conditions
function process(item) {
    if (item) {
        if (item.isValid) {
            if (item.data) {
                return transform(item.data);
            }
        }
    }
    return null;
}

// ✅ CORRECT - Early returns
function process(item) {
    if (!item) return null;
    if (!item.isValid) return null;
    if (!item.data) return null;
    return transform(item.data);
}
```

---

## Quick Reference

| Problem | Solution |
|---------|----------|
| Sequential API calls | `Promise.all()` |
| Large bundle size | `next/dynamic`, direct imports |
| Slow initial load | Suspense boundaries, streaming |
| Duplicate requests | SWR, React.cache() |
| Unnecessary re-renders | React.memo, useCallback |
| Long lists | content-visibility, virtualization |
| Repeated lookups | Map/Set instead of Array |

---

## Performance Checklist

- [ ] No sequential await for independent operations
- [ ] No barrel file imports
- [ ] Heavy components use dynamic import
- [ ] Each data-fetching component has Suspense boundary
- [ ] SWR used for client-side data
- [ ] Expensive components wrapped with React.memo
- [ ] useCallback for handlers passed to children
- [ ] Lazy state initialization for expensive defaults
- [ ] startTransition for non-urgent updates
- [ ] Long lists use content-visibility or virtualization
