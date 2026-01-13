# React Patterns

## Component Patterns

### Props with TypeScript
```typescript
type ButtonProps = {
  readonly variant?: 'primary' | 'secondary' | 'ghost';
  readonly size?: 'sm' | 'md' | 'lg';
  readonly isLoading?: boolean;
  readonly children: React.ReactNode;
  readonly onClick?: () => void;
  readonly className?: string;
};

export function Button({ 
  variant = 'primary', 
  size = 'md', 
  isLoading, 
  children, 
  onClick,
  className 
}: ButtonProps) {
  return (
    <button 
      className={cn(buttonVariants({ variant, size }), className)}
      onClick={onClick}
      disabled={isLoading}
    >
      {isLoading ? <Spinner /> : children}
    </button>
  );
}
```

### Compound Components
```typescript
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (tab: string) => void;
} | null>(null);

function Tabs({ children, defaultTab }: { children: React.ReactNode; defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabsTrigger({ value, children }: { value: string; children: React.ReactNode }) {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error('TabsTrigger must be within Tabs');
  
  return (
    <button
      className={cn('tab', ctx.activeTab === value && 'active')}
      onClick={() => ctx.setActiveTab(value)}
    >
      {children}
    </button>
  );
}

Tabs.Trigger = TabsTrigger;
Tabs.Content = TabsContent;
Tabs.List = TabsList;
```

### Render Props / Children as Function
```typescript
type DataLoaderProps<T> = {
  url: string;
  children: (data: T, isLoading: boolean) => React.ReactNode;
};

function DataLoader<T>({ url, children }: DataLoaderProps<T>) {
  const { data, isLoading } = useQuery({ queryKey: [url], queryFn: () => fetch(url) });
  return <>{children(data as T, isLoading)}</>;
}
```

## Custom Hooks

### Data Fetching Hook
```typescript
export function useItems(filters?: ItemFilters) {
  return useQuery({
    queryKey: ['items', filters],
    queryFn: () => fetchItems(filters),
    staleTime: 1000 * 60 * 5,
    gcTime: 1000 * 60 * 30,
    refetchOnWindowFocus: false,
    retry: (count, error) => error?.status !== 404 && count < 3,
  });
}
```

### Debounce Hook
```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debounced;
}
```

### Local Storage Hook
```typescript
export function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);
  
  return [value, setValue] as const;
}
```

### Media Query Hook
```typescript
export function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);
    
    const listener = (e: MediaQueryListEvent) => setMatches(e.matches);
    media.addEventListener('change', listener);
    return () => media.removeEventListener('change', listener);
  }, [query]);
  
  return matches;
}
```

## State Management

### useState for Simple State
```typescript
function SearchableList({ items }: { items: Item[] }) {
  const [search, setSearch] = useState('');
  
  const filtered = useMemo(
    () => items.filter(item => item.name.toLowerCase().includes(search.toLowerCase())),
    [items, search]
  );
  
  return (
    <>
      <Input value={search} onChange={e => setSearch(e.target.value)} />
      <ItemList items={filtered} />
    </>
  );
}
```

### useReducer for Complex State
```typescript
type State = { items: Item[]; loading: boolean; error: string | null };

type Action =
  | { type: 'LOADING' }
  | { type: 'SUCCESS'; payload: Item[] }
  | { type: 'ERROR'; payload: string }
  | { type: 'ADD'; payload: Item }
  | { type: 'UPDATE'; payload: { id: string; updates: Partial<Item> } }
  | { type: 'DELETE'; payload: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'LOADING': return { ...state, loading: true, error: null };
    case 'SUCCESS': return { ...state, loading: false, items: action.payload };
    case 'ERROR': return { ...state, loading: false, error: action.payload };
    case 'ADD': return { ...state, items: [...state.items, action.payload] };
    case 'UPDATE': return {
      ...state,
      items: state.items.map(i => i.id === action.payload.id ? { ...i, ...action.payload.updates } : i)
    };
    case 'DELETE': return { ...state, items: state.items.filter(i => i.id !== action.payload) };
  }
}
```

### Zustand for Global Client State
```typescript
import { create } from 'zustand';

type UIStore = {
  sidebarOpen: boolean;
  toggleSidebar: () => void;
  theme: 'light' | 'dark';
  setTheme: (theme: 'light' | 'dark') => void;
};

export const useUIStore = create<UIStore>((set) => ({
  sidebarOpen: true,
  toggleSidebar: () => set((s) => ({ sidebarOpen: !s.sidebarOpen })),
  theme: 'light',
  setTheme: (theme) => set({ theme }),
}));
```

## Performance Patterns

### React.memo for Expensive Components
```typescript
const ExpensiveList = React.memo(function ExpensiveList({ 
  items, 
  onSelect 
}: { 
  items: Item[]; 
  onSelect: (id: string) => void;
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onSelect(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});
```

### useCallback for Stable References
```typescript
function Parent() {
  const [items, setItems] = useState<Item[]>([]);
  
  const handleSelect = useCallback((id: string) => {
    setItems(prev => prev.map(i => ({ ...i, selected: i.id === id })));
  }, []);
  
  return <ExpensiveList items={items} onSelect={handleSelect} />;
}
```

### useMemo for Expensive Calculations
```typescript
function Dashboard({ transactions }: { transactions: Transaction[] }) {
  const stats = useMemo(() => ({
    total: transactions.reduce((sum, t) => sum + t.amount, 0),
    count: transactions.length,
    average: transactions.length ? transactions.reduce((s, t) => s + t.amount, 0) / transactions.length : 0,
  }), [transactions]);
  
  return <StatsDisplay stats={stats} />;
}
```

### Virtualization for Long Lists
```typescript
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });
  
  return (
    <div ref={parentRef} className="h-[400px] overflow-auto">
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map(virtual => (
          <div
            key={virtual.key}
            style={{
              position: 'absolute',
              top: virtual.start,
              height: virtual.size,
              width: '100%',
            }}
          >
            {items[virtual.index].name}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Accessibility Patterns

### Focus Management
```typescript
function Dialog({ isOpen, onClose, children }: DialogProps) {
  const closeRef = useRef<HTMLButtonElement>(null);
  
  useEffect(() => {
    if (isOpen) closeRef.current?.focus();
  }, [isOpen]);
  
  return isOpen ? (
    <div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
      <button ref={closeRef} onClick={onClose} aria-label="Close dialog">×</button>
      {children}
    </div>
  ) : null;
}
```

### Keyboard Navigation
```typescript
function Menu({ items, onSelect }: MenuProps) {
  const [focused, setFocused] = useState(0);
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown': setFocused(i => Math.min(i + 1, items.length - 1)); break;
      case 'ArrowUp': setFocused(i => Math.max(i - 1, 0)); break;
      case 'Enter': onSelect(items[focused]); break;
    }
  };
  
  return (
    <ul role="menu" onKeyDown={handleKeyDown}>
      {items.map((item, i) => (
        <li key={item.id} role="menuitem" tabIndex={i === focused ? 0 : -1}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```
