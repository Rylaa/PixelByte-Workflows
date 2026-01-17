# Core Patterns

Component patterns, data fetching, React 19 hooks, and state management patterns.

---

## 1. Component Patterns

### Standard Component Structure

```typescript
import React from 'react';
import useSWR from 'swr';
import { Card, CardHeader, CardTitle, CardContent } from '@/shared/components/ui/card';
import { cn } from '@/shared/utils/cn';
import { featureApi } from '../api/featureApi';
import type { Feature } from '../types';

interface FeatureCardProps {
    id: string;
    className?: string;
    onSelect?: (feature: Feature) => void;
}

export const FeatureCard: React.FC<FeatureCardProps> = ({
    id,
    className,
    onSelect,
}) => {
    const { data } = useSWR(
        `feature-${id}`,
        () => featureApi.getFeature(id),
        { suspense: true }
    );

    return (
        <Card
            className={cn("w-full cursor-pointer hover:bg-accent", className)}
            onClick={() => onSelect?.(data!)}
        >
            <CardHeader>
                <CardTitle>{data?.title}</CardTitle>
            </CardHeader>
            <CardContent>
                <p className="text-muted-foreground">{data?.description}</p>
            </CardContent>
        </Card>
    );
};

export default FeatureCard;
```

### Lazy Loading Pattern

```typescript
import React, { Suspense } from 'react';
import { SuspenseLoader } from '@/shared/components/SuspenseLoader';

// Lazy load heavy components
const DataGrid = React.lazy(() => import('@/shared/components/DataGrid'));
const RichTextEditor = React.lazy(() => import('@/shared/components/RichTextEditor'));
const Chart = React.lazy(() => import('./Chart'));

export const Dashboard: React.FC = () => {
    return (
        <div className="grid grid-cols-2 gap-4">
            <SuspenseLoader>
                <DataGrid data={[]} columns={[]} />
            </SuspenseLoader>

            <SuspenseLoader>
                <Chart type="line" />
            </SuspenseLoader>
        </div>
    );
};
```

### Props Interface Best Practices

```typescript
// Always define explicit interface
interface MyComponentProps {
    // Required props first
    id: string;
    title: string;

    // Optional props
    subtitle?: string;
    className?: string;

    // Callback props
    onSelect?: (id: string) => void;
    onClose?: () => void;

    // Children if needed
    children?: React.ReactNode;
}

// With default values
export const MyComponent: React.FC<MyComponentProps> = ({
    id,
    title,
    subtitle,
    className,
    onSelect,
    onClose,
    children,
}) => {
    // Component implementation
};
```

---

## 2. Data Fetching (SWR)

### Basic SWR with Suspense

```typescript
import useSWR from 'swr';
import { userApi } from '../api/userApi';

// Inner component - uses suspense
const UserProfileInner: React.FC<{ userId: string }> = ({ userId }) => {
    const { data: user } = useSWR(
        `user-${userId}`,
        () => userApi.getUser(userId),
        { suspense: true }
    );

    // data is always defined with suspense: true
    return <div>{user.name}</div>;
};

// Outer component - provides boundary
export const UserProfile: React.FC<{ userId: string }> = ({ userId }) => {
    return (
        <SuspenseLoader>
            <UserProfileInner userId={userId} />
        </SuspenseLoader>
    );
};
```

### SWR Configuration Options

```typescript
const { data, error, isValidating, mutate } = useSWR(
    'users',
    () => userApi.getUsers(),
    {
        suspense: true,                    // Enable Suspense mode
        revalidateOnFocus: false,          // Don't refetch on window focus
        revalidateOnReconnect: true,       // Refetch on reconnect
        dedupingInterval: 5 * 60 * 1000,   // 5 min deduplication
        refreshInterval: 0,                 // No auto-refresh (0 = disabled)
        errorRetryCount: 3,                // Retry 3 times on error
        onError: (error) => {
            toast.error('Failed to load data');
            Sentry.captureException(error);
        },
    }
);
```

### Cache Keys

