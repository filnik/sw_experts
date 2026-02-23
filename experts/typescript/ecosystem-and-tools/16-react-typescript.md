# React + TypeScript

## Profile

### What It Is
React + TypeScript combines React's component model with TypeScript's type system for type-safe UI development. Typed props, hooks, context, events, and generic components prevent entire categories of bugs while providing excellent IDE support (auto-complete, refactoring).

### Origin and Evolution
- React (2013) — Component-based UI library
- PropTypes (runtime) → TypeScript (compile-time) type checking
- React 16.8 (2019) — Hooks (typed with generics)
- React 18 (2022) — Concurrent features, improved types
- React 19 (2024) — Server Components, Actions, use() hook
- Current: TypeScript is the default for React projects

### Key Authors and References
- **Matt Pocock** — React + TypeScript patterns
- **Dan Abramov** — React core team, hooks design
- **Kent C. Dodds** — Testing React components
- **react-typescript-cheatsheet** — Community resource

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Typed Props | `interface Props { name: string; onClick: () => void }` |
| Generic Components | `<List<T> items={T[]} renderItem={(item: T) => ReactNode}` |
| Typed Hooks | `useState<User>()`, `useReducer` with typed actions |
| Typed Context | `createContext<T>` with proper default handling |
| Event Types | `React.ChangeEvent<HTMLInputElement>`, `React.FormEvent` |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Props are the component's type contract** — Define clear prop interfaces. They document what the component needs and TypeScript enforces correct usage.
2. **Infer where possible** — `useState(0)` infers `number`. `useState<User | null>(null)` when the initial state doesn't match the final type.
3. **Generic components for reusable UI** — `<Select<T> options={T[]} />`, `<Table<T> data={T[]} columns={Column<T>[]} />` — generics make components adapt to any data type.
4. **Don't type React internals** — Don't annotate `return` types of components, `useEffect` callbacks, or event handler closures. Let TypeScript infer.
5. **Children typing** — Use `React.ReactNode` for children that render anything. Use `(item: T) => React.ReactNode` for render props.

### When to Use vs. When to Avoid

**TypeScript in React always** for production applications. The question is how strict:
- **Strict**: Library components, shared UI, complex state
- **Pragmatic**: Quick prototypes, internal tools (still use TS, but don't over-type)

---

## Frameworks and Methodologies

### 1. Component Typing — step-by-step
1. Define props interface (not type alias, for extendability)
2. Destructure props in function signature
3. Use children: `React.ReactNode` for content
4. Use generic type parameter for data-driven components
5. Export component and props type
6. Use `ComponentProps<typeof X>` to extract props from existing components

### 2. Hook Typing — step-by-step
1. `useState<T>` when initial value doesn't match final type
2. `useReducer` with discriminated union actions
3. Custom hooks with typed return values
4. `useRef<HTMLElement>(null)` for DOM refs
5. Context with proper null handling

---

## How to Apply

### Decision Checklist
- [ ] Are component props defined as interfaces?
- [ ] Are generic components used for reusable data-driven UI?
- [ ] Are hooks properly typed (useState, useReducer, useRef)?
- [ ] Is context typed with null-safe access?
- [ ] Are event handlers using React event types?

### Implementation Patterns

**Component Props:**
```typescript
interface ButtonProps {
  variant: "primary" | "secondary" | "danger";
  size?: "sm" | "md" | "lg";
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  loading?: boolean;
}

function Button({ variant, size = "md", children, onClick, disabled, loading }: ButtonProps) {
  return (
    <button
      className={cn(styles.button, styles[variant], styles[size])}
      onClick={onClick}
      disabled={disabled || loading}
      aria-busy={loading}
    >
      {loading ? <Spinner size={size} /> : children}
    </button>
  );
}

// Extending HTML element props
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

function Input({ label, error, ...inputProps }: InputProps) {
  return (
    <div>
      <label>{label}</label>
      <input {...inputProps} aria-invalid={!!error} />
      {error && <span className="error">{error}</span>}
    </div>
  );
}
```

**Generic Components:**
```typescript
// Generic List component
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
  emptyMessage?: string;
}

function List<T>({ items, renderItem, keyExtractor, emptyMessage }: ListProps<T>) {
  if (items.length === 0) return <p>{emptyMessage ?? "No items"}</p>;
  return (
    <ul>
      {items.map((item, i) => (
        <li key={keyExtractor(item)}>{renderItem(item, i)}</li>
      ))}
    </ul>
  );
}

// Usage — T is inferred from items
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}  // user is User
  keyExtractor={(user) => user.id}
/>

// Generic Select
interface SelectProps<T> {
  options: T[];
  value: T;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string;
}

function Select<T>({ options, value, onChange, getLabel, getValue }: SelectProps<T>) {
  // ...
}
```

**Typed Hooks:**
```typescript
// useState with explicit type
const [user, setUser] = useState<User | null>(null);

// useReducer with discriminated unions
type CounterAction =
  | { type: "increment"; amount: number }
  | { type: "decrement"; amount: number }
  | { type: "reset" };

function counterReducer(state: number, action: CounterAction): number {
  switch (action.type) {
    case "increment": return state + action.amount;
    case "decrement": return state - action.amount;
    case "reset": return 0;
  }
}

const [count, dispatch] = useReducer(counterReducer, 0);
dispatch({ type: "increment", amount: 5 }); // Type-safe

// useRef for DOM elements
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();

// Custom hook with typed return
function useAsync<T>(fn: () => Promise<T>) {
  const [state, setState] = useState<{
    data: T | null;
    error: Error | null;
    loading: boolean;
  }>({ data: null, error: null, loading: false });

  const execute = useCallback(async () => {
    setState({ data: null, error: null, loading: true });
    try {
      const data = await fn();
      setState({ data, error: null, loading: false });
    } catch (error) {
      setState({ data: null, error: error as Error, loading: false });
    }
  }, [fn]);

  return { ...state, execute };
}
```

**Typed Context:**
```typescript
interface AuthContextValue {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextValue | null>(null);

function useAuth(): AuthContextValue {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth must be used within AuthProvider");
  }
  return context;
}

function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const value: AuthContextValue = {
    user,
    login: async (creds) => { /* ... */ },
    logout: () => setUser(null),
    isAuthenticated: user !== null,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}
```

### Common Mistakes
1. **`React.FC`** — Don't use `React.FC`/`React.FunctionComponent`; just type props in the function parameter
2. **`as` for event types** — `e as any` in event handlers; use proper React event types
3. **`any` in useState** — `useState<any>()` defeats type safety; define the state type
4. **Not using generics** — Duplicating components for different data types
5. **Over-typing** — Annotating everything TypeScript can infer; let inference work

### Integration with Other Topics
- **TypeScript Type System** — Component props leverage unions, generics, utility types
- **Zod** — Form validation with react-hook-form + Zod
- **Testing** — Testing Library with typed render and queries
- **Design Systems** — Typed component library with consistent API
- **State Management** — Typed Zustand, Jotai, or Redux stores

---

## Resources

### Essential Reading
- react-typescript-cheatsheet (github.com/typescript-cheatsheets/react)
- React documentation — TypeScript section (react.dev)
- Matt Pocock — React + TypeScript tutorials

### Online Resources
- Total TypeScript — React patterns
- Kent C. Dodds — Epic React (typed patterns)
- React TypeScript Playground

### Practice Exercises
1. Build 5 typed components with proper prop interfaces
2. Create a generic Table component with typed columns and data
3. Implement typed context with null-safe useContext hook
4. Build a form with useReducer and discriminated union actions
5. Type a custom hook that returns generic async state
