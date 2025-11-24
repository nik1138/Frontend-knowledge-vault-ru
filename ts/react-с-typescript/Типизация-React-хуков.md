---
aliases: [React Hooks Typing, Типизированные React хуки]
tags: [typescript, react, hooks, typing]
---

# Типизация React-хуков

## Обзор

React-хуки - это мощный инструмент для управления состоянием и побочными эффектами в функциональных компонентах. При использовании TypeScript важно правильно типизировать хуки, чтобы получить преимущества строгой типизации и избежать ошибок на этапе разработки.

## useState

### Базовая типизация

```typescript
import React, { useState } from 'react';

const Counter: React.FC = () => {
  // TypeScript может вывести тип автоматически
  const [count, setCount] = useState(0); // number
  
  // Или можно указать тип явно
  const [name, setName] = useState<string>(''); // string
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Введите имя"
      />
    </div>
  );
};
```

### Типизация сложных состояний

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

const UserForm: React.FC = () => {
  const [user, setUser] = useState<User>({
    id: 0,
    name: '',
    email: ''
  });
  
  const [users, setUsers] = useState<User[]>([]);
  
  const [loading, setLoading] = useState<boolean>(false);
  
  const [error, setError] = useState<string | null>(null);
  
  const updateUser = (id: number, updates: Partial<User>) => {
    setUsers(prevUsers => 
      prevUsers.map(user => 
        user.id === id ? { ...user, ...updates } : user
      )
    );
  };
  
  return (
    <div>
      {loading && <p>Загрузка...</p>}
      {error && <p style={{color: 'red'}}>{error}</p>}
      {/* JSX компонента */}
    </div>
  );
};
```

### Инициализация с помощью функции

```typescript
interface InitialState {
  items: string[];
  count: number;
}

const ItemList: React.FC = () => {
  // Используем функцию для инициализации состояния
  const [state, setState] = useState<InitialState>(() => {
    const savedItems = localStorage.getItem('items');
    return {
      items: savedItems ? JSON.parse(savedItems) : [],
      count: 0
    };
  });
  
  return <div>{/* JSX */}</div>;
};
```

## useEffect

### Базовая типизация

```typescript
import React, { useState, useEffect } from 'react';

const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
  const [userData, setUserData] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        setUserData(data);
      } catch (error) {
        console.error('Ошибка при загрузке пользователя:', error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]); // Зависимости
  
  return (
    <div>
      {loading ? <p>Загрузка...</p> : userData && <p>{userData.name}</p>}
    </div>
  );
};
```

### Очистка эффектов

```typescript
const Timer: React.FC = () => {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // Функция очистки
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  return <div>Таймер: {seconds} секунд</div>;
};
```

## useContext

### Создание типизированного контекста

```typescript
import React, { createContext, useContext, useState } from 'react';

interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

interface ThemeProviderProps {
  children: React.ReactNode;
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const value: ThemeContextType = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      <div className={theme}>
        {children}
      </div>
    </ThemeContext.Provider>
  );
};

// Использование в компоненте
const Header: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={`header ${theme}`}>
      <h1>Текущая тема: {theme}</h1>
      <button onClick={toggleTheme}>Переключить тему</button>
    </header>
  );
};
```

## useReducer

### Базовая типизация

```typescript
import React, { useReducer } from 'react';

interface State {
  count: number;
  step: number;
}

type Action = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' }
  | { type: 'SET_STEP'; payload: number };

const initialState: State = {
  count: 0,
  step: 1
};

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step };
    case 'DECREMENT':
      return { ...state, count: state.count - state.step };
    case 'RESET':
      return { ...state, count: 0 };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    default:
      return state;
  }
};

const CounterWithReducer: React.FC = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Счетчик: {state.count}</p>
      <p>Шаг: {state.step}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Сброс</button>
      <button onClick={() => dispatch({ type: 'SET_STEP', payload: 5 })}>
        Установить шаг 5
      </button>
    </div>
  );
};
```

### useReducer с дженериками

```typescript
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

type TodoAction = 
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: number }
  | { type: 'DELETE_TODO'; payload: number };

const todoReducer = (state: Todo[], action: TodoAction): Todo[] => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          id: Date.now(),
          text: action.payload,
          completed: false
        }
      ];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
};

const TodoApp: React.FC = () => {
  const [todos, dispatch] = useReducer(todoReducer, []);
  
  const addTodo = (text: string) => {
    dispatch({ type: 'ADD_TODO', payload: text });
  };
  
  return (
    <div>
      {/* Реализация списка дел */}
    </div>
  );
};
```

## useRef

### Типизация для DOM элементов

```typescript
import React, { useRef, useEffect } from 'react';

const FocusInput: React.FC = () => {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  }, []);
  
  const handleClick = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Фокусируемое поле" />
      <button onClick={handleClick}>Фокус</button>
    </div>
  );
};
```

### Типизация для значений

```typescript
const TimerWithRef: React.FC = () => {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  
  const startTimer = () => {
    if (intervalRef.current) return; // Предотвращаем дублирование
    
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };
  
  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <p>Таймер: {seconds} секунд</p>
      <button onClick={startTimer}>Старт</button>
      <button onClick={stopTimer}>Стоп</button>
    </div>
  );
};
```

### Использование без типа (для значений, не DOM элементов)

```typescript
const usePrevious = <T,>(value: T): T | undefined => {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
};

