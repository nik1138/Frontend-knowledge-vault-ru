---
aliases: []
tags: [vue, api, options-api]
---

# Options API

Options API - это традиционный способ определения компонентов Vue.js, при котором логика компонента организуется в предопределенные опции, такие как data, methods, computed, watch и другие. Это оригинальный подход к созданию компонентов Vue, который был доступен с самого начала фреймворка.

## Основные опции компонента

- `data` - реактивное состояние компонента
- `methods` - методы для обработки событий и бизнес-логики
- `computed` - вычисляемые свойства
- `watch` - наблюдатели за изменениями данных
- `created`, `mounted`, `updated`, `unmounted` - хуки жизненного цикла
- `props`, `emits` - определение входных параметров и событий
- `components` - регистрация дочерних компонентов

## Пример использования

```javascript
export default {
  name: 'UserProfile',
  props: {
    userId: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      user: null,
      loading: false
    }
  },
  computed: {
    fullName() {
      return this.user ? `${this.user.firstName} ${this.user.lastName}` : ''
    }
  },
  methods: {
    async fetchUser() {
      this.loading = true
      try {
        this.user = await api.getUser(this.userId)
      } finally {
        this.loading = false
      }
    }
  },
  created() {
    this.fetchUser()
  }
}
```

## Преимущества Options API

- Интуитивно понятен для новичков
- Хорошо структурирован для простых компонентов
- Отличная интеграция с Vue DevTools
- Ясная структура компонента

## Недостатки Options API

- Сложность при работе с большими компонентами
- Трудности с повторным использованием логики между компонентами
- Ограничения при использовании с TypeScript

См. также: [[Composition API]], [[Vue компоненты]], [[Vue Lifecycle Hooks]]