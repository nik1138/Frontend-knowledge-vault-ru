# Типизация хуков React в TypeScript

Типизация хуков React в TypeScript позволяет создавать строго типизированные и безопасные React-приложения. Эта секция охватывает все аспекты типизации встроенных и пользовательских хуков.

## Содержание

1. [Встроенные хуки](#встроенные-хуки)
2. [useState](#usestate)
3. [useEffect](#useeffect)
4. [useContext](#usecontext)
5. [useReducer](#usereducer)
6. [useCallback](#usecallback)
7. [useMemo](#usememo)
8. [useRef](#useref)
9. [Пользовательские хуки](#пользовательские-хуки)
10. [Продвинутые паттерны](#продвинутые-паттерны)
11. [Лучшие практики](#лучшие-практики)
12. [Распространенные ошибки](#распространенные-ошибки)

## Встроенные хуки

### Базовая типизация

```tsx
// Импорт типов из React
import { 
  useState, 
  useEffect, 
  useContext, 
  useReducer, 
  useCallback, 
  useMemo, 
  useRef 
} from 'react';

// Или использование глобальных типов
const [count, setCount] = useState<number>(0);
```

### Типы событий React

```tsx
// События мыши
type MouseEventHandler<T = Element> = (event: React.MouseEvent<T>) => void;

// События клавиатуры
type KeyboardEventHandler<T = Element> = (event: React.KeyboardEvent<T>) => void;

// События формы
type ChangeEventHandler<T = Element> = (event: React.ChangeEvent<T>) => void;

// События фокуса
type FocusEventHandler<T = Element> = (event: React.FocusEvent<T>) => void;
```

## useState

### Простые типы

```tsx
// Примитивные типы
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('');
const [isActive, setIsActive] = useState<boolean>(false);

// Union типы
const [status, setStatus] = useState<'loading' | 'success' | 'error'>('loading');

// Nullable типы
const [user, setUser] = useState<User | null>(null);
const [error, setError] = useState<string | undefined>(undefined);
```

### Сложные типы

```tsx
// Интерфейсы
interface User {
  id: number;
  name: string;
  email: string;
  preferences: {
    theme: 'light' | 'dark';
    notifications: boolean;
  };
}

const [user, setUser] = useState<User>({
  id: 1,
  name: 'John Doe',
  email: 'john@example.com',
  preferences: {
    theme: 'light',
    notifications: true
  }
});

// Типы
type FormState = {
  username: string;
  password: string;
  rememberMe: boolean;
};

const [formState, setFormState] = useState<FormState>({
  username: '',
  password: '',
  rememberMe: false
});
```

### Массивы и объекты

```tsx
// Массивы
const [items, setItems] = useState<string[]>([]);
const [users, setUsers] = useState<User[]>([]);

// Добавление элемента
const addItem = (item: string) => {
  setItems(prev => [...prev, item]);
};

// Объекты с динамическими ключами
const [preferences, setPreferences] = useState<Record<string, boolean>>({});

// Обновление объекта
const updatePreference = (key: string, value: boolean) => {
  setPreferences(prev => ({ ...prev, [key]: value }));
};
```

### Асинхронное состояние

```tsx
// Состояние загрузки
interface LoadingState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

const [state, setState] = useState<LoadingState<User[]>>({
  data: null,
  loading: false,
  error: null
});

// Функция для обновления состояния
const setLoadingState = (partial: Partial<LoadingState<User[]>>) => {
  setState(prev => ({ ...prev, ...partial }));
};
```

## useEffect

### Базовое использование

```tsx
// Без зависимостей (выполняется при каждом рендере)
useEffect(() => {
  document.title = 'My App';
});

// Пустой массив зависимостей (выполняется один раз)
useEffect(() => {
  const fetchInitialData = async () => {
    // Логика загрузки данных
  };
  
  fetchInitialData();
}, []);

// С зависимостями
const [userId, setUserId] = useState<number>(1);

useEffect(() => {
  const fetchUserData = async () => {
    // Логика загрузки данных пользователя
  };
  
  fetchUserData();
}, [userId]); // Перезапускается при изменении userId
```

### Очистка эффектов

```tsx
// С таймерами
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Timer tick');
  }, 1000);

  // Функция очистки
  return () => {
    clearInterval(timer);
  };
}, []);

// С подписками
useEffect(() => {
  const subscription = someObservable.subscribe(data => {
    console.log(data);
  });

  return () => {
    subscription.unsubscribe();
  };
}, []);
```

### Асинхронные эффекты

```tsx
// Правильный способ с асинхронными операциями
useEffect(() => {
  let isMounted = true;

  const fetchData = async () => {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      
      if (isMounted) {
        setData(data);
      }
    } catch (error) {
      if (isMounted) {
        setError(error instanceof Error ? error.message : 'Unknown error');
      }
    }
  };

  fetchData();

  return () => {
    isMounted = false;
  };
}, []);

// Альтернатива с IIFE
useEffect(() => {
  (async () => {
    try {
      const response = await fetch('/api/data');
      const data = await response.json();
      setData(data);
    } catch (error) {
      setError(error instanceof Error ? error.message : 'Unknown error');
    }
  })();
}, []);
```

## useContext

### Базовая типизация контекста

```tsx
// Интерфейс контекста
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// Создание контекста
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Провайдер
export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Хук для использования контекста
const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

### Сложные контексты

```tsx
// Интерфейс для сложного контекста
interface AppContextType {
  user: User | null;
  setUser: React.Dispatch<React.SetStateAction<User | null>>;
  notifications: Notification[];
  addNotification: (notification: Omit<Notification, 'id' | 'timestamp'>) => void;
  removeNotification: (id: string) => void;
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// Создание контекста
const AppContext = createContext<AppContextType | undefined>(undefined);

// Провайдер
export const AppProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const addNotification = (notification: Omit<Notification, 'id' | 'timestamp'>) => {
    const newNotification: Notification = {
      ...notification,
      id: Math.random().toString(36).substr(2, 9),
      timestamp: new Date()
    };
    setNotifications(prev => [...prev, newNotification]);
  };

  const removeNotification = (id: string) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  };

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <AppContext.Provider value={{
      user,
      setUser,
      notifications,
      addNotification,
      removeNotification,
      theme,
      toggleTheme
    }}>
      {children}
    </AppContext.Provider>
  );
};

// Хук для использования контекста
const useApp = (): AppContextType => {
  const context = useContext(AppContext);
  if (context === undefined) {
    throw new Error('useApp must be used within an AppProvider');
  }
  return context;
};
```

## useReducer

### Базовая типизация

```tsx
// Типы состояния и действий
interface State {
  count: number;
  loading: boolean;
}

type Action = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'RESET' };

// Начальное состояние
const initialState: State = {
  count: 0,
  loading: false
};

// Редьюсер
const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'RESET':
      return initialState;
    default:
      return state;
  }
};

// Использование
const [state, dispatch] = useReducer(reducer, initialState);
```

### Сложные редьюсеры

```tsx
// Состояние для списка задач
interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}

interface TodosState {
  items: Todo[];
  filter: 'all' | 'active' | 'completed';
  loading: boolean;
  error: string | null;
}

// Действия
type TodosAction = 
  | { type: 'ADD_TODO'; payload: Omit<Todo, 'id' | 'createdAt'> }
  | { type: 'REMOVE_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: string }
  | { type: 'SET_FILTER'; payload: 'all' | 'active' | 'completed' }
  | { type: 'SET_LOADING'; payload: boolean }
  | { type: 'SET_ERROR'; payload: string | null }
  | { type: 'LOAD_TODOS'; payload: Todo[] };

// Редьюсер
const todosReducer = (state: TodosState, action: TodosAction): TodosState => {
  switch (action.type) {
    case 'ADD_TODO':
      const newTodo: Todo = {
        id: Math.random().toString(36).substr(2, 9),
        ...action.payload,
        createdAt: new Date()
      };
      return {
        ...state,
        items: [...state.items, newTodo]
      };
    
    case 'REMOVE_TODO':
      return {
        ...state,
        items: state.items.filter(todo => todo.id !== action.payload)
      };
    
    case 'TOGGLE_TODO':
      return {
        ...state,
        items: state.items.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      };
    
    case 'SET_LOADING':
      return {
        ...state,
        loading: action.payload
      };
    
    case 'SET_ERROR':
      return {
        ...state,
        error: action.payload
      };
    
    case 'LOAD_TODOS':
      return {
        ...state,
        items: action.payload,
        loading: false
      };
    
    default:
      return state;
  }
};

// Начальное состояние
const initialTodosState: TodosState = {
  items: [],
  filter: 'all',
  loading: false,
  error: null
};

// Использование
const [todosState, todosDispatch] = useReducer(todosReducer, initialTodosState);
```

## useCallback

### Базовая типизация

```tsx
// Типизированный callback
const handleClick = useCallback((id: number) => {
  console.log('Clicked item with id:', id);
}, []);

// Callback с возвращаемым значением
const calculateTotal = useCallback((items: number[]): number => {
  return items.reduce((sum, item) => sum + item, 0);
}, []);

// Callback с асинхронной операцией
const fetchUserData = useCallback(async (userId: number): Promise<User> => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
}, []);
```

### С зависимостями

```tsx
// Callback с зависимостями
const [multiplier, setMultiplier] = useState<number>(1);

const multiply = useCallback((value: number): number => {
  return value * multiplier;
}, [multiplier]);

// Callback для обработчиков событий
const [formData, setFormData] = useState<FormState>({
  username: '',
  password: '',
  rememberMe: false
});

const handleInputChange = useCallback((
  e: React.ChangeEvent<HTMLInputElement>
) => {
  const { name, value, type, checked } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: type === 'checkbox' ? checked : value
  }));
}, []);
```

## useMemo

### Базовая типизация

```tsx
// Мемоизированные вычисления
const [items, setItems] = useState<number[]>([1, 2, 3, 4, 5]);
const [multiplier, setMultiplier] = useState<number>(2);

const multipliedItems = useMemo<number[]>(() => {
  return items.map(item => item * multiplier);
}, [items, multiplier]);

// Сложные вычисления
const [users, setUsers] = useState<User[]>([]);

const activeUsersCount = useMemo<number>(() => {
  return users.filter(user => user.isActive).length;
}, [users]);

// Мемоизированные объекты
const userPreferences = useMemo(() => ({
  theme: 'dark',
  notifications: true,
  language: 'en'
}), []);
```

### С зависимостями

```tsx
// Мемоизация с несколькими зависимостями
const [searchTerm, setSearchTerm] = useState<string>('');
const [users, setUsers] = useState<User[]>([]);

const filteredUsers = useMemo<User[]>(() => {
  return users.filter(user =>
    user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
    user.email.toLowerCase().includes(searchTerm.toLowerCase())
  );
}, [users, searchTerm]);

// Мемоизация функций
const [sortField, setSortField] = useState<'name' | 'email'>('name');
const [sortOrder, setSortOrder] = useState<'asc' | 'desc'>('asc');

const sortedUsers = useMemo<User[]>(() => {
  return [...users].sort((a, b) => {
    const aValue = a[sortField];
    const bValue = b[sortField];
    
    if (aValue < bValue) return sortOrder === 'asc' ? -1 : 1;
    if (aValue > bValue) return sortOrder === 'asc' ? 1 : -1;
    return 0;
  });
}, [users, sortField, sortOrder]);
```

## useRef

### Базовая типизация

```tsx
// Ref для DOM элементов
const inputRef = useRef<HTMLInputElement>(null);

const focusInput = () => {
  inputRef.current?.focus();
};

// Ref для значений
const renderCountRef = useRef<number>(0);
renderCountRef.current += 1;

// Ref для предыдущих значений
const prevValueRef = useRef<string>('');
useEffect(() => {
  prevValueRef.current = currentValue;
}, [currentValue]);
```

### Сложные типы

```tsx
// Ref для таймеров
const timerRef = useRef<NodeJS.Timeout | null>(null);

const startTimer = () => {
  if (timerRef.current) {
    clearInterval(timerRef.current);
  }
  
  timerRef.current = setInterval(() => {
    console.log('Timer tick');
  }, 1000);
};

const stopTimer = () => {
  if (timerRef.current) {
    clearInterval(timerRef.current);
    timerRef.current = null;
  }
};

// Ref для компонентов
const childRef = useRef<ChildComponentRef>(null);

interface ChildComponentRef {
  focus: () => void;
  getValue: () => string;
}

const ChildComponent = forwardRef<ChildComponentRef>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
    getValue: () => inputRef.current?.value || ''
  }));
  
  return <input ref={inputRef} />;
});
```

## Пользовательские хуки

### Базовые пользовательские хуки

```tsx
// Хук для управления состоянием переключателя
function useToggle(initialValue: boolean = false): [boolean, () => void] {
  const [value, setValue] = useState<boolean>(initialValue);
  
  const toggle = useCallback(() => {
    setValue(prev => !prev);
  }, []);
  
  return [value, toggle];
}

