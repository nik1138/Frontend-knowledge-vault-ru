---
aliases: ["Управление данными в React", "Паттерны данных React"]
tags: [react, data-handling, patterns, best-practices]
---

# Паттерны обработки данных в React

## Введение

Обработка данных в React — это критический аспект разработки приложений, который включает в себя управление состоянием, получение данных с сервера, кэширование, обработку ошибок и многое другое. В этом руководстве мы рассмотрим основные паттерны для эффективной работы с данными в React-приложениях.

## Клиентское управление данными

Клиентское управление данными включает в себя хранение и управление состоянием приложения на стороне клиента. Основные подходы:

- `useState` и `useReducer` для локального состояния компонентов
- [[react/context]] для передачи данных между компонентами без пропсов
- Внешние библиотеки: [[react/state-management]] (Redux, Zustand, Jotai)

### Пример использования useState

```jsx
import { useState } from 'react';

function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);

  const fetchUser = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/user');
      const userData = await response.json();
      setUser(userData);
    } catch (error) {
      console.error('Error fetching user:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      {loading ? <p>Загрузка...</p> : <p>Привет, {user?.name}!</p>}
      <button onClick={fetchUser}>Загрузить профиль</button>
    </div>
  );
}
```

## Получение данных с сервера

Существует несколько подходов для получения данных с сервера в React-приложениях:

- Классический способ: с помощью `useEffect`
- Современные библиотеки: [[react-query]] и SWR
- SSR/SSG: [[nextjs]] и [[gatsby]] для предварительной загрузки данных

### Использование useEffect

```jsx
import { useState, useEffect } from 'react';

function ProductList() {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchProducts = async () => {
      try {
        const response = await fetch('/api/products');
        const data = await response.json();
        setProducts(data);
      } catch (error) {
        console.error('Ошибка загрузки продуктов:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchProducts();
  }, []);

  if (loading) return <div>Загрузка...</div>;

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

## Паттерны потока данных

### Однонаправленный поток данных

В React реализуется однонаправленный поток данных (unidirectional data flow), где данные передаются от родительских компонентов к дочерним через пропсы. Это упрощает отладку и делает приложение более предсказуемым.

### Паттерн Flux

Flux — архитектурный паттерн от Facebook, который использует однонаправленный поток данных. Включает в себя:
- Actions — события, инициирующие изменения
- Dispatcher — центральный хаб для обработки действий
- Stores — хранение состояния
- Views — компоненты React

> [!tip] Совет
> Для реализации паттерна Flux в React часто используют Redux, который является производной от Flux.

## Локальная и удаленная синхронизация данных

Синхронизация локальных и удаленных данных важна для обеспечения согласованности информации. Основные проблемы:
- Состояние гонки (race conditions)
- Обновление данных в реальном времени
- Обработка конфликтов

### Паттерн "Single Source of Truth"

Все данные должны иметь один источник истины, обычно это состояние, управляемое внешней библиотекой, такой как Redux или Zustand.

## Оптимистические обновления

Оптимистические обновления улучшают UX, обновляя UI до получения ответа от сервера:

```jsx
const updateProduct = async (productId, newData) => {
  // Оптимистическое обновление
  setProducts(prev => 
    prev.map(p => p.id === productId ? { ...p, ...newData } : p)
  );

  try {
    await fetch(`/api/products/${productId}`, {
      method: 'PUT',
      body: JSON.stringify(newData),
    });
  } catch (error) {
    // В случае ошибки восстанавливаем предыдущее состояние
    setProducts(prev => 
      prev.map(p => p.id === productId ? { ...p, ...originalData } : p)
    );
  }
};
```

## Стратегии кэширования

### Кэширование на клиенте

- HTTP-кеширование
- Кэширование в памяти (React Query, SWR)
- Кэширование в localStorage/sessionStorage
- Кэширование с помощью библиотек (Redux Toolkit Query, Apollo)

### Кэширование с React Query

```jsx
import { useQuery } from 'react-query';

