# Styling, Routing & File Organization

Tailwind CSS patterns, Next.js App Router, and feature-based architecture.

---

## 1. Tailwind CSS

### Critical Rule: Never Hardcode Colors

```typescript
// ❌ WRONG - Hardcoded hex
<div className="bg-[#fe4601]">
<span className="text-[#ffaca9]">

// ✅ CORRECT - CSS variables from globals.css
<div className="bg-orange-1">
<span className="text-peach-1">
```

### Color Palettes (from globals.css)

| Palette | Variables | Usage |
|---------|-----------|-------|
| **Orange** | `orange-1` → `orange-3` | Primary brand, CTAs |
| **Peach** | `peach-1` → `peach-8` | Accents, highlights |
| **White** | `white-0` → `white-11` | Opacity variants (100% → 5%) |
| **Black** | `black-0` → `black-3` | Dark overlays |
| **Green** | `green-1` → `green-7` | Success states |
| **Red** | `red-1`, `red-2` | Error states |
| **Grey** | `grey-1` → `grey-4` | Neutral, borders |
| **Yellow** | `yellow-1` → `yellow-3` | Warnings |
| **Purple** | `purple-1` → `purple-15` | Gradients, accents |

### Semantic Colors (shadcn)

| Variable | Usage |
|----------|-------|
| `bg-background` / `text-foreground` | Base colors |
| `bg-primary` / `text-primary-foreground` | Brand colors |
| `bg-muted` / `text-muted-foreground` | Subtle text |
| `bg-destructive` / `text-destructive` | Error states |
| `bg-card` / `border` | Card styling |

### cn() Utility

```typescript
import { cn } from '@/shared/utils/cn';

// Basic conditional
<div className={cn("p-4", isActive && "bg-primary")}>

// Multiple conditions
<div className={cn(
    "flex items-center gap-2 rounded-md p-4",
    isActive && "bg-primary text-primary-foreground",
    isDisabled && "opacity-50 pointer-events-none"
)}>

// Parent override pattern
interface CardProps {
    className?: string;
    children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ className, children }) => (
    <div className={cn("rounded-lg border bg-card p-4", className)}>
        {children}
    </div>
);

// Usage - parent can override
<Card className="p-6 bg-muted">Custom styling</Card>
```

### CVA for Variants

```typescript
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva(
    "inline-flex items-center justify-center rounded-md font-medium transition-colors",
    {
        variants: {
            variant: {
                default: "bg-primary text-primary-foreground hover:bg-primary/90",
                destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
                outline: "border border-input bg-background hover:bg-accent",
                ghost: "hover:bg-accent hover:text-accent-foreground",
            },
            size: {
                sm: "h-8 px-3 text-sm",
                default: "h-10 px-4",
                lg: "h-11 px-8 text-lg",
                icon: "size-10",
            },
        },
        defaultVariants: {
            variant: "default",
            size: "default",
        },
    }
);

interface ButtonProps extends VariantProps<typeof buttonVariants> {
    className?: string;
    children: React.ReactNode;
}

export const Button: React.FC<ButtonProps> = ({ variant, size, className, children }) => (
    <button className={cn(buttonVariants({ variant, size }), className)}>
        {children}
    </button>
);
```

### Common Patterns

```typescript
// Flex layouts
<div className="flex items-center gap-2">           // Row, centered
<div className="flex flex-col gap-4">               // Column
<div className="flex items-center justify-between"> // Space between

// Grid layouts
<div className="grid grid-cols-1 md:grid-cols-2 gap-4">
<div className="grid grid-cols-3 gap-6">

// Card
<div className="rounded-lg border bg-card p-4">
<div className="rounded-lg border bg-card p-4 shadow-sm">

// Text styles
<h1 className="text-3xl font-bold">
<p className="text-sm text-muted-foreground">
<p className="truncate">  // Single line ellipsis
<p className="line-clamp-2">  // Multi-line clamp

// Interactive
<div className="hover:bg-accent transition-colors">
<button className="disabled:pointer-events-none disabled:opacity-50">

// Responsive
<div className="hidden md:block">   // Hidden on mobile
<div className="md:hidden">         // Hidden on desktop
<div className="p-4 md:p-6">        // Responsive padding
```

