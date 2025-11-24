---
aliases: [React Context Typing, Типизированный React контекст]
tags: [typescript, react, context, typing]
---

# Типизация React-контекста

## Обзор

React Context - это мощный механизм для передачи данных через дерево компонентов без необходимости передавать пропсы на каждом уровне. При использовании TypeScript важно правильно типизировать контекст, чтобы обеспечить безопасность типов и избежать ошибок при работе с данными контекста.

## Базовая типизация контекста

### Создание типизированного контекста

```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

// Определяем тип для данных контекста
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
  fontSize: number;
  setFontSize: (size: number) => void;
}

// Создаем контекст с типом и значением по умолчанию
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Кастомный хук для использования контекста
export const useTheme = (): ThemeContextType => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};

// Интерфейс для пропсов провайдера
interface ThemeProviderProps {
  children: ReactNode;
}

// Компонент-провайдер
export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const [fontSize, setFontSize] = useState(16);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  // Значение контекста
  const value: ThemeContextType = {
    theme,
    toggleTheme,
    fontSize,
    setFontSize
  };

  return (
    <ThemeContext.Provider value={value}>
      <div className={`app ${theme}`} style={{ fontSize }}>
        {children}
      </div>
    </ThemeContext.Provider>
  );
};
```

### Использование контекста

```typescript
const Header: React.FC = () => {
  const { theme, toggleTheme, fontSize, setFontSize } = useTheme();

  return (
    <header className={`header ${theme}`}>
      <h1>Текущая тема: {theme}</h1>
      <button onClick={toggleTheme}>Переключить тему</button>
      <div>
        <label>Размер шрифта: {fontSize}px</label>
        <input 
          type="range" 
          min="12" 
          max="24" 
          value={fontSize} 
          onChange={(e) => setFontSize(Number(e.target.value))}
        />
      </div>
    </header>
  );
};

const App: React.FC = () => {
  return (
    <ThemeProvider>
      <Header />
      <main>
        <p>Содержимое приложения</p>
      </main>
    </ThemeProvider>
  );
};
```

## Контекст с асинхронными данными

```typescript
import React, { createContext, useContext, useState, useEffect, ReactNode } from 'react';

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserContextType {
  user: User | null;
  loading: boolean;
  error: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  updateUser: (updates: Partial<User>) => void;
}

const UserContext = createContext<UserContextType | undefined>(undefined);

export const useUser = (): UserContextType => {
  const context = useContext(UserContext);
  if (context === undefined) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
};

interface UserProviderProps {
  children: ReactNode;
}

export const UserProvider: React.FC<UserProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Загрузка данных пользователя при инициализации
    const fetchUser = async () => {
      try {
        setLoading(true);
        // Симуляция API-запроса
        const response = await fetch('/api/current-user');
        if (response.ok) {
          const userData = await response.json();
          setUser(userData);
        }
      } catch (err) {
        setError((err as Error).message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, []);

  const login = async (email: string, password: string) => {
    try {
      setLoading(true);
      setError(null);
      // Симуляция API-запроса
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      if (!response.ok) throw new Error('Неверные учетные данные');
      
      const userData = await response.json();
      setUser(userData);
    } catch (err) {
      setError((err as Error).message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  const logout = () => {
    setUser(null);
    // Очистка данных аутентификации
  };

  const updateUser = (updates: Partial<User>) => {
    if (user) {
      setUser({ ...user, ...updates });
    }
  };

  const value: UserContextType = {
    user,
    loading,
    error,
    login,
    logout,
    updateUser
  };

  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
};
```

## Комбинирование нескольких контекстов

```typescript
import React, { createContext, useContext, ReactNode } from 'react';

// Определяем типы для каждого контекста
interface LanguageContextType {
  language: string;
  setLanguage: (lang: string) => void;
  translations: Record<string, string>;
}

interface NotificationContextType {
  notifications: string[];
  addNotification: (message: string) => void;
  removeNotification: (index: number) => void;
}

// Создаем контексты
const LanguageContext = createContext<LanguageContextType | undefined>(undefined);
const NotificationContext = createContext<NotificationContextType | undefined>(undefined);

// Хуки для каждого контекста
export const useLanguage = (): LanguageContextType => {
  const context = useContext(LanguageContext);
  if (context === undefined) {
    throw new Error('useLanguage must be used within a LanguageProvider');
  }
  return context;
};

export const useNotifications = (): NotificationContextType => {
  const context = useContext(NotificationContext);
  if (context === undefined) {
    throw new Error('useNotifications must be used within a NotificationProvider');
  }
  return context;
};

// Компонент, объединяющий несколько провайдеров
interface AppProvidersProps {
  children: ReactNode;
}

export const AppProviders: React.FC<AppProvidersProps> = ({ children }) => {
  return (
    <LanguageProvider>
      <NotificationProvider>
        {children}
      </NotificationProvider>
    </LanguageProvider>
  );
};
```

