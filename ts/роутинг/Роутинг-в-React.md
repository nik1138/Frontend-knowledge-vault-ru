---
aliases: [React Router, Роутинг в React, React Navigation]
tags: [react, routing, typescript, frontend]
---

# Роутинг в React

## Обзор

Роутинг в React позволяет создавать одностраничные приложения (SPA) с навигацией между различными представлениями без перезагрузки страницы. Для этого обычно используется библиотека React Router, которая предоставляет декларативный способ определения маршрутов.

## Основные понятия React Router

### BrowserRouter vs HashRouter

- `BrowserRouter` — использует API History для навигации (рекомендуется)
- `HashRouter` — использует хэш URL для навигации (для статических хостингов)

```typescript
import { BrowserRouter, HashRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/*" element={<Users />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### Основные компоненты

- `Routes` — контейнер для маршрутов
- `Route` — определяет маршрут и компонент для отображения
- `Link` — создает навигационную ссылку
- `NavLink` — `Link` с автоматическим выделением активного состояния
- `Navigate` — компонент для перенаправления

## Установка и настройка

```bash
npm install react-router-dom
```

### Базовая настройка

```typescript
// App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './components/Home';
import About from './components/About';
import Users from './components/Users';
import UserDetail from './components/UserDetail';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users" element={<Users />}>
          <Route path=":userId" element={<UserDetail />} />
        </Route>
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

## Типизированные маршруты

### Определение интерфейсов для параметров

```typescript
// types/routeTypes.ts
interface UserRouteParams {
  userId: string;
}

interface PostRouteParams {
  postId: string;
  category: string;
}
```

### Использование useParams с типами

```typescript
// components/UserDetail.tsx
import { useParams } from 'react-router-dom';
import { UserRouteParams } from '../types/routeTypes';

function UserDetail() {
  const { userId } = useParams<UserRouteParams>();
  
  // Теперь userId строго типизирован
  console.log('User ID:', userId);
  
  return <div>Детали пользователя: {userId}</div>;
}
```

## Продвинутые паттерны

### Вложенные маршруты

```typescript
// components/Users.tsx
import { Outlet, useOutletContext } from 'react-router-dom';

interface UserContext {
  users: User[];
  refreshUsers: () => void;
}

function Users() {
  const [users, setUsers] = useState<User[]>([]);
  
  const contextValue: UserContext = {
    users,
    refreshUsers: () => {
      // Логика обновления пользователей
    }
  };

  return (
    <div>
      <h1>Пользователи</h1>
      <nav>
        <Link to="new">Новый пользователь</Link>
      </nav>
      <Outlet context={contextValue} />
    </div>
  );
}

export function useUsersContext() {
  return useOutletContext<UserContext>();
}
```

### Защита маршрутов

```typescript
// components/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

interface ProtectedRouteProps {
  children: React.ReactNode;
  allowedRoles?: string[];
}

function ProtectedRoute({ children, allowedRoles }: ProtectedRouteProps) {
  const { isAuthenticated, user } = useAuth();
  const location = useLocation();

  if (!isAuthenticated) {
    // Перенаправление на страницу входа с сохранением исходного пути
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (allowedRoles && user && !allowedRoles.includes(user.role)) {
    // Пользователь не имеет необходимых прав
    return <Navigate to="/unauthorized" replace />;
  }

  return <>{children}</>;
}
```

### Динамическая загрузка

```typescript
// components/LazyRoutes.tsx
import { Suspense, lazy } from 'react';
import { Routes, Route } from 'react-router-dom';

const LazyHome = lazy(() => import('./Home'));
const LazyAbout = lazy(() => import('./About'));
const LazyDashboard = lazy(() => import('./Dashboard'));

function LazyRoutes() {
  return (
    <Suspense fallback={<div>Загрузка...</div>}>
      <Routes>
        <Route path="/" element={<LazyHome />} />
        <Route path="/about" element={<LazyAbout />} />
        <Route path="/dashboard" element={<LazyDashboard />} />
      </Routes>
    </Suspense>
  );
}
```

