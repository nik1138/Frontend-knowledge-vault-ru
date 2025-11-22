---
aliases: [Svelte Context, Контекст в Svelte, Svelte Context API]
tags: [svelte, context, state-management, frontend]
---

# Контекст-в-Svelte

## Обзор

В Svelte контекст реализуется через API `setContext` и `getContext`, которые позволяют компонентам передавать данные вниз по дереву компонентов без необходимости передавать пропсы на каждом уровне. Это встроенная функция Svelte, которая предоставляет простой способ для компонентов обмениваться информацией в пределах поддерева компонентов.

## Зачем использовать контекст в Svelte?

Контекст в Svelte решает проблему "prop drilling" (пробуривания пропсов), когда данные нужно передать через несколько уровней компонентов. Он позволяет родительским компонентам устанавливать контекст, который затем может быть получен любым дочерним компонентом в поддереве, независимо от глубины вложенности.

## Основные концепции

### setContext

Функция `setContext` позволяет компоненту установить значение в контексте, которое будет доступно всем его дочерним компонентам. Контекст устанавливается при инициализации компонента и доступен только в поддереве этого компонента.

### getContext

Функция `getContext` позволяет компоненту получить значение из контекста, установленное одним из его родительских компонентов. Если значение не найдено, возвращается `undefined`.

### hasContext

Функция `hasContext` проверяет, существует ли значение в контексте с заданным ключом.

## Практическое применение

### Базовое использование

```svelte
<!-- Parent.svelte -->
<script>
  import { setContext } from 'svelte';
  import Child from './Child.svelte';
  
  // Устанавливаем контекст
  setContext('theme', 'dark');
  setContext('user', { name: 'Alice', role: 'admin' });
</script>

<div class="parent">
  <h2>Родительский компонент</h2>
  <Child />
</div>

<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';
  import GrandChild from './GrandChild.svelte';
  
  // Получаем контекст
  const theme = getContext('theme');
  const user = getContext('user');
</script>

<div class="child" class:dark={theme === 'dark'}>
  <p>Тема: {theme}</p>
  <p>Пользователь: {user.name}</p>
  <GrandChild />
</div>

<!-- GrandChild.svelte -->
<script>
  import { getContext } from 'svelte';
  
  // Получаем контекст из родительского компонента
  const theme = getContext('theme');
  const user = getContext('user');
</script>

<div class="grandchild">
  <p>Тема во внуке: {theme}</p>
  <p>Роль: {user.role}</p>
</div>
```

### Использование объектов для группировки контекста

```svelte
<!-- App.svelte -->
<script>
  import { setContext } from 'svelte';
  import ThemeProvider from './ThemeProvider.svelte';
  
  // Группируем связанные данные
  const appContext = {
    theme: 'light',
    language: 'ru',
    apiUrl: 'https://api.example.com'
  };
  
  setContext('app', appContext);
</script>

<ThemeProvider>
  <main>
    <!-- Остальные компоненты -->
  </main>
</ThemeProvider>

<!-- ThemeProvider.svelte -->
<script>
  import { setContext, createEventDispatcher } from 'svelte';
  import { getContext } from 'svelte';
  
  const dispatch = createEventDispatcher();
  
  // Получаем основной контекст
  const appContext = getContext('app');
  
  // Расширяем контекст
  const themeContext = {
    ...appContext,
    toggleTheme: () => {
      const newTheme = appContext.theme === 'light' ? 'dark' : 'light';
      dispatch('themeChange', newTheme);
    }
  };
  
  setContext('theme', themeContext);
</script>

<div class="theme-provider">
  <slot />
</div>
```

### Использование символов в качестве ключей

Для избежания конфликта имен рекомендуется использовать символы в качестве ключей:

```javascript
// contextKeys.js
export const THEME_KEY = Symbol('theme');
export const USER_KEY = Symbol('user');
export const API_KEY = Symbol('api');

// Parent.svelte
<script>
  import { setContext } from 'svelte';
  import { THEME_KEY, USER_KEY } from './contextKeys.js';
  
  setContext(THEME_KEY, 'dark');
  setContext(USER_KEY, { name: 'Bob', permissions: [] });
</script>

<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';
  import { THEME_KEY, USER_KEY } from './contextKeys.js';
  
  const theme = getContext(THEME_KEY);
  const user = getContext(USER_KEY);
</script>
```

