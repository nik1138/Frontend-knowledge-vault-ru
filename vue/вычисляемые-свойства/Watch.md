---
aliases: ["Наблюдатели", "watch", "watcher", "реактивные наблюдатели"]
tags: ["#vue", "#reactivity", "#watch", "#side-effects"]
---

# Watch (Наблюдатели)

## Введение

Наблюдатели (watchers) в Vue.js - это механизм, позволяющий реагировать на изменения реактивных данных и выполнять побочные эффекты при их изменении. В отличие от вычисляемых свойств, наблюдатели предназначены для выполнения асинхронных или сложных синхронных операций в ответ на изменения состояния.

## Основы использования

Наблюдатели определяются в блоке `watch` компонента и вызываются при изменении отслеживаемого свойства.

### Простой пример

```vue
<template>
  <div>
    <input v-model="query" placeholder="Введите запрос поиска">
    <p>Поиск: {{ query }}</p>
    <ul>
      <li v-for="result in results" :key="result.id">{{ result.title }}</li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      query: '',
      results: []
    }
  },
  watch: {
    query(newQuery, oldQuery) {
      if (newQuery !== oldQuery) {
        this.search()
      }
    }
  },
  methods: {
    async search() {
      if (this.query.trim()) {
        try {
          // Имитация асинхронного поиска
          this.results = await this.fetchSearchResults(this.query)
        } catch (error) {
          console.error('Ошибка поиска:', error)
        }
      } else {
        this.results = []
      }
    },
    async fetchSearchResults(query) {
      // Здесь будет реальный API-вызов
      return [
        { id: 1, title: `Результат для "${query}" - 1` },
        { id: 2, title: `Результат для "${query}" - 2` }
      ]
    }
  }
}
</script>
```

## Глубокое наблюдение

По умолчанию наблюдатели отслеживают только поверхностные изменения. Для отслеживания изменений вложенных свойств используется опция `deep`.

```javascript
watch: {
  user: {
    handler(newVal, oldVal) {
      console.log('Пользователь изменился:', newVal, oldVal)
    },
    deep: true // Включает глубокое наблюдение
  }
}
```

## Немедленное выполнение

Опция `immediate` позволяет выполнить обработчик наблюдателя сразу при создании компонента, а не только при изменении значения.

```javascript
watch: {
  userId: {
    handler(newId, oldId) {
      this.fetchUserData(newId)
    },
    immediate: true // Вызовется сразу при создании
  }
}
```

## Наблюдение за массивами

При наблюдении за массивами можно использовать опцию `deep` для отслеживания изменений элементов:

```javascript
watch: {
  items: {
    handler(newItems, oldItems) {
      console.log('Массив изменился', newItems, oldItems)
    },
    deep: true,
    // Опционально: отслеживание длины массива
    flush: 'post' // Выполнение после обновления DOM
  }
}
```

## Наблюдение за несколькими свойствами

Можно наблюдать за несколькими свойствами одновременно:

```javascript
watch: {
  // Наблюдение за несколькими свойствами
  '$route'(to, from) {
    this.fetchData()
  },
  locale(newLocale, oldLocale) {
    this.updateTranslations()
  }
}
```

## Наблюдение за вычисляемыми свойствами

Наблюдатель можно установить и на вычисляемое свойство:

```javascript
computed: {
  fullName() {
    return `${this.firstName} ${this.lastName}`
  }
},
watch: {
  fullName(newName, oldName) {
    console.log(`Полное имя изменилось с "${oldName}" на "${newName}"`)
  }
}
```

## Динамические наблюдатели

В редких случаях можно создавать наблюдатели программно:

```javascript
// В методе или жизненном цикле компонента
this.$watch('someValue', (newVal, oldVal) => {
  console.log('Значение изменилось:', newVal, oldVal)
}, {
  immediate: true,
  deep: true
})
```

## Интеграция с Composition API

В Composition API наблюдатели создаются с помощью функции `watch`:

```javascript
import { ref, watch, watchEffect } from 'vue'

export default {
  setup() {
    const query = ref('')
    const results = ref([])

    // Простой наблюдатель
    watch(query, async (newQuery, oldQuery) => {
      if (newQuery !== oldQuery) {
        results.value = await fetchSearchResults(newQuery)
      }
    })

    // Наблюдатель с опциями
    watch(
      () => this.user.profile,
      (newProfile, oldProfile) => {
        console.log('Профиль пользователя изменился')
      },
      { deep: true, immediate: false }
    )

    return {
      query,
      results
    }
  }
}
```

## Практические применения

### Автосохранение данных

```javascript
watch: {
  formData: {
    handler(newData) {
      // Отложенное автосохранение
      clearTimeout(this.saveTimeout)
      this.saveTimeout = setTimeout(() => {
        this.saveFormData(newData)
      }, 1000)
    },
    deep: true
  }
}
```

### Синхронизация с внешними API

```javascript
watch: {
  '$route.params.id': {
    handler(newId) {
      if (newId) {
        this.loadItem(newId)
      }
    },
    immediate: true
  }
}
```

### Управление состоянием с localStorage

```javascript
watch: {
  preferences: {
    handler(newPrefs) {
      localStorage.setItem('userPreferences', JSON.stringify(newPrefs))
    },
    deep: true
  }
}
```

## Оптимизация производительности

1. **Избегайте ненужного глубокого наблюдения** - используйте `deep: true` только при необходимости.

2. **Дебаунсинг** - для частых изменений (например, ввода текста) используйте дебаунсинг:

```javascript
import { debounce } from 'lodash-es'

const debouncedSearch = debounce(async (query) => {
  results.value = await fetchSearchResults(query)
}, 300)

watch(query, debouncedSearch)
```

3. **Ограничьте количество наблюдателей** - чрезмерное количество наблюдателей может повлиять на производительность.

## Сравнение с другими подходами

| Характеристика | Watch | Computed | WatchEffect |
|----------------|-------|----------|-------------|
| Назначение | Сайд-эффекты | Производные данные | Сайд-эффекты |
| Кеширование | Нет | Да | Нет |
| Отслеживание зависимостей | Явное | Автоматическое | Автоматическое |
| Подходит для асинхронных операций | Да | Нет* | Да |

*Вычисляемые свойства не должны содержать асинхронных операций, так как это может нарушить кеширование.

## Связь с другими концепциями Vue

Наблюдатели тесно связаны с другими аспектами реактивности Vue:

- [[Computed]] - альтернатива для производных данных
- [[WatchEffect]] - современный подход к наблюдению
- [[Побочные-эффекты]] - общая концепция, которую реализуют наблюдатели
- [[Реактивные-объявления]] - основа, на которой строятся наблюдатели

## Заключение

Наблюдатели (watch) являются мощным инструментом для выполнения побочных эффектов в ответ на изменения данных. Они особенно полезны для асинхронных операций, таких как сетевые запросы, автосохранение и синхронизация с внешними API. 

При правильном использовании наблюдатели позволяют создавать реактивные приложения, которые эффективно реагируют на изменения состояния. Важно понимать разницу между `watch`, `computed` и `watchEffect`, чтобы выбирать наиболее подходящий инструмент для каждой конкретной задачи.

Для оптимальной производительности рекомендуется использовать наблюдатели в сочетании с другими подходами, описанными в [[Лучшие-практики]] и [[Оптимизация]].