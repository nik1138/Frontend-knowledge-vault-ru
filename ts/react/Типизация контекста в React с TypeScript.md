# Типизация контекста в React с TypeScript

Типизация контекста в React с TypeScript позволяет создавать строго типизированные и безопасные системы управления состоянием. Эта секция охватывает все аспекты типизации контекста в React-приложениях.

## Содержание

1. [Базовая типизация контекста](#базовая-типизация-контекста)
2. [Контекст с состоянием](#контекст-с-состоянием)
3. [Контекст с асинхронными операциями](#контекст-с-асинхронными-операциями)
4. [Вложенные контексты](#вложенные-контексты)
5. [Контекст с дженериками](#контекст-с-дженериками)
6. [Пользовательские хуки для контекста](#пользовательские-хуки-для-контекста)
7. [Лучшие практики](#лучшие-практики)
8. [Распространенные ошибки](#распространенные-ошибки)

## Базовая типизация контекста

### Простой контекст

```tsx
// Интерфейс для контекста темы
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// Создание контекста с типизацией
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Провайдер темы
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

// Пользовательский хук для использования контекста
export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
};

// Компонент, использующий контекст
const ThemedButton: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button 
      className={`btn btn-${theme}`}
      onClick={toggleTheme}
    >
      Toggle Theme
    </button>
  );
};

// Использование в приложении
const App: React.FC = () => {
  return (
    <ThemeProvider>
      <div className="app">
        <ThemedButton />
      </div>
    </ThemeProvider>
  );
};
```

### Контекст с примитивными значениями

```tsx
// Интерфейс для контекста локализации
interface LocaleContextType {
  locale: string;
  setLocale: (locale: string) => void;
  t: (key: string) => string;
}

// Создание контекста
const LocaleContext = createContext<LocaleContextType | undefined>(undefined);

// Словари переводов
const translations: Record<string, Record<string, string>> = {
  en: {
    hello: 'Hello',
    goodbye: 'Goodbye',
    welcome: 'Welcome to our application'
  },
  ru: {
    hello: 'Привет',
    goodbye: 'До свидания',
    welcome: 'Добро пожаловать в наше приложение'
  }
};

// Провайдер локализации
export const LocaleProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [locale, setLocale] = useState<string>('en');

  const t = (key: string): string => {
    return translations[locale]?.[key] || key;
  };

  return (
    <LocaleContext.Provider value={{ locale, setLocale, t }}>
      {children}
    </LocaleContext.Provider>
  );
};

// Пользовательский хук для локализации
export const useLocale = (): LocaleContextType => {
  const context = useContext(LocaleContext);
  
  if (context === undefined) {
    throw new Error('useLocale must be used within a LocaleProvider');
  }
  
  return context;
};

// Компонент с локализацией
const Greeting: React.FC = () => {
  const { locale, setLocale, t } = useLocale();
  
  return (
    <div>
      <h1>{t('hello')}</h1>
      <p>{t('welcome')}</p>
      
      <select value={locale} onChange={(e) => setLocale(e.target.value)}>
        <option value="en">English</option>
        <option value="ru">Русский</option>
      </select>
    </div>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <LocaleProvider>
      <div className="app">
        <Greeting />
      </div>
    </LocaleProvider>
  );
};
```

## Контекст с состоянием

### Контекст с объектом состояния

```tsx
// Интерфейсы для пользовательского контекста
interface User {
  id: number;
  name: string;
  email: string;
  role: 'user' | 'admin' | 'moderator';
}

interface UserContextType {
  user: User | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<boolean>;
  logout: () => void;
  updateUser: (userData: Partial<User>) => void;
}

// Создание контекста
const UserContext = createContext<UserContextType | undefined>(undefined);

// Провайдер пользователя
export const UserProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string): Promise<boolean> => {
    try {
      // Симуляция API вызова
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      if (response.ok) {
        const userData: User = await response.json();
        setUser(userData);
        return true;
      }
      
      return false;
    } catch (error) {
      console.error('Login error:', error);
      return false;
    }
  };

  const logout = () => {
    setUser(null);
    // Очистка токенов, сессий и т.д.
  };

  const updateUser = (userData: Partial<User>) => {
    if (user) {
      setUser({ ...user, ...userData });
    }
  };

  const isAuthenticated = !!user;

  return (
    <UserContext.Provider value={{ 
      user, 
      isAuthenticated, 
      login, 
      logout, 
      updateUser 
    }}>
      {children}
    </UserContext.Provider>
  );
};

// Пользовательский хук для пользователя
export const useUser = (): UserContextType => {
  const context = useContext(UserContext);
  
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  
  return context;
};

// Компонент профиля пользователя
const UserProfile: React.FC = () => {
  const { user, isAuthenticated, logout, updateUser } = useUser();
  
  if (!isAuthenticated || !user) {
    return <div>Please log in to view your profile</div>;
  }
  
  const handleUpdateName = () => {
    const newName = prompt('Enter new name:', user.name);
    if (newName) {
      updateUser({ name: newName });
    }
  };
  
  return (
    <div className="user-profile">
      <h2>User Profile</h2>
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
      <p>Role: {user.role}</p>
      
      <button onClick={handleUpdateName}>Update Name</button>
      <button onClick={logout}>Logout</button>
    </div>
  );
};

// Компонент формы входа
const LoginForm: React.FC = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const { login } = useUser();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const success = await login(email, password);
    if (success) {
      alert('Login successful!');
    } else {
      alert('Login failed!');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input
          type="password"
          id="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
};

// Использование
const App: React.FC = () => {
  const { isAuthenticated } = useUser();
  
  return (
    <UserProvider>
      <div className="app">
        {isAuthenticated ? <UserProfile /> : <LoginForm />}
      </div>
    </UserProvider>
  );
};
```

### Контекст с массивом состояния

```tsx
// Интерфейсы для контекста уведомлений
interface Notification {
  id: string;
  type: 'info' | 'success' | 'warning' | 'error';
  message: string;
  timestamp: Date;
}

interface NotificationsContextType {
  notifications: Notification[];
  addNotification: (notification: Omit<Notification, 'id' | 'timestamp'>) => void;
  removeNotification: (id: string) => void;
  clearNotifications: () => void;
}

// Создание контекста
const NotificationsContext = createContext<NotificationsContextType | undefined>(undefined);

// Провайдер уведомлений
export const NotificationsProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  const addNotification = (notification: Omit<Notification, 'id' | 'timestamp'>) => {
    const newNotification: Notification = {
      ...notification,
      id: Math.random().toString(36).substr(2, 9),
      timestamp: new Date()
    };
    
    setNotifications(prev => [...prev, newNotification]);
    
    // Автоматическое удаление уведомления через 5 секунд
    setTimeout(() => {
      removeNotification(newNotification.id);
    }, 5000);
  };

  const removeNotification = (id: string) => {
    setNotifications(prev => prev.filter(notification => notification.id !== id));
  };

  const clearNotifications = () => {
    setNotifications([]);
  };

  return (
    <NotificationsContext.Provider value={{ 
      notifications, 
      addNotification, 
      removeNotification, 
      clearNotifications 
    }}>
      {children}
    </NotificationsContext.Provider>
  );
};

// Пользовательский хук для уведомлений
export const useNotifications = (): NotificationsContextType => {
  const context = useContext(NotificationsContext);
  
  if (context === undefined) {
    throw new Error('useNotifications must be used within a NotificationsProvider');
  }
  
  return context;
};

// Компонент уведомлений
const Notifications: React.FC = () => {
  const { notifications, removeNotification } = useNotifications();
  
  if (notifications.length === 0) return null;
  
  return (
    <div className="notifications">
      {notifications.map(notification => (
        <div 
          key={notification.id} 
          className={`notification notification-${notification.type}`}
        >
          <span>{notification.message}</span>
          <button onClick={() => removeNotification(notification.id)}>×</button>
        </div>
      ))}
    </div>
  );
};

// Компонент для добавления уведомлений
const NotificationButton: React.FC = () => {
  const { addNotification } = useNotifications();
  
  const addRandomNotification = () => {
    const types: Array<'info' | 'success' | 'warning' | 'error'> = ['info', 'success', 'warning', 'error'];
    const messages = [
      'This is an info message',
      'Operation completed successfully',
      'Warning: This action cannot be undone',
      'Error: Something went wrong'
    ];
    
    const randomType = types[Math.floor(Math.random() * types.length)];
    const randomMessage = messages[Math.floor(Math.random() * messages.length)];
    
    addNotification({ type: randomType, message: randomMessage });
  };
  
  return (
    <button onClick={addRandomNotification}>
      Add Random Notification
    </button>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <NotificationsProvider>
      <div className="app">
        <Notifications />
        <NotificationButton />
      </div>
    </NotificationsProvider>
  );
};
```

## Контекст с асинхронными операциями

### Контекст с API интеграцией

```tsx
// Интерфейсы для контекста данных
interface DataItem {
  id: number;
  title: string;
  description: string;
  createdAt: string;
}

interface DataContextType {
  data: DataItem[];
  loading: boolean;
  error: string | null;
  fetchData: () => Promise<void>;
  addItem: (item: Omit<DataItem, 'id' | 'createdAt'>) => Promise<void>;
  updateItem: (id: number, updates: Partial<DataItem>) => Promise<void>;
  deleteItem: (id: number) => Promise<void>;
}

// Создание контекста
const DataContext = createContext<DataContextType | undefined>(undefined);

// Провайдер данных
export const DataProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [data, setData] = useState<DataItem[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);

  const fetchData = async (): Promise<void> => {
    try {
      setLoading(true);
      setError(null);
      
      // Симуляция API вызова
      const response = await fetch('/api/data');
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result: DataItem[] = await response.json();
      setData(result);
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Unknown error';
      setError(errorMessage);
      console.error('Fetch error:', errorMessage);
    } finally {
      setLoading(false);
    }
  };

  const addItem = async (item: Omit<DataItem, 'id' | 'createdAt'>): Promise<void> => {
    try {
      setLoading(true);
      setError(null);
      
      // Симуляция API вызова
      const response = await fetch('/api/data', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(item)
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const newItem: DataItem = await response.json();
      setData(prev => [...prev, newItem]);
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Unknown error';
      setError(errorMessage);
      console.error('Add item error:', errorMessage);
    } finally {
      setLoading(false);
    }
  };

  const updateItem = async (id: number, updates: Partial<DataItem>): Promise<void> => {
    try {
      setLoading(true);
      setError(null);
      
      // Симуляция API вызова
      const response = await fetch(`/api/data/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates)
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const updatedItem: DataItem = await response.json();
      setData(prev => prev.map(item => item.id === id ? updatedItem : item));
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Unknown error';
      setError(errorMessage);
      console.error('Update item error:', errorMessage);
    } finally {
      setLoading(false);
    }
  };

  const deleteItem = async (id: number): Promise<void> => {
    try {
      setLoading(true);
      setError(null);
      
      // Симуляция API вызова
      const response = await fetch(`/api/data/${id}`, {
        method: 'DELETE'
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      setData(prev => prev.filter(item => item.id !== id));
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Unknown error';
      setError(errorMessage);
      console.error('Delete item error:', errorMessage);
    } finally {
      setLoading(false);
    }
  };

  return (
    <DataContext.Provider value={{ 
      data, 
      loading, 
      error, 
      fetchData, 
      addItem, 
      updateItem, 
      deleteItem 
    }}>
      {children}
    </DataContext.Provider>
  );
};

// Пользовательский хук для данных
export const useData = (): DataContextType => {
  const context = useContext(DataContext);
  
  if (context === undefined) {
    throw new Error('useData must be used within a DataProvider');
  }
  
  return context;
};

// Компонент списка данных
const DataList: React.FC = () => {
  const { data, loading, error, fetchData } = useData();
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div className="data-list">
      <h2>Data Items</h2>
      <button onClick={fetchData}>Refresh</button>
      
      {data.length === 0 ? (
        <p>No data items found</p>
      ) : (
        <ul>
          {data.map(item => (
            <li key={item.id}>
              <h3>{item.title}</h3>
              <p>{item.description}</p>
              <small>Created: {new Date(item.createdAt).toLocaleDateString()}</small>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
};

// Компонент формы добавления данных
const AddDataForm: React.FC = () => {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const { addItem, loading } = useData();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (title.trim() && description.trim()) {
      await addItem({ title: title.trim(), description: description.trim() });
      setTitle('');
      setDescription('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="add-data-form">
      <h2>Add New Item</h2>
      
      <div>
        <label htmlFor="title">Title</label>
        <input
          type="text"
          id="title"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          required
        />
      </div>
      
      <div>
        <label htmlFor="description">Description</label>
        <textarea
          id="description"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          rows={3}
          required
        />
      </div>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Adding...' : 'Add Item'}
      </button>
    </form>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <DataProvider>
      <div className="app">
        <AddDataForm />
        <DataList />
      </div>
    </DataProvider>
  );
};
```

### Контекст с кэшированием

```tsx
// Интерфейсы для контекста кэширования
interface CacheEntry<T> {
  data: T;
  timestamp: number;
  expiresAt: number;
}

interface CacheContextType {
  get: <T>(key: string) => T | null;
  set: <T>(key: string, data: T, ttl?: number) => void;
  remove: (key: string) => void;
  clear: () => void;
  has: (key: string) => boolean;
  keys: () => string[];
}

// Создание контекста
const CacheContext = createContext<CacheContextType | undefined>(undefined);

// Провайдер кэширования
export const CacheProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [cache, setCache] = useState<Record<string, CacheEntry<any>>>({});

  const get = <T,>(key: string): T | null => {
    const entry = cache[key];
    
    if (!entry) return null;
    
    // Проверка истечения срока действия
    if (Date.now() > entry.expiresAt) {
      remove(key);
      return null;
    }
    
    return entry.data;
  };

  const set = <T,>(key: string, data: T, ttl: number = 5 * 60 * 1000): void => { // 5 минут по умолчанию
    const entry: CacheEntry<T> = {
      data,
      timestamp: Date.now(),
      expiresAt: Date.now() + ttl
    };
    
    setCache(prev => ({ ...prev, [key]: entry }));
  };

  const remove = (key: string): void => {
    setCache(prev => {
      const newCache = { ...prev };
      delete newCache[key];
      return newCache;
    });
  };

  const clear = (): void => {
    setCache({});
  };

  const has = (key: string): boolean => {
    const entry = cache[key];
    return !!entry && Date.now() <= entry.expiresAt;
  };

  const keys = (): string[] => {
    return Object.keys(cache).filter(key => has(key));
  };

  // Очистка устаревших записей
  useEffect(() => {
    const interval = setInterval(() => {
      const now = Date.now();
      setCache(prev => {
        const newCache = { ...prev };
        Object.keys(newCache).forEach(key => {
          if (now > newCache[key].expiresAt) {
            delete newCache[key];
          }
        });
        return newCache;
      });
    }, 60000); // Проверка каждую минуту

    return () => clearInterval(interval);
  }, []);

  return (
    <CacheContext.Provider value={{ get, set, remove, clear, has, keys }}>
      {children}
    </CacheContext.Provider>
  );
};

// Пользовательский хук для кэширования
export const useCache = (): CacheContextType => {
  const context = useContext(CacheContext);
  
  if (context === undefined) {
    throw new Error('useCache must be used within a CacheProvider');
  }
  
  return context;
};

// Компонент с кэшированием данных
const CachedDataComponent: React.FC<{ userId: number }> = ({ userId }) => {
  const [userData, setUserData] = useState<any>(null);
  const [loading, setLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);
  const cache = useCache();

  useEffect(() => {
    const fetchUserData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        // Проверка кэша
        const cachedData = cache.get(`user-${userId}`);
        if (cachedData) {
          setUserData(cachedData);
          setLoading(false);
          return;
        }
        
        // Загрузка данных
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        
        // Сохранение в кэш на 10 минут
        cache.set(`user-${userId}`, data, 10 * 60 * 1000);
        
        setUserData(data);
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : 'Unknown error';
        setError(errorMessage);
        console.error('Fetch user error:', errorMessage);
      } finally {
        setLoading(false);
      }
    };

    fetchUserData();
  }, [userId, cache]);

  if (loading) return <div>Loading user data...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!userData) return <div>No user data found</div>;

  return (
    <div className="user-data">
      <h2>User Data</h2>
      <p>Name: {userData.name}</p>
      <p>Email: {userData.email}</p>
      <p>Role: {userData.role}</p>
    </div>
  );
};

// Использование
const App: React.FC = () => {
  const [userId, setUserId] = useState<number>(1);
  
  return (
    <CacheProvider>
      <div className="app">
        <div>
          <label htmlFor="userId">User ID:</label>
          <input
            type="number"
            id="userId"
            value={userId}
            onChange={(e) => setUserId(Number(e.target.value))}
            min="1"
          />
        </div>
        
        <CachedDataComponent userId={userId} />
      </div>
    </CacheProvider>
  );
};
```

## Вложенные контексты

### Комбинирование нескольких контекстов

```tsx
// Интерфейсы для вложенных контекстов
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

interface UserContextType {
  user: { id: number; name: string; role: string } | null;
  login: (username: string, password: string) => Promise<boolean>;
  logout: () => void;
}

interface NotificationsContextType {
  notifications: Array<{ id: string; message: string; type: string }>;
  addNotification: (message: string, type: string) => void;
  removeNotification: (id: string) => void;
}

// Создание контекстов
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);
const UserContext = createContext<UserContextType | undefined>(undefined);
const NotificationsContext = createContext<NotificationsContextType | undefined>(undefined);

// Провайдеры
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
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

const UserProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<{ id: number; name: string; role: string } | null>(null);

  const login = async (username: string, password: string): Promise<boolean> => {
    // Симуляция аутентификации
    if (username === 'admin' && password === 'password') {
      setUser({ id: 1, name: 'Admin User', role: 'admin' });
      return true;
    }
    return false;
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
};

const NotificationsProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [notifications, setNotifications] = useState<Array<{ id: string; message: string; type: string }>>([]);

  const addNotification = (message: string, type: string) => {
    const id = Math.random().toString(36).substr(2, 9);
    setNotifications(prev => [...prev, { id, message, type }]);
    
    // Автоматическое удаление через 3 секунды
    setTimeout(() => {
      removeNotification(id);
    }, 3000);
  };

  const removeNotification = (id: string) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  };

  return (
    <NotificationsContext.Provider value={{ notifications, addNotification, removeNotification }}>
      {children}
    </NotificationsContext.Provider>
  );
};

// Пользовательские хуки
const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

const useUser = (): UserContextType => {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
};

const useNotifications = (): NotificationsContextType => {
  const context = useContext(NotificationsContext);
  if (context === undefined) {
    throw new Error('useNotifications must be used within a NotificationsProvider');
  }
  return context;
};

// Компоненты
const ThemeToggle: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? 'dark' : 'light'} theme
    </button>
  );
};

const UserStatus: React.FC = () => {
  const { user, logout } = useUser();
  const { addNotification } = useNotifications();
  
  const handleLogout = () => {
    logout();
    addNotification('You have been logged out', 'info');
  };
  
  if (!user) {
    return <div>Not logged in</div>;
  }
  
  return (
    <div>
      <span>Welcome, {user.name}!</span>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
};

const LoginForm: React.FC = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const { login } = useUser();
  const { addNotification } = useNotifications();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const success = await login(username, password);
    if (success) {
      addNotification('Login successful', 'success');
    } else {
      addNotification('Invalid credentials', 'error');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        <input
          type="text"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
        />
      </div>
      
      <div>
        <label>Password:</label>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
};

const NotificationList: React.FC = () => {
  const { notifications, removeNotification } = useNotifications();
  
  if (notifications.length === 0) return null;
  
  return (
    <div className="notifications">
      {notifications.map(notification => (
        <div key={notification.id} className={`notification ${notification.type}`}>
          <span>{notification.message}</span>
          <button onClick={() => removeNotification(notification.id)}>×</button>
        </div>
      ))}
    </div>
  );
};

// Главный компонент приложения
const App: React.FC = () => {
  const { theme } = useTheme();
  const { user } = useUser();
  
  return (
    <div className={`app ${theme}`}>
      <header>
        <ThemeToggle />
        <UserStatus />
      </header>
      
      <main>
        <NotificationList />
        
        {user ? (
          <div>Welcome to the dashboard, {user.name}!</div>
        ) : (
          <LoginForm />
        )}
      </main>
    </div>
  );
};

// Обертка для провайдеров
const AppWithProviders: React.FC = () => {
  return (
    <ThemeProvider>
      <UserProvider>
        <NotificationsProvider>
          <App />
        </NotificationsProvider>
      </UserProvider>
    </ThemeProvider>
  );
};

export default AppWithProviders;
```

### Композиция контекстов

```tsx
// Интерфейсы для композиции контекстов
interface AppContextType {
  theme: {
    current: 'light' | 'dark';
    toggle: () => void;
  };
  user: {
    data: { id: number; name: string; role: string } | null;
    login: (username: string, password: string) => Promise<boolean>;
    logout: () => void;
  };
  notifications: {
    list: Array<{ id: string; message: string; type: string }>;
    add: (message: string, type: string) => void;
    remove: (id: string) => void;
  };
}

// Создание единого контекста
const AppContext = createContext<AppContextType | undefined>(undefined);

// Провайдер композиции
export const AppProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  // Состояние темы
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  // Состояние пользователя
  const [user, setUser] = useState<{ id: number; name: string; role: string } | null>(null);
  const login = async (username: string, password: string): Promise<boolean> => {
    if (username === 'admin' && password === 'password') {
      setUser({ id: 1, name: 'Admin User', role: 'admin' });
      return true;
    }
    return false;
  };
  const logout = () => {
    setUser(null);
  };

  // Состояние уведомлений
  const [notifications, setNotifications] = useState<Array<{ id: string; message: string; type: string }>>([]);
  const addNotification = (message: string, type: string) => {
    const id = Math.random().toString(36).substr(2, 9);
    setNotifications(prev => [...prev, { id, message, type }]);
    setTimeout(() => {
      removeNotification(id);
    }, 3000);
  };
  const removeNotification = (id: string) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  };

  // Композиция контекста
  const contextValue: AppContextType = {
    theme: {
      current: theme,
      toggle: toggleTheme
    },
    user: {
      data: user,
      login,
      logout
    },
    notifications: {
      list: notifications,
      add: addNotification,
      remove: removeNotification
    }
  };

  return (
    <AppContext.Provider value={contextValue}>
      {children}
    </AppContext.Provider>
  );
};

// Пользовательский хук для композиции
export const useApp = (): AppContextType => {
  const context = useContext(AppContext);
  
  if (context === undefined) {
    throw new Error('useApp must be used within an AppProvider');
  }
  
  return context;
};

// Компоненты с использованием композиции
const ThemeToggle: React.FC = () => {
  const { theme } = useApp();
  
  return (
    <button onClick={theme.toggle}>
      Switch to {theme.current === 'light' ? 'dark' : 'light'} theme
    </button>
  );
};

const UserStatus: React.FC = () => {
  const { user, notifications } = useApp();
  
  const handleLogout = () => {
    user.logout();
    notifications.add('You have been logged out', 'info');
  };
  
  if (!user.data) {
    return <div>Not logged in</div>;
  }
  
  return (
    <div>
      <span>Welcome, {user.data.name}!</span>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
};

const LoginForm: React.FC = () => {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const { user, notifications } = useApp();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const success = await user.login(username, password);
    if (success) {
      notifications.add('Login successful', 'success');
    } else {
      notifications.add('Invalid credentials', 'error');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username:</label>
        <input
          type="text"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
        />
      </div>
      
      <div>
        <label>Password:</label>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
};

const NotificationList: React.FC = () => {
  const { notifications } = useApp();
  
  if (notifications.list.length === 0) return null;
  
  return (
    <div className="notifications">
      {notifications.list.map(notification => (
        <div key={notification.id} className={`notification ${notification.type}`}>
          <span>{notification.message}</span>
          <button onClick={() => notifications.remove(notification.id)}>×</button>
        </div>
      ))}
    </div>
  );
};

// Главный компонент приложения
const App: React.FC = () => {
  const { theme, user } = useApp();
  
  return (
    <div className={`app ${theme.current}`}>
      <header>
        <ThemeToggle />
        <UserStatus />
      </header>
      
      <main>
        <NotificationList />
        
        {user.data ? (
          <div>Welcome to the dashboard, {user.data.name}!</div>
        ) : (
          <LoginForm />
        )}
      </main>
    </div>
  );
};

// Использование
const AppWithProvider: React.FC = () => {
  return (
    <AppProvider>
      <App />
    </AppProvider>
  );
};

export default AppWithProvider;
```

## Контекст с дженериками

### Универсальный контекст

```tsx
// Интерфейсы для универсального контекста
interface GenericContextType<T> {
  data: T;
  update: (newData: T | ((prevData: T) => T)) => void;
  reset: () => void;
}

// Создание универсального контекста
const createGenericContext = <T,>(initialValue: T) => {
  const Context = createContext<GenericContextType<T> | undefined>(undefined);
  
  const Provider: React.FC<{ 
    children: React.ReactNode; 
    initialValue?: T 
  }> = ({ children, initialValue: initValue = initialValue }) => {
    const [data, setData] = useState<T>(initValue);
    
    const update = (newData: T | ((prevData: T) => T)) => {
      setData(newData);
    };
    
    const reset = () => {
      setData(initialValue);
    };
    
    return (
      <Context.Provider value={{ data, update, reset }}>
        {children}
      </Context.Provider>
    );
  };
  
  const useGeneric = (): GenericContextType<T> => {
    const context = useContext(Context);
    
    if (context === undefined) {
      throw new Error(`useGeneric must be used within a Provider`);
    }
    
    return context;
  };
  
  return { Provider, useGeneric };
};

// Создание конкретных контекстов
interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
}

interface ShoppingCart {
  items: Array<{ id: number; name: string; price: number; quantity: number }>;
  total: number;
}

const { Provider: PreferencesProvider, useGeneric: usePreferences } = 
  createGenericContext<UserPreferences>({
    theme: 'light',
    language: 'en',
    notifications: true
  });

const { Provider: CartProvider, useGeneric: useCart } = 
  createGenericContext<ShoppingCart>({
    items: [],
    total: 0
  });

// Компоненты для настроек
const ThemeToggle: React.FC = () => {
  const { data: preferences, update: updatePreferences } = usePreferences();
  
  const toggleTheme = () => {
    updatePreferences(prev => ({
      ...prev,
      theme: prev.theme === 'light' ? 'dark' : 'light'
    }));
  };
  
  return (
    <button onClick={toggleTheme}>
      Switch to {preferences.theme === 'light' ? 'dark' : 'light'} theme
    </button>
  );
};

const LanguageSelector: React.FC = () => {
  const { data: preferences, update: updatePreferences } = usePreferences();
  
  const changeLanguage = (language: string) => {
    updatePreferences(prev => ({
      ...prev,
      language
    }));
  };
  
  return (
    <select 
      value={preferences.language} 
      onChange={(e) => changeLanguage(e.target.value)}
    >
      <option value="en">English</option>
      <option value="ru">Русский</option>
      <option value="es">Español</option>
    </select>
  );
};

// Компоненты для корзины
const AddToCartButton: React.FC<{ 
  product: { id: number; name: string; price: number } 
}> = ({ product }) => {
  const { data: cart, update: updateCart } = useCart();
  
  const addToCart = () => {
    updateCart(prev => {
      const existingItem = prev.items.find(item => item.id === product.id);
      
      if (existingItem) {
        return {
          ...prev,
          items: prev.items.map(item =>
            item.id === product.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
          total: prev.total + product.price
        };
      } else {
        return {
          ...prev,
          items: [...prev.items, { ...product, quantity: 1 }],
          total: prev.total + product.price
        };
      }
    });
  };
  
  return (
    <button onClick={addToCart}>
      Add to Cart
    </button>
  );
};

const CartSummary: React.FC = () => {
  const { data: cart } = useCart();
  
  return (
    <div className="cart-summary">
      <h3>Cart Summary</h3>
      <p>Items: {cart.items.length}</p>
      <p>Total: ${cart.total.toFixed(2)}</p>
    </div>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <PreferencesProvider>
      <CartProvider>
        <div className="app">
          <header>
            <ThemeToggle />
            <LanguageSelector />
          </header>
          
          <main>
            <CartSummary />
            
            <div>
              <h2>Products</h2>
              <AddToCartButton product={{ id: 1, name: 'Product 1', price: 29.99 }} />
              <AddToCartButton product={{ id: 2, name: 'Product 2', price: 49.99 }} />
              <AddToCartButton product={{ id: 3, name: 'Product 3', price: 19.99 }} />
            </div>
          </main>
        </div>
      </CartProvider>
    </PreferencesProvider>
  );
};
```

### Дженерик контекст с асинхронными операциями

```tsx
// Интерфейсы для дженерик контекста с асинхронными операциями
interface AsyncDataContextType<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  fetch: () => Promise<void>;
  update: (newData: T) => Promise<void>;
  delete: () => Promise<void>;
  reset: () => void;
}

// Создание дженерик контекста с асинхронными операциями
const createAsyncDataContext = <T,>(
  fetchFn: () => Promise<T>,
  updateFn: (data: T) => Promise<T>,
  deleteFn: () => Promise<void>
) => {
  const Context = createContext<AsyncDataContextType<T> | undefined>(undefined);
  
  const Provider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
    const [data, setData] = useState<T | null>(null);
    const [loading, setLoading] = useState<boolean>(false);
    const [error, setError] = useState<string | null>(null);
    
    const fetch = async (): Promise<void> => {
      try {
        setLoading(true);
        setError(null);
        const result = await fetchFn();
        setData(result);
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : 'Unknown error';
        setError(errorMessage);
      } finally {
        setLoading(false);
      }
    };
    
    const update = async (newData: T): Promise<void> => {
      try {
        setLoading(true);
        setError(null);
        const result = await updateFn(newData);
        setData(result);
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : 'Unknown error';
        setError(errorMessage);
      } finally {
        setLoading(false);
      }
    };
    
    const deleteData = async (): Promise<void> => {
      try {
        setLoading(true);
        setError(null);
        await deleteFn();
        setData(null);
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : 'Unknown error';
        setError(errorMessage);
      } finally {
        setLoading(false);
      }
    };
    
    const reset = () => {
      setData(null);
      setLoading(false);
      setError(null);
    };
    
    return (
      <Context.Provider value={{ 
        data, 
        loading, 
        error, 
        fetch, 
        update, 
        delete: deleteData, 
        reset 
      }}>
        {children}
      </Context.Provider>
    );
  };
  
  const useAsyncData = (): AsyncDataContextType<T> => {
    const context = useContext(Context);
    
    if (context === undefined) {
      throw new Error(`useAsyncData must be used within a Provider`);
    }
    
    return context;
  };
  
  return { Provider, useAsyncData };
};

// Интерфейсы для данных
interface UserProfile {
  id: number;
  name: string;
  email: string;
  bio: string;
}

interface Product {
  id: number;
  title: string;
  price: number;
  description: string;
}

// Создание конкретных контекстов
const { Provider: UserProfileProvider, useAsyncData: useUserProfile } = 
  createAsyncDataContext<UserProfile>(
    async () => {
      // Симуляция API вызова
      const response = await fetch('/api/user/profile');
      if (!response.ok) throw new Error('Failed to fetch user profile');
      return response.json();
    },
    async (data) => {
      // Симуляция API вызова
      const response = await fetch('/api/user/profile', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      if (!response.ok) throw new Error('Failed to update user profile');
      return response.json();
    },
    async () => {
      // Симуляция API вызова
      const response = await fetch('/api/user/profile', { method: 'DELETE' });
      if (!response.ok) throw new Error('Failed to delete user profile');
    }
  );

const { Provider: ProductProvider, useAsyncData: useProduct } = 
  createAsyncDataContext<Product>(
    async () => {
      // Симуляция API вызова
      const response = await fetch('/api/products/1');
      if (!response.ok) throw new Error('Failed to fetch product');
      return response.json();
    },
    async (data) => {
      // Симуляция API вызова
      const response = await fetch('/api/products/1', {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      if (!response.ok) throw new Error('Failed to update product');
      return response.json();
    },
    async () => {
      // Симуляция API вызова
      const response = await fetch('/api/products/1', { method: 'DELETE' });
      if (!response.ok) throw new Error('Failed to delete product');
    }
  );

// Компоненты профиля пользователя
const UserProfileView: React.FC = () => {
  const { data: profile, loading, error, fetch } = useUserProfile();
  
  useEffect(() => {
    fetch();
  }, [fetch]);
  
  if (loading) return <div>Loading profile...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!profile) return <div>No profile data</div>;
  
  return (
    <div className="user-profile">
      <h2>{profile.name}</h2>
      <p>Email: {profile.email}</p>
      <p>Bio: {profile.bio}</p>
    </div>
  );
};

const EditProfileForm: React.FC = () => {
  const { data: profile, update, loading } = useUserProfile();
  const [formData, setFormData] = useState({ name: '', email: '', bio: '' });
  
  useEffect(() => {
    if (profile) {
      setFormData({
        name: profile.name,
        email: profile.email,
        bio: profile.bio
      });
    }
  }, [profile]);
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (profile) {
      await update({ ...profile, ...formData });
    }
  };
  
  if (!profile) return null;
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name:</label>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>Bio:</label>
        <textarea
          name="bio"
          value={formData.bio}
          onChange={handleChange}
          rows={3}
        />
      </div>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Saving...' : 'Save'}
      </button>
    </form>
  );
};

// Компоненты продукта
const ProductView: React.FC = () => {
  const { data: product, loading, error, fetch } = useProduct();
  
  useEffect(() => {
    fetch();
  }, [fetch]);
  
  if (loading) return <div>Loading product...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!product) return <div>No product data</div>;
  
  return (
    <div className="product">
      <h2>{product.title}</h2>
      <p>Price: ${product.price}</p>
      <p>{product.description}</p>
    </div>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <div className="app">
      <UserProfileProvider>
        <h1>User Profile</h1>
        <UserProfileView />
        <EditProfileForm />
      </UserProfileProvider>
      
      <ProductProvider>
        <h1>Product</h1>
        <ProductView />
      </ProductProvider>
    </div>
  );
};
```

## Пользовательские хуки для контекста

### Расширенный хук для контекста

```tsx
// Интерфейсы для расширенного хука контекста
interface ExtendedContextOptions<T> {
  initialValue: T;
  persistenceKey?: string;
  validator?: (value: T) => boolean;
  onUpdate?: (value: T) => void;
}

