# Testing & Complete Examples

Vitest + React Testing Library patterns and full working code examples.

---

## 1. Testing Setup

### Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
    plugins: [react()],
    test: {
        environment: 'jsdom',
        globals: true,
        setupFiles: ['./src/test/setup.ts'],
        include: ['**/*.{test,spec}.{ts,tsx}'],
    },
    resolve: {
        alias: {
            '@': path.resolve(__dirname, './src'),
        },
    },
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach, vi } from 'vitest';

afterEach(() => {
    cleanup();
});

// Mock next/navigation
vi.mock('next/navigation', () => ({
    useRouter: () => ({
        push: vi.fn(),
        replace: vi.fn(),
        back: vi.fn(),
    }),
    usePathname: () => '/',
    useSearchParams: () => new URLSearchParams(),
}));
```

---

## 2. Component Testing

### Basic Component Test

```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
    it('renders with children', () => {
        render(<Button>Click me</Button>);
        expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
    });

    it('calls onClick when clicked', async () => {
        const handleClick = vi.fn();
        const user = userEvent.setup();

        render(<Button onClick={handleClick}>Click me</Button>);
        await user.click(screen.getByRole('button'));

        expect(handleClick).toHaveBeenCalledOnce();
    });

    it('is disabled when disabled prop is true', () => {
        render(<Button disabled>Click me</Button>);
        expect(screen.getByRole('button')).toBeDisabled();
    });

    it('shows loading state', () => {
        render(<Button isLoading>Submit</Button>);
        expect(screen.getByRole('button')).toHaveAttribute('aria-busy', 'true');
    });
});
```

### Testing with SWR

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import { SWRConfig } from 'swr';
import { Suspense } from 'react';
import { describe, it, expect, vi } from 'vitest';
import { UserProfile } from './UserProfile';

// Mock API
vi.mock('../api/userApi', () => ({
    userApi: {
        getUser: vi.fn().mockResolvedValue({
            id: '1',
            name: 'John Doe',
            email: 'john@example.com',
        }),
    },
}));

const wrapper = ({ children }: { children: React.ReactNode }) => (
    <SWRConfig value={{ provider: () => new Map() }}>
        <Suspense fallback={<div>Loading...</div>}>
            {children}
        </Suspense>
    </SWRConfig>
);

describe('UserProfile', () => {
    it('renders user data', async () => {
        render(<UserProfile userId="1" />, { wrapper });

        await waitFor(() => {
            expect(screen.getByText('John Doe')).toBeInTheDocument();
        });

        expect(screen.getByText('john@example.com')).toBeInTheDocument();
    });
});
```

### Testing Forms

```typescript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
    it('submits with valid data', async () => {
        const onSubmit = vi.fn();
        const user = userEvent.setup();

        render(<LoginForm onSubmit={onSubmit} />);

        await user.type(screen.getByLabelText(/email/i), 'test@example.com');
        await user.type(screen.getByLabelText(/password/i), 'password123');
        await user.click(screen.getByRole('button', { name: /submit/i }));

        await waitFor(() => {
            expect(onSubmit).toHaveBeenCalledWith({
                email: 'test@example.com',
                password: 'password123',
            });
        });
    });

    it('shows validation errors', async () => {
        const user = userEvent.setup();

        render(<LoginForm onSubmit={vi.fn()} />);

        await user.click(screen.getByRole('button', { name: /submit/i }));

        await waitFor(() => {
            expect(screen.getByText(/email is required/i)).toBeInTheDocument();
        });
    });

    it('shows error for invalid email', async () => {
        const user = userEvent.setup();

        render(<LoginForm onSubmit={vi.fn()} />);

        await user.type(screen.getByLabelText(/email/i), 'invalid-email');
        await user.click(screen.getByRole('button', { name: /submit/i }));

        await waitFor(() => {
            expect(screen.getByText(/invalid email/i)).toBeInTheDocument();
        });
    });
});
```

### Testing Hooks

```typescript
import { renderHook, waitFor } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { useDebounce } from './useDebounce';

describe('useDebounce', () => {
    beforeEach(() => {
        vi.useFakeTimers();
    });

    afterEach(() => {
        vi.useRealTimers();
    });

    it('returns initial value immediately', () => {
        const { result } = renderHook(() => useDebounce('initial', 500));
        expect(result.current).toBe('initial');
    });

    it('debounces value updates', async () => {
        const { result, rerender } = renderHook(
            ({ value }) => useDebounce(value, 500),
            { initialProps: { value: 'initial' } }
        );

        rerender({ value: 'updated' });
        expect(result.current).toBe('initial');

        vi.advanceTimersByTime(500);

        await waitFor(() => {
            expect(result.current).toBe('updated');
        });
    });
});
```

### Accessibility Testing

