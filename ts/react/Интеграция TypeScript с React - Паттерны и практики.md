# Интеграция TypeScript с React: Паттерны и практики

React и TypeScript образуют мощную комбинацию для создания надежных и масштабируемых веб-приложений. TypeScript предоставляет статическую типизацию для React-компонентов, что позволяет обнаруживать ошибки на этапе разработки и улучшает опыт разработки.

## Основы типизации React компонентов

### 1. Функциональные компоненты

```typescript
import React from 'react';

// Простой функциональный компонент с типами
interface GreetingProps {
  name: string;
  age?: number; // Необязательное свойство
  onGreet?: () => void; // Необязательный обработчик
}

const Greeting: React.FC<GreetingProps> = ({ name, age = 0, onGreet }) => {
  return (
    <div onClick={onGreet}>
      <h1>Привет, {name}!</h1>
      {age > 0 && <p>Возраст: {age}</p>}
    </div>
  );
};

// Альтернативный способ без React.FC
const GreetingAlt = ({ name, age = 0, onGreet }: GreetingProps) => {
  return (
    <div onClick={onGreet}>
      <h1>Привет, {name}!</h1>
      {age > 0 && <p>Возраст: {age}</p>}
    </div>
  );
};
```

### 2. Компоненты с дженериками

```typescript
// Компонент с дженериком
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Использование
interface User {
  id: number;
  name: string;
  email: string;
}

const UserList: React.FC<{ users: User[] }> = ({ users }) => {
  return (
    <List
      items={users}
      keyExtractor={(user) => user.id}
      renderItem={(user) => (
        <div>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      )}
    />
  );
};
```

## Типизация хуков React

### 1. useState с типами

```typescript
import { useState } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

const UserProfile: React.FC = () => {
  // Типизация состояния с интерфейсом
  const [user, setUser] = useState<User | null>(null);
  
  // Типизация состояния с объединением
  const [loading, setLoading] = useState<boolean>(false);
  
  // Типизация состояния с перечислением
  type Status = 'idle' | 'loading' | 'success' | 'error';
  const [status, setStatus] = useState<Status>('idle');
  
  // Типизация состояния с массивом
  const [tags, setTags] = useState<string[]>([]);
  
  const fetchUser = async () => {
    setLoading(true);
    setStatus('loading');
    
    try {
      // Симуляция API вызова
      const userData: User = { id: 1, name: 'John', email: 'john@example.com' };
      setUser(userData);
      setStatus('success');
    } catch (error) {
      setStatus('error');
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      {loading && <p>Загрузка...</p>}
      {user && (
        <div>
          <h2>{user.name}</h2>
          <p>{user.email}</p>
        </div>
      )}
      <button onClick={fetchUser} disabled={loading}>
        Загрузить профиль
      </button>
    </div>
  );
};
```

### 2. useEffect и работа с зависимостями

```typescript
import { useEffect, useState } from 'react';

interface Post {
  id: number;
  title: string;
  content: string;
}

const PostList: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [userId, setUserId] = useState<number>(1);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchPosts = async () => {
      try {
        setError(null);
        // Симуляция API вызова
        const response = await fetch(`/api/users/${userId}/posts`);
        if (!response.ok) throw new Error('Failed to fetch posts');
        const data: Post[] = await response.json();
        setPosts(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'An error occurred');
      }
    };

    fetchPosts();
  }, [userId]); // userId в зависимостях

  return (
    <div>
      <select value={userId} onChange={(e) => setUserId(Number(e.target.value))}>
        <option value={1}>User 1</option>
        <option value={2}>User 2</option>
        <option value={3}>User 3</option>
      </select>
      
      {error && <p style={{ color: 'red' }}>Ошибка: {error}</p>}
      
      <ul>
        {posts.map(post => (
          <li key={post.id}>
            <h3>{post.title}</h3>
            <p>{post.content}</p>
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### 3. useRef с типами

```typescript
import { useRef, useEffect } from 'react';

const FocusInput: React.FC = () => {
  // Типизация для DOM элемента
  const inputRef = useRef<HTMLInputElement>(null);
  
  // Типизация для значений, не вызывающих перерендер
  const renderCountRef = useRef<number>(0);
  
  // Типизация для функций
  const callbackRef = useRef<(() => void) | null>(null);

  useEffect(() => {
    renderCountRef.current += 1;
    console.log(`Компонент перерендерен ${renderCountRef.current} раз`);
    
    // Фокус на инпут
    if (inputRef.current) {
      inputRef.current.focus();
    }
  });

  const handleClick = () => {
    if (inputRef.current) {
      alert(`Текущее значение: ${inputRef.current.value}`);
    }
  };

  return (
    <div>
      <input 
        ref={inputRef} 
        type="text" 
        placeholder="Фокус на этом инпуте"
      />
      <button onClick={handleClick}>Показать значение</button>
    </div>
  );
};
```

### 4. useReducer с типами

```typescript
import { useReducer } from 'react';