interface ExtendedContextResult<T> {
  value: T;
  setValue: (newValue: T | ((prevValue: T) => T)) => void;
  reset: () => void;
  isValid: boolean;
  persisted: boolean;
}

// Расширенный хук для контекста
function useExtendedContext<T>(
  options: ExtendedContextOptions<T>
): ExtendedContextResult<T> {
  const { initialValue, persistenceKey, validator, onUpdate } = options;
  
  // Инициализация значения из localStorage, если указан ключ
  const getInitialValue = (): T => {
    if (persistenceKey) {
      try {
        const stored = localStorage.getItem(persistenceKey);
        if (stored) {
          return JSON.parse(stored);
        }
      } catch (error) {
        console.error('Error reading from localStorage:', error);
      }
    }
    return initialValue;
  };
  
  const [value, setValue] = useState<T>(getInitialValue);
  const [isValid, setIsValid] = useState<boolean>(true);
  const [persisted, setPersisted] = useState<boolean>(false);
  
  // Сохранение в localStorage при изменении значения
  useEffect(() => {
    if (persistenceKey) {
      try {
        localStorage.setItem(persistenceKey, JSON.stringify(value));
        setPersisted(true);
      } catch (error) {
        console.error('Error writing to localStorage:', error);
        setPersisted(false);
      }
    }
    
    // Валидация значения
    if (validator) {
      setIsValid(validator(value));
    }
    
    // Вызов callback при обновлении
    if (onUpdate) {
      onUpdate(value);
    }
  }, [value, persistenceKey, validator, onUpdate]);
  
  const setContextValue = (newValue: T | ((prevValue: T) => T)) => {
    setValue(newValue);
  };
  
  const reset = () => {
    setValue(initialValue);
    if (persistenceKey) {
      try {
        localStorage.removeItem(persistenceKey);
        setPersisted(false);
      } catch (error) {
        console.error('Error removing from localStorage:', error);
      }
    }
  };
  
  return {
    value,
    setValue: setContextValue,
    reset,
    isValid,
    persisted
  };
}