### Spacing Scale

| Class | Value | CSS Variable |
|-------|-------|--------------|
| `gap-1` | 4px | - |
| `gap-2` | 8px | `spacing-inline` |
| `gap-3` | 12px | `spacing-element` |
| `gap-4` | 16px | `spacing-group` |
| `gap-6` | 24px | `spacing-card` |
| `gap-8` | 32px | `spacing-section` |

### Icon Sizing

```typescript
<Icon className="size-4" />   // 16px
<Icon className="size-5" />   // 20px
<Icon className="size-6" />   // 24px
```

---

## 2. Next.js App Router

### Route Groups

| Group | Layout | Use Case |
|-------|--------|----------|
| `(dashboard)` | No LandingLayout | Studio, Projects, Effects |
| `(main)` | With LandingLayout | Account, Profile, Pricing, Blog |
| `(auth)` | Auth-specific | Sign-up, SSO callback |
| `(legal)` | Minimal | Terms, Privacy, Cookie |
| `(payment)` | Payment flow | Checkout |

### Directory Structure

```
app/
├── (dashboard)/
│   ├── layout.tsx           # Dashboard layout (no LandingLayout)
│   ├── studio/
│   │   ├── page.tsx         # Server component
│   │   └── StudioClient.tsx # Client component
│   └── projects/
│       └── page.tsx
├── (main)/
│   ├── layout.tsx           # With LandingLayout
│   ├── profile/
│   │   └── [username]/
│   │       └── page.tsx
│   └── pricing/
│       └── page.tsx
├── api/
│   └── users/
│       └── route.ts
└── layout.tsx               # Root layout
```

### Page Pattern (Server + Client)

```typescript
// app/(main)/users/[userId]/page.tsx (Server Component)
import type { Metadata } from 'next';
import { Suspense } from 'react';
import { UserProfileClient } from './UserProfileClient';

// Next.js 15: params is Promise
interface PageProps {
    params: Promise<{ userId: string }>;
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
    const { userId } = await params;
    return { title: `User ${userId}` };
}

export default async function UserPage({ params }: PageProps) {
    const { userId } = await params;

    return (
        <Suspense fallback={<Skeleton />}>
            <UserProfileClient userId={userId} />
        </Suspense>
    );
}

// app/(main)/users/[userId]/UserProfileClient.tsx (Client Component)
'use client';

import useSWR from 'swr';
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

    return <div>{user?.name}</div>;
}
```

### Server vs Client Components

```typescript
// Server Component (default)
// - Can be async
// - Can fetch data directly
// - No useState, useEffect, event handlers
// - Smaller bundle size

async function ServerComponent() {
    const data = await fetchData();
    return <div>{data.title}</div>;
}

// Client Component
// - Must have 'use client' directive
// - Can use hooks and event handlers
// - Interactive

'use client';

function ClientComponent() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### Navigation

```typescript
// ❌ WRONG - Old Pages Router
import { useRouter } from 'next/router';

// ✅ CORRECT - App Router
import { useRouter, usePathname, useSearchParams } from 'next/navigation';

function Navigation() {
    const router = useRouter();
    const pathname = usePathname();
    const searchParams = useSearchParams();

    const handleNavigate = () => {
        router.push('/dashboard');
        // or
        router.replace('/login');
    };

    return (
        <nav>
            <Link href="/home" className={pathname === '/home' ? 'active' : ''}>
                Home
            </Link>
        </nav>
    );
}
```

### Dynamic Routes

```typescript
// app/users/[userId]/page.tsx
interface PageProps {
    params: Promise<{ userId: string }>;
}

// app/blog/[...slug]/page.tsx (catch-all)
interface PageProps {
    params: Promise<{ slug: string[] }>;
}

// app/[[...slug]]/page.tsx (optional catch-all)
interface PageProps {
    params: Promise<{ slug?: string[] }>;
}
```

### Loading and Error States

```typescript
// app/(dashboard)/studio/loading.tsx
export default function Loading() {
    return <Skeleton className="h-96" />;
}

// app/(dashboard)/studio/error.tsx
'use client';

