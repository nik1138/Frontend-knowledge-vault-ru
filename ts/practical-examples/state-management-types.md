---
aliases: ["Типизация систем управления состоянием", "TS State Management Types"]
tags: [typescript, state-management, patterns, architecture]
---

# Типизация систем управления состоянием

## Введение

Типизация систем управления состоянием в приложениях позволяет обеспечить безопасность типов, улучшить автодополнение в IDE и уменьшить количество ошибок. В этом разделе рассмотрим различные подходы к типизации систем управления состоянием, включая Redux, Zustand, и кастомные решения.

## Базовые паттерны типизации

### Простая система состояния

```typescript
// Интерфейс для базового состояния
interface State {
  [key: string]: any;
}

// Тип для обновления состояния
type StateUpdater<T> = (prevState: T) => T;

// Простая система управления состоянием
class SimpleStore<T extends State> {
  private state: T;
  private listeners: Array<() => void> = [];
  
  constructor(initialState: T) {
    this.state = initialState;
  }
  
  getState(): T {
    return { ...this.state }; // Возвращаем копию для иммутабельности
  }
  
  setState(updater: StateUpdater<T> | Partial<T>): void {
    const newState = typeof updater === 'function' 
      ? updater(this.state) 
      : { ...this.state, ...updater };
    
    if (newState !== this.state) {
      this.state = newState;
      this.notifyListeners();
    }
  }
  
  subscribe(listener: () => void): () => void {
    this.listeners.push(listener);
    
    // Возвращаем функцию для отписки
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  private notifyListeners(): void {
    this.listeners.forEach(listener => listener());
  }
}

// Пример использования
interface UserState {
  user: {
    id: number | null;
    name: string | null;
    email: string | null;
  };
  isLoggedIn: boolean;
  loading: boolean;
}

const initialUserState: UserState = {
  user: {
    id: null,
    name: null,
    email: null
  },
  isLoggedIn: false,
  loading: false
};

const userStore = new SimpleStore(initialUserState);

// Использование
userStore.setState(prev => ({
  ...prev,
  user: { ...prev.user, name: 'John Doe' },
  isLoggedIn: true
}));
```

## Продвинутые примеры

### Типизация с действиями (Actions)

```typescript
// Типы для действий
type Action<T = any> = {
  type: string;
  payload?: T;
};

// Тип для редьюсера
type Reducer<T, A extends Action = Action> = (state: T, action: A) => T;

// Типизированная система управления состоянием
class TypedStore<T, A extends Action = Action> {
  private state: T;
  private reducer: Reducer<T, A>;
  private listeners: Array<() => void> = [];
  
  constructor(reducer: Reducer<T, A>, initialState: T) {
    this.reducer = reducer;
    this.state = initialState;
  }
  
  getState(): T {
    return this.state;
  }
  
  dispatch(action: A): void {
    const newState = this.reducer(this.state, action);
    if (newState !== this.state) {
      this.state = newState;
      this.notifyListeners();
    }
  }
  
  subscribe(listener: () => void): () => void {
    this.listeners.push(listener);
    
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  private notifyListeners(): void {
    this.listeners.forEach(listener => listener());
  }
}

// Определение типов действий для пользовательского состояния
type UserAction = 
  | { type: 'USER_LOGIN'; payload: { id: number; name: string; email: string } }
  | { type: 'USER_LOGOUT' }
  | { type: 'USER_UPDATE'; payload: Partial<{ name: string; email: string }> }
  | { type: 'SET_LOADING'; payload: boolean };

// Редьюсер для пользовательского состояния
const userReducer: Reducer<UserState, UserAction> = (state, action) => {
  switch (action.type) {
    case 'USER_LOGIN':
      return {
        ...state,
        user: {
          id: action.payload.id,
          name: action.payload.name,
          email: action.payload.email
        },
        isLoggedIn: true,
        loading: false
      };
    
    case 'USER_LOGOUT':
      return {
        ...state,
        user: {
          id: null,
          name: null,
          email: null
        },
        isLoggedIn: false
      };
    
    case 'USER_UPDATE':
      return {
        ...state,
        user: {
          ...state.user,
          ...action.payload
        }
      };
    
    case 'SET_LOADING':
      return {
        ...state,
        loading: action.payload
      };
    
    default:
      return state;
  }
};

// Создание хранилища с типизацией
const typedUserStore = new TypedStore(userReducer, initialUserState);

// Использование
typedUserStore.dispatch({
  type: 'USER_LOGIN',
  payload: { id: 1, name: 'John Doe', email: 'john@example.com' }
});
```

### Типизация с middleware

