---
aliases: ["SvelteKit Routing", "Svelte Navigation"]
tags: [routing, svelte, sveltekit, frontend, javascript]
---

# Роутинг в Svelte

Роутинг в Svelte - это механизм навигации между различными компонентами Svelte-приложения. В отличие от React и Vue, где роутинг реализуется через отдельные библиотеки, в SvelteKit роутинг является встроенной функцией, основывающейся на файловой структуре проекта.

## Основные понятия

SvelteKit - это фреймворк для Svelte, который предоставляет полный стек для создания веб-приложений. Роутинг в SvelteKit основан на концепции "file-based routing", где маршруты определяются структурой файлов в директории `src/routes`.

### Основные особенности роутинга в SvelteKit

- **File-based routing** - маршруты определяются структурой файлов
- **Server-side rendering (SSR)** - встроенная поддержка SSR
- **Static site generation (SSG)** - возможность генерации статических сайтов
- **Client-side navigation** - плавные переходы между страницами
- **Endpoint functions** - API-маршруты в том же месте, что и UI-маршруты

## Установка SvelteKit

```bash
npm create svelte@latest my-app
cd my-app
npm install
```

## Базовая структура файлов

```
my-app/
├── src/
│   ├── routes/
│   │   ├── +layout.svelte
│   │   ├── +page.svelte
│   │   ├── about/
│   │   │   └── +page.svelte
│   │   └── contact/
│   │       └── +page.svelte
│   └── app.html
├── static/
└── svelte.config.js
```

## Базовая реализация

### Создание маршрутов

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import { Outlet } from '@sveltejs/kit';
</script>

<nav>
  <a href="/">Главная</a>
  <a href="/about">О нас</a>
  <a href="/contact">Контакты</a>
</nav>

<slot /> <!-- Здесь будет отображаться текущая страница -->
```

```svelte
<!-- src/routes/+page.svelte -->
<h1>Главная страница</h1>
<p>Добро пожаловать на главную страницу!</p>
```

```svelte
<!-- src/routes/about/+page.svelte -->
<h1>О нас</h1>
<p>Информация о нашей компании</p>
```

```svelte
<!-- src/routes/contact/+page.svelte -->
<h1>Контакты</h1>
<p>Свяжитесь с нами</p>
```

## Динамические маршруты

### Параметры маршрута

```svelte
<!-- src/routes/users/[id]/+page.svelte -->
<script>
  import { page } from '$app/stores';
  import { error } from '@sveltejs/kit';
  
  let { data } = $page;
  
  // Функция load будет вызвана на сервере и клиенте
  export async function load({ params }) {
    try {
      const response = await fetch(`/api/users/${params.id}`);
      const user = await response.json();
      
      if (!response.ok) {
        throw error(404, 'Пользователь не найден');
      }
      
      return {
        user
      };
    } catch (err) {
      throw error(500, 'Ошибка при загрузке пользователя');
    }
  }
</script>

<h1>Профиль пользователя {data.user.name}</h1>
<p>ID: {$page.params.id}</p>
<p>Email: {data.user.email}</p>
```

### Специальные файлы

```javascript
// src/routes/users/[id]/+page.js (load функция)
/** @type {import('./$types').PageLoad} */
export async function load({ params }) {
  const response = await fetch(`/api/users/${params.id}`);
  const user = await response.json();
  
  return {
    user
  };
}
```

## Вложенные маршруты

### Структура вложенных маршрутов

```
src/routes/
├── +layout.svelte
├── dashboard/
│   ├── +layout.svelte
│   ├── +page.svelte
│   ├── settings/
│   │   └── +page.svelte
│   └── profile/
│       └── +page.svelte
```

```svelte
<!-- src/routes/dashboard/+layout.svelte -->
<script>
  import { invalidate } from '$app/navigation';
</script>

<aside>
  <nav>
    <a href="/dashboard">Панель</a>
    <a href="/dashboard/profile">Профиль</a>
    <a href="/dashboard/settings">Настройки</a>
  </nav>
</aside>

<main>
  <slot />
