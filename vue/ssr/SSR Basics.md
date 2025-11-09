# SSR Basics

## Определение
Server-Side Rendering (SSR) - это техника рендеринга Vue приложений на сервере, при которой HTML-страницы генерируются на сервере и отправляются клиенту, а затем "оживляются" клиентским JavaScript. Это улучшает SEO, время до первого отображения и общее пользовательское восприятие производительности.

## Принцип работы SSR

### Клиентский рендеринг (CSR)
```
Клиент → Запрос HTML → Пустой HTML + JS → Vue монтируется → DOM генерируется
```

### Серверный рендеринг (SSR)
```
Клиент → Запрос → Vue рендерит HTML на сервере → Полный HTML → Vue "оживляет" на клиенте
```

## Основные преимущества SSR

1. **Лучшее SEO** - поисковые роботы получают полностью сформированный HTML
2. **Быстрое первое отображение** - пользователь видит контент быстрее
3. **Улучшенная производительность на слабых устройствах** - меньше работы для браузера

## Основные сложности SSR

1. **Ограничения окружения** - нет доступа к browser API на сервере
2. **Управление состоянием** - каждому запросу нужно собственное состояние
3. **Кэширование** - сложнее кэшировать динамический контент
4. **Разработка** - сложнее настроить и отлаживать

## Архитектура SSR приложения

### Серверная часть
```javascript
// server.js
import { createSSRApp } from 'vue'
import { renderToString } from '@vue/server-renderer'
import { createRouter } from 'vue-router'
import App from './App.vue'
import { createApp } from './app.js'

async function renderPage(url, manifest) {
  const { app, router } = createApp()

  // Установка маршрута для SSR
  await router.push(url)
  await router.isReady()

  // Рендеринг приложения в строку
  const appHtml = await renderToString(app)

  // Генерация полного HTML
  const html = `
    <!DOCTYPE html>
    <html>
      <head>
        <title>Мое SSR приложение</title>
      </head>
      <body>
        <div id="app">${appHtml}</div>
      </body>
    </html>
  `

  return html
}
```

### Клиентская часть
```javascript
// client.js
import { createSSRApp } from 'vue'
import { createApp } from './app.js'

// "Оживление" серверного HTML
const { app } = createApp()
app.mount('#app', true) // Режим гидрации
```

### Общий код (isomorphic)
```javascript
// app.js
import { createSSRApp } from 'vue'
import { createRouter } from 'vue-router'
import App from './App.vue'

export function createApp() {
  const app = createSSRApp(App)
  const router = createRouter({
    // конфигурация роутера
  })

  app.use(router)

  return { app, router }
}
```

## Обработка окружения

### Проверка окружения
```javascript
// environment.js
export const isServer = typeof window === 'undefined'
export const isClient = !isServer

// В компонентах
export default {
  data() {
    return {
      isMounted: false
    }
  },
  
  mounted() {
    // Этот код выполнится только на клиенте
    this.isMounted = true
    
    // Безопасная работа с browser API
    if (typeof window !== 'undefined') {
      window.addEventListener('resize', this.handleResize)
    }
  },
  
  beforeUnmount() {
    if (typeof window !== 'undefined') {
      window.removeEventListener('resize', this.handleResize)
    }
  }
}
```

### Универсальные компоненты
```vue
<template>
  <div>
    <h1>{{ title }}</h1>
    <p v-if="isClient">Клиентский рендер: {{ clientInfo }}</p>
    <p v-else>Серверный рендер</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const props = defineProps(['title'])
const isClient = typeof window !== 'undefined'
const clientInfo = ref('')

onMounted(() => {
  if (isClient) {
    clientInfo.value = `Ширина экрана: ${window.innerWidth}`
  }
})
</script>
```

## Управление состоянием в SSR

### Создание нового состояния для каждого запроса
```javascript
// store.js
import { createStore } from 'vuex'

export function createNewStore() {
  return createStore({
    state: () => ({
      // Обязательно функция, чтобы создавать новое состояние для каждого запроса
      user: null,
      posts: []
    }),
    mutations: {
      // мутации
    },
    actions: {
      // действия
    }
  })
}

// При создании приложения
export function createApp() {
  const app = createSSRApp(App)
  const store = createNewStore() // Новый store для каждого запроса!
  
  app.use(store)
  
  return { app, store }
}
```

### Серверные данные и prefetch
```javascript
// В компоненте
export default {
  async serverPrefetch() {
    // Этот метод будет вызван на сервере до рендеринга
    await this.fetchData()
  },
  
  data() {
    return {
      data: null
    }
  },
  
  methods: {
    async fetchData() {
      // Загрузка данных
      this.data = await api.getData(this.$route.params.id)
    }
  }
}
```