```typescript
// Simple key
useSWR('users', fetcher);

// Parameterized key
useSWR(`user-${userId}`, fetcher);

// Array key (for complex params)
useSWR(['users', { page, limit }], ([_, params]) => fetcher(params));

// Conditional fetching
useSWR(userId ? `user-${userId}` : null, fetcher);
```

### Mutations and Revalidation

```typescript
import useSWR, { mutate } from 'swr';

const { data: user } = useSWR(`user-${userId}`, fetcher, { suspense: true });

const handleUpdate = async (newData: Partial<User>) => {
    try {
        await userApi.updateUser(userId, newData);

        // Revalidate single key
        await mutate(`user-${userId}`);

        // Or optimistic update
        await mutate(
            `user-${userId}`,
            { ...user, ...newData },
            { revalidate: false }
        );

        toast.success('Updated successfully');
    } catch (error) {
        toast.error('Update failed');
    }
};

// Revalidate multiple keys
const handleBulkAction = async () => {
    await api.bulkAction();
    await mutate((key) => typeof key === 'string' && key.startsWith('user-'));
};
```

### API Service Pattern

```typescript
// features/users/api/userApi.ts
import apiClient from '@/lib/apiClient';
import type { User, CreateUserPayload, UpdateUserPayload } from '../types';

export const userApi = {
    getUser: async (userId: string): Promise<User> => {
        const { data } = await apiClient.get(`/users/${userId}`);
        return data;
    },

    getUsers: async (params?: { page?: number; limit?: number }): Promise<User[]> => {
        const { data } = await apiClient.get('/users', { params });
        return data;
    },

    createUser: async (payload: CreateUserPayload): Promise<User> => {
        const { data } = await apiClient.post('/users', payload);
        return data;
    },

    updateUser: async (userId: string, payload: UpdateUserPayload): Promise<User> => {
        const { data } = await apiClient.put(`/users/${userId}`, payload);
        return data;
    },

    deleteUser: async (userId: string): Promise<void> => {
        await apiClient.delete(`/users/${userId}`);
    },
};
```

### Custom SWR Hooks

```typescript
// features/users/hooks/useUser.ts
import useSWR from 'swr';
import { userApi } from '../api/userApi';
import type { User } from '../types';

export function useUser(userId: string) {
    return useSWR<User>(
        userId ? `user-${userId}` : null,
        () => userApi.getUser(userId),
        {
            suspense: true,
            dedupingInterval: 5 * 60 * 1000,
        }
    );
}

export function useUsers(options?: { page?: number; limit?: number }) {
    return useSWR<User[]>(
        ['users', options],
        () => userApi.getUsers(options),
        { suspense: true }
    );
}
```

---

## 3. React 19 Hooks

### useActionState (Form Actions)

```typescript
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

// Server Action
async function submitForm(prevState: FormState, formData: FormData): Promise<FormState> {
    const email = formData.get('email') as string;

    try {
        await api.subscribe(email);
        return { success: true, message: 'Subscribed!' };
    } catch (error) {
        return { success: false, message: 'Failed to subscribe' };
    }
}

// Submit Button Component
function SubmitButton() {
    const { pending } = useFormStatus();

    return (
        <Button type="submit" disabled={pending}>
            {pending ? <Loader2 className="animate-spin size-4" /> : 'Subscribe'}
        </Button>
    );
}

// Form Component
export const SubscribeForm: React.FC = () => {
    const [state, formAction] = useActionState(submitForm, { success: false, message: '' });

    return (
        <form action={formAction} className="space-y-4">
            <Input name="email" type="email" placeholder="Enter email" required />
            <SubmitButton />
            {state.message && (
                <p className={state.success ? 'text-green-600' : 'text-red-600'}>
                    {state.message}
                </p>
            )}
        </form>
    );
};
```

### useOptimistic

