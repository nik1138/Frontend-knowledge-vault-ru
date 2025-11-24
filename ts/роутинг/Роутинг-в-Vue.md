---
aliases: [Vue Router, Роутинг в Vue, Vue Navigation]
tags: [vue, routing, typescript, frontend]
---

# Роутинг в Vue

## Обзор

Роутинг в Vue.js реализуется с помощью официальной библиотеки Vue Router, которая позволяет создавать одностраничные приложения с навигацией между различными представлениями без перезагрузки страницы. Vue Router интегрируется с системой компонентов Vue, обеспечивая плавную навигацию и управление состоянием приложения.

## Установка и настройка

```bash
npm install vue-router@4
```

### Базовая настройка

```typescript
// router/index.ts
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router';
import Home from '../views/Home.vue';
import About from '../views/About.vue';
import User from '../views/User.vue';

const routes: Array<RouteRecordRaw> = [
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
    path: '/user/:id',
    name: 'User',
    component: User,
    props: true // передача параметров как пропсов
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

export default router;
```

### Подключение к приложению

```typescript
// main.ts
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

const app = createApp(App);

app.use(router);
app.mount('#app');
```

## Основные концепции

### RouteRecordRaw

```typescript
interface RouteRecordRaw {
  path: string;
  name?: string | symbol;
  component?: Component;
  components?: Record<string, Component>;
  redirect?: RouteRecordRedirectOption;
  props?: boolean | Record<string, any> | RoutePropsFunction;
  alias?: string | Array<string>;
  children?: RouteRecordRaw[];
  meta?: RouteMeta;
  // другие свойства...
}
```

### RouteMeta

```typescript
// types/routeMeta.ts
interface RouteMeta {
  requiresAuth?: boolean;
  role?: string;
  title?: string;
  layout?: string;
}

// Использование в маршрутах
const routes: Array<RouteRecordRaw> = [
  {
    path: '/admin',
    component: Admin,
    meta: {
      requiresAuth: true,
      role: 'admin',
      title: 'Админ панель'
    }
  }
];
```

## Навигация

### Декларативная навигация

```vue
<template>
  <!-- Использование <router-link> -->
  <router-link to="/">Главная</router-link>
  <router-link :to="{ name: 'User', params: { id: 123 } }">Пользователь</router-link>
  
  <!-- Активный класс автоматически добавляется -->
  <router-link to="/about" active-class="active">О нас</router-link>
  
  <!-- Exact активный класс -->
  <router-link to="/" exact-active-class="exact-active">Главная</router-link>
</template>
```

### Программная навигация

```typescript
// В компоненте Vue
import { useRouter } from 'vue-router';

export default {
  setup() {
    const router = useRouter();
    
    const goToUser = (id: number) => {
      // Навигация по имени маршрута
      router.push({ name: 'User', params: { id } });
      
      // Навигация с query параметрами
      router.push({ path: '/search', query: { q: 'vue' } });
      
      // Замена текущей записи в истории
      router.replace({ path: '/login' });
    };
    
    return { goToUser };
  }
};
```

## Параметры маршрута

### Route Props

```typescript
// router/index.ts
{
  path: '/user/:id',
  name: 'User',
  component: User,
  props: true // передача параметров как пропсов
}

// components/User.vue
<script setup lang="ts">
interface Props {
  id: string;
}

const props = defineProps<Props>();
console.log('User ID:', props.id);
</script>
```

### Route Query и Hash

```typescript
import { useRoute } from 'vue-router';

export default {
  setup() {
    const route = useRoute();
    
    // Получение query параметров
    const query = route.query; // { page: '1', limit: '10' }
    
    // Получение hash
    const hash = route.hash; // '#section'
    
    return { query, hash };
  }
};
```

## Вложенные маршруты

```typescript
// router/index.ts
{
  path: '/users',
  component: UsersLayout,
  children: [
    {
      path: '',
      name: 'Users',
      component: UsersList
    },
    {
      path: ':id',
      name: 'UserProfile',
      component: UserProfile,
      props: true
    },
    {
      path: ':id/posts',
      name: 'UserPosts',
      component: UserPosts,
      props: true
    }
  ]
}
```

```vue
<!-- components/UsersLayout.vue -->
<template>
  <div>
    <h1>Пользователи</h1>
    <nav>
      <router-link to="/users">Список пользователей</router-link>
    </nav>
    <!-- Вложенные маршруты отображаются здесь -->
    <router-view />
  </div>
</template>
```

## Защита маршрутов

### Глобальные guards

```typescript
// router/index.ts
router.beforeEach((to, from) => {
  // Проверка аутентификации
  const isAuthenticated = checkAuth();
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    return {
      path: '/login',
      query: { redirect: to.fullPath }
    };
  }
  
  // Проверка прав доступа
  if (to.meta.role && !hasRole(to.meta.role)) {
    return '/unauthorized';
  }
  
  // Установка заголовка страницы
  if (to.meta.title) {
    document.title = to.meta.title as string;
  }
});

router.afterEach((to, from) => {
  // Логика после навигации
  console.log(`Навигация с ${from.path} на ${to.path}`);
});
```

### Перед маршрутом (beforeRouteEnter)