// Создание контекста с расширенным хуком
interface Settings {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
  fontSize: number;
}

const SettingsContext = createContext<ExtendedContextResult<Settings> | undefined>(undefined);

const SettingsProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const settings = useExtendedContext<Settings>({
    initialValue: {
      theme: 'light',
      language: 'en',
      notifications: true,
      fontSize: 16
    },
    persistenceKey: 'app-settings',
    validator: (settings) => {
      return settings.fontSize >= 12 && settings.fontSize <= 24;
    },
    onUpdate: (settings) => {
      console.log('Settings updated:', settings);
      // Применение настроек к приложению
      document.documentElement.style.fontSize = `${settings.fontSize}px`;
    }
  });
  
  return (
    <SettingsContext.Provider value={settings}>
      {children}
    </SettingsContext.Provider>
  );
};

const useSettings = (): ExtendedContextResult<Settings> => {
  const context = useContext(SettingsContext);
  
  if (context === undefined) {
    throw new Error('useSettings must be used within a SettingsProvider');
  }
  
  return context;
};

// Компоненты настроек
const ThemeToggle: React.FC = () => {
  const { value: settings, setValue: setSettings } = useSettings();
  
  const toggleTheme = () => {
    setSettings(prev => ({
      ...prev,
      theme: prev.theme === 'light' ? 'dark' : 'light'
    }));
  };
  
  return (
    <button onClick={toggleTheme}>
      Switch to {settings.theme === 'light' ? 'dark' : 'light'} theme
    </button>
  );
};

