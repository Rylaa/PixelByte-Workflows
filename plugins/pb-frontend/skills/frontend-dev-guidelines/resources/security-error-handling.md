# Security & Error Handling

Frontend security patterns, error boundaries, and resilience strategies for production applications.

---

## 1. XSS Prevention

### Never Use dangerouslySetInnerHTML

```typescript
// ❌ DANGEROUS - XSS vulnerability
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ SAFE - React auto-escapes
<div>{userInput}</div>

// ✅ If HTML is required, sanitize first
import DOMPurify from 'dompurify';

const SafeHTML: React.FC<{ html: string }> = ({ html }) => {
    const sanitized = DOMPurify.sanitize(html, {
        ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
        ALLOWED_ATTR: ['href', 'target', 'rel'],
    });

    return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
};
```

### URL Sanitization

```typescript
// ❌ DANGEROUS - javascript: protocol XSS
<a href={userProvidedUrl}>Click</a>

// ✅ SAFE - Validate URL protocol
const isSafeUrl = (url: string): boolean => {
    try {
        const parsed = new URL(url);
        return ['http:', 'https:', 'mailto:'].includes(parsed.protocol);
    } catch {
        return false;
    }
};

const SafeLink: React.FC<{ href: string; children: React.ReactNode }> = ({
    href,
    children
}) => {
    if (!isSafeUrl(href)) {
        return <span>{children}</span>;
    }

    return (
        <a
            href={href}
            target="_blank"
            rel="noopener noreferrer"
        >
            {children}
        </a>
    );
};
```

### Event Handler Injection

```typescript
// ❌ DANGEROUS - Event handler from user data
<div onClick={window[userInput]} />

// ✅ SAFE - Use predefined handlers
const handlers = {
    submit: () => handleSubmit(),
    cancel: () => handleCancel(),
} as const;

type HandlerKey = keyof typeof handlers;

const SafeButton: React.FC<{ action: string }> = ({ action }) => {
    const handler = handlers[action as HandlerKey];
    if (!handler) return null;

    return <button onClick={handler}>{action}</button>;
};
```

---

## 2. CSRF Protection

### Token-Based Protection

```typescript
// API client with CSRF token
import { api } from '@/shared/lib/api-fetcher';

// Token is automatically included via cookies or headers
// Backend should validate X-CSRF-Token header

export const submitForm = async (data: FormData) => {
    // api-fetcher automatically handles CSRF tokens
    return api.post('/submit', data);
};
```

### SameSite Cookie Configuration

```typescript
// Next.js API route cookie settings
import { cookies } from 'next/headers';

export async function POST(request: Request) {
    const cookieStore = cookies();

    cookieStore.set('session', token, {
        httpOnly: true,           // No JS access
        secure: true,             // HTTPS only
        sameSite: 'strict',       // CSRF protection
        path: '/',
        maxAge: 60 * 60 * 24 * 7, // 1 week
    });
}
```

---

## 3. Content Security Policy (CSP)

### Next.js CSP Configuration

```typescript
// next.config.js
const cspHeader = `
    default-src 'self';
    script-src 'self' 'unsafe-eval' 'unsafe-inline';
    style-src 'self' 'unsafe-inline';
    img-src 'self' blob: data: https:;
    font-src 'self';
    connect-src 'self' https://api.example.com;
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self';
`;

module.exports = {
    async headers() {
        return [
            {
                source: '/(.*)',
                headers: [
                    {
                        key: 'Content-Security-Policy',
                        value: cspHeader.replace(/\n/g, ''),
                    },
                ],
            },
        ];
    },
};
```

### Nonce-Based Script Loading

```typescript
// For inline scripts with CSP
import { headers } from 'next/headers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
    const nonce = headers().get('x-nonce') || '';

    return (
        <html>
            <head>
                <script nonce={nonce}>
                    {/* Inline script content */}
                </script>
            </head>
            <body>{children}</body>
        </html>
    );
}
```

---

## 4. Input Validation & Sanitization

### Zod Schema Validation

