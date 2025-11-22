---
aliases: []
tags: [vue, api, composition-api]
---

# Composition API

Composition API - это набор API в Vue.js, который позволяет определять и структурировать логику компонентов Vue с использованием импортированных функций. Он предоставляет более гибкий способ организации логики компонентов по функциональным возможностям, а не по опциям (как в Options API).

## Основные функции Composition API

- `setup()` - входная точка для Composition API
- `ref()` и `reactive()` - для создания реактивных состояний
- `computed()` - для вычисляемых свойств
- `watch()` и `watchEffect()` - для наблюдения за изменениями
- `provide()` и `inject()` - для внедрения зависимостей

## Преимущества Composition API

Composition API решает несколько проблем, связанных с Options API:

1. Лучшая логическая группировка - связанный код может быть сгруппирован вместе
2. Лучшее переиспользование логики между компонентами через Composables
3. Лучшая поддержка TypeScript
4. Более предсказуемая система оптимизации

## Пример использования

```javascript
import { ref, computed, watch } from 'vue'

export default {
  setup() {
    const count = ref(0)
    const doubleCount = computed(() => count.value * 2)
    
    watch(doubleCount, (newVal) => {
      console.log(`New doubled value: ${newVal}`)
    })
    
    function increment() {
      count.value++
    }
    
    return {
      count,
      doubleCount,
      increment
    }
  }
}
```

## Сравнение с Options API

Composition API позволяет избежать ограничений Options API, особенно при создании сложных компонентов, где логика часто разбивается на несколько опций (data, methods, computed, watch), что затрудняет понимание и поддержку кода.

См. также: [[Options API]], [[Vue компоненты]], [[Reactivity System]]