## Примеры использования

### 1. Управление темой приложения

```svelte
<!-- ThemeContext.svelte -->
<script>
  import { setContext, createEventDispatcher } from 'svelte';
  import { readable, writable } from 'svelte/store';
  
  export let initialTheme = 'light';
  
  const { subscribe, set } = writable(initialTheme);
  
  const themeContext = {
    subscribe,
    setTheme: set,
    toggleTheme: () => {
      subscribe(current => {
        set(current === 'light' ? 'dark' : 'light');
      });
    }
  };
  
  setContext('theme', themeContext);
</script>

<slot />

<!-- ThemedComponent.svelte -->
<script>
  import { getContext } from 'svelte';
  
  const themeStore = getContext('theme');
</script>

<div class="themed-component">
  {#if $themeStore === 'dark'}
    <p class="dark-theme">Темная тема активна</p>
  {:else}
    <p class="light-theme">Светлая тема активна</p>
  {/if}
</div>
```

### 2. Управление аутентификацией

```svelte
<!-- AuthContext.svelte -->
<script>
  import { setContext } from 'svelte';
  import { writable } from 'svelte/store';
  
  // Создаем хранилище для состояния аутентификации
  const authStore = writable({
    user: null,
    isAuthenticated: false,
    token: null
  });
  
  const authContext = {
    ...authStore,
    login: async (credentials) => {
      try {
        const response = await fetch('/api/login', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(credentials)
        });
        
        if (response.ok) {
          const data = await response.json();
          authStore.set({
            user: data.user,
            isAuthenticated: true,
            token: data.token
          });
          return true;
        }
      } catch (error) {
        console.error('Login failed:', error);
      }
      return false;
    },
    logout: () => {
      authStore.set({
        user: null,
        isAuthenticated: false,
        token: null
      });
    }
  };
  
  setContext('auth', authContext);
</script>

<slot />
```

### 3. Управление API-клиентом

```svelte
<!-- ApiContext.svelte -->
<script>
  import { setContext } from 'svelte';
  
  class ApiClient {
    constructor(baseUrl) {
      this.baseUrl = baseUrl;
    }
    
    async request(endpoint, options = {}) {
      const url = `${this.baseUrl}${endpoint}`;
      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        ...options
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      return response.json();
    }
    
    get(endpoint) {
      return this.request(endpoint, { method: 'GET' });
    }
    
    post(endpoint, data) {
      return this.request(endpoint, {
        method: 'POST',
        body: JSON.stringify(data)
      });
    }
  }
  
  // Создаем экземпляр API-клиента
  const apiClient = new ApiClient('https://api.example.com');
  
  setContext('api', apiClient);
</script>

<slot />

<!-- DataComponent.svelte -->
<script>
  import { getContext } from 'svelte';
  import { onMount } from 'svelte';
  
  const api = getContext('api');
  let data = null;
  
  onMount(async () => {
    try {
      data = await api.get('/data');
    } catch (error) {
      console.error('Failed to fetch data:', error);
    }
  });
</script>

{#if data}
  <div>
    {#each data as item (item.id)}
      <p>{item.name}</p>
    {/each}
  </div>
{:else}
  <p>Загрузка данных...</p>
{/if}
```

## Лучшие практики

### 1. Используйте модули для определения ключей контекста

```javascript
// contexts/index.js
export const THEME_CONTEXT = Symbol('theme');
export const AUTH_CONTEXT = Symbol('auth');
export const API_CONTEXT = Symbol('api');

// components/ThemeProvider.svelte
<script>
  import { setContext } from 'svelte';
  import { THEME_CONTEXT } from '../contexts/index.js';
  
  setContext(THEME_CONTEXT, themeValue);
</script>
```

### 2. Оборачивайте контекст в пользовательские хуки