// Определение типов для состояния
interface State {
  count: number;
  step: number;
  loading: boolean;
}

// Определение типов для действий
type Action = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_STEP', payload: number }
  | { type: 'RESET' }
  | { type: 'LOADING_START' }
  | { type: 'LOADING_END' };

// Начальное состояние
const initialState: State = {
  count: 0,
  step: 1,
  loading: false
};

// Редьюсер с типами
const counterReducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + state.step };
    case 'DECREMENT':
      return { ...state, count: state.count - state.step };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    case 'RESET':
      return { ...state, count: 0 };
    case 'LOADING_START':
      return { ...state, loading: true };
    case 'LOADING_END':
      return { ...state, loading: false };
    default:
      return state;
  }
};

const Counter: React.FC = () => {
  const [state, dispatch] = useReducer(counterReducer, initialState);

  const incrementAsync = () => {
    dispatch({ type: 'LOADING_START' });
    setTimeout(() => {
      dispatch({ type: 'INCREMENT' });
      dispatch({ type: 'LOADING_END' });
    }, 1000);
  };

  return (
    <div>
      <h2>Счетчик: {state.count}</h2>
      <p>Шаг: {state.step}</p>
      
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={incrementAsync} disabled={state.loading}>
        {state.loading ? 'Загрузка...' : '+'} (асинхронно)
      </button>
      
      <div>
        <label>
          Шаг:
          <input
            type="number"
            value={state.step}
            onChange={(e) => dispatch({ 
              type: 'SET_STEP', 
              payload: Number(e.target.value) 
            })}
          />
        </label>
      </div>
      
      <button onClick={() => dispatch({ type: 'RESET' })}>Сброс</button>
    </div>
  );
};
```

## Продвинутая типизация компонентов

### 1. Компоненты высшего порядка (HOC)

```typescript
import React, { ComponentType } from 'react';

// Интерфейс для пропсов, добавляемых HOC
interface WithLoadingProps {
  loading: boolean;
}

// HOC для добавления состояния загрузки
function withLoading<T extends object>(
  WrappedComponent: ComponentType<T>
): ComponentType<Omit<T, keyof WithLoadingProps>> {
  return (props: Omit<T, keyof WithLoadingProps>) => {
    const [loading] = React.useState(false);
    
    if (loading) {
      return <div>Загрузка...</div>;
    }
    
    return <WrappedComponent {...props as T} loading={false} />;
  };
}

// Компонент, который будет обернут
interface UserCardProps {
  user: { name: string; email: string };
  loading: boolean;
}

const UserCard: React.FC<UserCardProps> = ({ user, loading }) => {
  if (loading) return <div>Загрузка пользователя...</div>;
  
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
};

// Обернутый компонент
const UserCardWithLoading = withLoading(UserCard);

// Использование
const App: React.FC = () => {
  const user = { name: 'John', email: 'john@example.com' };
  
  return (
    <UserCardWithLoading user={user} />
  );
};
```

### 2. Render Props и Children как функция

```typescript
interface DataProviderProps<T> {
  url: string;
  children: (data: T | null, loading: boolean, error: string | null) => React.ReactNode;
}

interface ApiResponse<T> {
  data: T;
  loading: boolean;
  error: string | null;
}

const DataProvider = <T extends {}>({ url, children }: DataProviderProps<T>) => {
  const [data, setData] = React.useState<T | null>(null);
  const [loading, setLoading] = React.useState(true);
  const [error, setError] = React.useState<string | null>(null);

  React.useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        // Симуляция API вызова
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch');
        
        const result: T = await response.json();
        setData(result);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'An error occurred');
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return <>{children(data, loading, error)}</>;
};

// Использование
const UserListWithProvider: React.FC = () => {
  return (
    <DataProvider<{ users: { id: number; name: string }[] }>
      url="/api/users"
    >
      {(users, loading, error) => {
        if (loading) return <div>Загрузка...</div>;
        if (error) return <div>Ошибка: {error}</div>;
        if (!users) return <div>Нет данных</div>;

        return (
          <ul>
            {users.users.map(user => (
              <li key={user.id}>{user.name}</li>
            ))}
          </ul>
        );
      }}
    </DataProvider>
  );
};
```

### 3. Compound Components

```typescript
import React, { createContext, useContext } from 'react';