```typescript
import { useOptimistic } from 'react';

interface Todo {
    id: string;
    title: string;
    completed: boolean;
}

export const TodoList: React.FC<{ todos: Todo[] }> = ({ todos }) => {
    const [optimisticTodos, addOptimisticTodo] = useOptimistic(
        todos,
        (state, newTodo: Todo) => [...state, newTodo]
    );

    const handleAdd = async (title: string) => {
        const tempTodo = { id: 'temp', title, completed: false };

        // Show immediately
        addOptimisticTodo(tempTodo);

        // Actually create
        await todoApi.create({ title });
        await mutate('todos');
    };

    return (
        <ul>
            {optimisticTodos.map((todo) => (
                <li key={todo.id} className={todo.id === 'temp' ? 'opacity-50' : ''}>
                    {todo.title}
                </li>
            ))}
        </ul>
    );
};
```

### use() Hook

```typescript
import { use, Suspense } from 'react';

// For promises
const UserName: React.FC<{ userPromise: Promise<User> }> = ({ userPromise }) => {
    const user = use(userPromise);
    return <span>{user.name}</span>;
};

// For context
const ThemeDisplay: React.FC = () => {
    const theme = use(ThemeContext);
    return <div className={theme}>{theme} mode</div>;
};

// Usage
export const UserCard: React.FC<{ userId: string }> = ({ userId }) => {
    const userPromise = userApi.getUser(userId);

    return (
        <Suspense fallback={<Skeleton />}>
            <UserName userPromise={userPromise} />
        </Suspense>
    );
};
```

---

## 4. State Management

### Decision Guide

| Data Type | Solution | Example |
|-----------|----------|---------|
| Server/API data | SWR | User profiles, lists |
| Form inputs | useState | Input values, validation |
| UI state (local) | useState | Modal open, tab index |
| UI state (shared) | Zustand | Sidebar collapsed |
| Complex client state | Zustand | Shopping cart, filters |

### SWR for Server State

```typescript
const { data: users } = useSWR('users', fetchUsers, { suspense: true });
```

> **For SWR performance features:** See [performance-guide.md](performance-guide.md) - Client-Side Data Fetching section

### useState for Local State

```typescript
export const SearchBox: React.FC = () => {
    const [query, setQuery] = useState('');
    const [isOpen, setIsOpen] = useState(false);

    return (
        <div>
            <Input
                value={query}
                onChange={(e) => setQuery(e.target.value)}
                onFocus={() => setIsOpen(true)}
            />
            {isOpen && <SearchResults query={query} />}
        </div>
    );
};
```

### Zustand for Shared Client State

```typescript
// stores/uiStore.ts
import { create } from 'zustand';

interface UIState {
    sidebarOpen: boolean;
    theme: 'light' | 'dark';
    toggleSidebar: () => void;
    setTheme: (theme: 'light' | 'dark') => void;
}

export const useUIStore = create<UIState>((set) => ({
    sidebarOpen: true,
    theme: 'dark',
    toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
    setTheme: (theme) => set({ theme }),
}));

// Usage - component only re-renders when subscribed state changes
const Sidebar: React.FC = () => {
    const isOpen = useUIStore((state) => state.sidebarOpen);
    return isOpen ? <SidebarContent /> : null;
};

const ToggleButton: React.FC = () => {
    const toggleSidebar = useUIStore((state) => state.toggleSidebar);
    return <Button onClick={toggleSidebar}>Toggle</Button>;
};
```

### Zustand with Slices (Complex State)

```typescript
// stores/studioStore.ts
import { create } from 'zustand';
import { createUISlice, UISlice } from './slices/ui.slice';
import { createMediaSlice, MediaSlice } from './slices/media.slice';

type StudioStore = UISlice & MediaSlice;

export const useStudioStore = create<StudioStore>()((...args) => ({
    ...createUISlice(...args),
    ...createMediaSlice(...args),
}));

// stores/slices/ui.slice.ts
export interface UISlice {
    activeTab: string;
    setActiveTab: (tab: string) => void;
}

export const createUISlice = (set: SetState<UISlice>): UISlice => ({
    activeTab: 'generate',
    setActiveTab: (tab) => set({ activeTab: tab }),
});
```

---

## 5. Common Patterns

### Forms with React Hook Form + Zod

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
    email: z.string().email('Invalid email'),
    password: z.string().min(8, 'Min 8 characters'),
});

type FormData = z.infer<typeof schema>;

