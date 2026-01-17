# Advanced TypeScript Patterns

Senior-level TypeScript patterns for React applications: generics, type guards, utility types, and inference optimization.

---

## 1. Generic Components

### Basic Generic Component

```typescript
// ✅ Generic list component
interface ListProps<T> {
    items: T[];
    renderItem: (item: T, index: number) => React.ReactNode;
    keyExtractor: (item: T) => string;
    emptyMessage?: string;
}

export function List<T>({
    items,
    renderItem,
    keyExtractor,
    emptyMessage = 'No items',
}: ListProps<T>) {
    if (items.length === 0) {
        return <p className="text-muted-foreground">{emptyMessage}</p>;
    }

    return (
        <ul className="space-y-2">
            {items.map((item, index) => (
                <li key={keyExtractor(item)}>{renderItem(item, index)}</li>
            ))}
        </ul>
    );
}

// Usage - type is inferred
<List
    items={users}
    renderItem={(user) => <UserCard user={user} />}
    keyExtractor={(user) => user.id}
/>
```

### Generic with Constraints

```typescript
// Constraint: T must have id and name
interface HasIdAndName {
    id: string;
    name: string;
}

interface SelectProps<T extends HasIdAndName> {
    items: T[];
    value: T | null;
    onChange: (item: T) => void;
    placeholder?: string;
}

export function Select<T extends HasIdAndName>({
    items,
    value,
    onChange,
    placeholder = 'Select...',
}: SelectProps<T>) {
    return (
        <select
            value={value?.id ?? ''}
            onChange={(e) => {
                const item = items.find((i) => i.id === e.target.value);
                if (item) onChange(item);
            }}
        >
            <option value="">{placeholder}</option>
            {items.map((item) => (
                <option key={item.id} value={item.id}>
                    {item.name}
                </option>
            ))}
        </select>
    );
}
```

### Multiple Type Parameters

```typescript
// K = key type, V = value type
interface TableColumn<T, K extends keyof T> {
    key: K;
    header: string;
    render?: (value: T[K], row: T) => React.ReactNode;
}

interface TableProps<T> {
    data: T[];
    columns: TableColumn<T, keyof T>[];
    onRowClick?: (row: T) => void;
}

export function Table<T extends Record<string, unknown>>({
    data,
    columns,
    onRowClick,
}: TableProps<T>) {
    return (
        <table className="w-full">
            <thead>
                <tr>
                    {columns.map((col) => (
                        <th key={String(col.key)}>{col.header}</th>
                    ))}
                </tr>
            </thead>
            <tbody>
                {data.map((row, i) => (
                    <tr key={i} onClick={() => onRowClick?.(row)}>
                        {columns.map((col) => (
                            <td key={String(col.key)}>
                                {col.render
                                    ? col.render(row[col.key], row)
                                    : String(row[col.key])}
                            </td>
                        ))}
                    </tr>
                ))}
            </tbody>
        </table>
    );
}
```

---

## 2. Type Guards & Narrowing

### Custom Type Guards

```typescript
// Type guard function
interface SuccessResponse<T> {
    success: true;
    data: T;
}

interface ErrorResponse {
    success: false;
    error: string;
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

// Type guard: narrows to SuccessResponse
function isSuccess<T>(response: ApiResponse<T>): response is SuccessResponse<T> {
    return response.success === true;
}

// Usage
const handleResponse = <T>(response: ApiResponse<T>) => {
    if (isSuccess(response)) {
        // TypeScript knows: response.data exists
        console.log(response.data);
    } else {
        // TypeScript knows: response.error exists
        console.error(response.error);
    }
};
```

### Discriminated Unions

