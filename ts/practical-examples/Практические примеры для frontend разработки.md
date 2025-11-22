---
tags: [typescript, frontend, practical-examples, react, vue]
aliases: [Практические примеры TypeScript, Примеры для frontend]
---

# Практические примеры для frontend разработки

## Введение

Практические примеры демонстрируют применение TypeScript в реальных сценариях frontend разработки. В этом разделе мы рассмотрим типичные задачи и решения, с которыми сталкиваются разработчики при создании веб-приложений.

## Типизация React компонентов

### Функциональные компоненты

```tsx
import React, { useState, useEffect } from 'react';

// Типизация пропсов компонента
interface UserCardProps {
    user: {
        id: number;
        name: string;
        email: string;
        avatar?: string;
    };
    onEdit?: (userId: number) => void;
    onDelete?: (userId: number) => void;
}

// Функциональный компонент с типизацией
const UserCard: React.FC<UserCardProps> = ({ user, onEdit, onDelete }) => {
    return (
        <div className="user-card">
            {user.avatar ? (
                <img src={user.avatar} alt={user.name} className="avatar" />
            ) : (
                <div className="default-avatar">
                    {user.name.charAt(0).toUpperCase()}
                </div>
            )}
            <div className="user-info">
                <h3>{user.name}</h3>
                <p>{user.email}</p>
            </div>
            <div className="actions">
                {onEdit && (
                    <button onClick={() => onEdit(user.id)}>Edit</button>
                )}
                {onDelete && (
                    <button onClick={() => onDelete(user.id)}>Delete</button>
                )}
            </div>
        </div>
    );
};

export default UserCard;
```

### Типизация хуков

```tsx
import { useState, useEffect, useCallback } from 'react';

// Типизация пользовательских хуков
interface UseApiResult<T> {
    data: T | null;
    loading: boolean;
    error: string | null;
    refetch: () => void;
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
            setError(err instanceof Error ? err.message : 'An error occurred');
        } finally {
            setLoading(false);
        }
    }, [url]);

    useEffect(() => {
        fetchData();
    }, [fetchData]);

    return { data, loading, error, refetch: fetchData };
}

// Использование кастомного хука
interface User {
    id: number;
    name: string;
    email: string;
}

const UserProfile: React.FC<{ userId: number }> = ({ userId }) => {
    const { data: user, loading, error, refetch } = useApi<User>(`/api/users/${userId}`);

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error}</div>;
    if (!user) return <div>User not found</div>;

    return (
        <div>
            <h1>{user.name}</h1>
            <p>{user.email}</p>
            <button onClick={refetch}>Refresh</button>
        </div>
    );
};
```

### Типизация форм

```tsx
import React, { useState } from 'react';

interface FormData {
    name: string;
    email: string;
    age: number;
    subscribe: boolean;
}

interface FormErrors {
    name?: string;
    email?: string;
    age?: string;
}

const UserForm: React.FC = () => {
    const [formData, setFormData] = useState<FormData>({
        name: '',
        email: '',
        age: 18,
        subscribe: false
    });

    const [errors, setErrors] = useState<FormErrors>({});

    const validate = (): boolean => {
        const newErrors: FormErrors = {};

        if (!formData.name.trim()) {
            newErrors.name = 'Name is required';
        }

        if (!formData.email.includes('@')) {
            newErrors.email = 'Email is invalid';
        }

        if (formData.age < 18) {
            newErrors.age = 'Must be at least 18 years old';
        }

        setErrors(newErrors);
        return Object.keys(newErrors).length === 0;
    };

    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
        const { name, value, type, checked } = e.target;
        
        setFormData(prev => ({
            ...prev,
            [name]: type === 'checkbox' ? checked : 
                    type === 'number' ? Number(value) : value
        }));
    };

    const handleSubmit = (e: React.FormEvent) => {
        e.preventDefault();
        
        if (validate()) {
            console.log('Form data:', formData);
            // Отправка данных
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <div>
                <label htmlFor="name">Name:</label>
                <input
                    type="text"
                    id="name"
                    name="name"
                    value={formData.name}
                    onChange={handleChange}
                />
                {errors.name && <span className="error">{errors.name}</span>}
            </div>

            <div>
                <label htmlFor="email">Email:</label>
                <input
                    type="email"
                    id="email"
                    name="email"
                    value={formData.email}
                    onChange={handleChange}
                />
                {errors.email && <span className="error">{errors.email}</span>}
            </div>

            <div>
                <label htmlFor="age">Age:</label>
                <input
                    type="number"
                    id="age"
                    name="age"
                    value={formData.age}
                    onChange={handleChange}
                />
                {errors.age && <span className="error">{errors.age}</span>}
            </div>

            <div>
                <label>
                    <input
                        type="checkbox"
                        name="subscribe"
                        checked={formData.subscribe}
                        onChange={handleChange}
                    />
                    Subscribe to newsletter
                </label>
            </div>

            <button type="submit">Submit</button>
        </form>
    );
};
```