```typescript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
import { describe, it, expect } from 'vitest';
import { Button } from './Button';

expect.extend(toHaveNoViolations);

describe('Button accessibility', () => {
    it('has no accessibility violations', async () => {
        const { container } = render(<Button>Click me</Button>);
        const results = await axe(container);
        expect(results).toHaveNoViolations();
    });

    it('icon button has aria-label', async () => {
        const { container } = render(
            <Button aria-label="Close">
                <XIcon />
            </Button>
        );
        const results = await axe(container);
        expect(results).toHaveNoViolations();
    });
});
```

---

## 3. Complete Examples

### Example 1: Modern Component with SWR

```typescript
import React, { useState, useCallback } from 'react';
import useSWR, { mutate } from 'swr';
import { Card, CardContent, CardHeader, CardTitle } from '@/shared/components/ui/card';
import { Button } from '@/shared/components/ui/button';
import { Avatar, AvatarFallback, AvatarImage } from '@/shared/components/ui/avatar';
import { toast } from 'sonner';
import { userApi } from '../api/userApi';
import type { User } from '../types';

interface UserProfileProps {
    userId: string;
    onUpdate?: () => void;
}

export const UserProfile: React.FC<UserProfileProps> = ({ userId, onUpdate }) => {
    const [isUpdating, setIsUpdating] = useState(false);

    const { data: user } = useSWR<User>(
        `user-${userId}`,
        () => userApi.getUser(userId),
        { suspense: true }
    );

    const handleSave = useCallback(async () => {
        setIsUpdating(true);
        try {
            await userApi.updateUser(userId, { /* data */ });
            await mutate(`user-${userId}`);
            toast.success('Profile updated');
            onUpdate?.();
        } catch (error) {
            toast.error('Failed to update profile');
        } finally {
            setIsUpdating(false);
        }
    }, [userId, onUpdate]);

    return (
        <Card className="max-w-xl mx-auto">
            <CardHeader>
                <div className="flex items-center gap-4">
                    <Avatar className="size-16">
                        <AvatarImage src={user?.avatarUrl} />
                        <AvatarFallback>{user?.firstName[0]}</AvatarFallback>
                    </Avatar>
                    <div>
                        <CardTitle>{user?.firstName} {user?.lastName}</CardTitle>
                        <p className="text-sm text-muted-foreground">{user?.email}</p>
                    </div>
                </div>
            </CardHeader>
            <CardContent>
                <Button onClick={handleSave} disabled={isUpdating}>
                    {isUpdating ? 'Saving...' : 'Save Changes'}
                </Button>
            </CardContent>
        </Card>
    );
};
```

### Example 2: Complete Feature Structure

> **For feature structure and API service patterns:** See [core-patterns.md](core-patterns.md) - API Service Pattern and Custom SWR Hooks sections
>
> **For file organization:** See [styling-routing.md](styling-routing.md) - File Organization section

### Example 3: App Router Page Pattern

```typescript
// app/(main)/users/[userId]/page.tsx
import type { Metadata } from 'next';
import { Suspense } from 'react';
import { UserProfileClient } from './UserProfileClient';
import { Skeleton } from '@/shared/components/ui/skeleton';

interface PageProps {
    params: Promise<{ userId: string }>;
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
    const { userId } = await params;
    return { title: `User ${userId}` };
}

export default async function UserProfilePage({ params }: PageProps) {
    const { userId } = await params;

    return (
        <Suspense fallback={<Skeleton className="h-64 w-full" />}>
            <UserProfileClient userId={userId} />
        </Suspense>
    );
}

// app/(main)/users/[userId]/UserProfileClient.tsx
'use client';

import useSWR from 'swr';
import { Card, CardHeader, CardTitle, CardContent } from '@/shared/components/ui/card';
import { userApi } from '@/features/users/api/userApi';

interface UserProfileClientProps {
    userId: string;
}

export function UserProfileClient({ userId }: UserProfileClientProps) {
    const { data: user } = useSWR(
        `user-${userId}`,
        () => userApi.getUser(userId),
        { suspense: true }
    );

    return (
        <Card className="max-w-md mx-auto">
            <CardHeader>
                <CardTitle>{user?.firstName} {user?.lastName}</CardTitle>
            </CardHeader>
            <CardContent>
                <p className="text-muted-foreground">{user?.email}</p>
            </CardContent>
        </Card>
    );
}
```

### Example 4: Form with Validation