</main>
```

## Программная навигация

### Использование $app/navigation

```svelte
<!-- src/routes/navigation/+page.svelte -->
<script>
  import { goto, invalidate, beforeNavigate, afterNavigate } from '$app/navigation';
  import { page } from '$app/stores';
  
  const navigateToProfile = async () => {
    await goto('/users/123');
  };
  
  const navigateWithParams = () => {
    goto('/search?q=svelte&category=frameworks');
  };
  
  // Перехват навигации
  beforeNavigate(({ from, to }) => {
    if (to?.url.pathname === '/protected' && !isAuthenticated) {
      return false; // Отменить навигацию
    }
  });
  
  afterNavigate(() => {
    console.log('Навигация завершена на:', $page.url.pathname);
  });
</script>

<button on:click={navigateToProfile}>Перейти к профилю</button>
<button on:click={navigateWithParams}>Поиск</button>
```

## Защищенные маршруты

### Middleware для аутентификации

```javascript
// src/hooks.server.js
import { sequence } from '@sveltejs/kit/hooks';
import { auth } from '$lib/server/auth';

export const handle = sequence(auth);
```

```javascript
// src/lib/server/auth.js
import { redirect } from '@sveltejs/kit';

export async function auth({ event, resolve }) {
  // Проверка аутентификации
  const token = event.cookies.get('auth_token');
  const isAuthenticated = await validateToken(token);
  
  // Добавление информации о пользователе в событие
  event.locals.user = isAuthenticated ? await getUserFromToken(token) : null;
  
  // Защищенные маршруты
  if (event.url.pathname.startsWith('/dashboard') && !isAuthenticated) {
    throw redirect(302, '/login');
  }
  
  const response = await resolve(event);
  return response;
}
```

```javascript
// src/routes/dashboard/+page.server.js
import { redirect } from '@sveltejs/kit';

export async function load({ locals }) {
  if (!locals.user) {
    throw redirect(302, '/login');
  }
  
  return {
    user: locals.user
  };
}
```

## API маршруты (Endpoints)

```javascript
// src/routes/api/users/[id]/+server.js
import { json } from '@sveltejs/kit';

/** @type {import('./$types').RequestHandler} */
export async function GET({ params }) {
  const userId = params.id;
  
  // Получение данных пользователя
  const user = await getUserById(userId);
  
  if (!user) {
    return new Response('Пользователь не найден', { status: 404 });
  }
  
  return json(user);
}

export async function PUT({ params, request }) {
  const userId = params.id;
  const data = await request.json();
  
  const updatedUser = await updateUser(userId, data);
  
  return json(updatedUser);
}
```

## Роутинг с параметрами и query

### Параметры маршрута и query

```svelte
<!-- src/routes/search/+page.svelte -->
<script>
  import { page } from '$app/stores';
  
  let { data } = $page;
  
  export async function load({ url }) {
    const query = url.searchParams.get('q') || '';
    const category = url.searchParams.get('category') || 'all';
    const page = parseInt(url.searchParams.get('page')) || 1;
    
    const results = await searchItems(query, category, page);
    
    return {
      query,
      category,
      page,
      results
    };
  }
</script>

<h1>Результаты поиска для "{data.query}"</h1>
<p>Категория: {data.category}</p>
<p>Страница: {data.page}</p>