interface TabsContextType {
  activeTab: string;
  setActiveTab: (tabId: string) => void;
}

const TabsContext = createContext<TabsContextType | undefined>(undefined);

const useTabs = () => {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('useTabs must be used within a TabsProvider');
  }
  return context;
};

interface TabsProps {
  children: React.ReactNode;
  defaultActiveTab?: string;
}

const Tabs: React.FC<TabsProps> = ({ children, defaultActiveTab = 'tab1' }) => {
  const [activeTab, setActiveTab] = React.useState(defaultActiveTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">
        {children}
      </div>
    </TabsContext.Provider>
  );
};

const TabList: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="tab-list">{children}</div>;
};

const Tab: React.FC<{ id: string; children: React.ReactNode }> = ({ id, children }) => {
  const { activeTab, setActiveTab } = useTabs();
  const isActive = activeTab === id;

  return (
    <button
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
};

const TabPanels: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="tab-panels">{children}</div>;
};

const TabPanel: React.FC<{ id: string; children: React.ReactNode }> = ({ id, children }) => {
  const { activeTab } = useTabs();
  const isVisible = activeTab === id;

  if (!isVisible) return null;

  return <div className="tab-panel">{children}</div>;
};

// Использование
const ExampleTabs: React.FC = () => {
  return (
    <Tabs defaultActiveTab="profile">
      <TabList>
        <Tab id="profile">Профиль</Tab>
        <Tab id="settings">Настройки</Tab>
        <Tab id="help">Помощь</Tab>
      </TabList>
      
      <TabPanels>
        <TabPanel id="profile">
          <h3>Профиль пользователя</h3>
          <p>Информация о пользователе...</p>
        </TabPanel>
        <TabPanel id="settings">
          <h3>Настройки</h3>
          <p>Настройки приложения...</p>
        </TabPanel>
        <TabPanel id="help">
          <h3>Помощь</h3>
          <p>Помощь и поддержка...</p>
        </TabPanel>
      </TabPanels>
    </Tabs>
  );
};
```

## Типизация событий и форм

### 1. Типизация событий

```typescript
import React from 'react';

const EventHandlingExample: React.FC = () => {
  const [formData, setFormData] = React.useState({
    name: '',
    email: '',
    message: ''
  });

  // Типизация изменения инпута
  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  // Типизация отправки формы
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log('Отправка формы:', formData);
  };

  // Типизация клика
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Кнопка нажата');
  };

  // Типизация фокуса
  const handleFocus = (e: React.FocusEvent<HTMLInputElement>) => {
    console.log('Инпут в фокусе:', e.target.name);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleInputChange}
          onFocus={handleFocus}
          placeholder="Имя"
        />
      </div>
      
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleInputChange}
          onFocus={handleFocus}
          placeholder="Email"
        />
      </div>
      
      <div>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleInputChange}
          placeholder="Сообщение"
        />
      </div>
      
      <button type="submit">Отправить</button>
      <button type="button" onClick={handleClick}>Дополнительная кнопка</button>
    </form>
  );
};
```

### 2. Кастомные хуки с типами

```typescript
import { useState, useEffect, useCallback } from 'react';

// Кастомный хук для работы с API
interface ApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useApi<T>(url: string): ApiState<T> {
  const [state, setState] = useState<ApiState<T>>({
    data: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    const fetchData = async () => {
      try {
        setState(prev => ({ ...prev, loading: true, error: null }));
        
        const response = await fetch(url);
        if (!response.ok) throw new Error('Network response was not ok');
        
        const data: T = await response.json();
        setState({ data, loading: false, error: null });
      } catch (error) {
        setState({
          data: null,
          loading: false,
          error: error instanceof Error ? error.message : 'An error occurred'
        });
      }
    };

    fetchData();
  }, [url]);

  return state;
}

// Кастомный хук для работы с localStorage
function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = useCallback((value: T) => {
    try {
      setStoredValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  }, [key]);

  return [storedValue, setValue];
}

// Использование кастомных хуков
interface User {
  id: number;
  name: string;
  email: string;
}