const LanguageSelector: React.FC = () => {
  const { value: settings, setValue: setSettings } = useSettings();
  
  const changeLanguage = (language: string) => {
    setSettings(prev => ({
      ...prev,
      language
    }));
  };
  
  return (
    <select 
      value={settings.language} 
      onChange={(e) => changeLanguage(e.target.value)}
    >
      <option value="en">English</option>
      <option value="ru">Русский</option>
      <option value="es">Español</option>
    </select>
  );
};

const FontSizeControl: React.FC = () => {
  const { value: settings, setValue: setSettings, isValid } = useSettings();
  
  const changeFontSize = (size: number) => {
    setSettings(prev => ({
      ...prev,
      fontSize: size
    }));
  };
  
  return (
    <div>
      <label>
        Font Size: {settings.fontSize}px
        <input
          type="range"
          min="12"
          max="24"
          value={settings.fontSize}
          onChange={(e) => changeFontSize(Number(e.target.value))}
        />
      </label>
      {!isValid && (
        <span style={{ color: 'red' }}>Font size must be between 12 and 24</span>
      )}
    </div>
  );
};

const SettingsInfo: React.FC = () => {
  const { value: settings, persisted } = useSettings();
  
  return (
    <div>
      <h3>Current Settings</h3>
      <p>Theme: {settings.theme}</p>
      <p>Language: {settings.language}</p>
      <p>Notifications: {settings.notifications ? 'Enabled' : 'Disabled'}</p>
      <p>Font Size: {settings.fontSize}px</p>
      <p>Persisted: {persisted ? 'Yes' : 'No'}</p>
    </div>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <SettingsProvider>
      <div className="app">
        <header>
          <ThemeToggle />
          <LanguageSelector />
        </header>
        
        <main>
          <FontSizeControl />
          <SettingsInfo />
        </main>
      </div>
    </SettingsProvider>
  );
};
```

### Хук для контекста с историей изменений

```tsx
// Интерфейсы для контекста с историей
interface HistoryContextType<T> {
  current: T;
  history: T[];
  canUndo: boolean;
  canRedo: boolean;
  set: (value: T) => void;
  undo: () => void;
  redo: () => void;
  reset: () => void;
}