## Типизация для других фреймворков

### Vue.js с TypeScript

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
    </div>
    
    <div class="actions">
      <button v-if="onEdit" @click="handleEdit">Edit</button>
      <button v-if="onDelete" @click="handleDelete">Delete</button>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue';

interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
}

export default defineComponent({
  name: 'UserCard',
  props: {
    user: {
      type: Object as PropType<User>,
      required: true
    },
    onEdit: {
      type: Function as PropType<(userId: number) => void>,
      required: false
    },
    onDelete: {
      type: Function as PropType<(userId: number) => void>,
      required: false
    }
  },
  methods: {
    handleEdit() {
      if (this.onEdit) {
        this.onEdit(this.user.id);
      }
    },
    handleDelete() {
      if (this.onDelete) {
        this.onDelete(this.user.id);
      }
    }
  }
});
</script>
```

### Типизация Composition API в Vue 3

```ts
// composables/useUsers.ts
import { ref, Ref } from 'vue';

export interface User {
  id: number;
  name: string;
  email: string;
}

interface UseUsersResult {
  users: Ref<User[]>;
  loading: Ref<boolean>;
  error: Ref<string | null>;
  fetchUsers: () => Promise<void>;
}

export function useUsers(): UseUsersResult {
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

  return {
    users,
    loading,
    error,
    fetchUsers
  };
}
```

## Работа с API и асинхронными операциями

### Типизация API клиентов

```ts
// api/types.ts
export interface ApiResponse<T> {
  data: T;
  status: number;
  message?: string;
}

export interface User {
  id: number;
  name: string;
  email: string;
  createdAt: string;
}

// api/client.ts
class ApiClient {
  private baseUrl: string;
  private defaultHeaders: HeadersInit;

  constructor(baseUrl: string, defaultHeaders: HeadersInit = {}) {
    this.baseUrl = baseUrl;
    this.defaultHeaders = defaultHeaders;
  }

  async request<T>(endpoint: string, options: RequestInit = {}): Promise<ApiResponse<T>> {
    const url = `${this.baseUrl}${endpoint}`;
    
    const config: RequestInit = {
      headers: {
        'Content-Type': 'application/json',
        ...this.defaultHeaders,
        ...options.headers,
      },
      ...options,
    };

    const response = await fetch(url, config);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    
    return {
      data,
      status: response.status,
    };
  }

  async get<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, { method: 'GET' });
  }

  async post<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  async put<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  async delete<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

// Создание экземпляра клиента
export const apiClient = new ApiClient(process.env.API_BASE_URL || 'https://api.example.com');

// API сервисы
export const userService = {
  getAll: () => apiClient.get<User[]>('/users'),
  getById: (id: number) => apiClient.get<User>(`/users/${id}`),
  create: (userData: Omit<User, 'id' | 'createdAt'>) => 
    apiClient.post<User>('/users', userData),
  update: (id: number, userData: Partial<User>) => 
    apiClient.put<User>(`/users/${id}`, userData),
  delete: (id: number) => apiClient.delete<User>(`/users/${id}`),
};
```

### Типизация с использованием SWR (React)

```tsx
import useSWR from 'swr';
import { User } from '../api/types';

// SWR fetcher с типизацией
const fetcher = async (url: string): Promise<User[]> => {
  const res = await fetch(url);
  if (!res.ok) {
    throw new Error('Failed to fetch users');
  }
  return res.json();
};

// Компонент с использованием SWR
const UserList: React.FC = () => {
  const { data, error, isLoading } = useSWR<User[], Error>('/api/users', fetcher);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return <div>No users found</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
};
```

## Типизация для популярных библиотек

### Типизация для Redux Toolkit

```ts
// store/userSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';
import { User } from '../api/types';

interface UserState {
  users: User[];
  loading: boolean;
  error: string | null;
  currentUser: User | null;
}

const initialState: UserState = {
  users: [],
  loading: false,
  error: null,
  currentUser: null,
};

export const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUsers: (state, action: PayloadAction<User[]>) => {
      state.users = action.payload;
    },
    addUser: (state, action: PayloadAction<User>) => {
      state.users.push(action.payload);
    },
    updateUser: (state, action: PayloadAction<User>) => {
      const index = state.users.findIndex(user => user.id === action.payload.id);
      if (index !== -1) {
        state.users[index] = action.payload;
      }
    },
    deleteUser: (state, action: PayloadAction<number>) => {
      state.users = state.users.filter(user => user.id !== action.payload);
    },
    setCurrentUser: (state, action: PayloadAction<User | null>) => {
      state.currentUser = action.payload;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setError: (state, action: PayloadAction<string | null>) => {
      state.error = action.payload;
    },
  },
});

export const {
  setUsers,
  addUser,
  updateUser,
  deleteUser,
  setCurrentUser,
  setLoading,
  setError,
} = userSlice.actions;

