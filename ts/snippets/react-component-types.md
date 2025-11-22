---
aliases: ["Типы для компонентов React", "TS React Component Types"]
tags: [typescript, react, components, frontend]
---

# Типы для компонентов React

## Введение

Типизация компонентов React обеспечивает безопасность типов, улучшает автодополнение в IDE и уменьшает количество ошибок. В этом разделе рассмотрим различные паттерны типизации компонентов React с использованием TypeScript.

## Базовые типы для компонентов

### Функциональные компоненты

```typescript
import React from 'react';

// Простой функциональный компонент
const Welcome: React.FC<{ name: string }> = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

// Альтернативный способ с явным указанием типа
const WelcomeAlt: React.FunctionComponent<{ name: string }> = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

// Рекомендуемый способ (без React.FC)
interface WelcomeProps {
  name: string;
  age?: number;
}

const WelcomeRecommended: React.FC<WelcomeProps> = ({ name, age }) => {
  return (
    <div>
      <h1>Hello, {name}!</h1>
      {age && <p>Age: {age}</p>}
    </div>
  );
};
```

### Компоненты с дочерними элементами

```typescript
interface CardProps {
  title: string;
  children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ title, children }) => {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="content">
        {children}
      </div>
    </div>
  );
};

// Использование
const App = () => {
  return (
    <Card title="My Card">
      <p>This is the card content</p>
    </Card>
  );
};
```

## Продвинутые паттерны

### Компоненты с условным рендерингом

```typescript
interface UserCardProps {
  user: {
    id: number;
    name: string;
    email: string;
    avatar?: string;
  };
  showEmail?: boolean;
  actions?: React.ReactNode;
}

const UserCard: React.FC<UserCardProps> = ({ user, showEmail = false, actions }) => {
  return (
    <div className="user-card">
      <div className="avatar">
        {user.avatar ? (
          <img src={user.avatar} alt={user.name} />
        ) : (
          <div className="default-avatar">{user.name.charAt(0)}</div>
        )}
      </div>
      <div className="info">
        <h3>{user.name}</h3>
        {showEmail && <p>{user.email}</p>}
      </div>
      {actions && <div className="actions">{actions}</div>}
    </div>
  );
};
```

### Типизация хуков

```typescript
import { useState, useEffect, useCallback } from 'react';

interface UseCounterOptions {
  initialValue?: number;
  step?: number;
}

const useCounter = (options: UseCounterOptions = {}) => {
  const { initialValue = 0, step = 1 } = options;
  const [count, setCount] = useState<number>(initialValue);
  
  const increment = useCallback(() => setCount(c => c + step), [step]);
  const decrement = useCallback(() => setCount(c => c - step), [step]);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  
  return {
    count,
    increment,
    decrement,
    reset,
    setCount
  } as const; // "as const" предотвращает мутацию возвращаемого объекта
};

// Использование хука
const Counter: React.FC = () => {
  const { count, increment, decrement, reset } = useCounter({ initialValue: 10, step: 2 });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
};
```

### Типизация событий

```typescript
interface FormProps {
  onSubmit: (data: { email: string; password: string }) => void;
}

const LoginForm: React.FC<FormProps> = ({ onSubmit }) => {
  const [formData, setFormData] = useState({ email: '', password: '' });
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
};
```

## Компоненты с дженериками

### Дженерик-компоненты

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  className?: string;
}

// Дженерик-компонент для отображения списков
function List<T>(props: ListProps<T>) {
  const { items, renderItem, className } = props;
  
  return (
    <ul className={className}>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item, index)}</li>
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
    <List<User>
      items={users}
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

### Компоненты с условными типами

```typescript
interface ConditionalComponentProps<T> {
  condition: boolean;
  component: React.ComponentType<T>;
  fallback?: React.ComponentType;
  props: T;
}

function ConditionalComponent<T extends Record<string, any>>({
  condition,
  component: Component,
  fallback: Fallback,
  props
}: ConditionalComponentProps<T>) {
  if (condition) {
    return <Component {...props} />;
  }
  
  if (Fallback) {
    return <Fallback />;
  }
  
  return null;
}

// Использование
const LoadingSpinner = () => <div>Loading...</div>;
const UserProfile = ({ name }: { name: string }) => <div>Hello, {name}!</div>;

const AppWithConditional: React.FC<{ user?: User }> = ({ user }) => {
  return (
    <ConditionalComponent
      condition={!!user}
      component={UserProfile}
      fallback={LoadingSpinner}
      props={{ name: user?.name || '' }}
    />
  );
};
```