```vue
<!-- components/Protected.vue -->
<script setup lang="ts">
import { onBeforeRouteLeave, onBeforeRouteUpdate } from 'vue-router';

// Вызывается перед входом в маршрут
onBeforeRouteEnter((to, from, next) => {
  // Доступ к компоненту через next
  next((vm) => {
    // vm - экземпляр компонента
  });
});

// Вызывается перед обновлением маршрута (при изменении параметров)
onBeforeRouteUpdate(async (to, from) => {
  // Можно выполнить асинхронные операции
  if (from.params.id !== to.params.id) {
    // Обновить данные компонента
  }
});

// Вызывается перед уходом с маршрута
onBeforeRouteLeave((to, from) => {
  const answer = window.confirm('Вы уверены, что хотите покинуть страницу?');
  if (!answer) return false;
});
</script>
```

## Ленивая загрузка

```typescript
// router/index.ts
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'Home',
    component: () => import('../views/Home.vue') // ленивая загрузка
  },
  {
    path: '/admin',
    name: 'Admin',
    component: () => import('../views/Admin.vue'),
    meta: {
      requiresAuth: true
    }
  }
];

// Для группировки в чанки
const AdminView = () => import(
  /* webpackChunkName: "admin" */ 
  '../views/Admin.vue'
);
```

## Обработка ошибок

### Глобальный обработчик ошибок

```typescript
// main.ts
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';

const app = createApp(App);

// Обработка ошибок маршрутизации
router.onError((error) => {
  if (error.message.includes('Failed to fetch dynamically imported module')) {
    // Ошибка загрузки чанка - возможно, файл изменился
    console.error('Ошибка загрузки модуля:', error);
    // Можно предложить пользователю обновить страницу
    alert('Произошла ошибка загрузки. Пожалуйста, обновите страницу.');
  }
});

app.use(router);
app.mount('#app');
```

### Компонент для отображения ошибок

```vue
<!-- components/RouteError.vue -->
<template>
  <div class="error-container">
    <h1>Ошибка маршрута</h1>
    <p v-if="error">{{ error }}</p>
    <button @click="goHome">Вернуться на главную</button>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();
const error = ref<string | null>(null);

onMounted(() => {
  // Получить ошибку из мета данных или параметров
  error.value = route.meta.error as string || null;
});

const goHome = () => {
  router.push('/');
};
</script>
```

## Лучшие практики

### Типизированные маршруты

```typescript
// types/router.ts
import { RouteRecordRaw } from 'vue-router';

export interface AppRouteRecordRaw extends RouteRecordRaw {
  name: string;
  meta?: {
    title: string;
    requiresAuth?: boolean;
    role?: string;
    layout?: string;
  };
}

// router/index.ts
const routes: AppRouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('../views/Home.vue'),
    meta: {
      title: 'Главная страница'
    }
  }
];
```

### Структура файлов

```
src/
├── router/
│   ├── index.ts
│   ├── routes/
│   │   ├── public.ts
│   │   ├── protected.ts
│   │   └── admin.ts
│   └── guards/
│       ├── auth.ts
│       └── permission.ts
├── views/
│   ├── Home.vue
│   ├── About.vue
│   └── User/
│       ├── List.vue
│       └── Detail.vue
└── components/
    └── layouts/
        ├── MainLayout.vue
        └── AuthLayout.vue
```

### Модульные маршруты

```typescript
// router/routes/public.ts
import { AppRouteRecordRaw } from '../types';

export const publicRoutes: AppRouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('../../views/Home.vue'),
    meta: { title: 'Главная' }
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('../../views/About.vue'),
    meta: { title: 'О нас' }
  }
];

// router/routes/protected.ts
import { AppRouteRecordRaw } from '../types';

export const protectedRoutes: AppRouteRecordRaw[] = [
  {
    path: '/dashboard',
    name: 'Dashboard',
    component: () => import('../../views/Dashboard.vue'),
    meta: {
      title: 'Панель управления',
      requiresAuth: true
    }
  }
];
```

### Управление скроллом

```typescript
// router/index.ts
const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    // Если есть сохраненная позиция (например, при возвращении назад)
    if (savedPosition) {
      return savedPosition;
    }
    
    // Если маршрут имеет якорь
    if (to.hash) {
      return {
        el: to.hash,
        behavior: 'smooth'
      };
    }
    
    // В остальных случаях прокрутка к верху
    return { top: 0 };
  }
});
```

## Миграция с Vue Router 3

### Основные отличия

```typescript
// Vue Router 3 (устаревший)
import Vue from 'vue';
import VueRouter from 'vue-router';

Vue.use(VueRouter);

const router = new VueRouter({
  mode: 'history',
  routes: [...]
});

// Vue Router 4 (современный)
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(),
  routes: [...]
});
```

### Composition API и маршруты

```vue
<!-- Vue Router 3 -->
<script>
export default {
  computed: {
    currentRoute() {
      return this.$route;
    }
  },
  methods: {
    navigateTo(path) {
      this.$router.push(path);
    }
  }
};
</script>

<!-- Vue Router 4 с Composition API -->
<script setup lang="ts">
import { computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

const currentRoute = computed(() => route);

const navigateTo = (path: string) => {
  router.push(path);
};
</script>
```

## Заключение

Роутинг в Vue с использованием Vue Router предоставляет мощный и гибкий способ управления навигацией в приложении. Правильное использование типизации, защита маршрутов и эффективная организация структуры приложения позволяют создавать масштабируемые и поддерживаемые приложения.

См. также: [[Клиентский-роутинг]], [[Серверный-роутинг]], [[Роутинг-в-React]], [[Роутинг-в-Angular]]