```typescript
// State machine with discriminated unions
type LoadingState = { status: 'loading' };
type SuccessState<T> = { status: 'success'; data: T };
type ErrorState = { status: 'error'; error: Error };

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// Exhaustive switch
const renderState = <T>(
    state: AsyncState<T>,
    render: (data: T) => React.ReactNode
): React.ReactNode => {
    switch (state.status) {
        case 'loading':
            return <Spinner />;
        case 'success':
            return render(state.data);
        case 'error':
            return <ErrorMessage error={state.error} />;
        default:
            // Exhaustive check - TypeScript error if case missed
            const _exhaustive: never = state;
            return _exhaustive;
    }
};
```

### Array Type Guards

```typescript
// Guard for non-empty array
function isNonEmpty<T>(arr: T[]): arr is [T, ...T[]] {
    return arr.length > 0;
}

// Guard for array of specific type
function isStringArray(arr: unknown[]): arr is string[] {
    return arr.every((item) => typeof item === 'string');
}

// Filter with type guard
const values: (string | null | undefined)[] = ['a', null, 'b', undefined];
const strings: string[] = values.filter((v): v is string => v != null);
```

### `in` Operator Narrowing

```typescript
interface Dog {
    bark: () => void;
    breed: string;
}

interface Cat {
    meow: () => void;
    color: string;
}

type Pet = Dog | Cat;

const handlePet = (pet: Pet) => {
    if ('bark' in pet) {
        // TypeScript knows: pet is Dog
        pet.bark();
        console.log(pet.breed);
    } else {
        // TypeScript knows: pet is Cat
        pet.meow();
        console.log(pet.color);
    }
};
```

---

## 3. Utility Types

### Built-in Utility Types

```typescript
interface User {
    id: string;
    name: string;
    email: string;
    password: string;
    createdAt: Date;
}

// Partial - all properties optional
type UpdateUserPayload = Partial<User>;
// { id?: string; name?: string; ... }

// Required - all properties required
type RequiredUser = Required<Partial<User>>;

// Pick - select specific properties
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: string; name: string }

// Omit - exclude specific properties
type PublicUser = Omit<User, 'password'>;
// { id: string; name: string; email: string; createdAt: Date }

// Readonly - all properties readonly
type ImmutableUser = Readonly<User>;

// Record - create object type
type UserRoles = Record<string, 'admin' | 'user' | 'guest'>;
// { [key: string]: 'admin' | 'user' | 'guest' }

// Extract - extract types from union
type StringOrNumber = string | number | boolean;
type OnlyStrings = Extract<StringOrNumber, string>;
// string

// Exclude - remove types from union
type NotBoolean = Exclude<StringOrNumber, boolean>;
// string | number

// NonNullable - remove null and undefined
type MaybeString = string | null | undefined;
type DefinitelyString = NonNullable<MaybeString>;
// string

// ReturnType - get return type of function
const getUser = () => ({ id: '1', name: 'John' });
type UserType = ReturnType<typeof getUser>;
// { id: string; name: string }

// Parameters - get parameter types
type GetUserParams = Parameters<typeof getUser>;
// []
```

### Custom Utility Types

```typescript
// DeepPartial - recursive partial
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// DeepReadonly - recursive readonly
type DeepReadonly<T> = {
    readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// Nullable - add null to all properties
type Nullable<T> = {
    [P in keyof T]: T[P] | null;
};

// RequiredKeys - make specific keys required
type RequiredKeys<T, K extends keyof T> = Omit<T, K> & Required<Pick<T, K>>;

// Usage
interface Config {
    name?: string;
    debug?: boolean;
    port?: number;
}

type ConfigWithRequiredPort = RequiredKeys<Config, 'port'>;
// { name?: string; debug?: boolean; port: number }

// PickByType - pick properties by value type
type PickByType<T, U> = {
    [P in keyof T as T[P] extends U ? P : never]: T[P];
};

type StringPropsOnly = PickByType<User, string>;
// { id: string; name: string; email: string; password: string }
```

---

## 4. Conditional Types

### Basic Conditional Types

```typescript
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Infer keyword - extract type from structure
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type FnReturn = GetReturnType<() => string>;  // string

// Extract array element type
type ArrayElement<T> = T extends (infer E)[] ? E : never;

type UserArrayElement = ArrayElement<User[]>;  // User

// Extract promise value
type Awaited<T> = T extends Promise<infer U> ? U : T;

type PromiseValue = Awaited<Promise<string>>;  // string
```