```typescript
import { z } from 'zod';

// ✅ Strict input validation
const userInputSchema = z.object({
    email: z.string().email().max(255),
    username: z.string()
        .min(3)
        .max(30)
        .regex(/^[a-zA-Z0-9_]+$/, 'Only alphanumeric and underscore'),
    bio: z.string().max(500).optional(),
    website: z.string().url().optional().or(z.literal('')),
});

// Validate before processing
const validateInput = (data: unknown) => {
    const result = userInputSchema.safeParse(data);

    if (!result.success) {
        throw new ValidationError(result.error.flatten());
    }

    return result.data;
};
```

### File Upload Validation

```typescript
const ALLOWED_MIME_TYPES = [
    'image/jpeg',
    'image/png',
    'image/webp',
    'image/gif',
] as const;

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

const validateFile = (file: File): { valid: boolean; error?: string } => {
    // Check MIME type
    if (!ALLOWED_MIME_TYPES.includes(file.type as typeof ALLOWED_MIME_TYPES[number])) {
        return { valid: false, error: 'Invalid file type' };
    }

    // Check file size
    if (file.size > MAX_FILE_SIZE) {
        return { valid: false, error: 'File too large (max 5MB)' };
    }

    // Check file extension matches MIME
    const extension = file.name.split('.').pop()?.toLowerCase();
    const validExtensions = ['jpg', 'jpeg', 'png', 'webp', 'gif'];

    if (!extension || !validExtensions.includes(extension)) {
        return { valid: false, error: 'Invalid file extension' };
    }

    return { valid: true };
};
```

---

## 5. Secure Data Handling

### Sensitive Data in State

```typescript
// ❌ DANGEROUS - Sensitive data in Redux/Zustand (persisted)
const useStore = create(
    persist(
        (set) => ({
            user: null,
            creditCard: null, // ❌ Never persist!
        }),
        { name: 'app-storage' }
    )
);

// ✅ SAFE - Sensitive data in session only
const useAuthStore = create<AuthState>((set) => ({
    user: null,
    // Credit card info should never be stored client-side
}));

// For temporary sensitive data, use ref or state without persistence
const useSensitiveData = () => {
    const [cardNumber, setCardNumber] = useState('');

    // Clear on unmount
    useEffect(() => {
        return () => setCardNumber('');
    }, []);

    return { cardNumber, setCardNumber };
};
```

### API Key Protection

```typescript
// ❌ DANGEROUS - API key in client code
const API_KEY = 'sk_live_xxx'; // Exposed in bundle!

// ✅ SAFE - Use environment variables (server-side only)
// .env.local
NEXT_PUBLIC_API_URL=https://api.example.com  // OK - public
API_SECRET_KEY=sk_live_xxx                    // Server-only

// ✅ SAFE - Call through API route
// app/api/external/route.ts
export async function GET() {
    const response = await fetch('https://external-api.com', {
        headers: {
            Authorization: `Bearer ${process.env.API_SECRET_KEY}`,
        },
    });
    return Response.json(await response.json());
}
```

---

## 6. Error Boundaries

### Granular Error Boundaries

```typescript
import { Component, ErrorInfo, ReactNode } from 'react';
import * as Sentry from '@sentry/nextjs';

interface Props {
    children: ReactNode;
    fallback?: ReactNode;
    onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
    hasError: boolean;
    error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
    constructor(props: Props) {
        super(props);
        this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error: Error): State {
        return { hasError: true, error };
    }

    componentDidCatch(error: Error, errorInfo: ErrorInfo) {
        // Log to Sentry
        Sentry.captureException(error, {
            extra: { componentStack: errorInfo.componentStack },
        });

        // Custom handler
        this.props.onError?.(error, errorInfo);
    }

    render() {
        if (this.state.hasError) {
            return this.props.fallback || (
                <div className="p-8 text-center">
                    <h2 className="text-xl font-semibold text-destructive">
                        Something went wrong
                    </h2>
                    <button
                        onClick={() => this.setState({ hasError: false, error: null })}
                        className="mt-4 px-4 py-2 bg-primary text-primary-foreground rounded"
                    >
                        Try Again
                    </button>
                </div>
            );
        }

        return this.props.children;
    }
}
```

