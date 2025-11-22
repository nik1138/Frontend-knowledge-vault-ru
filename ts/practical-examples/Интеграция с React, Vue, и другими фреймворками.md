---
tags: [typescript, frontend, react, vue, integration]
aliases: [Интеграция TypeScript с фреймворками]
---

# Интеграция с React, Vue, и другими фреймворками

## Введение

TypeScript отлично интегрируется с современными frontend фреймворками, обеспечивая типобезопасность и улучшенный опыт разработки. В этом разделе мы рассмотрим, как использовать TypeScript с различными фреймворками и библиотеками.

## Интеграция с React

### Настройка проекта

```bash
# Создание нового React проекта с TypeScript
npx create-react-app my-app --template typescript

# Или добавление TypeScript в существующий проект
npm install --save-dev typescript @types/node @types/react @types/react-dom
```

### Типизация компонентов

```tsx
import React, { useState, useEffect } from 'react';

// Интерфейс для пропсов компонента
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
  className?: string;
}

// Функциональный компонент с полной типизацией
const Button: React.FC<ButtonProps> = ({ 
  children, 
  onClick, 
  variant = 'primary', 
  disabled = false,
  className = ''
}) => {
  const baseClasses = 'btn';
  const variantClass = `btn-${variant}`;
  const disabledClass = disabled ? 'btn-disabled' : '';
  
  return (
    <button
      className={`${baseClasses} ${variantClass} ${disabledClass} ${className}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

// Использование компонента
const App: React.FC = () => {
  const [count, setCount] = useState<number>(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <Button 
        variant="primary" 
        onClick={() => setCount(count + 1)}
      >
        Increment
      </Button>
      <Button 
        variant="secondary" 
        onClick={() => setCount(0)}
        disabled={count === 0}
      >
        Reset
      </Button>
    </div>
  );
};
```

### Типизация хуков

```tsx
import { useState, useEffect, useRef, useCallback } from 'react';

// Кастомный хук с типизацией
function useCounter(initialValue: number = 0) {
  const [count, setCount] = useState<number>(initialValue);
  
  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  
  return { count, increment, decrement, reset };
}

// Хук для работы с localStorage
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
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Хук для обработки событий клавиатуры
function useKeyPress(targetKey: string) {
  const [keyPressed, setKeyPressed] = useState<boolean>(false);

  useEffect(() => {
    function downHandler({ key }: KeyboardEvent) {
      if (key === targetKey) {
        setKeyPressed(true);
      }
    }

    const upHandler = ({ key }: KeyboardEvent) => {
      if (key === targetKey) {
        setKeyPressed(false);
      }
    };

    window.addEventListener('keydown', downHandler);
    window.addEventListener('keyup', upHandler);

    return () => {
      window.removeEventListener('keydown', downHandler);
      window.removeEventListener('keyup', upHandler);
    };
  }, [targetKey]);

  return keyPressed;
}
```

### Типизация контекста

```tsx
import React, { createContext, useContext, useReducer, ReactNode } from 'react';

// Типы для состояния аутентификации
interface AuthState {
  user: { id: number; name: string; email: string } | null;
  isAuthenticated: boolean;
  loading: boolean;
}

interface AuthAction {
  type: 
    | 'LOGIN_START'
    | 'LOGIN_SUCCESS' 
    | 'LOGIN_FAILURE'
    | 'LOGOUT'
    | 'SET_LOADING';
  payload?: any;
}

// Начальное состояние
const initialAuthState: AuthState = {
  user: null,
  isAuthenticated: false,
  loading: false,
};

// Редьюсер для аутентификации
const authReducer = (state: AuthState, action: AuthAction): AuthState => {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, loading: true };
    case 'LOGIN_SUCCESS':
      return { 
        ...state, 
        user: action.payload, 
        isAuthenticated: true, 
        loading: false 
      };
    case 'LOGIN_FAILURE':
      return { 
        ...state, 
        user: null, 
        isAuthenticated: false, 
        loading: false 
      };
    case 'LOGOUT':
      return { ...state, user: null, isAuthenticated: false };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    default:
      return state;
  }
};

// Создание контекста с типизацией
interface AuthContextType extends AuthState {
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Провайдер контекста
interface AuthProviderProps {
  children: ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, initialAuthState);

