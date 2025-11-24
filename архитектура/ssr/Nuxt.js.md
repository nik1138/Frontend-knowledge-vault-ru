---
aliases: [NuxtJS, Nuxt.js Framework]
tags: [programming, framework, vue, ssr, web-development, nuxtjs]
---

# Nuxt.js - фреймворк для серверного рендеринга на Vue.js

## Общая информация

Nuxt.js - это фреймворк для Vue.js, который предоставляет абстракцию над Vue и позволяет легко создавать универсальные приложения с поддержкой серверного рендеринга (SSR), статической генерации (SSG) и клиентского рендеринга (CSR). В 2025 году Nuxt.js продолжает развиваться и остается одним из ведущих решений для разработки на Vue.js с поддержкой SSR.

## Архитектурные особенности

### Nuxt 3 архитектура
Nuxt 3 представляет собой полную перезапись фреймворка с акцентом на производительность и гибкость:

```
nuxt/
├── app/
│   ├── app.vue
│   ├── pages/
│   ├── layouts/
│   └── components/
├── server/
│   ├── api/
│   ├── middleware/
│   └── routes/
└── nuxt.config.ts
```

- Использование Composition API по умолчанию
- Автоматическая сборка и оптимизация
- Встроенная поддержка TypeScript

### Модульная архитектура
- Система модулей для расширения функциональности
- Автозагрузка компонентов
- Встроенная система плагинов

## Типы рендеринга в Nuxt.js

### Server-Side Rendering (SSR)
```vue
<template>
  <div>{{ data }}</div>
</template>

<script setup>
// Серверные данные
const { data } = await $fetch('/api/data');
</script>
```

Или с использованием asyncData:
```javascript
export default {
  async asyncData({ $axios }) {
    const data = await $axios.get('/api/data');
    return { data: data.data };
  }
}
```

- Каждый запрос генерирует новую HTML страницу
- Подходит для динамического контента
- Отличная поддержка SEO

### Static Site Generation (SSG)
```javascript
// nuxt.config.js
export default {
  target: 'static',
  generate: {
    routes: ['/users/1', '/users/2']
  }
}
```

- Генерация статических HTML файлов во время сборки
- Высокая производительность
- Подходит для статического контента

### Hybrid Rendering
Nuxt 3 поддерживает гибридный рендеринг:
```javascript
// pages/index.vue
definePageMeta({
  // Маршрут будет предварительно отрендерен
  prerender: true
})
```

## Роутинг и навигация

### Файловая система маршрутов
- Автоматическая генерация маршрутов из файловой структуры
- Поддержка динамических маршрутов `[slug].vue`
- Вложенные маршруты с помощью папок

### Навигация
```vue
<template>
  <!-- Декларативная навигация -->
  <NuxtLink to="/about">О нас</NuxtLink>
  
  <!-- Программная навигация -->
  <button @click="goToProfile">Профиль</button>
</template>

<script setup>
const router = useRouter();

function goToProfile() {
  router.push('/profile');
}
</script>
```

## Серверные API и обработка запросов

### Server Routes
```javascript
// server/api/users.get.js
export default defineEventHandler(async (event) => {
  // Получение query параметров
  const query = getQuery(event);
  
  // Получение данных
  const users = await getUsers(query);
  
  return { users };
});
```

### Middleware
```javascript
// server/middleware/auth.js
export default defineEventHandler(async (event) => {
  // Проверка аутентификации
  const token = getCookie(event, 'auth-token');
  
  if (!token) {
    return sendRedirect(event, '/login');
  }
});
```

## Гидрация в Nuxt.js [[Гидрация]]

Nuxt.js автоматически обрабатывает процесс гидрации:

- Клиентская часть активируется после загрузки HTML
- Состояние восстанавливается из серверных данных
- Обработчики событий привязываются к элементам DOM

## Composition API и хуки данных

### useAsyncData
```javascript
export default {
  async setup() {
    const { data: posts, pending, error } = await useAsyncData('posts', 
      () => $fetch('/api/posts')
    );
    
    return { posts, pending, error };
  }
}
```

### useFetch
```javascript
export default {
  setup() {
    const { data: user, refresh } = useFetch('/api/user');
    
    return { user, refresh };
  }
}
```

## Оптимизация производительности [[Оптимизация]]

### Code Splitting
- Автоматическое разделение кода по маршрутам
- Lazy loading компонентов
- Tree shaking неиспользуемого кода

### Image Optimization
```vue
<template>
  <NuxtImg
    src="/example.jpg"
    :width="300"
    :height="200"
    alt="Пример"
  />
</template>
```

### Module Bundling
- Использование Nitro для серверной части
- Встроенная оптимизация сборки
- Поддержка современных форматов (ESM, WASM)

## Российские особенности использования

### CDN и локализация
- Возможность настройки локальных CDN
- Поддержка российских шлюзов для развертывания
- Локализация дат, времени и чисел через i18n модуль

### Соответствие требованиям
- Возможность хранения данных в РФ
- Интеграция с российскими системами аутентификации
- Поддержка российских платежных систем

### Альтернативные хостинги
- Поддержка различных провайдеров хостинга
- Возможность самостоятельного хостинга
- Docker контейнеры для локального развертывания

## Сравнение с Next.js [[Next.js]]

| Особенность | Nuxt.js | Next.js |
|-------------|---------|---------|
| Экосистема | Vue.js | React |
| Философия | Множество встроенных функций | Минимализм |
| Конфигурация | Меньше конфигурации | Более гибкая настройка |
| Комьюнити | Меньше | Больше |

## Миграция с других решений

### С Vue CLI
- Возможность постепенной миграции
- Сохранение существующих компонентов
- Использование Composition API

### С Nuxt 2
- Значительные изменения в архитектуре
- Нужна переработка большинства компонентов
- Улучшенная производительность и типизация

## Лучшие практики

### Структура проекта
```
app/
├── assets/          # Стили, изображения
├── components/      # Переиспользуемые компоненты
├── composables/     # Composition функции
├── layouts/         # Макеты страниц
├── middleware/      # Роутинг мидлвары
├── pages/           # Страницы приложения
├── plugins/         # Vue плагины
├── public/          # Публичные файлы
├── server/          # Серверный код
└── utils/           # Вспомогательные функции
```

### Управление состоянием
- useState для простых случаев
- Pinia для сложных состояний
- Встроенная поддержка глобальных состояний

### Тестирование
- Vitest для юнит тестов
- Playwright для e2e тестов
- Специфические тесты для SSR компонентов

## Модули и экосистема

### Популярные модули
- @nuxtjs/tailwindcss - интеграция с Tailwind CSS
- @nuxtjs/i18n - международизация
- @nuxtjs/axios - HTTP клиент
- @nuxtjs/pwa - PWA функциональность

### Создание собственных модулей
```javascript
// modules/my-module.ts
export default defineNuxtModule({
  defaults: {
    option: 'default'
  },
  setup(options, nuxt) {
    // Логика модуля
  }
})
```

## Заключение

Nuxt.js в 2025 году остается мощным и гибким фреймворком для разработки на Vue.js с поддержкой SSR. Его модульная архитектура и богатая экосистема позволяют создавать масштабируемые приложения, адаптированные под российские реалии и требования.

Фреймворк активно развивается и предлагает современные подходы к разработке, включая встроенную поддержку TypeScript, Composition API и современные инструменты оптимизации.