// Хук для контекста с историей изменений
function useHistoryContext<T>(initialValue: T, maxSize: number = 50): HistoryContextType<T> {
  const [history, setHistory] = useState<T[]>([initialValue]);
  const [currentIndex, setCurrentIndex] = useState<number>(0);
  
  const current = history[currentIndex];
  const canUndo = currentIndex > 0;
  const canRedo = currentIndex < history.length - 1;
  
  const set = (value: T) => {
    const newHistory = [...history.slice(0, currentIndex + 1), value];
    
    // Ограничение размера истории
    if (newHistory.length > maxSize) {
      newHistory.shift();
      setCurrentIndex(prev => Math.max(0, prev - 1));
    }
    
    setHistory(newHistory);
    setCurrentIndex(newHistory.length - 1);
  };
  
  const undo = () => {
    if (canUndo) {
      setCurrentIndex(prev => prev - 1);
    }
  };
  
  const redo = () => {
    if (canRedo) {
      setCurrentIndex(prev => prev + 1);
    }
  };
  
  const reset = () => {
    setHistory([initialValue]);
    setCurrentIndex(0);
  };
  
  return {
    current,
    history,
    canUndo,
    canRedo,
    set,
    undo,
    redo,
    reset
  };
}

// Создание контекста с историей изменений
interface DocumentState {
  title: string;
  content: string;
  lastModified: Date;
}