```typescript
// Тип для middleware
type Middleware<T, A extends Action = Action> = (
  store: { getState: () => T; dispatch: (action: A) => void }
) => (next: (action: A) => void) => (action: A) => any;

// Расширенное хранилище с поддержкой middleware
class MiddlewareStore<T, A extends Action = Action> {
  private state: T;
  private reducer: Reducer<T, A>;
  private listeners: Array<() => void> = [];
  private middlewares: Middleware<T, A>[];
  
  constructor(
    reducer: Reducer<T, A>, 
    initialState: T, 
    middlewares: Middleware<T, A>[] = []
  ) {
    this.reducer = reducer;
    this.state = initialState;
    this.middlewares = middlewares;
  }
  
  getState(): T {
    return this.state;
  }
  
  dispatch(action: A): void {
    // Создаем цепочку middleware
    const dispatchChain = this.middlewares.map(middleware => 
      middleware(this)
    ).reduceRight((next, middleware) => middleware(next), (action: A) => {
      const newState = this.reducer(this.state, action);
      if (newState !== this.state) {
        this.state = newState;
        this.notifyListeners();
      }
    });
    
    dispatchChain(action);
  }
  
  subscribe(listener: () => void): () => void {
    this.listeners.push(listener);
    
    return () => {
      const index = this.listeners.indexOf(listener);
      if (index > -1) {
        this.listeners.splice(index, 1);
      }
    };
  }
  
  private notifyListeners(): void {
    this.listeners.forEach(listener => listener());
  }
}

// Пример middleware для логирования
const loggerMiddleware: Middleware<UserState, UserAction> = (store) => (next) => (action) => {
  console.log('Dispatching action:', action);
  console.log('Previous state:', store.getState());
  
  const result = next(action);
  
  console.log('Next state:', store.getState());
  return result;
};

// Пример асинхронного middleware
const asyncMiddleware: Middleware<UserState, UserAction> = (store) => (next) => async (action) => {
  if (typeof action === 'function') {
    return await action(store.dispatch, store.getState);
  }
  
  return next(action);
};

// Создание хранилища с middleware
const middlewareStore = new MiddlewareStore(
  userReducer, 
  initialUserState, 
  [loggerMiddleware]
);
```

## Практические примеры

### Типизация с Zustand (популярная библиотека)

```typescript
// Типы для Zustand-подобного хранилища
type StateCreator<T, CustomSet = SetState<T>, CustomGet = GetState<T>> = (
  set: CustomSet,
  get: CustomGet,
  api: StoreApi<T>
) => T;

type SetState<T> = (partial: T | Partial<T> | ((state: T) => T | Partial<T>), replace?: boolean) => void;
type GetState<T> = () => T;
type Subscribe<T> = (listener: (state: T, prevState: T) => void) => () => void;
type Destroy = () => void;

interface StoreApi<T> {
  setState: SetState<T>;
  getState: GetState<T>;
  subscribe: Subscribe<T>;
  destroy: Destroy;
}

// Упрощенная реализация Zustand-подобного хранилища
function create<T>(
  createState: StateCreator<T>
): StoreApi<T> & (() => T) {
  let state: T;
  const listeners = new Set<(state: T, prevState: T) => void>();
  
  const setState: SetState<T> = (partial, replace) => {
    const prevState = state;
    const nextState = typeof partial === 'function' ? partial(state) : partial;
    
    state = replace
      ? (nextState as T)
      : { ...state as any, ...nextState as any };
    
    listeners.forEach(listener => listener(state, prevState));
  };
  
  const getState: GetState<T> = () => state;
  
  const subscribe: Subscribe<T> = (listener) => {
    listeners.add(listener);
    return () => listeners.delete(listener);
  };
  
  const destroy: Destroy = () => {
    listeners.clear();
  };
  
  state = createState(setState, getState, { setState, getState, subscribe, destroy });
  
  const store = () => state;
  Object.assign(store, { setState, getState, subscribe, destroy });
  
  return store as StoreApi<T> & (() => T);
}

// Пример использования
interface CounterState {
  count: number;
  increment: () => void;
  decrement: () => void;
  incrementBy: (amount: number) => void;
}

const useCounterStore = create<CounterState>((set, get) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 })),
  decrement: () => set(state => ({ count: state.count - 1 })),
  incrementBy: (amount) => set(state => ({ count: state.count + amount }))
}));

// Использование в React
import { useEffect } from 'react';

const CounterComponent: React.FC = () => {
  const { count, increment, decrement, incrementBy } = useCounterStore();
  
  useEffect(() => {
    console.log('Count changed:', count);
  }, [count]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={() => incrementBy(5)}>+5</button>
    </div>
  );
};
```

### Типизация с Redux Toolkit