```typescript
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { mutate } from 'swr';
import { Card, CardContent, CardHeader, CardTitle } from '@/shared/components/ui/card';
import { Input } from '@/shared/components/ui/input';
import { Label } from '@/shared/components/ui/label';
import { Button } from '@/shared/components/ui/button';
import { cn } from '@/shared/utils/cn';
import { toast } from 'sonner';
import { Loader2 } from 'lucide-react';
import { userApi } from '../api/userApi';

const schema = z.object({
    username: z.string().min(3, 'Min 3 characters').max(50),
    email: z.string().email('Invalid email'),
    firstName: z.string().min(1, 'Required'),
    lastName: z.string().min(1, 'Required'),
});

type FormData = z.infer<typeof schema>;

interface CreateUserFormProps {
    onSuccess?: () => void;
}

export const CreateUserForm: React.FC<CreateUserFormProps> = ({ onSuccess }) => {
    const [isPending, setIsPending] = useState(false);

    const {
        register,
        handleSubmit,
        formState: { errors },
        reset,
    } = useForm<FormData>({
        resolver: zodResolver(schema),
    });

    const onSubmit = async (data: FormData) => {
        setIsPending(true);
        try {
            await userApi.createUser(data);
            await mutate('users');
            toast.success('User created successfully');
            reset();
            onSuccess?.();
        } catch (error) {
            toast.error('Failed to create user');
        } finally {
            setIsPending(false);
        }
    };

    return (
        <Card className="max-w-lg">
            <CardHeader>
                <CardTitle>Create User</CardTitle>
            </CardHeader>
            <CardContent>
                <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
                    <div className="space-y-2">
                        <Label htmlFor="username">Username</Label>
                        <Input
                            id="username"
                            {...register('username')}
                            className={cn(errors.username && 'border-destructive')}
                        />
                        {errors.username && (
                            <p className="text-sm text-destructive">{errors.username.message}</p>
                        )}
                    </div>

                    <div className="space-y-2">
                        <Label htmlFor="email">Email</Label>
                        <Input
                            id="email"
                            type="email"
                            autoComplete="email"
                            {...register('email')}
                            className={cn(errors.email && 'border-destructive')}
                        />
                        {errors.email && (
                            <p className="text-sm text-destructive">{errors.email.message}</p>
                        )}
                    </div>

                    <div className="grid grid-cols-2 gap-4">
                        <div className="space-y-2">
                            <Label htmlFor="firstName">First Name</Label>
                            <Input id="firstName" {...register('firstName')} />
                            {errors.firstName && (
                                <p className="text-sm text-destructive">{errors.firstName.message}</p>
                            )}
                        </div>
                        <div className="space-y-2">
                            <Label htmlFor="lastName">Last Name</Label>
                            <Input id="lastName" {...register('lastName')} />
                            {errors.lastName && (
                                <p className="text-sm text-destructive">{errors.lastName.message}</p>
                            )}
                        </div>
                    </div>

                    <Button type="submit" className="w-full" aria-busy={isPending}>
                        {isPending ? (
                            <>
                                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                                Creating...
                            </>
                        ) : (
                            'Create User'
                        )}
                    </Button>
                </form>
            </CardContent>
        </Card>
    );
};
```

### Example 5: Dashboard with Parallel Fetching

```typescript
import React from 'react';
import useSWR from 'swr';
import { Card, CardContent, CardHeader, CardTitle } from '@/shared/components/ui/card';
import { SuspenseLoader } from '@/shared/components/SuspenseLoader';
import { statsApi } from '../api/statsApi';
import { userApi } from '../api/userApi';
import { activityApi } from '../api/activityApi';

const DashboardContent: React.FC = () => {
    // SWR automatically parallelizes these requests
    const { data: stats } = useSWR('stats', statsApi.getStats, { suspense: true });
    const { data: users } = useSWR('users-active', userApi.getActiveUsers, { suspense: true });
    const { data: activity } = useSWR('activity-recent', activityApi.getRecent, { suspense: true });

    return (
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 p-6">
            <Card>
                <CardHeader>
                    <CardTitle className="text-lg">Stats</CardTitle>
                </CardHeader>
                <CardContent>
                    <p className="text-2xl font-bold">{stats?.total}</p>
                    <p className="text-sm text-muted-foreground">Total items</p>
                </CardContent>
            </Card>

            <Card>
                <CardHeader>
                    <CardTitle className="text-lg">Active Users</CardTitle>
                </CardHeader>
                <CardContent>
                    <p className="text-2xl font-bold">{users?.length}</p>
                    <p className="text-sm text-muted-foreground">Online now</p>
                </CardContent>
            </Card>

            <Card>
                <CardHeader>
                    <CardTitle className="text-lg">Recent Activity</CardTitle>
                </CardHeader>
                <CardContent>
                    <p className="text-2xl font-bold">{activity?.length}</p>
                    <p className="text-sm text-muted-foreground">Events today</p>
                </CardContent>
            </Card>
        </div>
    );
};

export const Dashboard: React.FC = () => (
    <SuspenseLoader>
        <DashboardContent />
    </SuspenseLoader>
);
```

---

## Testing Checklist

- [ ] Component renders correctly
- [ ] User interactions work (click, type, submit)
- [ ] Loading states show correctly
- [ ] Error states handled
- [ ] Form validation works
- [ ] Accessibility (jest-axe)
- [ ] Hooks work in isolation
- [ ] Mocked API calls work
