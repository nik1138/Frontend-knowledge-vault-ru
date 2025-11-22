---
aliases: []
tags: [vue, state-management, vuex]
---

# Vuex

Vuex - это официальный инструмент управления состоянием для Vue.js. Это централизованное хранилище для всех компонентов приложения, реализующее паттерн Flux. Vuex помогает управлять общим состоянием приложения, делая изменения состояния предсказуемыми через строгую архитектуру.

## Архитектура Vuex

Vuex состоит из четырех основных концепций:

1. **State** - исходное состояние приложения
2. **Getters** - вычисляемые свойства для хранилища
3. **Mutations** - методы для изменения состояния (только синхронные)
4. **Actions** - методы для выполнения асинхронных операций и вызова мутаций

## Пример использования

```javascript
import { createStore } from 'vuex'

const store = createStore({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    }
  },
  actions: {
    incrementAsync({ commit }) {
      setTimeout(() => {
        commit('increment')
      }, 1000)
    }
  },
  getters: {
    doubleCount: state => {
      return state.count * 2
    }
  }
})
```

## Модули

Vuex позволяет разбивать хранилище на модули для лучшей организации:

```javascript
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})
```

## Интеграция с компонентами Vue

В компонентах можно получить доступ к хранилищу через `this.$store`:

```javascript
export default {
  computed: {
    count() {
      return this.$store.state.count
    },
    doubleCount() {
      return this.$store.getters.doubleCount
    }
  },
  methods: {
    increment() {
      this.$store.commit('increment')
    }
  }
}
```

## Сравнение с Pinia

Pinia теперь рекомендуется как современная альтернатива Vuex с более простым и интуитивным API. Vuex остается поддерживаемым, но для новых проектов рекомендуется использовать Pinia.

См. также: [[Pinia]], [[State Management]], [[Vue компоненты]]