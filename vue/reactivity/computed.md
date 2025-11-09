# computed

## Определение
`computed` - это функция в Composition API Vue 3, которая создает вычисляемое (реактивное) свойство на основе других реактивных значений. Вычисляемое свойство автоматически обновляется при изменении зависимостей.

## Синтаксис
```javascript
import { computed } from 'vue'

const computedValue = computed(() => {
  // вычисления
  return result
})
```

## Особенности
- Автоматически отслеживает зависимости
- Кэширует результат до изменения зависимостей
- Реактивен - обновляется при изменении зависимостей
- Может быть как геттером, так и геттером/сеттером

## Примеры использования

### Простой геттер
```javascript
import { ref, computed } from 'vue'

export default {
  setup() {
    const firstName = ref('John')
    const lastName = ref('Doe')
    
    const fullName = computed(() => {
      return `${firstName.value} ${lastName.value}`
    })
    
    return {
      firstName,
      lastName,
      fullName
    }
  }
}
```

### Геттер и сеттер
```javascript
import { ref, computed } from 'vue'

export default {
  setup() {
    const firstName = ref('John')
    const lastName = ref('Doe')
    
    const fullName = computed({
      get: () => {
        return `${firstName.value} ${lastName.value}`
      },
      set: (newValue) => {
        const parts = newValue.split(' ')
        firstName.value = parts[0]
        lastName.value = parts[1]
      }
    })
    
    return {
      firstName,
      lastName,
      fullName
    }
  }
}
```

### Использование в шаблоне
```vue
<template>
  <div>
    <p>Имя: {{ firstName }}</p>
    <p>Фамилия: {{ lastName }}</p>
    <p>Полное имя: {{ fullName }}</p>
    <input v-model="fullName" placeholder="Введите полное имя">
  </div>
</template>
```

## Сравнение с методами
- Методы вызываются при каждом рендере
- `computed` кэширует результат и пересчитывает только при изменении зависимостей
- Эффективнее для сложных вычислений

## Связь с другими концепциями
- [[ref]] - для создания реактивных значений
- [[reactive]] - для реактивных объектов
- [[watch]] - для отслеживания изменений
- [[watchEffect]] - для побочных эффектов

## Примечания
- Не используйте `computed` для операций с побочными эффектами
- Используйте `watch` или `watchEffect` для асинхронных операций