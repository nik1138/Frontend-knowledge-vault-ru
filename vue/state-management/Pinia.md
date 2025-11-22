---
aliases: []
tags: [vue, state-management, pinia]
---

# Pinia

Pinia - это официальный рекомендуемый инструмент управления состоянием для Vue.js. Это современное, легкое решение для управления состоянием, которое заменило Vuex как предпочтительный способ централизованного управления состоянием в приложениях Vue.

## Основные особенности Pinia

- Простота в использовании
- Поддержка TypeScript "из коробки"
- Легкая установка и интеграция
- Поддержка модулей для организации состояния
- Встроенная поддержка DevTools
- Нет необходимости оборачивать состояние в actions для асинхронных операций

## Создание хранилища

```javascript
import { defineStore } from 'pinia'

export const useMainStore = defineStore('main', {
  // состояние
  state: () => ({
    counter: 0,
    name: 'Eduardo',
    isAdmin: true
  }),
  
  // геттеры
  getters: {
    doubleCount: (state) => state.counter * 2,
    doubleCountPlusOne(): number {
      return this.doubleCount + 1
    }
  },
  
  // действия
  actions: {
    increment() {
      this.counter++
    },
    randomizeCounter() {
      this.counter = Math.floor(Math.random() * 100)
    }
  }
})
```

## Использование в компонентах

```javascript
import { useMainStore } from '@/stores/main'

export default {
  setup() {
    const main = useMainStore()
    
    return { main }
  }
}
```

## Преимущества перед Vuex

- Более простой и понятный API
- Лучшая поддержка TypeScript
- Более интуитивная работа с асинхронными действиями
- Лучшая производительность
- Проще для новичков

См. также: [[Vuex]], [[State Management]], [[Vue компоненты]]