const DocumentContext = createContext<HistoryContextType<DocumentState> | undefined>(undefined);

const DocumentProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const documentHistory = useHistoryContext<DocumentState>({
    title: 'Untitled Document',
    content: '',
    lastModified: new Date()
  });
  
  return (
    <DocumentContext.Provider value={documentHistory}>
      {children}
    </DocumentContext.Provider>
  );
};

const useDocument = (): HistoryContextType<DocumentState> => {
  const context = useContext(DocumentContext);
  
  if (context === undefined) {
    throw new Error('useDocument must be used within a DocumentProvider');
  }
  
  return context;
};

// Компоненты редактора документов
const DocumentEditor: React.FC = () => {
  const { current: document, set } = useDocument();
  const [title, setTitle] = useState(document.title);
  const [content, setContent] = useState(document.content);
  
  useEffect(() => {
    setTitle(document.title);
    setContent(document.content);
  }, [document]);
  
  const handleSave = () => {
    set({
      title,
      content,
      lastModified: new Date()
    });
  };
  
  return (
    <div className="document-editor">
      <div>
        <label>Title:</label>
        <input
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
        />
      </div>
      
      <div>
        <label>Content:</label>
        <textarea
          value={content}
          onChange={(e) => setContent(e.target.value)}
          rows={10}
        />
      </div>
      
      <button onClick={handleSave}>Save</button>
    </div>
  );
};