// Использование
const [isVisible, toggleVisibility] = useToggle(false);
```

### Хуки для API

```tsx
// Хук для загрузки данных
interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
}

function useApi<T>(url: string): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result: T = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const UserList: React.FC = () => {
  const { data: users, loading, error, refetch } = useApi<User[]>('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!users) return <div>No users found</div>;

  return (
    <div>
      <button onClick={() => refetch()}>Refresh</button>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### Хуки для форм

```tsx
// Хук для управления формой
interface UseFormOptions<T> {
  initialValues: T;
  onSubmit: (values: T) => void | Promise<void>;
  validate?: (values: T) => Record<string, string>;
}

interface UseFormResult<T> {
  values: T;
  errors: Record<string, string>;
  handleChange: (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => void;
  handleSubmit: (e: React.FormEvent) => void;
  setValues: React.Dispatch<React.SetStateAction<T>>;
  setErrors: React.Dispatch<React.SetStateAction<Record<string, string>>>;
}

function useForm<T>({ 
  initialValues, 
  onSubmit, 
  validate 
}: UseFormOptions<T>): UseFormResult<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>
  ) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }
    
    onSubmit(values);
  };

  return {
    values,
    errors,
    handleChange,
    handleSubmit,
    setValues,
    setErrors
  };
}

// Использование
interface LoginFormValues {
  email: string;
  password: string;
  rememberMe: boolean;
}

const LoginForm: React.FC = () => {
  const { values, errors, handleChange, handleSubmit } = useForm<LoginFormValues>({
    initialValues: {
      email: '',
      password: '',
      rememberMe: false
    },
    onSubmit: async (values) => {
      // Логика отправки формы
      console.log('Login with:', values);
    },
    validate: (values) => {
      const errors: Record<string, string> = {};
      
      if (!values.email) {
        errors.email = 'Email is required';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Email is invalid';
      }
      
      if (!values.password) {
        errors.password = 'Password is required';
      } else if (values.password.length < 6) {
        errors.password = 'Password must be at least 6 characters';
      }
      
      return errors;
    }
  });

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span>{errors.email}</span>}
      </div>
      
      <div>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {errors.password && <span>{errors.password}</span>}
      </div>
      
      <label>
        <input
          type="checkbox"
          name="rememberMe"
          checked={values.rememberMe}
          onChange={handleChange}
        />
        Remember me
      </label>
      
      <button type="submit">Login</button>
    </form>
  );
};
```

## Продвинутые паттерны

### Хуки с дженериками

```tsx
// Универсальный хук для локального хранилища
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  };

  return [storedValue, setValue];
}

// Использование
const [userPreferences, setUserPreferences] = useLocalStorage<UserPreferences>(
  'user-preferences',
  { theme: 'light', notifications: true }
);
```

### Хуки с условной типизацией

```tsx
// Хук с условными типами
type UseApiOptions<T, E = string> = {
  url: string;
  method?: 'GET' | 'POST' | 'PUT' | 'DELETE';
  body?: T extends object ? T : never;
  headers?: Record<string, string>;
  errorType?: E;
};

function useApi<T, E = string>(options: UseApiOptions<T, E>): {
  data: T | null;
  loading: boolean;
  error: E | null;
  execute: () => Promise<void>;
} {
  // Реализация...
  return {
    data: null,
    loading: false,
    error: null,
    execute: async () => {}
  };
}
```

### Хуки с mapped types

```tsx
// Хук для управления множественными состояниями
type StateMap<T> = {
  [K in keyof T]: [T[K], React.Dispatch<React.SetStateAction<T[K]>>];
};

function useMultipleState<T extends Record<string, any>>(initialState: T): StateMap<T> {
  type Result = { [K in keyof T]: [T[K], React.Dispatch<React.SetStateAction<T[K]>>] };
  
  const result = {} as Result;
  
  for (const key in initialState) {
    const [value, setValue] = useState(initialState[key]);
    result[key] = [value, setValue];
  }
  
  return result;
}

// Использование
const states = useMultipleState({
  name: '',
  age: 0,
  isActive: false
});

const [name, setName] = states.name;
const [age, setAge] = states.age;
const [isActive, setIsActive] = states.isActive;
```

## Лучшие практики

### 1. Явная типизация

```tsx
// Хорошо: явная типизация
const [count, setCount] = useState<number>(0);

// Плохо: неявная типизация
const [count, setCount] = useState(0); // number inferred

// Хорошо: типизация объектов
interface User {
  id: number;
  name: string;
}

const [user, setUser] = useState<User | null>(null);

// Плохо: использование any
const [user, setUser] = useState<any>(null);
```

### 2. Типизация событий

```tsx
// Хорошо: точная типизация событий
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};

// Плохо: общая типизация
const handleChange = (e: React.SyntheticEvent) => {
  // e.target.value может не существовать
};
```

### 3. Использование utility types

```tsx
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Использование Partial для форм
interface UserFormValues extends Partial<User> {
  name: string; // Обязательное поле
  email: string; // Обязательное поле
}

// Использование Omit для исключения полей
interface UserPublic extends Omit<User, 'password'> {}

// Использование Pick для выбора полей
interface UserBasic extends Pick<User, 'id' | 'name'> {}
```

### 4. Типизация пользовательских хуков

```tsx
// Хорошо: четкая типизация возвращаемых значений
interface UseCounterResult {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

function useCounter(initialValue: number = 0): UseCounterResult {
  const [count, setCount] = useState(initialValue);
  
  return {
    count,
    increment: () => setCount(prev => prev + 1),
    decrement: () => setCount(prev => prev - 1),
    reset: () => setCount(initialValue)
  };
}

// Плохо: неструктурированный возврат
function useCounter(initialValue: number = 0) {
  const [count, setCount] = useState(initialValue);
  
  return [
    count,
    () => setCount(prev => prev + 1),
    () => setCount(prev => prev - 1)
  ]; // [number, () => void, () => void] - неочевидно
}
```

## Распространенные ошибки

### 1. Неявная типизация any

```tsx
// Проблема: неявный any
function BadComponent({ data }) {  // data: any
  return <div>{data.name}</div>;  // Нет проверки типов
}

// Исправление: явная типизация
interface Data {
  name: string;
}

interface Props {
  data: Data;
}

function GoodComponent({ data }: Props) {
  return <div>{data.name}</div>;
}
```

### 2. Неправильная типизация событий

```tsx
// Проблема: неправильный тип события
const handleChange = (e) => {  // e: any
  console.log(e.target.value);  // Нет автозаполнения
};

// Исправление: правильная типизация
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);  // Полная автозаполнение и проверка типов
};
```

### 3. Проблемы с useState

```tsx
// Проблема: неопределенное состояние
const [user, setUser] = useState();  // user: undefined

// Исправление: явная типизация
interface User {
  id: number;
  name: string;
}

const [user, setUser] = useState<User | null>(null);
```

### 4. Неправильная типизация контекста

```tsx
// Проблема: контекст без проверки
const ThemeContext = createContext();  // any

function BadComponent() {
  const theme = useContext(ThemeContext);  // any
  return <div className={theme} />;  // Нет проверки типов
}

// Исправление: типизированный контекст
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

function GoodComponent() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('Component must be used within ThemeProvider');
  }
  
  const { theme, toggleTheme } = context;
  return <div className={theme} />;
}
```

### 5. Проблемы с useRef

```tsx
// Проблема: неправильная инициализация ref
const inputRef = useRef();  // RefObject<undefined>

// Исправление: правильная типизация
const inputRef = useRef<HTMLInputElement>(null);
```

## Связи с другими концепциями

- [[ts/react/React_с_TypeScript|TypeScript с React]] - Основы использования TypeScript в React
- [[ts/advanced/generics|Дженерики]] - Использование дженериков в хуках
- [[ts/advanced/utility-types|Вспомогательные типы]] - Utility types для типизации хуков
- [[ts/react/components|Компоненты]] - Типизация компонентов с хуками
- [[ts/react/forms|Формы]] - Типизация форм с использованием хуков

## Рекомендации по изучению

1. Начните с базовой типизации встроенных хуков
2. Освойте типизацию useState и useEffect
3. Практикуйтесь в типизации useContext и useReducer
4. Изучите типизацию useCallback и useMemo
5. Освойте useRef и useImperativeHandle
6. Практикуйтесь в создании пользовательских хуков
7. Изучите продвинутые паттерны типизации хуков

## Следующие шаги

- [[ts/react/components|Компоненты]] - Типизация различных видов компонентов
- [[ts/react/forms|Формы]] - Работа с формами в TypeScript
- [[ts/react/context|Контекст]] - Типизация контекста и провайдеров
- [[ts/react/testing|Тестирование]] - Тестирование React-компонентов с TypeScript
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация React-приложений с TypeScript