### Feature-Level Boundaries

```typescript
// Wrap each major feature independently
export const Dashboard: React.FC = () => {
    return (
        <div className="grid grid-cols-3 gap-4">
            <ErrorBoundary fallback={<WidgetError name="Stats" />}>
                <StatsWidget />
            </ErrorBoundary>

            <ErrorBoundary fallback={<WidgetError name="Chart" />}>
                <ChartWidget />
            </ErrorBoundary>

            <ErrorBoundary fallback={<WidgetError name="Activity" />}>
                <ActivityWidget />
            </ErrorBoundary>
        </div>
    );
};

const WidgetError: React.FC<{ name: string }> = ({ name }) => (
    <div className="p-4 border border-destructive/20 rounded bg-destructive/5">
        <p className="text-sm text-muted-foreground">
            Failed to load {name}
        </p>
    </div>
);
```

---

## 7. Resilience Patterns

### Retry with Exponential Backoff

```typescript
interface RetryOptions {
    maxRetries?: number;
    baseDelay?: number;
    maxDelay?: number;
}

async function withRetry<T>(
    fn: () => Promise<T>,
    options: RetryOptions = {}
): Promise<T> {
    const { maxRetries = 3, baseDelay = 1000, maxDelay = 10000 } = options;

    let lastError: Error;

    for (let attempt = 0; attempt <= maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            lastError = error as Error;

            if (attempt === maxRetries) break;

            // Exponential backoff with jitter
            const delay = Math.min(
                baseDelay * Math.pow(2, attempt) + Math.random() * 1000,
                maxDelay
            );

            await new Promise((resolve) => setTimeout(resolve, delay));
        }
    }

    throw lastError!;
}

// Usage
const fetchWithRetry = () => withRetry(
    () => api.get('/data'),
    { maxRetries: 3, baseDelay: 1000 }
);
```

### Circuit Breaker Pattern

```typescript
type CircuitState = 'closed' | 'open' | 'half-open';

class CircuitBreaker {
    private state: CircuitState = 'closed';
    private failureCount = 0;
    private lastFailureTime = 0;

    constructor(
        private readonly threshold: number = 5,
        private readonly timeout: number = 30000
    ) {}

    async execute<T>(fn: () => Promise<T>): Promise<T> {
        if (this.state === 'open') {
            if (Date.now() - this.lastFailureTime > this.timeout) {
                this.state = 'half-open';
            } else {
                throw new Error('Circuit breaker is open');
            }
        }

        try {
            const result = await fn();
            this.onSuccess();
            return result;
        } catch (error) {
            this.onFailure();
            throw error;
        }
    }

    private onSuccess() {
        this.failureCount = 0;
        this.state = 'closed';
    }

    private onFailure() {
        this.failureCount++;
        this.lastFailureTime = Date.now();

        if (this.failureCount >= this.threshold) {
            this.state = 'open';
        }
    }
}

// Usage
const apiCircuitBreaker = new CircuitBreaker(5, 30000);

const fetchData = () => apiCircuitBreaker.execute(() => api.get('/data'));
```

### Graceful Degradation

```typescript
// Feature that degrades gracefully
const AdvancedFeature: React.FC = () => {
    const { data, error } = useSWR('feature-data', fetcher, { suspense: false });

    // If feature fails, show basic version
    if (error) {
        return <BasicFeature />;
    }

    // If loading, show skeleton
    if (!data) {
        return <FeatureSkeleton />;
    }

    return <FullFeature data={data} />;
};

// Network-aware degradation
const useNetworkAware = () => {
    const [isOnline, setIsOnline] = useState(true);
    const [connectionType, setConnectionType] = useState<string>('unknown');

    useEffect(() => {
        const updateStatus = () => {
            setIsOnline(navigator.onLine);

            const connection = (navigator as any).connection;
            if (connection) {
                setConnectionType(connection.effectiveType);
            }
        };

        window.addEventListener('online', updateStatus);
        window.addEventListener('offline', updateStatus);

        updateStatus();

        return () => {
            window.removeEventListener('online', updateStatus);
            window.removeEventListener('offline', updateStatus);
        };
    }, []);

    const shouldReduceData = connectionType === '2g' || connectionType === 'slow-2g';

    return { isOnline, connectionType, shouldReduceData };
};
```