## Hydration (гидрация)

### Что такое гидрация
Гидрация - это процесс "оживления" статического HTML, сгенерированного на сервере, с помощью клиентского JavaScript.

### Проблемы гидрации
```vue
<!-- ПЛОХО: различие между сервером и клиентом -->
<template>
  <p>Серверное время: {{ serverTime }}</p>
  <p>Клиентское время: {{ clientTime }}</p>
</template>

<script setup>
import { ref } from 'vue'

// Эти значения будут разными на сервере и клиенте!
const serverTime = new Date() // Серверное время
const clientTime = ref(new Date()) // Клиентское время
</script>
```

### Правильная гидрация
```vue
<template>
  <p>Время: {{ time }}</p>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const time = ref('Загрузка...') // Одинаковое начальное состояние

onMounted(() => {
  // Только на клиенте обновляем время
  time.value = new Date().toString()
})
</script>
```

## Рендеринг маршрутов

### Роутинг в SSR
```javascript
// router.js
import { createRouter, createMemoryHistory, createWebHistory } from 'vue-router'

export function createRouter(isServer = false) {
  return createRouter({
    // Используем разные истории для сервера и клиента
    history: isServer ? createMemoryHistory() : createWebHistory(),
    routes: [
      // определение маршрутов
    ]
  })
}

// app.js
export function createApp({ url } = {}) {
  const isServer = typeof window === 'undefined'
  const app = createSSRApp(App)
  const router = createRouter(isServer)
  
  if (url && isServer) {
    router.push(url) // Устанавливаем маршрут на сервере
  }
  
  app.use(router)
  
  return { app, router }
}
```

## Сборка для SSR

### Конфигурация Vite для SSR
```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  
  build: {
    // Конфигурация для серверного бандла
    ssr: true,
    rollupOptions: {
      input: {
        // Входной файл для серверной части
        server: './src/entry-server.js'
      }
    }
  },
  
  // Конфигурация для клиентского бандла
  ssr: {
    noExternal: ['vue', 'vue-router'] // Не исключать из серверного бандла
  }
})
```

## Лучшие практики SSR

### 1. Изоморфный код
```javascript
// Правильно: безопасная проверка окружения
if (typeof window !== 'undefined') {
  // Клиентский код
  localStorage.setItem('key', value)
}
```

### 2. Управление сайд-эффектами
```javascript
// Вместо побочных эффектов в setup()
export default {
  setup() {
    const data = ref(null)
    
    // Не вызываем асинхронные операции прямо в setup!
    // Вместо этого используем жизненные циклы
    onMounted(() => {
      // Клиентские побочные эффекты
    })
    
    return { data }
  }
}
```

### 3. Кэширование
```javascript
// Кэширование результатов рендеринга
const cache = new Map()

async function renderPage(url) {
  if (cache.has(url)) {
    return cache.get(url)
  }
  
  const result = await actualRender(url)
  cache.set(url, result)
  
  return result
}
```

## Популярные решения SSR

### Nuxt.js
```bash
# Создание SSR приложения с Nuxt
npx nuxi@latest init my-ssr-app
```

```vue
<!-- pages/index.vue в Nuxt -->
<template>
  <div>
    <h1>{{ data.title }}</h1>
  </div>
</template>

<script setup>
// Nuxt автоматически обрабатывает SSR
const { data } = await useAsyncData('home', () => $fetch('/api/home'))
</script>
```

## Тестирование SSR
```javascript
// ssr.test.js
import { renderToString } from '@vue/server-renderer'
import App from '@/App.vue'

test('рендерится без ошибок на сервере', async () => {
  const html = await renderToString(App)
  
  expect(html).toContain('<div') // Должен содержать HTML
  expect(html).not.toEqual('') // Не должен быть пустым
})
```

## Сравнение подходов

| Подход | Преимущества | Недостатки |
|--------|-------------|------------|
| CSR | Простая разработка, SPA | Плохое SEO, медленный FCP |
| SSR | Хорошее SEO, быстрый FCP | Сложная разработка, больше нагрузка на сервер |
| SSG | Быстрая доставка, хорошее SEO | Подходит только для статического контента |
| CSR + SSG | Баланс между всеми | Более сложная архитектура |

## Связь с другими концепциями
- [[ssr/data-fetching]] - получение данных в SSR
- [[ssr/hydration]] - процесс оживления серверного HTML
- [[routing/Routing]] - маршрутизация в SSR приложениях
- [[performance/Performance]] - оптимизация SSR производительности