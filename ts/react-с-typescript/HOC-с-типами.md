---
aliases: [Typed HOC, Higher-Order Components with Types, Типизированные HOC]
tags: [typescript, react, hoc, patterns, typing]
---

# HOC с типами

## Обзор

Higher-Order Components (HOC) - это паттерн в React, при котором функция принимает компонент и возвращает новый компонент с дополнительной функциональностью. При использовании TypeScript важно правильно типизировать HOC, чтобы сохранить информацию о типах и обеспечить безопасность типов.

## Базовая структура HOC

### Простой HOC с типами

```typescript
import React, { ComponentType } from 'react';

// Определяем тип для HOC
function withLoading<TProps extends {}>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<TProps & { loading: boolean }> {
  return function WithLoading(props: TProps & { loading: boolean }) {
    if (props.loading) {
      return <div>Загрузка...</div>;
    }
    
    return <WrappedComponent {...props} />;
  };
}

// Использование
interface UserCardProps {
  name: string;
  email: string;
}

const UserCard: React.FC<UserCardProps> = ({ name, email }) => (
  <div>
    <h3>{name}</h3>
    <p>{email}</p>
  </div>
);

const UserCardWithLoading = withLoading(UserCard);

// Теперь UserCardWithLoading требует пропс loading
const App: React.FC = () => {
  return (
    <UserCardWithLoading 
      name="Иван Иванов" 
      email="ivan@example.com" 
      loading={false} 
    />
  );
};
```

### HOC с инъекцией пропсов

```typescript
import React, { ComponentType } from 'react';

// Определяем тип для пропсов, которые будут инжектированы
interface WithAuthProps {
  isAuthenticated: boolean;
  userRole: string;
}

// HOC для проверки аутентификации
function withAuth<TProps extends WithAuthProps>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<Omit<TProps, keyof WithAuthProps>> {
  return function WithAuth(props: Omit<TProps, keyof WithAuthProps>) {
    // Здесь мы можем получить данные аутентификации из какого-то источника
    const { isAuthenticated, userRole } = useAuth(); // предполагаемый хук
    
    return (
      <WrappedComponent 
        {...props as TProps} 
        isAuthenticated={isAuthenticated} 
        userRole={userRole} 
      />
    );
  };
}

// Компонент, который ожидает пропсы аутентификации
interface ProtectedComponentProps extends WithAuthProps {
  title: string;
}

const ProtectedComponent: React.FC<ProtectedComponentProps> = ({ 
  title, 
  isAuthenticated, 
  userRole 
}) => {
  if (!isAuthenticated) {
    return <div>Доступ запрещен</div>;
  }
  
  return (
    <div>
      <h1>{title}</h1>
      <p>Роль пользователя: {userRole}</p>
    </div>
  );
};

// Использование HOC
const EnhancedProtectedComponent = withAuth(ProtectedComponent);

// Теперь при использовании не нужно передавать isAuthenticated и userRole
const App: React.FC = () => {
  return <EnhancedProtectedComponent title="Защищенная страница" />;
};
```

## Продвинутая типизация HOC

### Использование дженериков для сохранения типов

```typescript
import React, { ComponentType } from 'react';

// Более сложный пример с сохранением типов
function withData<TData, TProps extends {}>(
  fetchData: () => Promise<TData>,
  LoadingComponent: ComponentType = () => <div>Загрузка...</div>,
  ErrorComponent: ComponentType<{ error: string }> = ({ error }) => <div>Ошибка: {error}</div>
) {
  return function withDataHOC(
    WrappedComponent: ComponentType<TProps & { data: TData }>
  ): ComponentType<TProps> {
    return class WithData extends React.Component<TProps, { 
      data: TData | null; 
      loading: boolean; 
      error: string | null 
    }> {
      constructor(props: TProps) {
        super(props);
        this.state = {
          data: null,
          loading: true,
          error: null
        };
      }

      async componentDidMount() {
        try {
          const data = await fetchData();
          this.setState({ data, loading: false });
        } catch (error) {
          this.setState({ 
            error: (error as Error).message, 
            loading: false 
          });
        }
      }

      render() {
        const { data, loading, error } = this.state;
        
        if (loading) return <LoadingComponent />;
        if (error) return <ErrorComponent error={error} />;
        if (!data) return null;
        
        return <WrappedComponent {...this.props} data={data} />;
      }
    };
  };
}

// Использование
interface Post {
  id: number;
  title: string;
  content: string;
}

interface PostListProps {
  data: Post[];
}

const PostList: React.FC<PostListProps> = ({ data }) => (
  <div>
    {data.map(post => (
      <div key={post.id}>
        <h3>{post.title}</h3>
        <p>{post.content}</p>
      </div>
    ))}
  </div>
);

const PostListWithData = withData<Post[], {}>(
  () => fetch('/api/posts').then(res => res.json())
)(PostList);
```

### HOC с условным рендерингом