Для более сложной логики можно создать пользовательские функции:

```javascript
// lib/useContext.js
import { getContext, hasContext } from 'svelte';
import { writable, readable } from 'svelte/store';

export function useThemeContext() {
  const THEME_KEY = Symbol('theme');
  
  if (hasContext(THEME_KEY)) {
    return getContext(THEME_KEY);
  }
  
  // Если контекст не установлен, создаем дефолтное значение
  const theme = writable('light');
  
  const context = {
    theme,
    toggleTheme: () => {
      theme.update(current => current === 'light' ? 'dark' : 'light');
    }
  };
  
  return context;
}
```

### 3. Обрабатывайте отсутствие контекста

```svelte
<!-- SafeComponent.svelte -->
<script>
  import { getContext, hasContext } from 'svelte';
  
  const REQUIRED_CONTEXT = Symbol('required');
  
  let context;
  let hasCtx = hasContext(REQUIRED_CONTEXT);
  
  if (hasCtx) {
    context = getContext(REQUIRED_CONTEXT);
  } else {
    // Логика для обработки отсутствия контекста
    console.warn('Required context not found, using defaults');
    context = { theme: 'light', user: null };
  }
</script>

{#if hasCtx}
  <div>Контекст доступен: {context.theme}</div>
{:else}
  <div>Контекст недоступен, используем значения по умолчанию</div>
{/if}
```

### 4. Используйте TypeScript для строгой типизации

```typescript
// contexts/theme.ts
import type { Writable } from 'svelte/store';

export interface ThemeContext {
  theme: Writable<'light' | 'dark'>;
  toggleTheme: () => void;
}

export const THEME_CONTEXT = Symbol('theme') as symbol & {
  brand: 'ThemeContext';
};
```

## Потенциальные проблемы и решения

### 1. Отладка контекста

Поскольку связи между компонентами неявны, отладка может быть затруднена. Решения:
- Используйте инструменты разработчика Svelte
- Добавляйте логирование при установке и получении контекста
- Документируйте использование контекста в компонентах

### 2. Тестирование

При тестировании компонентов, использующих контекст, необходимо устанавливать соответствующий контекст:

```javascript
// tests/Component.test.js
import { render, screen } from '@testing-library/svelte';
import { setContext } from 'svelte';
import Component from '../Component.svelte';

test('component uses context value', () => {
  // Создаем mock-компонент для установки контекста
  const TestWrapper = {
    template: '<slot />',
    onMount() {
      setContext('theme', 'dark');
    }
  };
  
  render(Component, { 
    // Устанавливаем контекст перед рендерингом компонента
  });
  
  expect(screen.getByText(/dark/i)).toBeInTheDocument();
});
```

### 3. Изоляция контекста

Контекст в Svelte изолирован в пределах поддерева компонентов, что является преимуществом, но требует понимания архитектуры приложения.

## Сравнение с другими решениями

### Context API в Svelte vs Svelte Stores

| Характеристика | Context API | Svelte Stores |
|---|---|---|
| Область видимости | Поддерево компонентов | Глобальная или импортируемая |
| Реактивность | Через передаваемые значения | Встроенная в stores |
| Сложность настройки | Низкая | Низкая |
| Использование | Для передачи данных вниз по дереву | Для глобального состояния |

Context API в Svelte лучше подходит для передачи данных вниз по дереву компонентов, в то время как Svelte Stores лучше подходят для глобального управления состоянием.

## Ссылки и ресурсы

- [[Провайдеры-контекста]]
- [[Потребители-контекста]]
- [[Глобальное-состояние]]
- [[Компонентная-архитектура]]
- [[Хранилища-данных]]
- [[Слоты]]

## Заключение

Контекст в Svelte предоставляет простой и эффективный способ передачи данных через дерево компонентов без необходимости передавать пропсы на каждом уровне. При правильном использовании он может значительно упростить архитектуру приложения и улучшить его поддерживаемость. Однако важно понимать ограничения контекста и использовать его там, где это действительно необходимо, не заменяя им полнофункциональные решения для управления глобальным состоянием.