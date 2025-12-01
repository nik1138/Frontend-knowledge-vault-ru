---
aliases: [Создание реактивных объектов, reactive в Vue]
tags: [vue, реактивность, reactive, javascript]
---

# Reactive

`reactive` - это функция в Vue 3, которая делает объект реактивным. В отличие от `ref`, которая работает с примитивными значениями, `reactive` используется для создания реактивных объектов. Она возвращает прокси-объект, который отслеживает изменения его свойств.

## Основное использование

```javascript
import { reactive } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue',
  items: []
})

console.log(state.count) // 0
state.count++
console.log(state.count) // 1
```

## Особенности reactive

- `reactive` принимает объект и возвращает реактивную версию этого объекта
- Все вложенные объекты также становятся реактивными
- Не работает с примитивными значениями
- При деструктуризации теряется реактивность

## Пример в компоненте

```vue
<template>
  <div>
    <p>Счетчик: {{ state.count }}</p>
    <p>Имя: {{ state.name }}</p>
    <button @click="increment">Увеличить</button>
  </div>
</template>

<script setup>
import { reactive } from 'vue'

const state = reactive({
  count: 0,
  name: 'Vue 3'
})

const increment = () => {
  state.count++
}
</script>
```

## Потеря реактивности при деструктуризации

```javascript
import { reactive } from 'vue'

const state = reactive({ count: 0 })

// ПЛОХО - потеряется реактивность
const { count } = state

// ХОРОШО - сохраняется реактивность
const count = state.count
```

## Решение проблемы деструктуризации

Для сохранения реактивности при деструктуризации используйте `toRefs`:

```javascript
import { reactive, toRefs } from 'vue'

const state = reactive({ count: 0, name: 'Vue' })

// Сохраняет реактивность
const { count, name } = toRefs(state)
```

## Глубокая реактивность

`reactive` автоматически делает объекты глубоко реактивными:

```javascript
const state = reactive({
  user: {
    profile: {
      name: 'John',
      age: 30
    }
  }
})

// Изменения отслеживаются на всех уровнях
state.user.profile.age = 31
```

## Ограничения reactive

- Не может отслеживать добавление или удаление свойств
- Не работает с Map, Set, WeakMap, WeakSet (для них есть специальные функции)
- Не сохраняет реактивность при деструктуризации

## Лучшие практики

- Используйте `reactive` для объектов, а `ref` для примитивов
- Избегайте деструктуризации реактивных объектов без `toRefs`
- Используйте `readonly` для создания неизменяемых реактивных объектов

## См. также

- [[Ref]]
- [[ToRefs]]
- [[Readonly]]
- [[IsRef]]