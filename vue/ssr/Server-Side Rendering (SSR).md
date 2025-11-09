# Server-Side Rendering (SSR)

Server-Side Rendering (SSR) - это техника рендеринга Vue приложений на сервере и отправки полностью сформированного HTML клиенту. Это улучшает SEO и время до первого отображения.

## Основные концепции

### [SSR Basics](SSR%20Basics.md)
Основы серверного рендеринга и его преимущества

### [SSR vs SSG vs CSR](ssr-ssg-csr.md)
Сравнение Server-Side Rendering, Static Site Generation и Client-Side Rendering

### [Nuxt.js](nuxt.md)
Фреймворк для SSR приложений на Vue

### [Data Fetching](data-fetching.md)
Получение данных на сервере и передача их клиенту

### [Hydration](hydration.md)
Процесс оживления серверного HTML клиентским JavaScript

### [Performance](performance.md)
Оптимизация производительности SSR приложений

## Архитектура SSR

### Клиентский код
```javascript
// В клиентском коде Vue монтируется поверх серверного HTML
import { createSSRApp } from 'vue'
import { createStore } from 'vuex'
import { createRouter } from 'vue-router'
import App from './App.vue'

export function createApp() {
  const app = createSSRApp(App)
  const store = createStore()
  const router = createRouter()
  
  return { app, store, router }
}
```

### Серверный код
```javascript
// Серверный рендеринг
import { renderToString } from '@vue/server-renderer'
import { createApp } from './app.js'

server.get('*', async (req, res) => {
  const { app } = createApp()
  
  const appContent = await renderToString(app)
  
  const html = `
    <div id="app">${appContent}</div>
  `
  
  res.end(html)
})
```

## Решение проблем SSR

### Код, специфичный для браузера
```javascript
// Проверка окружения
if (typeof window !== 'undefined') {
  // Клиентский код
  document.addEventListener('click', handler)
}
```

### Управление состоянием
```javascript
// Создание нового экземпляра состояния для каждого запроса
export function createStore() {
  return createStore({
    state: () => ({
      // начальное состояние
    })
  })
}
```

## Vue 3 SSR особенности

### Composition API в SSR
```javascript
// Composition API работает в SSR
import { ref, onMounted } from 'vue'

export default {
  setup() {
    const data = ref(null)
    
    // onMounted не выполнится на сервере
    onMounted(async () => {
      data.value = await fetchData()
    })
    
    return { data }
  }
}
```

## Инструменты SSR

### @vue/server-renderer
```bash
npm install @vue/server-renderer
```

### Nuxt 3
```bash
npm init nuxt
```

## Лучшие практики

### Изоморфный код
- Писать код, который работает как на сервере, так и на клиенте
- Избегать прямого доступа к browser API

### Управление кэшированием
- Кэширование результатов рендеринга для улучшения производительности
- Использование кэшей для частично динамических страниц

### Обработка ошибок
```javascript
// Обработка ошибок SSR
server.get('*', async (req, res) => {
  try {
    const appHtml = await renderToString(app)
    res.send(renderFullPage(appHtml))
  } catch (error) {
    console.error(error)
    res.status(500).send('Internal Server Error')
  }
})
```

## Связь с другими концепциями
- [[routing/Routing]] - маршрутизация в SSR приложениях
- [[performance/Performance]] - оптимизация SSR производительности
- [[state-management/State Management]] - управление состоянием в SSR