## Контекст с дженериками

```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

// Дженерик для универсального провайдера состояния
interface StateProviderProps<T> {
  children: ReactNode;
  initialState: T;
  name: string; // Имя для отладки
}

// Тип для значений провайдера
interface StateContextType<T> {
  state: T;
  setState: React.Dispatch<React.SetStateAction<T>>;
  updateState: (updater: (prevState: T) => T) => void;
}

// Функция для создания типизированного провайдера
function createStateProvider<T>(name: string) {
  const StateContext = createContext<StateContextType<T> | undefined>(undefined);

  const Provider: React.FC<StateProviderProps<T>> = ({ 
    children, 
    initialState 
  }) => {
    const [state, setState] = useState<T>(initialState);

    const updateState = (updater: (prevState: T) => T) => {
      setState(prevState => updater(prevState));
    };

    const value: StateContextType<T> = {
      state,
      setState,
      updateState
    };

    return (
      <StateContext.Provider value={value}>
        {children}
      </StateContext.Provider>
    );
  };

  const useStateContext = (): StateContextType<T> => {
    const context = useContext(StateContext);
    if (context === undefined) {
      throw new Error(`use${name} must be used within a ${name}Provider`);
    }
    return context;
  };

  return [Provider, useStateContext] as const;
}

// Создание конкретных провайдеров
const [CartProvider, useCart] = createStateProvider<{ items: string[]; total: number }>('Cart');
const [UserProfileProvider, useUserProfile] = createStateProvider<{ name: string; avatar: string }>('UserProfile');
```

## Контекст для управления формой

```typescript
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface FormContextType<T> {
  values: T;
  errors: Partial<Record<keyof T, string>>;
  handleChange: (name: keyof T, value: any) => void;
  handleSubmit: (onSubmit: (values: T) => void) => void;
  setFieldError: (name: keyof T, error: string) => void;
  resetForm: () => void;
}

interface FormProviderProps<T> {
  children: ReactNode;
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
}

function createFormProvider<T>() {
  const FormContext = createContext<FormContextType<T> | undefined>(undefined);

  const FormProvider: React.FC<FormProviderProps<T>> = ({ 
    children, 
    initialValues,
    validate
  }) => {
    const [values, setValues] = useState<T>(initialValues);
    const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});

    const handleChange = (name: keyof T, value: any) => {
      setValues(prev => ({ ...prev, [name]: value }));
      
      // Очистка ошибки при изменении поля
      if (errors[name]) {
        setErrors(prev => {
          const newErrors = { ...prev };
          delete newErrors[name];
          return newErrors;
        });
      }
    };

    const setFieldError = (name: keyof T, error: string) => {
      setErrors(prev => ({ ...prev, [name]: error }));
    };

    const handleSubmit = (onSubmit: (values: T) => void) => {
      if (validate) {
        const validationErrors = validate(values);
        setErrors(validationErrors);
        
        if (Object.keys(validationErrors).length > 0) {
          return; // Не отправляем форму при ошибках валидации
        }
      }
      
      onSubmit(values);
    };

    const resetForm = () => {
      setValues(initialValues);
      setErrors({});
    };

    const contextValue: FormContextType<T> = {
      values,
      errors,
      handleChange,
      handleSubmit,
      setFieldError,
      resetForm
    };

    return (
      <FormContext.Provider value={contextValue}>
        {children}
      </FormContext.Provider>
    );
  };

  const useForm = (): FormContextType<T> => {
    const context = useContext(FormContext);
    if (context === undefined) {
      throw new Error('useForm must be used within a FormProvider');
    }
    return context;
  };

  return [FormProvider, useForm] as const;
}

// Использование для конкретной формы
interface LoginFormValues {
  email: string;
  password: string;
}

const [LoginFormProvider, useLoginForm] = createFormProvider<LoginFormValues>();

const LoginForm: React.FC = () => {
  const { values, errors, handleChange, handleSubmit } = useLoginForm();

  const validate = (values: LoginFormValues) => {
    const errors: Partial<Record<keyof LoginFormValues, string>> = {};
    
    if (!values.email) {
      errors.email = 'Email обязателен';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = 'Email некорректен';
    }
    
    if (!values.password) {
      errors.password = 'Пароль обязателен';
    } else if (values.password.length < 6) {
      errors.password = 'Пароль должен быть не менее 6 символов';
    }
    
    return errors;
  };

  return (
    <LoginFormProvider 
      initialValues={{ email: '', password: '' }} 
      validate={validate}
    >
      <form onSubmit={(e) => {
        e.preventDefault();
        handleSubmit((values) => {
          console.log('Отправка формы:', values);
        });
      }}>
        <div>
          <input
            type="email"
            value={values.email}
            onChange={(e) => handleChange('email', e.target.value)}
            placeholder="Email"
          />
          {errors.email && <span className="error">{errors.email}</span>}
        </div>
        
        <div>
          <input
            type="password"
            value={values.password}
            onChange={(e) => handleChange('password', e.target.value)}
            placeholder="Пароль"
          />
          {errors.password && <span className="error">{errors.password}</span>}
        </div>
        
        <button type="submit">Войти</button>
      </form>
    </LoginFormProvider>
  );
};
```