function UserComponent({ userId }) {
  const { data, isLoading, error } = useQuery(
    ['user', userId], // уникальный ключ запроса
    () => fetchUser(userId),
    {
      cacheTime: 5 * 60 * 1000, // 5 минут
      staleTime: 1 * 60 * 1000, // 1 минута до устаревания
    }
  );

  if (isLoading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;

  return <div>{data.name}</div>;
}
```

## Нормализация данных

Нормализация данных — это процесс структурирования данных так, чтобы избежать дублирования. Вместо вложенных объектов данные хранятся в плоской структуре с ссылками.

```js
// До нормализации
const users = [
  { id: 1, name: 'Иван', posts: [{ id: 1, title: 'Пост 1' }] }
];

// После нормализации
const entities = {
  users: { 1: { id: 1, name: 'Иван', postIds: [1] } },
  posts: { 1: { id: 1, title: 'Пост 1', userId: 1 } }
};
```

## Пагинация

### Пагинация на основе страниц

```jsx
function ProductList() {
  const [page, setPage] = useState(1);
  const { data, isLoading } = useQuery(
    ['products', page],
    () => fetchProducts(page)
  );

  return (
    <div>
      {data?.products.map(product => (
        <div key={product.id}>{product.name}</div>
      ))}
      <button onClick={() => setPage(prev => prev + 1)}>Следующая страница</button>
    </div>
  );
}
```

## Бесконечный скролл

```jsx
import { useInfiniteQuery } from 'react-query';

function InfiniteScroll() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isLoading,
    isFetchingNextPage
  } = useInfiniteQuery(
    'projects',
    ({ pageParam = 1 }) => fetchProjects(pageParam),
    {
      getNextPageParam: (lastPage, pages) => {
        return lastPage.hasNextPage ? pages.length + 1 : undefined;
      },
    }
  );

  return (
    <div>
      {data?.pages.map((group, i) => (
        <div key={i}>
          {group.projects.map(project => (
            <div key={project.id}>{project.title}</div>
          ))}
        </div>
      ))}
      <button
        onClick={() => fetchNextPage()}
        disabled={!hasNextPage || isFetchingNextPage}
      >
        {isFetchingNextPage
          ? 'Загрузка...'
          : hasNextPage
          ? 'Загрузить еще'
          : 'Больше нет'}
      </button>
    </div>
  );
}
```

## Фильтрация и поиск

Фильтрация может происходить на клиенте или сервере:

```jsx
function SearchableList({ items }) {
  const [query, setQuery] = useState('');
  
  const filteredItems = useMemo(() => {
    if (!query) return items;
    return items.filter(item => 
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [items, query]);

  return (
    <div>
      <input 
        type="text" 
        value={query} 
        onChange={e => setQuery(e.target.value)} 
        placeholder="Поиск..." 
      />
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## Паттерны преобразования данных

### Композиция функций

```jsx
const processUserData = (rawData) => {
  return rawData
    .filter(user => user.active)
    .map(user => ({ ...user, fullName: `${user.firstName} ${user.lastName}` }))
    .sort((a, b) => a.fullName.localeCompare(b.fullName));
};
```

## Обработка ошибок в операциях с данными

### Глобальная обработка ошибок

```jsx
function DataComponent() {
  const { data, error, isLoading } = useQuery(
    'data',
    fetchData,
    {
      onError: (error) => {
        // Логирование ошибки
        console.error('Ошибка получения данных:', error);
        // Отображение пользовательского интерфейса ошибки
        showErrorNotification(error.message);
      }
    }
  );

  if (error) return <ErrorMessage error={error} />;
  if (isLoading) return <LoadingSpinner />;
  
  return <DataDisplay data={data} />;
}
```

## Состояния загрузки

Важно предоставлять пользователю обратную связь во время загрузки данных:

- Индикаторы загрузки
- Плейсхолдеры
- Skeleton-экраны
- Оптимистические обновления

## Обработка устаревших данных

### stale-while-revalidate

```jsx
const { data } = useQuery('data', fetchData, {
  staleTime: 1000 * 60 * 5, // 5 минут до устаревания
  cacheTime: 1000 * 60 * 10, // 10 минут в кэше
});
```

## Паттерны согласованности данных

- Идемпотентные операции
- Транзакции (где возможно)
- Согласование конфликтов
- Версионирование данных

## Использование React Query и SWR

### React Query

React Query предоставляет мощные возможности для управления серверными данными:

```jsx
import { QueryClient, QueryClientProvider, useQuery } from 'react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MyComponents />
    </QueryClientProvider>
  );
}

function MyComponent() {
  const { data, isLoading, error } = useQuery(
    'userData',
    fetchUserData
  );
  
  if (isLoading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;
  
  return <div>{data.name}</div>;
}
```

### SWR

SWR (stale-while-revalidate) — библиотека от Vercel:

```jsx
import useSWR from 'swr';

function Profile() {
  const { data, error } = useSWR('/api/user', fetcher);

  if (error) return <div>Failed to load</div>;
  if (!data) return <div>Loading...</div>;
  return <div>Hello {data.name}!</div>;
}
```

## Пользовательские хуки для данных

Создание пользовательских хуков помогает повторно использовать логику получения данных:

```jsx
function useUser(userId) {
  return useQuery(
    ['user', userId],
    () => fetchUser(userId),
    {
      enabled: !!userId, // выполняется только если userId существует
    }
  );
}

function UserProfile({ userId }) {
  const { data: user, isLoading, error } = useUser(userId);
  
  if (isLoading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error.message}</div>;
  
  return <div>{user.name}</div>;
}
```

## Рассмотрение производительности при работе с данными

### Оптимизация рендеринга

- Использование `React.memo` для предотвращения ненужных рендеров
- Правильное использование `useMemo` и `useCallback`
- Виртуализация списков с помощью `react-window`

### Оптимизация запросов

- Дебаунсинг запросов
- Батчинг запросов
- Удаление неиспользуемых запросов
- Эффективное кэширование

## Заключение

Эффективное управление данными в React требует понимания различных паттернов и инструментов. Выбор подходящего подхода зависит от конкретных требований приложения, сложности состояния и архитектурных решений.

Для более глубокого понимания рекомендуется ознакомиться с:
- [[react/performance]]
- [[react/error-boundaries]]
- [[react/state-management]]
- [[react/hooks]]
- [[react/context]]
- [[react-query]]
- [[swr]]

> [!note] Примечание
> Правильное управление данными критически важно для производительности и надежности React-приложений.