---
aliases: [Создание неизменяемых реактивных объектов, readonly в Vue]
tags: [vue, реактивность, readonly, javascript]
---

# Readonly

`readonly` - это функция в Vue 3, которая создает неизменяемую (только для чтения) реактивную версию исходного объекта или ref. Она предотвращает любые изменения в объекте, но сохраняет реактивность, что позволяет отслеживать изменения исходного объекта, если он изменяется извне.

## Основное использование

```javascript
import { reactive, readonly } from 'vue'

const original = reactive({ count: 0 })
const copy = readonly(original)

// Отслеживание изменений
original.count++ // Работает
// copy.count++ // ОШИБКА! Нельзя изменять readonly объект
```

## Особенности readonly

- Создает реактивный объект, который нельзя изменить
- Сохраняет все реактивные связи
- Генерирует ошибку при попытке изменить свойства в development режиме
- Полезно для передачи данных в дочерние компоненты, которые не должны изменять данные

## Пример с компонентами

```vue
<!-- Родительский компонент -->
<template>
  <ChildComponent :data="readonlyData" />
</template>

<script setup>
import { reactive, readonly } from 'vue'
import ChildComponent from './ChildComponent.vue'

const data = reactive({
  message: 'Привет, мир!',
  count: 0
})

const readonlyData = readonly(data)
</script>
```

```vue
<!-- Дочерний компонент -->
<template>
  <div>
    <p>{{ data.message }}</p>
    <p>Счетчик: {{ data.count }}</p>
    <!-- Кнопка не будет работать как ожидается -->
    <button @click="tryToChange">Попытка изменить</button>
  </div>
</template>

<script setup>
import { toRefs } from 'vue'

const props = defineProps({
  data: {
    type: Object,
    required: true
  }
})

const tryToChange = () => {
  // Это вызовет ошибку в development режиме
  // props.data.count++ // ОШИБКА!
}
</script>
```

## Использование с ref

```javascript
import { ref, readonly } from 'vue'

const originalRef = ref(0)
const readonlyRef = readonly(originalRef)

// originalRef.value = 5 // Работает
// readonlyRef.value = 10 // ОШИБКА!
```

## Практические применения

1. **Защита состояния**: Предотвращение несанкционированных изменений данных
2. **Компоненты только для чтения**: Передача данных в компоненты, которые не должны их изменять
3. **Публичные API**: Предоставление доступа к данным без возможности их изменения
4. **Кэширование**: Обеспечение неизменности кэшированных данных

## Вложенные объекты

`readonly` также работает с вложенными объектами:

```javascript
import { reactive, readonly } from 'vue'

const original = reactive({
  user: {
    profile: {
      name: 'John',
      settings: {
        theme: 'dark'
      }
    }
  }
})

const readonlyOriginal = readonly(original)

// original.user.profile.name = 'Jane' // Работает
// readonlyOriginal.user.profile.name = 'Jane' // ОШИБКА!
```

## Совместимость с toRefs

При использовании `toRefs` с `readonly` результат также будет неизменяемым:

```javascript
import { reactive, readonly, toRefs } from 'vue'

const original = reactive({ count: 0, name: 'Vue' })
const readonlyOriginal = readonly(original)
const { count, name } = toRefs(readonlyOriginal)

// count.value = 10 // ОШИБКА!
```

## Лучшие практики

- Используйте `readonly` для защиты данных от изменений
- Особенно полезно при передаче данных в сторонние библиотеки или компоненты
- В production режиме ошибки не генерируются, но изменения все равно не применяются
- Объединяйте с `computed` для создания сложных вычисляемых значений только для чтения

## См. также

- [[Reactive]]
- [[Ref]]
- [[ToRefs]]
- [[IsRef]]