## Навигация программно

### useNavigate

```typescript
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();
    // Логика аутентификации
    
    if (loginSuccessful) {
      // Перенаправление после успешного входа
      navigate('/dashboard', { replace: true });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* форма входа */}
    </form>
  );
}
```

### useSearchParams

```typescript
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q') || '';
  const page = parseInt(searchParams.get('page') || '1');
  
  const handleSearch = (newQuery: string) => {
    setSearchParams({ q: newQuery, page: '1' });
  };
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Поиск..."
      />
      <p>Страница: {page}</p>
    </div>
  );
}
```

## Обработка ошибок

### ErrorBoundary для маршрутов

```typescript
// components/RouteErrorBoundary.tsx
import { Component, ReactNode } from 'react';
import { useRouteError } from 'react-router-dom';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
}

class RouteErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(): State {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    console.error('Routing error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Что-то пошло не так.</h1>;
    }

    return this.props.children;
  }
}

// Использование в App.tsx
function App() {
  return (
    <BrowserRouter>
      <RouteErrorBoundary>
        <Routes>
          <Route path="/" element={<Home />} />
          {/* другие маршруты */}
        </Routes>
      </RouteErrorBoundary>
    </BrowserRouter>
  );
}
```

## Лучшие практики

### Структура файлов

```
src/
├── components/
│   ├── routes/
│   │   ├── Home.tsx
│   │   ├── About.tsx
│   │   └── Users/
│   │       ├── Users.tsx
│   │       ├── UserList.tsx
│   │       └── UserDetail.tsx
│   └── layouts/
│       ├── MainLayout.tsx
│       └── AuthLayout.tsx
├── types/
│   └── routeTypes.ts
├── hooks/
│   └── useAuth.ts
└── App.tsx
```

### Централизованное определение маршрутов

```typescript
// routes/config.ts
import { RouteObject } from 'react-router-dom';
import { ProtectedRoute } from '../components/ProtectedRoute';
import { Home } from '../components/Home';
import { About } from '../components/About';
import { Dashboard } from '../components/Dashboard';

export const routes: RouteObject[] = [
  {
    path: '/',
    element: <Home />,
    index: true
  },
  {
    path: '/about',
    element: <About />
  },
  {
    path: '/dashboard',
    element: (
      <ProtectedRoute allowedRoles={['admin', 'user']}>
        <Dashboard />
      </ProtectedRoute>
    )
  }
];
```

### Типизированные хуки

```typescript
// hooks/useTypedParams.ts
import { useParams } from 'react-router-dom';

export function useTypedParams<T extends Record<string, string>>(): T {
  return useParams<T>();
}

// hooks/useTypedSearchParams.ts
import { useSearchParams } from 'react-router-dom';

export function useTypedSearchParams<T extends Record<string, string>>() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const getTypedParam = (key: keyof T): string | null => {
    return searchParams.get(key as string);
  };
  
  return {
    searchParams,
    setSearchParams,
    getTypedParam
  };
}
```

## Миграция с классических версий

### React Router v5 vs v6

```typescript
// React Router v5 (устаревший)
import { Switch, Route, useHistory, useParams } from 'react-router-dom';

function App() {
  return (
    <Switch>
      <Route exact path="/" component={Home} />
      <Route path="/users/:id" component={UserDetail} />
    </Switch>
  );
}

// React Router v6 (современный)
import { Routes, Route, useNavigate, useParams } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/users/:id" element={<UserDetail />} />
    </Routes>
  );
}
```

## Заключение

Роутинг в React с использованием React Router предоставляет мощный и гибкий способ управления навигацией в приложении. Правильное использование типизации, защита маршрутов и эффективная организация структуры приложения позволяют создавать масштабируемые и поддерживаемые приложения.

См. также: [[Клиентский-роутинг]], [[Серверный-роутинг]], [[Роутинг-в-Vue]], [[Роутинг-в-Angular]]