const DocumentViewer: React.FC = () => {
  const { current: document } = useDocument();
  
  return (
    <div className="document-viewer">
      <h2>{document.title}</h2>
      <div>{document.content}</div>
      <small>Last modified: {document.lastModified.toLocaleString()}</small>
    </div>
  );
};

const HistoryControls: React.FC = () => {
  const { canUndo, canRedo, undo, redo, reset, history, currentIndex } = useDocument();
  
  return (
    <div className="history-controls">
      <button onClick={undo} disabled={!canUndo}>
        Undo
      </button>
      <button onClick={redo} disabled={!canRedo}>
        Redo
      </button>
      <button onClick={reset}>
        Reset
      </button>
      
      <div>
        <p>History: {history.length} states</p>
        <p>Current: {currentIndex + 1}</p>
      </div>
    </div>
  );
};

const HistoryTimeline: React.FC = () => {
  const { history, currentIndex, set } = useDocument();
  
  return (
    <div className="history-timeline">
      <h3>History Timeline</h3>
      <div className="timeline">
        {history.map((state, index) => (
          <div
            key={index}
            className={`timeline-item ${index === currentIndex ? 'current' : ''}`}
            onClick={() => set(state)}
          >
            <div className="timestamp">
              {state.lastModified.toLocaleTimeString()}
            </div>
            <div className="title">{state.title || 'Untitled'}</div>
          </div>
        ))}
      </div>
    </div>
  );
};