## Контекст с кэшированием данных

```typescript
import React, { createContext, useContext, useState, useCallback, ReactNode } from 'react';

interface CacheContextType<T> {
  get: (key: string) => T | undefined;
  set: (key: string, value: T) => void;
  remove: (key: string) => void;
  clear: () => void;
  has: (key: string) => boolean;
}

function createCacheProvider<T>() {
  const CacheContext = createContext<CacheContextType<T> | undefined>(undefined);

  const CacheProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
    const [cache, setCache] = useState<Map<string, T>>(new Map());

    const get = useCallback((key: string) => {
      return cache.get(key);
    }, [cache]);

    const set = useCallback((key: string, value: T) => {
      setCache(prev => new Map(prev).set(key, value));
    }, []);

    const remove = useCallback((key: string) => {
      setCache(prev => {
        const newMap = new Map(prev);
        newMap.delete(key);
        return newMap;
      });
    }, []);

    const clear = useCallback(() => {
      setCache(new Map());
    }, []);

    const has = useCallback((key: string) => {
      return cache.has(key);
    }, [cache]);

    const value: CacheContextType<T> = {
      get,
      set,
      remove,
      clear,
      has
    };

    return (
      <CacheContext.Provider value={value}>
        {children}
      </CacheContext.Provider>
    );
  };

  const useCache = (): CacheContextType<T> => {
    const context = useContext(CacheContext);
    if (context === undefined) {
      throw new Error('useCache must be used within a CacheProvider');
    }
    return context;
  };

  return [CacheProvider, useCache] as const;
}

// Пример использования для кэширования пользовательских данных
interface UserData {
  id: number;
  name: string;
  email: string;
}

const [UserCacheProvider, useUserCache] = createCacheProvider<UserData>();

const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
  const cache = useUserCache();
  const [loading, setLoading] = useState(false);

  const loadUser = async () => {
    if (cache.has(userId.toString())) {
      // Возвращаем кэшированные данные
      return cache.get(userId.toString());
    }

    setLoading(true);
    try {
      const response = await fetch(`/api/users/${userId}`);
      const userData = await response.json();
      cache.set(userId.toString(), userData);
      return userData;
    } finally {
      setLoading(false);
    }
  };

  // Использование данных
  return (
    <div>
      {loading ? <p>Загрузка...</p> : <p>{cache.get(userId.toString())?.name}</p>}
    </div>
  );
};
```

## Лучшие практики

1. **Всегда проверяйте наличие контекста**:
   ```typescript
   const context = useContext(MyContext);
   if (context === undefined) {
     throw new Error('useMyContext must be used within a MyProvider');
   }
   ```

2. **Используйте интерфейсы для определения типов контекста**:
   ```typescript
   interface MyContextType {
     // определение типа
   }
   ```

3. **Разделяйте контексты по функциональности**:
   Создавайте отдельные контексты для разных аспектов приложения (тема, пользователь, настройки и т.д.)

4. **Используйте дженерики для универсальных провайдеров**:
   Это позволяет создавать переиспользуемые компоненты-провайдеры.

5. **Используйте React.memo для дочерних компонентов**:
   Это помогает избежать ненужных перерендеров при изменении контекста.

## Связанные темы

- [[Типизация-React-компонентов]]
- [[Типизация-React-хуков]]
- [[React-паттерны-типов]]
- [[HOC-с-типами]]
- [[Рендер-пропсы-с-типами]]

## Заключение

Типизация React-контекста с использованием TypeScript обеспечивает безопасность типов при передаче данных через дерево компонентов. Правильная организация контекста помогает создавать масштабируемые и поддерживаемые приложения.