const ComponentWithPrevious: React.FC = () => {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Текущее: {count}</p>
      <p>Предыдущее: {prevCount ?? 'нет'}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
};
```

## useCallback и useMemo

### useCallback

```typescript
import React, { useState, useCallback, memo } from 'react';

interface ChildComponentProps {
  onClick: () => void;
  count: number;
}

const ChildComponent = memo<ChildComponentProps>(({ onClick, count }) => {
  console.log('Рендер ChildComponent');
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={onClick}>Увеличить</button>
    </div>
  );
});

const ParentComponent: React.FC = () => {
  const [count, setCount] = useState(0);
  const [otherState, setOtherState] = useState('');
  
  // useCallback с типизацией
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // Зависимости
  
  return (
    <div>
      <input 
        value={otherState} 
        onChange={(e) => setOtherState(e.target.value)} 
        placeholder="Другое состояние"
      />
      <ChildComponent count={count} onClick={increment} />
    </div>
  );
};
```

### useMemo

```typescript
const ExpensiveComponent: React.FC<{ items: number[] }> = ({ items }) => {
  // Использование useMemo для оптимизации вычислений
  const expensiveValue = useMemo(() => {
    console.log('Выполнение дорогостоящего вычисления');
    return items.reduce((sum, item) => sum + item, 0);
  }, [items]); // Зависимости
  
  // Использование useMemo для создания объектов
  const config = useMemo(() => ({
    apiUrl: 'https://api.example.com',
    timeout: 5000,
    retries: 3
  }), []); // Пустой массив - объект создается только один раз
  
  return (
    <div>
      <p>Сумма: {expensiveValue}</p>
      <pre>{JSON.stringify(config, null, 2)}</pre>
    </div>
  );
};
```

## Пользовательские хуки

### Базовая типизация пользовательских хуков

```typescript
// useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}

// Использование
const MyComponent: React.FC = () => {
  const [name, setName] = useLocalStorage<string>('userName', '');
  
  return (
    <input 
      value={name} 
      onChange={(e) => setName(e.target.value)} 
      placeholder="Имя пользователя"
    />
  );
};
```

### Пользовательский хук с объектом результата

```typescript
interface ApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

export function useApi<T>(url: string): ApiState<T> {
  const [state, setState] = useState<ApiState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        setState({ data: null, loading: true, error: null });
        const response = await fetch(url);
        if (!response.ok) throw new Error('Ошибка запроса');
        const data = await response.json();
        setState({ data, loading: false, error: null });
      } catch (error) {
        setState({ 
          data: null, 
          loading: false, 
          error: (error as Error).message 
        });
      }
    };
    
    fetchData();
  }, [url]);
  
  return state;
}
```

### Хук с асинхронными операциями

```typescript
interface UseAsyncReturn<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  execute: () => Promise<void>;
}

export function useAsync<T>(
  asyncFunction: () => Promise<T>,
  deps: React.DependencyList = []
): UseAsyncReturn<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  
  const execute = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const result = await asyncFunction();
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, deps);
  
  useEffect(() => {
    execute();
  }, [execute]);
  
  return { data, loading, error, execute };
}

// Использование
const DataComponent: React.FC = () => {
  const { data, loading, error, execute } = useAsync<User[]>(
    () => fetch('/api/users').then(res => res.json()),
    []
  );
  
  if (loading) return <p>Загрузка...</p>;
  if (error) return <p>Ошибка: {error.message}</p>;
  
  return (
    <div>
      {data?.map(user => <div key={user.id}>{user.name}</div>)}
      <button onClick={execute}>Обновить</button>
    </div>
  );
};
```

## Лучшие практики

1. **Явно указывайте типы для useState**:
   ```typescript
   const [user, setUser] = useState<User | null>(null);
   ```

2. **Используйте типизированные action для useReducer**:
   ```typescript
   type Action = { type: 'ACTION'; payload: DataType };
   ```

3. **Используйте useCallback для функций, передаваемых в зависимости**:
   ```typescript
   const handleClick = useCallback(() => {
     // код
   }, [dependencies]);
   ```

4. **Типизируйте пользовательские хуки как отдельные функции**:
   ```typescript
   export function useCustomHook<T>(param: T): ReturnType<T> {
     // реализация
   }
   ```

5. **Используйте дженерики для универсальных хуков**:
   ```typescript
   export function useGenericHook<T>(initialValue: T) {
     // реализация
   }
   ```

## Связанные темы

- [[Типизация-React-компонентов]]
- [[Типизация-React-контекста]]
- [[React-паттерны-типов]]
- [[Типы-для-React-приложений]]

## Заключение

Правильная типизация React-хуков с использованием TypeScript помогает создавать более надежные и поддерживаемые приложения. Понимание типов, возвращаемых хуками, и правильное их использование позволяет избежать многих ошибок и улучшает качество кода.