const UserProfileWithHooks: React.FC<{ userId: number }> = ({ userId }) => {
  const { data: user, loading, error } = useApi<User>(`/api/users/${userId}`);
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');

  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error}</div>;
  if (!user) return <div>Пользователь не найден</div>;

  return (
    <div className={`profile ${theme}`}>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Переключить тему
      </button>
    </div>
  );
};
```

## Практические примеры из реальной разработки

### 1. Таблица с сортировкой и фильтрацией

```typescript
import React, { useState, useMemo } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'moderator';
  createdAt: Date;
}

interface TableProps {
  data: User[];
  columns: Column<User>[];
}

interface Column<T> {
  key: keyof T;
  title: string;
  render?: (value: any, record: T) => React.ReactNode;
  sortable?: boolean;
}

const UserTable: React.FC<TableProps> = ({ data, columns }) => {
  const [sortConfig, setSortConfig] = useState<{ key: string; direction: 'asc' | 'desc' } | null>(null);
  const [filter, setFilter] = useState('');

  const sortedData = useMemo(() => {
    if (!sortConfig) return data;

    return [...data].sort((a, b) => {
      const aValue = a[sortConfig.key as keyof User];
      const bValue = b[sortConfig.key as keyof User];

      if (aValue < bValue) {
        return sortConfig.direction === 'asc' ? -1 : 1;
      }
      if (aValue > bValue) {
        return sortConfig.direction === 'asc' ? 1 : -1;
      }
      return 0;
    });
  }, [data, sortConfig]);

  const filteredData = useMemo(() => {
    if (!filter) return sortedData;
    
    return sortedData.filter(user => 
      Object.values(user).some(value => 
        String(value).toLowerCase().includes(filter.toLowerCase())
      )
    );
  }, [sortedData, filter]);

  const handleSort = (key: string) => {
    let direction: 'asc' | 'desc' = 'asc';
    
    if (sortConfig && sortConfig.key === key && sortConfig.direction === 'asc') {
      direction = 'desc';
    }
    
    setSortConfig({ key, direction });
  };

  return (
    <div>
      <input
        type="text"
        placeholder="Поиск..."
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
      />
      
      <table>
        <thead>
          <tr>
            {columns.map(column => (
              <th 
                key={String(column.key)} 
                onClick={column.sortable ? () => handleSort(String(column.key)) : undefined}
                style={{ cursor: column.sortable ? 'pointer' : 'default' }}
              >
                {column.title}
                {sortConfig?.key === String(column.key) && (
                  <span>{sortConfig.direction === 'asc' ? ' ↑' : ' ↓'}</span>
                )}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {filteredData.map((item, index) => (
            <tr key={item.id}>
              {columns.map(column => (
                <td key={String(column.key)}>
                  {column.render 
                    ? column.render(item[column.key], item) 
                    : String(item[column.key])
                  }
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};

// Использование
const UserManagement: React.FC = () => {
  const users: User[] = [
    { id: 1, name: 'John Doe', email: 'john@example.com', role: 'admin', createdAt: new Date() },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'user', createdAt: new Date() },
    { id: 3, name: 'Bob Johnson', email: 'bob@example.com', role: 'moderator', createdAt: new Date() }
  ];

  const columns: Column<User>[] = [
    { key: 'name', title: 'Имя', sortable: true },
    { key: 'email', title: 'Email', sortable: true },
    { key: 'role', title: 'Роль', sortable: true, 
      render: (role) => <span className={`role-${role}`}>{role}</span> 
    },
    { 
      key: 'createdAt', 
      title: 'Дата создания', 
      sortable: true,
      render: (date) => new Date(date).toLocaleDateString()
    }
  ];

  return <UserTable data={users} columns={columns} />;
};
```

### 2. Форма с валидацией

```typescript
import React, { useState, useEffect } from 'react';

interface FormErrors {
  [key: string]: string;
}

interface FormData {
  name: string;
  email: string;
  age: number;
  consent: boolean;
}

const RegistrationForm: React.FC = () => {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
    age: 18,
    consent: false
  });
  
  const [errors, setErrors] = useState<FormErrors>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validateField = (name: keyof FormData, value: any): string => {
    switch (name) {
      case 'name':
        return value.trim().length < 2 ? 'Имя должно содержать не менее 2 символов' : '';
      case 'email':
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return !emailRegex.test(value) ? 'Неверный формат email' : '';
      case 'age':
        return value < 18 || value > 120 ? 'Возраст должен быть от 18 до 120' : '';
      case 'consent':
        return !value ? 'Необходимо дать согласие' : '';
      default:
        return '';
    }
  };

  const validateForm = (): boolean => {
    const newErrors: FormErrors = {};
    let isValid = true;

    for (const key in formData) {
      const error = validateField(key as keyof FormData, formData[key as keyof FormData]);
      if (error) {
        newErrors[key] = error;
        isValid = false;
      }
    }

    setErrors(newErrors);
    return isValid;
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value, type, checked } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : type === 'number' ? Number(value) : value
    }));

    // Валидация поля при изменении
    const error = validateField(name as keyof FormData, type === 'checkbox' ? checked : value);
    setErrors(prev => ({
      ...prev,
      [name]: error
    }));
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!validateForm()) return;
    
    setIsSubmitting(true);
    
    try {
      // Симуляция API вызова
      await new Promise(resolve => setTimeout(resolve, 1000));
      console.log('Форма отправлена:', formData);
      alert('Регистрация успешна!');
    } catch (error) {
      console.error('Ошибка регистрации:', error);
      alert('Ошибка регистрации');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>
          Имя:
          <input
            type="text"
            name="name"
            value={formData.name}
            onChange={handleInputChange}
          />
        </label>
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <label>
          Email:
          <input
            type="email"
            name="email"
            value={formData.email}
            onChange={handleInputChange}
          />
        </label>
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <label>
          Возраст:
          <input
            type="number"
            name="age"
            value={formData.age}
            onChange={handleInputChange}
          />
        </label>
        {errors.age && <span className="error">{errors.age}</span>}
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="consent"
            checked={formData.consent}
            onChange={handleInputChange}
          />
          Согласие на обработку данных
        </label>
        {errors.consent && <span className="error">{errors.consent}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Отправка...' : 'Зарегистрироваться'}
      </button>
    </form>
  );
};
```

## Ошибки и лучшие практики

### 1. Распространенные ошибки

```typescript
// ОШИБКА: Неиспользование строгой типизации
// const BadComponent = ({ data }) => { // Не типизированы пропсы
//   return <div>{data.name}</div>; // Ошибка во время выполнения если data.name нет
// };

// ХОРОШО: Правильная типизация
interface GoodComponentProps {
  user: {
    name: string;
    email: string;
  };
}

const GoodComponent: React.FC<GoodComponentProps> = ({ user }) => {
  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
};

// ОШИБКА: Неправильная типизация useRef
// const inputRef = useRef(); // Тип unknown

// ХОРОШО: Правильная типизация
const CorrectInputRef = () => {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Фокус на инпут</button>
    </div>
  );
};
```

### 2. Лучшие практики

```typescript
// 1. Используйте интерфейсы для определения пропсов
interface UserProfileProps {
  user: {
    id: number;
    name: string;
    email: string;
  };
  onEdit: (userId: number) => void;
  onDelete: (userId: number) => void;
}

// 2. Используйте React.FC только когда нужно
// В большинстве случаев достаточно типизации пропсов напрямую
const ComponentWithDirectTyping: React.FC<UserProfileProps> = ({ user, onEdit, onDelete }) => {
  // компонент
  return <div>Пример</div>;
};

// 3. Используйте дженерики для универсальных компонентов
function GenericList<T>({
  items,
  renderItem,
  keyExtractor
}: {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// 4. Используйте utility types для создания производных типов
type RequiredUserFields = Required<Pick<UserProfileProps['user'], 'name' | 'email'>>;
type OptionalUserFields = Partial<Omit<UserProfileProps['user'], 'name' | 'email'>>;
```

## Лучшие практики

1. **Используйте строгую типизацию** - всегда типизируйте пропсы, состояние и возвращаемые значения
2. **Создавайте переиспользуемые типы** - определяйте общие интерфейсы и типы в отдельных файлах
3. **Используйте дженерики для универсальных компонентов** - для создания гибких, переиспользуемых компонентов
4. **Типизируйте события** - используйте соответствующие типы для обработчиков событий
5. **Используйте кастомные хуки** - для извлечения логики и улучшения переиспользуемости
6. **Документируйте сложные типы** - особенно когда они используют дженерики или условные типы

## Связи с другими концепциями

- [[ts/type-system/type-inference|Вывод типов]] - как TypeScript выводит типы в React компонентах
- [[ts/generics/TS Generics|Обобщения]] - использование дженериков в React компонентах
- [[Утилиты типов TypeScript|Утилиты типов]] - для создания производных типов
- [[Типизация компонентов React в TypeScript]] - основы React компонентов
- [[Типизация хуков React в TypeScript]] - использование хуков в TypeScript
- [[testing/react-testing]] - тестирование TypeScript React компонентов