export default function Error({
    error,
    reset,
}: {
    error: Error;
    reset: () => void;
}) {
    return (
        <div>
            <h2>Something went wrong!</h2>
            <button onClick={reset}>Try again</button>
        </div>
    );
}
```

---

## 3. File Organization

### Feature-Based Architecture

```
src/
├── features/                    # Domain-specific modules
│   ├── auth/
│   │   ├── api/                 # API services
│   │   │   └── authApi.ts
│   │   ├── components/          # Feature components
│   │   │   ├── LoginForm.tsx
│   │   │   └── AuthModal.tsx
│   │   ├── hooks/               # Feature hooks
│   │   │   └── useAuth.ts
│   │   ├── types/               # TypeScript types
│   │   │   └── index.ts
│   │   └── index.ts             # Public exports
│   ├── users/
│   ├── studio/
│   └── media-gallery/
├── shared/                      # Shared code
│   ├── components/
│   │   ├── ui/                  # shadcn/ui components
│   │   ├── SuspenseLoader.tsx
│   │   └── LoadingOverlay.tsx
│   ├── hooks/
│   │   └── useDebouncedValue.ts
│   └── utils/
│       └── cn.ts
├── lib/                         # Third-party configs
│   └── apiClient.ts
└── types/                       # Global types
    └── index.ts
```

### Feature Directory Structure

```
src/features/users/
├── api/
│   └── userApi.ts              # API service
├── components/
│   ├── UserCard.tsx
│   ├── UserList.tsx
│   └── UserProfile/
│       ├── UserProfile.tsx
│       ├── UserAvatar.tsx
│       └── index.ts
├── hooks/
│   ├── useUser.ts
│   └── useUsers.ts
├── types/
│   └── index.ts
└── index.ts                    # Public exports only
```

### Public Exports (index.ts)

```typescript
// features/users/index.ts
// Only export what other features need

export { UserCard } from './components/UserCard';
export { UserProfile } from './components/UserProfile';
export { useUser, useUsers } from './hooks/useUser';
export { userApi } from './api/userApi';
export type { User, CreateUserPayload } from './types';

// Internal components NOT exported
// - UserAvatar (internal to UserProfile)
// - Helper functions
```

### Import Aliases

| Alias | Resolves To | Example |
|-------|-------------|---------|
| `@/` | `src/` | `import { apiClient } from '@/lib/apiClient'` |
| `@/shared` | `src/shared/` | `import { cn } from '@/shared/utils/cn'` |
| `@/features` | `src/features/` | `import { authApi } from '@/features/auth'` |
| `~types` | `src/types/` | `import type { User } from '~types'` |

### Import Order

```typescript
// 1. React/Next.js
import React, { useState, useCallback } from 'react';
import { useRouter } from 'next/navigation';
import dynamic from 'next/dynamic';

// 2. Third-party libraries
import useSWR from 'swr';
import { z } from 'zod';

// 3. Shared components and utilities
import { Button } from '@/shared/components/ui/button';
import { cn } from '@/shared/utils/cn';

// 4. Feature imports
import { userApi } from '@/features/users';

// 5. Local imports (relative)
import { UserCard } from './UserCard';
import type { UserCardProps } from './types';
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase with `use` prefix | `useUser.ts` |
| Utils | camelCase | `formatDate.ts` |
| Types | `.types.ts` or `types/` | `user.types.ts` |
| API Services | `{name}Api.ts` | `userApi.ts` |
| Stores | `{name}Store.ts` | `uiStore.ts` |

---

## Quick Reference

### Styling

```typescript
// Card
className="rounded-lg border bg-card p-4"

// Muted text
className="text-sm text-muted-foreground"

// Primary button
className="bg-primary text-primary-foreground hover:bg-primary/90"

// Flex row centered
className="flex items-center gap-2"

// Responsive grid
className="grid grid-cols-1 md:grid-cols-2 gap-4"
```

### Routing

```typescript
// Navigation
import { useRouter, usePathname } from 'next/navigation';

// Link
import Link from 'next/link';
<Link href="/dashboard">Dashboard</Link>

// Dynamic route params (Next.js 15)
const { userId } = await params;  // params is Promise
```

### File Organization

```
features/{name}/
├── api/         # API services
├── components/  # Components
├── hooks/       # Hooks
├── types/       # Types
└── index.ts     # Public exports
```