  const login = async (email: string, password: string) => {
    dispatch({ type: 'LOGIN_START' });
    
    try {
      // Симуляция API вызова
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });
      
      if (response.ok) {
        const user = await response.json();
        dispatch({ type: 'LOGIN_SUCCESS', payload: user });
      } else {
        dispatch({ type: 'LOGIN_FAILURE' });
      }
    } catch (error) {
      dispatch({ type: 'LOGIN_FAILURE' });
    }
  };

  const logout = () => {
    dispatch({ type: 'LOGOUT' });
  };

  return (
    <AuthContext.Provider value={{ ...state, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

// Хук для использования контекста
export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

// Использование контекста
const Profile: React.FC = () => {
  const { user, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>Please log in</div>;
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <p>Email: {user.email}</p>
    </div>
  );
};
```

## Интеграция с Vue.js

### Настройка проекта

```bash
# Создание нового Vue проекта с TypeScript
npm create vue@latest my-vue-app
# Выберите TypeScript при создании

# Или добавление TypeScript в существующий проект
npm install --save-dev typescript @vue/tsconfig
```

### Типизация компонентов в Vue 3

```vue
<!-- UserCard.vue -->
<template>
  <div class="user-card">
    <img v-if="user.avatar" :src="user.avatar" :alt="user.name" class="avatar" />
    <div v-else class="default-avatar">
      {{ user.name.charAt(0).toUpperCase() }}
    </div>
    
    <div class="user-info">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <p v-if="showAge">Age: {{ user.age }}</p>
    </div>
    
    <div class="actions">
      <button @click="$emit('edit', user.id)">Edit</button>
      <button @click="$emit('delete', user.id)">Delete</button>
    </div>
  </div>
</template>

<script setup lang="ts">
// Типизация пропсов
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;
  avatar?: string;
}

interface Props {
  user: User;
  showAge?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  showAge: false
});

// Типизация emit
interface Emits {
  (e: 'edit', userId: number): void;
  (e: 'delete', userId: number): void;
}

const emit = defineEmits<Emits>();
</script>
```

### Composition API с типизацией

```ts
// composables/useUsers.ts
import { ref, Ref, onMounted } from 'vue';

export interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

interface UseUsersReturn {
  users: Ref<User[]>;
  loading: Ref<boolean>;
  error: Ref<string | null>;
  fetchUsers: () => Promise<void>;
  addUser: (user: Omit<User, 'id'>) => Promise<void>;
  updateUser: (id: number, updates: Partial<User>) => Promise<void>;
  deleteUser: (id: number) => Promise<void>;
}

export function useUsers(): UseUsersReturn {
  const users = ref<User[]>([]);
  const loading = ref<boolean>(false);
  const error = ref<string | null>(null);

  const fetchUsers = async () => {
    loading.value = true;
    error.value = null;

    try {
      const response = await fetch('/api/users');
      if (!response.ok) {
        throw new Error('Failed to fetch users');
      }
      users.value = await response.json();
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'An error occurred';
    } finally {
      loading.value = false;
    }
  };

  const addUser = async (userData: Omit<User, 'id'>) => {
    try {
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData),
      });
      
      if (!response.ok) {
        throw new Error('Failed to add user');
      }
      
      const newUser: User = await response.json();
      users.value.push(newUser);
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'An error occurred';
    }
  };

  const updateUser = async (id: number, updates: Partial<User>) => {
    try {
      const response = await fetch(`/api/users/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(updates),
      });
      
      if (!response.ok) {
        throw new Error('Failed to update user');
      }
      
      const updatedUser: User = await response.json();
      const index = users.value.findIndex(user => user.id === id);
      if (index !== -1) {
        users.value[index] = updatedUser;
      }
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'An error occurred';
    }
  };

  const deleteUser = async (id: number) => {
    try {
      const response = await fetch(`/api/users/${id}`, {
        method: 'DELETE',
      });
      
      if (!response.ok) {
        throw new Error('Failed to delete user');
      }
      
      users.value = users.value.filter(user => user.id !== id);
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'An error occurred';
    }
  };

  onMounted(() => {
    fetchUsers();
  });

  return {
    users,
    loading,
    error,
    fetchUsers,
    addUser,
    updateUser,
    deleteUser
  };
}
```

### Типизация в компоненте

```vue
<!-- UserList.vue -->
<template>
  <div>
    <h2>Users</h2>
    
    <div v-if="loading">Loading...</div>
    <div v-else-if="error" class="error">Error: {{ error }}</div>
    <div v-else>
      <UserCard
        v-for="user in users"
        :key="user.id"
        :user="user"
        :show-age="true"
        @edit="handleEdit"
        @delete="handleDelete"
      />
    </div>
    
    <button @click="addNewUser">Add User</button>
  </div>
</template>

<script setup lang="ts">
import { User, useUsers } from '@/composables/useUsers';
import UserCard from './UserCard.vue';

const { users, loading, error, addUser } = useUsers();

const handleEdit = (userId: number) => {
  console.log('Edit user:', userId);
  // Логика редактирования
};

const handleDelete = (userId: number) => {
  console.log('Delete user:', userId);
  // Логика удаления
};

const addNewUser = async () => {
  await addUser({
    name: 'New User',
    email: 'new@example.com',
    age: 25
  });
};
</script>
```

## Интеграция с другими библиотеками

### React Router с TypeScript

```tsx
import { 
  createBrowserRouter, 
  RouterProvider, 
  Outlet,
  useLoaderData,
  useNavigate,
  useParams
} from 'react-router-dom';
import { User } from '../types';

// Типизация для loader функций
interface UserLoaderData {
  user: User;
}

const userLoader = async ({ params }: { params: { id: string } }) => {
  const response = await fetch(`/api/users/${params.id}`);
  if (!response.ok) {
    throw new Error('User not found');
  }
  const user: User = await response.json();
  return { user };
};

// Типизация для action функций
import { ActionFunctionArgs } from 'react-router-dom';

const updateUserAction = async ({ request, params }: ActionFunctionArgs) => {
  const formData = await request.formData();
  const updates = Object.fromEntries(formData);
  
  const response = await fetch(`/api/users/${params.id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(updates),
  });
  
  return response.json();
};

// Компонент с типизированными данными
const UserDetail: React.FC = () => {
  const { user } = useLoaderData() as UserLoaderData;
  const navigate = useNavigate();
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      <button onClick={() => navigate(-1)}>Back</button>
    </div>
  );
};

