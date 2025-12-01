---
aliases: ["SSR деплой Vue", "Vue Server-Side Rendering", "Vue SSR"]
tags: [vue, ssr, deployment, server-side-rendering, nuxt]
---

# SSR-деплой Vue.js приложений

## Обзор

Server-Side Rendering (SSR) позволяет генерировать HTML-страницы на сервере вместо рендеринга в браузере. Это особенно полезно для SEO и улучшения первоначальной загрузки страницы. В контексте Vue.js SSR может быть реализован как с помощью Nuxt.js, так и с помощью чистого Vue.

## Преимущества SSR

- **Улучшенное SEO**: Поисковые роботы получают готовый HTML
- **Быстрая первоначальная загрузка**: Пользователь видит контент быстрее
- **Лучшая производительность на слабых устройствах**: Часть работы выполняется на сервере
- **Улучшенный пользовательский опыт**: Снижение времени до интерактивности

## Подготовка приложения к SSR

### С использованием Nuxt.js

Nuxt.js предоставляет встроенную поддержку SSR:

1. Установка Nuxt:
```bash
npx nuxi@latest init my-nuxt-app
```

2. Настройка `nuxt.config.ts`:
```typescript
export default defineNuxtConfig({
  // Настройки SSR
  ssr: true,
  
  // Публичный путь
  app: {
    baseURL: process.env.BASE_URL || '/'
  },
  
  // Настройки рендеринга
  nitro: {
    preset: 'node-server', // или 'vercel', 'netlify', 'static'
    compressPublicAssets: true
  },
  
  // Настройки безопасности
  security: {
    headers: {
      crossOriginEmbedderPolicy: 'unsafe-none',
      crossOriginOpenerPolicy: 'same-origin',
      crossOriginResourcePolicy: 'cross-origin',
      referrerPolicy: 'no-referrer'
    }
  }
})
```

### Чистый Vue с SSR

Для чистого Vue.js потребуется настройка серверного рендеринга:

1. Создание серверного entry point (`server.js`):
```javascript
import { createSSRApp } from 'vue'
import { renderToString } from '@vue/server-renderer'
import { createRouter } from './router'
import App from './App.vue'

const express = require('express')
const server = express()

server.get('*', async (req, res) => {
  const app = createSSRApp(App)
  const router = createRouter()
  router.push(req.url)
  await router.isReady()
  
  const html = await renderToString(app)
  
  const template = `
    <!DOCTYPE html>
    <html>
      <head>
        <title>Vue SSR App</title>
      </head>
      <body>
        <div id="app">${html}</div>
        <script src="/client.js"></script>
      </body>
    </html>
  `
  
  res.send(template)
})
```

## Популярные платформы для SSR-деплоя

### Vercel

Vercel предоставляет отличную поддержку для Nuxt.js приложений:

1. В `nuxt.config.ts` укажите preset:
```typescript
export default defineNuxtConfig({
  nitro: {
    preset: 'vercel'
  }
})
```

2. Добавьте в `package.json`:
```json
{
  "scripts": {
    "build": "nuxt build",
    "dev": "nuxt dev"
  }
}
```

### Node.js сервер

Для размещения на собственном сервере:

1. Сборка приложения:
```bash
npm run build
```

2. Запуск сервера:
```bash
node .output/server/index.mjs
```

### Яндекс.Облако (Compute Cloud)

В 2025 году Яндекс.Облако активно развивает Compute Cloud для размещения SSR-приложений:

1. Создайте виртуальную машину
2. Установите Node.js
3. Загрузите и разверните приложение
4. Настройте nginx как reverse proxy

### AWS/GCP (с ограничениями)

Важно учитывать санкционные ограничения и возможность ограничения доступа к иностранным облакам.

## Российские особенности SSR-деплоя

### Локальные провайдеры

- [[МегаФон.Облако]]
- [[Ростелеком.Облако]]
- [[СберОблако]]
- [[VK Cloud]]

### Региональные ограничения

- Учитывайте локализацию и хранение данных
- Обеспечьте резервные копии в РФ
- Соблюдайте требования 152-ФЗ

## Производительность и масштабирование

### Кэширование

Для улучшения производительности SSR-приложений:

```typescript
// В Nuxt.js
export default defineNuxtConfig({
  nitro: {
    storage: {
      cache: {
        driver: 'redis',
        host: process.env.REDIS_HOST,
        port: process.env.REDIS_PORT
      }
    }
  }
})
```

### Кэширование HTML

Используйте кэширование для часто запрашиваемых страниц:

```typescript
// В Nuxt.js
export default defineNuxtConfig({
  routeRules: {
    '/': { swr: 300 }, // Кэширование на 5 минут
    '/articles/**': { swr: 3600 } // Кэширование на 1 час
  }
})
```

### Load Balancing

Для масштабирования SSR-приложений используйте:

- NGINX как load balancer
- Docker контейнеры с [[Docker]]
- Kubernetes кластеры

## Безопасность SSR-приложений

### Защита от XSS

- Санитизация пользовательского ввода
- Использование безопасных шаблонов
- Правильная обработка заголовков

### CORS и безопасность API

- Настройка правильных CORS заголовков
- Использование HTTPS
- Валидация токенов аутентификации

## Мониторинг и логирование

Для SSR-приложений особенно важно:

- Мониторинг времени ответа сервера
- Отслеживание ошибок на сервере
- Логирование производительности
- [[Мониторинг]] с использованием APM-инструментов

## Связанные темы

- [[Vue.js]] - основы фреймворка
- [[Статический-деплой]] - альтернативный подход
- [[CI-CD]] - автоматизация процесса деплоя
- [[Docker]] - контейнеризация SSR-приложений
- [[Мониторинг]] - отслеживание состояния SSR-приложения

## Заключение

SSR-деплой Vue.js приложений требует больше ресурсов, но обеспечивает лучшее SEO и пользовательский опыт. При правильной реализации и учете российских реалий 2025 года это мощное решение для современных веб-приложений.