---
aliases: ["Вычисляемые свойства", "computed", "computed properties"]
tags: ["#vue", "#reactivity", "#computed", "#performance"]
---

# Computed (Вычисляемые свойства)

## Введение

Вычисляемые свойства (computed properties) в Vue.js - это мощный механизм для создания реактивных значений, которые автоматически обновляются при изменении зависимостей. Они позволяют создавать сложные вычисления с автоматической оптимизацией производительности за счет кеширования результатов.

## Основы вычисляемых свойств

Вычисляемые свойства определяются в блоке `computed` компонента и работают как обычные свойства, но с автоматическим кешированием. Они пересчитываются только при изменении зависимостей, что делает их эффективными для сложных вычислений.

### Простой пример

```vue
<template>
  <div>
    <p>Исходное имя: {{ firstName }}</p>
    <p>Полное имя: {{ fullName }}</p>
    <p>Количество символов: {{ fullNameLength }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      firstName: 'Иван',
      lastName: 'Иванов'
    }
  },
  computed: {
    fullName() {
      return `${this.firstName} ${this.lastName}`
    },
    fullNameLength() {
      return this.fullName.length
    }
  }
}
</script>
```

## Кеширование и производительность

Одним из ключевых преимуществ вычисляемых свойств является их кеширование. Vue отслеживает реактивные зависимости, используемые в вычисляемом свойстве, и пересчитывает его только при изменении этих зависимостей.

```javascript
computed: {
  // Это вычисляемое свойство будет пересчитано только при изменении this.items
  processedItems() {
    console.log('Пересчет processedItems')
    return this.items.map(item => ({
      ...item,
      processed: true,
      processedAt: new Date()
    }))
  }
}
```

В отличие от методов, вызываемых в шаблоне, вычисляемые свойства не будут пересчитываться при каждом рендеринге компонента.

## Геттеры и сеттеры

Вычисляемые свойства могут иметь как геттер, так и сеттер, что позволяет создавать двусторонние вычисляемые связи.

```javascript
computed: {
  fullName: {
    get() {
      return `${this.firstName} ${this.lastName}`
    },
    set(newValue) {
      const names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

## Сравнение с методами

| Характеристика | Вычисляемое свойство | Метод |
|----------------|---------------------|-------|
| Кеширование | Да | Нет |
| Пересчет | При изменении зависимостей | При каждом вызове |
| Использование в шаблоне | {{ propertyName }} | {{ methodName() }} |

## Практические применения

### Фильтрация и сортировка

```javascript
computed: {
  filteredAndSortedUsers() {
    return this.users
      .filter(user => user.active)
      .sort((a, b) => a.name.localeCompare(b.name))
  }
}
```

### Сложные вычисления

```javascript
computed: {
  statistics() {
    const values = this.dataPoints.map(item => item.value)
    const sum = values.reduce((a, b) => a + b, 0)
    const average = sum / values.length
    const max = Math.max(...values)
    const min = Math.min(...values)
    
    return {
      sum,
      average,
      max,
      min,
      range: max - min
    }
  }
}
```

## Лучшие практики

1. **Используйте для производных данных** - вычисляемые свойства идеально подходят для преобразования и фильтрации исходных данных.

2. **Не мутируйте зависимости внутри вычисляемого свойства** - это может привести к неожиданным результатам.

3. **Избегайте сложных сайд-эффектов** - вычисляемые свойства должны быть чистыми функциями, возвращающими значение на основе зависимостей.

4. **Разделяйте сложные вычисления** - если вычисление становится слишком сложным, разбейте его на несколько более простых вычисляемых свойств.

## Интеграция с Composition API

В Composition API вычисляемые свойства создаются с помощью функции `computed`:

```javascript
import { ref, computed } from 'vue'

export default {
  setup() {
    const firstName = ref('Иван')
    const lastName = ref('Иванов')
    
    const fullName = computed(() => `${firstName.value} ${lastName.value}`)
    
    const fullNameReversed = computed({
      get: () => fullName.value.split('').reverse().join(''),
      set: (value) => {
        const original = value.split('').reverse().join('')
        const [first, last] = original.split(' ')
        firstName.value = first
        lastName.value = last
      }
    })
    
    return {
      firstName,
      lastName,
      fullName,
      fullNameReversed
    }
  }
}
```

## Обработка ошибок

В редких случаях, когда вычисляемое свойство может выбросить ошибку, рекомендуется оборачивать потенциально опасные операции в try-catch:

```javascript
computed: {
  safeCalculation() {
    try {
      return this.complexData.reduce((acc, item) => acc + item.value, 0) / this.complexData.length
    } catch (error) {
      console.error('Ошибка при вычислении:', error)
      return 0
    }
  }
}
```

## Связь с другими концепциями Vue

Вычисляемые свойства тесно связаны с другими аспектами реактивности Vue:

- [[Реактивные-объявления]] - основа, на которой строится реактивность
- [[Watch]] - альтернатива для выполнения сайд-эффектов
- [[WatchEffect]] - современный подход к наблюдению за изменениями
- [[Оптимизация]] - как вычисляемые свойства влияют на производительность

## Заключение

Вычисляемые свойства являются важной частью арсенала разработчика Vue.js, обеспечивая эффективный способ работы с производными данными. Правильное использование computed-свойств позволяет создавать высокопроизводительные приложения с чистой и понятной логикой.

Для максимальной эффективности рекомендуется сочетать вычисляемые свойства с другими возможностями реактивности Vue, такими как [[Лучшие-практики]] для построения масштабируемых приложений.