```typescript
import React, { ComponentType, ReactNode } from 'react';

interface WithConditionProps {
  condition: boolean;
  fallback?: ReactNode;
}

function withCondition<TProps extends WithConditionProps>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<Omit<TProps, keyof WithConditionProps> & WithConditionProps> {
  return function WithCondition(props: Omit<TProps, keyof WithConditionProps> & WithConditionProps) {
    const { condition, fallback, ...restProps } = props;
    
    if (!condition) {
      return fallback || null;
    }
    
    return <WrappedComponent {...restProps as TProps} condition={condition} fallback={fallback} />;
  };
}

// Пример использования
interface DashboardProps {
  userRole: string;
  condition: boolean;
  fallback?: ReactNode;
}

const Dashboard: React.FC<DashboardProps> = ({ userRole }) => (
  <div>
    <h1>Панель управления</h1>
    <p>Роль: {userRole}</p>
  </div>
);

const ConditionalDashboard = withCondition(Dashboard);

const App: React.FC = () => {
  const userRole = 'admin';
  const hasAccess = userRole === 'admin';
  
  return (
    <ConditionalDashboard
      userRole={userRole}
      condition={hasAccess}
      fallback={<div>Доступ запрещен</div>}
    />
  );
};
```

## Композиция HOC

### Функция для композиции HOC

```typescript
import React, { ComponentType } from 'react';

// Функция для композиции нескольких HOC
function compose<TProps>(
  ...hocs: Array<(component: ComponentType<any>) => ComponentType<any>>
): (component: ComponentType<TProps>) => ComponentType<TProps> {
  return (component: ComponentType<TProps>) => 
    hocs.reverse().reduce((acc, hoc) => hoc(acc), component as any);
}

// Пример HOC для логирования
function withLogging<TProps extends {}>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<TProps> {
  return function WithLogging(props: TProps) {
    React.useEffect(() => {
      console.log('Компонент отрендерен с пропсами:', props);
    });
    
    return <WrappedComponent {...props} />;
  };
}

// HOC для обработки ошибок
function withErrorBoundary<TProps extends {}>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<TProps> {
  return class WithErrorBoundary extends React.Component<TProps> {
    state = { hasError: false };
    
    static getDerivedStateFromError() {
      return { hasError: true };
    }
    
    render() {
      if (this.state.hasError) {
        return <div>Произошла ошибка</div>;
      }
      
      return <WrappedComponent {...this.props} />;
    }
  };
}

// Композиция HOC
interface UserProfileProps {
  name: string;
  email: string;
}

const UserProfile: React.FC<UserProfileProps> = ({ name, email }) => (
  <div>
    <h3>{name}</h3>
    <p>{email}</p>
  </div>
);

// Композиция нескольких HOC
const EnhancedUserProfile = compose<UserProfileProps>(
  withLogging,
  withErrorBoundary,
  withAuth
)(UserProfile);

// Теперь EnhancedUserProfile имеет все функции HOC
```

## HOC с использованием React.forwardRef

```typescript
import React, { ComponentType, forwardRef, Ref } from 'react';

function withRef<TProps extends { ref?: React.Ref<any> }, TRef>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<TProps & { forwardedRef?: Ref<TRef> }> {
  return forwardRef<TRef, TProps & { forwardedRef?: Ref<TRef> }>(
    (props: TProps & { forwardedRef?: Ref<TRef> }, ref) => {
      const { forwardedRef, ...restProps } = props;
      
      return (
        <WrappedComponent
          {...restProps as TProps}
          ref={ref || forwardedRef}
        />
      );
    }
  );
}

// Пример использования
interface InputProps {
  value: string;
  onChange: (value: string) => void;
  ref?: React.Ref<HTMLInputElement>;
}

const Input: React.FC<InputProps> = React.forwardRef<HTMLInputElement, InputProps>(
  ({ value, onChange }, ref) => (
    <input
      ref={ref}
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  )
);

const InputWithRef = withRef<InputProps, HTMLInputElement>(Input);

const App: React.FC = () => {
  const inputRef = React.useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    if (inputRef.current) {
      inputRef.current.focus();
    }
  };
  
  return (
    <div>
      <InputWithRef 
        value="Тест" 
        onChange={() => {}} 
        forwardedRef={inputRef} 
      />
      <button onClick={focusInput}>Фокус</button>
    </div>
  );
};
```

## Практические примеры HOC

### HOC для управления состоянием

