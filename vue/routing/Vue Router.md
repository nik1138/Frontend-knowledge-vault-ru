# Vue Router

Vue Router - это официальная библиотека маршрутизации для Vue.js. Она позволяет создавать одностраничные приложения (SPA) с навигацией между различными представлениями.

## Основные понятия

### [Router Instance](router-instance.md)
Создание и настройка экземпляра роутера

### [Routes](routes.md)
Определение маршрутов и их конфигурация

### [Navigation](navigation.md)
Навигация между маршрутами (программная и через ссылки)

### [Route Parameters](route-params.md)
Работа с параметрами маршрутов и динамическими сегментами

### [Nested Routes](nested-routes.md)
Вложенные маршруты и вложенные компоненты

### [Route Guards](route-guards.md)
Хуки для контроля навигации и аутентификации

## Установка и начальная настройка

```bash
npm install vue-router@4
```

### Базовая настройка
```javascript
import { createRouter, createWebHistory } from 'vue-router'
import Home from './views/Home.vue'
import About from './views/About.vue'

const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

### Использование в приложении
```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'

createApp(App).use(router).mount('#app')
```

## Основные компоненты

### RouterView
Компонент `<router-view>` отображает компонент, соответствующий текущему маршруту.

### RouterLink
Компонент `<router-link>` используется для навигации между маршрутами без перезагрузки страницы.

## Composition API и маршруты

```javascript
import { useRouter, useRoute } from 'vue-router'

export default {
  setup() {
    const router = useRouter()
    const route = useRoute()
    
    // Доступ к текущему маршруту
    console.log(route.params)
    console.log(route.query)
    
    // Программная навигация
    const navigate = () => {
      router.push('/about')
    }
    
    return { navigate }
  }
}
```

## Связь с другими концепциями
- [[components/Components]] - маршруты как компоненты представлений
- [[state-management/State Management]] - управление состоянием в роутах
- [[lifecycle/Lifecycle Hooks]] - жизненные циклы при навигации