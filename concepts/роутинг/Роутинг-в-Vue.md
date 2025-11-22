---
aliases: ["Vue Router", "Vue Navigation"]
tags: [routing, vue, frontend, javascript]
---

# Роутинг в Vue

Роутинг в Vue - это механизм навигации между различными компонентами Vue-приложения без перезагрузки страницы. Для реализации роутинга в Vue-приложениях используется официальная библиотека Vue Router.

## Основные понятия

Vue Router - официальная библиотека маршрутизации для Vue.js. Она интегрируется с Vue.js и предоставляет возможности для:
- Определения маршрутов с помощью компонентов
- Навигации между страницами без перезагрузки
- Передачи параметров между маршрутами
- Управления историей браузера

### Основные компоненты Vue Router

- `<router-link>` - создает ссылки для навигации
- `<router-view>` - отображает соответствующий компонент для текущего маршрута
- `createRouter` - функция для создания экземпляра роутера
- `createWebHistory` - создает историю для работы с URL
- `useRoute` - хук для получения информации о текущем маршруте
- `useRouter` - хук для программной навигации

## Установка Vue Router

```bash
npm install vue-router@4
```

## Базовая реализация

### Настройка маршрутов

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import About from '../views/About.vue'
import Contact from '../views/Contact.vue'
import NotFound from '../views/NotFound.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    component: About
  },
  {
    path: '/contact',
    name: 'Contact',
    component: Contact
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: NotFound
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### Подключение к приложению

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(router)
app.mount('#app')
```

### Использование в компонентах

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <nav>
      <router-link to="/">Главная</router-link>
      <router-link to="/about">О нас</router-link>
      <router-link to="/contact">Контакты</router-link>
    </nav>
    
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
```

## Продвинутые возможности

### Динамические маршруты

```javascript
// router/index.js
{
  path: '/users/:id',
  name: 'UserProfile',
  component: UserProfile
}

// UserProfile.vue
<template>
  <div>
    <h1>Профиль пользователя {{ $route.params.id }}</h1>
  </div>
</template>

<script>
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    
    // Реагирование на изменения параметров
    watch(() => route.params.id, (newId, oldId) => {
      // Обработка изменения ID
    })
    
    return {}
  }
}
</script>
```

### Использование Composition API

```vue
<!-- UserProfile.vue -->
<template>
  <div>
    <h1>Пользователь: {{ userId }}</h1>
    <p>Имя: {{ user.name }}</p>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { getUserById } from '@/api/users'

export default {
  setup() {
    const route = useRoute()
    const user = ref({})
    const userId = computed(() => route.params.id)
    
    onMounted(async () => {
      user.value = await getUserById(userId.value)
    })
    
    return {
      user,
      userId
    }
  }
}
</script>
```

### Вложенные маршруты

```javascript
// router/index.js
{
  path: '/user',
  component: UserLayout,
  children: [
    {
      path: '',
      name: 'UserProfile',
      component: UserProfile
    },
    {
      path: 'settings',
      name: 'UserSettings',
      component: UserSettings
    },
    {
      path: 'posts',
      name: 'UserPosts',
      component: UserPosts
    }
  ]
}
```

```vue
<!-- UserLayout.vue -->
<template>
  <div class="user-layout">
    <aside class="user-sidebar">
      <router-link to="/user">Профиль</router-link>
      <router-link to="/user/settings">Настройки</router-link>
      <router-link to="/user/posts">Посты</router-link>
    </aside>
    
    <main class="user-content">
      <router-view />
    </main>
  </div>
</template>
```

## Хуки и навигация

### useRoute и useRouter

```vue
<script>
import { useRoute, useRouter } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    const router = useRouter()
    
    // Получение параметров
    const userId = route.params.id
    
    // Программная навигация
    const goToProfile = () => {
      router.push(`/user/${userId}`)
    }
    
    const goBack = () => {
      router.go(-1)
    }
    
    return {
      userId,
      goToProfile,
      goBack
    }
  }
}
</script>
```

### Навигационные хуки

```javascript
// Глобальные хуки
router.beforeEach((to, from, next) => {
  // Проверка аутентификации
  if (to.name !== 'Login' && !isAuthenticated) {
    next({ name: 'Login' })
  } else {
    next()
  }
})

// Хуки конкретного маршрута
{
  path: '/dashboard',
  component: Dashboard,
  beforeEnter: (to, from, next) => {
    // Проверка прав доступа
    if (userHasAccess(to.params.id)) {
      next()
    } else {
      next('/unauthorized')
    }
  }
}

// Хуки компонента
export default {
  beforeRouteEnter(to, from, next) {
    // Выполняется до входа в маршрут
    // this недоступен (компонент еще не создан)
    next()
  },
  
  beforeRouteUpdate(to, from, next) {
    // Выполняется при обновлении параметров маршрута
    // this доступен
    next()
  },
  
  beforeRouteLeave(to, from, next) {
    // Выполняется перед покиданием маршрута
    // Подтверждение перед уходом
    const answer = window.confirm('Вы уверены, что хотите покинуть эту страницу?')
    if (answer) {
      next()
    } else {
      next(false)
    }
  }
}
```

## Защищенные маршруты

```javascript
// router/index.js
import { useAuthStore } from '@/stores/auth'

// Глобальный навигационный хук
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore()
  
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    next({
      path: '/login',
      query: { redirect: to.fullPath }
    })
  } else {
    next()
  }
})

// Определение маршрутов с метаданными
{
  path: '/dashboard',
  name: 'Dashboard',
  component: Dashboard,
  meta: { requiresAuth: true }
}
```

## Lazy Loading

```javascript
// router/index.js
const Home = () => import('../views/Home.vue')
const About = () => import('../views/About.vue')

// Или с именованными чанками
const UserProfile = () => import(/* webpackChunkName: "user" */ '../views/UserProfile.vue')
const UserSettings = () => import(/* webpackChunkName: "user" */ '../views/UserSettings.vue')
```

## Роутинг с параметрами

### Параметры маршрута

```javascript
// router/index.js
{
  path: '/users/:userId/posts/:postId',
  name: 'PostDetail',
  component: PostDetail
}

// PostDetail.vue
<script>
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    
    // Получение параметров
    const userId = route.params.userId
    const postId = route.params.postId
    
    return {
      userId,
      postId
    }
  }
}
</script>
```

### Query параметры

```vue
<script>
import { useRoute } from 'vue-router'

export default {
  setup() {
    const route = useRoute()
    
    // Получение query параметров
    const category = route.query.category || 'all'
    const page = parseInt(route.query.page) || 1
    
    return {
      category,
      page
    }
  }
}
</script>
```

## Обработка ошибок

```javascript
// Глобальная обработка ошибок роутинга
router.onError((error, to) => {
  if (error.name === 'ChunkLoadError') {
    // Ошибка загрузки чанка
    alert('Произошла ошибка при загрузке страницы. Пожалуйста, обновите страницу.')
  }
  console.error('Ошибка роутинга:', error)
})
```

## Практические рекомендации

### 1. Структура файлов

```
src/
├── components/
├── views/
│   ├── Home.vue
│   ├── About.vue
│   └── NotFound.vue
├── router/
│   └── index.js
├── stores/
└── App.vue
```

### 2. Централизованное определение маршрутов

```javascript
// router/modules/user.js
export default {
  path: '/user',
  component: UserLayout,
  meta: { requiresAuth: true },
  children: [
    {
      path: '',
      name: 'UserProfile',
      component: () => import('@/views/user/Profile.vue')
    },
    {
      path: 'settings',
      name: 'UserSettings',
      component: () => import('@/views/user/Settings.vue')
    }
  ]
}
```

```javascript
// router/index.js
import userRoutes from './modules/user'
import productRoutes from './modules/product'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  ...userRoutes,
  ...productRoutes
]
```

### 3. Типизация (с TypeScript)

```typescript
import { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/user/:id',
    name: 'UserProfile',
    component: UserProfile,
    props: true,
    meta: {
      requiresAuth: true
    }
  }
]
```

### 4. Анимации при переходе

```vue
<!-- App.vue -->
<template>
  <div id="app">
    <nav>
      <router-link to="/">Главная</router-link>
      <router-link to="/about">О нас</router-link>
    </nav>
    
    <transition name="fade" mode="out-in">
      <router-view />
    </transition>
  </div>
</template>

<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>
```

## Сравнение с другими подходами

| Характеристика | Vue Router | [[Клиентский-роутинг]] | [[Серверный-роутинг]] |
|----------------|------------|------------------------|----------------------|
| Реализация | Библиотека | Встроенный механизм | Серверный код |
| Интеграция | Официальная библиотека | Любая библиотека | Встроено в сервер |
| Сложность | Средняя | Высокая | Низкая |

## Заключение

Vue Router предоставляет мощный и гибкий способ реализации навигации в Vue-приложениях. Его интеграция с Vue позволяет создавать сложные маршруты с параметрами, вложенными компонентами и защитой доступа. Понимание принципов работы роутинга в Vue важно для создания качественных одностраничных приложений.

Для получения дополнительной информации о роутинге смотрите:
- [[Клиентский-роутинг]]
- [[Серверный-роутинг]]
- [[Роутинг-в-React]]
- [[Роутинг-в-Svelte]]

## Ключевые теги

#routing #vue #vue-router #frontend #javascript #spa #web-development