## Практические примеры

### Типизация формы с валидацией

```typescript
interface FormField<T> {
  value: T;
  error?: string;
  isValid: boolean;
}

interface FormState<T> {
  [K: string]: FormField<any>;
}

interface FormProps<T> {
  initialValues: T;
  onSubmit: (values: T) => void;
  validator?: (values: T) => Partial<Record<keyof T, string>>;
  children: (formState: {
    values: T;
    errors: Partial<Record<keyof T, string>>;
    handleChange: (name: keyof T, value: any) => void;
    handleSubmit: (e: React.FormEvent) => void;
    isValid: boolean;
  }) => React.ReactNode;
}

function Form<T extends Record<string, any>>({
  initialValues,
  onSubmit,
  validator,
  children
}: FormProps<T>) {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  
  const handleChange = (name: keyof T, value: any) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Очистка ошибки при изменении поля
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: undefined }));
    }
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (validator) {
      const validationErrors = validator(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length === 0) {
        onSubmit(values);
      }
    } else {
      onSubmit(values);
    }
  };
  
  const isValid = Object.keys(errors).length === 0;
  
  return children({
    values,
    errors,
    handleChange,
    handleSubmit,
    isValid
  });
}

// Использование формы
interface LoginFormValues {
  email: string;
  password: string;
}

const MyForm: React.FC = () => {
  return (
    <Form<LoginFormValues>
      initialValues={{ email: '', password: '' }}
      validator={(values) => {
        const errors: Partial<Record<keyof LoginFormValues, string>> = {};
        
        if (!values.email) {
          errors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(values.email)) {
          errors.email = 'Email is invalid';
        }
        
        if (!values.password) {
          errors.password = 'Password is required';
        } else if (values.password.length < 8) {
          errors.password = 'Password must be at least 8 characters';
        }
        
        return errors;
      }}
      onSubmit={(values) => console.log('Form submitted:', values)}
    >
      {({ values, errors, handleChange, handleSubmit, isValid }) => (
        <form onSubmit={handleSubmit}>
          <div>
            <input
              type="email"
              value={values.email}
              onChange={(e) => handleChange('email', e.target.value)}
              placeholder="Email"
            />
            {errors.email && <span>{errors.email}</span>}
          </div>
          
          <div>
            <input
              type="password"
              value={values.password}
              onChange={(e) => handleChange('password', e.target.value)}
              placeholder="Password"
            />
            {errors.password && <span>{errors.password}</span>}
          </div>
          
          <button type="submit" disabled={!isValid}>
            Submit
          </button>
        </form>
      )}
    </Form>
  );
};
```

### Типизация кастомных хуков

```typescript
// Типизация кастомного хука для работы с API
interface UseApiOptions {
  initialData?: any;
  autoLoad?: boolean;
}

interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

function useApi<T>(
  url: string,
  options: UseApiOptions = {}
): UseApiResult<T> {
  const { initialData, autoLoad = true } = options;
  const [data, setData] = useState<T | null>(initialData);
  const [loading, setLoading] = useState<boolean>(autoLoad);
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
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    if (autoLoad) {
      fetchData();
    }
  }, [autoLoad, fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// Использование хука
interface UserApiResponse {
  id: number;
  name: string;
  email: string;
}

const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
  const { data: user, loading, error, refetch } = useApi<UserApiResponse>(
    `/api/users/${userId}`
  );
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return user ? (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <button onClick={refetch}>Refresh</button>
    </div>
  ) : null;
};
```

## Заключение

Правильная типизация компонентов React улучшает качество кода, обеспечивает безопасность типов и упрощает рефакторинг. Используйте эти паттерны как основу для создания типизированных компонентов в ваших React-приложениях.

См. также: [[vue-component-types]] | [[api-type-definitions]] | [[validation-utilities]]