### Distributive Conditional Types

```typescript
// Distributes over union types
type ToArray<T> = T extends any ? T[] : never;

type StringOrNumberArray = ToArray<string | number>;
// string[] | number[]

// Prevent distribution with tuple
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type MixedArray = ToArrayNonDist<string | number>;
// (string | number)[]
```

---

## 5. Template Literal Types

### Basic Template Literals

```typescript
// String manipulation at type level
type EventName = 'click' | 'focus' | 'blur';
type HandlerName = `on${Capitalize<EventName>}`;
// 'onClick' | 'onFocus' | 'onBlur'

// CSS property patterns
type CSSSize = `${number}${'px' | 'rem' | 'em' | '%'}`;
const size: CSSSize = '16px';  // OK
const bad: CSSSize = '16';     // Error

// API endpoint patterns
type ApiEndpoint = `/api/${string}`;
const endpoint: ApiEndpoint = '/api/users';  // OK
```

### Advanced Template Patterns

```typescript
// Parse path parameters
type ExtractParams<T extends string> =
    T extends `${infer _Start}:${infer Param}/${infer Rest}`
        ? Param | ExtractParams<`/${Rest}`>
        : T extends `${infer _Start}:${infer Param}`
            ? Param
            : never;

type Params = ExtractParams<'/users/:userId/posts/:postId'>;
// 'userId' | 'postId'

// Type-safe event emitter
type EventMap = {
    click: { x: number; y: number };
    focus: { target: HTMLElement };
    submit: { data: FormData };
};

type EventHandler<K extends keyof EventMap> = (event: EventMap[K]) => void;

class TypedEmitter {
    private handlers = new Map<string, Function[]>();

    on<K extends keyof EventMap>(event: K, handler: EventHandler<K>) {
        const list = this.handlers.get(event) ?? [];
        list.push(handler);
        this.handlers.set(event, list);
    }

    emit<K extends keyof EventMap>(event: K, data: EventMap[K]) {
        this.handlers.get(event)?.forEach((fn) => fn(data));
    }
}
```

---

## 6. Mapped Types

### Basic Mapped Types

```typescript
// Transform all properties
type Getters<T> = {
    [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
    name: string;
    age: number;
}

type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

// Optional to required
type Concrete<T> = {
    [K in keyof T]-?: T[K];
};

// Remove readonly
type Mutable<T> = {
    -readonly [K in keyof T]: T[K];
};
```

### Key Remapping

```typescript
// Filter keys by value type
type FilterByValue<T, V> = {
    [K in keyof T as T[K] extends V ? K : never]: T[K];
};

interface Mixed {
    id: string;
    count: number;
    active: boolean;
    name: string;
}

type StringProps = FilterByValue<Mixed, string>;
// { id: string; name: string }

// Prefix all keys
type Prefixed<T, P extends string> = {
    [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};

type PrefixedPerson = Prefixed<Person, 'user'>;
// { userName: string; userAge: number }
```

---

## 7. as const & Const Assertions

### Literal Types with as const

```typescript
// Without as const
const routes = {
    home: '/',
    about: '/about',
    contact: '/contact',
};
type Routes = typeof routes;
// { home: string; about: string; contact: string }

// With as const
const routesConst = {
    home: '/',
    about: '/about',
    contact: '/contact',
} as const;
type RoutesConst = typeof routesConst;
// { readonly home: '/'; readonly about: '/about'; readonly contact: '/contact' }

// Extract values as union
type RouteValues = (typeof routesConst)[keyof typeof routesConst];
// '/' | '/about' | '/contact'
```

### Tuple Types