```typescript
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';

// Определение состояния
interface TodoState {
  items: Todo[];
  loading: boolean;
  error: string | null;
}

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

// Создание слайса с типизацией
const todosSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    loading: false,
    error: null
  } as TodoState,
  reducers: {
    // Типизация действия с payload
    addTodo: (state, action: PayloadAction<string>) => {
      const newTodo: Todo = {
        id: Date.now(),
        text: action.payload,
        completed: false
      };
      state.items.push(newTodo);
    },
    toggleTodo: (state, action: PayloadAction<number>) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    deleteTodo: (state, action: PayloadAction<number>) => {
      state.items = state.items.filter(t => t.id !== action.payload);
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setError: (state, action: PayloadAction<string | null>) => {
      state.error = action.payload;
    }
  }
});

// Экшены
export const { addTodo, toggleTodo, deleteTodo, setLoading, setError } = todosSlice.actions;

// Создание хранилища
export const store = configureStore({
  reducer: {
    todos: todosSlice.reducer
  }
});

// Типы для состояния и dispatch
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// Пример использования в компоненте
import { useSelector, useDispatch } from 'react-redux';

const TodoComponent: React.FC = () => {
  const { items, loading, error } = useSelector((state: RootState) => state.todos);
  const dispatch = useDispatch<AppDispatch>();
  
  const handleAddTodo = (text: string) => {
    dispatch(addTodo(text));
  };
  
  return (
    <div>
      {loading && <p>Loading...</p>}
      {error && <p>Error: {error}</p>}
      <ul>
        {items.map(todo => (
          <li key={todo.id}>
            <span 
              style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
              onClick={() => dispatch(toggleTodo(todo.id))}
            >
              {todo.text}
            </span>
            <button onClick={() => dispatch(deleteTodo(todo.id))}>Delete</button>
          </li>
        ))}
      </ul>
      <button onClick={() => handleAddTodo('New todo')}>Add Todo</button>
    </div>
  );
};
```

### Типизация глобального состояния приложения

```typescript
// Определение глобального состояния приложения
interface AppState {
  user: UserState;
  ui: {
    theme: 'light' | 'dark';
    language: string;
    notifications: Notification[];
  };
  features: {
    [featureName: string]: boolean;
  };
}

// Типы для уведомлений
interface Notification {
  id: string;
  type: 'info' | 'success' | 'warning' | 'error';
  message: string;
  timestamp: Date;
  read: boolean;
}

// Типизированный контекст для React
import { createContext, useContext, ReactNode } from 'react';

interface StoreContextValue {
  state: AppState;
  dispatch: (action: AppAction) => void;
}

const StoreContext = createContext<StoreContextValue | null>(null);

interface StoreProviderProps {
  children: ReactNode;
}

export const StoreProvider: React.FC<StoreProviderProps> = ({ children }) => {
  const [state, setState] = useState<AppState>(initialAppState);
  
  const dispatch = useCallback((action: AppAction) => {
    setState(prevState => appReducer(prevState, action));
  }, []);
  
  return (
    <StoreContext.Provider value={{ state, dispatch }}>
      {children}
    </StoreContext.Provider>
  );
};

export const useStore = (): StoreContextValue => {
  const context = useContext(StoreContext);
  if (!context) {
    throw new Error('useStore must be used within a StoreProvider');
  }
  return context;
};

// Типы действий для всего приложения
type AppAction = 
  | UserAction
  | { type: 'SET_THEME'; payload: 'light' | 'dark' }
  | { type: 'SET_LANGUAGE'; payload: string }
  | { type: 'ADD_NOTIFICATION'; payload: Omit<Notification, 'id' | 'timestamp' | 'read'> }
  | { type: 'MARK_NOTIFICATION_READ'; payload: string }
  | { type: 'TOGGLE_FEATURE'; payload: { feature: string; enabled: boolean } };

// Редьюсер для всего приложения
const appReducer: Reducer<AppState, AppAction> = (state, action) => {
  return {
    ...state,
    user: userReducer(state.user, action as UserAction),
    ui: uiReducer(state.ui, action),
    features: featureReducer(state.features, action)
  };
};

const uiReducer: Reducer<AppState['ui'], AppAction> = (state, action) => {
  switch (action.type) {
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    
    case 'SET_LANGUAGE':
      return { ...state, language: action.payload };
    
    case 'ADD_NOTIFICATION':
      return {
        ...state,
        notifications: [
          ...state.notifications,
          {
            id: Date.now().toString(),
            ...action.payload,
            timestamp: new Date(),
            read: false
          }
        ]
      };
    
    case 'MARK_NOTIFICATION_READ':
      return {
        ...state,
        notifications: state.notifications.map(n => 
          n.id === action.payload ? { ...n, read: true } : n
        )
      };
    
    default:
      return state;
  }
};

const featureReducer: Reducer<AppState['features'], AppAction> = (state, action) => {
  switch (action.type) {
    case 'TOGGLE_FEATURE':
      return {
        ...state,
        [action.payload.feature]: action.payload.enabled
      };
    
    default:
      return state;
  }
};
```

## Заключение

Типизация систем управления состоянием - ключевой аспект разработки надежных приложений. Правильно спроектированные типы обеспечивают безопасность, улучшают опыт разработки и упрощают поддержку кода. Используйте эти паттерны как основу для создания типизированных систем управления состоянием в ваших приложениях.

См. также: [[dependency-injection-types]] | [[middleware-types]] | [[testing-types]]