export const LoginForm: React.FC = () => {
    const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<FormData>({
        resolver: zodResolver(schema),
    });

    const onSubmit = async (data: FormData) => {
        try {
            await authApi.login(data);
            toast.success('Logged in!');
        } catch {
            toast.error('Login failed');
        }
    };

    return (
        <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
            <div>
                <Input {...register('email')} placeholder="Email" />
                {errors.email && (
                    <p className="text-sm text-destructive mt-1">{errors.email.message}</p>
                )}
            </div>
            <div>
                <Input {...register('password')} type="password" placeholder="Password" />
                {errors.password && (
                    <p className="text-sm text-destructive mt-1">{errors.password.message}</p>
                )}
            </div>
            <Button type="submit" disabled={isSubmitting} className="w-full">
                {isSubmitting ? <Loader2 className="animate-spin" /> : 'Login'}
            </Button>
        </form>
    );
};
```

### Dialog Pattern

```typescript
import {
    Dialog,
    DialogContent,
    DialogHeader,
    DialogTitle,
    DialogFooter,
} from '@/shared/components/ui/dialog';

interface ConfirmDialogProps {
    open: boolean;
    onOpenChange: (open: boolean) => void;
    title: string;
    description: string;
    onConfirm: () => void;
    loading?: boolean;
}

export const ConfirmDialog: React.FC<ConfirmDialogProps> = ({
    open,
    onOpenChange,
    title,
    description,
    onConfirm,
    loading = false,
}) => {
    return (
        <Dialog open={open} onOpenChange={onOpenChange}>
            <DialogContent>
                <DialogHeader>
                    <DialogTitle>{title}</DialogTitle>
                </DialogHeader>
                <p className="text-muted-foreground">{description}</p>
                <DialogFooter>
                    <Button variant="outline" onClick={() => onOpenChange(false)}>
                        Cancel
                    </Button>
                    <Button variant="destructive" onClick={onConfirm} disabled={loading}>
                        {loading ? <Loader2 className="animate-spin" /> : 'Confirm'}
                    </Button>
                </DialogFooter>
            </DialogContent>
        </Dialog>
    );
};
```

### Debounced Search

```typescript
import { useState, useMemo } from 'react';
import { useDebouncedValue } from '@/shared/hooks/useDebouncedValue';
import useSWR from 'swr';

export const SearchInput: React.FC = () => {
    const [query, setQuery] = useState('');
    const debouncedQuery = useDebouncedValue(query, 300);

    const { data: results } = useSWR(
        debouncedQuery ? `search-${debouncedQuery}` : null,
        () => searchApi.search(debouncedQuery),
        { suspense: false } // Don't suspend for search
    );

    return (
        <div>
            <Input
                value={query}
                onChange={(e) => setQuery(e.target.value)}
                placeholder="Search..."
            />
            {results?.map((item) => (
                <div key={item.id}>{item.title}</div>
            ))}
        </div>
    );
};
```

### Error Boundary

```typescript
import { ErrorBoundary, FallbackProps } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }: FallbackProps) {
    return (
        <div className="p-8 text-center">
            <h2 className="text-xl font-semibold text-destructive">Something went wrong</h2>
            <p className="text-muted-foreground mt-2">{error.message}</p>
            <Button onClick={resetErrorBoundary} className="mt-4">
                Try Again
            </Button>
        </div>
    );
}

export const SafeComponent: React.FC<{ children: React.ReactNode }> = ({ children }) => {
    return (
        <ErrorBoundary
            FallbackComponent={ErrorFallback}
            onError={(error) => Sentry.captureException(error)}
        >
            {children}
        </ErrorBoundary>
    );
};
```

---

## Summary

| Pattern | When to Use |
|---------|-------------|
| `React.FC<Props>` | All components |
| `useSWR` with suspense | Server data fetching |
| `SuspenseLoader` | Loading boundaries |
| `React.lazy()` | Heavy components |
| `useActionState` | Form submissions |
| `useOptimistic` | Optimistic UI updates |
| `useState` | Local UI state |
| `Zustand` | Shared client state |
| React Hook Form + Zod | Form validation |