{#each data.results as item}
  <div class="item">
    <h3>{item.title}</h3>
    <p>{item.description}</p>
  </div>
{/each}
```

## Обработка ошибок

### Страница ошибки

```svelte
<!-- src/routes/+error.svelte -->
<script>
  import { page } from '$app/stores';
</script>

<h1>{$page.status}: {$page.error.message}</h1>

{#if $page.status === 404}
  <p>Страница не найдена</p>
{:else if $page.status === 500}
  <p>Внутренняя ошибка сервера</p>
{:else}
  <p>Произошла ошибка</p>
{/if}

<a href="/">Вернуться на главную</a>
```

## Продвинутые возможности

### Loading состояния

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import { navigating } from '$app/stores';
</script>

{#if $navigating}
  <div class="loading">Загрузка...</div>
{/if}

<slot />
```

### Prefetching

```svelte
<!-- src/routes/+page.svelte -->
<script>
  import { enhance } from '$app/forms';
</script>

<a href="/next-page" rel="prefetch">Следующая страница (с предзагрузкой)</a>

<!-- Или с помощью JavaScript -->
<script>
  import { prefetch } from '$app/navigation';
  
  const prefetchPage = () => {
    prefetch('/next-page');
  };
</script>
```

## SSR и SSG оптимизации

### Page options

```javascript
// src/routes/ssr-example/+page.js
export const csr = true;  // Включить клиентский рендеринг
export const ssr = true;  // Включить серверный рендеринг
export const prerender = true;  // Предварительная генерация
export const trailingSlash = 'ignore'; // Обработка завершающих слэшей
```

### Conditional rendering

```svelte
<!-- src/routes/conditional/+page.svelte -->
<script>
  import { browser } from '$app/environment';
  
  let mounted = false;
  
  $: if (browser && !mounted) {
    mounted = true;
  }
</script>

{#if mounted}
  <!-- Контент, который отображается только в браузере -->
  <p>Этот контент отображается только в браузере</p>
{/if}
```

## Практические рекомендации

### 1. Структура файлов

```
src/
├── routes/
│   ├── +layout.svelte
│   ├── +page.svelte
│   ├── +page.server.js
│   ├── +layout.server.js
│   ├── api/
│   │   └── [...path]/
│   │       └── +server.js
│   └── posts/
│       ├── [slug]/
│       │   ├── +page.svelte
│       │   └── +page.server.js
│       └── +page.svelte
├── lib/
├── app.html
└── hooks.server.js
```

### 2. Оптимизация производительности

```javascript
// src/routes/optimized/+page.js
// Ограничение количества одновременных запросов
export async function load({ fetch, params }) {
  const [posts, user] = await Promise.all([
    fetch('/api/posts').then(r => r.json()),
    fetch('/api/user').then(r => r.json())
  ]);
  
  return { posts, user };
}
```

### 3. Типизация (с TypeScript)

```typescript
// src/routes/typescript-example/+page.server.ts
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ params, locals }) => {
  const userId = params.id;
  
  // Типизированный пользователь
  const user: User | null = locals.user;
  
  if (!user) {
    return {
      status: 401,
      error: new Error('Необходима аутентификация')
    };
  }
  
  return {
    user
  };
};
```

### 4. Анимации при переходе

```svelte
<!-- src/routes/animated/+layout.svelte -->
<script>
  import { fade, fly } from 'svelte/transition';
  import { draw } from 'svelte/animate';
</script>

<slot 
  name="child"
  let:child
  let:params
  let:data
>
  <div 
    in:fly={{ x: 300, duration: 300 }}
    out:fly={{ x: -300, duration: 300 }}
  >
    {@render child()}
  </div>
</slot>
```

## Сравнение с другими подходами

| Характеристика | SvelteKit | [[Клиентский-роутинг]] | [[Серверный-роутинг]] |
|----------------|-----------|------------------------|----------------------|
| Реализация | File-based | Встроенный механизм | Серверный код |
| SSR | Встроен | Требует дополнительной библиотеки | Встроен |
| Производительность | Высокая | Зависит от реализации | Зависит от сервера |
| SEO | Отличная | Требует дополнительной обработки | Хорошая |

## Заключение

Роутинг в Svelte и SvelteKit предоставляет мощный и интуитивный способ организации навигации в веб-приложениях. Благодаря файловой системе маршруты легко организовывать и поддерживать. Комбинация клиентской и серверной навигации, встроенные возможности SSR и SSG делают SvelteKit отличным выбором для современных веб-приложений.

Для получения дополнительной информации о роутинге смотрите:
- [[Клиентский-роутинг]]
- [[Серверный-роутинг]]
- [[Роутинг-в-React]]
- [[Роутинг-в-Vue]]

## Ключевые теги

#routing #svelte #sveltekit #frontend #javascript #ssr #ssg #web-development