// Конфигурация маршрутов
const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        path: "users",
        element: <UserList />,
      },
      {
        path: "users/:id",
        element: <UserDetail />,
        loader: userLoader,
        action: updateUserAction,
      },
    ]
  }
]);

const Root: React.FC = () => {
  return (
    <div>
      <nav>
        <Link to="/users">Users</Link>
      </nav>
      <main>
        <Outlet />
      </main>
    </div>
  );
};
```

### Redux Toolkit с TypeScript

```ts
// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import userReducer from './userSlice';
import authReducer from './authSlice';

export const store = configureStore({
  reducer: {
    user: userReducer,
    auth: authReducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }),
});

// Типизация store
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```ts
// hooks.ts
import { useDispatch, useSelector, TypedUseSelectorHook } from 'react-redux';
import type { RootState, AppDispatch } from './store';

// Типизированные хуки
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

```ts
// store/userSlice.ts
import { createSlice, PayloadAction, createAsyncThunk } from '@reduxjs/toolkit';
import { User } from '../types';

// Async thunk с типизацией
interface FetchUsersResponse {
  users: User[];
  total: number;
}

export const fetchUsers = createAsyncThunk<
  FetchUsersResponse,  // Тип возвращаемого значения
  number,              // Тип аргумента
  {
    rejectValue: string;  // Тип значения при отклонении
  }