```typescript
// Without as const - inferred as array
const tuple = [1, 'hello', true];
type Tuple = typeof tuple;  // (string | number | boolean)[]

// With as const - inferred as tuple
const tupleConst = [1, 'hello', true] as const;
type TupleConst = typeof tupleConst;  // readonly [1, 'hello', true]

// Function returning tuple
function usePair<T>(initial: T) {
    const [value, setValue] = useState(initial);
    return [value, setValue] as const;
    // Returns: readonly [T, Dispatch<SetStateAction<T>>]
}
```

---

## 8. Type Inference Optimization

### Explicit vs Inferred Types

```typescript
// ✅ Let TypeScript infer when obvious
const users = [{ id: '1', name: 'John' }];  // Type is inferred
const count = users.length;  // number is inferred

// ✅ Explicit types for function signatures
const getUser = (id: string): User | undefined => {
    return users.find((u) => u.id === id);
};

// ✅ Explicit types for complex objects
const config: AppConfig = {
    apiUrl: 'https://api.example.com',
    timeout: 5000,
};

// ❌ Don't over-annotate
const name: string = 'John';  // Unnecessary
const numbers: number[] = [1, 2, 3];  // Unnecessary
```

### satisfies Operator (TS 4.9+)

```typescript
// Problem: as const loses helpful errors
const colors = {
    primary: '#ff0000',
    secondary: '#00ff00',
    typo: '#0000ff',  // Typo in key name - no error!
} as const;

// Solution: satisfies validates structure while preserving literal types
type ColorPalette = Record<'primary' | 'secondary' | 'accent', string>;

const colorsChecked = {
    primary: '#ff0000',
    secondary: '#00ff00',
    accent: '#0000ff',
} satisfies ColorPalette;
// Type: { primary: '#ff0000'; secondary: '#00ff00'; accent: '#0000ff' }

// Error: Object literal may only specify known properties
const colorsBad = {
    primary: '#ff0000',
    secondary: '#00ff00',
    typo: '#0000ff',  // Error!
} satisfies ColorPalette;
```

---

## 9. Function Overloads

### Overload Signatures

```typescript
// Multiple signatures for different use cases
function createElement(tag: 'input'): HTMLInputElement;
function createElement(tag: 'button'): HTMLButtonElement;
function createElement(tag: 'div'): HTMLDivElement;
function createElement(tag: string): HTMLElement;
function createElement(tag: string): HTMLElement {
    return document.createElement(tag);
}

const input = createElement('input');  // HTMLInputElement
const button = createElement('button');  // HTMLButtonElement
const div = createElement('div');  // HTMLDivElement
const span = createElement('span');  // HTMLElement

// Generic overloads
function fetch<T>(url: string): Promise<T>;
function fetch<T>(url: string, options: RequestInit): Promise<T>;
function fetch<T>(url: string, options?: RequestInit): Promise<T> {
    return window.fetch(url, options).then((r) => r.json());
}
```

---

## 10. Best Practices Summary

| Pattern | When to Use |
|---------|-------------|
| Generics | Reusable components that work with multiple types |
| Type Guards | Narrowing union types safely |
| Discriminated Unions | State machines, API responses |
| Utility Types | Transforming existing types |
| Conditional Types | Type-level logic |
| Template Literals | String pattern types |
| as const | Preserving literal types |
| satisfies | Type checking + literal preservation |
| Function Overloads | Multiple call signatures |

### Anti-Patterns to Avoid

```typescript
// ❌ any
const data: any = fetchData();

// ✅ unknown + type guard
const data: unknown = fetchData();
if (isValidData(data)) {
    // use data
}

// ❌ Type assertions without validation
const user = data as User;

// ✅ Runtime validation
const user = userSchema.parse(data);

// ❌ Non-null assertion overuse
const name = user!.name!;

// ✅ Proper null checks
const name = user?.name ?? 'Unknown';

// ❌ Overly complex types
type Nightmare<T, U, V> = T extends U ? V extends T ? ... : ... : ...;

// ✅ Simple, readable types
type SimpleResult<T> = { success: true; data: T } | { success: false; error: string };
```