---

## 8. Request Cancellation

### AbortController Pattern

```typescript
// Cancel requests on unmount or navigation
const useDataWithCancellation = (url: string) => {
    const [data, setData] = useState(null);
    const [error, setError] = useState<Error | null>(null);

    useEffect(() => {
        const abortController = new AbortController();

        const fetchData = async () => {
            try {
                const response = await fetch(url, {
                    signal: abortController.signal,
                });

                if (!response.ok) throw new Error('Failed to fetch');

                const result = await response.json();
                setData(result);
            } catch (err) {
                if ((err as Error).name !== 'AbortError') {
                    setError(err as Error);
                }
            }
        };

        fetchData();

        return () => abortController.abort();
    }, [url]);

    return { data, error };
};
```

### SWR with Cancellation

```typescript
import useSWR from 'swr';

const fetcher = async (url: string, { signal }: { signal?: AbortSignal }) => {
    const response = await fetch(url, { signal });
    if (!response.ok) throw new Error('Failed to fetch');
    return response.json();
};

// SWR automatically handles cancellation on key change
const { data } = useSWR(
    shouldFetch ? '/api/data' : null,
    fetcher
);
```

---

## 9. Error Logging & Monitoring

### Sentry Integration

```typescript
import * as Sentry from '@sentry/nextjs';

// Capture with context
export const captureError = (
    error: Error,
    context?: Record<string, unknown>
) => {
    Sentry.captureException(error, {
        extra: context,
        tags: {
            component: context?.component as string,
            action: context?.action as string,
        },
    });
};

// Usage in catch blocks
try {
    await submitForm(data);
} catch (error) {
    captureError(error as Error, {
        component: 'ContactForm',
        action: 'submit',
        formData: { email: data.email }, // Don't log sensitive data
    });

    toast.error('Failed to submit form');
}
```

### User-Friendly Error Messages

```typescript
// Error message mapping
const ERROR_MESSAGES: Record<string, string> = {
    NETWORK_ERROR: 'Unable to connect. Please check your internet connection.',
    UNAUTHORIZED: 'Your session has expired. Please log in again.',
    FORBIDDEN: "You don't have permission to perform this action.",
    NOT_FOUND: 'The requested resource was not found.',
    RATE_LIMITED: 'Too many requests. Please try again later.',
    SERVER_ERROR: 'Something went wrong on our end. Please try again.',
    VALIDATION_ERROR: 'Please check your input and try again.',
    DEFAULT: 'An unexpected error occurred. Please try again.',
};

const getErrorMessage = (error: unknown): string => {
    if (error instanceof Error) {
        // Check for known error codes
        const code = (error as any).code;
        if (code && ERROR_MESSAGES[code]) {
            return ERROR_MESSAGES[code];
        }

        // Check HTTP status
        const status = (error as any).status;
        if (status === 401) return ERROR_MESSAGES.UNAUTHORIZED;
        if (status === 403) return ERROR_MESSAGES.FORBIDDEN;
        if (status === 404) return ERROR_MESSAGES.NOT_FOUND;
        if (status === 429) return ERROR_MESSAGES.RATE_LIMITED;
        if (status >= 500) return ERROR_MESSAGES.SERVER_ERROR;
    }

    return ERROR_MESSAGES.DEFAULT;
};
```

---

## Summary

| Category | Key Patterns |
|----------|--------------|
| XSS Prevention | Never `dangerouslySetInnerHTML`, sanitize URLs, validate event handlers |
| CSRF | SameSite cookies, token-based protection |
| CSP | Strict policy, nonce for inline scripts |
| Input Validation | Zod schemas, file upload validation |
| Error Boundaries | Granular per-feature, graceful fallbacks |
| Resilience | Retry with backoff, circuit breaker, graceful degradation |
| Request Handling | AbortController, proper cleanup |
| Monitoring | Sentry integration, user-friendly messages |