>(
  'users/fetchUsers',
  async (page, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users?page=${page}`);
      if (!response.ok) {
        throw new Error('Failed to fetch users');
      }
      const data: FetchUsersResponse = await response.json();
      return data;
    } catch (error) {
      return rejectWithValue(
        error instanceof Error ? error.message : 'Unknown error'
      );
    }
  }
);

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
  currentPage: number;
  totalPages: number;
}

const initialState: UserState = {
  users: [],
  loading: false,
  error: null,
  currentPage: 1,
  totalPages: 0,
};

const userSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    setCurrentPage: (state, action: PayloadAction<number>) => {
      state.currentPage = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, { payload }) => {
        state.loading = false;
        state.users = payload.users;
        state.totalPages = Math.ceil(payload.total / 10); // Предполагаем 10 на страницу
      })
      .addCase(fetchUsers.rejected, (state, { payload }) => {
        state.loading = false;
        state.error = payload || 'Failed to fetch users';
      });
  },
});

export const { setCurrentPage } = userSlice.actions;
export default userSlice.reducer;
```

## Практические примеры

### Типизация для асинхронных компонентов

```tsx
import { lazy, Suspense } from 'react';

// Ленивая загрузка компонентов с типизацией
const LazyDashboard = lazy(() => import('./Dashboard'));
const LazyProfile = lazy(() => import('./Profile'));
const LazySettings = lazy(() => import('./Settings'));

interface RouteConfig {
  path: string;
  component: React.LazyExoticComponent<React.FC>;
  name: string;
}

const routes: RouteConfig[] = [
  { path: '/dashboard', component: LazyDashboard, name: 'Dashboard' },
  { path: '/profile', component: LazyProfile, name: 'Profile' },
  { path: '/settings', component: LazySettings, name: 'Settings' },
];

const AppRouter: React.FC = () => {
  const [currentPath, setCurrentPath] = useState('/dashboard');
  
  const CurrentComponent = routes.find(r => r.path === currentPath)?.component;
  
  return (
    <div>
      <nav>
        {routes.map(route => (
          <button
            key={route.path}
            onClick={() => setCurrentPath(route.path)}
            className={currentPath === route.path ? 'active' : ''}
          >
            {route.name}
          </button>
        ))}
      </nav>
      
      <Suspense fallback={<div>Loading...</div>}>
        {CurrentComponent && <CurrentComponent />}
      </Suspense>
    </div>
  );
};
```

### Типизация для HOC (Higher-Order Components)

```tsx
import React from 'react';

// HOC для аутентификации с типизацией
export function withAuth<P extends object>(
  Component: React.ComponentType<P>
): React.FC<Omit<P, 'user'> & { user?: { id: number; name: string } }> {
  return function AuthenticatedComponent(props) {
    const { user } = useAuth(); // предполагаем существование useAuth хука
    
    if (!user) {
      return <div>Please log in to access this page</div>;
    }
    
    return <Component {...props as P} user={user} />;
  };
}

// Использование HOC
interface UserProfileProps {
  user: { id: number; name: string; email: string };
  showEmail?: boolean;
}

const UserProfile: React.FC<UserProfileProps> = ({ user, showEmail = false }) => (
  <div>
    <h1>{user.name}</h1>
    {showEmail && <p>{user.email}</p>}
  </div>
);

const ProtectedUserProfile = withAuth(UserProfile);

// Типизация для Render Props
interface DataProviderProps<T> {
  render: (data: T, loading: boolean) => React.ReactNode;
  url: string;
}

function DataProvider<T>({ render, url }: DataProviderProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  
  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch(url);
        const result: T = await response.json();
        setData(result);
      } catch (error) {
        console.error('Error fetching data:', error);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return render(data!, loading);
}

// Использование
const UserList = () => (
  <DataProvider<{ users: { id: number; name: string }[] }>
    url="/api/users"
    render={(data, loading) => 
      loading ? 
        <div>Loading...</div> : 
        <ul>
          {data.users.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
    }
  />
);
```

## Заключение

Интеграция TypeScript с различными фреймворками значительно улучшает качество кода и опыт разработки. Правильная типизация компонентов, хуков, действий и состояния помогает избежать многих ошибок и делает код более надежным и понятным.

> [!tip] Совет
> Используйте строгую типизацию для всех публичных API и интерфейсов компонентов - это упрощает их использование и поддержку.

> [!warning] Важно
> При интеграции с новыми библиотеками всегда проверяйте наличие типов в DefinitelyTyped или возможности настройки собственных типов.

## Связанные темы

- [[Практические примеры для frontend разработки]]
- [[Утилиты типов TypeScript]]
- [[Дженерики и условные типы]]