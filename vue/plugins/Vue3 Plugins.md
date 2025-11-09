# Plugins

Плагины Vue - это самодостаточные объекты, которые расширяют функциональность Vue приложения. Они могут добавлять глобальные API, компоненты, директивы, методы и другие возможности.

## Основные концепции

### [Plugin Structure](structure.md)
Структура и создание плагинов

### [Installation](installation.md)
Установка и регистрация плагинов

### [Global API Extension](global-api.md)
Расширение глобального API приложения

### [Component & Directive Registration](registration.md)
Регистрация глобальных компонентов и директив

### [State Management Plugins](state-plugins.md)
Плагины для управления состоянием (Pinia, Vuex)

### [UI Frameworks](ui-frameworks.md)
Популярные UI фреймворки как плагины

## Создание плагинов

### Простой плагин
```javascript
// myPlugin.js
export const myPlugin = {
  install(app, options) {
    // Добавление глобального метода
    app.config.globalProperties.$myMethod = function() {
      console.log('Мой метод')
    }
    
    // Добавление глобальной директивы
    app.directive('focus', {
      mounted(el) {
        el.focus()
      }
    })
    
    // Добавление глобального компонента
    app.component('GlobalButton', {
      template: '<button><slot /></button>'
    })
  }
}

// Использование
app.use(myPlugin)
```

### Функциональный плагин
```javascript
export function createMyPlugin(options) {
  return {
    install(app) {
      // логика установки
    }
  }
}
```

## Популярные плагины

### Vue Router
```javascript
import { createRouter, createWebHistory } from 'vue-router'
import { createApp } from 'vue'
import App from './App.vue'

const router = createRouter({
  history: createWebHistory(),
  routes: []
})

const app = createApp(App)
app.use(router)
```

### Pinia
```javascript
import { createPinia } from 'pinia'
import { createApp } from 'vue'
import App from './App.vue'

const pinia = createPinia()
const app = createApp(App)
app.use(pinia)
```

## Параметры установки

### Опциональные параметры
```javascript
app.use(myPlugin, {
  option1: 'value1',
  option2: 'value2'
})
```

## Плагины и Composition API

### Использование в композаблах
```javascript
import { getCurrentInstance } from 'vue'

export function usePluginFeature() {
  const instance = getCurrentInstance()
  const myMethod = instance.appContext.config.globalProperties.$myMethod
  
  return { myMethod }
}
```

## Сторонние плагины

### Axios для HTTP запросов
```javascript
import axios from 'axios'
import { createApp } from 'vue'

const app = createApp(App)

// Добавление axios в глобальные свойства
app.config.globalProperties.$http = axios
```

## Лучшие практики

### Проверка установки
```javascript
export const myPlugin = {
  install(app) {
    if (this.installed) return
    
    this.installed = true
    // установка плагина
  }
}
```

### Использование provide/inject
```javascript
export const myPlugin = {
  install(app) {
    app.provide('myService', myServiceInstance)
  }
}
```

## Связь с другими концепциями
- [[components/Component Registration]] - регистрация компонентов через плагины
- [[vue3 Directives]] - регистрация директив через плагины
- [[state-management/State Management]] - плагины управления состоянием
- [[routing/Routing]] - плагин маршрутизации