```typescript
import React, { ComponentType, useState } from 'react';

interface WithStateProps<T> {
  state: T;
  setState: React.Dispatch<React.SetStateAction<T>>;
}

function withState<T>(
  initialState: T
): <TProps extends WithStateProps<T>>(
  Component: ComponentType<TProps>
) => ComponentType<Omit<TProps, keyof WithStateProps<T>>> {
  return function withStateHOC<TProps extends WithStateProps<T>>(
    WrappedComponent: ComponentType<TProps>
  ): ComponentType<Omit<TProps, keyof WithStateProps<T>>> {
    return function WithState(props: Omit<TProps, keyof WithStateProps<T>>) {
      const [state, setState] = useState<T>(initialState);
      
      return (
        <WrappedComponent
          {...props as TProps}
          state={state}
          setState={setState}
        />
      );
    };
  };
}

// Пример использования
interface CounterProps {
  state: number;
  setState: React.Dispatch<React.SetStateAction<number>>;
}

const Counter: React.FC<CounterProps> = ({ state, setState }) => (
  <div>
    <p>Счетчик: {state}</p>
    <button onClick={() => setState(prev => prev + 1)}>+</button>
    <button onClick={() => setState(prev => prev - 1)}>-</button>
  </div>
);

const CounterWithState = withState(0)(Counter);

// Теперь CounterWithState управляет своим состоянием
```

### HOC для подписки на события

```typescript
import React, { ComponentType, useEffect, useState } from 'react';

interface WithWindowSizeProps {
  windowSize: { width: number; height: number };
}

function withWindowSize<TProps extends WithWindowSizeProps>(
  WrappedComponent: ComponentType<TProps>
): ComponentType<Omit<TProps, keyof WithWindowSizeProps>> {
  return function WithWindowSize(props: Omit<TProps, keyof WithWindowSizeProps>) {
    const [windowSize, setWindowSize] = useState({
      width: window.innerWidth,
      height: window.innerHeight
    });

    useEffect(() => {
      const handleResize = () => {
        setWindowSize({
          width: window.innerWidth,
          height: window.innerHeight
        });
      };

      window.addEventListener('resize', handleResize);
      
      return () => {
        window.removeEventListener('resize', handleResize);
      };
    }, []);

    return (
      <WrappedComponent
        {...props as TProps}
        windowSize={windowSize}
      />
    );
  };
}

// Использование
interface ResponsiveComponentProps {
  windowSize: { width: number; height: number };
}

const ResponsiveComponent: React.FC<ResponsiveComponentProps> = ({ windowSize }) => (
  <div>
    <p>Ширина окна: {windowSize.width}px</p>
    <p>Высота окна: {windowSize.height}px</p>
  </div>
);

const ResponsiveComponentWithSize = withWindowSize(ResponsiveComponent);
```

## Современные альтернативы HOC

Хотя HOC - мощный паттерн, в современном React часто используются альтернативы:

### Использование хуков вместо HOC

```typescript
// Вместо HOC для аутентификации
function useAuth() {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [userRole, setUserRole] = useState('');
  
  // Логика проверки аутентификации
  useEffect(() => {
    // проверка сессии
  }, []);
  
  return { isAuthenticated, userRole };
}

// Компонент использует хук напрямую
const ProtectedComponent: React.FC<{ title: string }> = ({ title }) => {
  const { isAuthenticated, userRole } = useAuth();
  
  if (!isAuthenticated) {
    return <div>Доступ запрещен</div>;
  }
  
  return (
    <div>
      <h1>{title}</h1>
      <p>Роль пользователя: {userRole}</p>
    </div>
  );
};
```

### Использование контекста вместо HOC

```typescript
// Контекст для аутентификации
interface AuthContextType {
  isAuthenticated: boolean;
  userRole: string;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Провайдер аутентификации
const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  // Реализация провайдера
};

// Компонент использует контекст
const ProtectedComponent: React.FC<{ title: string }> = ({ title }) => {
  const auth = useAuthContext(); // кастомный хук для контекста
  
  if (!auth?.isAuthenticated) {
    return <div>Доступ запрещен</div>;
  }
  
  return (
    <div>
      <h1>{title}</h1>
      <button onClick={auth.logout}>Выйти</button>
    </div>
  );
};
```

## Лучшие практики

1. **Сохраняйте displayName для отладки**:
   ```typescript
   WithHOC.displayName = `withHOC(${getDisplayName(WrappedComponent)})`;
   ```

2. **Передавайте статические методы**:
   ```typescript
   function getDisplayName(WrappedComponent: ComponentType) {
     return WrappedComponent.displayName || WrappedComponent.name || 'Component';
   }
   ```

3. **Используйте дженерики для сохранения типов**:
   Это помогает сохранить информацию о типах оригинального компонента.

4. **Рассмотрите альтернативы HOC**:
   В современном React часто предпочтительнее использовать хуки или контекст.

5. **Избегайте чрезмерной вложенности HOC**:
   Это может затруднить отладку и понимание кода.

## Связанные темы

- [[Типизация-React-компонентов]]
- [[Типизация-React-хуков]]
- [[Типизация-React-контекста]]
- [[Рендер-пропсы-с-типами]]
- [[React-паттерны-типов]]

## Заключение

HOC - мощный паттерн для повторного использования логики в React-компонентах. При использовании TypeScript важно правильно типизировать HOC, чтобы сохранить информацию о типах и обеспечить безопасность типов. Однако в современном React часто предпочтительнее использовать хуки или контекст для решения схожих задач.