export default userSlice.reducer;
```

### Типизация для React Router

```tsx
import { 
  createBrowserRouter, 
  RouterProvider, 
  useParams, 
  useNavigate,
  NavigateFunction
} from 'react-router-dom';

interface User {
  id: number;
  name: string;
  email: string;
}

// Типизация параметров маршрута
interface UserDetailParams {
  userId: string;
}

// Компонент с типизированными параметрами
const UserDetail: React.FC = () => {
  const { userId } = useParams<UserDetailParams>();
  const navigate: NavigateFunction = useNavigate();
  
  // Типизация userId как number
  const numericUserId = Number(userId);
  
  if (isNaN(numericUserId)) {
    return <Navigate to="/users" replace />;
  }
  
  // Логика получения и отображения пользователя
  const user: User | undefined = undefined; // Здесь будет логика получения пользователя
  
  return (
    <div>
      <h1>User Detail: {numericUserId}</h1>
      {user ? (
        <div>
          <h2>{user.name}</h2>
          <p>{user.email}</p>
        </div>
      ) : (
        <p>User not found</p>
      )}
      <button onClick={() => navigate('/users')}>Back to Users</button>
    </div>
  );
};

// Конфигурация маршрутов
const router = createBrowserRouter([
  {
    path: "/",
    element: <div>Home</div>,
  },
  {
    path: "/users",
    element: <div>Users List</div>,
  },
  {
    path: "/users/:userId",
    element: <UserDetail />,
  },
]);

const App: React.FC = () => {
  return <RouterProvider router={router} />;
};
```

## Решение распространенных задач

### Типизация для валидации форм

```ts
// utils/validation.ts
export interface ValidationRule<T> {
  condition: (value: T) => boolean;
  message: string;
}

export interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

export function validateField<T>(
  value: T,
  rules: ValidationRule<T>[]
): ValidationResult {
  const errors: string[] = [];
  
  for (const rule of rules) {
    if (!rule.condition(value)) {
      errors.push(rule.message);
    }
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}

// Пример использования
interface UserFormData {
  name: string;
  email: string;
  age: number;
}

const nameRules: ValidationRule<string>[] = [
  { 
    condition: (value) => value.length >= 2, 
    message: 'Name must be at least 2 characters' 
  },
  { 
    condition: (value) => /^[a-zA-Z\s]+$/.test(value), 
    message: 'Name can only contain letters and spaces' 
  }
];

const emailRules: ValidationRule<string>[] = [
  { 
    condition: (value) => /\S+@\S+\.\S+/.test(value), 
    message: 'Email must be valid' 
  }
];

const ageRules: ValidationRule<number>[] = [
  { 
    condition: (value) => value >= 18, 
    message: 'Age must be at least 18' 
  },
  { 
    condition: (value) => value <= 120, 
    message: 'Age must be less than 120' 
  }
];

// Валидация всей формы
function validateUserForm(data: UserFormData): Record<keyof UserFormData, ValidationResult> {
  return {
    name: validateField(data.name, nameRules),
    email: validateField(data.email, emailRules),
    age: validateField(data.age, ageRules)
  };
}
```

### Типизация для работы с localStorage

```ts
// utils/storage.ts
export class StorageService {
  static getItem<T>(key: string, defaultValue: T): T {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error(`Error parsing localStorage item with key "${key}":`, error);
      return defaultValue;
    }
  }

  static setItem<T>(key: string, value: T): void {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(`Error storing item with key "${key}":`, error);
    }
  }

  static removeItem(key: string): void {
    try {
      window.localStorage.removeItem(key);
    } catch (error) {
      console.error(`Error removing item with key "${key}":`, error);
    }
  }

  static clear(): void {
    try {
      window.localStorage.clear();
    } catch (error) {
      console.error('Error clearing localStorage:', error);
    }
  }
}

// Типизированный хук для localStorage
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    return StorageService.getItem<T>(key, initialValue);
  });

  const setValue = (value: T) => {
    try {
      setStoredValue(value);
      StorageService.setItem<T>(key, value);
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Использование хука
interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
}

const [preferences, setPreferences] = useLocalStorage<UserPreferences>(
  'user-preferences',
  { theme: 'light', language: 'en', notifications: true }
);
```

## Заключение

Практические примеры показывают, как TypeScript применяется в реальных сценариях frontend разработки. Правильная типизация компонентов, API клиентов, форм и других частей приложения значительно повышает надежность и читаемость кода.

> [!tip] Совет
> Используйте интерфейсы для определения форм и API ответов - это упрощает поддержку и рефакторинг кода.

> [!warning] Важно
> Не забывайте обновлять типы при изменении API или структуры данных в приложении.

## Связанные темы

- [[Интерфейсы и классы]]
- [[Модули и системы сборки]]
- [[Утилиты типов TypeScript]]