// Использование
const App: React.FC = () => {
  return (
    <DocumentProvider>
      <div className="app">
        <header>
          <h1>Document Editor with History</h1>
        </header>
        
        <main>
          <div className="editor-section">
            <DocumentEditor />
            <HistoryControls />
          </div>
          
          <div className="viewer-section">
            <DocumentViewer />
            <HistoryTimeline />
          </div>
        </main>
      </div>
    </DocumentProvider>
  );
};
```

## Лучшие практики

### 1. Явная типизация контекста

```tsx
// Хорошо: явная типизация
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Плохо: неявная типизация
const ThemeContext = createContext(undefined); // any тип
```

### 2. Проверка контекста в хуках

```tsx
// Хорошо: проверка контекста
const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
};

// Плохо: отсутствие проверки
const useTheme = () => {
  return useContext(ThemeContext); // Может вернуть undefined
};
```

### 3. Использование дженериков для универсальных контекстов

```tsx
// Хорошо: универсальный контекст с дженериками
interface GenericContextType<T> {
  data: T;
  update: (newData: T | ((prevData: T) => T)) => void;
}

const createGenericContext = <T,>(initialValue: T) => {
  const Context = createContext<GenericContextType<T> | undefined>(undefined);
  // Реализация
};

// Плохо: дублирование кода для каждого типа
const StringContext = createContext<string | undefined>(undefined);
const NumberContext = createContext<number | undefined>(undefined);
const BooleanContext = createContext<boolean | undefined>(undefined);
```

### 4. Оптимизация производительности

```tsx
// Хорошо: мемоизация значений контекста
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  const contextValue = useMemo(() => ({
    theme,
    toggleTheme
  }), [theme, toggleTheme]);
  
  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
};

// Плохо: создание нового объекта при каждом рендере
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  // Новый объект создается при каждом рендере
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```

## Распространенные ошибки

### 1. Неявная типизация any

```tsx
// Проблема: неявный any
const ThemeContext = createContext(); // any тип

function BadComponent() {
  const theme = useContext(ThemeContext); // any тип
  return <div className={theme} />; // Нет проверки типов
}

// Исправление: явная типизация
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

### 2. Отсутствие проверки контекста

```tsx
// Проблема: отсутствие проверки
const useTheme = () => {
  return useContext(ThemeContext); // Может вернуть undefined
};

function BadComponent() {
  const { theme, toggleTheme } = useTheme(); // Ошибка при undefined
  return <button onClick={toggleTheme}>{theme}</button>;
}

// Исправление: проверка контекста
const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  
  return context;
};

function GoodComponent() {
  const { theme, toggleTheme } = useTheme();
  return <button onClick={toggleTheme}>{theme}</button>;
}
```

### 3. Неправильная типизация дженериков

```tsx
// Проблема: неправильная типизация дженериков
const createGenericContext = <T,>(initialValue: T) => {
  const Context = createContext<T | undefined>(undefined);
  // Реализация
};

// Исправление: правильная типизация дженериков
interface GenericContextType<T> {
  data: T;
  update: (newData: T | ((prevData: T) => T)) => void;
}

const createGenericContext = <T,>(initialValue: T) => {
  const Context = createContext<GenericContextType<T> | undefined>(undefined);
  // Реализация
};
```

### 4. Проблемы с производительностью

```tsx
// Проблема: создание нового объекта при каждом рендере
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  // Новый объект создается при каждом рендере
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// Исправление: мемоизация значений
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = useCallback(() => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  }, []);
  
  const contextValue = useMemo(() => ({
    theme,
    toggleTheme
  }), [theme, toggleTheme]);
  
  return (
    <ThemeContext.Provider value={contextValue}>
      {children}
    </ThemeContext.Provider>
  );
};
```

## Связи с другими концепциями

- [[TypeScript с React|TypeScript с React]] - Основы использования TypeScript в React
- [[Типизация хуков React в TypeScript|Хуки]] - Типизация хуков в React
- [[Типизация компонентов React в TypeScript|Компоненты]] - Типизация компонентов с контекстом
- [[ts/advanced/generics|Дженерики]] - Использование дженериков в контексте
- [[ts/advanced/utility-types|Вспомогательные типы]] - Utility types для типизации контекста

## Рекомендации по изучению

1. Начните с базовой типизации простых контекстов
2. Освойте контекст с состоянием и асинхронными операциями
3. Практикуйтесь в создании вложенных контекстов
4. Изучите использование дженериков в контексте
5. Освойте создание пользовательских хуков для контекста
6. Практикуйтесь в оптимизации производительности контекста
7. Изучите лучшие практики типизации контекста
8. Освойте обработку распространенных ошибок

## Следующие шаги

- [[ts/react/testing|Тестирование]] - Тестирование React-компонентов с TypeScript
- [[ts/performance/advanced-performance-optimization|Оптимизация производительности]] - Оптимизация React-приложений с TypeScript
- [[Тестирование в TypeScript|Тестирование]] - Общие принципы тестирования TypeScript-приложений
- [[Обзор-архитектуры-TypeScript|Архитектура]] - Архитектурные паттерны для TypeScript-приложений
